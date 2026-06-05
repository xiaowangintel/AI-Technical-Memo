# check-declarations.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Semantics/check-declarations.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Static declaration checking.
- **Purpose (CN)**: 实现 check declarations 相关的语义检查、符号推理与语言规则约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- lib/Semantics/check-declarations.cpp ------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

// Static declaration checking

#include "check-declarations.h"
#include "definable.h"
#include "pointer-assignment.h"
#include "flang/Evaluate/check-expression.h"
#include "flang/Evaluate/fold.h"
#include "flang/Evaluate/tools.h"
#include "flang/Parser/characters.h"
#include "flang/Semantics/scope.h"
#include "flang/Semantics/semantics.h"
#include "flang/Semantics/symbol.h"
#include "flang/Semantics/tools.h"
#include "flang/Semantics/type.h"
#include <algorithm>
#include <map>
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, intent, or metadata: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明附近代码的逻辑、意图或元数据：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, intent, or metadata: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明附近代码的逻辑、意图或元数据：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, intent, or metadata: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明附近代码的逻辑、意图或元数据：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L9 EN**: Comment explains nearby logic, intent, or metadata: `Static declaration checking`.
  **L9 CN**: 注释说明附近代码的逻辑、意图或元数据：`Static declaration checking`。
- **L10 EN**: Blank line separating nearby declarations or logic blocks.
  **L10 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L11 EN**: Includes "check-declarations.h" to access local declarations paired with this implementation.
  **L11 CN**: 引入 "check-declarations.h" 以使用与该实现配套的本地声明。
- **L12 EN**: Includes "definable.h" to access local declarations paired with this implementation.
  **L12 CN**: 引入 "definable.h" 以使用与该实现配套的本地声明。
- **L13 EN**: Includes "pointer-assignment.h" to access local declarations paired with this implementation.
  **L13 CN**: 引入 "pointer-assignment.h" 以使用与该实现配套的本地声明。
- **L14 EN**: Includes "flang/Evaluate/check-expression.h" to access Fortran constant-folding and evaluation facilities.
  **L14 CN**: 引入 "flang/Evaluate/check-expression.h" 以使用Fortran 常量折叠与求值能力。
- **L15 EN**: Includes "flang/Evaluate/fold.h" to access Fortran constant-folding and evaluation facilities.
  **L15 CN**: 引入 "flang/Evaluate/fold.h" 以使用Fortran 常量折叠与求值能力。
- **L16 EN**: Includes "flang/Evaluate/tools.h" to access Fortran constant-folding and evaluation facilities.
  **L16 CN**: 引入 "flang/Evaluate/tools.h" 以使用Fortran 常量折叠与求值能力。
- **L17 EN**: Includes "flang/Parser/characters.h" to access parse-tree, token, or source representation support.
  **L17 CN**: 引入 "flang/Parser/characters.h" 以使用语法树、词法单元或源码表示支持。
- **L18 EN**: Includes "flang/Semantics/scope.h" to access Fortran semantic analysis, symbol, and type information.
  **L18 CN**: 引入 "flang/Semantics/scope.h" 以使用Fortran 语义分析、符号与类型信息。
- **L19 EN**: Includes "flang/Semantics/semantics.h" to access Fortran semantic analysis, symbol, and type information.
  **L19 CN**: 引入 "flang/Semantics/semantics.h" 以使用Fortran 语义分析、符号与类型信息。
- **L20 EN**: Includes "flang/Semantics/symbol.h" to access Fortran semantic analysis, symbol, and type information.
  **L20 CN**: 引入 "flang/Semantics/symbol.h" 以使用Fortran 语义分析、符号与类型信息。
- **L21 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L21 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L22 EN**: Includes "flang/Semantics/type.h" to access Fortran semantic analysis, symbol, and type information.
  **L22 CN**: 引入 "flang/Semantics/type.h" 以使用Fortran 语义分析、符号与类型信息。
- **L23 EN**: Includes <algorithm> to access supporting declarations used by this translation unit.
  **L23 CN**: 引入 <algorithm> 以使用当前编译单元使用的辅助声明。
- **L24 EN**: Includes <map> to access supporting declarations used by this translation unit.
  **L24 CN**: 引入 <map> 以使用当前编译单元使用的辅助声明。

### Lines 25-48

````cpp
#include <string>

namespace Fortran::semantics {

namespace characteristics = evaluate::characteristics;
using characteristics::DummyArgument;
using characteristics::DummyDataObject;
using characteristics::DummyProcedure;
using characteristics::FunctionResult;
using characteristics::Procedure;

class DistinguishabilityHelper;

class CheckHelper {
public:
  explicit CheckHelper(SemanticsContext &c) : context_{c} {}

  SemanticsContext &context() { return context_; }
  void Check() { Check(context_.globalScope()); }
  void Check(const ParamValue &, bool canBeAssumed);
  void Check(const Bound &bound) {
    CheckSpecExpr(bound.GetExplicit(), /*forElementalFunctionResult=*/false);
  }
  void Check(const ShapeSpec &spec) {
````
- **L25 EN**: Includes <string> to access supporting declarations used by this translation unit.
  **L25 CN**: 引入 <string> 以使用当前编译单元使用的辅助声明。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Opens namespace scope `Fortran::semantics`.
  **L27 CN**: 打开命名空间作用域 `Fortran::semantics`。
- **L28 EN**: Blank line separating nearby declarations or logic blocks.
  **L28 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L29 EN**: Initializes variable `characteristics` from the right-hand expression.
  **L29 CN**: 使用右侧表达式初始化变量 `characteristics`。
- **L30 EN**: Executes a standalone statement or declaration: `using characteristics::DummyArgument;`.
  **L30 CN**: 执行一条独立语句或声明：`using characteristics::DummyArgument;`。
- **L31 EN**: Executes a standalone statement or declaration: `using characteristics::DummyDataObject;`.
  **L31 CN**: 执行一条独立语句或声明：`using characteristics::DummyDataObject;`。
- **L32 EN**: Executes a standalone statement or declaration: `using characteristics::DummyProcedure;`.
  **L32 CN**: 执行一条独立语句或声明：`using characteristics::DummyProcedure;`。
- **L33 EN**: Executes a standalone statement or declaration: `using characteristics::FunctionResult;`.
  **L33 CN**: 执行一条独立语句或声明：`using characteristics::FunctionResult;`。
- **L34 EN**: Executes a standalone statement or declaration: `using characteristics::Procedure;`.
  **L34 CN**: 执行一条独立语句或声明：`using characteristics::Procedure;`。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L36 EN**: Declares class `DistinguishabilityHelper;`.
  **L36 CN**: 声明 class `DistinguishabilityHelper;`。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L38 EN**: Declares class `CheckHelper`.
  **L38 CN**: 声明 class `CheckHelper`。
- **L39 EN**: Sets the following members to `public` access.
  **L39 CN**: 将后续成员的访问级别设为 `public`。
- **L40 EN**: Continues logic associated with callable symbol `CheckHelper`.
  **L40 CN**: 继续与可调用符号 `CheckHelper` 相关的逻辑。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L42 EN**: Continues logic associated with callable symbol `context`.
  **L42 CN**: 继续与可调用符号 `context` 相关的逻辑。
- **L43 EN**: Continues logic associated with callable symbol `Check`.
  **L43 CN**: 继续与可调用符号 `Check` 相关的逻辑。
- **L44 EN**: Executes a call or declaration centered on `Check`.
  **L44 CN**: 执行以 `Check` 为核心的调用或声明。
- **L45 EN**: Starts a function, method, lambda, or structured scope: `void Check(const Bound &bound) {`.
  **L45 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const Bound &bound) {`。
- **L46 EN**: Executes a call or declaration centered on `CheckSpecExpr`.
  **L46 CN**: 执行以 `CheckSpecExpr` 为核心的调用或声明。
- **L47 EN**: Closes the current lexical scope or compound statement.
  **L47 CN**: 结束当前词法作用域或复合语句块。
- **L48 EN**: Starts a function, method, lambda, or structured scope: `void Check(const ShapeSpec &spec) {`.
  **L48 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const ShapeSpec &spec) {`。

### Lines 49-72

````cpp
    Check(spec.lbound());
    Check(spec.ubound());
  }
  void Check(const ArraySpec &);
  void Check(const DeclTypeSpec &, bool canHaveAssumedTypeParameters);
  void Check(const Symbol &);
  void CheckCommonBlock(const Symbol &);
  void Check(const Scope &);
  const Procedure *Characterize(const Symbol &);

private:
  template <typename A>
  void CheckSpecExpr(const A &x, bool forElementalFunctionResult) {
    evaluate::CheckSpecificationExpr(
        x, DEREF(scope_), foldingContext_, forElementalFunctionResult);
  }
  void CheckValue(const Symbol &, const DerivedTypeSpec *);
  void CheckVolatile(const Symbol &, const DerivedTypeSpec *);
  void CheckContiguous(const Symbol &);
  void CheckPointer(const Symbol &);
  void CheckPassArg(
      const Symbol &proc, const Symbol *interface, const WithPassArg &);
  void CheckProcBinding(const Symbol &, const ProcBindingDetails &);
  void CheckObjectEntity(const Symbol &, const ObjectEntityDetails &);
````
- **L49 EN**: Executes a call or declaration centered on `Check`.
  **L49 CN**: 执行以 `Check` 为核心的调用或声明。
- **L50 EN**: Executes a call or declaration centered on `Check`.
  **L50 CN**: 执行以 `Check` 为核心的调用或声明。
- **L51 EN**: Closes the current lexical scope or compound statement.
  **L51 CN**: 结束当前词法作用域或复合语句块。
- **L52 EN**: Executes a call or declaration centered on `Check`.
  **L52 CN**: 执行以 `Check` 为核心的调用或声明。
- **L53 EN**: Executes a call or declaration centered on `Check`.
  **L53 CN**: 执行以 `Check` 为核心的调用或声明。
- **L54 EN**: Executes a call or declaration centered on `Check`.
  **L54 CN**: 执行以 `Check` 为核心的调用或声明。
- **L55 EN**: Executes a call or declaration centered on `CheckCommonBlock`.
  **L55 CN**: 执行以 `CheckCommonBlock` 为核心的调用或声明。
- **L56 EN**: Executes a call or declaration centered on `Check`.
  **L56 CN**: 执行以 `Check` 为核心的调用或声明。
- **L57 EN**: Executes a call or declaration centered on `*Characterize`.
  **L57 CN**: 执行以 `*Characterize` 为核心的调用或声明。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L59 EN**: Sets the following members to `private` access.
  **L59 CN**: 将后续成员的访问级别设为 `private`。
- **L60 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L60 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。
- **L61 EN**: Starts a function, method, lambda, or structured scope: `void CheckSpecExpr(const A &x, bool forElementalFunctionResult) {`.
  **L61 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckSpecExpr(const A &x, bool forElementalFunctionResult) {`。
- **L62 EN**: Continues logic associated with callable symbol `CheckSpecificationExpr`.
  **L62 CN**: 继续与可调用符号 `CheckSpecificationExpr` 相关的逻辑。
- **L63 EN**: Executes a call or declaration centered on `DEREF`.
  **L63 CN**: 执行以 `DEREF` 为核心的调用或声明。
- **L64 EN**: Closes the current lexical scope or compound statement.
  **L64 CN**: 结束当前词法作用域或复合语句块。
- **L65 EN**: Executes a call or declaration centered on `CheckValue`.
  **L65 CN**: 执行以 `CheckValue` 为核心的调用或声明。
- **L66 EN**: Executes a call or declaration centered on `CheckVolatile`.
  **L66 CN**: 执行以 `CheckVolatile` 为核心的调用或声明。
- **L67 EN**: Executes a call or declaration centered on `CheckContiguous`.
  **L67 CN**: 执行以 `CheckContiguous` 为核心的调用或声明。
- **L68 EN**: Executes a call or declaration centered on `CheckPointer`.
  **L68 CN**: 执行以 `CheckPointer` 为核心的调用或声明。
- **L69 EN**: Continues logic associated with callable symbol `CheckPassArg`.
  **L69 CN**: 继续与可调用符号 `CheckPassArg` 相关的逻辑。
- **L70 EN**: Executes a standalone statement or declaration: `const Symbol &proc, const Symbol *interface, const WithPassArg &);`.
  **L70 CN**: 执行一条独立语句或声明：`const Symbol &proc, const Symbol *interface, const WithPassArg &);`。
- **L71 EN**: Executes a call or declaration centered on `CheckProcBinding`.
  **L71 CN**: 执行以 `CheckProcBinding` 为核心的调用或声明。
- **L72 EN**: Executes a call or declaration centered on `CheckObjectEntity`.
  **L72 CN**: 执行以 `CheckObjectEntity` 为核心的调用或声明。

### Lines 73-96

````cpp
  void CheckPointerInitialization(const Symbol &);
  void CheckArraySpec(const Symbol &, const ArraySpec &);
  void CheckProcEntity(const Symbol &, const ProcEntityDetails &);
  void CheckSubprogram(const Symbol &, const SubprogramDetails &);
  void CheckExternal(const Symbol &);
  void CheckAssumedTypeEntity(const Symbol &, const ObjectEntityDetails &);
  void CheckDerivedType(const Symbol &, const DerivedTypeDetails &);
  bool CheckFinal(
      const Symbol &subroutine, SourceName, const Symbol &derivedType);
  bool CheckDistinguishableFinals(const Symbol &f1, SourceName f1name,
      const Symbol &f2, SourceName f2name, const Symbol &derivedType);
  void CheckGeneric(const Symbol &, const GenericDetails &);
  void CheckHostAssoc(const Symbol &, const HostAssocDetails &);
  bool CheckDefinedOperator(
      SourceName, GenericKind, const Symbol &, const Procedure &);
  std::optional<parser::MessageFixedText> CheckNumberOfArgs(
      const GenericKind &, std::size_t);
  bool CheckDefinedOperatorArg(
      const SourceName &, const Symbol &, const Procedure &, std::size_t);
  bool CheckDefinedAssignment(const Symbol &, const Procedure &);
  bool CheckDefinedAssignmentArg(const Symbol &, const DummyArgument &, int);
  void CollectSpecifics(
      DistinguishabilityHelper &, const Symbol &, const GenericDetails &);
  void CheckSpecifics(const Symbol &, const GenericDetails &);
````
- **L73 EN**: Executes a call or declaration centered on `CheckPointerInitialization`.
  **L73 CN**: 执行以 `CheckPointerInitialization` 为核心的调用或声明。
- **L74 EN**: Executes a call or declaration centered on `CheckArraySpec`.
  **L74 CN**: 执行以 `CheckArraySpec` 为核心的调用或声明。
- **L75 EN**: Executes a call or declaration centered on `CheckProcEntity`.
  **L75 CN**: 执行以 `CheckProcEntity` 为核心的调用或声明。
- **L76 EN**: Executes a call or declaration centered on `CheckSubprogram`.
  **L76 CN**: 执行以 `CheckSubprogram` 为核心的调用或声明。
- **L77 EN**: Executes a call or declaration centered on `CheckExternal`.
  **L77 CN**: 执行以 `CheckExternal` 为核心的调用或声明。
- **L78 EN**: Executes a call or declaration centered on `CheckAssumedTypeEntity`.
  **L78 CN**: 执行以 `CheckAssumedTypeEntity` 为核心的调用或声明。
- **L79 EN**: Executes a call or declaration centered on `CheckDerivedType`.
  **L79 CN**: 执行以 `CheckDerivedType` 为核心的调用或声明。
- **L80 EN**: Continues logic associated with callable symbol `CheckFinal`.
  **L80 CN**: 继续与可调用符号 `CheckFinal` 相关的逻辑。
- **L81 EN**: Executes a standalone statement or declaration: `const Symbol &subroutine, SourceName, const Symbol &derivedType);`.
  **L81 CN**: 执行一条独立语句或声明：`const Symbol &subroutine, SourceName, const Symbol &derivedType);`。
- **L82 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool CheckDistinguishableFinals(const Symbol &f1, SourceName f1name,`.
  **L82 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool CheckDistinguishableFinals(const Symbol &f1, SourceName f1name,`。
- **L83 EN**: Executes a standalone statement or declaration: `const Symbol &f2, SourceName f2name, const Symbol &derivedType);`.
  **L83 CN**: 执行一条独立语句或声明：`const Symbol &f2, SourceName f2name, const Symbol &derivedType);`。
- **L84 EN**: Executes a call or declaration centered on `CheckGeneric`.
  **L84 CN**: 执行以 `CheckGeneric` 为核心的调用或声明。
- **L85 EN**: Executes a call or declaration centered on `CheckHostAssoc`.
  **L85 CN**: 执行以 `CheckHostAssoc` 为核心的调用或声明。
- **L86 EN**: Continues logic associated with callable symbol `CheckDefinedOperator`.
  **L86 CN**: 继续与可调用符号 `CheckDefinedOperator` 相关的逻辑。
- **L87 EN**: Executes a standalone statement or declaration: `SourceName, GenericKind, const Symbol &, const Procedure &);`.
  **L87 CN**: 执行一条独立语句或声明：`SourceName, GenericKind, const Symbol &, const Procedure &);`。
- **L88 EN**: Continues logic associated with callable symbol `CheckNumberOfArgs`.
  **L88 CN**: 继续与可调用符号 `CheckNumberOfArgs` 相关的逻辑。
- **L89 EN**: Executes a standalone statement or declaration: `const GenericKind &, std::size_t);`.
  **L89 CN**: 执行一条独立语句或声明：`const GenericKind &, std::size_t);`。
- **L90 EN**: Continues logic associated with callable symbol `CheckDefinedOperatorArg`.
  **L90 CN**: 继续与可调用符号 `CheckDefinedOperatorArg` 相关的逻辑。
- **L91 EN**: Executes a standalone statement or declaration: `const SourceName &, const Symbol &, const Procedure &, std::size_t);`.
  **L91 CN**: 执行一条独立语句或声明：`const SourceName &, const Symbol &, const Procedure &, std::size_t);`。
- **L92 EN**: Executes a call or declaration centered on `CheckDefinedAssignment`.
  **L92 CN**: 执行以 `CheckDefinedAssignment` 为核心的调用或声明。
- **L93 EN**: Executes a call or declaration centered on `CheckDefinedAssignmentArg`.
  **L93 CN**: 执行以 `CheckDefinedAssignmentArg` 为核心的调用或声明。
- **L94 EN**: Continues logic associated with callable symbol `CollectSpecifics`.
  **L94 CN**: 继续与可调用符号 `CollectSpecifics` 相关的逻辑。
- **L95 EN**: Executes a standalone statement or declaration: `DistinguishabilityHelper &, const Symbol &, const GenericDetails &);`.
  **L95 CN**: 执行一条独立语句或声明：`DistinguishabilityHelper &, const Symbol &, const GenericDetails &);`。
- **L96 EN**: Executes a call or declaration centered on `CheckSpecifics`.
  **L96 CN**: 执行以 `CheckSpecifics` 为核心的调用或声明。

### Lines 97-120

````cpp
  void CheckEquivalenceSet(const EquivalenceSet &);
  void CheckEquivalenceObject(const EquivalenceObject &);
  void CheckBlockData(const Scope &);
  void CheckGenericOps(const Scope &);
  bool CheckConflicting(const Symbol &, Attr, Attr);
  void WarnMissingFinal(const Symbol &);
  void CheckSymbolType(const Symbol &); // C702
  bool InPure() const {
    return innermostSymbol_ && IsPureProcedure(*innermostSymbol_);
  }
  bool InElemental() const {
    return innermostSymbol_ && IsElementalProcedure(*innermostSymbol_);
  }
  bool InFunction() const {
    return innermostSymbol_ && IsFunction(*innermostSymbol_);
  }
  bool InInterface() const {
    const SubprogramDetails *subp{innermostSymbol_
            ? innermostSymbol_->detailsIf<SubprogramDetails>()
            : nullptr};
    return subp && subp->isInterface();
  }
  template <typename... A>
  parser::Message *SayWithDeclaration(const Symbol &symbol, A &&...x) {
````
- **L97 EN**: Executes a call or declaration centered on `CheckEquivalenceSet`.
  **L97 CN**: 执行以 `CheckEquivalenceSet` 为核心的调用或声明。
- **L98 EN**: Executes a call or declaration centered on `CheckEquivalenceObject`.
  **L98 CN**: 执行以 `CheckEquivalenceObject` 为核心的调用或声明。
- **L99 EN**: Executes a call or declaration centered on `CheckBlockData`.
  **L99 CN**: 执行以 `CheckBlockData` 为核心的调用或声明。
- **L100 EN**: Executes a call or declaration centered on `CheckGenericOps`.
  **L100 CN**: 执行以 `CheckGenericOps` 为核心的调用或声明。
- **L101 EN**: Executes a call or declaration centered on `CheckConflicting`.
  **L101 CN**: 执行以 `CheckConflicting` 为核心的调用或声明。
- **L102 EN**: Executes a call or declaration centered on `WarnMissingFinal`.
  **L102 CN**: 执行以 `WarnMissingFinal` 为核心的调用或声明。
- **L103 EN**: Continues logic associated with callable symbol `CheckSymbolType`.
  **L103 CN**: 继续与可调用符号 `CheckSymbolType` 相关的逻辑。
- **L104 EN**: Starts a function, method, lambda, or structured scope: `bool InPure() const {`.
  **L104 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool InPure() const {`。
- **L105 EN**: Returns from the current function with `innermostSymbol_ && IsPureProcedure(*innermostSymbol_)`.
  **L105 CN**: 以 `innermostSymbol_ && IsPureProcedure(*innermostSymbol_)` 从当前函数返回。
- **L106 EN**: Closes the current lexical scope or compound statement.
  **L106 CN**: 结束当前词法作用域或复合语句块。
- **L107 EN**: Starts a function, method, lambda, or structured scope: `bool InElemental() const {`.
  **L107 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool InElemental() const {`。
- **L108 EN**: Returns from the current function with `innermostSymbol_ && IsElementalProcedure(*innermostSymbol_)`.
  **L108 CN**: 以 `innermostSymbol_ && IsElementalProcedure(*innermostSymbol_)` 从当前函数返回。
- **L109 EN**: Closes the current lexical scope or compound statement.
  **L109 CN**: 结束当前词法作用域或复合语句块。
- **L110 EN**: Starts a function, method, lambda, or structured scope: `bool InFunction() const {`.
  **L110 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool InFunction() const {`。
- **L111 EN**: Returns from the current function with `innermostSymbol_ && IsFunction(*innermostSymbol_)`.
  **L111 CN**: 以 `innermostSymbol_ && IsFunction(*innermostSymbol_)` 从当前函数返回。
- **L112 EN**: Closes the current lexical scope or compound statement.
  **L112 CN**: 结束当前词法作用域或复合语句块。
- **L113 EN**: Starts a function, method, lambda, or structured scope: `bool InInterface() const {`.
  **L113 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool InInterface() const {`。
- **L114 EN**: Continues the surrounding expression or declaration: `const SubprogramDetails *subp{innermostSymbol_`.
  **L114 CN**: 继续构造周围的表达式或声明：`const SubprogramDetails *subp{innermostSymbol_`。
- **L115 EN**: Continues logic associated with callable symbol `detailsIf<SubprogramDetails>`.
  **L115 CN**: 继续与可调用符号 `detailsIf<SubprogramDetails>` 相关的逻辑。
- **L116 EN**: Executes a standalone statement or declaration: `: nullptr};`.
  **L116 CN**: 执行一条独立语句或声明：`: nullptr};`。
- **L117 EN**: Returns from the current function with `subp && subp->isInterface()`.
  **L117 CN**: 以 `subp && subp->isInterface()` 从当前函数返回。
- **L118 EN**: Closes the current lexical scope or compound statement.
  **L118 CN**: 结束当前词法作用域或复合语句块。
- **L119 EN**: Introduces template parameters or specialization context: `template <typename... A>`.
  **L119 CN**: 为后续声明引入模板参数或特化上下文：`template <typename... A>`。
- **L120 EN**: Starts a function, method, lambda, or structured scope: `parser::Message *SayWithDeclaration(const Symbol &symbol, A &&...x) {`.
  **L120 CN**: 开始一个函数、方法、lambda 或结构化作用域：`parser::Message *SayWithDeclaration(const Symbol &symbol, A &&...x) {`。

### Lines 121-144

````cpp
    parser::Message *msg{messages_.Say(std::forward<A>(x)...)};
    if (msg && messages_.at().begin() != symbol.name().begin()) {
      evaluate::AttachDeclaration(*msg, symbol);
    }
    return msg;
  }
  bool InModuleFile() const {
    return FindModuleFileContaining(context_.FindScope(messages_.at())) !=
        nullptr;
  }
  template <typename FeatureOrUsageWarning, typename... A>
  parser::Message *Warn(FeatureOrUsageWarning warning, A &&...x) {
    return messages_.Warn(InModuleFile(), context_.languageFeatures(), warning,
        std::forward<A>(x)...);
  }
  template <typename FeatureOrUsageWarning, typename... A>
  parser::Message *Warn(
      FeatureOrUsageWarning warning, parser::CharBlock source, A &&...x) {
    return messages_.Warn(FindModuleFileContaining(context_.FindScope(source)),
        context_.languageFeatures(), warning, source, std::forward<A>(x)...);
  }
  bool IsResultOkToDiffer(const FunctionResult &);
  void CheckGlobalName(const Symbol &);
  void CheckProcedureAssemblyName(const Symbol &symbol);
````
- **L121 EN**: Executes a call or declaration centered on `*msg{messages_.Say`.
  **L121 CN**: 执行以 `*msg{messages_.Say` 为核心的调用或声明。
- **L122 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L122 CN**: 开始 `if` 控制流语句并计算其条件。
- **L123 EN**: Executes a call or declaration centered on `evaluate::AttachDeclaration`.
  **L123 CN**: 执行以 `evaluate::AttachDeclaration` 为核心的调用或声明。
- **L124 EN**: Closes the current lexical scope or compound statement.
  **L124 CN**: 结束当前词法作用域或复合语句块。
- **L125 EN**: Returns from the current function with `msg`.
  **L125 CN**: 以 `msg` 从当前函数返回。
- **L126 EN**: Closes the current lexical scope or compound statement.
  **L126 CN**: 结束当前词法作用域或复合语句块。
- **L127 EN**: Starts a function, method, lambda, or structured scope: `bool InModuleFile() const {`.
  **L127 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool InModuleFile() const {`。
- **L128 EN**: Returns from the current function with `FindModuleFileContaining(context_.FindScope(messages_.at())) !=`.
  **L128 CN**: 以 `FindModuleFileContaining(context_.FindScope(messages_.at())) !=` 从当前函数返回。
- **L129 EN**: Executes a standalone statement or declaration: `nullptr;`.
  **L129 CN**: 执行一条独立语句或声明：`nullptr;`。
- **L130 EN**: Closes the current lexical scope or compound statement.
  **L130 CN**: 结束当前词法作用域或复合语句块。
- **L131 EN**: Introduces template parameters or specialization context: `template <typename FeatureOrUsageWarning, typename... A>`.
  **L131 CN**: 为后续声明引入模板参数或特化上下文：`template <typename FeatureOrUsageWarning, typename... A>`。
- **L132 EN**: Starts a function, method, lambda, or structured scope: `parser::Message *Warn(FeatureOrUsageWarning warning, A &&...x) {`.
  **L132 CN**: 开始一个函数、方法、lambda 或结构化作用域：`parser::Message *Warn(FeatureOrUsageWarning warning, A &&...x) {`。
- **L133 EN**: Returns from the current function with `messages_.Warn(InModuleFile(), context_.languageFeatures(), warning,`.
  **L133 CN**: 以 `messages_.Warn(InModuleFile(), context_.languageFeatures(), warning,` 从当前函数返回。
- **L134 EN**: Executes a call or declaration centered on `std::forward<A>`.
  **L134 CN**: 执行以 `std::forward<A>` 为核心的调用或声明。
- **L135 EN**: Closes the current lexical scope or compound statement.
  **L135 CN**: 结束当前词法作用域或复合语句块。
- **L136 EN**: Introduces template parameters or specialization context: `template <typename FeatureOrUsageWarning, typename... A>`.
  **L136 CN**: 为后续声明引入模板参数或特化上下文：`template <typename FeatureOrUsageWarning, typename... A>`。
- **L137 EN**: Continues logic associated with callable symbol `Warn`.
  **L137 CN**: 继续与可调用符号 `Warn` 相关的逻辑。
- **L138 EN**: Continues the surrounding expression or declaration: `FeatureOrUsageWarning warning, parser::CharBlock source, A &&...x) {`.
  **L138 CN**: 继续构造周围的表达式或声明：`FeatureOrUsageWarning warning, parser::CharBlock source, A &&...x) {`。
- **L139 EN**: Returns from the current function with `messages_.Warn(FindModuleFileContaining(context_.FindScope(source)),`.
  **L139 CN**: 以 `messages_.Warn(FindModuleFileContaining(context_.FindScope(source)),` 从当前函数返回。
- **L140 EN**: Executes a call or declaration centered on `context_.languageFeatures`.
  **L140 CN**: 执行以 `context_.languageFeatures` 为核心的调用或声明。
- **L141 EN**: Closes the current lexical scope or compound statement.
  **L141 CN**: 结束当前词法作用域或复合语句块。
- **L142 EN**: Executes a call or declaration centered on `IsResultOkToDiffer`.
  **L142 CN**: 执行以 `IsResultOkToDiffer` 为核心的调用或声明。
- **L143 EN**: Executes a call or declaration centered on `CheckGlobalName`.
  **L143 CN**: 执行以 `CheckGlobalName` 为核心的调用或声明。
- **L144 EN**: Executes a call or declaration centered on `CheckProcedureAssemblyName`.
  **L144 CN**: 执行以 `CheckProcedureAssemblyName` 为核心的调用或声明。

### Lines 145-168

````cpp
  void CheckExplicitSave(const Symbol &);
  parser::Messages WhyNotInteroperableDerivedType(const Symbol &);
  parser::Messages WhyNotInteroperableObject(const Symbol &,
      bool allowNonInteroperableType = false, bool forCommonBlock = false);
  parser::Messages WhyNotInteroperableFunctionResult(const Symbol &);
  parser::Messages WhyNotInteroperableProcedure(const Symbol &, bool isError);
  void CheckBindC(const Symbol &);
  // Check functions for defined I/O procedures
  void CheckDefinedIoProc(
      const Symbol &, const GenericDetails &, common::DefinedIo);
  bool CheckDioDummyIsData(const Symbol &, const Symbol *, std::size_t);
  void CheckDioDummyIsDerived(
      const Symbol &, const Symbol &, common::DefinedIo ioKind, const Symbol &);
  void CheckDioDummyIsDefaultInteger(const Symbol &, const Symbol &);
  void CheckDioDummyIsScalar(const Symbol &, const Symbol &);
  void CheckDioDummyAttrs(const Symbol &, const Symbol &, Attr);
  void CheckDioDtvArg(const Symbol &proc, const Symbol &subp, const Symbol *arg,
      common::DefinedIo, const Symbol &generic);
  void CheckGenericVsIntrinsic(const Symbol &, const GenericDetails &);
  void CheckDefaultIntegerArg(const Symbol &, const Symbol *, Attr);
  void CheckDioAssumedLenCharacterArg(
      const Symbol &, const Symbol *, std::size_t, Attr);
  void CheckDioVlistArg(const Symbol &, const Symbol *, std::size_t);
  void CheckDioArgCount(const Symbol &, common::DefinedIo ioKind, std::size_t);
````
- **L145 EN**: Executes a call or declaration centered on `CheckExplicitSave`.
  **L145 CN**: 执行以 `CheckExplicitSave` 为核心的调用或声明。
- **L146 EN**: Executes a call or declaration centered on `WhyNotInteroperableDerivedType`.
  **L146 CN**: 执行以 `WhyNotInteroperableDerivedType` 为核心的调用或声明。
- **L147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::Messages WhyNotInteroperableObject(const Symbol &,`.
  **L147 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::Messages WhyNotInteroperableObject(const Symbol &,`。
- **L148 EN**: Initializes variable `allowNonInteroperableType` from the right-hand expression.
  **L148 CN**: 使用右侧表达式初始化变量 `allowNonInteroperableType`。
- **L149 EN**: Executes a call or declaration centered on `WhyNotInteroperableFunctionResult`.
  **L149 CN**: 执行以 `WhyNotInteroperableFunctionResult` 为核心的调用或声明。
- **L150 EN**: Executes a call or declaration centered on `WhyNotInteroperableProcedure`.
  **L150 CN**: 执行以 `WhyNotInteroperableProcedure` 为核心的调用或声明。
- **L151 EN**: Executes a call or declaration centered on `CheckBindC`.
  **L151 CN**: 执行以 `CheckBindC` 为核心的调用或声明。
- **L152 EN**: Comment explains nearby logic, intent, or metadata: `Check functions for defined I/O procedures`.
  **L152 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check functions for defined I/O procedures`。
- **L153 EN**: Continues logic associated with callable symbol `CheckDefinedIoProc`.
  **L153 CN**: 继续与可调用符号 `CheckDefinedIoProc` 相关的逻辑。
- **L154 EN**: Executes a standalone statement or declaration: `const Symbol &, const GenericDetails &, common::DefinedIo);`.
  **L154 CN**: 执行一条独立语句或声明：`const Symbol &, const GenericDetails &, common::DefinedIo);`。
- **L155 EN**: Executes a call or declaration centered on `CheckDioDummyIsData`.
  **L155 CN**: 执行以 `CheckDioDummyIsData` 为核心的调用或声明。
- **L156 EN**: Continues logic associated with callable symbol `CheckDioDummyIsDerived`.
  **L156 CN**: 继续与可调用符号 `CheckDioDummyIsDerived` 相关的逻辑。
- **L157 EN**: Executes a standalone statement or declaration: `const Symbol &, const Symbol &, common::DefinedIo ioKind, const Symbol &);`.
  **L157 CN**: 执行一条独立语句或声明：`const Symbol &, const Symbol &, common::DefinedIo ioKind, const Symbol &);`。
- **L158 EN**: Executes a call or declaration centered on `CheckDioDummyIsDefaultInteger`.
  **L158 CN**: 执行以 `CheckDioDummyIsDefaultInteger` 为核心的调用或声明。
- **L159 EN**: Executes a call or declaration centered on `CheckDioDummyIsScalar`.
  **L159 CN**: 执行以 `CheckDioDummyIsScalar` 为核心的调用或声明。
- **L160 EN**: Executes a call or declaration centered on `CheckDioDummyAttrs`.
  **L160 CN**: 执行以 `CheckDioDummyAttrs` 为核心的调用或声明。
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckDioDtvArg(const Symbol &proc, const Symbol &subp, const Symbol *arg,`.
  **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckDioDtvArg(const Symbol &proc, const Symbol &subp, const Symbol *arg,`。
- **L162 EN**: Executes a standalone statement or declaration: `common::DefinedIo, const Symbol &generic);`.
  **L162 CN**: 执行一条独立语句或声明：`common::DefinedIo, const Symbol &generic);`。
- **L163 EN**: Executes a call or declaration centered on `CheckGenericVsIntrinsic`.
  **L163 CN**: 执行以 `CheckGenericVsIntrinsic` 为核心的调用或声明。
- **L164 EN**: Executes a call or declaration centered on `CheckDefaultIntegerArg`.
  **L164 CN**: 执行以 `CheckDefaultIntegerArg` 为核心的调用或声明。
- **L165 EN**: Continues logic associated with callable symbol `CheckDioAssumedLenCharacterArg`.
  **L165 CN**: 继续与可调用符号 `CheckDioAssumedLenCharacterArg` 相关的逻辑。
- **L166 EN**: Executes a standalone statement or declaration: `const Symbol &, const Symbol *, std::size_t, Attr);`.
  **L166 CN**: 执行一条独立语句或声明：`const Symbol &, const Symbol *, std::size_t, Attr);`。
- **L167 EN**: Executes a call or declaration centered on `CheckDioVlistArg`.
  **L167 CN**: 执行以 `CheckDioVlistArg` 为核心的调用或声明。
- **L168 EN**: Executes a call or declaration centered on `CheckDioArgCount`.
  **L168 CN**: 执行以 `CheckDioArgCount` 为核心的调用或声明。

### Lines 169-192

````cpp
  struct TypeWithDefinedIo {
    const DerivedTypeSpec &type;
    common::DefinedIo ioKind;
    const Symbol &proc;
    const Symbol &generic;
  };
  void CheckAlreadySeenDefinedIo(const DerivedTypeSpec &, common::DefinedIo,
      const Symbol &, const Symbol &generic);
  void CheckModuleProcedureDef(const Symbol &);

  SemanticsContext &context_;
  evaluate::FoldingContext &foldingContext_{context_.foldingContext()};
  parser::ContextualMessages &messages_{foldingContext_.messages()};
  const Scope *scope_{nullptr};
  bool scopeIsUninstantiatedPDT_{false};
  // This symbol is the one attached to the innermost enclosing scope
  // that has a symbol.
  const Symbol *innermostSymbol_{nullptr};
  // Cache of calls to Procedure::Characterize(Symbol)
  std::map<SymbolRef, std::optional<Procedure>, SymbolAddressCompare>
      characterizeCache_;
  // Collection of module procedure symbols with non-BIND(C)
  // global names, qualified by their module.
  std::map<std::pair<SourceName, const Symbol *>, SymbolRef> moduleProcs_;
````
- **L169 EN**: Declares struct `TypeWithDefinedIo`.
  **L169 CN**: 声明 struct `TypeWithDefinedIo`。
- **L170 EN**: Executes a standalone statement or declaration: `const DerivedTypeSpec &type;`.
  **L170 CN**: 执行一条独立语句或声明：`const DerivedTypeSpec &type;`。
- **L171 EN**: Executes a standalone statement or declaration: `common::DefinedIo ioKind;`.
  **L171 CN**: 执行一条独立语句或声明：`common::DefinedIo ioKind;`。
- **L172 EN**: Executes a standalone statement or declaration: `const Symbol &proc;`.
  **L172 CN**: 执行一条独立语句或声明：`const Symbol &proc;`。
- **L173 EN**: Executes a standalone statement or declaration: `const Symbol &generic;`.
  **L173 CN**: 执行一条独立语句或声明：`const Symbol &generic;`。
- **L174 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L174 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckAlreadySeenDefinedIo(const DerivedTypeSpec &, common::DefinedIo,`.
  **L175 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckAlreadySeenDefinedIo(const DerivedTypeSpec &, common::DefinedIo,`。
- **L176 EN**: Executes a standalone statement or declaration: `const Symbol &, const Symbol &generic);`.
  **L176 CN**: 执行一条独立语句或声明：`const Symbol &, const Symbol &generic);`。
- **L177 EN**: Executes a call or declaration centered on `CheckModuleProcedureDef`.
  **L177 CN**: 执行以 `CheckModuleProcedureDef` 为核心的调用或声明。
- **L178 EN**: Blank line separating nearby declarations or logic blocks.
  **L178 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L179 EN**: Executes a standalone statement or declaration: `SemanticsContext &context_;`.
  **L179 CN**: 执行一条独立语句或声明：`SemanticsContext &context_;`。
- **L180 EN**: Executes a call or declaration centered on `&foldingContext_{context_.foldingContext`.
  **L180 CN**: 执行以 `&foldingContext_{context_.foldingContext` 为核心的调用或声明。
- **L181 EN**: Executes a call or declaration centered on `&messages_{foldingContext_.messages`.
  **L181 CN**: 执行以 `&messages_{foldingContext_.messages` 为核心的调用或声明。
- **L182 EN**: Executes a standalone statement or declaration: `const Scope *scope_{nullptr};`.
  **L182 CN**: 执行一条独立语句或声明：`const Scope *scope_{nullptr};`。
- **L183 EN**: Executes a standalone statement or declaration: `bool scopeIsUninstantiatedPDT_{false};`.
  **L183 CN**: 执行一条独立语句或声明：`bool scopeIsUninstantiatedPDT_{false};`。
- **L184 EN**: Comment explains nearby logic, intent, or metadata: `This symbol is the one attached to the innermost enclosing scope`.
  **L184 CN**: 注释说明附近代码的逻辑、意图或元数据：`This symbol is the one attached to the innermost enclosing scope`。
- **L185 EN**: Comment explains nearby logic, intent, or metadata: `that has a symbol.`.
  **L185 CN**: 注释说明附近代码的逻辑、意图或元数据：`that has a symbol.`。
- **L186 EN**: Executes a standalone statement or declaration: `const Symbol *innermostSymbol_{nullptr};`.
  **L186 CN**: 执行一条独立语句或声明：`const Symbol *innermostSymbol_{nullptr};`。
- **L187 EN**: Comment explains nearby logic, intent, or metadata: `Cache of calls to Procedure::Characterize(Symbol)`.
  **L187 CN**: 注释说明附近代码的逻辑、意图或元数据：`Cache of calls to Procedure::Characterize(Symbol)`。
- **L188 EN**: Continues the surrounding expression or declaration: `std::map<SymbolRef, std::optional<Procedure>, SymbolAddressCompare>`.
  **L188 CN**: 继续构造周围的表达式或声明：`std::map<SymbolRef, std::optional<Procedure>, SymbolAddressCompare>`。
- **L189 EN**: Executes a standalone statement or declaration: `characterizeCache_;`.
  **L189 CN**: 执行一条独立语句或声明：`characterizeCache_;`。
- **L190 EN**: Comment explains nearby logic, intent, or metadata: `Collection of module procedure symbols with non-BIND(C)`.
  **L190 CN**: 注释说明附近代码的逻辑、意图或元数据：`Collection of module procedure symbols with non-BIND(C)`。
- **L191 EN**: Comment explains nearby logic, intent, or metadata: `global names, qualified by their module.`.
  **L191 CN**: 注释说明附近代码的逻辑、意图或元数据：`global names, qualified by their module.`。
- **L192 EN**: Executes a standalone statement or declaration: `std::map<std::pair<SourceName, const Symbol *>, SymbolRef> moduleProcs_;`.
  **L192 CN**: 执行一条独立语句或声明：`std::map<std::pair<SourceName, const Symbol *>, SymbolRef> moduleProcs_;`。

### Lines 193-216

````cpp
  // Collection of symbols with global names, BIND(C) or otherwise
  std::map<std::string, SymbolRef> globalNames_;
  // Collection of external procedures without global definitions
  std::map<std::string, SymbolRef> externalNames_;
  // Collection of target dependent assembly names of external and BIND(C)
  // procedures.
  std::map<std::string, SymbolRef> procedureAssemblyNames_;
  // Derived types that have been examined by WhyNotInteroperable_XXX
  UnorderedSymbolSet examinedByWhyNotInteroperable_;
};

class DistinguishabilityHelper {
public:
  DistinguishabilityHelper(SemanticsContext &context) : context_{context} {}
  void Add(const Symbol &, GenericKind, const Symbol &, const Procedure &);
  void Check(const Scope &);

private:
  void SayNotDistinguishable(const Scope &, const SourceName &, GenericKind,
      const Symbol &, const Symbol &, bool isHardConflict);
  void AttachDeclaration(parser::Message &, const Scope &, const Symbol &);

  SemanticsContext &context_;
  struct ProcedureInfo {
````
- **L193 EN**: Comment explains nearby logic, intent, or metadata: `Collection of symbols with global names, BIND(C) or otherwise`.
  **L193 CN**: 注释说明附近代码的逻辑、意图或元数据：`Collection of symbols with global names, BIND(C) or otherwise`。
- **L194 EN**: Executes a standalone statement or declaration: `std::map<std::string, SymbolRef> globalNames_;`.
  **L194 CN**: 执行一条独立语句或声明：`std::map<std::string, SymbolRef> globalNames_;`。
- **L195 EN**: Comment explains nearby logic, intent, or metadata: `Collection of external procedures without global definitions`.
  **L195 CN**: 注释说明附近代码的逻辑、意图或元数据：`Collection of external procedures without global definitions`。
- **L196 EN**: Executes a standalone statement or declaration: `std::map<std::string, SymbolRef> externalNames_;`.
  **L196 CN**: 执行一条独立语句或声明：`std::map<std::string, SymbolRef> externalNames_;`。
- **L197 EN**: Comment explains nearby logic, intent, or metadata: `Collection of target dependent assembly names of external and BIND(C)`.
  **L197 CN**: 注释说明附近代码的逻辑、意图或元数据：`Collection of target dependent assembly names of external and BIND(C)`。
- **L198 EN**: Comment explains nearby logic, intent, or metadata: `procedures.`.
  **L198 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedures.`。
- **L199 EN**: Executes a standalone statement or declaration: `std::map<std::string, SymbolRef> procedureAssemblyNames_;`.
  **L199 CN**: 执行一条独立语句或声明：`std::map<std::string, SymbolRef> procedureAssemblyNames_;`。
- **L200 EN**: Comment explains nearby logic, intent, or metadata: `Derived types that have been examined by WhyNotInteroperable_XXX`.
  **L200 CN**: 注释说明附近代码的逻辑、意图或元数据：`Derived types that have been examined by WhyNotInteroperable_XXX`。
- **L201 EN**: Executes a standalone statement or declaration: `UnorderedSymbolSet examinedByWhyNotInteroperable_;`.
  **L201 CN**: 执行一条独立语句或声明：`UnorderedSymbolSet examinedByWhyNotInteroperable_;`。
- **L202 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L202 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L203 EN**: Blank line separating nearby declarations or logic blocks.
  **L203 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L204 EN**: Declares class `DistinguishabilityHelper`.
  **L204 CN**: 声明 class `DistinguishabilityHelper`。
- **L205 EN**: Sets the following members to `public` access.
  **L205 CN**: 将后续成员的访问级别设为 `public`。
- **L206 EN**: Continues logic associated with callable symbol `DistinguishabilityHelper`.
  **L206 CN**: 继续与可调用符号 `DistinguishabilityHelper` 相关的逻辑。
- **L207 EN**: Executes a call or declaration centered on `Add`.
  **L207 CN**: 执行以 `Add` 为核心的调用或声明。
- **L208 EN**: Executes a call or declaration centered on `Check`.
  **L208 CN**: 执行以 `Check` 为核心的调用或声明。
- **L209 EN**: Blank line separating nearby declarations or logic blocks.
  **L209 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L210 EN**: Sets the following members to `private` access.
  **L210 CN**: 将后续成员的访问级别设为 `private`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void SayNotDistinguishable(const Scope &, const SourceName &, GenericKind,`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`void SayNotDistinguishable(const Scope &, const SourceName &, GenericKind,`。
- **L212 EN**: Executes a standalone statement or declaration: `const Symbol &, const Symbol &, bool isHardConflict);`.
  **L212 CN**: 执行一条独立语句或声明：`const Symbol &, const Symbol &, bool isHardConflict);`。
- **L213 EN**: Executes a call or declaration centered on `AttachDeclaration`.
  **L213 CN**: 执行以 `AttachDeclaration` 为核心的调用或声明。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L215 EN**: Executes a standalone statement or declaration: `SemanticsContext &context_;`.
  **L215 CN**: 执行一条独立语句或声明：`SemanticsContext &context_;`。
- **L216 EN**: Declares struct `ProcedureInfo`.
  **L216 CN**: 声明 struct `ProcedureInfo`。

### Lines 217-240

````cpp
    GenericKind kind;
    const Procedure &procedure;
  };
  std::map<SourceName, std::map<const Symbol *, ProcedureInfo>>
      nameToSpecifics_;
};

void CheckHelper::Check(const ParamValue &value, bool canBeAssumed) {
  if (value.isAssumed()) {
    if (!canBeAssumed) { // C795, C721, C726
      messages_.Say(
          "An assumed (*) type parameter may be used only for a (non-statement function) dummy argument, associate name, character named constant, or external function result"_err_en_US);
    }
  } else {
    CheckSpecExpr(value.GetExplicit(), /*forElementalFunctionResult=*/false);
  }
}

void CheckHelper::Check(const ArraySpec &shape) {
  for (const auto &spec : shape) {
    Check(spec);
  }
}

````
- **L217 EN**: Executes a standalone statement or declaration: `GenericKind kind;`.
  **L217 CN**: 执行一条独立语句或声明：`GenericKind kind;`。
- **L218 EN**: Executes a standalone statement or declaration: `const Procedure &procedure;`.
  **L218 CN**: 执行一条独立语句或声明：`const Procedure &procedure;`。
- **L219 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L219 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L220 EN**: Continues the surrounding expression or declaration: `std::map<SourceName, std::map<const Symbol *, ProcedureInfo>>`.
  **L220 CN**: 继续构造周围的表达式或声明：`std::map<SourceName, std::map<const Symbol *, ProcedureInfo>>`。
- **L221 EN**: Executes a standalone statement or declaration: `nameToSpecifics_;`.
  **L221 CN**: 执行一条独立语句或声明：`nameToSpecifics_;`。
- **L222 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L222 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L224 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::Check(const ParamValue &value, bool canBeAssumed) {`.
  **L224 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::Check(const ParamValue &value, bool canBeAssumed) {`。
- **L225 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L225 CN**: 开始 `if` 控制流语句并计算其条件。
- **L226 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L226 CN**: 开始 `if` 控制流语句并计算其条件。
- **L227 EN**: Continues logic associated with callable symbol `Say`.
  **L227 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L228 EN**: Executes a call or declaration centered on `assumed`.
  **L228 CN**: 执行以 `assumed` 为核心的调用或声明。
- **L229 EN**: Closes the current lexical scope or compound statement.
  **L229 CN**: 结束当前词法作用域或复合语句块。
- **L230 EN**: Transitions from the previous branch into the alternative path.
  **L230 CN**: 从前一个分支过渡到备选路径。
- **L231 EN**: Executes a call or declaration centered on `CheckSpecExpr`.
  **L231 CN**: 执行以 `CheckSpecExpr` 为核心的调用或声明。
- **L232 EN**: Closes the current lexical scope or compound statement.
  **L232 CN**: 结束当前词法作用域或复合语句块。
- **L233 EN**: Closes the current lexical scope or compound statement.
  **L233 CN**: 结束当前词法作用域或复合语句块。
- **L234 EN**: Blank line separating nearby declarations or logic blocks.
  **L234 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L235 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::Check(const ArraySpec &shape) {`.
  **L235 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::Check(const ArraySpec &shape) {`。
- **L236 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L236 CN**: 开始 `for` 控制流语句并计算其条件。
- **L237 EN**: Executes a call or declaration centered on `Check`.
  **L237 CN**: 执行以 `Check` 为核心的调用或声明。
- **L238 EN**: Closes the current lexical scope or compound statement.
  **L238 CN**: 结束当前词法作用域或复合语句块。
- **L239 EN**: Closes the current lexical scope or compound statement.
  **L239 CN**: 结束当前词法作用域或复合语句块。
- **L240 EN**: Blank line separating nearby declarations or logic blocks.
  **L240 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 241-264

````cpp
void CheckHelper::Check(
    const DeclTypeSpec &type, bool canHaveAssumedTypeParameters) {
  if (type.category() == DeclTypeSpec::Character) {
    Check(type.characterTypeSpec().length(), canHaveAssumedTypeParameters);
  } else if (const DerivedTypeSpec *derived{type.AsDerived()}) {
    for (auto &parm : derived->parameters()) {
      Check(parm.second, canHaveAssumedTypeParameters);
    }
  }
}

static bool IsBlockData(const Scope &scope) {
  return scope.kind() == Scope::Kind::BlockData;
}

static bool IsBlockData(const Symbol &symbol) {
  return symbol.scope() && IsBlockData(*symbol.scope());
}

void CheckHelper::Check(const Symbol &symbol) {
  if (symbol.has<UseErrorDetails>()) {
    return;
  }
  if (symbol.name().size() > common::maxNameLen &&
````
- **L241 EN**: Continues logic associated with callable symbol `Check`.
  **L241 CN**: 继续与可调用符号 `Check` 相关的逻辑。
- **L242 EN**: Continues the surrounding expression or declaration: `const DeclTypeSpec &type, bool canHaveAssumedTypeParameters) {`.
  **L242 CN**: 继续构造周围的表达式或声明：`const DeclTypeSpec &type, bool canHaveAssumedTypeParameters) {`。
- **L243 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L243 CN**: 开始 `if` 控制流语句并计算其条件。
- **L244 EN**: Executes a call or declaration centered on `Check`.
  **L244 CN**: 执行以 `Check` 为核心的调用或声明。
- **L245 EN**: Transitions from the previous branch into an `else if` condition.
  **L245 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L246 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L246 CN**: 开始 `for` 控制流语句并计算其条件。
- **L247 EN**: Executes a call or declaration centered on `Check`.
  **L247 CN**: 执行以 `Check` 为核心的调用或声明。
- **L248 EN**: Closes the current lexical scope or compound statement.
  **L248 CN**: 结束当前词法作用域或复合语句块。
- **L249 EN**: Closes the current lexical scope or compound statement.
  **L249 CN**: 结束当前词法作用域或复合语句块。
- **L250 EN**: Closes the current lexical scope or compound statement.
  **L250 CN**: 结束当前词法作用域或复合语句块。
- **L251 EN**: Blank line separating nearby declarations or logic blocks.
  **L251 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L252 EN**: Starts a function, method, lambda, or structured scope: `static bool IsBlockData(const Scope &scope) {`.
  **L252 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool IsBlockData(const Scope &scope) {`。
- **L253 EN**: Returns from the current function with `scope.kind() == Scope::Kind::BlockData`.
  **L253 CN**: 以 `scope.kind() == Scope::Kind::BlockData` 从当前函数返回。
- **L254 EN**: Closes the current lexical scope or compound statement.
  **L254 CN**: 结束当前词法作用域或复合语句块。
- **L255 EN**: Blank line separating nearby declarations or logic blocks.
  **L255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L256 EN**: Starts a function, method, lambda, or structured scope: `static bool IsBlockData(const Symbol &symbol) {`.
  **L256 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool IsBlockData(const Symbol &symbol) {`。
- **L257 EN**: Returns from the current function with `symbol.scope() && IsBlockData(*symbol.scope())`.
  **L257 CN**: 以 `symbol.scope() && IsBlockData(*symbol.scope())` 从当前函数返回。
- **L258 EN**: Closes the current lexical scope or compound statement.
  **L258 CN**: 结束当前词法作用域或复合语句块。
- **L259 EN**: Blank line separating nearby declarations or logic blocks.
  **L259 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L260 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::Check(const Symbol &symbol) {`.
  **L260 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::Check(const Symbol &symbol) {`。
- **L261 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L261 CN**: 开始 `if` 控制流语句并计算其条件。
- **L262 EN**: Returns from the current function with `void`.
  **L262 CN**: 以 `void` 从当前函数返回。
- **L263 EN**: Closes the current lexical scope or compound statement.
  **L263 CN**: 结束当前词法作用域或复合语句块。
- **L264 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L264 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 265-288

````cpp
      &symbol == &symbol.GetUltimate()) {
    Warn(common::LanguageFeature::LongNames, symbol.name(),
        "%s has length %d, which is greater than the maximum name length %d"_port_en_US,
        symbol.name(), symbol.name().size(), common::maxNameLen);
  }
  if (context_.HasError(symbol)) {
    return;
  }
  auto restorer{messages_.SetLocation(symbol.name())};
  context_.set_location(symbol.name());
  const DeclTypeSpec *type{symbol.GetType()};
  const DerivedTypeSpec *derived{type ? type->AsDerived() : nullptr};
  bool isDone{false};
  common::visit(
      common::visitors{
          [&](const UseDetails &x) { isDone = true; },
          [&](const HostAssocDetails &x) {
            CheckHostAssoc(symbol, x);
            isDone = true;
          },
          [&](const ProcBindingDetails &x) {
            CheckProcBinding(symbol, x);
            isDone = true;
          },
````
- **L265 EN**: Starts a function, method, lambda, or structured scope: `&symbol == &symbol.GetUltimate()) {`.
  **L265 CN**: 开始一个函数、方法、lambda 或结构化作用域：`&symbol == &symbol.GetUltimate()) {`。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::LanguageFeature::LongNames, symbol.name(),`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::LanguageFeature::LongNames, symbol.name(),`。
- **L267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s has length %d, which is greater than the maximum name length %d"_port_en_US,`.
  **L267 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s has length %d, which is greater than the maximum name length %d"_port_en_US,`。
- **L268 EN**: Executes a call or declaration centered on `symbol.name`.
  **L268 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L269 EN**: Closes the current lexical scope or compound statement.
  **L269 CN**: 结束当前词法作用域或复合语句块。
- **L270 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L270 CN**: 开始 `if` 控制流语句并计算其条件。
- **L271 EN**: Returns from the current function with `void`.
  **L271 CN**: 以 `void` 从当前函数返回。
- **L272 EN**: Closes the current lexical scope or compound statement.
  **L272 CN**: 结束当前词法作用域或复合语句块。
- **L273 EN**: Executes a call or declaration centered on `restorer{messages_.SetLocation`.
  **L273 CN**: 执行以 `restorer{messages_.SetLocation` 为核心的调用或声明。
- **L274 EN**: Executes a call or declaration centered on `context_.set_location`.
  **L274 CN**: 执行以 `context_.set_location` 为核心的调用或声明。
- **L275 EN**: Executes a call or declaration centered on `*type{symbol.GetType`.
  **L275 CN**: 执行以 `*type{symbol.GetType` 为核心的调用或声明。
- **L276 EN**: Executes a call or declaration centered on `type->AsDerived`.
  **L276 CN**: 执行以 `type->AsDerived` 为核心的调用或声明。
- **L277 EN**: Executes a standalone statement or declaration: `bool isDone{false};`.
  **L277 CN**: 执行一条独立语句或声明：`bool isDone{false};`。
- **L278 EN**: Continues logic associated with callable symbol `visit`.
  **L278 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L279 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L279 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](const UseDetails &x) { isDone = true; },`.
  **L280 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](const UseDetails &x) { isDone = true; },`。
- **L281 EN**: Starts a function, method, lambda, or structured scope: `[&](const HostAssocDetails &x) {`.
  **L281 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const HostAssocDetails &x) {`。
- **L282 EN**: Executes a call or declaration centered on `CheckHostAssoc`.
  **L282 CN**: 执行以 `CheckHostAssoc` 为核心的调用或声明。
- **L283 EN**: Executes a standalone statement or declaration: `isDone = true;`.
  **L283 CN**: 执行一条独立语句或声明：`isDone = true;`。
- **L284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L284 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L285 EN**: Starts a function, method, lambda, or structured scope: `[&](const ProcBindingDetails &x) {`.
  **L285 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const ProcBindingDetails &x) {`。
- **L286 EN**: Executes a call or declaration centered on `CheckProcBinding`.
  **L286 CN**: 执行以 `CheckProcBinding` 为核心的调用或声明。
- **L287 EN**: Executes a standalone statement or declaration: `isDone = true;`.
  **L287 CN**: 执行一条独立语句或声明：`isDone = true;`。
- **L288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L288 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。

### Lines 289-312

````cpp
          [&](const ObjectEntityDetails &x) { CheckObjectEntity(symbol, x); },
          [&](const ProcEntityDetails &x) { CheckProcEntity(symbol, x); },
          [&](const SubprogramDetails &x) { CheckSubprogram(symbol, x); },
          [&](const DerivedTypeDetails &x) { CheckDerivedType(symbol, x); },
          [&](const GenericDetails &x) { CheckGeneric(symbol, x); },
          [](const auto &) {},
      },
      symbol.details());
  if (symbol.attrs().test(Attr::VOLATILE)) {
    CheckVolatile(symbol, derived);
  }
  if (symbol.attrs().test(Attr::BIND_C)) {
    CheckBindC(symbol);
  }
  if (symbol.attrs().test(Attr::SAVE) &&
      !symbol.implicitAttrs().test(Attr::SAVE)) {
    CheckExplicitSave(symbol);
  }
  if (symbol.attrs().test(Attr::CONTIGUOUS)) {
    CheckContiguous(symbol);
  }
  CheckGlobalName(symbol);
  CheckProcedureAssemblyName(symbol);
  if (symbol.attrs().test(Attr::ASYNCHRONOUS) &&
````
- **L289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](const ObjectEntityDetails &x) { CheckObjectEntity(symbol, x); },`.
  **L289 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](const ObjectEntityDetails &x) { CheckObjectEntity(symbol, x); },`。
- **L290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](const ProcEntityDetails &x) { CheckProcEntity(symbol, x); },`.
  **L290 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](const ProcEntityDetails &x) { CheckProcEntity(symbol, x); },`。
- **L291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](const SubprogramDetails &x) { CheckSubprogram(symbol, x); },`.
  **L291 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](const SubprogramDetails &x) { CheckSubprogram(symbol, x); },`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](const DerivedTypeDetails &x) { CheckDerivedType(symbol, x); },`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](const DerivedTypeDetails &x) { CheckDerivedType(symbol, x); },`。
- **L293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](const GenericDetails &x) { CheckGeneric(symbol, x); },`.
  **L293 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](const GenericDetails &x) { CheckGeneric(symbol, x); },`。
- **L294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[](const auto &) {},`.
  **L294 CN**: 继续一个多行参数列表、初始化器或聚合项：`[](const auto &) {},`。
- **L295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L295 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L296 EN**: Executes a call or declaration centered on `symbol.details`.
  **L296 CN**: 执行以 `symbol.details` 为核心的调用或声明。
- **L297 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L297 CN**: 开始 `if` 控制流语句并计算其条件。
- **L298 EN**: Executes a call or declaration centered on `CheckVolatile`.
  **L298 CN**: 执行以 `CheckVolatile` 为核心的调用或声明。
- **L299 EN**: Closes the current lexical scope or compound statement.
  **L299 CN**: 结束当前词法作用域或复合语句块。
- **L300 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L300 CN**: 开始 `if` 控制流语句并计算其条件。
- **L301 EN**: Executes a call or declaration centered on `CheckBindC`.
  **L301 CN**: 执行以 `CheckBindC` 为核心的调用或声明。
- **L302 EN**: Closes the current lexical scope or compound statement.
  **L302 CN**: 结束当前词法作用域或复合语句块。
- **L303 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L303 CN**: 开始 `if` 控制流语句并计算其条件。
- **L304 EN**: Starts a function, method, lambda, or structured scope: `!symbol.implicitAttrs().test(Attr::SAVE)) {`.
  **L304 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!symbol.implicitAttrs().test(Attr::SAVE)) {`。
- **L305 EN**: Executes a call or declaration centered on `CheckExplicitSave`.
  **L305 CN**: 执行以 `CheckExplicitSave` 为核心的调用或声明。
- **L306 EN**: Closes the current lexical scope or compound statement.
  **L306 CN**: 结束当前词法作用域或复合语句块。
- **L307 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L307 CN**: 开始 `if` 控制流语句并计算其条件。
- **L308 EN**: Executes a call or declaration centered on `CheckContiguous`.
  **L308 CN**: 执行以 `CheckContiguous` 为核心的调用或声明。
- **L309 EN**: Closes the current lexical scope or compound statement.
  **L309 CN**: 结束当前词法作用域或复合语句块。
- **L310 EN**: Executes a call or declaration centered on `CheckGlobalName`.
  **L310 CN**: 执行以 `CheckGlobalName` 为核心的调用或声明。
- **L311 EN**: Executes a call or declaration centered on `CheckProcedureAssemblyName`.
  **L311 CN**: 执行以 `CheckProcedureAssemblyName` 为核心的调用或声明。
- **L312 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L312 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 313-336

````cpp
      !evaluate::IsVariable(symbol)) {
    messages_.Say(
        "An entity may not have the ASYNCHRONOUS attribute unless it is a variable"_err_en_US);
  }
  if (symbol.attrs().HasAny({Attr::INTENT_IN, Attr::INTENT_INOUT,
          Attr::INTENT_OUT, Attr::OPTIONAL, Attr::VALUE}) &&
      !IsDummy(symbol)) {
    if (context_.IsEnabled(
            common::LanguageFeature::IgnoreIrrelevantAttributes)) {
      Warn(common::LanguageFeature::IgnoreIrrelevantAttributes,
          "Only a dummy argument should have an INTENT, VALUE, or OPTIONAL attribute"_warn_en_US);
    } else {
      messages_.Say(
          "Only a dummy argument may have an INTENT, VALUE, or OPTIONAL attribute"_err_en_US);
    }
  } else if (symbol.attrs().test(Attr::VALUE)) {
    CheckValue(symbol, derived);
  }

  if (isDone) {
    return; // following checks do not apply
  }

  if (symbol.attrs().test(Attr::PROTECTED)) {
````
- **L313 EN**: Starts a function, method, lambda, or structured scope: `!evaluate::IsVariable(symbol)) {`.
  **L313 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!evaluate::IsVariable(symbol)) {`。
- **L314 EN**: Continues logic associated with callable symbol `Say`.
  **L314 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L315 EN**: Executes a standalone statement or declaration: `"An entity may not have the ASYNCHRONOUS attribute unless it is a variable"_err_en_US);`.
  **L315 CN**: 执行一条独立语句或声明：`"An entity may not have the ASYNCHRONOUS attribute unless it is a variable"_err_en_US);`。
- **L316 EN**: Closes the current lexical scope or compound statement.
  **L316 CN**: 结束当前词法作用域或复合语句块。
- **L317 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L317 CN**: 开始 `if` 控制流语句并计算其条件。
- **L318 EN**: Continues the surrounding expression or declaration: `Attr::INTENT_OUT, Attr::OPTIONAL, Attr::VALUE}) &&`.
  **L318 CN**: 继续构造周围的表达式或声明：`Attr::INTENT_OUT, Attr::OPTIONAL, Attr::VALUE}) &&`。
- **L319 EN**: Starts a function, method, lambda, or structured scope: `!IsDummy(symbol)) {`.
  **L319 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!IsDummy(symbol)) {`。
- **L320 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L320 CN**: 开始 `if` 控制流语句并计算其条件。
- **L321 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::IgnoreIrrelevantAttributes)) {`.
  **L321 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::IgnoreIrrelevantAttributes)) {`。
- **L322 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::LanguageFeature::IgnoreIrrelevantAttributes,`.
  **L322 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::LanguageFeature::IgnoreIrrelevantAttributes,`。
- **L323 EN**: Executes a standalone statement or declaration: `"Only a dummy argument should have an INTENT, VALUE, or OPTIONAL attribute"_warn_en_US);`.
  **L323 CN**: 执行一条独立语句或声明：`"Only a dummy argument should have an INTENT, VALUE, or OPTIONAL attribute"_warn_en_US);`。
- **L324 EN**: Transitions from the previous branch into the alternative path.
  **L324 CN**: 从前一个分支过渡到备选路径。
- **L325 EN**: Continues logic associated with callable symbol `Say`.
  **L325 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L326 EN**: Executes a standalone statement or declaration: `"Only a dummy argument may have an INTENT, VALUE, or OPTIONAL attribute"_err_en_US);`.
  **L326 CN**: 执行一条独立语句或声明：`"Only a dummy argument may have an INTENT, VALUE, or OPTIONAL attribute"_err_en_US);`。
- **L327 EN**: Closes the current lexical scope or compound statement.
  **L327 CN**: 结束当前词法作用域或复合语句块。
- **L328 EN**: Transitions from the previous branch into an `else if` condition.
  **L328 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L329 EN**: Executes a call or declaration centered on `CheckValue`.
  **L329 CN**: 执行以 `CheckValue` 为核心的调用或声明。
- **L330 EN**: Closes the current lexical scope or compound statement.
  **L330 CN**: 结束当前词法作用域或复合语句块。
- **L331 EN**: Blank line separating nearby declarations or logic blocks.
  **L331 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L332 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L332 CN**: 开始 `if` 控制流语句并计算其条件。
- **L333 EN**: Returns from the current function with `; // following checks do not apply`.
  **L333 CN**: 以 `; // following checks do not apply` 从当前函数返回。
- **L334 EN**: Closes the current lexical scope or compound statement.
  **L334 CN**: 结束当前词法作用域或复合语句块。
- **L335 EN**: Blank line separating nearby declarations or logic blocks.
  **L335 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L336 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L336 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 337-360

````cpp
    if (symbol.owner().kind() != Scope::Kind::Module) { // C854
      messages_.Say(
          "A PROTECTED entity must be in the specification part of a module"_err_en_US);
    }
    if (!evaluate::IsVariable(symbol) && !IsProcedurePointer(symbol)) { // C855
      messages_.Say(
          "A PROTECTED entity must be a variable or pointer"_err_en_US);
    }
    if (FindCommonBlockContaining(symbol)) { // C856
      messages_.Say(
          "A PROTECTED entity may not be in a common block"_err_en_US);
    }
  }
  if (IsPointer(symbol)) {
    CheckPointer(symbol);
  }
  if (InPure()) {
    if (InInterface()) {
      // Declarations in interface definitions "have no effect" if they
      // are not pertinent to the characteristics of the procedure.
      // Restrictions on entities in pure procedure interfaces don't need
      // enforcement.
    } else if (symbol.has<AssocEntityDetails>() ||
        FindCommonBlockContaining(symbol)) {
````
- **L337 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L337 CN**: 开始 `if` 控制流语句并计算其条件。
- **L338 EN**: Continues logic associated with callable symbol `Say`.
  **L338 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L339 EN**: Executes a standalone statement or declaration: `"A PROTECTED entity must be in the specification part of a module"_err_en_US);`.
  **L339 CN**: 执行一条独立语句或声明：`"A PROTECTED entity must be in the specification part of a module"_err_en_US);`。
- **L340 EN**: Closes the current lexical scope or compound statement.
  **L340 CN**: 结束当前词法作用域或复合语句块。
- **L341 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L341 CN**: 开始 `if` 控制流语句并计算其条件。
- **L342 EN**: Continues logic associated with callable symbol `Say`.
  **L342 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L343 EN**: Executes a standalone statement or declaration: `"A PROTECTED entity must be a variable or pointer"_err_en_US);`.
  **L343 CN**: 执行一条独立语句或声明：`"A PROTECTED entity must be a variable or pointer"_err_en_US);`。
- **L344 EN**: Closes the current lexical scope or compound statement.
  **L344 CN**: 结束当前词法作用域或复合语句块。
- **L345 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L345 CN**: 开始 `if` 控制流语句并计算其条件。
- **L346 EN**: Continues logic associated with callable symbol `Say`.
  **L346 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L347 EN**: Executes a standalone statement or declaration: `"A PROTECTED entity may not be in a common block"_err_en_US);`.
  **L347 CN**: 执行一条独立语句或声明：`"A PROTECTED entity may not be in a common block"_err_en_US);`。
- **L348 EN**: Closes the current lexical scope or compound statement.
  **L348 CN**: 结束当前词法作用域或复合语句块。
- **L349 EN**: Closes the current lexical scope or compound statement.
  **L349 CN**: 结束当前词法作用域或复合语句块。
- **L350 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L350 CN**: 开始 `if` 控制流语句并计算其条件。
- **L351 EN**: Executes a call or declaration centered on `CheckPointer`.
  **L351 CN**: 执行以 `CheckPointer` 为核心的调用或声明。
- **L352 EN**: Closes the current lexical scope or compound statement.
  **L352 CN**: 结束当前词法作用域或复合语句块。
- **L353 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L353 CN**: 开始 `if` 控制流语句并计算其条件。
- **L354 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L354 CN**: 开始 `if` 控制流语句并计算其条件。
- **L355 EN**: Comment explains nearby logic, intent, or metadata: `Declarations in interface definitions "have no effect" if they`.
  **L355 CN**: 注释说明附近代码的逻辑、意图或元数据：`Declarations in interface definitions "have no effect" if they`。
- **L356 EN**: Comment explains nearby logic, intent, or metadata: `are not pertinent to the characteristics of the procedure.`.
  **L356 CN**: 注释说明附近代码的逻辑、意图或元数据：`are not pertinent to the characteristics of the procedure.`。
- **L357 EN**: Comment explains nearby logic, intent, or metadata: `Restrictions on entities in pure procedure interfaces don't need`.
  **L357 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restrictions on entities in pure procedure interfaces don't need`。
- **L358 EN**: Comment explains nearby logic, intent, or metadata: `enforcement.`.
  **L358 CN**: 注释说明附近代码的逻辑、意图或元数据：`enforcement.`。
- **L359 EN**: Transitions from the previous branch into an `else if` condition.
  **L359 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L360 EN**: Starts a function, method, lambda, or structured scope: `FindCommonBlockContaining(symbol)) {`.
  **L360 CN**: 开始一个函数、方法、lambda 或结构化作用域：`FindCommonBlockContaining(symbol)) {`。

### Lines 361-384

````cpp
      // can look like they have SAVE but are fine in PURE
    } else if (IsSaved(symbol)) {
      if (IsInitialized(symbol)) {
        messages_.Say(
            "A pure subprogram may not initialize a variable"_err_en_US);
      } else {
        messages_.Say(
            "A pure subprogram may not have a variable with the SAVE attribute"_err_en_US);
      }
    }
    if (symbol.attrs().test(Attr::VOLATILE) &&
        (IsDummy(symbol) || !InInterface())) {
      messages_.Say(
          "A pure subprogram may not have a variable with the VOLATILE attribute"_err_en_US);
    }
    if (innermostSymbol_ && innermostSymbol_->name() == "__builtin_c_funloc") {
      // The intrinsic procedure C_FUNLOC() gets a pass on this check.
    } else if (IsProcedure(symbol) && !IsPureProcedure(symbol) &&
        IsDummy(symbol)) {
      messages_.Say(
          "A dummy procedure of a pure subprogram must be pure"_err_en_US);
    }
  }
  const auto *object{symbol.detailsIf<ObjectEntityDetails>()};
````
- **L361 EN**: Comment explains nearby logic, intent, or metadata: `can look like they have SAVE but are fine in PURE`.
  **L361 CN**: 注释说明附近代码的逻辑、意图或元数据：`can look like they have SAVE but are fine in PURE`。
- **L362 EN**: Transitions from the previous branch into an `else if` condition.
  **L362 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L363 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L363 CN**: 开始 `if` 控制流语句并计算其条件。
- **L364 EN**: Continues logic associated with callable symbol `Say`.
  **L364 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L365 EN**: Executes a standalone statement or declaration: `"A pure subprogram may not initialize a variable"_err_en_US);`.
  **L365 CN**: 执行一条独立语句或声明：`"A pure subprogram may not initialize a variable"_err_en_US);`。
- **L366 EN**: Transitions from the previous branch into the alternative path.
  **L366 CN**: 从前一个分支过渡到备选路径。
- **L367 EN**: Continues logic associated with callable symbol `Say`.
  **L367 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L368 EN**: Executes a standalone statement or declaration: `"A pure subprogram may not have a variable with the SAVE attribute"_err_en_US);`.
  **L368 CN**: 执行一条独立语句或声明：`"A pure subprogram may not have a variable with the SAVE attribute"_err_en_US);`。
- **L369 EN**: Closes the current lexical scope or compound statement.
  **L369 CN**: 结束当前词法作用域或复合语句块。
- **L370 EN**: Closes the current lexical scope or compound statement.
  **L370 CN**: 结束当前词法作用域或复合语句块。
- **L371 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L371 CN**: 开始 `if` 控制流语句并计算其条件。
- **L372 EN**: Starts a function, method, lambda, or structured scope: `(IsDummy(symbol) || !InInterface())) {`.
  **L372 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(IsDummy(symbol) || !InInterface())) {`。
- **L373 EN**: Continues logic associated with callable symbol `Say`.
  **L373 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L374 EN**: Executes a standalone statement or declaration: `"A pure subprogram may not have a variable with the VOLATILE attribute"_err_en_US);`.
  **L374 CN**: 执行一条独立语句或声明：`"A pure subprogram may not have a variable with the VOLATILE attribute"_err_en_US);`。
- **L375 EN**: Closes the current lexical scope or compound statement.
  **L375 CN**: 结束当前词法作用域或复合语句块。
- **L376 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L376 CN**: 开始 `if` 控制流语句并计算其条件。
- **L377 EN**: Comment explains nearby logic, intent, or metadata: `The intrinsic procedure C_FUNLOC() gets a pass on this check.`.
  **L377 CN**: 注释说明附近代码的逻辑、意图或元数据：`The intrinsic procedure C_FUNLOC() gets a pass on this check.`。
- **L378 EN**: Transitions from the previous branch into an `else if` condition.
  **L378 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L379 EN**: Starts a function, method, lambda, or structured scope: `IsDummy(symbol)) {`.
  **L379 CN**: 开始一个函数、方法、lambda 或结构化作用域：`IsDummy(symbol)) {`。
- **L380 EN**: Continues logic associated with callable symbol `Say`.
  **L380 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L381 EN**: Executes a standalone statement or declaration: `"A dummy procedure of a pure subprogram must be pure"_err_en_US);`.
  **L381 CN**: 执行一条独立语句或声明：`"A dummy procedure of a pure subprogram must be pure"_err_en_US);`。
- **L382 EN**: Closes the current lexical scope or compound statement.
  **L382 CN**: 结束当前词法作用域或复合语句块。
- **L383 EN**: Closes the current lexical scope or compound statement.
  **L383 CN**: 结束当前词法作用域或复合语句块。
- **L384 EN**: Executes a call or declaration centered on `*object{symbol.detailsIf<ObjectEntityDetails>`.
  **L384 CN**: 执行以 `*object{symbol.detailsIf<ObjectEntityDetails>` 为核心的调用或声明。

### Lines 385-408

````cpp
  if (type) { // Section 7.2, paragraph 7; C795
    bool isChar{type->category() == DeclTypeSpec::Character};
    bool canHaveAssumedParameter{(isChar && IsNamedConstant(symbol)) ||
        (IsAssumedLengthCharacter(symbol) && // C722
            (IsExternal(symbol) ||
                ClassifyProcedure(symbol) ==
                    ProcedureDefinitionClass::Dummy)) ||
        symbol.test(Symbol::Flag::ParentComp)};
    if (!IsStmtFunctionDummy(symbol)) { // C726
      if (object) {
        canHaveAssumedParameter |= object->isDummy() ||
            (isChar && object->isFuncResult()) ||
            IsStmtFunctionResult(symbol); // Avoids multiple messages
      } else {
        canHaveAssumedParameter |= symbol.has<AssocEntityDetails>();
      }
    }
    if (IsProcedurePointer(symbol) && symbol.HasExplicitInterface()) {
      // Don't check function result types here
    } else {
      Check(*type, canHaveAssumedParameter);
    }
    if (InFunction() && IsFunctionResult(symbol)) {
      if (InPure()) {
````
- **L385 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L385 CN**: 开始 `if` 控制流语句并计算其条件。
- **L386 EN**: Executes a call or declaration centered on `isChar{type->category`.
  **L386 CN**: 执行以 `isChar{type->category` 为核心的调用或声明。
- **L387 EN**: Continues logic associated with callable symbol `IsNamedConstant`.
  **L387 CN**: 继续与可调用符号 `IsNamedConstant` 相关的逻辑。
- **L388 EN**: Continues logic associated with callable symbol `IsAssumedLengthCharacter`.
  **L388 CN**: 继续与可调用符号 `IsAssumedLengthCharacter` 相关的逻辑。
- **L389 EN**: Continues logic associated with callable symbol `IsExternal`.
  **L389 CN**: 继续与可调用符号 `IsExternal` 相关的逻辑。
- **L390 EN**: Continues logic associated with callable symbol `ClassifyProcedure`.
  **L390 CN**: 继续与可调用符号 `ClassifyProcedure` 相关的逻辑。
- **L391 EN**: Continues the surrounding expression or declaration: `ProcedureDefinitionClass::Dummy)) ||`.
  **L391 CN**: 继续构造周围的表达式或声明：`ProcedureDefinitionClass::Dummy)) ||`。
- **L392 EN**: Executes a call or declaration centered on `symbol.test`.
  **L392 CN**: 执行以 `symbol.test` 为核心的调用或声明。
- **L393 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L393 CN**: 开始 `if` 控制流语句并计算其条件。
- **L394 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L394 CN**: 开始 `if` 控制流语句并计算其条件。
- **L395 EN**: Continues logic associated with callable symbol `isDummy`.
  **L395 CN**: 继续与可调用符号 `isDummy` 相关的逻辑。
- **L396 EN**: Continues logic associated with callable symbol `isFuncResult`.
  **L396 CN**: 继续与可调用符号 `isFuncResult` 相关的逻辑。
- **L397 EN**: Continues logic associated with callable symbol `IsStmtFunctionResult`.
  **L397 CN**: 继续与可调用符号 `IsStmtFunctionResult` 相关的逻辑。
- **L398 EN**: Transitions from the previous branch into the alternative path.
  **L398 CN**: 从前一个分支过渡到备选路径。
- **L399 EN**: Executes a call or declaration centered on `symbol.has<AssocEntityDetails>`.
  **L399 CN**: 执行以 `symbol.has<AssocEntityDetails>` 为核心的调用或声明。
- **L400 EN**: Closes the current lexical scope or compound statement.
  **L400 CN**: 结束当前词法作用域或复合语句块。
- **L401 EN**: Closes the current lexical scope or compound statement.
  **L401 CN**: 结束当前词法作用域或复合语句块。
- **L402 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L402 CN**: 开始 `if` 控制流语句并计算其条件。
- **L403 EN**: Comment explains nearby logic, intent, or metadata: `Don't check function result types here`.
  **L403 CN**: 注释说明附近代码的逻辑、意图或元数据：`Don't check function result types here`。
- **L404 EN**: Transitions from the previous branch into the alternative path.
  **L404 CN**: 从前一个分支过渡到备选路径。
- **L405 EN**: Executes a call or declaration centered on `Check`.
  **L405 CN**: 执行以 `Check` 为核心的调用或声明。
- **L406 EN**: Closes the current lexical scope or compound statement.
  **L406 CN**: 结束当前词法作用域或复合语句块。
- **L407 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L407 CN**: 开始 `if` 控制流语句并计算其条件。
- **L408 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L408 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 409-432

````cpp
        if (type->IsPolymorphic() && IsAllocatable(symbol)) { // C1585
          messages_.Say(
              "Result of pure function may not be both polymorphic and ALLOCATABLE"_err_en_US);
        }
        if (derived) {
          // These cases would be caught be the general validation of local
          // variables in a pure context, but these messages are more specific.
          if (HasImpureFinal(symbol)) { // C1584
            messages_.Say(
                "Result of pure function may not have an impure FINAL subroutine"_err_en_US);
          }
          if (auto bad{
                  FindPolymorphicAllocatablePotentialComponent(*derived)}) {
            SayWithDeclaration(*bad,
                "Result of pure function may not have polymorphic ALLOCATABLE potential component '%s'"_err_en_US,
                bad.BuildResultDesignatorName());
          }
        }
      }
      if (InElemental() && isChar) { // F'2023 C15121
        CheckSpecExpr(type->characterTypeSpec().length().GetExplicit(),
            /*forElementalFunctionResult=*/true);
        // TODO: check PDT LEN parameters
      }
````
- **L409 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L409 CN**: 开始 `if` 控制流语句并计算其条件。
- **L410 EN**: Continues logic associated with callable symbol `Say`.
  **L410 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L411 EN**: Executes a standalone statement or declaration: `"Result of pure function may not be both polymorphic and ALLOCATABLE"_err_en_US);`.
  **L411 CN**: 执行一条独立语句或声明：`"Result of pure function may not be both polymorphic and ALLOCATABLE"_err_en_US);`。
- **L412 EN**: Closes the current lexical scope or compound statement.
  **L412 CN**: 结束当前词法作用域或复合语句块。
- **L413 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L413 CN**: 开始 `if` 控制流语句并计算其条件。
- **L414 EN**: Comment explains nearby logic, intent, or metadata: `These cases would be caught be the general validation of local`.
  **L414 CN**: 注释说明附近代码的逻辑、意图或元数据：`These cases would be caught be the general validation of local`。
- **L415 EN**: Comment explains nearby logic, intent, or metadata: `variables in a pure context, but these messages are more specific.`.
  **L415 CN**: 注释说明附近代码的逻辑、意图或元数据：`variables in a pure context, but these messages are more specific.`。
- **L416 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L416 CN**: 开始 `if` 控制流语句并计算其条件。
- **L417 EN**: Continues logic associated with callable symbol `Say`.
  **L417 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L418 EN**: Executes a standalone statement or declaration: `"Result of pure function may not have an impure FINAL subroutine"_err_en_US);`.
  **L418 CN**: 执行一条独立语句或声明：`"Result of pure function may not have an impure FINAL subroutine"_err_en_US);`。
- **L419 EN**: Closes the current lexical scope or compound statement.
  **L419 CN**: 结束当前词法作用域或复合语句块。
- **L420 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L420 CN**: 开始 `if` 控制流语句并计算其条件。
- **L421 EN**: Starts a function, method, lambda, or structured scope: `FindPolymorphicAllocatablePotentialComponent(*derived)}) {`.
  **L421 CN**: 开始一个函数、方法、lambda 或结构化作用域：`FindPolymorphicAllocatablePotentialComponent(*derived)}) {`。
- **L422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*bad,`.
  **L422 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*bad,`。
- **L423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Result of pure function may not have polymorphic ALLOCATABLE potential component '%s'"_err_en_US,`.
  **L423 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Result of pure function may not have polymorphic ALLOCATABLE potential component '%s'"_err_en_US,`。
- **L424 EN**: Executes a call or declaration centered on `bad.BuildResultDesignatorName`.
  **L424 CN**: 执行以 `bad.BuildResultDesignatorName` 为核心的调用或声明。
- **L425 EN**: Closes the current lexical scope or compound statement.
  **L425 CN**: 结束当前词法作用域或复合语句块。
- **L426 EN**: Closes the current lexical scope or compound statement.
  **L426 CN**: 结束当前词法作用域或复合语句块。
- **L427 EN**: Closes the current lexical scope or compound statement.
  **L427 CN**: 结束当前词法作用域或复合语句块。
- **L428 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L428 CN**: 开始 `if` 控制流语句并计算其条件。
- **L429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckSpecExpr(type->characterTypeSpec().length().GetExplicit(),`.
  **L429 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckSpecExpr(type->characterTypeSpec().length().GetExplicit(),`。
- **L430 EN**: Comment explains nearby logic, intent, or metadata: `forElementalFunctionResult=*/true);`.
  **L430 CN**: 注释说明附近代码的逻辑、意图或元数据：`forElementalFunctionResult=*/true);`。
- **L431 EN**: Comment records a pending task or caution: `TODO: check PDT LEN parameters`.
  **L431 CN**: 注释记录待办事项或注意点：`TODO: check PDT LEN parameters`。
- **L432 EN**: Closes the current lexical scope or compound statement.
  **L432 CN**: 结束当前词法作用域或复合语句块。

### Lines 433-456

````cpp
    }
  }
  if (IsAssumedLengthCharacter(symbol) && IsFunction(symbol)) { // C723
    if (symbol.attrs().test(Attr::RECURSIVE)) {
      messages_.Say(
          "An assumed-length CHARACTER(*) function cannot be RECURSIVE"_err_en_US);
    }
    if (symbol.Rank() > 0) {
      messages_.Say(
          "An assumed-length CHARACTER(*) function cannot return an array"_err_en_US);
    }
    if (!IsStmtFunction(symbol)) {
      if (IsElementalProcedure(symbol)) {
        messages_.Say(
            "An assumed-length CHARACTER(*) function cannot be ELEMENTAL"_err_en_US);
      } else if (IsPureProcedure(symbol)) {
        messages_.Say(
            "An assumed-length CHARACTER(*) function cannot be PURE"_err_en_US);
      }
    }
    if (const Symbol *result{FindFunctionResult(symbol)}) {
      if (IsPointer(*result)) {
        messages_.Say(
            "An assumed-length CHARACTER(*) function cannot return a POINTER"_err_en_US);
````
- **L433 EN**: Closes the current lexical scope or compound statement.
  **L433 CN**: 结束当前词法作用域或复合语句块。
- **L434 EN**: Closes the current lexical scope or compound statement.
  **L434 CN**: 结束当前词法作用域或复合语句块。
- **L435 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L435 CN**: 开始 `if` 控制流语句并计算其条件。
- **L436 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L436 CN**: 开始 `if` 控制流语句并计算其条件。
- **L437 EN**: Continues logic associated with callable symbol `Say`.
  **L437 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L438 EN**: Executes a call or declaration centered on `CHARACTER`.
  **L438 CN**: 执行以 `CHARACTER` 为核心的调用或声明。
- **L439 EN**: Closes the current lexical scope or compound statement.
  **L439 CN**: 结束当前词法作用域或复合语句块。
- **L440 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L440 CN**: 开始 `if` 控制流语句并计算其条件。
- **L441 EN**: Continues logic associated with callable symbol `Say`.
  **L441 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L442 EN**: Executes a call or declaration centered on `CHARACTER`.
  **L442 CN**: 执行以 `CHARACTER` 为核心的调用或声明。
- **L443 EN**: Closes the current lexical scope or compound statement.
  **L443 CN**: 结束当前词法作用域或复合语句块。
- **L444 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L444 CN**: 开始 `if` 控制流语句并计算其条件。
- **L445 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L445 CN**: 开始 `if` 控制流语句并计算其条件。
- **L446 EN**: Continues logic associated with callable symbol `Say`.
  **L446 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L447 EN**: Executes a call or declaration centered on `CHARACTER`.
  **L447 CN**: 执行以 `CHARACTER` 为核心的调用或声明。
- **L448 EN**: Transitions from the previous branch into an `else if` condition.
  **L448 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L449 EN**: Continues logic associated with callable symbol `Say`.
  **L449 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L450 EN**: Executes a call or declaration centered on `CHARACTER`.
  **L450 CN**: 执行以 `CHARACTER` 为核心的调用或声明。
- **L451 EN**: Closes the current lexical scope or compound statement.
  **L451 CN**: 结束当前词法作用域或复合语句块。
- **L452 EN**: Closes the current lexical scope or compound statement.
  **L452 CN**: 结束当前词法作用域或复合语句块。
- **L453 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L453 CN**: 开始 `if` 控制流语句并计算其条件。
- **L454 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L454 CN**: 开始 `if` 控制流语句并计算其条件。
- **L455 EN**: Continues logic associated with callable symbol `Say`.
  **L455 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L456 EN**: Executes a call or declaration centered on `CHARACTER`.
  **L456 CN**: 执行以 `CHARACTER` 为核心的调用或声明。

### Lines 457-480

````cpp
      }
    }
    if (IsProcedurePointer(symbol) && IsDummy(symbol)) {
      Warn(common::UsageWarning::Portability,
          "A dummy procedure pointer should not have assumed-length CHARACTER(*) result type"_port_en_US);
      // The non-dummy case is a hard error that's caught elsewhere.
    }
  }
  if (IsDummy(symbol)) {
    if (IsNamedConstant(symbol)) {
      messages_.Say(
          "A dummy argument may not also be a named constant"_err_en_US);
    }
  } else if (IsFunctionResult(symbol)) {
    if (IsNamedConstant(symbol)) {
      messages_.Say(
          "A function result may not also be a named constant"_err_en_US);
    }
    if (!IsProcedurePointer(symbol) && IsProcedure(symbol)) {
      messages_.Say(
          "A function result may not be a procedure unless it is a procedure pointer"_err_en_US);
    }
  }
  if (IsAutomatic(symbol)) {
````
- **L457 EN**: Closes the current lexical scope or compound statement.
  **L457 CN**: 结束当前词法作用域或复合语句块。
- **L458 EN**: Closes the current lexical scope or compound statement.
  **L458 CN**: 结束当前词法作用域或复合语句块。
- **L459 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L459 CN**: 开始 `if` 控制流语句并计算其条件。
- **L460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::Portability,`.
  **L460 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::Portability,`。
- **L461 EN**: Executes a call or declaration centered on `CHARACTER`.
  **L461 CN**: 执行以 `CHARACTER` 为核心的调用或声明。
- **L462 EN**: Comment explains nearby logic, intent, or metadata: `The non-dummy case is a hard error that's caught elsewhere.`.
  **L462 CN**: 注释说明附近代码的逻辑、意图或元数据：`The non-dummy case is a hard error that's caught elsewhere.`。
- **L463 EN**: Closes the current lexical scope or compound statement.
  **L463 CN**: 结束当前词法作用域或复合语句块。
- **L464 EN**: Closes the current lexical scope or compound statement.
  **L464 CN**: 结束当前词法作用域或复合语句块。
- **L465 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L465 CN**: 开始 `if` 控制流语句并计算其条件。
- **L466 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L466 CN**: 开始 `if` 控制流语句并计算其条件。
- **L467 EN**: Continues logic associated with callable symbol `Say`.
  **L467 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L468 EN**: Executes a standalone statement or declaration: `"A dummy argument may not also be a named constant"_err_en_US);`.
  **L468 CN**: 执行一条独立语句或声明：`"A dummy argument may not also be a named constant"_err_en_US);`。
- **L469 EN**: Closes the current lexical scope or compound statement.
  **L469 CN**: 结束当前词法作用域或复合语句块。
- **L470 EN**: Transitions from the previous branch into an `else if` condition.
  **L470 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L471 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L471 CN**: 开始 `if` 控制流语句并计算其条件。
- **L472 EN**: Continues logic associated with callable symbol `Say`.
  **L472 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L473 EN**: Executes a standalone statement or declaration: `"A function result may not also be a named constant"_err_en_US);`.
  **L473 CN**: 执行一条独立语句或声明：`"A function result may not also be a named constant"_err_en_US);`。
- **L474 EN**: Closes the current lexical scope or compound statement.
  **L474 CN**: 结束当前词法作用域或复合语句块。
- **L475 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L475 CN**: 开始 `if` 控制流语句并计算其条件。
- **L476 EN**: Continues logic associated with callable symbol `Say`.
  **L476 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L477 EN**: Executes a standalone statement or declaration: `"A function result may not be a procedure unless it is a procedure pointer"_err_en_US);`.
  **L477 CN**: 执行一条独立语句或声明：`"A function result may not be a procedure unless it is a procedure pointer"_err_en_US);`。
- **L478 EN**: Closes the current lexical scope or compound statement.
  **L478 CN**: 结束当前词法作用域或复合语句块。
- **L479 EN**: Closes the current lexical scope or compound statement.
  **L479 CN**: 结束当前词法作用域或复合语句块。
- **L480 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L480 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 481-504

````cpp
    if (const Symbol * common{FindCommonBlockContaining(symbol)}) {
      messages_.Say(
          "Automatic data object '%s' may not appear in COMMON block /%s/"_err_en_US,
          symbol.name(), common->name());
    } else if (symbol.owner().IsModule()) {
      messages_.Say(
          "Automatic data object '%s' may not appear in a module"_err_en_US,
          symbol.name());
    } else if (IsBlockData(symbol.owner())) {
      messages_.Say(
          "Automatic data object '%s' may not appear in a BLOCK DATA subprogram"_err_en_US,
          symbol.name());
    } else if (symbol.owner().kind() == Scope::Kind::MainProgram) {
      if (context_.IsEnabled(common::LanguageFeature::AutomaticInMainProgram)) {
        Warn(common::LanguageFeature::AutomaticInMainProgram,
            "Automatic data object '%s' should not appear in the specification part of a main program"_port_en_US,
            symbol.name());
      } else {
        messages_.Say(
            "Automatic data object '%s' may not appear in the specification part of a main program"_err_en_US,
            symbol.name());
      }
    }
  }
````
- **L481 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L481 CN**: 开始 `if` 控制流语句并计算其条件。
- **L482 EN**: Continues logic associated with callable symbol `Say`.
  **L482 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Automatic data object '%s' may not appear in COMMON block /%s/"_err_en_US,`.
  **L483 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Automatic data object '%s' may not appear in COMMON block /%s/"_err_en_US,`。
- **L484 EN**: Executes a call or declaration centered on `symbol.name`.
  **L484 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L485 EN**: Transitions from the previous branch into an `else if` condition.
  **L485 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L486 EN**: Continues logic associated with callable symbol `Say`.
  **L486 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Automatic data object '%s' may not appear in a module"_err_en_US,`.
  **L487 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Automatic data object '%s' may not appear in a module"_err_en_US,`。
- **L488 EN**: Executes a call or declaration centered on `symbol.name`.
  **L488 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L489 EN**: Transitions from the previous branch into an `else if` condition.
  **L489 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L490 EN**: Continues logic associated with callable symbol `Say`.
  **L490 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Automatic data object '%s' may not appear in a BLOCK DATA subprogram"_err_en_US,`.
  **L491 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Automatic data object '%s' may not appear in a BLOCK DATA subprogram"_err_en_US,`。
- **L492 EN**: Executes a call or declaration centered on `symbol.name`.
  **L492 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L493 EN**: Transitions from the previous branch into an `else if` condition.
  **L493 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L494 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L494 CN**: 开始 `if` 控制流语句并计算其条件。
- **L495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::LanguageFeature::AutomaticInMainProgram,`.
  **L495 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::LanguageFeature::AutomaticInMainProgram,`。
- **L496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Automatic data object '%s' should not appear in the specification part of a main program"_port_en_US,`.
  **L496 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Automatic data object '%s' should not appear in the specification part of a main program"_port_en_US,`。
- **L497 EN**: Executes a call or declaration centered on `symbol.name`.
  **L497 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L498 EN**: Transitions from the previous branch into the alternative path.
  **L498 CN**: 从前一个分支过渡到备选路径。
- **L499 EN**: Continues logic associated with callable symbol `Say`.
  **L499 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Automatic data object '%s' may not appear in the specification part of a main program"_err_en_US,`.
  **L500 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Automatic data object '%s' may not appear in the specification part of a main program"_err_en_US,`。
- **L501 EN**: Executes a call or declaration centered on `symbol.name`.
  **L501 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L502 EN**: Closes the current lexical scope or compound statement.
  **L502 CN**: 结束当前词法作用域或复合语句块。
- **L503 EN**: Closes the current lexical scope or compound statement.
  **L503 CN**: 结束当前词法作用域或复合语句块。
- **L504 EN**: Closes the current lexical scope or compound statement.
  **L504 CN**: 结束当前词法作用域或复合语句块。

### Lines 505-528

````cpp
  if (IsProcedure(symbol)) {
    if (IsAllocatable(symbol)) {
      messages_.Say(
          "Procedure '%s' may not be ALLOCATABLE"_err_en_US, symbol.name());
    }
    if (!symbol.HasExplicitInterface() && symbol.Rank() > 0) {
      messages_.Say(
          "Procedure '%s' may not be an array without an explicit interface"_err_en_US,
          symbol.name());
    }
  }
}

void CheckHelper::CheckCommonBlock(const Symbol &symbol) {
  CheckGlobalName(symbol);
  const auto &common{symbol.get<CommonBlockDetails>()};
  SourceName location{symbol.name()};
  if (location.empty()) {
    location = common.sourceLocation();
  }
  bool isBindCCommon{symbol.attrs().test(Attr::BIND_C)};
  if (isBindCCommon) {
    CheckBindC(symbol);
  }
````
- **L505 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L505 CN**: 开始 `if` 控制流语句并计算其条件。
- **L506 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L506 CN**: 开始 `if` 控制流语句并计算其条件。
- **L507 EN**: Continues logic associated with callable symbol `Say`.
  **L507 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L508 EN**: Executes a call or declaration centered on `symbol.name`.
  **L508 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L509 EN**: Closes the current lexical scope or compound statement.
  **L509 CN**: 结束当前词法作用域或复合语句块。
- **L510 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L510 CN**: 开始 `if` 控制流语句并计算其条件。
- **L511 EN**: Continues logic associated with callable symbol `Say`.
  **L511 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Procedure '%s' may not be an array without an explicit interface"_err_en_US,`.
  **L512 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Procedure '%s' may not be an array without an explicit interface"_err_en_US,`。
- **L513 EN**: Executes a call or declaration centered on `symbol.name`.
  **L513 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L514 EN**: Closes the current lexical scope or compound statement.
  **L514 CN**: 结束当前词法作用域或复合语句块。
- **L515 EN**: Closes the current lexical scope or compound statement.
  **L515 CN**: 结束当前词法作用域或复合语句块。
- **L516 EN**: Closes the current lexical scope or compound statement.
  **L516 CN**: 结束当前词法作用域或复合语句块。
- **L517 EN**: Blank line separating nearby declarations or logic blocks.
  **L517 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L518 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckCommonBlock(const Symbol &symbol) {`.
  **L518 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckCommonBlock(const Symbol &symbol) {`。
- **L519 EN**: Executes a call or declaration centered on `CheckGlobalName`.
  **L519 CN**: 执行以 `CheckGlobalName` 为核心的调用或声明。
- **L520 EN**: Executes a call or declaration centered on `&common{symbol.get<CommonBlockDetails>`.
  **L520 CN**: 执行以 `&common{symbol.get<CommonBlockDetails>` 为核心的调用或声明。
- **L521 EN**: Executes a call or declaration centered on `location{symbol.name`.
  **L521 CN**: 执行以 `location{symbol.name` 为核心的调用或声明。
- **L522 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L522 CN**: 开始 `if` 控制流语句并计算其条件。
- **L523 EN**: Executes a call or declaration centered on `common.sourceLocation`.
  **L523 CN**: 执行以 `common.sourceLocation` 为核心的调用或声明。
- **L524 EN**: Closes the current lexical scope or compound statement.
  **L524 CN**: 结束当前词法作用域或复合语句块。
- **L525 EN**: Executes a call or declaration centered on `isBindCCommon{symbol.attrs`.
  **L525 CN**: 执行以 `isBindCCommon{symbol.attrs` 为核心的调用或声明。
- **L526 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L526 CN**: 开始 `if` 控制流语句并计算其条件。
- **L527 EN**: Executes a call or declaration centered on `CheckBindC`.
  **L527 CN**: 执行以 `CheckBindC` 为核心的调用或声明。
- **L528 EN**: Closes the current lexical scope or compound statement.
  **L528 CN**: 结束当前词法作用域或复合语句块。

### Lines 529-552

````cpp
  for (auto ref : symbol.get<CommonBlockDetails>().objects()) {
    auto restorer{
        messages_.SetLocation(location.empty() ? ref->name() : location)};
    if (isBindCCommon && ref->has<ObjectEntityDetails>()) {
      if (auto msgs{WhyNotInteroperableObject(*ref,
              /*allowInteroperableType=*/false, /*forCommonBlock=*/true)};
          !msgs.empty()) {
        parser::Message &reason{msgs.messages().front()};
        parser::Message *msg{nullptr};
        if (reason.IsFatal()) {
          msg = messages_.Say(
              "'%s' may not be a member of BIND(C) COMMON block /%s/"_err_en_US,
              ref->name(), symbol.name());
        } else {
          msg = messages_.Say(
              "'%s' should not be a member of BIND(C) COMMON block /%s/"_warn_en_US,
              ref->name(), symbol.name());
        }
        if (msg) {
          msg = &msg->Attach(
              std::move(reason.set_severity(parser::Severity::Because)));
        }
        evaluate::AttachDeclaration(msg, *ref);
      }
````
- **L529 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L529 CN**: 开始 `for` 控制流语句并计算其条件。
- **L530 EN**: Continues the surrounding expression or declaration: `auto restorer{`.
  **L530 CN**: 继续构造周围的表达式或声明：`auto restorer{`。
- **L531 EN**: Executes a call or declaration centered on `messages_.SetLocation`.
  **L531 CN**: 执行以 `messages_.SetLocation` 为核心的调用或声明。
- **L532 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L532 CN**: 开始 `if` 控制流语句并计算其条件。
- **L533 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L533 CN**: 开始 `if` 控制流语句并计算其条件。
- **L534 EN**: Comment explains nearby logic, intent, or metadata: `allowInteroperableType=*/false, /*forCommonBlock=*/true)};`.
  **L534 CN**: 注释说明附近代码的逻辑、意图或元数据：`allowInteroperableType=*/false, /*forCommonBlock=*/true)};`。
- **L535 EN**: Starts a function, method, lambda, or structured scope: `!msgs.empty()) {`.
  **L535 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!msgs.empty()) {`。
- **L536 EN**: Executes a call or declaration centered on `&reason{msgs.messages`.
  **L536 CN**: 执行以 `&reason{msgs.messages` 为核心的调用或声明。
- **L537 EN**: Executes a standalone statement or declaration: `parser::Message *msg{nullptr};`.
  **L537 CN**: 执行一条独立语句或声明：`parser::Message *msg{nullptr};`。
- **L538 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L538 CN**: 开始 `if` 控制流语句并计算其条件。
- **L539 EN**: Continues logic associated with callable symbol `Say`.
  **L539 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' may not be a member of BIND(C) COMMON block /%s/"_err_en_US,`.
  **L540 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' may not be a member of BIND(C) COMMON block /%s/"_err_en_US,`。
- **L541 EN**: Executes a call or declaration centered on `ref->name`.
  **L541 CN**: 执行以 `ref->name` 为核心的调用或声明。
- **L542 EN**: Transitions from the previous branch into the alternative path.
  **L542 CN**: 从前一个分支过渡到备选路径。
- **L543 EN**: Continues logic associated with callable symbol `Say`.
  **L543 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' should not be a member of BIND(C) COMMON block /%s/"_warn_en_US,`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' should not be a member of BIND(C) COMMON block /%s/"_warn_en_US,`。
- **L545 EN**: Executes a call or declaration centered on `ref->name`.
  **L545 CN**: 执行以 `ref->name` 为核心的调用或声明。
- **L546 EN**: Closes the current lexical scope or compound statement.
  **L546 CN**: 结束当前词法作用域或复合语句块。
- **L547 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L547 CN**: 开始 `if` 控制流语句并计算其条件。
- **L548 EN**: Continues logic associated with callable symbol `Attach`.
  **L548 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L549 EN**: Executes a call or declaration centered on `std::move`.
  **L549 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L550 EN**: Closes the current lexical scope or compound statement.
  **L550 CN**: 结束当前词法作用域或复合语句块。
- **L551 EN**: Executes a call or declaration centered on `evaluate::AttachDeclaration`.
  **L551 CN**: 执行以 `evaluate::AttachDeclaration` 为核心的调用或声明。
- **L552 EN**: Closes the current lexical scope or compound statement.
  **L552 CN**: 结束当前词法作用域或复合语句块。

### Lines 553-576

````cpp
    }
    if (ref->test(Symbol::Flag::CrayPointee)) {
      evaluate::AttachDeclaration(
          messages_.Say(
              "Cray pointee '%s' may not be a member of COMMON block /%s/"_err_en_US,
              ref->name(), symbol.name()),
          *ref);
    }
    if (IsAllocatable(*ref)) {
      evaluate::AttachDeclaration(
          messages_.Say(
              "ALLOCATABLE object '%s' may not appear in COMMON block /%s/"_err_en_US,
              ref->name(), symbol.name()),
          *ref);
    }
    if (ref->attrs().test(Attr::BIND_C)) {
      evaluate::AttachDeclaration(
          messages_.Say(
              "BIND(C) object '%s' may not appear in COMMON block /%s/"_err_en_US,
              ref->name(), symbol.name()),
          *ref);
    }
    if (IsNamedConstant(*ref)) {
      evaluate::AttachDeclaration(
````
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L554 CN**: 开始 `if` 控制流语句并计算其条件。
- **L555 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L555 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L556 EN**: Continues logic associated with callable symbol `Say`.
  **L556 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Cray pointee '%s' may not be a member of COMMON block /%s/"_err_en_US,`.
  **L557 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Cray pointee '%s' may not be a member of COMMON block /%s/"_err_en_US,`。
- **L558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ref->name(), symbol.name()),`.
  **L558 CN**: 继续一个多行参数列表、初始化器或聚合项：`ref->name(), symbol.name()),`。
- **L559 EN**: Comment explains nearby logic, intent, or metadata: `ref);`.
  **L559 CN**: 注释说明附近代码的逻辑、意图或元数据：`ref);`。
- **L560 EN**: Closes the current lexical scope or compound statement.
  **L560 CN**: 结束当前词法作用域或复合语句块。
- **L561 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L561 CN**: 开始 `if` 控制流语句并计算其条件。
- **L562 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L562 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L563 EN**: Continues logic associated with callable symbol `Say`.
  **L563 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L564 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ALLOCATABLE object '%s' may not appear in COMMON block /%s/"_err_en_US,`.
  **L564 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ALLOCATABLE object '%s' may not appear in COMMON block /%s/"_err_en_US,`。
- **L565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ref->name(), symbol.name()),`.
  **L565 CN**: 继续一个多行参数列表、初始化器或聚合项：`ref->name(), symbol.name()),`。
- **L566 EN**: Comment explains nearby logic, intent, or metadata: `ref);`.
  **L566 CN**: 注释说明附近代码的逻辑、意图或元数据：`ref);`。
- **L567 EN**: Closes the current lexical scope or compound statement.
  **L567 CN**: 结束当前词法作用域或复合语句块。
- **L568 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L568 CN**: 开始 `if` 控制流语句并计算其条件。
- **L569 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L569 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L570 EN**: Continues logic associated with callable symbol `Say`.
  **L570 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"BIND(C) object '%s' may not appear in COMMON block /%s/"_err_en_US,`.
  **L571 CN**: 继续一个多行参数列表、初始化器或聚合项：`"BIND(C) object '%s' may not appear in COMMON block /%s/"_err_en_US,`。
- **L572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ref->name(), symbol.name()),`.
  **L572 CN**: 继续一个多行参数列表、初始化器或聚合项：`ref->name(), symbol.name()),`。
- **L573 EN**: Comment explains nearby logic, intent, or metadata: `ref);`.
  **L573 CN**: 注释说明附近代码的逻辑、意图或元数据：`ref);`。
- **L574 EN**: Closes the current lexical scope or compound statement.
  **L574 CN**: 结束当前词法作用域或复合语句块。
- **L575 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L575 CN**: 开始 `if` 控制流语句并计算其条件。
- **L576 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L576 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。

### Lines 577-600

````cpp
          messages_.Say(
              "Named constant '%s' may not appear in COMMON block /%s/"_err_en_US,
              ref->name(), symbol.name()),
          *ref);
    }
    if (IsDummy(*ref)) {
      evaluate::AttachDeclaration(
          messages_.Say(
              "Dummy argument '%s' may not appear in COMMON block /%s/"_err_en_US,
              ref->name(), symbol.name()),
          *ref);
    }
    if (ref->IsFuncResult()) {
      evaluate::AttachDeclaration(
          messages_.Say(
              "Function result '%s' may not appear in COMMON block /%s/"_err_en_US,
              ref->name(), symbol.name()),
          *ref);
    }
    if (const auto *type{ref->GetType()}) {
      if (type->category() == DeclTypeSpec::ClassStar) {
        evaluate::AttachDeclaration(
            messages_.Say(
                "Unlimited polymorphic pointer '%s' may not appear in COMMON block /%s/"_err_en_US,
````
- **L577 EN**: Continues logic associated with callable symbol `Say`.
  **L577 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Named constant '%s' may not appear in COMMON block /%s/"_err_en_US,`.
  **L578 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Named constant '%s' may not appear in COMMON block /%s/"_err_en_US,`。
- **L579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ref->name(), symbol.name()),`.
  **L579 CN**: 继续一个多行参数列表、初始化器或聚合项：`ref->name(), symbol.name()),`。
- **L580 EN**: Comment explains nearby logic, intent, or metadata: `ref);`.
  **L580 CN**: 注释说明附近代码的逻辑、意图或元数据：`ref);`。
- **L581 EN**: Closes the current lexical scope or compound statement.
  **L581 CN**: 结束当前词法作用域或复合语句块。
- **L582 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L582 CN**: 开始 `if` 控制流语句并计算其条件。
- **L583 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L583 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L584 EN**: Continues logic associated with callable symbol `Say`.
  **L584 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy argument '%s' may not appear in COMMON block /%s/"_err_en_US,`.
  **L585 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy argument '%s' may not appear in COMMON block /%s/"_err_en_US,`。
- **L586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ref->name(), symbol.name()),`.
  **L586 CN**: 继续一个多行参数列表、初始化器或聚合项：`ref->name(), symbol.name()),`。
- **L587 EN**: Comment explains nearby logic, intent, or metadata: `ref);`.
  **L587 CN**: 注释说明附近代码的逻辑、意图或元数据：`ref);`。
- **L588 EN**: Closes the current lexical scope or compound statement.
  **L588 CN**: 结束当前词法作用域或复合语句块。
- **L589 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L589 CN**: 开始 `if` 控制流语句并计算其条件。
- **L590 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L590 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L591 EN**: Continues logic associated with callable symbol `Say`.
  **L591 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Function result '%s' may not appear in COMMON block /%s/"_err_en_US,`.
  **L592 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Function result '%s' may not appear in COMMON block /%s/"_err_en_US,`。
- **L593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ref->name(), symbol.name()),`.
  **L593 CN**: 继续一个多行参数列表、初始化器或聚合项：`ref->name(), symbol.name()),`。
- **L594 EN**: Comment explains nearby logic, intent, or metadata: `ref);`.
  **L594 CN**: 注释说明附近代码的逻辑、意图或元数据：`ref);`。
- **L595 EN**: Closes the current lexical scope or compound statement.
  **L595 CN**: 结束当前词法作用域或复合语句块。
- **L596 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L596 CN**: 开始 `if` 控制流语句并计算其条件。
- **L597 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L597 CN**: 开始 `if` 控制流语句并计算其条件。
- **L598 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L598 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L599 EN**: Continues logic associated with callable symbol `Say`.
  **L599 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Unlimited polymorphic pointer '%s' may not appear in COMMON block /%s/"_err_en_US,`.
  **L600 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Unlimited polymorphic pointer '%s' may not appear in COMMON block /%s/"_err_en_US,`。

### Lines 601-624

````cpp
                ref->name(), symbol.name()),
            *ref);
      } else if (const auto *derived{type->AsDerived()}) {
        if (!IsSequenceOrBindCType(derived)) {
          evaluate::AttachDeclaration(
              evaluate::AttachDeclaration(
                  messages_.Say(
                      "Object '%s' whose derived type '%s' is neither SEQUENCE nor BIND(C) may not appear in COMMON block /%s/"_err_en_US,
                      ref->name(), derived->name(), symbol.name()),
                  derived->typeSymbol()),
              *ref);
        } else if (auto componentPath{
                       derived->ComponentWithDefaultInitialization()}) {
          evaluate::AttachDeclaration(
              evaluate::AttachDeclaration(
                  messages_.Say(
                      "COMMON block /%s/ may not have the member '%s' whose derived type '%s' has a component '%s' that is ALLOCATABLE or has default initialization"_err_en_US,
                      symbol.name(), ref->name(), derived->name(),
                      *componentPath),
                  derived->typeSymbol()),
              *ref);
        }
      }
    }
````
- **L601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ref->name(), symbol.name()),`.
  **L601 CN**: 继续一个多行参数列表、初始化器或聚合项：`ref->name(), symbol.name()),`。
- **L602 EN**: Comment explains nearby logic, intent, or metadata: `ref);`.
  **L602 CN**: 注释说明附近代码的逻辑、意图或元数据：`ref);`。
- **L603 EN**: Transitions from the previous branch into an `else if` condition.
  **L603 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L604 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L604 CN**: 开始 `if` 控制流语句并计算其条件。
- **L605 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L605 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L606 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L606 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L607 EN**: Continues logic associated with callable symbol `Say`.
  **L607 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Object '%s' whose derived type '%s' is neither SEQUENCE nor BIND(C) may not appear in COMMON block /%s/"_err_en_US,`.
  **L608 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Object '%s' whose derived type '%s' is neither SEQUENCE nor BIND(C) may not appear in COMMON block /%s/"_err_en_US,`。
- **L609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ref->name(), derived->name(), symbol.name()),`.
  **L609 CN**: 继续一个多行参数列表、初始化器或聚合项：`ref->name(), derived->name(), symbol.name()),`。
- **L610 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `derived->typeSymbol()),`.
  **L610 CN**: 继续一个多行参数列表、初始化器或聚合项：`derived->typeSymbol()),`。
- **L611 EN**: Comment explains nearby logic, intent, or metadata: `ref);`.
  **L611 CN**: 注释说明附近代码的逻辑、意图或元数据：`ref);`。
- **L612 EN**: Transitions from the previous branch into an `else if` condition.
  **L612 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L613 EN**: Starts a function, method, lambda, or structured scope: `derived->ComponentWithDefaultInitialization()}) {`.
  **L613 CN**: 开始一个函数、方法、lambda 或结构化作用域：`derived->ComponentWithDefaultInitialization()}) {`。
- **L614 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L614 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L615 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L615 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L616 EN**: Continues logic associated with callable symbol `Say`.
  **L616 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"COMMON block /%s/ may not have the member '%s' whose derived type '%s' has a component '%s' that is ALLOCATABLE or has default initialization"_err_en_US,`.
  **L617 CN**: 继续一个多行参数列表、初始化器或聚合项：`"COMMON block /%s/ may not have the member '%s' whose derived type '%s' has a component '%s' that is ALLOCATABLE or has default initialization"_err_en_US,`。
- **L618 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol.name(), ref->name(), derived->name(),`.
  **L618 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol.name(), ref->name(), derived->name(),`。
- **L619 EN**: Comment explains nearby logic, intent, or metadata: `componentPath),`.
  **L619 CN**: 注释说明附近代码的逻辑、意图或元数据：`componentPath),`。
- **L620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `derived->typeSymbol()),`.
  **L620 CN**: 继续一个多行参数列表、初始化器或聚合项：`derived->typeSymbol()),`。
- **L621 EN**: Comment explains nearby logic, intent, or metadata: `ref);`.
  **L621 CN**: 注释说明附近代码的逻辑、意图或元数据：`ref);`。
- **L622 EN**: Closes the current lexical scope or compound statement.
  **L622 CN**: 结束当前词法作用域或复合语句块。
- **L623 EN**: Closes the current lexical scope or compound statement.
  **L623 CN**: 结束当前词法作用域或复合语句块。
- **L624 EN**: Closes the current lexical scope or compound statement.
  **L624 CN**: 结束当前词法作用域或复合语句块。

### Lines 625-648

````cpp
  }
}

// C859, C860
void CheckHelper::CheckExplicitSave(const Symbol &symbol) {
  const Symbol &ultimate{symbol.GetUltimate()};
  if (ultimate.test(Symbol::Flag::InDataStmt)) {
    // checked elsewhere
  } else if (symbol.has<UseDetails>()) {
    messages_.Say(
        "The USE-associated name '%s' may not have an explicit SAVE attribute"_err_en_US,
        symbol.name());
  } else if (IsDummy(ultimate)) {
    messages_.Say(
        "The dummy argument '%s' may not have an explicit SAVE attribute"_err_en_US,
        symbol.name());
  } else if (IsFunctionResult(ultimate)) {
    messages_.Say(
        "The function result variable '%s' may not have an explicit SAVE attribute"_err_en_US,
        symbol.name());
  } else if (const Symbol * common{FindCommonBlockContaining(ultimate)}) {
    messages_.Say(
        "The entity '%s' in COMMON block /%s/ may not have an explicit SAVE attribute"_err_en_US,
        symbol.name(), common->name());
````
- **L625 EN**: Closes the current lexical scope or compound statement.
  **L625 CN**: 结束当前词法作用域或复合语句块。
- **L626 EN**: Closes the current lexical scope or compound statement.
  **L626 CN**: 结束当前词法作用域或复合语句块。
- **L627 EN**: Blank line separating nearby declarations or logic blocks.
  **L627 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L628 EN**: Comment explains nearby logic, intent, or metadata: `C859, C860`.
  **L628 CN**: 注释说明附近代码的逻辑、意图或元数据：`C859, C860`。
- **L629 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckExplicitSave(const Symbol &symbol) {`.
  **L629 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckExplicitSave(const Symbol &symbol) {`。
- **L630 EN**: Executes a call or declaration centered on `&ultimate{symbol.GetUltimate`.
  **L630 CN**: 执行以 `&ultimate{symbol.GetUltimate` 为核心的调用或声明。
- **L631 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L631 CN**: 开始 `if` 控制流语句并计算其条件。
- **L632 EN**: Comment explains nearby logic, intent, or metadata: `checked elsewhere`.
  **L632 CN**: 注释说明附近代码的逻辑、意图或元数据：`checked elsewhere`。
- **L633 EN**: Transitions from the previous branch into an `else if` condition.
  **L633 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L634 EN**: Continues logic associated with callable symbol `Say`.
  **L634 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The USE-associated name '%s' may not have an explicit SAVE attribute"_err_en_US,`.
  **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The USE-associated name '%s' may not have an explicit SAVE attribute"_err_en_US,`。
- **L636 EN**: Executes a call or declaration centered on `symbol.name`.
  **L636 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L637 EN**: Transitions from the previous branch into an `else if` condition.
  **L637 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L638 EN**: Continues logic associated with callable symbol `Say`.
  **L638 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The dummy argument '%s' may not have an explicit SAVE attribute"_err_en_US,`.
  **L639 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The dummy argument '%s' may not have an explicit SAVE attribute"_err_en_US,`。
- **L640 EN**: Executes a call or declaration centered on `symbol.name`.
  **L640 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L641 EN**: Transitions from the previous branch into an `else if` condition.
  **L641 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L642 EN**: Continues logic associated with callable symbol `Say`.
  **L642 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The function result variable '%s' may not have an explicit SAVE attribute"_err_en_US,`.
  **L643 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The function result variable '%s' may not have an explicit SAVE attribute"_err_en_US,`。
- **L644 EN**: Executes a call or declaration centered on `symbol.name`.
  **L644 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L645 EN**: Transitions from the previous branch into an `else if` condition.
  **L645 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L646 EN**: Continues logic associated with callable symbol `Say`.
  **L646 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The entity '%s' in COMMON block /%s/ may not have an explicit SAVE attribute"_err_en_US,`.
  **L647 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The entity '%s' in COMMON block /%s/ may not have an explicit SAVE attribute"_err_en_US,`。
- **L648 EN**: Executes a call or declaration centered on `symbol.name`.
  **L648 CN**: 执行以 `symbol.name` 为核心的调用或声明。

### Lines 649-672

````cpp
  } else if (IsAutomatic(ultimate)) {
    messages_.Say(
        "The automatic object '%s' may not have an explicit SAVE attribute"_err_en_US,
        symbol.name());
  } else if (!evaluate::IsVariable(ultimate) && !IsProcedurePointer(ultimate)) {
    messages_.Say(
        "The entity '%s' with an explicit SAVE attribute must be a variable, procedure pointer, or COMMON block"_err_en_US,
        symbol.name());
  }
}

void CheckHelper::CheckValue(
    const Symbol &symbol, const DerivedTypeSpec *derived) { // C863 - C865
  if (IsProcedure(symbol)) {
    messages_.Say(
        "VALUE attribute may apply only to a dummy data object"_err_en_US);
    return; // don't pile on
  }
  if (IsAssumedSizeArray(symbol)) {
    messages_.Say(
        "VALUE attribute may not apply to an assumed-size array"_err_en_US);
  }
  if (evaluate::IsCoarray(symbol)) {
    messages_.Say("VALUE attribute may not apply to a coarray"_err_en_US);
````
- **L649 EN**: Transitions from the previous branch into an `else if` condition.
  **L649 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L650 EN**: Continues logic associated with callable symbol `Say`.
  **L650 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The automatic object '%s' may not have an explicit SAVE attribute"_err_en_US,`.
  **L651 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The automatic object '%s' may not have an explicit SAVE attribute"_err_en_US,`。
- **L652 EN**: Executes a call or declaration centered on `symbol.name`.
  **L652 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L653 EN**: Transitions from the previous branch into an `else if` condition.
  **L653 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L654 EN**: Continues logic associated with callable symbol `Say`.
  **L654 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The entity '%s' with an explicit SAVE attribute must be a variable, procedure pointer, or COMMON block"_err_en_US,`.
  **L655 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The entity '%s' with an explicit SAVE attribute must be a variable, procedure pointer, or COMMON block"_err_en_US,`。
- **L656 EN**: Executes a call or declaration centered on `symbol.name`.
  **L656 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L657 EN**: Closes the current lexical scope or compound statement.
  **L657 CN**: 结束当前词法作用域或复合语句块。
- **L658 EN**: Closes the current lexical scope or compound statement.
  **L658 CN**: 结束当前词法作用域或复合语句块。
- **L659 EN**: Blank line separating nearby declarations or logic blocks.
  **L659 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L660 EN**: Continues logic associated with callable symbol `CheckValue`.
  **L660 CN**: 继续与可调用符号 `CheckValue` 相关的逻辑。
- **L661 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, const DerivedTypeSpec *derived) { // C863 - C865`.
  **L661 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, const DerivedTypeSpec *derived) { // C863 - C865`。
- **L662 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L662 CN**: 开始 `if` 控制流语句并计算其条件。
- **L663 EN**: Continues logic associated with callable symbol `Say`.
  **L663 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L664 EN**: Executes a standalone statement or declaration: `"VALUE attribute may apply only to a dummy data object"_err_en_US);`.
  **L664 CN**: 执行一条独立语句或声明：`"VALUE attribute may apply only to a dummy data object"_err_en_US);`。
- **L665 EN**: Returns from the current function with `; // don't pile on`.
  **L665 CN**: 以 `; // don't pile on` 从当前函数返回。
- **L666 EN**: Closes the current lexical scope or compound statement.
  **L666 CN**: 结束当前词法作用域或复合语句块。
- **L667 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L667 CN**: 开始 `if` 控制流语句并计算其条件。
- **L668 EN**: Continues logic associated with callable symbol `Say`.
  **L668 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L669 EN**: Executes a standalone statement or declaration: `"VALUE attribute may not apply to an assumed-size array"_err_en_US);`.
  **L669 CN**: 执行一条独立语句或声明：`"VALUE attribute may not apply to an assumed-size array"_err_en_US);`。
- **L670 EN**: Closes the current lexical scope or compound statement.
  **L670 CN**: 结束当前词法作用域或复合语句块。
- **L671 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L671 CN**: 开始 `if` 控制流语句并计算其条件。
- **L672 EN**: Executes a call or declaration centered on `messages_.Say`.
  **L672 CN**: 执行以 `messages_.Say` 为核心的调用或声明。

### Lines 673-696

````cpp
  }
  if (IsAllocatable(symbol)) {
    messages_.Say("VALUE attribute may not apply to an ALLOCATABLE"_err_en_US);
  } else if (IsPointer(symbol)) {
    messages_.Say("VALUE attribute may not apply to a POINTER"_err_en_US);
  }
  if (IsIntentInOut(symbol)) {
    messages_.Say(
        "VALUE attribute may not apply to an INTENT(IN OUT) argument"_err_en_US);
  } else if (IsIntentOut(symbol)) {
    messages_.Say(
        "VALUE attribute may not apply to an INTENT(OUT) argument"_err_en_US);
  }
  if (symbol.attrs().test(Attr::VOLATILE)) {
    messages_.Say("VALUE attribute may not apply to a VOLATILE"_err_en_US);
  }
  if (innermostSymbol_ && IsBindCProcedure(*innermostSymbol_)) {
    if (IsOptional(symbol)) {
      messages_.Say(
          "VALUE attribute may not apply to an OPTIONAL in a BIND(C) procedure"_err_en_US);
    }
    if (symbol.Rank() > 0) {
      messages_.Say(
          "VALUE attribute may not apply to an array in a BIND(C) procedure"_err_en_US);
````
- **L673 EN**: Closes the current lexical scope or compound statement.
  **L673 CN**: 结束当前词法作用域或复合语句块。
- **L674 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L674 CN**: 开始 `if` 控制流语句并计算其条件。
- **L675 EN**: Executes a call or declaration centered on `messages_.Say`.
  **L675 CN**: 执行以 `messages_.Say` 为核心的调用或声明。
- **L676 EN**: Transitions from the previous branch into an `else if` condition.
  **L676 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L677 EN**: Executes a call or declaration centered on `messages_.Say`.
  **L677 CN**: 执行以 `messages_.Say` 为核心的调用或声明。
- **L678 EN**: Closes the current lexical scope or compound statement.
  **L678 CN**: 结束当前词法作用域或复合语句块。
- **L679 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L679 CN**: 开始 `if` 控制流语句并计算其条件。
- **L680 EN**: Continues logic associated with callable symbol `Say`.
  **L680 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L681 EN**: Executes a call or declaration centered on `INTENT`.
  **L681 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L682 EN**: Transitions from the previous branch into an `else if` condition.
  **L682 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L683 EN**: Continues logic associated with callable symbol `Say`.
  **L683 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L684 EN**: Executes a call or declaration centered on `INTENT`.
  **L684 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L685 EN**: Closes the current lexical scope or compound statement.
  **L685 CN**: 结束当前词法作用域或复合语句块。
- **L686 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L686 CN**: 开始 `if` 控制流语句并计算其条件。
- **L687 EN**: Executes a call or declaration centered on `messages_.Say`.
  **L687 CN**: 执行以 `messages_.Say` 为核心的调用或声明。
- **L688 EN**: Closes the current lexical scope or compound statement.
  **L688 CN**: 结束当前词法作用域或复合语句块。
- **L689 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L689 CN**: 开始 `if` 控制流语句并计算其条件。
- **L690 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L690 CN**: 开始 `if` 控制流语句并计算其条件。
- **L691 EN**: Continues logic associated with callable symbol `Say`.
  **L691 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L692 EN**: Executes a call or declaration centered on `BIND`.
  **L692 CN**: 执行以 `BIND` 为核心的调用或声明。
- **L693 EN**: Closes the current lexical scope or compound statement.
  **L693 CN**: 结束当前词法作用域或复合语句块。
- **L694 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L694 CN**: 开始 `if` 控制流语句并计算其条件。
- **L695 EN**: Continues logic associated with callable symbol `Say`.
  **L695 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L696 EN**: Executes a call or declaration centered on `BIND`.
  **L696 CN**: 执行以 `BIND` 为核心的调用或声明。

### Lines 697-720

````cpp
    }
  }
  if (derived) {
    if (FindCoarrayUltimateComponent(*derived)) {
      messages_.Say(
          "VALUE attribute may not apply to a type with a coarray ultimate component"_err_en_US);
    }
  }
  if (IsAssumedRank(symbol)) {
    messages_.Say(
        "VALUE attribute may not apply to an assumed-rank array"_err_en_US);
  }
  if (IsAssumedLengthCharacter(symbol)) {
    // F'2008 feature not widely implemented
    Warn(common::UsageWarning::Portability,
        "VALUE attribute on assumed-length CHARACTER may not be portable"_port_en_US);
  }
}

void CheckHelper::CheckAssumedTypeEntity( // C709
    const Symbol &symbol, const ObjectEntityDetails &details) {
  if (const DeclTypeSpec *type{symbol.GetType()};
      type && type->category() == DeclTypeSpec::TypeStar) {
    if (!IsDummy(symbol)) {
````
- **L697 EN**: Closes the current lexical scope or compound statement.
  **L697 CN**: 结束当前词法作用域或复合语句块。
- **L698 EN**: Closes the current lexical scope or compound statement.
  **L698 CN**: 结束当前词法作用域或复合语句块。
- **L699 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L699 CN**: 开始 `if` 控制流语句并计算其条件。
- **L700 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L700 CN**: 开始 `if` 控制流语句并计算其条件。
- **L701 EN**: Continues logic associated with callable symbol `Say`.
  **L701 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L702 EN**: Executes a standalone statement or declaration: `"VALUE attribute may not apply to a type with a coarray ultimate component"_err_en_US);`.
  **L702 CN**: 执行一条独立语句或声明：`"VALUE attribute may not apply to a type with a coarray ultimate component"_err_en_US);`。
- **L703 EN**: Closes the current lexical scope or compound statement.
  **L703 CN**: 结束当前词法作用域或复合语句块。
- **L704 EN**: Closes the current lexical scope or compound statement.
  **L704 CN**: 结束当前词法作用域或复合语句块。
- **L705 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L705 CN**: 开始 `if` 控制流语句并计算其条件。
- **L706 EN**: Continues logic associated with callable symbol `Say`.
  **L706 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L707 EN**: Executes a standalone statement or declaration: `"VALUE attribute may not apply to an assumed-rank array"_err_en_US);`.
  **L707 CN**: 执行一条独立语句或声明：`"VALUE attribute may not apply to an assumed-rank array"_err_en_US);`。
- **L708 EN**: Closes the current lexical scope or compound statement.
  **L708 CN**: 结束当前词法作用域或复合语句块。
- **L709 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L709 CN**: 开始 `if` 控制流语句并计算其条件。
- **L710 EN**: Comment explains nearby logic, intent, or metadata: `F'2008 feature not widely implemented`.
  **L710 CN**: 注释说明附近代码的逻辑、意图或元数据：`F'2008 feature not widely implemented`。
- **L711 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::Portability,`.
  **L711 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::Portability,`。
- **L712 EN**: Executes a standalone statement or declaration: `"VALUE attribute on assumed-length CHARACTER may not be portable"_port_en_US);`.
  **L712 CN**: 执行一条独立语句或声明：`"VALUE attribute on assumed-length CHARACTER may not be portable"_port_en_US);`。
- **L713 EN**: Closes the current lexical scope or compound statement.
  **L713 CN**: 结束当前词法作用域或复合语句块。
- **L714 EN**: Closes the current lexical scope or compound statement.
  **L714 CN**: 结束当前词法作用域或复合语句块。
- **L715 EN**: Blank line separating nearby declarations or logic blocks.
  **L715 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L716 EN**: Continues logic associated with callable symbol `CheckAssumedTypeEntity`.
  **L716 CN**: 继续与可调用符号 `CheckAssumedTypeEntity` 相关的逻辑。
- **L717 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, const ObjectEntityDetails &details) {`.
  **L717 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, const ObjectEntityDetails &details) {`。
- **L718 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L718 CN**: 开始 `if` 控制流语句并计算其条件。
- **L719 EN**: Starts a function, method, lambda, or structured scope: `type && type->category() == DeclTypeSpec::TypeStar) {`.
  **L719 CN**: 开始一个函数、方法、lambda 或结构化作用域：`type && type->category() == DeclTypeSpec::TypeStar) {`。
- **L720 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L720 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 721-744

````cpp
      messages_.Say(
          "Assumed-type entity '%s' must be a dummy argument"_err_en_US,
          symbol.name());
    } else {
      if (symbol.attrs().test(Attr::ALLOCATABLE)) {
        messages_.Say("Assumed-type argument '%s' cannot have the ALLOCATABLE"
                      " attribute"_err_en_US,
            symbol.name());
      }
      if (symbol.attrs().test(Attr::POINTER)) {
        messages_.Say("Assumed-type argument '%s' cannot have the POINTER"
                      " attribute"_err_en_US,
            symbol.name());
      }
      if (symbol.attrs().test(Attr::VALUE)) {
        messages_.Say("Assumed-type argument '%s' cannot have the VALUE"
                      " attribute"_err_en_US,
            symbol.name());
      }
      if (symbol.attrs().test(Attr::INTENT_OUT)) {
        messages_.Say(
            "Assumed-type argument '%s' cannot be INTENT(OUT)"_err_en_US,
            symbol.name());
      }
````
- **L721 EN**: Continues logic associated with callable symbol `Say`.
  **L721 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L722 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-type entity '%s' must be a dummy argument"_err_en_US,`.
  **L722 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-type entity '%s' must be a dummy argument"_err_en_US,`。
- **L723 EN**: Executes a call or declaration centered on `symbol.name`.
  **L723 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L724 EN**: Transitions from the previous branch into the alternative path.
  **L724 CN**: 从前一个分支过渡到备选路径。
- **L725 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L725 CN**: 开始 `if` 控制流语句并计算其条件。
- **L726 EN**: Continues logic associated with callable symbol `Say`.
  **L726 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L727 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" attribute"_err_en_US,`.
  **L727 CN**: 继续一个多行参数列表、初始化器或聚合项：`" attribute"_err_en_US,`。
- **L728 EN**: Executes a call or declaration centered on `symbol.name`.
  **L728 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L729 EN**: Closes the current lexical scope or compound statement.
  **L729 CN**: 结束当前词法作用域或复合语句块。
- **L730 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L730 CN**: 开始 `if` 控制流语句并计算其条件。
- **L731 EN**: Continues logic associated with callable symbol `Say`.
  **L731 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L732 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" attribute"_err_en_US,`.
  **L732 CN**: 继续一个多行参数列表、初始化器或聚合项：`" attribute"_err_en_US,`。
- **L733 EN**: Executes a call or declaration centered on `symbol.name`.
  **L733 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L734 EN**: Closes the current lexical scope or compound statement.
  **L734 CN**: 结束当前词法作用域或复合语句块。
- **L735 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L735 CN**: 开始 `if` 控制流语句并计算其条件。
- **L736 EN**: Continues logic associated with callable symbol `Say`.
  **L736 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" attribute"_err_en_US,`.
  **L737 CN**: 继续一个多行参数列表、初始化器或聚合项：`" attribute"_err_en_US,`。
- **L738 EN**: Executes a call or declaration centered on `symbol.name`.
  **L738 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L739 EN**: Closes the current lexical scope or compound statement.
  **L739 CN**: 结束当前词法作用域或复合语句块。
- **L740 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L740 CN**: 开始 `if` 控制流语句并计算其条件。
- **L741 EN**: Continues logic associated with callable symbol `Say`.
  **L741 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-type argument '%s' cannot be INTENT(OUT)"_err_en_US,`.
  **L742 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-type argument '%s' cannot be INTENT(OUT)"_err_en_US,`。
- **L743 EN**: Executes a call or declaration centered on `symbol.name`.
  **L743 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L744 EN**: Closes the current lexical scope or compound statement.
  **L744 CN**: 结束当前词法作用域或复合语句块。

### Lines 745-768

````cpp
      if (evaluate::IsCoarray(symbol)) {
        messages_.Say(
            "Assumed-type argument '%s' cannot be a coarray"_err_en_US,
            symbol.name());
      }
      if (details.IsArray() && details.shape().IsExplicitShape()) {
        messages_.Say("Assumed-type array argument '%s' must be assumed shape,"
                      " assumed size, or assumed rank"_err_en_US,
            symbol.name());
      }
    }
  }
}

void CheckHelper::CheckObjectEntity(
    const Symbol &symbol, const ObjectEntityDetails &details) {
  CheckSymbolType(symbol);
  CheckArraySpec(symbol, details.shape());
  CheckConflicting(symbol, Attr::ALLOCATABLE, Attr::PARAMETER);
  CheckConflicting(symbol, Attr::ASYNCHRONOUS, Attr::PARAMETER);
  CheckConflicting(symbol, Attr::SAVE, Attr::PARAMETER);
  CheckConflicting(symbol, Attr::TARGET, Attr::PARAMETER);
  CheckConflicting(symbol, Attr::VOLATILE, Attr::PARAMETER);
  Check(details.shape());
````
- **L745 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L745 CN**: 开始 `if` 控制流语句并计算其条件。
- **L746 EN**: Continues logic associated with callable symbol `Say`.
  **L746 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L747 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-type argument '%s' cannot be a coarray"_err_en_US,`.
  **L747 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-type argument '%s' cannot be a coarray"_err_en_US,`。
- **L748 EN**: Executes a call or declaration centered on `symbol.name`.
  **L748 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L749 EN**: Closes the current lexical scope or compound statement.
  **L749 CN**: 结束当前词法作用域或复合语句块。
- **L750 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L750 CN**: 开始 `if` 控制流语句并计算其条件。
- **L751 EN**: Continues logic associated with callable symbol `Say`.
  **L751 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L752 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" assumed size, or assumed rank"_err_en_US,`.
  **L752 CN**: 继续一个多行参数列表、初始化器或聚合项：`" assumed size, or assumed rank"_err_en_US,`。
- **L753 EN**: Executes a call or declaration centered on `symbol.name`.
  **L753 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L754 EN**: Closes the current lexical scope or compound statement.
  **L754 CN**: 结束当前词法作用域或复合语句块。
- **L755 EN**: Closes the current lexical scope or compound statement.
  **L755 CN**: 结束当前词法作用域或复合语句块。
- **L756 EN**: Closes the current lexical scope or compound statement.
  **L756 CN**: 结束当前词法作用域或复合语句块。
- **L757 EN**: Closes the current lexical scope or compound statement.
  **L757 CN**: 结束当前词法作用域或复合语句块。
- **L758 EN**: Blank line separating nearby declarations or logic blocks.
  **L758 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L759 EN**: Continues logic associated with callable symbol `CheckObjectEntity`.
  **L759 CN**: 继续与可调用符号 `CheckObjectEntity` 相关的逻辑。
- **L760 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, const ObjectEntityDetails &details) {`.
  **L760 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, const ObjectEntityDetails &details) {`。
- **L761 EN**: Executes a call or declaration centered on `CheckSymbolType`.
  **L761 CN**: 执行以 `CheckSymbolType` 为核心的调用或声明。
- **L762 EN**: Executes a call or declaration centered on `CheckArraySpec`.
  **L762 CN**: 执行以 `CheckArraySpec` 为核心的调用或声明。
- **L763 EN**: Executes a call or declaration centered on `CheckConflicting`.
  **L763 CN**: 执行以 `CheckConflicting` 为核心的调用或声明。
- **L764 EN**: Executes a call or declaration centered on `CheckConflicting`.
  **L764 CN**: 执行以 `CheckConflicting` 为核心的调用或声明。
- **L765 EN**: Executes a call or declaration centered on `CheckConflicting`.
  **L765 CN**: 执行以 `CheckConflicting` 为核心的调用或声明。
- **L766 EN**: Executes a call or declaration centered on `CheckConflicting`.
  **L766 CN**: 执行以 `CheckConflicting` 为核心的调用或声明。
- **L767 EN**: Executes a call or declaration centered on `CheckConflicting`.
  **L767 CN**: 执行以 `CheckConflicting` 为核心的调用或声明。
- **L768 EN**: Executes a call or declaration centered on `Check`.
  **L768 CN**: 执行以 `Check` 为核心的调用或声明。

### Lines 769-792

````cpp
  Check(details.coshape());
  if (details.shape().Rank() > common::maxRank) {
    messages_.Say(
        "'%s' has rank %d, which is greater than the maximum supported rank %d"_err_en_US,
        symbol.name(), details.shape().Rank(), common::maxRank);
  } else if (details.shape().Rank() + details.coshape().Rank() >
      common::maxRank) {
    messages_.Say(
        "'%s' has rank %d and corank %d, whose sum is greater than the maximum supported rank %d"_err_en_US,
        symbol.name(), details.shape().Rank(), details.coshape().Rank(),
        common::maxRank);
  }
  CheckAssumedTypeEntity(symbol, details);
  WarnMissingFinal(symbol);
  const DeclTypeSpec *type{details.type()};
  const DerivedTypeSpec *derived{type ? type->AsDerived() : nullptr};
  bool isComponent{symbol.owner().IsDerivedType()};
  const Symbol *commonBlock{FindCommonBlockContaining(symbol)};
  bool isLocalVariable{!commonBlock && !isComponent && !details.isDummy() &&
      symbol.owner().kind() != Scope::Kind::OtherConstruct};
  if (int corank{evaluate::GetCorank(symbol)}; corank > 0) { // it's a coarray
    bool isDeferredCoshape{details.coshape().CanBeDeferredShape()};
    if (IsAllocatable(symbol)) {
      if (!isDeferredCoshape) { // C827
````
- **L769 EN**: Executes a call or declaration centered on `Check`.
  **L769 CN**: 执行以 `Check` 为核心的调用或声明。
- **L770 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L770 CN**: 开始 `if` 控制流语句并计算其条件。
- **L771 EN**: Continues logic associated with callable symbol `Say`.
  **L771 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L772 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' has rank %d, which is greater than the maximum supported rank %d"_err_en_US,`.
  **L772 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' has rank %d, which is greater than the maximum supported rank %d"_err_en_US,`。
- **L773 EN**: Executes a call or declaration centered on `symbol.name`.
  **L773 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L774 EN**: Transitions from the previous branch into an `else if` condition.
  **L774 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L775 EN**: Continues the surrounding expression or declaration: `common::maxRank) {`.
  **L775 CN**: 继续构造周围的表达式或声明：`common::maxRank) {`。
- **L776 EN**: Continues logic associated with callable symbol `Say`.
  **L776 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L777 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' has rank %d and corank %d, whose sum is greater than the maximum supported rank %d"_err_en_US,`.
  **L777 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' has rank %d and corank %d, whose sum is greater than the maximum supported rank %d"_err_en_US,`。
- **L778 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol.name(), details.shape().Rank(), details.coshape().Rank(),`.
  **L778 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol.name(), details.shape().Rank(), details.coshape().Rank(),`。
- **L779 EN**: Executes a standalone statement or declaration: `common::maxRank);`.
  **L779 CN**: 执行一条独立语句或声明：`common::maxRank);`。
- **L780 EN**: Closes the current lexical scope or compound statement.
  **L780 CN**: 结束当前词法作用域或复合语句块。
- **L781 EN**: Executes a call or declaration centered on `CheckAssumedTypeEntity`.
  **L781 CN**: 执行以 `CheckAssumedTypeEntity` 为核心的调用或声明。
- **L782 EN**: Executes a call or declaration centered on `WarnMissingFinal`.
  **L782 CN**: 执行以 `WarnMissingFinal` 为核心的调用或声明。
- **L783 EN**: Executes a call or declaration centered on `*type{details.type`.
  **L783 CN**: 执行以 `*type{details.type` 为核心的调用或声明。
- **L784 EN**: Executes a call or declaration centered on `type->AsDerived`.
  **L784 CN**: 执行以 `type->AsDerived` 为核心的调用或声明。
- **L785 EN**: Executes a call or declaration centered on `isComponent{symbol.owner`.
  **L785 CN**: 执行以 `isComponent{symbol.owner` 为核心的调用或声明。
- **L786 EN**: Executes a call or declaration centered on `*commonBlock{FindCommonBlockContaining`.
  **L786 CN**: 执行以 `*commonBlock{FindCommonBlockContaining` 为核心的调用或声明。
- **L787 EN**: Continues logic associated with callable symbol `isDummy`.
  **L787 CN**: 继续与可调用符号 `isDummy` 相关的逻辑。
- **L788 EN**: Executes a call or declaration centered on `symbol.owner`.
  **L788 CN**: 执行以 `symbol.owner` 为核心的调用或声明。
- **L789 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L789 CN**: 开始 `if` 控制流语句并计算其条件。
- **L790 EN**: Executes a call or declaration centered on `isDeferredCoshape{details.coshape`.
  **L790 CN**: 执行以 `isDeferredCoshape{details.coshape` 为核心的调用或声明。
- **L791 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L791 CN**: 开始 `if` 控制流语句并计算其条件。
- **L792 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L792 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 793-816

````cpp
        messages_.Say("'%s' is an ALLOCATABLE coarray and must have a deferred"
                      " coshape"_err_en_US,
            symbol.name());
      }
    } else if (isComponent) { // C746
      std::string deferredMsg{
          isDeferredCoshape ? "" : " and have a deferred coshape"};
      messages_.Say("Component '%s' is a coarray and must have the ALLOCATABLE"
                    " attribute%s"_err_en_US,
          symbol.name(), deferredMsg);
    } else {
      if (!details.coshape().CanBeAssumedSize()) { // C828
        messages_.Say(
            "'%s' is a non-ALLOCATABLE coarray and must have an explicit coshape"_err_en_US,
            symbol.name());
      }
    }
    if (IsBadCoarrayType(derived)) { // C747 & C824
      messages_.Say(
          "Coarray '%s' may not have type TEAM_TYPE, C_PTR, or C_FUNPTR"_err_en_US,
          symbol.name());
    }
    if (IsAssumedRank(symbol)) {
      messages_.Say("Coarray '%s' may not be an assumed-rank array"_err_en_US,
````
- **L793 EN**: Continues logic associated with callable symbol `Say`.
  **L793 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L794 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" coshape"_err_en_US,`.
  **L794 CN**: 继续一个多行参数列表、初始化器或聚合项：`" coshape"_err_en_US,`。
- **L795 EN**: Executes a call or declaration centered on `symbol.name`.
  **L795 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L796 EN**: Closes the current lexical scope or compound statement.
  **L796 CN**: 结束当前词法作用域或复合语句块。
- **L797 EN**: Transitions from the previous branch into an `else if` condition.
  **L797 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L798 EN**: Continues the surrounding expression or declaration: `std::string deferredMsg{`.
  **L798 CN**: 继续构造周围的表达式或声明：`std::string deferredMsg{`。
- **L799 EN**: Executes a standalone statement or declaration: `isDeferredCoshape ? "" : " and have a deferred coshape"};`.
  **L799 CN**: 执行一条独立语句或声明：`isDeferredCoshape ? "" : " and have a deferred coshape"};`。
- **L800 EN**: Continues logic associated with callable symbol `Say`.
  **L800 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" attribute%s"_err_en_US,`.
  **L801 CN**: 继续一个多行参数列表、初始化器或聚合项：`" attribute%s"_err_en_US,`。
- **L802 EN**: Executes a call or declaration centered on `symbol.name`.
  **L802 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L803 EN**: Transitions from the previous branch into the alternative path.
  **L803 CN**: 从前一个分支过渡到备选路径。
- **L804 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L804 CN**: 开始 `if` 控制流语句并计算其条件。
- **L805 EN**: Continues logic associated with callable symbol `Say`.
  **L805 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L806 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' is a non-ALLOCATABLE coarray and must have an explicit coshape"_err_en_US,`.
  **L806 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' is a non-ALLOCATABLE coarray and must have an explicit coshape"_err_en_US,`。
- **L807 EN**: Executes a call or declaration centered on `symbol.name`.
  **L807 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L808 EN**: Closes the current lexical scope or compound statement.
  **L808 CN**: 结束当前词法作用域或复合语句块。
- **L809 EN**: Closes the current lexical scope or compound statement.
  **L809 CN**: 结束当前词法作用域或复合语句块。
- **L810 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L810 CN**: 开始 `if` 控制流语句并计算其条件。
- **L811 EN**: Continues logic associated with callable symbol `Say`.
  **L811 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Coarray '%s' may not have type TEAM_TYPE, C_PTR, or C_FUNPTR"_err_en_US,`.
  **L812 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Coarray '%s' may not have type TEAM_TYPE, C_PTR, or C_FUNPTR"_err_en_US,`。
- **L813 EN**: Executes a call or declaration centered on `symbol.name`.
  **L813 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L814 EN**: Closes the current lexical scope or compound statement.
  **L814 CN**: 结束当前词法作用域或复合语句块。
- **L815 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L815 CN**: 开始 `if` 控制流语句并计算其条件。
- **L816 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say("Coarray '%s' may not be an assumed-rank array"_err_en_US,`.
  **L816 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say("Coarray '%s' may not be an assumed-rank array"_err_en_US,`。

### Lines 817-840

````cpp
          symbol.name());
    }
    if (IsNamedConstant(symbol)) {
      messages_.Say(
          "Coarray '%s' may not be a named constant"_err_en_US, symbol.name());
    }
    if (IsFunctionResult(symbol)) {
      messages_.Say("Function result may not be a coarray"_err_en_US);
    } else if (commonBlock) {
      messages_.Say("Coarray '%s' may not be in COMMON block '/%s/'"_err_en_US,
          symbol.name(), commonBlock->name());
    } else if (isLocalVariable && !IsAllocatableOrPointer(symbol) &&
        !IsSaved(symbol)) {
      messages_.Say(
          "Local coarray must have the SAVE or ALLOCATABLE attribute"_err_en_US);
    }
    for (int j{0}; j < corank; ++j) {
      if (auto lcbv{evaluate::ToInt64(evaluate::Fold(
              context().foldingContext(), evaluate::GetLCOBOUND(symbol, j)))}) {
        if (auto ucbv{
                evaluate::ToInt64(evaluate::Fold(context().foldingContext(),
                    evaluate::GetUCOBOUND(symbol, j)))}) {
          if (ucbv < lcbv) {
            messages_.Say(
````
- **L817 EN**: Executes a call or declaration centered on `symbol.name`.
  **L817 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L818 EN**: Closes the current lexical scope or compound statement.
  **L818 CN**: 结束当前词法作用域或复合语句块。
- **L819 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L819 CN**: 开始 `if` 控制流语句并计算其条件。
- **L820 EN**: Continues logic associated with callable symbol `Say`.
  **L820 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L821 EN**: Executes a call or declaration centered on `symbol.name`.
  **L821 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L822 EN**: Closes the current lexical scope or compound statement.
  **L822 CN**: 结束当前词法作用域或复合语句块。
- **L823 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L823 CN**: 开始 `if` 控制流语句并计算其条件。
- **L824 EN**: Executes a call or declaration centered on `messages_.Say`.
  **L824 CN**: 执行以 `messages_.Say` 为核心的调用或声明。
- **L825 EN**: Transitions from the previous branch into an `else if` condition.
  **L825 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L826 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say("Coarray '%s' may not be in COMMON block '/%s/'"_err_en_US,`.
  **L826 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say("Coarray '%s' may not be in COMMON block '/%s/'"_err_en_US,`。
- **L827 EN**: Executes a call or declaration centered on `symbol.name`.
  **L827 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L828 EN**: Transitions from the previous branch into an `else if` condition.
  **L828 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L829 EN**: Starts a function, method, lambda, or structured scope: `!IsSaved(symbol)) {`.
  **L829 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!IsSaved(symbol)) {`。
- **L830 EN**: Continues logic associated with callable symbol `Say`.
  **L830 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L831 EN**: Executes a standalone statement or declaration: `"Local coarray must have the SAVE or ALLOCATABLE attribute"_err_en_US);`.
  **L831 CN**: 执行一条独立语句或声明：`"Local coarray must have the SAVE or ALLOCATABLE attribute"_err_en_US);`。
- **L832 EN**: Closes the current lexical scope or compound statement.
  **L832 CN**: 结束当前词法作用域或复合语句块。
- **L833 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L833 CN**: 开始 `for` 控制流语句并计算其条件。
- **L834 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L834 CN**: 开始 `if` 控制流语句并计算其条件。
- **L835 EN**: Starts a function, method, lambda, or structured scope: `context().foldingContext(), evaluate::GetLCOBOUND(symbol, j)))}) {`.
  **L835 CN**: 开始一个函数、方法、lambda 或结构化作用域：`context().foldingContext(), evaluate::GetLCOBOUND(symbol, j)))}) {`。
- **L836 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L836 CN**: 开始 `if` 控制流语句并计算其条件。
- **L837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::ToInt64(evaluate::Fold(context().foldingContext(),`.
  **L837 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::ToInt64(evaluate::Fold(context().foldingContext(),`。
- **L838 EN**: Starts a function, method, lambda, or structured scope: `evaluate::GetUCOBOUND(symbol, j)))}) {`.
  **L838 CN**: 开始一个函数、方法、lambda 或结构化作用域：`evaluate::GetUCOBOUND(symbol, j)))}) {`。
- **L839 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L839 CN**: 开始 `if` 控制流语句并计算其条件。
- **L840 EN**: Continues logic associated with callable symbol `Say`.
  **L840 CN**: 继续与可调用符号 `Say` 相关的逻辑。

### Lines 841-864

````cpp
                "Cobounds %jd:%jd of codimension %d produce an empty coarray"_err_en_US,
                std::intmax_t{*lcbv}, std::intmax_t{*ucbv}, j + 1);
          }
        }
      }
    }
  } else { // not a coarray
    if (!isComponent && !IsPointer(symbol) && derived) {
      if (IsEventTypeOrLockType(derived)) {
        messages_.Say(
            "Variable '%s' with EVENT_TYPE or LOCK_TYPE must be a coarray"_err_en_US,
            symbol.name());
      } else if (auto component{FindEventOrLockPotentialComponent(
                     *derived, /*ignoreCoarrays=*/true)}) {
        messages_.Say(
            "Variable '%s' with EVENT_TYPE or LOCK_TYPE potential component '%s' must be a coarray"_err_en_US,
            symbol.name(), component.BuildResultDesignatorName());
      } else if (IsNotifyType(derived)) { // C1612
        messages_.Say(
            "Variable '%s' with NOTIFY_TYPE must be a coarray"_err_en_US,
            symbol.name());
      } else if (auto component{FindNotifyPotentialComponent( // C1611
                     *derived, /*ignoreCoarrays=*/true)}) {
        messages_.Say(
````
- **L841 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Cobounds %jd:%jd of codimension %d produce an empty coarray"_err_en_US,`.
  **L841 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Cobounds %jd:%jd of codimension %d produce an empty coarray"_err_en_US,`。
- **L842 EN**: Executes a standalone statement or declaration: `std::intmax_t{*lcbv}, std::intmax_t{*ucbv}, j + 1);`.
  **L842 CN**: 执行一条独立语句或声明：`std::intmax_t{*lcbv}, std::intmax_t{*ucbv}, j + 1);`。
- **L843 EN**: Closes the current lexical scope or compound statement.
  **L843 CN**: 结束当前词法作用域或复合语句块。
- **L844 EN**: Closes the current lexical scope or compound statement.
  **L844 CN**: 结束当前词法作用域或复合语句块。
- **L845 EN**: Closes the current lexical scope or compound statement.
  **L845 CN**: 结束当前词法作用域或复合语句块。
- **L846 EN**: Closes the current lexical scope or compound statement.
  **L846 CN**: 结束当前词法作用域或复合语句块。
- **L847 EN**: Transitions from the previous branch into the alternative path.
  **L847 CN**: 从前一个分支过渡到备选路径。
- **L848 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L848 CN**: 开始 `if` 控制流语句并计算其条件。
- **L849 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L849 CN**: 开始 `if` 控制流语句并计算其条件。
- **L850 EN**: Continues logic associated with callable symbol `Say`.
  **L850 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L851 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Variable '%s' with EVENT_TYPE or LOCK_TYPE must be a coarray"_err_en_US,`.
  **L851 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Variable '%s' with EVENT_TYPE or LOCK_TYPE must be a coarray"_err_en_US,`。
- **L852 EN**: Executes a call or declaration centered on `symbol.name`.
  **L852 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L853 EN**: Transitions from the previous branch into an `else if` condition.
  **L853 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L854 EN**: Comment explains nearby logic, intent, or metadata: `derived, /*ignoreCoarrays=*/true)}) {`.
  **L854 CN**: 注释说明附近代码的逻辑、意图或元数据：`derived, /*ignoreCoarrays=*/true)}) {`。
- **L855 EN**: Continues logic associated with callable symbol `Say`.
  **L855 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Variable '%s' with EVENT_TYPE or LOCK_TYPE potential component '%s' must be a coarray"_err_en_US,`.
  **L856 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Variable '%s' with EVENT_TYPE or LOCK_TYPE potential component '%s' must be a coarray"_err_en_US,`。
- **L857 EN**: Executes a call or declaration centered on `symbol.name`.
  **L857 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L858 EN**: Transitions from the previous branch into an `else if` condition.
  **L858 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L859 EN**: Continues logic associated with callable symbol `Say`.
  **L859 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L860 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Variable '%s' with NOTIFY_TYPE must be a coarray"_err_en_US,`.
  **L860 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Variable '%s' with NOTIFY_TYPE must be a coarray"_err_en_US,`。
- **L861 EN**: Executes a call or declaration centered on `symbol.name`.
  **L861 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L862 EN**: Transitions from the previous branch into an `else if` condition.
  **L862 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L863 EN**: Comment explains nearby logic, intent, or metadata: `derived, /*ignoreCoarrays=*/true)}) {`.
  **L863 CN**: 注释说明附近代码的逻辑、意图或元数据：`derived, /*ignoreCoarrays=*/true)}) {`。
- **L864 EN**: Continues logic associated with callable symbol `Say`.
  **L864 CN**: 继续与可调用符号 `Say` 相关的逻辑。

### Lines 865-888

````cpp
            "Variable '%s' with NOTIFY_TYPE potential component '%s' must be a coarray"_err_en_US,
            symbol.name(), component.BuildResultDesignatorName());
      }
    }
  }
  if (details.isDummy()) {
    if (IsIntentOut(symbol)) {
      // Some of these errors would also be caught by the general check
      // for definability of automatically deallocated local variables,
      // but these messages are more specific.
      if (FindUltimateComponent(symbol, [](const Symbol &x) {
            return evaluate::IsCoarray(x) && IsAllocatable(x);
          })) { // C846
        messages_.Say(
            "An INTENT(OUT) dummy argument may not be, or contain, an ALLOCATABLE coarray"_err_en_US);
      }
      if (IsOrContainsEventOrLockComponent(symbol)) { // C847
        messages_.Say(
            "An INTENT(OUT) dummy argument may not be, or contain, EVENT_TYPE or LOCK_TYPE"_err_en_US);
      }
      if (IsOrContainsNotifyComponent(symbol)) { // C1613
        messages_.Say(
            "An INTENT(OUT) dummy argument may not be, or contain, NOTIFY_TYPE"_err_en_US);
      }
````
- **L865 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Variable '%s' with NOTIFY_TYPE potential component '%s' must be a coarray"_err_en_US,`.
  **L865 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Variable '%s' with NOTIFY_TYPE potential component '%s' must be a coarray"_err_en_US,`。
- **L866 EN**: Executes a call or declaration centered on `symbol.name`.
  **L866 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L867 EN**: Closes the current lexical scope or compound statement.
  **L867 CN**: 结束当前词法作用域或复合语句块。
- **L868 EN**: Closes the current lexical scope or compound statement.
  **L868 CN**: 结束当前词法作用域或复合语句块。
- **L869 EN**: Closes the current lexical scope or compound statement.
  **L869 CN**: 结束当前词法作用域或复合语句块。
- **L870 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L870 CN**: 开始 `if` 控制流语句并计算其条件。
- **L871 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L871 CN**: 开始 `if` 控制流语句并计算其条件。
- **L872 EN**: Comment explains nearby logic, intent, or metadata: `Some of these errors would also be caught by the general check`.
  **L872 CN**: 注释说明附近代码的逻辑、意图或元数据：`Some of these errors would also be caught by the general check`。
- **L873 EN**: Comment explains nearby logic, intent, or metadata: `for definability of automatically deallocated local variables,`.
  **L873 CN**: 注释说明附近代码的逻辑、意图或元数据：`for definability of automatically deallocated local variables,`。
- **L874 EN**: Comment explains nearby logic, intent, or metadata: `but these messages are more specific.`.
  **L874 CN**: 注释说明附近代码的逻辑、意图或元数据：`but these messages are more specific.`。
- **L875 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L875 CN**: 开始 `if` 控制流语句并计算其条件。
- **L876 EN**: Returns from the current function with `evaluate::IsCoarray(x) && IsAllocatable(x)`.
  **L876 CN**: 以 `evaluate::IsCoarray(x) && IsAllocatable(x)` 从当前函数返回。
- **L877 EN**: Continues the surrounding expression or declaration: `})) { // C846`.
  **L877 CN**: 继续构造周围的表达式或声明：`})) { // C846`。
- **L878 EN**: Continues logic associated with callable symbol `Say`.
  **L878 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L879 EN**: Executes a call or declaration centered on `INTENT`.
  **L879 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L880 EN**: Closes the current lexical scope or compound statement.
  **L880 CN**: 结束当前词法作用域或复合语句块。
- **L881 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L881 CN**: 开始 `if` 控制流语句并计算其条件。
- **L882 EN**: Continues logic associated with callable symbol `Say`.
  **L882 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L883 EN**: Executes a call or declaration centered on `INTENT`.
  **L883 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L884 EN**: Closes the current lexical scope or compound statement.
  **L884 CN**: 结束当前词法作用域或复合语句块。
- **L885 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L885 CN**: 开始 `if` 控制流语句并计算其条件。
- **L886 EN**: Continues logic associated with callable symbol `Say`.
  **L886 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L887 EN**: Executes a call or declaration centered on `INTENT`.
  **L887 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L888 EN**: Closes the current lexical scope or compound statement.
  **L888 CN**: 结束当前词法作用域或复合语句块。

### Lines 889-912

````cpp
      if (IsAssumedSizeArray(symbol)) { // C834
        if (type && type->IsPolymorphic()) {
          messages_.Say(
              "An INTENT(OUT) assumed-size dummy argument array may not be polymorphic"_err_en_US);
        }
        if (derived) {
          if (derived->HasDefaultInitialization()) {
            messages_.Say(
                "An INTENT(OUT) assumed-size dummy argument array may not have a derived type with any default component initialization"_err_en_US);
          }
          if (IsFinalizable(*derived)) {
            messages_.Say(
                "An INTENT(OUT) assumed-size dummy argument array may not be finalizable"_err_en_US);
          }
        }
      }
    }
    if (InPure() && !IsStmtFunction(DEREF(innermostSymbol_)) &&
        !IsPointer(symbol) && !IsIntentIn(symbol) &&
        !symbol.attrs().test(Attr::VALUE)) {
      const char *what{InFunction() ? "function" : "subroutine"};
      bool ok{true};
      if (IsIntentOut(symbol)) {
        if (type && type->IsPolymorphic()) { // C1588
````
- **L889 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L889 CN**: 开始 `if` 控制流语句并计算其条件。
- **L890 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L890 CN**: 开始 `if` 控制流语句并计算其条件。
- **L891 EN**: Continues logic associated with callable symbol `Say`.
  **L891 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L892 EN**: Executes a call or declaration centered on `INTENT`.
  **L892 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L893 EN**: Closes the current lexical scope or compound statement.
  **L893 CN**: 结束当前词法作用域或复合语句块。
- **L894 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L894 CN**: 开始 `if` 控制流语句并计算其条件。
- **L895 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L895 CN**: 开始 `if` 控制流语句并计算其条件。
- **L896 EN**: Continues logic associated with callable symbol `Say`.
  **L896 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L897 EN**: Executes a call or declaration centered on `INTENT`.
  **L897 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L898 EN**: Closes the current lexical scope or compound statement.
  **L898 CN**: 结束当前词法作用域或复合语句块。
- **L899 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L899 CN**: 开始 `if` 控制流语句并计算其条件。
- **L900 EN**: Continues logic associated with callable symbol `Say`.
  **L900 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L901 EN**: Executes a call or declaration centered on `INTENT`.
  **L901 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L902 EN**: Closes the current lexical scope or compound statement.
  **L902 CN**: 结束当前词法作用域或复合语句块。
- **L903 EN**: Closes the current lexical scope or compound statement.
  **L903 CN**: 结束当前词法作用域或复合语句块。
- **L904 EN**: Closes the current lexical scope or compound statement.
  **L904 CN**: 结束当前词法作用域或复合语句块。
- **L905 EN**: Closes the current lexical scope or compound statement.
  **L905 CN**: 结束当前词法作用域或复合语句块。
- **L906 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L906 CN**: 开始 `if` 控制流语句并计算其条件。
- **L907 EN**: Continues logic associated with callable symbol `IsPointer`.
  **L907 CN**: 继续与可调用符号 `IsPointer` 相关的逻辑。
- **L908 EN**: Starts a function, method, lambda, or structured scope: `!symbol.attrs().test(Attr::VALUE)) {`.
  **L908 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!symbol.attrs().test(Attr::VALUE)) {`。
- **L909 EN**: Executes a call or declaration centered on `*what{InFunction`.
  **L909 CN**: 执行以 `*what{InFunction` 为核心的调用或声明。
- **L910 EN**: Executes a standalone statement or declaration: `bool ok{true};`.
  **L910 CN**: 执行一条独立语句或声明：`bool ok{true};`。
- **L911 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L911 CN**: 开始 `if` 控制流语句并计算其条件。
- **L912 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L912 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 913-936

````cpp
          messages_.Say(
              "An INTENT(OUT) dummy argument of a pure %s may not be polymorphic"_err_en_US,
              what);
          ok = false;
        } else if (derived) {
          if (FindUltimateComponent(*derived, [](const Symbol &x) {
                const DeclTypeSpec *type{x.GetType()};
                return type && type->IsPolymorphic();
              })) { // C1588
            messages_.Say(
                "An INTENT(OUT) dummy argument of a pure %s may not have a polymorphic ultimate component"_err_en_US,
                what);
            ok = false;
          }
          if (HasImpureFinal(symbol)) { // C1587
            messages_.Say(
                "An INTENT(OUT) dummy argument of a pure %s may not have an impure FINAL subroutine"_err_en_US,
                what);
            ok = false;
          }
        }
      } else if (!IsIntentInOut(symbol)) { // C1586
        messages_.Say(
            "non-POINTER dummy argument of pure %s must have INTENT() or VALUE attribute"_err_en_US,
````
- **L913 EN**: Continues logic associated with callable symbol `Say`.
  **L913 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"An INTENT(OUT) dummy argument of a pure %s may not be polymorphic"_err_en_US,`.
  **L914 CN**: 继续一个多行参数列表、初始化器或聚合项：`"An INTENT(OUT) dummy argument of a pure %s may not be polymorphic"_err_en_US,`。
- **L915 EN**: Executes a standalone statement or declaration: `what);`.
  **L915 CN**: 执行一条独立语句或声明：`what);`。
- **L916 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L916 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L917 EN**: Transitions from the previous branch into an `else if` condition.
  **L917 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L918 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L918 CN**: 开始 `if` 控制流语句并计算其条件。
- **L919 EN**: Executes a call or declaration centered on `*type{x.GetType`.
  **L919 CN**: 执行以 `*type{x.GetType` 为核心的调用或声明。
- **L920 EN**: Returns from the current function with `type && type->IsPolymorphic()`.
  **L920 CN**: 以 `type && type->IsPolymorphic()` 从当前函数返回。
- **L921 EN**: Continues the surrounding expression or declaration: `})) { // C1588`.
  **L921 CN**: 继续构造周围的表达式或声明：`})) { // C1588`。
- **L922 EN**: Continues logic associated with callable symbol `Say`.
  **L922 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L923 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"An INTENT(OUT) dummy argument of a pure %s may not have a polymorphic ultimate component"_err_en_US,`.
  **L923 CN**: 继续一个多行参数列表、初始化器或聚合项：`"An INTENT(OUT) dummy argument of a pure %s may not have a polymorphic ultimate component"_err_en_US,`。
- **L924 EN**: Executes a standalone statement or declaration: `what);`.
  **L924 CN**: 执行一条独立语句或声明：`what);`。
- **L925 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L925 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L926 EN**: Closes the current lexical scope or compound statement.
  **L926 CN**: 结束当前词法作用域或复合语句块。
- **L927 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L927 CN**: 开始 `if` 控制流语句并计算其条件。
- **L928 EN**: Continues logic associated with callable symbol `Say`.
  **L928 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L929 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"An INTENT(OUT) dummy argument of a pure %s may not have an impure FINAL subroutine"_err_en_US,`.
  **L929 CN**: 继续一个多行参数列表、初始化器或聚合项：`"An INTENT(OUT) dummy argument of a pure %s may not have an impure FINAL subroutine"_err_en_US,`。
- **L930 EN**: Executes a standalone statement or declaration: `what);`.
  **L930 CN**: 执行一条独立语句或声明：`what);`。
- **L931 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L931 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L932 EN**: Closes the current lexical scope or compound statement.
  **L932 CN**: 结束当前词法作用域或复合语句块。
- **L933 EN**: Closes the current lexical scope or compound statement.
  **L933 CN**: 结束当前词法作用域或复合语句块。
- **L934 EN**: Transitions from the previous branch into an `else if` condition.
  **L934 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L935 EN**: Continues logic associated with callable symbol `Say`.
  **L935 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L936 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"non-POINTER dummy argument of pure %s must have INTENT() or VALUE attribute"_err_en_US,`.
  **L936 CN**: 继续一个多行参数列表、初始化器或聚合项：`"non-POINTER dummy argument of pure %s must have INTENT() or VALUE attribute"_err_en_US,`。

### Lines 937-960

````cpp
            what);
        ok = false;
      }
      if (ok && InFunction() && !InModuleFile() && !InElemental()) {
        if (context_.IsEnabled(common::LanguageFeature::RelaxedPureDummy)) {
          Warn(common::LanguageFeature::RelaxedPureDummy,
              "non-POINTER dummy argument of pure function should be INTENT(IN) or VALUE"_warn_en_US);
        } else {
          messages_.Say(
              "non-POINTER dummy argument of pure function must be INTENT(IN) or VALUE"_err_en_US);
        }
      }
    }
    if (auto ignoreTKR{GetIgnoreTKR(symbol)}; !ignoreTKR.empty()) {
      const Symbol *ownerSymbol{symbol.owner().symbol()};
      bool inModuleProc{ownerSymbol && IsModuleProcedure(*ownerSymbol)};
      bool inExplicitExternalInterface{
          InInterface() && !IsSeparateModuleProcedureInterface(ownerSymbol)};
      if (!InInterface() && !inModuleProc) {
        messages_.Say(
            "!DIR$ IGNORE_TKR may apply only in an interface or a module procedure"_err_en_US);
      }
      if (ownerSymbol && ownerSymbol->attrs().test(Attr::ELEMENTAL) &&
          details.ignoreTKR().test(common::IgnoreTKR::Rank)) {
````
- **L937 EN**: Executes a standalone statement or declaration: `what);`.
  **L937 CN**: 执行一条独立语句或声明：`what);`。
- **L938 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L938 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L939 EN**: Closes the current lexical scope or compound statement.
  **L939 CN**: 结束当前词法作用域或复合语句块。
- **L940 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L940 CN**: 开始 `if` 控制流语句并计算其条件。
- **L941 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L941 CN**: 开始 `if` 控制流语句并计算其条件。
- **L942 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::LanguageFeature::RelaxedPureDummy,`.
  **L942 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::LanguageFeature::RelaxedPureDummy,`。
- **L943 EN**: Executes a call or declaration centered on `INTENT`.
  **L943 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L944 EN**: Transitions from the previous branch into the alternative path.
  **L944 CN**: 从前一个分支过渡到备选路径。
- **L945 EN**: Continues logic associated with callable symbol `Say`.
  **L945 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L946 EN**: Executes a call or declaration centered on `INTENT`.
  **L946 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L947 EN**: Closes the current lexical scope or compound statement.
  **L947 CN**: 结束当前词法作用域或复合语句块。
- **L948 EN**: Closes the current lexical scope or compound statement.
  **L948 CN**: 结束当前词法作用域或复合语句块。
- **L949 EN**: Closes the current lexical scope or compound statement.
  **L949 CN**: 结束当前词法作用域或复合语句块。
- **L950 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L950 CN**: 开始 `if` 控制流语句并计算其条件。
- **L951 EN**: Executes a call or declaration centered on `*ownerSymbol{symbol.owner`.
  **L951 CN**: 执行以 `*ownerSymbol{symbol.owner` 为核心的调用或声明。
- **L952 EN**: Executes a call or declaration centered on `IsModuleProcedure`.
  **L952 CN**: 执行以 `IsModuleProcedure` 为核心的调用或声明。
- **L953 EN**: Continues the surrounding expression or declaration: `bool inExplicitExternalInterface{`.
  **L953 CN**: 继续构造周围的表达式或声明：`bool inExplicitExternalInterface{`。
- **L954 EN**: Executes a call or declaration centered on `InInterface`.
  **L954 CN**: 执行以 `InInterface` 为核心的调用或声明。
- **L955 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L955 CN**: 开始 `if` 控制流语句并计算其条件。
- **L956 EN**: Continues logic associated with callable symbol `Say`.
  **L956 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L957 EN**: Executes a standalone statement or declaration: `"!DIR$ IGNORE_TKR may apply only in an interface or a module procedure"_err_en_US);`.
  **L957 CN**: 执行一条独立语句或声明：`"!DIR$ IGNORE_TKR may apply only in an interface or a module procedure"_err_en_US);`。
- **L958 EN**: Closes the current lexical scope or compound statement.
  **L958 CN**: 结束当前词法作用域或复合语句块。
- **L959 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L959 CN**: 开始 `if` 控制流语句并计算其条件。
- **L960 EN**: Starts a function, method, lambda, or structured scope: `details.ignoreTKR().test(common::IgnoreTKR::Rank)) {`.
  **L960 CN**: 开始一个函数、方法、lambda 或结构化作用域：`details.ignoreTKR().test(common::IgnoreTKR::Rank)) {`。

### Lines 961-984

````cpp
        messages_.Say(
            "!DIR$ IGNORE_TKR(R) may not apply in an ELEMENTAL procedure"_err_en_US);
      }
      // Descriptor based dummy args passed with ignore_tkr(c) are allowed
      // to have type/kind/rank differences
      if (IsPassedViaDescriptor(symbol) &&
          !ignoreTKR.test(common::IgnoreTKR::Contiguous)) {
        if (IsAllocatableOrObjectPointer(&symbol) &&
            !ignoreTKR.test(common::IgnoreTKR::Pointer)) {
          if (inExplicitExternalInterface) {
            Warn(common::UsageWarning::IgnoreTKRUsage,
                "!DIR$ IGNORE_TKR should not apply to an allocatable or pointer"_warn_en_US);
          } else {
            messages_.Say(
                "!DIR$ IGNORE_TKR may not apply to an allocatable or pointer"_err_en_US);
          }
        } else if (ignoreTKR.test(common::IgnoreTKR::Rank)) {
          if (ignoreTKR.count() == 1 && IsAssumedRank(symbol)) {
            Warn(common::UsageWarning::IgnoreTKRUsage,
                "!DIR$ IGNORE_TKR(R) is not meaningful for an assumed-rank array"_warn_en_US);
          } else if (inExplicitExternalInterface) {
            Warn(common::UsageWarning::IgnoreTKRUsage,
                "!DIR$ IGNORE_TKR(R) should not apply to a dummy argument passed via descriptor"_warn_en_US);
          } else {
````
- **L961 EN**: Continues logic associated with callable symbol `Say`.
  **L961 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L962 EN**: Executes a call or declaration centered on `IGNORE_TKR`.
  **L962 CN**: 执行以 `IGNORE_TKR` 为核心的调用或声明。
- **L963 EN**: Closes the current lexical scope or compound statement.
  **L963 CN**: 结束当前词法作用域或复合语句块。
- **L964 EN**: Comment explains nearby logic, intent, or metadata: `Descriptor based dummy args passed with ignore_tkr(c) are allowed`.
  **L964 CN**: 注释说明附近代码的逻辑、意图或元数据：`Descriptor based dummy args passed with ignore_tkr(c) are allowed`。
- **L965 EN**: Comment explains nearby logic, intent, or metadata: `to have type/kind/rank differences`.
  **L965 CN**: 注释说明附近代码的逻辑、意图或元数据：`to have type/kind/rank differences`。
- **L966 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L966 CN**: 开始 `if` 控制流语句并计算其条件。
- **L967 EN**: Starts a function, method, lambda, or structured scope: `!ignoreTKR.test(common::IgnoreTKR::Contiguous)) {`.
  **L967 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!ignoreTKR.test(common::IgnoreTKR::Contiguous)) {`。
- **L968 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L968 CN**: 开始 `if` 控制流语句并计算其条件。
- **L969 EN**: Starts a function, method, lambda, or structured scope: `!ignoreTKR.test(common::IgnoreTKR::Pointer)) {`.
  **L969 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!ignoreTKR.test(common::IgnoreTKR::Pointer)) {`。
- **L970 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L970 CN**: 开始 `if` 控制流语句并计算其条件。
- **L971 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::IgnoreTKRUsage,`.
  **L971 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::IgnoreTKRUsage,`。
- **L972 EN**: Executes a standalone statement or declaration: `"!DIR$ IGNORE_TKR should not apply to an allocatable or pointer"_warn_en_US);`.
  **L972 CN**: 执行一条独立语句或声明：`"!DIR$ IGNORE_TKR should not apply to an allocatable or pointer"_warn_en_US);`。
- **L973 EN**: Transitions from the previous branch into the alternative path.
  **L973 CN**: 从前一个分支过渡到备选路径。
- **L974 EN**: Continues logic associated with callable symbol `Say`.
  **L974 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L975 EN**: Executes a standalone statement or declaration: `"!DIR$ IGNORE_TKR may not apply to an allocatable or pointer"_err_en_US);`.
  **L975 CN**: 执行一条独立语句或声明：`"!DIR$ IGNORE_TKR may not apply to an allocatable or pointer"_err_en_US);`。
- **L976 EN**: Closes the current lexical scope or compound statement.
  **L976 CN**: 结束当前词法作用域或复合语句块。
- **L977 EN**: Transitions from the previous branch into an `else if` condition.
  **L977 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L978 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L978 CN**: 开始 `if` 控制流语句并计算其条件。
- **L979 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::IgnoreTKRUsage,`.
  **L979 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::IgnoreTKRUsage,`。
- **L980 EN**: Executes a call or declaration centered on `IGNORE_TKR`.
  **L980 CN**: 执行以 `IGNORE_TKR` 为核心的调用或声明。
- **L981 EN**: Transitions from the previous branch into an `else if` condition.
  **L981 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L982 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::IgnoreTKRUsage,`.
  **L982 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::IgnoreTKRUsage,`。
- **L983 EN**: Executes a call or declaration centered on `IGNORE_TKR`.
  **L983 CN**: 执行以 `IGNORE_TKR` 为核心的调用或声明。
- **L984 EN**: Transitions from the previous branch into the alternative path.
  **L984 CN**: 从前一个分支过渡到备选路径。

### Lines 985-1008

````cpp
            messages_.Say(
                "!DIR$ IGNORE_TKR(R) may not apply to a dummy argument passed via descriptor"_err_en_US);
          }
        }
      }
    }
  } else if (!details.ignoreTKR().empty()) {
    messages_.Say(
        "!DIR$ IGNORE_TKR directive may apply only to a dummy data argument"_err_en_US);
  }
  if (InElemental()) {
    if (details.isDummy()) { // C15100
      if (details.shape().Rank() > 0) {
        messages_.Say(
            "A dummy argument of an ELEMENTAL procedure must be scalar"_err_en_US);
      }
      if (IsAllocatable(symbol)) {
        messages_.Say(
            "A dummy argument of an ELEMENTAL procedure may not be ALLOCATABLE"_err_en_US);
      }
      if (evaluate::IsCoarray(symbol)) {
        messages_.Say(
            "A dummy argument of an ELEMENTAL procedure may not be a coarray"_err_en_US);
      }
````
- **L985 EN**: Continues logic associated with callable symbol `Say`.
  **L985 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L986 EN**: Executes a call or declaration centered on `IGNORE_TKR`.
  **L986 CN**: 执行以 `IGNORE_TKR` 为核心的调用或声明。
- **L987 EN**: Closes the current lexical scope or compound statement.
  **L987 CN**: 结束当前词法作用域或复合语句块。
- **L988 EN**: Closes the current lexical scope or compound statement.
  **L988 CN**: 结束当前词法作用域或复合语句块。
- **L989 EN**: Closes the current lexical scope or compound statement.
  **L989 CN**: 结束当前词法作用域或复合语句块。
- **L990 EN**: Closes the current lexical scope or compound statement.
  **L990 CN**: 结束当前词法作用域或复合语句块。
- **L991 EN**: Transitions from the previous branch into an `else if` condition.
  **L991 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L992 EN**: Continues logic associated with callable symbol `Say`.
  **L992 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L993 EN**: Executes a standalone statement or declaration: `"!DIR$ IGNORE_TKR directive may apply only to a dummy data argument"_err_en_US);`.
  **L993 CN**: 执行一条独立语句或声明：`"!DIR$ IGNORE_TKR directive may apply only to a dummy data argument"_err_en_US);`。
- **L994 EN**: Closes the current lexical scope or compound statement.
  **L994 CN**: 结束当前词法作用域或复合语句块。
- **L995 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L995 CN**: 开始 `if` 控制流语句并计算其条件。
- **L996 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L996 CN**: 开始 `if` 控制流语句并计算其条件。
- **L997 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L997 CN**: 开始 `if` 控制流语句并计算其条件。
- **L998 EN**: Continues logic associated with callable symbol `Say`.
  **L998 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L999 EN**: Executes a standalone statement or declaration: `"A dummy argument of an ELEMENTAL procedure must be scalar"_err_en_US);`.
  **L999 CN**: 执行一条独立语句或声明：`"A dummy argument of an ELEMENTAL procedure must be scalar"_err_en_US);`。
- **L1000 EN**: Closes the current lexical scope or compound statement.
  **L1000 CN**: 结束当前词法作用域或复合语句块。
- **L1001 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1001 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1002 EN**: Continues logic associated with callable symbol `Say`.
  **L1002 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1003 EN**: Executes a standalone statement or declaration: `"A dummy argument of an ELEMENTAL procedure may not be ALLOCATABLE"_err_en_US);`.
  **L1003 CN**: 执行一条独立语句或声明：`"A dummy argument of an ELEMENTAL procedure may not be ALLOCATABLE"_err_en_US);`。
- **L1004 EN**: Closes the current lexical scope or compound statement.
  **L1004 CN**: 结束当前词法作用域或复合语句块。
- **L1005 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1005 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1006 EN**: Continues logic associated with callable symbol `Say`.
  **L1006 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1007 EN**: Executes a standalone statement or declaration: `"A dummy argument of an ELEMENTAL procedure may not be a coarray"_err_en_US);`.
  **L1007 CN**: 执行一条独立语句或声明：`"A dummy argument of an ELEMENTAL procedure may not be a coarray"_err_en_US);`。
- **L1008 EN**: Closes the current lexical scope or compound statement.
  **L1008 CN**: 结束当前词法作用域或复合语句块。

### Lines 1009-1032

````cpp
      if (IsPointer(symbol)) {
        messages_.Say(
            "A dummy argument of an ELEMENTAL procedure may not be a POINTER"_err_en_US);
      }
      if (!symbol.attrs().HasAny(Attrs{Attr::VALUE, Attr::INTENT_IN,
              Attr::INTENT_INOUT, Attr::INTENT_OUT})) { // F'2023 C15120
        messages_.Say(
            "A dummy argument of an ELEMENTAL procedure must have an INTENT() or VALUE attribute"_err_en_US);
      }
    } else if (IsFunctionResult(symbol)) { // C15101
      if (details.shape().Rank() > 0) {
        messages_.Say(
            "The result of an ELEMENTAL function must be scalar"_err_en_US);
      }
      if (IsAllocatable(symbol)) {
        messages_.Say(
            "The result of an ELEMENTAL function may not be ALLOCATABLE"_err_en_US);
      }
      if (IsPointer(symbol)) {
        messages_.Say(
            "The result of an ELEMENTAL function may not be a POINTER"_err_en_US);
      }
    }
  }
````
- **L1009 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1009 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1010 EN**: Continues logic associated with callable symbol `Say`.
  **L1010 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1011 EN**: Executes a standalone statement or declaration: `"A dummy argument of an ELEMENTAL procedure may not be a POINTER"_err_en_US);`.
  **L1011 CN**: 执行一条独立语句或声明：`"A dummy argument of an ELEMENTAL procedure may not be a POINTER"_err_en_US);`。
- **L1012 EN**: Closes the current lexical scope or compound statement.
  **L1012 CN**: 结束当前词法作用域或复合语句块。
- **L1013 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1013 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1014 EN**: Continues the surrounding expression or declaration: `Attr::INTENT_INOUT, Attr::INTENT_OUT})) { // F'2023 C15120`.
  **L1014 CN**: 继续构造周围的表达式或声明：`Attr::INTENT_INOUT, Attr::INTENT_OUT})) { // F'2023 C15120`。
- **L1015 EN**: Continues logic associated with callable symbol `Say`.
  **L1015 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1016 EN**: Executes a call or declaration centered on `INTENT`.
  **L1016 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L1017 EN**: Closes the current lexical scope or compound statement.
  **L1017 CN**: 结束当前词法作用域或复合语句块。
- **L1018 EN**: Transitions from the previous branch into an `else if` condition.
  **L1018 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1019 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1019 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1020 EN**: Continues logic associated with callable symbol `Say`.
  **L1020 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1021 EN**: Executes a standalone statement or declaration: `"The result of an ELEMENTAL function must be scalar"_err_en_US);`.
  **L1021 CN**: 执行一条独立语句或声明：`"The result of an ELEMENTAL function must be scalar"_err_en_US);`。
- **L1022 EN**: Closes the current lexical scope or compound statement.
  **L1022 CN**: 结束当前词法作用域或复合语句块。
- **L1023 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1023 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1024 EN**: Continues logic associated with callable symbol `Say`.
  **L1024 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1025 EN**: Executes a standalone statement or declaration: `"The result of an ELEMENTAL function may not be ALLOCATABLE"_err_en_US);`.
  **L1025 CN**: 执行一条独立语句或声明：`"The result of an ELEMENTAL function may not be ALLOCATABLE"_err_en_US);`。
- **L1026 EN**: Closes the current lexical scope or compound statement.
  **L1026 CN**: 结束当前词法作用域或复合语句块。
- **L1027 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1027 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1028 EN**: Continues logic associated with callable symbol `Say`.
  **L1028 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1029 EN**: Executes a standalone statement or declaration: `"The result of an ELEMENTAL function may not be a POINTER"_err_en_US);`.
  **L1029 CN**: 执行一条独立语句或声明：`"The result of an ELEMENTAL function may not be a POINTER"_err_en_US);`。
- **L1030 EN**: Closes the current lexical scope or compound statement.
  **L1030 CN**: 结束当前词法作用域或复合语句块。
- **L1031 EN**: Closes the current lexical scope or compound statement.
  **L1031 CN**: 结束当前词法作用域或复合语句块。
- **L1032 EN**: Closes the current lexical scope or compound statement.
  **L1032 CN**: 结束当前词法作用域或复合语句块。

### Lines 1033-1056

````cpp
  if (HasDeclarationInitializer(symbol)) { // C808; ignore DATA initialization
    CheckPointerInitialization(symbol);
    if (IsAutomatic(symbol)) {
      messages_.Say(
          "An automatic variable or component must not be initialized"_err_en_US);
    } else if (IsDummy(symbol)) {
      messages_.Say("A dummy argument must not be initialized"_err_en_US);
    } else if (IsFunctionResult(symbol)) {
      messages_.Say("A function result must not be initialized"_err_en_US);
    } else if (IsInBlankCommon(symbol)) {
      Warn(common::LanguageFeature::InitBlankCommon,
          "A variable in blank COMMON should not be initialized"_port_en_US);
    }
  }
  if (symbol.owner().kind() == Scope::Kind::BlockData) {
    if (IsAllocatable(symbol)) {
      messages_.Say(
          "An ALLOCATABLE variable may not appear in a BLOCK DATA subprogram"_err_en_US);
    } else if (IsInitialized(symbol) && !FindCommonBlockContaining(symbol)) {
      messages_.Say(
          "An initialized variable in BLOCK DATA must be in a COMMON block"_err_en_US);
    }
  }
  if (derived && InPure() && !InInterface() &&
````
- **L1033 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1033 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1034 EN**: Executes a call or declaration centered on `CheckPointerInitialization`.
  **L1034 CN**: 执行以 `CheckPointerInitialization` 为核心的调用或声明。
- **L1035 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1035 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1036 EN**: Continues logic associated with callable symbol `Say`.
  **L1036 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1037 EN**: Executes a standalone statement or declaration: `"An automatic variable or component must not be initialized"_err_en_US);`.
  **L1037 CN**: 执行一条独立语句或声明：`"An automatic variable or component must not be initialized"_err_en_US);`。
- **L1038 EN**: Transitions from the previous branch into an `else if` condition.
  **L1038 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1039 EN**: Executes a call or declaration centered on `messages_.Say`.
  **L1039 CN**: 执行以 `messages_.Say` 为核心的调用或声明。
- **L1040 EN**: Transitions from the previous branch into an `else if` condition.
  **L1040 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1041 EN**: Executes a call or declaration centered on `messages_.Say`.
  **L1041 CN**: 执行以 `messages_.Say` 为核心的调用或声明。
- **L1042 EN**: Transitions from the previous branch into an `else if` condition.
  **L1042 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1043 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::LanguageFeature::InitBlankCommon,`.
  **L1043 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::LanguageFeature::InitBlankCommon,`。
- **L1044 EN**: Executes a standalone statement or declaration: `"A variable in blank COMMON should not be initialized"_port_en_US);`.
  **L1044 CN**: 执行一条独立语句或声明：`"A variable in blank COMMON should not be initialized"_port_en_US);`。
- **L1045 EN**: Closes the current lexical scope or compound statement.
  **L1045 CN**: 结束当前词法作用域或复合语句块。
- **L1046 EN**: Closes the current lexical scope or compound statement.
  **L1046 CN**: 结束当前词法作用域或复合语句块。
- **L1047 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1047 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1048 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1048 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1049 EN**: Continues logic associated with callable symbol `Say`.
  **L1049 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1050 EN**: Executes a standalone statement or declaration: `"An ALLOCATABLE variable may not appear in a BLOCK DATA subprogram"_err_en_US);`.
  **L1050 CN**: 执行一条独立语句或声明：`"An ALLOCATABLE variable may not appear in a BLOCK DATA subprogram"_err_en_US);`。
- **L1051 EN**: Transitions from the previous branch into an `else if` condition.
  **L1051 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1052 EN**: Continues logic associated with callable symbol `Say`.
  **L1052 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1053 EN**: Executes a standalone statement or declaration: `"An initialized variable in BLOCK DATA must be in a COMMON block"_err_en_US);`.
  **L1053 CN**: 执行一条独立语句或声明：`"An initialized variable in BLOCK DATA must be in a COMMON block"_err_en_US);`。
- **L1054 EN**: Closes the current lexical scope or compound statement.
  **L1054 CN**: 结束当前词法作用域或复合语句块。
- **L1055 EN**: Closes the current lexical scope or compound statement.
  **L1055 CN**: 结束当前词法作用域或复合语句块。
- **L1056 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1056 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1057-1080

````cpp
      IsAutomaticallyDestroyed(symbol) &&
      !IsIntentOut(symbol) /*has better messages*/ &&
      !IsFunctionResult(symbol) /*ditto*/) {
    // Check automatically deallocated local variables for possible
    // problems with finalization in PURE.
    if (auto whyNot{WhyNotDefinable(symbol.name(), symbol.owner(),
            {DefinabilityFlag::PotentialDeallocation}, symbol)}) {
      if (auto *msg{messages_.Say(
              "'%s' may not be a local variable in a pure subprogram"_err_en_US,
              symbol.name())}) {
        msg->Attach(std::move(whyNot->set_severity(parser::Severity::Because)));
      }
    }
  }
  if (symbol.attrs().test(Attr::EXTERNAL)) {
    SayWithDeclaration(symbol,
        "'%s' is a data object and may not be EXTERNAL"_err_en_US,
        symbol.name());
  }
  if (symbol.test(Symbol::Flag::CrayPointee)) {
    // NB, IsSaved was too smart here.
    if (details.init()) {
      messages_.Say(
          "Cray pointee '%s' may not be initialized"_err_en_US, symbol.name());
````
- **L1057 EN**: Continues logic associated with callable symbol `IsAutomaticallyDestroyed`.
  **L1057 CN**: 继续与可调用符号 `IsAutomaticallyDestroyed` 相关的逻辑。
- **L1058 EN**: Continues logic associated with callable symbol `IsIntentOut`.
  **L1058 CN**: 继续与可调用符号 `IsIntentOut` 相关的逻辑。
- **L1059 EN**: Starts a function, method, lambda, or structured scope: `!IsFunctionResult(symbol) /*ditto*/) {`.
  **L1059 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!IsFunctionResult(symbol) /*ditto*/) {`。
- **L1060 EN**: Comment explains nearby logic, intent, or metadata: `Check automatically deallocated local variables for possible`.
  **L1060 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check automatically deallocated local variables for possible`。
- **L1061 EN**: Comment explains nearby logic, intent, or metadata: `problems with finalization in PURE.`.
  **L1061 CN**: 注释说明附近代码的逻辑、意图或元数据：`problems with finalization in PURE.`。
- **L1062 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1062 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1063 EN**: Continues the surrounding expression or declaration: `{DefinabilityFlag::PotentialDeallocation}, symbol)}) {`.
  **L1063 CN**: 继续构造周围的表达式或声明：`{DefinabilityFlag::PotentialDeallocation}, symbol)}) {`。
- **L1064 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1064 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' may not be a local variable in a pure subprogram"_err_en_US,`.
  **L1065 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' may not be a local variable in a pure subprogram"_err_en_US,`。
- **L1066 EN**: Starts a function, method, lambda, or structured scope: `symbol.name())}) {`.
  **L1066 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.name())}) {`。
- **L1067 EN**: Executes a call or declaration centered on `msg->Attach`.
  **L1067 CN**: 执行以 `msg->Attach` 为核心的调用或声明。
- **L1068 EN**: Closes the current lexical scope or compound statement.
  **L1068 CN**: 结束当前词法作用域或复合语句块。
- **L1069 EN**: Closes the current lexical scope or compound statement.
  **L1069 CN**: 结束当前词法作用域或复合语句块。
- **L1070 EN**: Closes the current lexical scope or compound statement.
  **L1070 CN**: 结束当前词法作用域或复合语句块。
- **L1071 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1071 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(symbol,`.
  **L1072 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(symbol,`。
- **L1073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' is a data object and may not be EXTERNAL"_err_en_US,`.
  **L1073 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' is a data object and may not be EXTERNAL"_err_en_US,`。
- **L1074 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1074 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1075 EN**: Closes the current lexical scope or compound statement.
  **L1075 CN**: 结束当前词法作用域或复合语句块。
- **L1076 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1076 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1077 EN**: Comment explains nearby logic, intent, or metadata: `NB, IsSaved was too smart here.`.
  **L1077 CN**: 注释说明附近代码的逻辑、意图或元数据：`NB, IsSaved was too smart here.`。
- **L1078 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1078 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1079 EN**: Continues logic associated with callable symbol `Say`.
  **L1079 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1080 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1080 CN**: 执行以 `symbol.name` 为核心的调用或声明。

### Lines 1081-1104

````cpp
    }
    if (symbol.attrs().test(Attr::SAVE)) {
      messages_.Say(
          "Cray pointee '%s' may not have the SAVE attribute"_err_en_US,
          symbol.name());
    }
  }
  if (derived) {
    bool isUnsavedLocal{
        isLocalVariable && !IsAllocatable(symbol) && !IsSaved(symbol)};
    if (IsFunctionResult(symbol) || IsPointer(symbol) ||
        evaluate::IsCoarray(symbol) || isUnsavedLocal) {
      if (auto badPotential{FindCoarrayPotentialComponent(*derived)}) {
        if (IsFunctionResult(symbol)) { // F'2023 C825
          SayWithDeclaration(*badPotential,
              "Function result '%s' may not have a coarray potential component '%s'"_err_en_US,
              symbol.name(), badPotential.BuildResultDesignatorName());
        } else if (IsPointer(symbol)) { // F'2023 C825
          SayWithDeclaration(*badPotential,
              "Pointer '%s' may not have a coarray potential component '%s'"_err_en_US,
              symbol.name(), badPotential.BuildResultDesignatorName());
        } else if (evaluate::IsCoarray(symbol)) { // F'2023 C825
          SayWithDeclaration(*badPotential,
              "Coarray '%s' may not have a coarray potential component '%s'"_err_en_US,
````
- **L1081 EN**: Closes the current lexical scope or compound statement.
  **L1081 CN**: 结束当前词法作用域或复合语句块。
- **L1082 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1082 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1083 EN**: Continues logic associated with callable symbol `Say`.
  **L1083 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1084 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Cray pointee '%s' may not have the SAVE attribute"_err_en_US,`.
  **L1084 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Cray pointee '%s' may not have the SAVE attribute"_err_en_US,`。
- **L1085 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1085 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1086 EN**: Closes the current lexical scope or compound statement.
  **L1086 CN**: 结束当前词法作用域或复合语句块。
- **L1087 EN**: Closes the current lexical scope or compound statement.
  **L1087 CN**: 结束当前词法作用域或复合语句块。
- **L1088 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1088 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1089 EN**: Continues the surrounding expression or declaration: `bool isUnsavedLocal{`.
  **L1089 CN**: 继续构造周围的表达式或声明：`bool isUnsavedLocal{`。
- **L1090 EN**: Executes a call or declaration centered on `!IsAllocatable`.
  **L1090 CN**: 执行以 `!IsAllocatable` 为核心的调用或声明。
- **L1091 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1091 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1092 EN**: Starts a function, method, lambda, or structured scope: `evaluate::IsCoarray(symbol) || isUnsavedLocal) {`.
  **L1092 CN**: 开始一个函数、方法、lambda 或结构化作用域：`evaluate::IsCoarray(symbol) || isUnsavedLocal) {`。
- **L1093 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1093 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1094 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1094 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1095 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*badPotential,`.
  **L1095 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*badPotential,`。
- **L1096 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Function result '%s' may not have a coarray potential component '%s'"_err_en_US,`.
  **L1096 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Function result '%s' may not have a coarray potential component '%s'"_err_en_US,`。
- **L1097 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1097 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1098 EN**: Transitions from the previous branch into an `else if` condition.
  **L1098 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1099 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*badPotential,`.
  **L1099 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*badPotential,`。
- **L1100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Pointer '%s' may not have a coarray potential component '%s'"_err_en_US,`.
  **L1100 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Pointer '%s' may not have a coarray potential component '%s'"_err_en_US,`。
- **L1101 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1101 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1102 EN**: Transitions from the previous branch into an `else if` condition.
  **L1102 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*badPotential,`.
  **L1103 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*badPotential,`。
- **L1104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Coarray '%s' may not have a coarray potential component '%s'"_err_en_US,`.
  **L1104 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Coarray '%s' may not have a coarray potential component '%s'"_err_en_US,`。

### Lines 1105-1128

````cpp
              symbol.name(), badPotential.BuildResultDesignatorName());
        } else if (isUnsavedLocal) { // F'2023 C826
          SayWithDeclaration(*badPotential,
              "Local variable '%s' without the SAVE or ALLOCATABLE attribute may not have a coarray potential subobject component '%s'"_err_en_US,
              symbol.name(), badPotential.BuildResultDesignatorName());
        } else {
          DIE("caught unexpected bad coarray potential component");
        }
      }
    } else if (isComponent && (IsAllocatable(symbol) || symbol.Rank() > 0)) {
      if (auto badUltimate{FindCoarrayUltimateComponent(*derived)}) {
        // TODO: still an error in F'2023?
        SayWithDeclaration(*badUltimate,
            "Allocatable or array component '%s' may not have a coarray ultimate component '%s'"_err_en_US,
            symbol.name(), badUltimate.BuildResultDesignatorName());
      }
    }
  }

  // Check CUDA attributes and special circumstances of being in device
  // subprograms
  const Scope &progUnit{GetProgramUnitContaining(symbol)};
  const auto *subpDetails{!isComponent && progUnit.symbol()
          ? progUnit.symbol()->detailsIf<SubprogramDetails>()
````
- **L1105 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1105 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1106 EN**: Transitions from the previous branch into an `else if` condition.
  **L1106 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*badPotential,`.
  **L1107 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*badPotential,`。
- **L1108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Local variable '%s' without the SAVE or ALLOCATABLE attribute may not have a coarray potential subobject component '%s'"_err_en_US,`.
  **L1108 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Local variable '%s' without the SAVE or ALLOCATABLE attribute may not have a coarray potential subobject component '%s'"_err_en_US,`。
- **L1109 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1109 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1110 EN**: Transitions from the previous branch into the alternative path.
  **L1110 CN**: 从前一个分支过渡到备选路径。
- **L1111 EN**: Executes a call or declaration centered on `DIE`.
  **L1111 CN**: 执行以 `DIE` 为核心的调用或声明。
- **L1112 EN**: Closes the current lexical scope or compound statement.
  **L1112 CN**: 结束当前词法作用域或复合语句块。
- **L1113 EN**: Closes the current lexical scope or compound statement.
  **L1113 CN**: 结束当前词法作用域或复合语句块。
- **L1114 EN**: Transitions from the previous branch into an `else if` condition.
  **L1114 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1115 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1115 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1116 EN**: Comment records a pending task or caution: `TODO: still an error in F'2023?`.
  **L1116 CN**: 注释记录待办事项或注意点：`TODO: still an error in F'2023?`。
- **L1117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*badUltimate,`.
  **L1117 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*badUltimate,`。
- **L1118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Allocatable or array component '%s' may not have a coarray ultimate component '%s'"_err_en_US,`.
  **L1118 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Allocatable or array component '%s' may not have a coarray ultimate component '%s'"_err_en_US,`。
- **L1119 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1119 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1120 EN**: Closes the current lexical scope or compound statement.
  **L1120 CN**: 结束当前词法作用域或复合语句块。
- **L1121 EN**: Closes the current lexical scope or compound statement.
  **L1121 CN**: 结束当前词法作用域或复合语句块。
- **L1122 EN**: Closes the current lexical scope or compound statement.
  **L1122 CN**: 结束当前词法作用域或复合语句块。
- **L1123 EN**: Blank line separating nearby declarations or logic blocks.
  **L1123 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1124 EN**: Comment explains nearby logic, intent, or metadata: `Check CUDA attributes and special circumstances of being in device`.
  **L1124 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check CUDA attributes and special circumstances of being in device`。
- **L1125 EN**: Comment explains nearby logic, intent, or metadata: `subprograms`.
  **L1125 CN**: 注释说明附近代码的逻辑、意图或元数据：`subprograms`。
- **L1126 EN**: Executes a call or declaration centered on `&progUnit{GetProgramUnitContaining`.
  **L1126 CN**: 执行以 `&progUnit{GetProgramUnitContaining` 为核心的调用或声明。
- **L1127 EN**: Continues logic associated with callable symbol `symbol`.
  **L1127 CN**: 继续与可调用符号 `symbol` 相关的逻辑。
- **L1128 EN**: Continues logic associated with callable symbol `symbol`.
  **L1128 CN**: 继续与可调用符号 `symbol` 相关的逻辑。

### Lines 1129-1152

````cpp
          : nullptr};
  bool inDeviceSubprogram{IsCUDADeviceContext(&symbol.owner())};
  if (inDeviceSubprogram) {
    if (IsSaved(symbol)) {
      Warn(common::UsageWarning::CUDAUsage,
          "'%s' should not have the SAVE attribute or initialization in a device subprogram"_warn_en_US,
          symbol.name());
    }
    if (IsPointer(symbol)) {
      Warn(common::UsageWarning::CUDAUsage,
          "Pointer '%s' may not be associated in a device subprogram"_warn_en_US,
          symbol.name());
    }
    if (details.isDummy() &&
        details.cudaDataAttr().value_or(common::CUDADataAttr::Device) !=
            common::CUDADataAttr::Device &&
        details.cudaDataAttr().value_or(common::CUDADataAttr::Device) !=
            common::CUDADataAttr::Managed &&
        details.cudaDataAttr().value_or(common::CUDADataAttr::Device) !=
            common::CUDADataAttr::Shared) {
      Warn(common::UsageWarning::CUDAUsage,
          "Dummy argument '%s' may not have ATTRIBUTES(%s) in a device subprogram"_warn_en_US,
          symbol.name(),
          parser::ToUpperCaseLetters(
````
- **L1129 EN**: Executes a standalone statement or declaration: `: nullptr};`.
  **L1129 CN**: 执行一条独立语句或声明：`: nullptr};`。
- **L1130 EN**: Executes a call or declaration centered on `inDeviceSubprogram{IsCUDADeviceContext`.
  **L1130 CN**: 执行以 `inDeviceSubprogram{IsCUDADeviceContext` 为核心的调用或声明。
- **L1131 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1131 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1132 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1132 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::CUDAUsage,`.
  **L1133 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::CUDAUsage,`。
- **L1134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' should not have the SAVE attribute or initialization in a device subprogram"_warn_en_US,`.
  **L1134 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' should not have the SAVE attribute or initialization in a device subprogram"_warn_en_US,`。
- **L1135 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1135 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1136 EN**: Closes the current lexical scope or compound statement.
  **L1136 CN**: 结束当前词法作用域或复合语句块。
- **L1137 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1137 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::CUDAUsage,`.
  **L1138 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::CUDAUsage,`。
- **L1139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Pointer '%s' may not be associated in a device subprogram"_warn_en_US,`.
  **L1139 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Pointer '%s' may not be associated in a device subprogram"_warn_en_US,`。
- **L1140 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1140 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1141 EN**: Closes the current lexical scope or compound statement.
  **L1141 CN**: 结束当前词法作用域或复合语句块。
- **L1142 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1142 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1143 EN**: Continues logic associated with callable symbol `cudaDataAttr`.
  **L1143 CN**: 继续与可调用符号 `cudaDataAttr` 相关的逻辑。
- **L1144 EN**: Continues the surrounding expression or declaration: `common::CUDADataAttr::Device &&`.
  **L1144 CN**: 继续构造周围的表达式或声明：`common::CUDADataAttr::Device &&`。
- **L1145 EN**: Continues logic associated with callable symbol `cudaDataAttr`.
  **L1145 CN**: 继续与可调用符号 `cudaDataAttr` 相关的逻辑。
- **L1146 EN**: Continues the surrounding expression or declaration: `common::CUDADataAttr::Managed &&`.
  **L1146 CN**: 继续构造周围的表达式或声明：`common::CUDADataAttr::Managed &&`。
- **L1147 EN**: Continues logic associated with callable symbol `cudaDataAttr`.
  **L1147 CN**: 继续与可调用符号 `cudaDataAttr` 相关的逻辑。
- **L1148 EN**: Continues the surrounding expression or declaration: `common::CUDADataAttr::Shared) {`.
  **L1148 CN**: 继续构造周围的表达式或声明：`common::CUDADataAttr::Shared) {`。
- **L1149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::CUDAUsage,`.
  **L1149 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::CUDAUsage,`。
- **L1150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy argument '%s' may not have ATTRIBUTES(%s) in a device subprogram"_warn_en_US,`.
  **L1150 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy argument '%s' may not have ATTRIBUTES(%s) in a device subprogram"_warn_en_US,`。
- **L1151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol.name(),`.
  **L1151 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol.name(),`。
- **L1152 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L1152 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。

### Lines 1153-1176

````cpp
              common::EnumToString(*details.cudaDataAttr())));
    }
  }
  if (details.cudaDataAttr()) {
    if (auto dyType{evaluate::DynamicType::From(symbol)}) {
      if (dyType->category() != TypeCategory::Derived) {
        if (!IsCUDAIntrinsicType(*dyType)) {
          messages_.Say(
              "'%s' has intrinsic type '%s' that is not available on the device"_err_en_US,
              symbol.name(), dyType->AsFortran());
        }
      }
    }
    auto attr{*details.cudaDataAttr()};
    switch (attr) {
    case common::CUDADataAttr::Value:
      break; // Nothing to check for VALUE attribute
    case common::CUDADataAttr::Constant:
      if (subpDetails && !inDeviceSubprogram) {
        messages_.Say(
            "Object '%s' with ATTRIBUTES(CONSTANT) may not be declared in a host subprogram"_err_en_US,
            symbol.name());
      } else if (IsAllocatableOrPointer(symbol) ||
          symbol.attrs().test(Attr::TARGET)) {
````
- **L1153 EN**: Executes a call or declaration centered on `common::EnumToString`.
  **L1153 CN**: 执行以 `common::EnumToString` 为核心的调用或声明。
- **L1154 EN**: Closes the current lexical scope or compound statement.
  **L1154 CN**: 结束当前词法作用域或复合语句块。
- **L1155 EN**: Closes the current lexical scope or compound statement.
  **L1155 CN**: 结束当前词法作用域或复合语句块。
- **L1156 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1156 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1157 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1157 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1158 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1158 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1159 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1159 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1160 EN**: Continues logic associated with callable symbol `Say`.
  **L1160 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' has intrinsic type '%s' that is not available on the device"_err_en_US,`.
  **L1161 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' has intrinsic type '%s' that is not available on the device"_err_en_US,`。
- **L1162 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1162 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1163 EN**: Closes the current lexical scope or compound statement.
  **L1163 CN**: 结束当前词法作用域或复合语句块。
- **L1164 EN**: Closes the current lexical scope or compound statement.
  **L1164 CN**: 结束当前词法作用域或复合语句块。
- **L1165 EN**: Closes the current lexical scope or compound statement.
  **L1165 CN**: 结束当前词法作用域或复合语句块。
- **L1166 EN**: Executes a call or declaration centered on `attr{*details.cudaDataAttr`.
  **L1166 CN**: 执行以 `attr{*details.cudaDataAttr` 为核心的调用或声明。
- **L1167 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L1167 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1168 EN**: Introduces a switch dispatch label: `case common::CUDADataAttr::Value:`.
  **L1168 CN**: 引入一个 switch 分发标签：`case common::CUDADataAttr::Value:`。
- **L1169 EN**: Exits the nearest loop or switch statement.
  **L1169 CN**: 退出最近的循环或 switch 语句。
- **L1170 EN**: Introduces a switch dispatch label: `case common::CUDADataAttr::Constant:`.
  **L1170 CN**: 引入一个 switch 分发标签：`case common::CUDADataAttr::Constant:`。
- **L1171 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1171 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1172 EN**: Continues logic associated with callable symbol `Say`.
  **L1172 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Object '%s' with ATTRIBUTES(CONSTANT) may not be declared in a host subprogram"_err_en_US,`.
  **L1173 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Object '%s' with ATTRIBUTES(CONSTANT) may not be declared in a host subprogram"_err_en_US,`。
- **L1174 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1174 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1175 EN**: Transitions from the previous branch into an `else if` condition.
  **L1175 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1176 EN**: Starts a function, method, lambda, or structured scope: `symbol.attrs().test(Attr::TARGET)) {`.
  **L1176 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.attrs().test(Attr::TARGET)) {`。

### Lines 1177-1200

````cpp
        messages_.Say(
            "Object '%s' with ATTRIBUTES(CONSTANT) may not be allocatable, pointer, or target"_err_en_US,
            symbol.name());
      } else if (auto shape{evaluate::GetShape(foldingContext_, symbol)};
                 !shape ||
                 !evaluate::AsConstantExtents(foldingContext_, *shape)) {
        messages_.Say(
            "Object '%s' with ATTRIBUTES(CONSTANT) must have constant array bounds"_err_en_US,
            symbol.name());
      }
      break;
    case common::CUDADataAttr::Device:
      if (isComponent && !IsAllocatable(symbol) && !IsPointer(symbol)) {
        messages_.Say(
            "Component '%s' with ATTRIBUTES(DEVICE) must also be allocatable or pointer"_err_en_US,
            symbol.name());
      }
      break;
    case common::CUDADataAttr::Managed:
      if (!IsAutomatic(symbol) && !IsAllocatableOrPointer(symbol) &&
          !details.isDummy() && !evaluate::IsExplicitShape(symbol)) {
        messages_.Say(
            "Object '%s' with ATTRIBUTES(MANAGED) must also be allocatable, pointer, automatic, explicit shape, or a dummy argument"_err_en_US,
            symbol.name());
````
- **L1177 EN**: Continues logic associated with callable symbol `Say`.
  **L1177 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Object '%s' with ATTRIBUTES(CONSTANT) may not be allocatable, pointer, or target"_err_en_US,`.
  **L1178 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Object '%s' with ATTRIBUTES(CONSTANT) may not be allocatable, pointer, or target"_err_en_US,`。
- **L1179 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1179 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1180 EN**: Transitions from the previous branch into an `else if` condition.
  **L1180 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1181 EN**: Continues the surrounding expression or declaration: `!shape ||`.
  **L1181 CN**: 继续构造周围的表达式或声明：`!shape ||`。
- **L1182 EN**: Starts a function, method, lambda, or structured scope: `!evaluate::AsConstantExtents(foldingContext_, *shape)) {`.
  **L1182 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!evaluate::AsConstantExtents(foldingContext_, *shape)) {`。
- **L1183 EN**: Continues logic associated with callable symbol `Say`.
  **L1183 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Object '%s' with ATTRIBUTES(CONSTANT) must have constant array bounds"_err_en_US,`.
  **L1184 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Object '%s' with ATTRIBUTES(CONSTANT) must have constant array bounds"_err_en_US,`。
- **L1185 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1185 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1186 EN**: Closes the current lexical scope or compound statement.
  **L1186 CN**: 结束当前词法作用域或复合语句块。
- **L1187 EN**: Exits the nearest loop or switch statement.
  **L1187 CN**: 退出最近的循环或 switch 语句。
- **L1188 EN**: Introduces a switch dispatch label: `case common::CUDADataAttr::Device:`.
  **L1188 CN**: 引入一个 switch 分发标签：`case common::CUDADataAttr::Device:`。
- **L1189 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1189 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1190 EN**: Continues logic associated with callable symbol `Say`.
  **L1190 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Component '%s' with ATTRIBUTES(DEVICE) must also be allocatable or pointer"_err_en_US,`.
  **L1191 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Component '%s' with ATTRIBUTES(DEVICE) must also be allocatable or pointer"_err_en_US,`。
- **L1192 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1192 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1193 EN**: Closes the current lexical scope or compound statement.
  **L1193 CN**: 结束当前词法作用域或复合语句块。
- **L1194 EN**: Exits the nearest loop or switch statement.
  **L1194 CN**: 退出最近的循环或 switch 语句。
- **L1195 EN**: Introduces a switch dispatch label: `case common::CUDADataAttr::Managed:`.
  **L1195 CN**: 引入一个 switch 分发标签：`case common::CUDADataAttr::Managed:`。
- **L1196 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1196 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1197 EN**: Starts a function, method, lambda, or structured scope: `!details.isDummy() && !evaluate::IsExplicitShape(symbol)) {`.
  **L1197 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!details.isDummy() && !evaluate::IsExplicitShape(symbol)) {`。
- **L1198 EN**: Continues logic associated with callable symbol `Say`.
  **L1198 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Object '%s' with ATTRIBUTES(MANAGED) must also be allocatable, pointer, automatic, explicit shape, or a dummy argument"_err_en_US,`.
  **L1199 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Object '%s' with ATTRIBUTES(MANAGED) must also be allocatable, pointer, automatic, explicit shape, or a dummy argument"_err_en_US,`。
- **L1200 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1200 CN**: 执行以 `symbol.name` 为核心的调用或声明。

### Lines 1201-1224

````cpp
      }
      break;
    case common::CUDADataAttr::Pinned:
      if (inDeviceSubprogram) {
        Warn(common::UsageWarning::CUDAUsage,
            "Object '%s' with ATTRIBUTES(PINNED) may not be declared in a device subprogram"_warn_en_US,
            symbol.name());
      } else if (IsPointer(symbol)) {
        Warn(common::UsageWarning::CUDAUsage,
            "Object '%s' with ATTRIBUTES(PINNED) may not be a pointer"_warn_en_US,
            symbol.name());
      } else if (!IsAllocatable(symbol)) {
        Warn(common::UsageWarning::CUDAUsage,
            "Object '%s' with ATTRIBUTES(PINNED) should also be allocatable"_warn_en_US,
            symbol.name());
      }
      break;
    case common::CUDADataAttr::Shared:
      if (IsAllocatableOrPointer(symbol) || symbol.attrs().test(Attr::TARGET)) {
        messages_.Say(
            "Object '%s' with ATTRIBUTES(SHARED) may not be allocatable, pointer, or target"_err_en_US,
            symbol.name());
      } else if (!inDeviceSubprogram) {
        messages_.Say(
````
- **L1201 EN**: Closes the current lexical scope or compound statement.
  **L1201 CN**: 结束当前词法作用域或复合语句块。
- **L1202 EN**: Exits the nearest loop or switch statement.
  **L1202 CN**: 退出最近的循环或 switch 语句。
- **L1203 EN**: Introduces a switch dispatch label: `case common::CUDADataAttr::Pinned:`.
  **L1203 CN**: 引入一个 switch 分发标签：`case common::CUDADataAttr::Pinned:`。
- **L1204 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1204 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::CUDAUsage,`.
  **L1205 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::CUDAUsage,`。
- **L1206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Object '%s' with ATTRIBUTES(PINNED) may not be declared in a device subprogram"_warn_en_US,`.
  **L1206 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Object '%s' with ATTRIBUTES(PINNED) may not be declared in a device subprogram"_warn_en_US,`。
- **L1207 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1207 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1208 EN**: Transitions from the previous branch into an `else if` condition.
  **L1208 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::CUDAUsage,`.
  **L1209 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::CUDAUsage,`。
- **L1210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Object '%s' with ATTRIBUTES(PINNED) may not be a pointer"_warn_en_US,`.
  **L1210 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Object '%s' with ATTRIBUTES(PINNED) may not be a pointer"_warn_en_US,`。
- **L1211 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1211 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1212 EN**: Transitions from the previous branch into an `else if` condition.
  **L1212 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::CUDAUsage,`.
  **L1213 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::CUDAUsage,`。
- **L1214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Object '%s' with ATTRIBUTES(PINNED) should also be allocatable"_warn_en_US,`.
  **L1214 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Object '%s' with ATTRIBUTES(PINNED) should also be allocatable"_warn_en_US,`。
- **L1215 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1215 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1216 EN**: Closes the current lexical scope or compound statement.
  **L1216 CN**: 结束当前词法作用域或复合语句块。
- **L1217 EN**: Exits the nearest loop or switch statement.
  **L1217 CN**: 退出最近的循环或 switch 语句。
- **L1218 EN**: Introduces a switch dispatch label: `case common::CUDADataAttr::Shared:`.
  **L1218 CN**: 引入一个 switch 分发标签：`case common::CUDADataAttr::Shared:`。
- **L1219 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1219 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1220 EN**: Continues logic associated with callable symbol `Say`.
  **L1220 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Object '%s' with ATTRIBUTES(SHARED) may not be allocatable, pointer, or target"_err_en_US,`.
  **L1221 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Object '%s' with ATTRIBUTES(SHARED) may not be allocatable, pointer, or target"_err_en_US,`。
- **L1222 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1222 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1223 EN**: Transitions from the previous branch into an `else if` condition.
  **L1223 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1224 EN**: Continues logic associated with callable symbol `Say`.
  **L1224 CN**: 继续与可调用符号 `Say` 相关的逻辑。

### Lines 1225-1248

````cpp
            "Object '%s' with ATTRIBUTES(SHARED) must be declared in a device subprogram"_err_en_US,
            symbol.name());
      }
      break;
    case common::CUDADataAttr::Unified:
      if (((!subpDetails &&
               symbol.owner().kind() != Scope::Kind::MainProgram) ||
              inDeviceSubprogram) &&
          !isComponent) {
        messages_.Say(
            "Object '%s' with ATTRIBUTES(UNIFIED) must be declared in a host subprogram"_err_en_US,
            symbol.name());
      }
      break;
    case common::CUDADataAttr::Texture:
      messages_.Say(
          "ATTRIBUTES(TEXTURE) is obsolete and no longer supported"_err_en_US);
      break;
    case common::CUDADataAttr::UseDevice:
      break;
    }
    // CUDADataAttr::UseDevice is not user-spellable; it is set internally on
    // construct-scoped symbol copies created for OpenACC `host_data
    // use_device(...)` operands so that later passes can resolve them to the
````
- **L1225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Object '%s' with ATTRIBUTES(SHARED) must be declared in a device subprogram"_err_en_US,`.
  **L1225 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Object '%s' with ATTRIBUTES(SHARED) must be declared in a device subprogram"_err_en_US,`。
- **L1226 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1226 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1227 EN**: Closes the current lexical scope or compound statement.
  **L1227 CN**: 结束当前词法作用域或复合语句块。
- **L1228 EN**: Exits the nearest loop or switch statement.
  **L1228 CN**: 退出最近的循环或 switch 语句。
- **L1229 EN**: Introduces a switch dispatch label: `case common::CUDADataAttr::Unified:`.
  **L1229 CN**: 引入一个 switch 分发标签：`case common::CUDADataAttr::Unified:`。
- **L1230 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1230 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1231 EN**: Continues logic associated with callable symbol `owner`.
  **L1231 CN**: 继续与可调用符号 `owner` 相关的逻辑。
- **L1232 EN**: Continues the surrounding expression or declaration: `inDeviceSubprogram) &&`.
  **L1232 CN**: 继续构造周围的表达式或声明：`inDeviceSubprogram) &&`。
- **L1233 EN**: Continues the surrounding expression or declaration: `!isComponent) {`.
  **L1233 CN**: 继续构造周围的表达式或声明：`!isComponent) {`。
- **L1234 EN**: Continues logic associated with callable symbol `Say`.
  **L1234 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Object '%s' with ATTRIBUTES(UNIFIED) must be declared in a host subprogram"_err_en_US,`.
  **L1235 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Object '%s' with ATTRIBUTES(UNIFIED) must be declared in a host subprogram"_err_en_US,`。
- **L1236 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1236 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1237 EN**: Closes the current lexical scope or compound statement.
  **L1237 CN**: 结束当前词法作用域或复合语句块。
- **L1238 EN**: Exits the nearest loop or switch statement.
  **L1238 CN**: 退出最近的循环或 switch 语句。
- **L1239 EN**: Introduces a switch dispatch label: `case common::CUDADataAttr::Texture:`.
  **L1239 CN**: 引入一个 switch 分发标签：`case common::CUDADataAttr::Texture:`。
- **L1240 EN**: Continues logic associated with callable symbol `Say`.
  **L1240 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1241 EN**: Executes a call or declaration centered on `"ATTRIBUTES`.
  **L1241 CN**: 执行以 `"ATTRIBUTES` 为核心的调用或声明。
- **L1242 EN**: Exits the nearest loop or switch statement.
  **L1242 CN**: 退出最近的循环或 switch 语句。
- **L1243 EN**: Introduces a switch dispatch label: `case common::CUDADataAttr::UseDevice:`.
  **L1243 CN**: 引入一个 switch 分发标签：`case common::CUDADataAttr::UseDevice:`。
- **L1244 EN**: Exits the nearest loop or switch statement.
  **L1244 CN**: 退出最近的循环或 switch 语句。
- **L1245 EN**: Closes the current lexical scope or compound statement.
  **L1245 CN**: 结束当前词法作用域或复合语句块。
- **L1246 EN**: Comment explains nearby logic, intent, or metadata: `CUDADataAttr::UseDevice is not user-spellable; it is set internally on`.
  **L1246 CN**: 注释说明附近代码的逻辑、意图或元数据：`CUDADataAttr::UseDevice is not user-spellable; it is set internally on`。
- **L1247 EN**: Comment explains nearby logic, intent, or metadata: `construct-scoped symbol copies created for OpenACC `host_data`.
  **L1247 CN**: 注释说明附近代码的逻辑、意图或元数据：`construct-scoped symbol copies created for OpenACC `host_data`。
- **L1248 EN**: Comment explains nearby logic, intent, or metadata: `use_device(...)` operands so that later passes can resolve them to the`.
  **L1248 CN**: 注释说明附近代码的逻辑、意图或元数据：`use_device(...)` operands so that later passes can resolve them to the`。

### Lines 1249-1272

````cpp
    // device address. The original symbol that actually lives in COMMON or an
    // equivalence group carries no CUDA attribute, so the CUDA Fortran
    // restrictions on user-written ATTRIBUTES(...) do not apply to it.
    if (attr != common::CUDADataAttr::Pinned &&
        attr != common::CUDADataAttr::UseDevice) {
      if (details.commonBlock()) {
        messages_.Say(
            "Object '%s' with ATTRIBUTES(%s) may not be in COMMON"_err_en_US,
            symbol.name(),
            parser::ToUpperCaseLetters(common::EnumToString(attr)));
      } else if (FindEquivalenceSet(symbol)) {
        messages_.Say(
            "Object '%s' with ATTRIBUTES(%s) may not be in an equivalence group"_err_en_US,
            symbol.name(),
            parser::ToUpperCaseLetters(common::EnumToString(attr)));
      }
    }
    if (subpDetails /* not a module variable */ && IsSaved(symbol) &&
        !inDeviceSubprogram && !IsAllocatable(symbol) &&
        attr == common::CUDADataAttr::Device) {
      messages_.Say(
          "Saved object '%s' in host code may not have ATTRIBUTES(DEVICE) unless allocatable"_err_en_US,
          symbol.name(),
          parser::ToUpperCaseLetters(common::EnumToString(attr)));
````
- **L1249 EN**: Comment explains nearby logic, intent, or metadata: `device address. The original symbol that actually lives in COMMON or an`.
  **L1249 CN**: 注释说明附近代码的逻辑、意图或元数据：`device address. The original symbol that actually lives in COMMON or an`。
- **L1250 EN**: Comment explains nearby logic, intent, or metadata: `equivalence group carries no CUDA attribute, so the CUDA Fortran`.
  **L1250 CN**: 注释说明附近代码的逻辑、意图或元数据：`equivalence group carries no CUDA attribute, so the CUDA Fortran`。
- **L1251 EN**: Comment explains nearby logic, intent, or metadata: `restrictions on user-written ATTRIBUTES(...) do not apply to it.`.
  **L1251 CN**: 注释说明附近代码的逻辑、意图或元数据：`restrictions on user-written ATTRIBUTES(...) do not apply to it.`。
- **L1252 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1252 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1253 EN**: Continues the surrounding expression or declaration: `attr != common::CUDADataAttr::UseDevice) {`.
  **L1253 CN**: 继续构造周围的表达式或声明：`attr != common::CUDADataAttr::UseDevice) {`。
- **L1254 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1254 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1255 EN**: Continues logic associated with callable symbol `Say`.
  **L1255 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Object '%s' with ATTRIBUTES(%s) may not be in COMMON"_err_en_US,`.
  **L1256 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Object '%s' with ATTRIBUTES(%s) may not be in COMMON"_err_en_US,`。
- **L1257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol.name(),`.
  **L1257 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol.name(),`。
- **L1258 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L1258 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L1259 EN**: Transitions from the previous branch into an `else if` condition.
  **L1259 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1260 EN**: Continues logic associated with callable symbol `Say`.
  **L1260 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Object '%s' with ATTRIBUTES(%s) may not be in an equivalence group"_err_en_US,`.
  **L1261 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Object '%s' with ATTRIBUTES(%s) may not be in an equivalence group"_err_en_US,`。
- **L1262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol.name(),`.
  **L1262 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol.name(),`。
- **L1263 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L1263 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L1264 EN**: Closes the current lexical scope or compound statement.
  **L1264 CN**: 结束当前词法作用域或复合语句块。
- **L1265 EN**: Closes the current lexical scope or compound statement.
  **L1265 CN**: 结束当前词法作用域或复合语句块。
- **L1266 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1266 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1267 EN**: Continues logic associated with callable symbol `IsAllocatable`.
  **L1267 CN**: 继续与可调用符号 `IsAllocatable` 相关的逻辑。
- **L1268 EN**: Continues the surrounding expression or declaration: `attr == common::CUDADataAttr::Device) {`.
  **L1268 CN**: 继续构造周围的表达式或声明：`attr == common::CUDADataAttr::Device) {`。
- **L1269 EN**: Continues logic associated with callable symbol `Say`.
  **L1269 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Saved object '%s' in host code may not have ATTRIBUTES(DEVICE) unless allocatable"_err_en_US,`.
  **L1270 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Saved object '%s' in host code may not have ATTRIBUTES(DEVICE) unless allocatable"_err_en_US,`。
- **L1271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol.name(),`.
  **L1271 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol.name(),`。
- **L1272 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L1272 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。

### Lines 1273-1296

````cpp
    }
    if (isComponent) {
      if (attr == common::CUDADataAttr::Device) {
        const DeclTypeSpec *type{symbol.GetType()};
        if (const DerivedTypeSpec *
            derived{type ? type->AsDerived() : nullptr}) {
          DirectComponentIterator directs{*derived};
          if (auto iter{std::find_if(directs.begin(), directs.end(),
                  [](const Symbol &) { return false; })}) {
            messages_.Say(
                "Derived type component '%s' may not have ATTRIBUTES(DEVICE) as it has a direct device component '%s'"_err_en_US,
                symbol.name(), iter.BuildResultDesignatorName());
          }
        }
      } else if (attr == common::CUDADataAttr::Constant ||
          attr == common::CUDADataAttr::Shared) {
        messages_.Say(
            "Derived type component '%s' may not have ATTRIBUTES(%s)"_err_en_US,
            symbol.name(),
            parser::ToUpperCaseLetters(common::EnumToString(attr)));
      }
    } else if (!subpDetails && symbol.owner().kind() != Scope::Kind::Module &&
        symbol.owner().kind() != Scope::Kind::MainProgram &&
        symbol.owner().kind() != Scope::Kind::BlockConstruct &&
````
- **L1273 EN**: Closes the current lexical scope or compound statement.
  **L1273 CN**: 结束当前词法作用域或复合语句块。
- **L1274 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1274 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1275 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1275 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1276 EN**: Executes a call or declaration centered on `*type{symbol.GetType`.
  **L1276 CN**: 执行以 `*type{symbol.GetType` 为核心的调用或声明。
- **L1277 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1277 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1278 EN**: Starts a function, method, lambda, or structured scope: `derived{type ? type->AsDerived() : nullptr}) {`.
  **L1278 CN**: 开始一个函数、方法、lambda 或结构化作用域：`derived{type ? type->AsDerived() : nullptr}) {`。
- **L1279 EN**: Executes a standalone statement or declaration: `DirectComponentIterator directs{*derived};`.
  **L1279 CN**: 执行一条独立语句或声明：`DirectComponentIterator directs{*derived};`。
- **L1280 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1280 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1281 EN**: Starts a function, method, lambda, or structured scope: `[](const Symbol &) { return false; })}) {`.
  **L1281 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[](const Symbol &) { return false; })}) {`。
- **L1282 EN**: Continues logic associated with callable symbol `Say`.
  **L1282 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Derived type component '%s' may not have ATTRIBUTES(DEVICE) as it has a direct device component '%s'"_err_en_US,`.
  **L1283 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Derived type component '%s' may not have ATTRIBUTES(DEVICE) as it has a direct device component '%s'"_err_en_US,`。
- **L1284 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1284 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1285 EN**: Closes the current lexical scope or compound statement.
  **L1285 CN**: 结束当前词法作用域或复合语句块。
- **L1286 EN**: Closes the current lexical scope or compound statement.
  **L1286 CN**: 结束当前词法作用域或复合语句块。
- **L1287 EN**: Transitions from the previous branch into an `else if` condition.
  **L1287 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1288 EN**: Continues the surrounding expression or declaration: `attr == common::CUDADataAttr::Shared) {`.
  **L1288 CN**: 继续构造周围的表达式或声明：`attr == common::CUDADataAttr::Shared) {`。
- **L1289 EN**: Continues logic associated with callable symbol `Say`.
  **L1289 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Derived type component '%s' may not have ATTRIBUTES(%s)"_err_en_US,`.
  **L1290 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Derived type component '%s' may not have ATTRIBUTES(%s)"_err_en_US,`。
- **L1291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol.name(),`.
  **L1291 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol.name(),`。
- **L1292 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L1292 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L1293 EN**: Closes the current lexical scope or compound statement.
  **L1293 CN**: 结束当前词法作用域或复合语句块。
- **L1294 EN**: Transitions from the previous branch into an `else if` condition.
  **L1294 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1295 EN**: Continues logic associated with callable symbol `owner`.
  **L1295 CN**: 继续与可调用符号 `owner` 相关的逻辑。
- **L1296 EN**: Continues logic associated with callable symbol `owner`.
  **L1296 CN**: 继续与可调用符号 `owner` 相关的逻辑。

### Lines 1297-1320

````cpp
        symbol.owner().kind() != Scope::Kind::OpenACCConstruct) {
      messages_.Say(
          "ATTRIBUTES(%s) may apply only to module, host subprogram, block, or device subprogram data"_err_en_US,
          parser::ToUpperCaseLetters(common::EnumToString(attr)));
    }
  }

  if (derived && derived->IsVectorType()) {
    CHECK(type);
    std::string typeName{type->AsFortran()};
    if (IsAssumedShape(symbol)) {
      SayWithDeclaration(symbol,
          "Assumed-shape entity of %s type is not supported"_err_en_US,
          typeName);
    } else if (IsDeferredShape(symbol)) {
      SayWithDeclaration(symbol,
          "Deferred-shape entity of %s type is not supported"_err_en_US,
          typeName);
    } else if (IsAssumedRank(symbol)) {
      SayWithDeclaration(symbol,
          "Assumed rank entity of %s type is not supported"_err_en_US,
          typeName);
    }
  }
````
- **L1297 EN**: Starts a function, method, lambda, or structured scope: `symbol.owner().kind() != Scope::Kind::OpenACCConstruct) {`.
  **L1297 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.owner().kind() != Scope::Kind::OpenACCConstruct) {`。
- **L1298 EN**: Continues logic associated with callable symbol `Say`.
  **L1298 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ATTRIBUTES(%s) may apply only to module, host subprogram, block, or device subprogram data"_err_en_US,`.
  **L1299 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ATTRIBUTES(%s) may apply only to module, host subprogram, block, or device subprogram data"_err_en_US,`。
- **L1300 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L1300 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L1301 EN**: Closes the current lexical scope or compound statement.
  **L1301 CN**: 结束当前词法作用域或复合语句块。
- **L1302 EN**: Closes the current lexical scope or compound statement.
  **L1302 CN**: 结束当前词法作用域或复合语句块。
- **L1303 EN**: Blank line separating nearby declarations or logic blocks.
  **L1303 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1304 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1304 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1305 EN**: Executes a call or declaration centered on `CHECK`.
  **L1305 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L1306 EN**: Executes a call or declaration centered on `typeName{type->AsFortran`.
  **L1306 CN**: 执行以 `typeName{type->AsFortran` 为核心的调用或声明。
- **L1307 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1307 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(symbol,`.
  **L1308 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(symbol,`。
- **L1309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-shape entity of %s type is not supported"_err_en_US,`.
  **L1309 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-shape entity of %s type is not supported"_err_en_US,`。
- **L1310 EN**: Executes a standalone statement or declaration: `typeName);`.
  **L1310 CN**: 执行一条独立语句或声明：`typeName);`。
- **L1311 EN**: Transitions from the previous branch into an `else if` condition.
  **L1311 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(symbol,`.
  **L1312 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(symbol,`。
- **L1313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Deferred-shape entity of %s type is not supported"_err_en_US,`.
  **L1313 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Deferred-shape entity of %s type is not supported"_err_en_US,`。
- **L1314 EN**: Executes a standalone statement or declaration: `typeName);`.
  **L1314 CN**: 执行一条独立语句或声明：`typeName);`。
- **L1315 EN**: Transitions from the previous branch into an `else if` condition.
  **L1315 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(symbol,`.
  **L1316 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(symbol,`。
- **L1317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed rank entity of %s type is not supported"_err_en_US,`.
  **L1317 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed rank entity of %s type is not supported"_err_en_US,`。
- **L1318 EN**: Executes a standalone statement or declaration: `typeName);`.
  **L1318 CN**: 执行一条独立语句或声明：`typeName);`。
- **L1319 EN**: Closes the current lexical scope or compound statement.
  **L1319 CN**: 结束当前词法作用域或复合语句块。
- **L1320 EN**: Closes the current lexical scope or compound statement.
  **L1320 CN**: 结束当前词法作用域或复合语句块。

### Lines 1321-1344

````cpp
}

void CheckHelper::CheckPointerInitialization(const Symbol &symbol) {
  if (IsPointer(symbol) && !context_.HasError(symbol) &&
      !scopeIsUninstantiatedPDT_) {
    if (const auto *object{symbol.detailsIf<ObjectEntityDetails>()}) {
      if (object->init()) { // C764, C765; C808
        if (auto designator{evaluate::AsGenericExpr(symbol)}) {
          auto restorer{messages_.SetLocation(symbol.name())};
          context_.set_location(symbol.name());
          CheckInitialDataPointerTarget(
              context_, *designator, *object->init(), DEREF(scope_));
        }
      }
    } else if (const auto *proc{symbol.detailsIf<ProcEntityDetails>()}) {
      if (proc->init() && *proc->init()) {
        // C1519 - must be nonelemental external or module procedure,
        // or an unrestricted specific intrinsic function.
        const Symbol &local{DEREF(*proc->init())};
        const Symbol &ultimate{local.GetUltimate()};
        bool checkTarget{true};
        if (ultimate.attrs().test(Attr::INTRINSIC)) {
          if (auto intrinsic{context_.intrinsics().IsSpecificIntrinsicFunction(
                  ultimate.name().ToString())};
````
- **L1321 EN**: Closes the current lexical scope or compound statement.
  **L1321 CN**: 结束当前词法作用域或复合语句块。
- **L1322 EN**: Blank line separating nearby declarations or logic blocks.
  **L1322 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1323 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckPointerInitialization(const Symbol &symbol) {`.
  **L1323 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckPointerInitialization(const Symbol &symbol) {`。
- **L1324 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1324 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1325 EN**: Continues the surrounding expression or declaration: `!scopeIsUninstantiatedPDT_) {`.
  **L1325 CN**: 继续构造周围的表达式或声明：`!scopeIsUninstantiatedPDT_) {`。
- **L1326 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1326 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1327 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1327 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1328 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1328 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1329 EN**: Executes a call or declaration centered on `restorer{messages_.SetLocation`.
  **L1329 CN**: 执行以 `restorer{messages_.SetLocation` 为核心的调用或声明。
- **L1330 EN**: Executes a call or declaration centered on `context_.set_location`.
  **L1330 CN**: 执行以 `context_.set_location` 为核心的调用或声明。
- **L1331 EN**: Continues logic associated with callable symbol `CheckInitialDataPointerTarget`.
  **L1331 CN**: 继续与可调用符号 `CheckInitialDataPointerTarget` 相关的逻辑。
- **L1332 EN**: Executes a call or declaration centered on `*object->init`.
  **L1332 CN**: 执行以 `*object->init` 为核心的调用或声明。
- **L1333 EN**: Closes the current lexical scope or compound statement.
  **L1333 CN**: 结束当前词法作用域或复合语句块。
- **L1334 EN**: Closes the current lexical scope or compound statement.
  **L1334 CN**: 结束当前词法作用域或复合语句块。
- **L1335 EN**: Transitions from the previous branch into an `else if` condition.
  **L1335 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1336 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1336 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1337 EN**: Comment explains nearby logic, intent, or metadata: `C1519 - must be nonelemental external or module procedure,`.
  **L1337 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1519 - must be nonelemental external or module procedure,`。
- **L1338 EN**: Comment explains nearby logic, intent, or metadata: `or an unrestricted specific intrinsic function.`.
  **L1338 CN**: 注释说明附近代码的逻辑、意图或元数据：`or an unrestricted specific intrinsic function.`。
- **L1339 EN**: Executes a call or declaration centered on `&local{DEREF`.
  **L1339 CN**: 执行以 `&local{DEREF` 为核心的调用或声明。
- **L1340 EN**: Executes a call or declaration centered on `&ultimate{local.GetUltimate`.
  **L1340 CN**: 执行以 `&ultimate{local.GetUltimate` 为核心的调用或声明。
- **L1341 EN**: Executes a standalone statement or declaration: `bool checkTarget{true};`.
  **L1341 CN**: 执行一条独立语句或声明：`bool checkTarget{true};`。
- **L1342 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1342 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1343 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1343 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1344 EN**: Executes a call or declaration centered on `ultimate.name`.
  **L1344 CN**: 执行以 `ultimate.name` 为核心的调用或声明。

### Lines 1345-1368

````cpp
              !intrinsic || intrinsic->isRestrictedSpecific) { // C1030
            context_.Say(
                "Intrinsic procedure '%s' is not an unrestricted specific "
                "intrinsic permitted for use as the initializer for procedure "
                "pointer '%s'"_err_en_US,
                ultimate.name(), symbol.name());
            checkTarget = false;
          }
        } else if (!(ultimate.attrs().test(Attr::EXTERNAL) ||
                       ultimate.owner().kind() == Scope::Kind::Module ||
                       ultimate.owner().IsTopLevel()) ||
            IsDummy(ultimate) || IsPointer(ultimate)) {
          context_.Say(
              "Procedure pointer '%s' initializer '%s' is neither an external nor a module procedure"_err_en_US,
              symbol.name(), ultimate.name());
          checkTarget = false;
        } else if (IsElementalProcedure(ultimate)) {
          context_.Say("Procedure pointer '%s' cannot be initialized with the "
                       "elemental procedure '%s'"_err_en_US,
              symbol.name(), ultimate.name());
          checkTarget = false;
        }
        if (checkTarget) {
          SomeExpr lhs{evaluate::ProcedureDesignator{symbol}};
````
- **L1345 EN**: Continues the surrounding expression or declaration: `!intrinsic || intrinsic->isRestrictedSpecific) { // C1030`.
  **L1345 CN**: 继续构造周围的表达式或声明：`!intrinsic || intrinsic->isRestrictedSpecific) { // C1030`。
- **L1346 EN**: Continues logic associated with callable symbol `Say`.
  **L1346 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1347 EN**: Continues the surrounding expression or declaration: `"Intrinsic procedure '%s' is not an unrestricted specific "`.
  **L1347 CN**: 继续构造周围的表达式或声明：`"Intrinsic procedure '%s' is not an unrestricted specific "`。
- **L1348 EN**: Continues the surrounding expression or declaration: `"intrinsic permitted for use as the initializer for procedure "`.
  **L1348 CN**: 继续构造周围的表达式或声明：`"intrinsic permitted for use as the initializer for procedure "`。
- **L1349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"pointer '%s'"_err_en_US,`.
  **L1349 CN**: 继续一个多行参数列表、初始化器或聚合项：`"pointer '%s'"_err_en_US,`。
- **L1350 EN**: Executes a call or declaration centered on `ultimate.name`.
  **L1350 CN**: 执行以 `ultimate.name` 为核心的调用或声明。
- **L1351 EN**: Executes a standalone statement or declaration: `checkTarget = false;`.
  **L1351 CN**: 执行一条独立语句或声明：`checkTarget = false;`。
- **L1352 EN**: Closes the current lexical scope or compound statement.
  **L1352 CN**: 结束当前词法作用域或复合语句块。
- **L1353 EN**: Transitions from the previous branch into an `else if` condition.
  **L1353 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1354 EN**: Continues logic associated with callable symbol `owner`.
  **L1354 CN**: 继续与可调用符号 `owner` 相关的逻辑。
- **L1355 EN**: Continues logic associated with callable symbol `owner`.
  **L1355 CN**: 继续与可调用符号 `owner` 相关的逻辑。
- **L1356 EN**: Starts a function, method, lambda, or structured scope: `IsDummy(ultimate) || IsPointer(ultimate)) {`.
  **L1356 CN**: 开始一个函数、方法、lambda 或结构化作用域：`IsDummy(ultimate) || IsPointer(ultimate)) {`。
- **L1357 EN**: Continues logic associated with callable symbol `Say`.
  **L1357 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Procedure pointer '%s' initializer '%s' is neither an external nor a module procedure"_err_en_US,`.
  **L1358 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Procedure pointer '%s' initializer '%s' is neither an external nor a module procedure"_err_en_US,`。
- **L1359 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1359 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1360 EN**: Executes a standalone statement or declaration: `checkTarget = false;`.
  **L1360 CN**: 执行一条独立语句或声明：`checkTarget = false;`。
- **L1361 EN**: Transitions from the previous branch into an `else if` condition.
  **L1361 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1362 EN**: Continues logic associated with callable symbol `Say`.
  **L1362 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"elemental procedure '%s'"_err_en_US,`.
  **L1363 CN**: 继续一个多行参数列表、初始化器或聚合项：`"elemental procedure '%s'"_err_en_US,`。
- **L1364 EN**: Executes a call or declaration centered on `symbol.name`.
  **L1364 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L1365 EN**: Executes a standalone statement or declaration: `checkTarget = false;`.
  **L1365 CN**: 执行一条独立语句或声明：`checkTarget = false;`。
- **L1366 EN**: Closes the current lexical scope or compound statement.
  **L1366 CN**: 结束当前词法作用域或复合语句块。
- **L1367 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1367 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1368 EN**: Executes a standalone statement or declaration: `SomeExpr lhs{evaluate::ProcedureDesignator{symbol}};`.
  **L1368 CN**: 执行一条独立语句或声明：`SomeExpr lhs{evaluate::ProcedureDesignator{symbol}};`。

### Lines 1369-1392

````cpp
          SomeExpr rhs{evaluate::ProcedureDesignator{**proc->init()}};
          CheckPointerAssignment(context_, lhs, rhs,
              GetProgramUnitOrBlockConstructContaining(symbol),
              /*isBoundsRemapping=*/false, /*isAssumedRank=*/false);
        }
      }
    }
  }
}

// The six different kinds of array-specs:
//   array-spec     -> explicit-shape-list | deferred-shape-list
//                     | assumed-shape-list | implied-shape-list
//                     | assumed-size | assumed-rank
//   explicit-shape -> [ lb : ] ub
//   deferred-shape -> :
//   assumed-shape  -> [ lb ] :
//   implied-shape  -> [ lb : ] *
//   assumed-size   -> [ explicit-shape-list , ] [ lb : ] *
//   assumed-rank   -> ..
// Note:
// - deferred-shape is also an assumed-shape
// - A single "*" or "lb:*" might be assumed-size or implied-shape-list
void CheckHelper::CheckArraySpec(
````
- **L1369 EN**: Executes a call or declaration centered on `rhs{evaluate::ProcedureDesignator{**proc->init`.
  **L1369 CN**: 执行以 `rhs{evaluate::ProcedureDesignator{**proc->init` 为核心的调用或声明。
- **L1370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckPointerAssignment(context_, lhs, rhs,`.
  **L1370 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckPointerAssignment(context_, lhs, rhs,`。
- **L1371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `GetProgramUnitOrBlockConstructContaining(symbol),`.
  **L1371 CN**: 继续一个多行参数列表、初始化器或聚合项：`GetProgramUnitOrBlockConstructContaining(symbol),`。
- **L1372 EN**: Comment explains nearby logic, intent, or metadata: `isBoundsRemapping=*/false, /*isAssumedRank=*/false);`.
  **L1372 CN**: 注释说明附近代码的逻辑、意图或元数据：`isBoundsRemapping=*/false, /*isAssumedRank=*/false);`。
- **L1373 EN**: Closes the current lexical scope or compound statement.
  **L1373 CN**: 结束当前词法作用域或复合语句块。
- **L1374 EN**: Closes the current lexical scope or compound statement.
  **L1374 CN**: 结束当前词法作用域或复合语句块。
- **L1375 EN**: Closes the current lexical scope or compound statement.
  **L1375 CN**: 结束当前词法作用域或复合语句块。
- **L1376 EN**: Closes the current lexical scope or compound statement.
  **L1376 CN**: 结束当前词法作用域或复合语句块。
- **L1377 EN**: Closes the current lexical scope or compound statement.
  **L1377 CN**: 结束当前词法作用域或复合语句块。
- **L1378 EN**: Blank line separating nearby declarations or logic blocks.
  **L1378 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1379 EN**: Comment explains nearby logic, intent, or metadata: `The six different kinds of array-specs:`.
  **L1379 CN**: 注释说明附近代码的逻辑、意图或元数据：`The six different kinds of array-specs:`。
- **L1380 EN**: Comment explains nearby logic, intent, or metadata: `array-spec     -> explicit-shape-list | deferred-shape-list`.
  **L1380 CN**: 注释说明附近代码的逻辑、意图或元数据：`array-spec     -> explicit-shape-list | deferred-shape-list`。
- **L1381 EN**: Comment explains nearby logic, intent, or metadata: `| assumed-shape-list | implied-shape-list`.
  **L1381 CN**: 注释说明附近代码的逻辑、意图或元数据：`| assumed-shape-list | implied-shape-list`。
- **L1382 EN**: Comment explains nearby logic, intent, or metadata: `| assumed-size | assumed-rank`.
  **L1382 CN**: 注释说明附近代码的逻辑、意图或元数据：`| assumed-size | assumed-rank`。
- **L1383 EN**: Comment explains nearby logic, intent, or metadata: `explicit-shape -> [ lb : ] ub`.
  **L1383 CN**: 注释说明附近代码的逻辑、意图或元数据：`explicit-shape -> [ lb : ] ub`。
- **L1384 EN**: Comment explains nearby logic, intent, or metadata: `deferred-shape -> :`.
  **L1384 CN**: 注释说明附近代码的逻辑、意图或元数据：`deferred-shape -> :`。
- **L1385 EN**: Comment explains nearby logic, intent, or metadata: `assumed-shape  -> [ lb ] :`.
  **L1385 CN**: 注释说明附近代码的逻辑、意图或元数据：`assumed-shape  -> [ lb ] :`。
- **L1386 EN**: Comment explains nearby logic, intent, or metadata: `implied-shape  -> [ lb : ]`.
  **L1386 CN**: 注释说明附近代码的逻辑、意图或元数据：`implied-shape  -> [ lb : ]`。
- **L1387 EN**: Comment explains nearby logic, intent, or metadata: `assumed-size   -> [ explicit-shape-list , ] [ lb : ]`.
  **L1387 CN**: 注释说明附近代码的逻辑、意图或元数据：`assumed-size   -> [ explicit-shape-list , ] [ lb : ]`。
- **L1388 EN**: Comment explains nearby logic, intent, or metadata: `assumed-rank   -> ..`.
  **L1388 CN**: 注释说明附近代码的逻辑、意图或元数据：`assumed-rank   -> ..`。
- **L1389 EN**: Comment explains nearby logic, intent, or metadata: `Note:`.
  **L1389 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note:`。
- **L1390 EN**: Comment explains nearby logic, intent, or metadata: `- deferred-shape is also an assumed-shape`.
  **L1390 CN**: 注释说明附近代码的逻辑、意图或元数据：`- deferred-shape is also an assumed-shape`。
- **L1391 EN**: Comment explains nearby logic, intent, or metadata: `- A single "*" or "lb:*" might be assumed-size or implied-shape-list`.
  **L1391 CN**: 注释说明附近代码的逻辑、意图或元数据：`- A single "*" or "lb:*" might be assumed-size or implied-shape-list`。
- **L1392 EN**: Continues logic associated with callable symbol `CheckArraySpec`.
  **L1392 CN**: 继续与可调用符号 `CheckArraySpec` 相关的逻辑。

### Lines 1393-1416

````cpp
    const Symbol &symbol, const ArraySpec &arraySpec) {
  if (arraySpec.Rank() == 0) {
    return;
  }
  bool isExplicit{arraySpec.IsExplicitShape()};
  bool canBeDeferred{arraySpec.CanBeDeferredShape()};
  bool canBeImplied{arraySpec.CanBeImpliedShape()};
  bool canBeAssumedShape{arraySpec.CanBeAssumedShape()};
  bool canBeAssumedSize{arraySpec.CanBeAssumedSize()};
  bool isAssumedRank{arraySpec.IsAssumedRank()};
  bool isCUDAShared{
      GetCUDADataAttr(&symbol).value_or(common::CUDADataAttr::Device) ==
      common::CUDADataAttr::Shared};
  bool isCrayPointee{symbol.test(Symbol::Flag::CrayPointee)};
  std::optional<parser::MessageFixedText> msg;
  if (isCrayPointee && !isExplicit && !canBeAssumedSize) {
    msg =
        "Cray pointee '%s' must have explicit shape or assumed size"_err_en_US;
  } else if (IsAllocatableOrPointer(symbol) && !canBeDeferred &&
      !isAssumedRank) {
    if (symbol.owner().IsDerivedType()) { // C745
      if (IsAllocatable(symbol)) {
        msg = "Allocatable array component '%s' must have"
              " deferred shape"_err_en_US;
````
- **L1393 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, const ArraySpec &arraySpec) {`.
  **L1393 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, const ArraySpec &arraySpec) {`。
- **L1394 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1394 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1395 EN**: Returns from the current function with `void`.
  **L1395 CN**: 以 `void` 从当前函数返回。
- **L1396 EN**: Closes the current lexical scope or compound statement.
  **L1396 CN**: 结束当前词法作用域或复合语句块。
- **L1397 EN**: Executes a call or declaration centered on `isExplicit{arraySpec.IsExplicitShape`.
  **L1397 CN**: 执行以 `isExplicit{arraySpec.IsExplicitShape` 为核心的调用或声明。
- **L1398 EN**: Executes a call or declaration centered on `canBeDeferred{arraySpec.CanBeDeferredShape`.
  **L1398 CN**: 执行以 `canBeDeferred{arraySpec.CanBeDeferredShape` 为核心的调用或声明。
- **L1399 EN**: Executes a call or declaration centered on `canBeImplied{arraySpec.CanBeImpliedShape`.
  **L1399 CN**: 执行以 `canBeImplied{arraySpec.CanBeImpliedShape` 为核心的调用或声明。
- **L1400 EN**: Executes a call or declaration centered on `canBeAssumedShape{arraySpec.CanBeAssumedShape`.
  **L1400 CN**: 执行以 `canBeAssumedShape{arraySpec.CanBeAssumedShape` 为核心的调用或声明。
- **L1401 EN**: Executes a call or declaration centered on `canBeAssumedSize{arraySpec.CanBeAssumedSize`.
  **L1401 CN**: 执行以 `canBeAssumedSize{arraySpec.CanBeAssumedSize` 为核心的调用或声明。
- **L1402 EN**: Executes a call or declaration centered on `isAssumedRank{arraySpec.IsAssumedRank`.
  **L1402 CN**: 执行以 `isAssumedRank{arraySpec.IsAssumedRank` 为核心的调用或声明。
- **L1403 EN**: Continues the surrounding expression or declaration: `bool isCUDAShared{`.
  **L1403 CN**: 继续构造周围的表达式或声明：`bool isCUDAShared{`。
- **L1404 EN**: Continues logic associated with callable symbol `GetCUDADataAttr`.
  **L1404 CN**: 继续与可调用符号 `GetCUDADataAttr` 相关的逻辑。
- **L1405 EN**: Executes a standalone statement or declaration: `common::CUDADataAttr::Shared};`.
  **L1405 CN**: 执行一条独立语句或声明：`common::CUDADataAttr::Shared};`。
- **L1406 EN**: Executes a call or declaration centered on `isCrayPointee{symbol.test`.
  **L1406 CN**: 执行以 `isCrayPointee{symbol.test` 为核心的调用或声明。
- **L1407 EN**: Executes a standalone statement or declaration: `std::optional<parser::MessageFixedText> msg;`.
  **L1407 CN**: 执行一条独立语句或声明：`std::optional<parser::MessageFixedText> msg;`。
- **L1408 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1408 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1409 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L1409 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L1410 EN**: Executes a standalone statement or declaration: `"Cray pointee '%s' must have explicit shape or assumed size"_err_en_US;`.
  **L1410 CN**: 执行一条独立语句或声明：`"Cray pointee '%s' must have explicit shape or assumed size"_err_en_US;`。
- **L1411 EN**: Transitions from the previous branch into an `else if` condition.
  **L1411 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1412 EN**: Continues the surrounding expression or declaration: `!isAssumedRank) {`.
  **L1412 CN**: 继续构造周围的表达式或声明：`!isAssumedRank) {`。
- **L1413 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1413 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1414 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1414 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1415 EN**: Continues the surrounding expression or declaration: `msg = "Allocatable array component '%s' must have"`.
  **L1415 CN**: 继续构造周围的表达式或声明：`msg = "Allocatable array component '%s' must have"`。
- **L1416 EN**: Executes a standalone statement or declaration: `" deferred shape"_err_en_US;`.
  **L1416 CN**: 执行一条独立语句或声明：`" deferred shape"_err_en_US;`。

### Lines 1417-1440

````cpp
      } else {
        msg = "Array pointer component '%s' must have deferred shape"_err_en_US;
      }
    } else {
      if (IsAllocatable(symbol)) { // C832
        msg = "Allocatable array '%s' must have deferred shape or"
              " assumed rank"_err_en_US;
      } else {
        msg = "Array pointer '%s' must have deferred shape or"
              " assumed rank"_err_en_US;
      }
    }
  } else if (IsDummy(symbol)) {
    if (canBeImplied && !canBeAssumedSize) { // C836
      msg = "Dummy array argument '%s' may not have implied shape"_err_en_US;
    }
  } else if (canBeAssumedShape && !canBeDeferred) {
    msg = "Assumed-shape array '%s' must be a dummy argument"_err_en_US;
  } else if (isAssumedRank) { // C837
    msg = "Assumed-rank array '%s' must be a dummy argument"_err_en_US;
  } else if (canBeAssumedSize && !canBeImplied && !isCUDAShared &&
      !isCrayPointee) { // C833
    msg = "Assumed-size array '%s' must be a dummy argument"_err_en_US;
  } else if (canBeImplied) {
````
- **L1417 EN**: Transitions from the previous branch into the alternative path.
  **L1417 CN**: 从前一个分支过渡到备选路径。
- **L1418 EN**: Executes a standalone statement or declaration: `msg = "Array pointer component '%s' must have deferred shape"_err_en_US;`.
  **L1418 CN**: 执行一条独立语句或声明：`msg = "Array pointer component '%s' must have deferred shape"_err_en_US;`。
- **L1419 EN**: Closes the current lexical scope or compound statement.
  **L1419 CN**: 结束当前词法作用域或复合语句块。
- **L1420 EN**: Transitions from the previous branch into the alternative path.
  **L1420 CN**: 从前一个分支过渡到备选路径。
- **L1421 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1421 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1422 EN**: Continues the surrounding expression or declaration: `msg = "Allocatable array '%s' must have deferred shape or"`.
  **L1422 CN**: 继续构造周围的表达式或声明：`msg = "Allocatable array '%s' must have deferred shape or"`。
- **L1423 EN**: Executes a standalone statement or declaration: `" assumed rank"_err_en_US;`.
  **L1423 CN**: 执行一条独立语句或声明：`" assumed rank"_err_en_US;`。
- **L1424 EN**: Transitions from the previous branch into the alternative path.
  **L1424 CN**: 从前一个分支过渡到备选路径。
- **L1425 EN**: Continues the surrounding expression or declaration: `msg = "Array pointer '%s' must have deferred shape or"`.
  **L1425 CN**: 继续构造周围的表达式或声明：`msg = "Array pointer '%s' must have deferred shape or"`。
- **L1426 EN**: Executes a standalone statement or declaration: `" assumed rank"_err_en_US;`.
  **L1426 CN**: 执行一条独立语句或声明：`" assumed rank"_err_en_US;`。
- **L1427 EN**: Closes the current lexical scope or compound statement.
  **L1427 CN**: 结束当前词法作用域或复合语句块。
- **L1428 EN**: Closes the current lexical scope or compound statement.
  **L1428 CN**: 结束当前词法作用域或复合语句块。
- **L1429 EN**: Transitions from the previous branch into an `else if` condition.
  **L1429 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1430 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1430 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1431 EN**: Executes a standalone statement or declaration: `msg = "Dummy array argument '%s' may not have implied shape"_err_en_US;`.
  **L1431 CN**: 执行一条独立语句或声明：`msg = "Dummy array argument '%s' may not have implied shape"_err_en_US;`。
- **L1432 EN**: Closes the current lexical scope or compound statement.
  **L1432 CN**: 结束当前词法作用域或复合语句块。
- **L1433 EN**: Transitions from the previous branch into an `else if` condition.
  **L1433 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1434 EN**: Executes a standalone statement or declaration: `msg = "Assumed-shape array '%s' must be a dummy argument"_err_en_US;`.
  **L1434 CN**: 执行一条独立语句或声明：`msg = "Assumed-shape array '%s' must be a dummy argument"_err_en_US;`。
- **L1435 EN**: Transitions from the previous branch into an `else if` condition.
  **L1435 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1436 EN**: Executes a standalone statement or declaration: `msg = "Assumed-rank array '%s' must be a dummy argument"_err_en_US;`.
  **L1436 CN**: 执行一条独立语句或声明：`msg = "Assumed-rank array '%s' must be a dummy argument"_err_en_US;`。
- **L1437 EN**: Transitions from the previous branch into an `else if` condition.
  **L1437 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1438 EN**: Continues the surrounding expression or declaration: `!isCrayPointee) { // C833`.
  **L1438 CN**: 继续构造周围的表达式或声明：`!isCrayPointee) { // C833`。
- **L1439 EN**: Executes a standalone statement or declaration: `msg = "Assumed-size array '%s' must be a dummy argument"_err_en_US;`.
  **L1439 CN**: 执行一条独立语句或声明：`msg = "Assumed-size array '%s' must be a dummy argument"_err_en_US;`。
- **L1440 EN**: Transitions from the previous branch into an `else if` condition.
  **L1440 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 1441-1464

````cpp
    if (!IsNamedConstant(symbol) && !isCUDAShared &&
        !isCrayPointee) { // C835, C836
      msg = "Implied-shape array '%s' must be a named constant or a "
            "dummy argument"_err_en_US;
    }
  } else if (IsNamedConstant(symbol)) {
    if (!isExplicit && !canBeImplied) {
      msg = "Named constant '%s' array must have constant or"
            " implied shape"_err_en_US;
    }
  } else if (!isExplicit &&
      !(IsAllocatableOrPointer(symbol) || isCrayPointee)) {
    if (symbol.owner().IsDerivedType()) { // C749
      msg = "Component array '%s' without ALLOCATABLE or POINTER attribute must"
            " have explicit shape"_err_en_US;
    } else { // C816
      msg = "Array '%s' without ALLOCATABLE or POINTER attribute must have"
            " explicit shape"_err_en_US;
    }
  }
  if (msg) {
    context_.Say(std::move(*msg), symbol.name());
  }
}
````
- **L1441 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1441 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1442 EN**: Continues the surrounding expression or declaration: `!isCrayPointee) { // C835, C836`.
  **L1442 CN**: 继续构造周围的表达式或声明：`!isCrayPointee) { // C835, C836`。
- **L1443 EN**: Continues the surrounding expression or declaration: `msg = "Implied-shape array '%s' must be a named constant or a "`.
  **L1443 CN**: 继续构造周围的表达式或声明：`msg = "Implied-shape array '%s' must be a named constant or a "`。
- **L1444 EN**: Executes a standalone statement or declaration: `"dummy argument"_err_en_US;`.
  **L1444 CN**: 执行一条独立语句或声明：`"dummy argument"_err_en_US;`。
- **L1445 EN**: Closes the current lexical scope or compound statement.
  **L1445 CN**: 结束当前词法作用域或复合语句块。
- **L1446 EN**: Transitions from the previous branch into an `else if` condition.
  **L1446 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1447 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1447 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1448 EN**: Continues the surrounding expression or declaration: `msg = "Named constant '%s' array must have constant or"`.
  **L1448 CN**: 继续构造周围的表达式或声明：`msg = "Named constant '%s' array must have constant or"`。
- **L1449 EN**: Executes a standalone statement or declaration: `" implied shape"_err_en_US;`.
  **L1449 CN**: 执行一条独立语句或声明：`" implied shape"_err_en_US;`。
- **L1450 EN**: Closes the current lexical scope or compound statement.
  **L1450 CN**: 结束当前词法作用域或复合语句块。
- **L1451 EN**: Transitions from the previous branch into an `else if` condition.
  **L1451 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1452 EN**: Starts a function, method, lambda, or structured scope: `!(IsAllocatableOrPointer(symbol) || isCrayPointee)) {`.
  **L1452 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!(IsAllocatableOrPointer(symbol) || isCrayPointee)) {`。
- **L1453 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1453 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1454 EN**: Continues the surrounding expression or declaration: `msg = "Component array '%s' without ALLOCATABLE or POINTER attribute must"`.
  **L1454 CN**: 继续构造周围的表达式或声明：`msg = "Component array '%s' without ALLOCATABLE or POINTER attribute must"`。
- **L1455 EN**: Executes a standalone statement or declaration: `" have explicit shape"_err_en_US;`.
  **L1455 CN**: 执行一条独立语句或声明：`" have explicit shape"_err_en_US;`。
- **L1456 EN**: Transitions from the previous branch into the alternative path.
  **L1456 CN**: 从前一个分支过渡到备选路径。
- **L1457 EN**: Continues the surrounding expression or declaration: `msg = "Array '%s' without ALLOCATABLE or POINTER attribute must have"`.
  **L1457 CN**: 继续构造周围的表达式或声明：`msg = "Array '%s' without ALLOCATABLE or POINTER attribute must have"`。
- **L1458 EN**: Executes a standalone statement or declaration: `" explicit shape"_err_en_US;`.
  **L1458 CN**: 执行一条独立语句或声明：`" explicit shape"_err_en_US;`。
- **L1459 EN**: Closes the current lexical scope or compound statement.
  **L1459 CN**: 结束当前词法作用域或复合语句块。
- **L1460 EN**: Closes the current lexical scope or compound statement.
  **L1460 CN**: 结束当前词法作用域或复合语句块。
- **L1461 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1461 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1462 EN**: Executes a call or declaration centered on `context_.Say`.
  **L1462 CN**: 执行以 `context_.Say` 为核心的调用或声明。
- **L1463 EN**: Closes the current lexical scope or compound statement.
  **L1463 CN**: 结束当前词法作用域或复合语句块。
- **L1464 EN**: Closes the current lexical scope or compound statement.
  **L1464 CN**: 结束当前词法作用域或复合语句块。

### Lines 1465-1488

````cpp

void CheckHelper::CheckProcEntity(
    const Symbol &symbol, const ProcEntityDetails &details) {
  CheckSymbolType(symbol);
  const Symbol *interface{details.procInterface()};
  if (details.isDummy()) {
    if (!symbol.attrs().test(Attr::POINTER) && // C843
        symbol.attrs().HasAny(
            {Attr::INTENT_IN, Attr::INTENT_OUT, Attr::INTENT_INOUT})) {
      messages_.Say("A dummy procedure without the POINTER attribute"
                    " may not have an INTENT attribute"_err_en_US);
    }
    if (InElemental()) { // C15100
      messages_.Say(
          "An ELEMENTAL subprogram may not have a dummy procedure"_err_en_US);
    }
    if (interface && IsElementalProcedure(*interface)) {
      // There's no explicit constraint or "shall" that we can find in the
      // standard for this check, but it seems to be implied in multiple
      // sites, and ELEMENTAL non-intrinsic actual arguments *are*
      // explicitly forbidden.  But we allow "PROCEDURE(SIN)::dummy"
      // because it is explicitly legal to *pass* the specific intrinsic
      // function SIN as an actual argument.
      if (interface->attrs().test(Attr::INTRINSIC)) {
````
- **L1465 EN**: Blank line separating nearby declarations or logic blocks.
  **L1465 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1466 EN**: Continues logic associated with callable symbol `CheckProcEntity`.
  **L1466 CN**: 继续与可调用符号 `CheckProcEntity` 相关的逻辑。
- **L1467 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, const ProcEntityDetails &details) {`.
  **L1467 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, const ProcEntityDetails &details) {`。
- **L1468 EN**: Executes a call or declaration centered on `CheckSymbolType`.
  **L1468 CN**: 执行以 `CheckSymbolType` 为核心的调用或声明。
- **L1469 EN**: Executes a call or declaration centered on `*interface{details.procInterface`.
  **L1469 CN**: 执行以 `*interface{details.procInterface` 为核心的调用或声明。
- **L1470 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1470 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1471 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1471 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1472 EN**: Continues logic associated with callable symbol `attrs`.
  **L1472 CN**: 继续与可调用符号 `attrs` 相关的逻辑。
- **L1473 EN**: Continues the surrounding expression or declaration: `{Attr::INTENT_IN, Attr::INTENT_OUT, Attr::INTENT_INOUT})) {`.
  **L1473 CN**: 继续构造周围的表达式或声明：`{Attr::INTENT_IN, Attr::INTENT_OUT, Attr::INTENT_INOUT})) {`。
- **L1474 EN**: Continues logic associated with callable symbol `Say`.
  **L1474 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1475 EN**: Executes a standalone statement or declaration: `" may not have an INTENT attribute"_err_en_US);`.
  **L1475 CN**: 执行一条独立语句或声明：`" may not have an INTENT attribute"_err_en_US);`。
- **L1476 EN**: Closes the current lexical scope or compound statement.
  **L1476 CN**: 结束当前词法作用域或复合语句块。
- **L1477 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1477 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1478 EN**: Continues logic associated with callable symbol `Say`.
  **L1478 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1479 EN**: Executes a standalone statement or declaration: `"An ELEMENTAL subprogram may not have a dummy procedure"_err_en_US);`.
  **L1479 CN**: 执行一条独立语句或声明：`"An ELEMENTAL subprogram may not have a dummy procedure"_err_en_US);`。
- **L1480 EN**: Closes the current lexical scope or compound statement.
  **L1480 CN**: 结束当前词法作用域或复合语句块。
- **L1481 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1481 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1482 EN**: Comment explains nearby logic, intent, or metadata: `There's no explicit constraint or "shall" that we can find in the`.
  **L1482 CN**: 注释说明附近代码的逻辑、意图或元数据：`There's no explicit constraint or "shall" that we can find in the`。
- **L1483 EN**: Comment explains nearby logic, intent, or metadata: `standard for this check, but it seems to be implied in multiple`.
  **L1483 CN**: 注释说明附近代码的逻辑、意图或元数据：`standard for this check, but it seems to be implied in multiple`。
- **L1484 EN**: Comment explains nearby logic, intent, or metadata: `sites, and ELEMENTAL non-intrinsic actual arguments *are`.
  **L1484 CN**: 注释说明附近代码的逻辑、意图或元数据：`sites, and ELEMENTAL non-intrinsic actual arguments *are`。
- **L1485 EN**: Comment explains nearby logic, intent, or metadata: `explicitly forbidden.  But we allow "PROCEDURE(SIN)::dummy"`.
  **L1485 CN**: 注释说明附近代码的逻辑、意图或元数据：`explicitly forbidden.  But we allow "PROCEDURE(SIN)::dummy"`。
- **L1486 EN**: Comment explains nearby logic, intent, or metadata: `because it is explicitly legal to *pass* the specific intrinsic`.
  **L1486 CN**: 注释说明附近代码的逻辑、意图或元数据：`because it is explicitly legal to *pass* the specific intrinsic`。
- **L1487 EN**: Comment explains nearby logic, intent, or metadata: `function SIN as an actual argument.`.
  **L1487 CN**: 注释说明附近代码的逻辑、意图或元数据：`function SIN as an actual argument.`。
- **L1488 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1488 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1489-1512

````cpp
        Warn(common::UsageWarning::Portability,
            "A dummy procedure should not have an ELEMENTAL intrinsic as its interface"_port_en_US);
      } else {
        messages_.Say("A dummy procedure may not be ELEMENTAL"_err_en_US);
      }
    }
  } else if (IsPointer(symbol)) {
    CheckPointerInitialization(symbol);
    if (interface) {
      if (interface->attrs().test(Attr::INTRINSIC)) {
        auto intrinsic{context_.intrinsics().IsSpecificIntrinsicFunction(
            interface->name().ToString())};
        if (!intrinsic || intrinsic->isRestrictedSpecific) { // C1515
          messages_.Say(
              "Intrinsic procedure '%s' is not an unrestricted specific "
              "intrinsic permitted for use as the definition of the interface "
              "to procedure pointer '%s'"_err_en_US,
              interface->name(), symbol.name());
        } else if (IsElementalProcedure(*interface)) {
          Warn(common::UsageWarning::Portability,
              "Procedure pointer '%s' should not have an ELEMENTAL intrinsic as its interface"_port_en_US,
              symbol.name()); // C1517
        }
      } else if (IsElementalProcedure(*interface)) {
````
- **L1489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::Portability,`.
  **L1489 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::Portability,`。
- **L1490 EN**: Executes a standalone statement or declaration: `"A dummy procedure should not have an ELEMENTAL intrinsic as its interface"_port_en_US);`.
  **L1490 CN**: 执行一条独立语句或声明：`"A dummy procedure should not have an ELEMENTAL intrinsic as its interface"_port_en_US);`。
- **L1491 EN**: Transitions from the previous branch into the alternative path.
  **L1491 CN**: 从前一个分支过渡到备选路径。
- **L1492 EN**: Executes a call or declaration centered on `messages_.Say`.
  **L1492 CN**: 执行以 `messages_.Say` 为核心的调用或声明。
- **L1493 EN**: Closes the current lexical scope or compound statement.
  **L1493 CN**: 结束当前词法作用域或复合语句块。
- **L1494 EN**: Closes the current lexical scope or compound statement.
  **L1494 CN**: 结束当前词法作用域或复合语句块。
- **L1495 EN**: Transitions from the previous branch into an `else if` condition.
  **L1495 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1496 EN**: Executes a call or declaration centered on `CheckPointerInitialization`.
  **L1496 CN**: 执行以 `CheckPointerInitialization` 为核心的调用或声明。
- **L1497 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1497 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1498 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1498 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1499 EN**: Continues logic associated with callable symbol `intrinsics`.
  **L1499 CN**: 继续与可调用符号 `intrinsics` 相关的逻辑。
- **L1500 EN**: Executes a call or declaration centered on `interface->name`.
  **L1500 CN**: 执行以 `interface->name` 为核心的调用或声明。
- **L1501 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1501 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1502 EN**: Continues logic associated with callable symbol `Say`.
  **L1502 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1503 EN**: Continues the surrounding expression or declaration: `"Intrinsic procedure '%s' is not an unrestricted specific "`.
  **L1503 CN**: 继续构造周围的表达式或声明：`"Intrinsic procedure '%s' is not an unrestricted specific "`。
- **L1504 EN**: Continues the surrounding expression or declaration: `"intrinsic permitted for use as the definition of the interface "`.
  **L1504 CN**: 继续构造周围的表达式或声明：`"intrinsic permitted for use as the definition of the interface "`。
- **L1505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"to procedure pointer '%s'"_err_en_US,`.
  **L1505 CN**: 继续一个多行参数列表、初始化器或聚合项：`"to procedure pointer '%s'"_err_en_US,`。
- **L1506 EN**: Executes a call or declaration centered on `interface->name`.
  **L1506 CN**: 执行以 `interface->name` 为核心的调用或声明。
- **L1507 EN**: Transitions from the previous branch into an `else if` condition.
  **L1507 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::Portability,`.
  **L1508 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::Portability,`。
- **L1509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Procedure pointer '%s' should not have an ELEMENTAL intrinsic as its interface"_port_en_US,`.
  **L1509 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Procedure pointer '%s' should not have an ELEMENTAL intrinsic as its interface"_port_en_US,`。
- **L1510 EN**: Continues logic associated with callable symbol `name`.
  **L1510 CN**: 继续与可调用符号 `name` 相关的逻辑。
- **L1511 EN**: Closes the current lexical scope or compound statement.
  **L1511 CN**: 结束当前词法作用域或复合语句块。
- **L1512 EN**: Transitions from the previous branch into an `else if` condition.
  **L1512 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 1513-1536

````cpp
        messages_.Say("Procedure pointer '%s' may not be ELEMENTAL"_err_en_US,
            symbol.name()); // C1517
      }
    }
    if (symbol.owner().IsDerivedType()) {
      CheckPassArg(symbol, interface, details);
    }
  } else if (symbol.owner().IsDerivedType()) {
    const auto &name{symbol.name()};
    messages_.Say(name,
        "Procedure component '%s' must have POINTER attribute"_err_en_US, name);
  }
  CheckExternal(symbol);
}

// When a module subprogram has the MODULE prefix the following must match
// with the corresponding separate module procedure interface body:
// - C1549: characteristics and dummy argument names
// - C1550: binding label
// - C1551: NON_RECURSIVE prefix
class SubprogramMatchHelper {
public:
  explicit SubprogramMatchHelper(CheckHelper &checkHelper)
      : checkHelper{checkHelper} {}
````
- **L1513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say("Procedure pointer '%s' may not be ELEMENTAL"_err_en_US,`.
  **L1513 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say("Procedure pointer '%s' may not be ELEMENTAL"_err_en_US,`。
- **L1514 EN**: Continues logic associated with callable symbol `name`.
  **L1514 CN**: 继续与可调用符号 `name` 相关的逻辑。
- **L1515 EN**: Closes the current lexical scope or compound statement.
  **L1515 CN**: 结束当前词法作用域或复合语句块。
- **L1516 EN**: Closes the current lexical scope or compound statement.
  **L1516 CN**: 结束当前词法作用域或复合语句块。
- **L1517 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1517 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1518 EN**: Executes a call or declaration centered on `CheckPassArg`.
  **L1518 CN**: 执行以 `CheckPassArg` 为核心的调用或声明。
- **L1519 EN**: Closes the current lexical scope or compound statement.
  **L1519 CN**: 结束当前词法作用域或复合语句块。
- **L1520 EN**: Transitions from the previous branch into an `else if` condition.
  **L1520 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1521 EN**: Executes a call or declaration centered on `&name{symbol.name`.
  **L1521 CN**: 执行以 `&name{symbol.name` 为核心的调用或声明。
- **L1522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(name,`.
  **L1522 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(name,`。
- **L1523 EN**: Executes a standalone statement or declaration: `"Procedure component '%s' must have POINTER attribute"_err_en_US, name);`.
  **L1523 CN**: 执行一条独立语句或声明：`"Procedure component '%s' must have POINTER attribute"_err_en_US, name);`。
- **L1524 EN**: Closes the current lexical scope or compound statement.
  **L1524 CN**: 结束当前词法作用域或复合语句块。
- **L1525 EN**: Executes a call or declaration centered on `CheckExternal`.
  **L1525 CN**: 执行以 `CheckExternal` 为核心的调用或声明。
- **L1526 EN**: Closes the current lexical scope or compound statement.
  **L1526 CN**: 结束当前词法作用域或复合语句块。
- **L1527 EN**: Blank line separating nearby declarations or logic blocks.
  **L1527 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1528 EN**: Comment explains nearby logic, intent, or metadata: `When a module subprogram has the MODULE prefix the following must match`.
  **L1528 CN**: 注释说明附近代码的逻辑、意图或元数据：`When a module subprogram has the MODULE prefix the following must match`。
- **L1529 EN**: Comment explains nearby logic, intent, or metadata: `with the corresponding separate module procedure interface body:`.
  **L1529 CN**: 注释说明附近代码的逻辑、意图或元数据：`with the corresponding separate module procedure interface body:`。
- **L1530 EN**: Comment explains nearby logic, intent, or metadata: `- C1549: characteristics and dummy argument names`.
  **L1530 CN**: 注释说明附近代码的逻辑、意图或元数据：`- C1549: characteristics and dummy argument names`。
- **L1531 EN**: Comment explains nearby logic, intent, or metadata: `- C1550: binding label`.
  **L1531 CN**: 注释说明附近代码的逻辑、意图或元数据：`- C1550: binding label`。
- **L1532 EN**: Comment explains nearby logic, intent, or metadata: `- C1551: NON_RECURSIVE prefix`.
  **L1532 CN**: 注释说明附近代码的逻辑、意图或元数据：`- C1551: NON_RECURSIVE prefix`。
- **L1533 EN**: Declares class `SubprogramMatchHelper`.
  **L1533 CN**: 声明 class `SubprogramMatchHelper`。
- **L1534 EN**: Sets the following members to `public` access.
  **L1534 CN**: 将后续成员的访问级别设为 `public`。
- **L1535 EN**: Continues logic associated with callable symbol `SubprogramMatchHelper`.
  **L1535 CN**: 继续与可调用符号 `SubprogramMatchHelper` 相关的逻辑。
- **L1536 EN**: Continues the surrounding expression or declaration: `: checkHelper{checkHelper} {}`.
  **L1536 CN**: 继续构造周围的表达式或声明：`: checkHelper{checkHelper} {}`。

### Lines 1537-1560

````cpp

  void Check(const Symbol &, const Symbol &);

private:
  SemanticsContext &context() { return checkHelper.context(); }
  void CheckDummyArg(const Symbol &, const Symbol &, const DummyArgument &,
      const DummyArgument &);
  void CheckDummyDataObject(const Symbol &, const Symbol &,
      const DummyDataObject &, const DummyDataObject &);
  void CheckDummyProcedure(const Symbol &, const Symbol &,
      const DummyProcedure &, const DummyProcedure &);
  bool CheckSameIntent(
      const Symbol &, const Symbol &, common::Intent, common::Intent);
  template <typename... A>
  void Say(
      const Symbol &, const Symbol &, parser::MessageFixedText &&, A &&...);
  template <typename ATTRS>
  bool CheckSameAttrs(const Symbol &, const Symbol &, ATTRS, ATTRS);
  bool ShapesAreCompatible(const DummyDataObject &, const DummyDataObject &);
  evaluate::Shape FoldShape(const evaluate::Shape &);
  std::optional<evaluate::Shape> FoldShape(
      const std::optional<evaluate::Shape> &shape) {
    if (shape) {
      return FoldShape(*shape);
````
- **L1537 EN**: Blank line separating nearby declarations or logic blocks.
  **L1537 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1538 EN**: Executes a call or declaration centered on `Check`.
  **L1538 CN**: 执行以 `Check` 为核心的调用或声明。
- **L1539 EN**: Blank line separating nearby declarations or logic blocks.
  **L1539 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1540 EN**: Sets the following members to `private` access.
  **L1540 CN**: 将后续成员的访问级别设为 `private`。
- **L1541 EN**: Continues logic associated with callable symbol `context`.
  **L1541 CN**: 继续与可调用符号 `context` 相关的逻辑。
- **L1542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckDummyArg(const Symbol &, const Symbol &, const DummyArgument &,`.
  **L1542 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckDummyArg(const Symbol &, const Symbol &, const DummyArgument &,`。
- **L1543 EN**: Executes a standalone statement or declaration: `const DummyArgument &);`.
  **L1543 CN**: 执行一条独立语句或声明：`const DummyArgument &);`。
- **L1544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckDummyDataObject(const Symbol &, const Symbol &,`.
  **L1544 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckDummyDataObject(const Symbol &, const Symbol &,`。
- **L1545 EN**: Executes a standalone statement or declaration: `const DummyDataObject &, const DummyDataObject &);`.
  **L1545 CN**: 执行一条独立语句或声明：`const DummyDataObject &, const DummyDataObject &);`。
- **L1546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckDummyProcedure(const Symbol &, const Symbol &,`.
  **L1546 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckDummyProcedure(const Symbol &, const Symbol &,`。
- **L1547 EN**: Executes a standalone statement or declaration: `const DummyProcedure &, const DummyProcedure &);`.
  **L1547 CN**: 执行一条独立语句或声明：`const DummyProcedure &, const DummyProcedure &);`。
- **L1548 EN**: Continues logic associated with callable symbol `CheckSameIntent`.
  **L1548 CN**: 继续与可调用符号 `CheckSameIntent` 相关的逻辑。
- **L1549 EN**: Executes a standalone statement or declaration: `const Symbol &, const Symbol &, common::Intent, common::Intent);`.
  **L1549 CN**: 执行一条独立语句或声明：`const Symbol &, const Symbol &, common::Intent, common::Intent);`。
- **L1550 EN**: Introduces template parameters or specialization context: `template <typename... A>`.
  **L1550 CN**: 为后续声明引入模板参数或特化上下文：`template <typename... A>`。
- **L1551 EN**: Continues logic associated with callable symbol `Say`.
  **L1551 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1552 EN**: Executes a standalone statement or declaration: `const Symbol &, const Symbol &, parser::MessageFixedText &&, A &&...);`.
  **L1552 CN**: 执行一条独立语句或声明：`const Symbol &, const Symbol &, parser::MessageFixedText &&, A &&...);`。
- **L1553 EN**: Introduces template parameters or specialization context: `template <typename ATTRS>`.
  **L1553 CN**: 为后续声明引入模板参数或特化上下文：`template <typename ATTRS>`。
- **L1554 EN**: Executes a call or declaration centered on `CheckSameAttrs`.
  **L1554 CN**: 执行以 `CheckSameAttrs` 为核心的调用或声明。
- **L1555 EN**: Executes a call or declaration centered on `ShapesAreCompatible`.
  **L1555 CN**: 执行以 `ShapesAreCompatible` 为核心的调用或声明。
- **L1556 EN**: Executes a call or declaration centered on `FoldShape`.
  **L1556 CN**: 执行以 `FoldShape` 为核心的调用或声明。
- **L1557 EN**: Continues logic associated with callable symbol `FoldShape`.
  **L1557 CN**: 继续与可调用符号 `FoldShape` 相关的逻辑。
- **L1558 EN**: Continues the surrounding expression or declaration: `const std::optional<evaluate::Shape> &shape) {`.
  **L1558 CN**: 继续构造周围的表达式或声明：`const std::optional<evaluate::Shape> &shape) {`。
- **L1559 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1559 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1560 EN**: Returns from the current function with `FoldShape(*shape)`.
  **L1560 CN**: 以 `FoldShape(*shape)` 从当前函数返回。

### Lines 1561-1584

````cpp
    }
    return std::nullopt;
  }
  std::string AsFortran(DummyDataObject::Attr attr) {
    return parser::ToUpperCaseLetters(DummyDataObject::EnumToString(attr));
  }
  std::string AsFortran(DummyProcedure::Attr attr) {
    return parser::ToUpperCaseLetters(DummyProcedure::EnumToString(attr));
  }

  CheckHelper &checkHelper;
};

// 15.6.2.6 para 3 - can the result of an ENTRY differ from its function?
bool CheckHelper::IsResultOkToDiffer(const FunctionResult &result) {
  if (result.attrs.test(FunctionResult::Attr::Allocatable) ||
      result.attrs.test(FunctionResult::Attr::Pointer)) {
    return false;
  }
  const auto *typeAndShape{result.GetTypeAndShape()};
  if (!typeAndShape || typeAndShape->Rank() != 0) {
    return false;
  }
  auto category{typeAndShape->type().category()};
````
- **L1561 EN**: Closes the current lexical scope or compound statement.
  **L1561 CN**: 结束当前词法作用域或复合语句块。
- **L1562 EN**: Returns from the current function with `std::nullopt`.
  **L1562 CN**: 以 `std::nullopt` 从当前函数返回。
- **L1563 EN**: Closes the current lexical scope or compound statement.
  **L1563 CN**: 结束当前词法作用域或复合语句块。
- **L1564 EN**: Starts a function, method, lambda, or structured scope: `std::string AsFortran(DummyDataObject::Attr attr) {`.
  **L1564 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::string AsFortran(DummyDataObject::Attr attr) {`。
- **L1565 EN**: Returns from the current function with `parser::ToUpperCaseLetters(DummyDataObject::EnumToString(attr))`.
  **L1565 CN**: 以 `parser::ToUpperCaseLetters(DummyDataObject::EnumToString(attr))` 从当前函数返回。
- **L1566 EN**: Closes the current lexical scope or compound statement.
  **L1566 CN**: 结束当前词法作用域或复合语句块。
- **L1567 EN**: Starts a function, method, lambda, or structured scope: `std::string AsFortran(DummyProcedure::Attr attr) {`.
  **L1567 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::string AsFortran(DummyProcedure::Attr attr) {`。
- **L1568 EN**: Returns from the current function with `parser::ToUpperCaseLetters(DummyProcedure::EnumToString(attr))`.
  **L1568 CN**: 以 `parser::ToUpperCaseLetters(DummyProcedure::EnumToString(attr))` 从当前函数返回。
- **L1569 EN**: Closes the current lexical scope or compound statement.
  **L1569 CN**: 结束当前词法作用域或复合语句块。
- **L1570 EN**: Blank line separating nearby declarations or logic blocks.
  **L1570 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1571 EN**: Executes a standalone statement or declaration: `CheckHelper &checkHelper;`.
  **L1571 CN**: 执行一条独立语句或声明：`CheckHelper &checkHelper;`。
- **L1572 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1572 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1573 EN**: Blank line separating nearby declarations or logic blocks.
  **L1573 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1574 EN**: Comment explains nearby logic, intent, or metadata: `15.6.2.6 para 3 - can the result of an ENTRY differ from its function?`.
  **L1574 CN**: 注释说明附近代码的逻辑、意图或元数据：`15.6.2.6 para 3 - can the result of an ENTRY differ from its function?`。
- **L1575 EN**: Starts a function, method, lambda, or structured scope: `bool CheckHelper::IsResultOkToDiffer(const FunctionResult &result) {`.
  **L1575 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool CheckHelper::IsResultOkToDiffer(const FunctionResult &result) {`。
- **L1576 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1576 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1577 EN**: Starts a function, method, lambda, or structured scope: `result.attrs.test(FunctionResult::Attr::Pointer)) {`.
  **L1577 CN**: 开始一个函数、方法、lambda 或结构化作用域：`result.attrs.test(FunctionResult::Attr::Pointer)) {`。
- **L1578 EN**: Returns from the current function with `false`.
  **L1578 CN**: 以 `false` 从当前函数返回。
- **L1579 EN**: Closes the current lexical scope or compound statement.
  **L1579 CN**: 结束当前词法作用域或复合语句块。
- **L1580 EN**: Executes a call or declaration centered on `*typeAndShape{result.GetTypeAndShape`.
  **L1580 CN**: 执行以 `*typeAndShape{result.GetTypeAndShape` 为核心的调用或声明。
- **L1581 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1581 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1582 EN**: Returns from the current function with `false`.
  **L1582 CN**: 以 `false` 从当前函数返回。
- **L1583 EN**: Closes the current lexical scope or compound statement.
  **L1583 CN**: 结束当前词法作用域或复合语句块。
- **L1584 EN**: Executes a call or declaration centered on `category{typeAndShape->type`.
  **L1584 CN**: 执行以 `category{typeAndShape->type` 为核心的调用或声明。

### Lines 1585-1608

````cpp
  if (category == TypeCategory::Character ||
      category == TypeCategory::Derived) {
    return false;
  }
  int kind{typeAndShape->type().kind()};
  return kind == context_.GetDefaultKind(category) ||
      (category == TypeCategory::Real &&
          kind == context_.doublePrecisionKind());
}

void CheckHelper::CheckSubprogram(
    const Symbol &symbol, const SubprogramDetails &details) {
  // Evaluate a procedure definition's characteristics to flush out
  // any errors that analysis might expose, in case this subprogram hasn't
  // had any calls in this compilation unit that would have validated them.
  if (!context_.HasError(symbol) && !details.isDummy() &&
      !details.isInterface() && !details.stmtFunction()) {
    if (!Procedure::Characterize(symbol, foldingContext_)) {
      context_.SetError(symbol);
    }
  }
  // F2023 C1553
  if (symbol.attrs().test(Attr::SIMPLE) && symbol.attrs().test(Attr::IMPURE)) {
    messages_.Say(symbol.name(),
````
- **L1585 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1585 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1586 EN**: Continues the surrounding expression or declaration: `category == TypeCategory::Derived) {`.
  **L1586 CN**: 继续构造周围的表达式或声明：`category == TypeCategory::Derived) {`。
- **L1587 EN**: Returns from the current function with `false`.
  **L1587 CN**: 以 `false` 从当前函数返回。
- **L1588 EN**: Closes the current lexical scope or compound statement.
  **L1588 CN**: 结束当前词法作用域或复合语句块。
- **L1589 EN**: Executes a call or declaration centered on `kind{typeAndShape->type`.
  **L1589 CN**: 执行以 `kind{typeAndShape->type` 为核心的调用或声明。
- **L1590 EN**: Returns from the current function with `kind == context_.GetDefaultKind(category) ||`.
  **L1590 CN**: 以 `kind == context_.GetDefaultKind(category) ||` 从当前函数返回。
- **L1591 EN**: Continues the surrounding expression or declaration: `(category == TypeCategory::Real &&`.
  **L1591 CN**: 继续构造周围的表达式或声明：`(category == TypeCategory::Real &&`。
- **L1592 EN**: Executes a call or declaration centered on `context_.doublePrecisionKind`.
  **L1592 CN**: 执行以 `context_.doublePrecisionKind` 为核心的调用或声明。
- **L1593 EN**: Closes the current lexical scope or compound statement.
  **L1593 CN**: 结束当前词法作用域或复合语句块。
- **L1594 EN**: Blank line separating nearby declarations or logic blocks.
  **L1594 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1595 EN**: Continues logic associated with callable symbol `CheckSubprogram`.
  **L1595 CN**: 继续与可调用符号 `CheckSubprogram` 相关的逻辑。
- **L1596 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, const SubprogramDetails &details) {`.
  **L1596 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, const SubprogramDetails &details) {`。
- **L1597 EN**: Comment explains nearby logic, intent, or metadata: `Evaluate a procedure definition's characteristics to flush out`.
  **L1597 CN**: 注释说明附近代码的逻辑、意图或元数据：`Evaluate a procedure definition's characteristics to flush out`。
- **L1598 EN**: Comment explains nearby logic, intent, or metadata: `any errors that analysis might expose, in case this subprogram hasn't`.
  **L1598 CN**: 注释说明附近代码的逻辑、意图或元数据：`any errors that analysis might expose, in case this subprogram hasn't`。
- **L1599 EN**: Comment explains nearby logic, intent, or metadata: `had any calls in this compilation unit that would have validated them.`.
  **L1599 CN**: 注释说明附近代码的逻辑、意图或元数据：`had any calls in this compilation unit that would have validated them.`。
- **L1600 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1600 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1601 EN**: Starts a function, method, lambda, or structured scope: `!details.isInterface() && !details.stmtFunction()) {`.
  **L1601 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!details.isInterface() && !details.stmtFunction()) {`。
- **L1602 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1602 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1603 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L1603 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L1604 EN**: Closes the current lexical scope or compound statement.
  **L1604 CN**: 结束当前词法作用域或复合语句块。
- **L1605 EN**: Closes the current lexical scope or compound statement.
  **L1605 CN**: 结束当前词法作用域或复合语句块。
- **L1606 EN**: Comment explains nearby logic, intent, or metadata: `F2023 C1553`.
  **L1606 CN**: 注释说明附近代码的逻辑、意图或元数据：`F2023 C1553`。
- **L1607 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1607 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(symbol.name(),`.
  **L1608 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(symbol.name(),`。

### Lines 1609-1632

````cpp
        "A procedure may not have both the SIMPLE and IMPURE attributes"_err_en_US);
    context_.SetError(symbol);
  }
  if (const Symbol *iface{FindSeparateModuleSubprogramInterface(&symbol)}) {
    SubprogramMatchHelper{*this}.Check(symbol, *iface);
  }
  if (const Scope *entryScope{details.entryScope()}) {
    // ENTRY F'2023 15.6.2.6
    std::optional<parser::MessageFixedText> error;
    const Symbol *subprogram{entryScope->symbol()};
    const SubprogramDetails *subprogramDetails{nullptr};
    if (subprogram) {
      subprogramDetails = subprogram->detailsIf<SubprogramDetails>();
    }
    if (!(entryScope->parent().IsGlobal() || entryScope->parent().IsModule() ||
            entryScope->parent().IsSubmodule())) {
      error = "ENTRY may not appear in an internal subprogram"_err_en_US;
    } else if (subprogramDetails && details.isFunction() &&
        subprogramDetails->isFunction() &&
        !context_.HasError(details.result()) &&
        !context_.HasError(subprogramDetails->result())) {
      auto result{FunctionResult::Characterize(
          details.result(), context_.foldingContext())};
      auto subpResult{FunctionResult::Characterize(
````
- **L1609 EN**: Executes a standalone statement or declaration: `"A procedure may not have both the SIMPLE and IMPURE attributes"_err_en_US);`.
  **L1609 CN**: 执行一条独立语句或声明：`"A procedure may not have both the SIMPLE and IMPURE attributes"_err_en_US);`。
- **L1610 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L1610 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L1611 EN**: Closes the current lexical scope or compound statement.
  **L1611 CN**: 结束当前词法作用域或复合语句块。
- **L1612 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1612 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1613 EN**: Executes a call or declaration centered on `SubprogramMatchHelper{*this}.Check`.
  **L1613 CN**: 执行以 `SubprogramMatchHelper{*this}.Check` 为核心的调用或声明。
- **L1614 EN**: Closes the current lexical scope or compound statement.
  **L1614 CN**: 结束当前词法作用域或复合语句块。
- **L1615 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1615 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1616 EN**: Comment explains nearby logic, intent, or metadata: `ENTRY F'2023 15.6.2.6`.
  **L1616 CN**: 注释说明附近代码的逻辑、意图或元数据：`ENTRY F'2023 15.6.2.6`。
- **L1617 EN**: Executes a standalone statement or declaration: `std::optional<parser::MessageFixedText> error;`.
  **L1617 CN**: 执行一条独立语句或声明：`std::optional<parser::MessageFixedText> error;`。
- **L1618 EN**: Executes a call or declaration centered on `*subprogram{entryScope->symbol`.
  **L1618 CN**: 执行以 `*subprogram{entryScope->symbol` 为核心的调用或声明。
- **L1619 EN**: Executes a standalone statement or declaration: `const SubprogramDetails *subprogramDetails{nullptr};`.
  **L1619 CN**: 执行一条独立语句或声明：`const SubprogramDetails *subprogramDetails{nullptr};`。
- **L1620 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1620 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1621 EN**: Executes a call or declaration centered on `subprogram->detailsIf<SubprogramDetails>`.
  **L1621 CN**: 执行以 `subprogram->detailsIf<SubprogramDetails>` 为核心的调用或声明。
- **L1622 EN**: Closes the current lexical scope or compound statement.
  **L1622 CN**: 结束当前词法作用域或复合语句块。
- **L1623 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1623 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1624 EN**: Starts a function, method, lambda, or structured scope: `entryScope->parent().IsSubmodule())) {`.
  **L1624 CN**: 开始一个函数、方法、lambda 或结构化作用域：`entryScope->parent().IsSubmodule())) {`。
- **L1625 EN**: Executes a standalone statement or declaration: `error = "ENTRY may not appear in an internal subprogram"_err_en_US;`.
  **L1625 CN**: 执行一条独立语句或声明：`error = "ENTRY may not appear in an internal subprogram"_err_en_US;`。
- **L1626 EN**: Transitions from the previous branch into an `else if` condition.
  **L1626 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1627 EN**: Continues logic associated with callable symbol `isFunction`.
  **L1627 CN**: 继续与可调用符号 `isFunction` 相关的逻辑。
- **L1628 EN**: Continues logic associated with callable symbol `HasError`.
  **L1628 CN**: 继续与可调用符号 `HasError` 相关的逻辑。
- **L1629 EN**: Starts a function, method, lambda, or structured scope: `!context_.HasError(subprogramDetails->result())) {`.
  **L1629 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!context_.HasError(subprogramDetails->result())) {`。
- **L1630 EN**: Continues logic associated with callable symbol `Characterize`.
  **L1630 CN**: 继续与可调用符号 `Characterize` 相关的逻辑。
- **L1631 EN**: Executes a call or declaration centered on `details.result`.
  **L1631 CN**: 执行以 `details.result` 为核心的调用或声明。
- **L1632 EN**: Continues logic associated with callable symbol `Characterize`.
  **L1632 CN**: 继续与可调用符号 `Characterize` 相关的逻辑。

### Lines 1633-1656

````cpp
          subprogramDetails->result(), context_.foldingContext())};
      if (result && subpResult && *result != *subpResult &&
          (!IsResultOkToDiffer(*result) || !IsResultOkToDiffer(*subpResult))) {
        error =
            "Result of ENTRY is not compatible with result of containing function"_err_en_US;
      }
    }
    if (error) {
      if (auto *msg{messages_.Say(symbol.name(), *error)}) {
        if (subprogram) {
          msg->Attach(subprogram->name(), "Containing subprogram"_en_US);
        }
      }
    }
  }
  if (details.isFunction() &&
      details.result().name() != symbol.name()) { // F'2023 C1569 & C1583
    if (auto iter{symbol.owner().find(details.result().name())};
        iter != symbol.owner().end()) {
      const Symbol &resNameSym{*iter->second};
      if (const auto *resNameSubp{resNameSym.detailsIf<SubprogramDetails>()}) {
        if (const Scope * resNameEntryScope{resNameSubp->entryScope()}) {
          const Scope *myScope{
              details.entryScope() ? details.entryScope() : symbol.scope()};
````
- **L1633 EN**: Executes a call or declaration centered on `subprogramDetails->result`.
  **L1633 CN**: 执行以 `subprogramDetails->result` 为核心的调用或声明。
- **L1634 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1634 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1635 EN**: Starts a function, method, lambda, or structured scope: `(!IsResultOkToDiffer(*result) || !IsResultOkToDiffer(*subpResult))) {`.
  **L1635 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(!IsResultOkToDiffer(*result) || !IsResultOkToDiffer(*subpResult))) {`。
- **L1636 EN**: Continues the surrounding expression or declaration: `error =`.
  **L1636 CN**: 继续构造周围的表达式或声明：`error =`。
- **L1637 EN**: Executes a standalone statement or declaration: `"Result of ENTRY is not compatible with result of containing function"_err_en_US;`.
  **L1637 CN**: 执行一条独立语句或声明：`"Result of ENTRY is not compatible with result of containing function"_err_en_US;`。
- **L1638 EN**: Closes the current lexical scope or compound statement.
  **L1638 CN**: 结束当前词法作用域或复合语句块。
- **L1639 EN**: Closes the current lexical scope or compound statement.
  **L1639 CN**: 结束当前词法作用域或复合语句块。
- **L1640 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1640 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1641 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1641 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1642 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1642 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1643 EN**: Executes a call or declaration centered on `msg->Attach`.
  **L1643 CN**: 执行以 `msg->Attach` 为核心的调用或声明。
- **L1644 EN**: Closes the current lexical scope or compound statement.
  **L1644 CN**: 结束当前词法作用域或复合语句块。
- **L1645 EN**: Closes the current lexical scope or compound statement.
  **L1645 CN**: 结束当前词法作用域或复合语句块。
- **L1646 EN**: Closes the current lexical scope or compound statement.
  **L1646 CN**: 结束当前词法作用域或复合语句块。
- **L1647 EN**: Closes the current lexical scope or compound statement.
  **L1647 CN**: 结束当前词法作用域或复合语句块。
- **L1648 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1648 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1649 EN**: Continues logic associated with callable symbol `result`.
  **L1649 CN**: 继续与可调用符号 `result` 相关的逻辑。
- **L1650 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1650 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1651 EN**: Starts a function, method, lambda, or structured scope: `iter != symbol.owner().end()) {`.
  **L1651 CN**: 开始一个函数、方法、lambda 或结构化作用域：`iter != symbol.owner().end()) {`。
- **L1652 EN**: Executes a standalone statement or declaration: `const Symbol &resNameSym{*iter->second};`.
  **L1652 CN**: 执行一条独立语句或声明：`const Symbol &resNameSym{*iter->second};`。
- **L1653 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1653 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1654 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1654 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1655 EN**: Continues the surrounding expression or declaration: `const Scope *myScope{`.
  **L1655 CN**: 继续构造周围的表达式或声明：`const Scope *myScope{`。
- **L1656 EN**: Executes a call or declaration centered on `details.entryScope`.
  **L1656 CN**: 执行以 `details.entryScope` 为核心的调用或声明。

### Lines 1657-1680

````cpp
          if (resNameEntryScope == myScope) {
            if (auto *msg{messages_.Say(symbol.name(),
                    "Explicit RESULT('%s') of function '%s' cannot have the same name as a distinct ENTRY into the same scope"_err_en_US,
                    details.result().name(), symbol.name())}) {
              msg->Attach(
                  resNameSym.name(), "ENTRY with conflicting name"_en_US);
            }
          }
        }
      }
    }
  }
  if (const MaybeExpr & stmtFunction{details.stmtFunction()}) {
    if (auto msg{evaluate::CheckStatementFunction(
            symbol, *stmtFunction, context_.foldingContext())}) {
      SayWithDeclaration(symbol, std::move(*msg));
    } else if (IsPointer(symbol)) {
      SayWithDeclaration(symbol,
          "A statement function must not have the POINTER attribute"_err_en_US);
    } else if (details.result().flags().test(Symbol::Flag::Implicit)) {
      // 15.6.4 p2 weird requirement
      if (const Symbol *
          host{symbol.owner().parent().FindSymbol(symbol.name())}) {
        evaluate::AttachDeclaration(
````
- **L1657 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1657 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1658 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1658 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Explicit RESULT('%s') of function '%s' cannot have the same name as a distinct ENTRY into the same scope"_err_en_US,`.
  **L1659 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Explicit RESULT('%s') of function '%s' cannot have the same name as a distinct ENTRY into the same scope"_err_en_US,`。
- **L1660 EN**: Starts a function, method, lambda, or structured scope: `details.result().name(), symbol.name())}) {`.
  **L1660 CN**: 开始一个函数、方法、lambda 或结构化作用域：`details.result().name(), symbol.name())}) {`。
- **L1661 EN**: Continues logic associated with callable symbol `Attach`.
  **L1661 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L1662 EN**: Executes a call or declaration centered on `resNameSym.name`.
  **L1662 CN**: 执行以 `resNameSym.name` 为核心的调用或声明。
- **L1663 EN**: Closes the current lexical scope or compound statement.
  **L1663 CN**: 结束当前词法作用域或复合语句块。
- **L1664 EN**: Closes the current lexical scope or compound statement.
  **L1664 CN**: 结束当前词法作用域或复合语句块。
- **L1665 EN**: Closes the current lexical scope or compound statement.
  **L1665 CN**: 结束当前词法作用域或复合语句块。
- **L1666 EN**: Closes the current lexical scope or compound statement.
  **L1666 CN**: 结束当前词法作用域或复合语句块。
- **L1667 EN**: Closes the current lexical scope or compound statement.
  **L1667 CN**: 结束当前词法作用域或复合语句块。
- **L1668 EN**: Closes the current lexical scope or compound statement.
  **L1668 CN**: 结束当前词法作用域或复合语句块。
- **L1669 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1669 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1670 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1670 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1671 EN**: Starts a function, method, lambda, or structured scope: `symbol, *stmtFunction, context_.foldingContext())}) {`.
  **L1671 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol, *stmtFunction, context_.foldingContext())}) {`。
- **L1672 EN**: Executes a call or declaration centered on `SayWithDeclaration`.
  **L1672 CN**: 执行以 `SayWithDeclaration` 为核心的调用或声明。
- **L1673 EN**: Transitions from the previous branch into an `else if` condition.
  **L1673 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(symbol,`.
  **L1674 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(symbol,`。
- **L1675 EN**: Executes a standalone statement or declaration: `"A statement function must not have the POINTER attribute"_err_en_US);`.
  **L1675 CN**: 执行一条独立语句或声明：`"A statement function must not have the POINTER attribute"_err_en_US);`。
- **L1676 EN**: Transitions from the previous branch into an `else if` condition.
  **L1676 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1677 EN**: Comment explains nearby logic, intent, or metadata: `15.6.4 p2 weird requirement`.
  **L1677 CN**: 注释说明附近代码的逻辑、意图或元数据：`15.6.4 p2 weird requirement`。
- **L1678 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1678 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1679 EN**: Starts a function, method, lambda, or structured scope: `host{symbol.owner().parent().FindSymbol(symbol.name())}) {`.
  **L1679 CN**: 开始一个函数、方法、lambda 或结构化作用域：`host{symbol.owner().parent().FindSymbol(symbol.name())}) {`。
- **L1680 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L1680 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。

### Lines 1681-1704

````cpp
            Warn(common::LanguageFeature::StatementFunctionExtensions,
                symbol.name(),
                "An implicitly typed statement function should not appear when the same symbol is available in its host scope"_port_en_US),
            *host);
      }
    }
    if (GetProgramUnitOrBlockConstructContaining(symbol).kind() ==
        Scope::Kind::BlockConstruct) { // C1107
      messages_.Say(symbol.name(),
          "A statement function definition may not appear in a BLOCK construct"_err_en_US);
    }
  }
  if (IsElementalProcedure(symbol)) {
    // See comment on the similar check in CheckProcEntity()
    if (details.isDummy()) {
      messages_.Say("A dummy procedure may not be ELEMENTAL"_err_en_US);
    } else {
      for (const Symbol *dummy : details.dummyArgs()) {
        if (!dummy) { // C15100
          messages_.Say(
              "An ELEMENTAL subroutine may not have an alternate return dummy argument"_err_en_US);
        }
      }
    }
````
- **L1681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::LanguageFeature::StatementFunctionExtensions,`.
  **L1681 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::LanguageFeature::StatementFunctionExtensions,`。
- **L1682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol.name(),`.
  **L1682 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol.name(),`。
- **L1683 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"An implicitly typed statement function should not appear when the same symbol is available in its host scope"_port_en_US),`.
  **L1683 CN**: 继续一个多行参数列表、初始化器或聚合项：`"An implicitly typed statement function should not appear when the same symbol is available in its host scope"_port_en_US),`。
- **L1684 EN**: Comment explains nearby logic, intent, or metadata: `host);`.
  **L1684 CN**: 注释说明附近代码的逻辑、意图或元数据：`host);`。
- **L1685 EN**: Closes the current lexical scope or compound statement.
  **L1685 CN**: 结束当前词法作用域或复合语句块。
- **L1686 EN**: Closes the current lexical scope or compound statement.
  **L1686 CN**: 结束当前词法作用域或复合语句块。
- **L1687 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1687 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1688 EN**: Continues the surrounding expression or declaration: `Scope::Kind::BlockConstruct) { // C1107`.
  **L1688 CN**: 继续构造周围的表达式或声明：`Scope::Kind::BlockConstruct) { // C1107`。
- **L1689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(symbol.name(),`.
  **L1689 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(symbol.name(),`。
- **L1690 EN**: Executes a standalone statement or declaration: `"A statement function definition may not appear in a BLOCK construct"_err_en_US);`.
  **L1690 CN**: 执行一条独立语句或声明：`"A statement function definition may not appear in a BLOCK construct"_err_en_US);`。
- **L1691 EN**: Closes the current lexical scope or compound statement.
  **L1691 CN**: 结束当前词法作用域或复合语句块。
- **L1692 EN**: Closes the current lexical scope or compound statement.
  **L1692 CN**: 结束当前词法作用域或复合语句块。
- **L1693 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1693 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1694 EN**: Comment explains nearby logic, intent, or metadata: `See comment on the similar check in CheckProcEntity()`.
  **L1694 CN**: 注释说明附近代码的逻辑、意图或元数据：`See comment on the similar check in CheckProcEntity()`。
- **L1695 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1695 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1696 EN**: Executes a call or declaration centered on `messages_.Say`.
  **L1696 CN**: 执行以 `messages_.Say` 为核心的调用或声明。
- **L1697 EN**: Transitions from the previous branch into the alternative path.
  **L1697 CN**: 从前一个分支过渡到备选路径。
- **L1698 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1698 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1699 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1699 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1700 EN**: Continues logic associated with callable symbol `Say`.
  **L1700 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1701 EN**: Executes a standalone statement or declaration: `"An ELEMENTAL subroutine may not have an alternate return dummy argument"_err_en_US);`.
  **L1701 CN**: 执行一条独立语句或声明：`"An ELEMENTAL subroutine may not have an alternate return dummy argument"_err_en_US);`。
- **L1702 EN**: Closes the current lexical scope or compound statement.
  **L1702 CN**: 结束当前词法作用域或复合语句块。
- **L1703 EN**: Closes the current lexical scope or compound statement.
  **L1703 CN**: 结束当前词法作用域或复合语句块。
- **L1704 EN**: Closes the current lexical scope or compound statement.
  **L1704 CN**: 结束当前词法作用域或复合语句块。

### Lines 1705-1728

````cpp
  }
  if (details.isInterface()) {
    if (!details.isDummy() && details.isFunction() &&
        IsAssumedLengthCharacter(details.result())) { // C721
      messages_.Say(details.result().name(),
          "A function interface may not declare an assumed-length CHARACTER(*) result"_err_en_US);
    }
    if (symbol.attrs().test(Attr::ABSTRACT) &&
        (symbol.name() == "integer" || symbol.name() == "unsigned" ||
            symbol.name() == "real" || symbol.name() == "complex" ||
            symbol.name() == "character" ||
            symbol.name() == "logical")) { // F'2023 C1503
      messages_.Say(
          "An ABSTRACT interface may not have the same name as an intrinsic type"_err_en_US);
    }
  }
  CheckExternal(symbol);
  CheckModuleProcedureDef(symbol);
  auto cudaAttrs{details.cudaSubprogramAttrs()};
  if (cudaAttrs &&
      (*cudaAttrs == common::CUDASubprogramAttrs::Global ||
          *cudaAttrs == common::CUDASubprogramAttrs::Grid_Global) &&
      details.isFunction()) {
    messages_.Say(symbol.name(),
````
- **L1705 EN**: Closes the current lexical scope or compound statement.
  **L1705 CN**: 结束当前词法作用域或复合语句块。
- **L1706 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1706 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1707 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1707 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1708 EN**: Continues logic associated with callable symbol `IsAssumedLengthCharacter`.
  **L1708 CN**: 继续与可调用符号 `IsAssumedLengthCharacter` 相关的逻辑。
- **L1709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(details.result().name(),`.
  **L1709 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(details.result().name(),`。
- **L1710 EN**: Executes a call or declaration centered on `CHARACTER`.
  **L1710 CN**: 执行以 `CHARACTER` 为核心的调用或声明。
- **L1711 EN**: Closes the current lexical scope or compound statement.
  **L1711 CN**: 结束当前词法作用域或复合语句块。
- **L1712 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1712 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1713 EN**: Continues logic associated with callable symbol `name`.
  **L1713 CN**: 继续与可调用符号 `name` 相关的逻辑。
- **L1714 EN**: Continues logic associated with callable symbol `name`.
  **L1714 CN**: 继续与可调用符号 `name` 相关的逻辑。
- **L1715 EN**: Continues logic associated with callable symbol `name`.
  **L1715 CN**: 继续与可调用符号 `name` 相关的逻辑。
- **L1716 EN**: Continues logic associated with callable symbol `name`.
  **L1716 CN**: 继续与可调用符号 `name` 相关的逻辑。
- **L1717 EN**: Continues logic associated with callable symbol `Say`.
  **L1717 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1718 EN**: Executes a standalone statement or declaration: `"An ABSTRACT interface may not have the same name as an intrinsic type"_err_en_US);`.
  **L1718 CN**: 执行一条独立语句或声明：`"An ABSTRACT interface may not have the same name as an intrinsic type"_err_en_US);`。
- **L1719 EN**: Closes the current lexical scope or compound statement.
  **L1719 CN**: 结束当前词法作用域或复合语句块。
- **L1720 EN**: Closes the current lexical scope or compound statement.
  **L1720 CN**: 结束当前词法作用域或复合语句块。
- **L1721 EN**: Executes a call or declaration centered on `CheckExternal`.
  **L1721 CN**: 执行以 `CheckExternal` 为核心的调用或声明。
- **L1722 EN**: Executes a call or declaration centered on `CheckModuleProcedureDef`.
  **L1722 CN**: 执行以 `CheckModuleProcedureDef` 为核心的调用或声明。
- **L1723 EN**: Executes a call or declaration centered on `cudaAttrs{details.cudaSubprogramAttrs`.
  **L1723 CN**: 执行以 `cudaAttrs{details.cudaSubprogramAttrs` 为核心的调用或声明。
- **L1724 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1724 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1725 EN**: Continues the surrounding expression or declaration: `(*cudaAttrs == common::CUDASubprogramAttrs::Global ||`.
  **L1725 CN**: 继续构造周围的表达式或声明：`(*cudaAttrs == common::CUDASubprogramAttrs::Global ||`。
- **L1726 EN**: Comment explains nearby logic, intent, or metadata: `cudaAttrs == common::CUDASubprogramAttrs::Grid_Global) &&`.
  **L1726 CN**: 注释说明附近代码的逻辑、意图或元数据：`cudaAttrs == common::CUDASubprogramAttrs::Grid_Global) &&`。
- **L1727 EN**: Starts a function, method, lambda, or structured scope: `details.isFunction()) {`.
  **L1727 CN**: 开始一个函数、方法、lambda 或结构化作用域：`details.isFunction()) {`。
- **L1728 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(symbol.name(),`.
  **L1728 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(symbol.name(),`。

### Lines 1729-1752

````cpp
        "A function may not have ATTRIBUTES(GLOBAL) or ATTRIBUTES(GRID_GLOBAL)"_err_en_US);
  }
  if (cudaAttrs &&
      (*cudaAttrs == common::CUDASubprogramAttrs::Global ||
          *cudaAttrs == common::CUDASubprogramAttrs::Grid_Global) &&
      symbol.attrs().HasAny({Attr::RECURSIVE, Attr::PURE, Attr::ELEMENTAL})) {
    messages_.Say(symbol.name(),
        "A kernel subprogram may not be RECURSIVE, PURE, or ELEMENTAL"_err_en_US);
  }
  if (cudaAttrs && *cudaAttrs != common::CUDASubprogramAttrs::Host) {
    // CUDA device subprogram checks
    if (ClassifyProcedure(symbol) == ProcedureDefinitionClass::Internal) {
      messages_.Say(symbol.name(),
          "A device subprogram may not be an internal subprogram"_err_en_US);
    }
  }
  if ((!details.cudaLaunchBounds().empty() ||
          !details.cudaClusterDims().empty()) &&
      !(cudaAttrs &&
          (*cudaAttrs == common::CUDASubprogramAttrs::Global ||
              *cudaAttrs == common::CUDASubprogramAttrs::Grid_Global))) {
    messages_.Say(symbol.name(),
        "A subroutine may not have LAUNCH_BOUNDS() or CLUSTER_DIMS() unless it has ATTRIBUTES(GLOBAL) or ATTRIBUTES(GRID_GLOBAL)"_err_en_US);
  }
````
- **L1729 EN**: Executes a call or declaration centered on `ATTRIBUTES`.
  **L1729 CN**: 执行以 `ATTRIBUTES` 为核心的调用或声明。
- **L1730 EN**: Closes the current lexical scope or compound statement.
  **L1730 CN**: 结束当前词法作用域或复合语句块。
- **L1731 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1731 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1732 EN**: Continues the surrounding expression or declaration: `(*cudaAttrs == common::CUDASubprogramAttrs::Global ||`.
  **L1732 CN**: 继续构造周围的表达式或声明：`(*cudaAttrs == common::CUDASubprogramAttrs::Global ||`。
- **L1733 EN**: Comment explains nearby logic, intent, or metadata: `cudaAttrs == common::CUDASubprogramAttrs::Grid_Global) &&`.
  **L1733 CN**: 注释说明附近代码的逻辑、意图或元数据：`cudaAttrs == common::CUDASubprogramAttrs::Grid_Global) &&`。
- **L1734 EN**: Starts a function, method, lambda, or structured scope: `symbol.attrs().HasAny({Attr::RECURSIVE, Attr::PURE, Attr::ELEMENTAL})) {`.
  **L1734 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.attrs().HasAny({Attr::RECURSIVE, Attr::PURE, Attr::ELEMENTAL})) {`。
- **L1735 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(symbol.name(),`.
  **L1735 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(symbol.name(),`。
- **L1736 EN**: Executes a standalone statement or declaration: `"A kernel subprogram may not be RECURSIVE, PURE, or ELEMENTAL"_err_en_US);`.
  **L1736 CN**: 执行一条独立语句或声明：`"A kernel subprogram may not be RECURSIVE, PURE, or ELEMENTAL"_err_en_US);`。
- **L1737 EN**: Closes the current lexical scope or compound statement.
  **L1737 CN**: 结束当前词法作用域或复合语句块。
- **L1738 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1738 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1739 EN**: Comment explains nearby logic, intent, or metadata: `CUDA device subprogram checks`.
  **L1739 CN**: 注释说明附近代码的逻辑、意图或元数据：`CUDA device subprogram checks`。
- **L1740 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1740 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1741 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(symbol.name(),`.
  **L1741 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(symbol.name(),`。
- **L1742 EN**: Executes a standalone statement or declaration: `"A device subprogram may not be an internal subprogram"_err_en_US);`.
  **L1742 CN**: 执行一条独立语句或声明：`"A device subprogram may not be an internal subprogram"_err_en_US);`。
- **L1743 EN**: Closes the current lexical scope or compound statement.
  **L1743 CN**: 结束当前词法作用域或复合语句块。
- **L1744 EN**: Closes the current lexical scope or compound statement.
  **L1744 CN**: 结束当前词法作用域或复合语句块。
- **L1745 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1745 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1746 EN**: Continues logic associated with callable symbol `cudaClusterDims`.
  **L1746 CN**: 继续与可调用符号 `cudaClusterDims` 相关的逻辑。
- **L1747 EN**: Continues the surrounding expression or declaration: `!(cudaAttrs &&`.
  **L1747 CN**: 继续构造周围的表达式或声明：`!(cudaAttrs &&`。
- **L1748 EN**: Continues the surrounding expression or declaration: `(*cudaAttrs == common::CUDASubprogramAttrs::Global ||`.
  **L1748 CN**: 继续构造周围的表达式或声明：`(*cudaAttrs == common::CUDASubprogramAttrs::Global ||`。
- **L1749 EN**: Comment explains nearby logic, intent, or metadata: `cudaAttrs == common::CUDASubprogramAttrs::Grid_Global))) {`.
  **L1749 CN**: 注释说明附近代码的逻辑、意图或元数据：`cudaAttrs == common::CUDASubprogramAttrs::Grid_Global))) {`。
- **L1750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(symbol.name(),`.
  **L1750 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(symbol.name(),`。
- **L1751 EN**: Executes a call or declaration centered on `LAUNCH_BOUNDS`.
  **L1751 CN**: 执行以 `LAUNCH_BOUNDS` 为核心的调用或声明。
- **L1752 EN**: Closes the current lexical scope or compound statement.
  **L1752 CN**: 结束当前词法作用域或复合语句块。

### Lines 1753-1776

````cpp
  if (!IsStmtFunction(symbol)) {
    if (const Scope * outerDevice{FindCUDADeviceContext(&symbol.owner())};
        outerDevice && outerDevice->symbol()) {
      if (auto *msg{messages_.Say(symbol.name(),
              "'%s' may not be an internal procedure of CUDA device subprogram '%s'"_err_en_US,
              symbol.name(), outerDevice->symbol()->name())}) {
        msg->Attach(outerDevice->symbol()->name(),
            "Containing CUDA device subprogram"_en_US);
      }
    }
  }
}

void CheckHelper::CheckExternal(const Symbol &symbol) {
  if (IsExternal(symbol)) {
    std::string interfaceName{symbol.name().ToString()};
    if (const auto *bind{symbol.GetBindName()}) {
      interfaceName = *bind;
    }
    if (const Symbol * global{FindGlobal(symbol)};
        global && global != &symbol) {
      std::string definitionName{global->name().ToString()};
      if (const auto *bind{global->GetBindName()}) {
        definitionName = *bind;
````
- **L1753 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1753 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1754 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1754 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1755 EN**: Starts a function, method, lambda, or structured scope: `outerDevice && outerDevice->symbol()) {`.
  **L1755 CN**: 开始一个函数、方法、lambda 或结构化作用域：`outerDevice && outerDevice->symbol()) {`。
- **L1756 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1756 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' may not be an internal procedure of CUDA device subprogram '%s'"_err_en_US,`.
  **L1757 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' may not be an internal procedure of CUDA device subprogram '%s'"_err_en_US,`。
- **L1758 EN**: Starts a function, method, lambda, or structured scope: `symbol.name(), outerDevice->symbol()->name())}) {`.
  **L1758 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.name(), outerDevice->symbol()->name())}) {`。
- **L1759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg->Attach(outerDevice->symbol()->name(),`.
  **L1759 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg->Attach(outerDevice->symbol()->name(),`。
- **L1760 EN**: Executes a standalone statement or declaration: `"Containing CUDA device subprogram"_en_US);`.
  **L1760 CN**: 执行一条独立语句或声明：`"Containing CUDA device subprogram"_en_US);`。
- **L1761 EN**: Closes the current lexical scope or compound statement.
  **L1761 CN**: 结束当前词法作用域或复合语句块。
- **L1762 EN**: Closes the current lexical scope or compound statement.
  **L1762 CN**: 结束当前词法作用域或复合语句块。
- **L1763 EN**: Closes the current lexical scope or compound statement.
  **L1763 CN**: 结束当前词法作用域或复合语句块。
- **L1764 EN**: Closes the current lexical scope or compound statement.
  **L1764 CN**: 结束当前词法作用域或复合语句块。
- **L1765 EN**: Blank line separating nearby declarations or logic blocks.
  **L1765 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1766 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckExternal(const Symbol &symbol) {`.
  **L1766 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckExternal(const Symbol &symbol) {`。
- **L1767 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1767 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1768 EN**: Executes a call or declaration centered on `interfaceName{symbol.name`.
  **L1768 CN**: 执行以 `interfaceName{symbol.name` 为核心的调用或声明。
- **L1769 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1769 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1770 EN**: Executes a standalone statement or declaration: `interfaceName = *bind;`.
  **L1770 CN**: 执行一条独立语句或声明：`interfaceName = *bind;`。
- **L1771 EN**: Closes the current lexical scope or compound statement.
  **L1771 CN**: 结束当前词法作用域或复合语句块。
- **L1772 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1772 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1773 EN**: Continues the surrounding expression or declaration: `global && global != &symbol) {`.
  **L1773 CN**: 继续构造周围的表达式或声明：`global && global != &symbol) {`。
- **L1774 EN**: Executes a call or declaration centered on `definitionName{global->name`.
  **L1774 CN**: 执行以 `definitionName{global->name` 为核心的调用或声明。
- **L1775 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1775 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1776 EN**: Executes a standalone statement or declaration: `definitionName = *bind;`.
  **L1776 CN**: 执行一条独立语句或声明：`definitionName = *bind;`。

### Lines 1777-1800

````cpp
      }
      if (interfaceName == definitionName) {
        parser::Message *msg{nullptr};
        if (!IsProcedure(*global)) {
          if ((symbol.flags().test(Symbol::Flag::Function) ||
                  symbol.flags().test(Symbol::Flag::Subroutine))) {
            msg = Warn(common::UsageWarning::ExternalNameConflict,
                "The global entity '%s' corresponding to the local procedure '%s' is not a callable subprogram"_warn_en_US,
                global->name(), symbol.name());
          }
        } else if (auto chars{Characterize(symbol)}) {
          if (auto globalChars{Characterize(*global)}) {
            if (chars->HasExplicitInterface()) {
              std::string whyNot;
              if (!chars->IsCompatibleWith(*globalChars,
                      /*ignoreImplicitVsExplicit=*/false, &whyNot)) {
                msg = Warn(common::UsageWarning::ExternalInterfaceMismatch,
                    "The global subprogram '%s' is not compatible with its local procedure declaration (%s)"_warn_en_US,
                    global->name(), whyNot);
              }
            } else if (!globalChars->CanBeCalledViaImplicitInterface()) {
              // TODO: This should be a hard error if the procedure has
              // actually been called (as opposed to just being used as a
              // procedure pointer target or passed as an actual argument).
````
- **L1777 EN**: Closes the current lexical scope or compound statement.
  **L1777 CN**: 结束当前词法作用域或复合语句块。
- **L1778 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1778 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1779 EN**: Executes a standalone statement or declaration: `parser::Message *msg{nullptr};`.
  **L1779 CN**: 执行一条独立语句或声明：`parser::Message *msg{nullptr};`。
- **L1780 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1780 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1781 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1781 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1782 EN**: Starts a function, method, lambda, or structured scope: `symbol.flags().test(Symbol::Flag::Subroutine))) {`.
  **L1782 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.flags().test(Symbol::Flag::Subroutine))) {`。
- **L1783 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg = Warn(common::UsageWarning::ExternalNameConflict,`.
  **L1783 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg = Warn(common::UsageWarning::ExternalNameConflict,`。
- **L1784 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The global entity '%s' corresponding to the local procedure '%s' is not a callable subprogram"_warn_en_US,`.
  **L1784 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The global entity '%s' corresponding to the local procedure '%s' is not a callable subprogram"_warn_en_US,`。
- **L1785 EN**: Executes a call or declaration centered on `global->name`.
  **L1785 CN**: 执行以 `global->name` 为核心的调用或声明。
- **L1786 EN**: Closes the current lexical scope or compound statement.
  **L1786 CN**: 结束当前词法作用域或复合语句块。
- **L1787 EN**: Transitions from the previous branch into an `else if` condition.
  **L1787 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1788 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1788 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1789 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1789 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1790 EN**: Executes a standalone statement or declaration: `std::string whyNot;`.
  **L1790 CN**: 执行一条独立语句或声明：`std::string whyNot;`。
- **L1791 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1791 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1792 EN**: Comment explains nearby logic, intent, or metadata: `ignoreImplicitVsExplicit=*/false, &whyNot)) {`.
  **L1792 CN**: 注释说明附近代码的逻辑、意图或元数据：`ignoreImplicitVsExplicit=*/false, &whyNot)) {`。
- **L1793 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg = Warn(common::UsageWarning::ExternalInterfaceMismatch,`.
  **L1793 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg = Warn(common::UsageWarning::ExternalInterfaceMismatch,`。
- **L1794 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The global subprogram '%s' is not compatible with its local procedure declaration (%s)"_warn_en_US,`.
  **L1794 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The global subprogram '%s' is not compatible with its local procedure declaration (%s)"_warn_en_US,`。
- **L1795 EN**: Executes a call or declaration centered on `global->name`.
  **L1795 CN**: 执行以 `global->name` 为核心的调用或声明。
- **L1796 EN**: Closes the current lexical scope or compound statement.
  **L1796 CN**: 结束当前词法作用域或复合语句块。
- **L1797 EN**: Transitions from the previous branch into an `else if` condition.
  **L1797 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1798 EN**: Comment records a pending task or caution: `TODO: This should be a hard error if the procedure has`.
  **L1798 CN**: 注释记录待办事项或注意点：`TODO: This should be a hard error if the procedure has`。
- **L1799 EN**: Comment explains nearby logic, intent, or metadata: `actually been called (as opposed to just being used as a`.
  **L1799 CN**: 注释说明附近代码的逻辑、意图或元数据：`actually been called (as opposed to just being used as a`。
- **L1800 EN**: Comment explains nearby logic, intent, or metadata: `procedure pointer target or passed as an actual argument).`.
  **L1800 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedure pointer target or passed as an actual argument).`。

### Lines 1801-1824

````cpp
              msg = Warn(common::UsageWarning::ExternalInterfaceMismatch,
                  "The global subprogram '%s' should not be referenced via the implicit interface '%s'"_warn_en_US,
                  global->name(), symbol.name());
            }
          }
        }
        if (msg) {
          if (msg->IsFatal()) {
            context_.SetError(symbol);
          }
          evaluate::AttachDeclaration(msg, *global);
          evaluate::AttachDeclaration(msg, symbol);
        }
      }
    } else if (auto iter{externalNames_.find(interfaceName)};
               iter != externalNames_.end()) {
      const Symbol &previous{*iter->second};
      if (auto chars{Characterize(symbol)}) {
        if (auto previousChars{Characterize(previous)}) {
          std::string whyNot;
          if (!chars->IsCompatibleWith(*previousChars,
                  /*ignoreImplicitVsExplicit=*/true, &whyNot)) {
            if (auto *msg{Warn(common::UsageWarning::ExternalInterfaceMismatch,
                    "The external interface '%s' is not compatible with an earlier definition (%s)"_warn_en_US,
````
- **L1801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg = Warn(common::UsageWarning::ExternalInterfaceMismatch,`.
  **L1801 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg = Warn(common::UsageWarning::ExternalInterfaceMismatch,`。
- **L1802 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The global subprogram '%s' should not be referenced via the implicit interface '%s'"_warn_en_US,`.
  **L1802 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The global subprogram '%s' should not be referenced via the implicit interface '%s'"_warn_en_US,`。
- **L1803 EN**: Executes a call or declaration centered on `global->name`.
  **L1803 CN**: 执行以 `global->name` 为核心的调用或声明。
- **L1804 EN**: Closes the current lexical scope or compound statement.
  **L1804 CN**: 结束当前词法作用域或复合语句块。
- **L1805 EN**: Closes the current lexical scope or compound statement.
  **L1805 CN**: 结束当前词法作用域或复合语句块。
- **L1806 EN**: Closes the current lexical scope or compound statement.
  **L1806 CN**: 结束当前词法作用域或复合语句块。
- **L1807 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1807 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1808 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1808 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1809 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L1809 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L1810 EN**: Closes the current lexical scope or compound statement.
  **L1810 CN**: 结束当前词法作用域或复合语句块。
- **L1811 EN**: Executes a call or declaration centered on `evaluate::AttachDeclaration`.
  **L1811 CN**: 执行以 `evaluate::AttachDeclaration` 为核心的调用或声明。
- **L1812 EN**: Executes a call or declaration centered on `evaluate::AttachDeclaration`.
  **L1812 CN**: 执行以 `evaluate::AttachDeclaration` 为核心的调用或声明。
- **L1813 EN**: Closes the current lexical scope or compound statement.
  **L1813 CN**: 结束当前词法作用域或复合语句块。
- **L1814 EN**: Closes the current lexical scope or compound statement.
  **L1814 CN**: 结束当前词法作用域或复合语句块。
- **L1815 EN**: Transitions from the previous branch into an `else if` condition.
  **L1815 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1816 EN**: Starts a function, method, lambda, or structured scope: `iter != externalNames_.end()) {`.
  **L1816 CN**: 开始一个函数、方法、lambda 或结构化作用域：`iter != externalNames_.end()) {`。
- **L1817 EN**: Executes a standalone statement or declaration: `const Symbol &previous{*iter->second};`.
  **L1817 CN**: 执行一条独立语句或声明：`const Symbol &previous{*iter->second};`。
- **L1818 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1818 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1819 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1819 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1820 EN**: Executes a standalone statement or declaration: `std::string whyNot;`.
  **L1820 CN**: 执行一条独立语句或声明：`std::string whyNot;`。
- **L1821 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1821 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1822 EN**: Comment explains nearby logic, intent, or metadata: `ignoreImplicitVsExplicit=*/true, &whyNot)) {`.
  **L1822 CN**: 注释说明附近代码的逻辑、意图或元数据：`ignoreImplicitVsExplicit=*/true, &whyNot)) {`。
- **L1823 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1823 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The external interface '%s' is not compatible with an earlier definition (%s)"_warn_en_US,`.
  **L1824 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The external interface '%s' is not compatible with an earlier definition (%s)"_warn_en_US,`。

### Lines 1825-1848

````cpp
                    symbol.name(), whyNot)}) {
              evaluate::AttachDeclaration(msg, previous);
              evaluate::AttachDeclaration(msg, symbol);
            }
          }
        }
      }
    } else {
      externalNames_.emplace(interfaceName, symbol);
    }
  }
}

void CheckHelper::CheckDerivedType(
    const Symbol &derivedType, const DerivedTypeDetails &details) {
  if (details.isForwardReferenced() && !context_.HasError(derivedType)) {
    messages_.Say("The derived type '%s' has not been defined"_err_en_US,
        derivedType.name());
  }
  const Scope *scope{derivedType.scope()};
  if (!scope) {
    CHECK(details.isForwardReferenced());
    return;
  }
````
- **L1825 EN**: Starts a function, method, lambda, or structured scope: `symbol.name(), whyNot)}) {`.
  **L1825 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.name(), whyNot)}) {`。
- **L1826 EN**: Executes a call or declaration centered on `evaluate::AttachDeclaration`.
  **L1826 CN**: 执行以 `evaluate::AttachDeclaration` 为核心的调用或声明。
- **L1827 EN**: Executes a call or declaration centered on `evaluate::AttachDeclaration`.
  **L1827 CN**: 执行以 `evaluate::AttachDeclaration` 为核心的调用或声明。
- **L1828 EN**: Closes the current lexical scope or compound statement.
  **L1828 CN**: 结束当前词法作用域或复合语句块。
- **L1829 EN**: Closes the current lexical scope or compound statement.
  **L1829 CN**: 结束当前词法作用域或复合语句块。
- **L1830 EN**: Closes the current lexical scope or compound statement.
  **L1830 CN**: 结束当前词法作用域或复合语句块。
- **L1831 EN**: Closes the current lexical scope or compound statement.
  **L1831 CN**: 结束当前词法作用域或复合语句块。
- **L1832 EN**: Transitions from the previous branch into the alternative path.
  **L1832 CN**: 从前一个分支过渡到备选路径。
- **L1833 EN**: Executes a call or declaration centered on `externalNames_.emplace`.
  **L1833 CN**: 执行以 `externalNames_.emplace` 为核心的调用或声明。
- **L1834 EN**: Closes the current lexical scope or compound statement.
  **L1834 CN**: 结束当前词法作用域或复合语句块。
- **L1835 EN**: Closes the current lexical scope or compound statement.
  **L1835 CN**: 结束当前词法作用域或复合语句块。
- **L1836 EN**: Closes the current lexical scope or compound statement.
  **L1836 CN**: 结束当前词法作用域或复合语句块。
- **L1837 EN**: Blank line separating nearby declarations or logic blocks.
  **L1837 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1838 EN**: Continues logic associated with callable symbol `CheckDerivedType`.
  **L1838 CN**: 继续与可调用符号 `CheckDerivedType` 相关的逻辑。
- **L1839 EN**: Continues the surrounding expression or declaration: `const Symbol &derivedType, const DerivedTypeDetails &details) {`.
  **L1839 CN**: 继续构造周围的表达式或声明：`const Symbol &derivedType, const DerivedTypeDetails &details) {`。
- **L1840 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1840 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1841 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say("The derived type '%s' has not been defined"_err_en_US,`.
  **L1841 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say("The derived type '%s' has not been defined"_err_en_US,`。
- **L1842 EN**: Executes a call or declaration centered on `derivedType.name`.
  **L1842 CN**: 执行以 `derivedType.name` 为核心的调用或声明。
- **L1843 EN**: Closes the current lexical scope or compound statement.
  **L1843 CN**: 结束当前词法作用域或复合语句块。
- **L1844 EN**: Executes a call or declaration centered on `*scope{derivedType.scope`.
  **L1844 CN**: 执行以 `*scope{derivedType.scope` 为核心的调用或声明。
- **L1845 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1845 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1846 EN**: Executes a call or declaration centered on `CHECK`.
  **L1846 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L1847 EN**: Returns from the current function with `void`.
  **L1847 CN**: 以 `void` 从当前函数返回。
- **L1848 EN**: Closes the current lexical scope or compound statement.
  **L1848 CN**: 结束当前词法作用域或复合语句块。

### Lines 1849-1872

````cpp
  CHECK(scope->symbol() == &derivedType);
  CHECK(scope->IsDerivedType());
  if (derivedType.attrs().test(Attr::ABSTRACT) && // C734
      (derivedType.attrs().test(Attr::BIND_C) || details.sequence())) {
    messages_.Say("An ABSTRACT derived type must be extensible"_err_en_US);
  }
  if (const DeclTypeSpec *parent{FindParentTypeSpec(derivedType)}) {
    const DerivedTypeSpec *parentDerived{parent->AsDerived()};
    if (!IsExtensibleType(parentDerived)) { // C705
      messages_.Say("The parent type is not extensible"_err_en_US);
    }
    if (!derivedType.attrs().test(Attr::ABSTRACT) && parentDerived &&
        parentDerived->typeSymbol().attrs().test(Attr::ABSTRACT)) {
      ScopeComponentIterator components{*parentDerived};
      for (const Symbol &component : components) {
        if (component.attrs().test(Attr::DEFERRED)) {
          if (scope->FindComponent(component.name()) == &component) {
            SayWithDeclaration(component,
                "Non-ABSTRACT extension of ABSTRACT derived type '%s' lacks a binding for DEFERRED procedure '%s'"_err_en_US,
                parentDerived->typeSymbol().name(), component.name());
          }
        }
      }
    }
````
- **L1849 EN**: Executes a call or declaration centered on `CHECK`.
  **L1849 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L1850 EN**: Executes a call or declaration centered on `CHECK`.
  **L1850 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L1851 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1851 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1852 EN**: Starts a function, method, lambda, or structured scope: `(derivedType.attrs().test(Attr::BIND_C) || details.sequence())) {`.
  **L1852 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(derivedType.attrs().test(Attr::BIND_C) || details.sequence())) {`。
- **L1853 EN**: Executes a call or declaration centered on `messages_.Say`.
  **L1853 CN**: 执行以 `messages_.Say` 为核心的调用或声明。
- **L1854 EN**: Closes the current lexical scope or compound statement.
  **L1854 CN**: 结束当前词法作用域或复合语句块。
- **L1855 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1855 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1856 EN**: Executes a call or declaration centered on `*parentDerived{parent->AsDerived`.
  **L1856 CN**: 执行以 `*parentDerived{parent->AsDerived` 为核心的调用或声明。
- **L1857 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1857 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1858 EN**: Executes a call or declaration centered on `messages_.Say`.
  **L1858 CN**: 执行以 `messages_.Say` 为核心的调用或声明。
- **L1859 EN**: Closes the current lexical scope or compound statement.
  **L1859 CN**: 结束当前词法作用域或复合语句块。
- **L1860 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1860 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1861 EN**: Starts a function, method, lambda, or structured scope: `parentDerived->typeSymbol().attrs().test(Attr::ABSTRACT)) {`.
  **L1861 CN**: 开始一个函数、方法、lambda 或结构化作用域：`parentDerived->typeSymbol().attrs().test(Attr::ABSTRACT)) {`。
- **L1862 EN**: Executes a standalone statement or declaration: `ScopeComponentIterator components{*parentDerived};`.
  **L1862 CN**: 执行一条独立语句或声明：`ScopeComponentIterator components{*parentDerived};`。
- **L1863 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1863 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1864 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1864 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1865 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1865 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(component,`.
  **L1866 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(component,`。
- **L1867 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Non-ABSTRACT extension of ABSTRACT derived type '%s' lacks a binding for DEFERRED procedure '%s'"_err_en_US,`.
  **L1867 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Non-ABSTRACT extension of ABSTRACT derived type '%s' lacks a binding for DEFERRED procedure '%s'"_err_en_US,`。
- **L1868 EN**: Executes a call or declaration centered on `parentDerived->typeSymbol`.
  **L1868 CN**: 执行以 `parentDerived->typeSymbol` 为核心的调用或声明。
- **L1869 EN**: Closes the current lexical scope or compound statement.
  **L1869 CN**: 结束当前词法作用域或复合语句块。
- **L1870 EN**: Closes the current lexical scope or compound statement.
  **L1870 CN**: 结束当前词法作用域或复合语句块。
- **L1871 EN**: Closes the current lexical scope or compound statement.
  **L1871 CN**: 结束当前词法作用域或复合语句块。
- **L1872 EN**: Closes the current lexical scope or compound statement.
  **L1872 CN**: 结束当前词法作用域或复合语句块。

### Lines 1873-1896

````cpp
    DerivedTypeSpec derived{derivedType.name(), derivedType};
    derived.set_scope(*scope);
    if (FindCoarrayUltimateComponent(derived) && // C736
        !(parentDerived && FindCoarrayUltimateComponent(*parentDerived))) {
      messages_.Say(
          "Type '%s' has a coarray ultimate component so the type at the base "
          "of its type extension chain ('%s') must be a type that has a "
          "coarray ultimate component"_err_en_US,
          derivedType.name(), scope->GetDerivedTypeBase().GetSymbol()->name());
    }
    if (FindEventOrLockPotentialComponent(derived) && // C737
        !(FindEventOrLockPotentialComponent(*parentDerived) ||
            IsEventTypeOrLockType(parentDerived))) {
      messages_.Say(
          "Type '%s' has an EVENT_TYPE or LOCK_TYPE component, so the type "
          "at the base of its type extension chain ('%s') must either have an "
          "EVENT_TYPE or LOCK_TYPE component, or be EVENT_TYPE or "
          "LOCK_TYPE"_err_en_US,
          derivedType.name(), scope->GetDerivedTypeBase().GetSymbol()->name());
    }
  }
  if (HasIntrinsicTypeName(derivedType)) { // C729
    messages_.Say("A derived type name cannot be the name of an intrinsic"
                  " type"_err_en_US);
````
- **L1873 EN**: Executes a call or declaration centered on `derived{derivedType.name`.
  **L1873 CN**: 执行以 `derived{derivedType.name` 为核心的调用或声明。
- **L1874 EN**: Executes a call or declaration centered on `derived.set_scope`.
  **L1874 CN**: 执行以 `derived.set_scope` 为核心的调用或声明。
- **L1875 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1875 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1876 EN**: Starts a function, method, lambda, or structured scope: `!(parentDerived && FindCoarrayUltimateComponent(*parentDerived))) {`.
  **L1876 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!(parentDerived && FindCoarrayUltimateComponent(*parentDerived))) {`。
- **L1877 EN**: Continues logic associated with callable symbol `Say`.
  **L1877 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1878 EN**: Continues the surrounding expression or declaration: `"Type '%s' has a coarray ultimate component so the type at the base "`.
  **L1878 CN**: 继续构造周围的表达式或声明：`"Type '%s' has a coarray ultimate component so the type at the base "`。
- **L1879 EN**: Continues logic associated with callable symbol `chain`.
  **L1879 CN**: 继续与可调用符号 `chain` 相关的逻辑。
- **L1880 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"coarray ultimate component"_err_en_US,`.
  **L1880 CN**: 继续一个多行参数列表、初始化器或聚合项：`"coarray ultimate component"_err_en_US,`。
- **L1881 EN**: Executes a call or declaration centered on `derivedType.name`.
  **L1881 CN**: 执行以 `derivedType.name` 为核心的调用或声明。
- **L1882 EN**: Closes the current lexical scope or compound statement.
  **L1882 CN**: 结束当前词法作用域或复合语句块。
- **L1883 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1883 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1884 EN**: Continues logic associated with callable symbol `FindEventOrLockPotentialComponent`.
  **L1884 CN**: 继续与可调用符号 `FindEventOrLockPotentialComponent` 相关的逻辑。
- **L1885 EN**: Starts a function, method, lambda, or structured scope: `IsEventTypeOrLockType(parentDerived))) {`.
  **L1885 CN**: 开始一个函数、方法、lambda 或结构化作用域：`IsEventTypeOrLockType(parentDerived))) {`。
- **L1886 EN**: Continues logic associated with callable symbol `Say`.
  **L1886 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1887 EN**: Continues the surrounding expression or declaration: `"Type '%s' has an EVENT_TYPE or LOCK_TYPE component, so the type "`.
  **L1887 CN**: 继续构造周围的表达式或声明：`"Type '%s' has an EVENT_TYPE or LOCK_TYPE component, so the type "`。
- **L1888 EN**: Continues logic associated with callable symbol `chain`.
  **L1888 CN**: 继续与可调用符号 `chain` 相关的逻辑。
- **L1889 EN**: Continues the surrounding expression or declaration: `"EVENT_TYPE or LOCK_TYPE component, or be EVENT_TYPE or "`.
  **L1889 CN**: 继续构造周围的表达式或声明：`"EVENT_TYPE or LOCK_TYPE component, or be EVENT_TYPE or "`。
- **L1890 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"LOCK_TYPE"_err_en_US,`.
  **L1890 CN**: 继续一个多行参数列表、初始化器或聚合项：`"LOCK_TYPE"_err_en_US,`。
- **L1891 EN**: Executes a call or declaration centered on `derivedType.name`.
  **L1891 CN**: 执行以 `derivedType.name` 为核心的调用或声明。
- **L1892 EN**: Closes the current lexical scope or compound statement.
  **L1892 CN**: 结束当前词法作用域或复合语句块。
- **L1893 EN**: Closes the current lexical scope or compound statement.
  **L1893 CN**: 结束当前词法作用域或复合语句块。
- **L1894 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1894 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1895 EN**: Continues logic associated with callable symbol `Say`.
  **L1895 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1896 EN**: Executes a standalone statement or declaration: `" type"_err_en_US);`.
  **L1896 CN**: 执行一条独立语句或声明：`" type"_err_en_US);`。

### Lines 1897-1920

````cpp
  }
  std::map<SourceName, SymbolRef> previous;
  for (const auto &pair : details.finals()) {
    SourceName source{pair.first};
    const Symbol &ref{*pair.second};
    if (CheckFinal(ref, source, derivedType) &&
        std::all_of(previous.begin(), previous.end(),
            [&](std::pair<SourceName, SymbolRef> prev) {
              return CheckDistinguishableFinals(
                  ref, source, *prev.second, prev.first, derivedType);
            })) {
      previous.emplace(source, ref);
    }
  }
}

// C786
bool CheckHelper::CheckFinal(
    const Symbol &subroutine, SourceName finalName, const Symbol &derivedType) {
  if (!IsModuleProcedure(subroutine)) {
    SayWithDeclaration(subroutine, finalName,
        "FINAL subroutine '%s' of derived type '%s' must be a module procedure"_err_en_US,
        subroutine.name(), derivedType.name());
    return false;
````
- **L1897 EN**: Closes the current lexical scope or compound statement.
  **L1897 CN**: 结束当前词法作用域或复合语句块。
- **L1898 EN**: Executes a standalone statement or declaration: `std::map<SourceName, SymbolRef> previous;`.
  **L1898 CN**: 执行一条独立语句或声明：`std::map<SourceName, SymbolRef> previous;`。
- **L1899 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1899 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1900 EN**: Executes a standalone statement or declaration: `SourceName source{pair.first};`.
  **L1900 CN**: 执行一条独立语句或声明：`SourceName source{pair.first};`。
- **L1901 EN**: Executes a standalone statement or declaration: `const Symbol &ref{*pair.second};`.
  **L1901 CN**: 执行一条独立语句或声明：`const Symbol &ref{*pair.second};`。
- **L1902 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1902 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1903 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::all_of(previous.begin(), previous.end(),`.
  **L1903 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::all_of(previous.begin(), previous.end(),`。
- **L1904 EN**: Starts a function, method, lambda, or structured scope: `[&](std::pair<SourceName, SymbolRef> prev) {`.
  **L1904 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](std::pair<SourceName, SymbolRef> prev) {`。
- **L1905 EN**: Returns from the current function with `CheckDistinguishableFinals(`.
  **L1905 CN**: 以 `CheckDistinguishableFinals(` 从当前函数返回。
- **L1906 EN**: Executes a standalone statement or declaration: `ref, source, *prev.second, prev.first, derivedType);`.
  **L1906 CN**: 执行一条独立语句或声明：`ref, source, *prev.second, prev.first, derivedType);`。
- **L1907 EN**: Continues the surrounding expression or declaration: `})) {`.
  **L1907 CN**: 继续构造周围的表达式或声明：`})) {`。
- **L1908 EN**: Executes a call or declaration centered on `previous.emplace`.
  **L1908 CN**: 执行以 `previous.emplace` 为核心的调用或声明。
- **L1909 EN**: Closes the current lexical scope or compound statement.
  **L1909 CN**: 结束当前词法作用域或复合语句块。
- **L1910 EN**: Closes the current lexical scope or compound statement.
  **L1910 CN**: 结束当前词法作用域或复合语句块。
- **L1911 EN**: Closes the current lexical scope or compound statement.
  **L1911 CN**: 结束当前词法作用域或复合语句块。
- **L1912 EN**: Blank line separating nearby declarations or logic blocks.
  **L1912 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1913 EN**: Comment explains nearby logic, intent, or metadata: `C786`.
  **L1913 CN**: 注释说明附近代码的逻辑、意图或元数据：`C786`。
- **L1914 EN**: Continues logic associated with callable symbol `CheckFinal`.
  **L1914 CN**: 继续与可调用符号 `CheckFinal` 相关的逻辑。
- **L1915 EN**: Continues the surrounding expression or declaration: `const Symbol &subroutine, SourceName finalName, const Symbol &derivedType) {`.
  **L1915 CN**: 继续构造周围的表达式或声明：`const Symbol &subroutine, SourceName finalName, const Symbol &derivedType) {`。
- **L1916 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1916 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1917 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(subroutine, finalName,`.
  **L1917 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(subroutine, finalName,`。
- **L1918 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FINAL subroutine '%s' of derived type '%s' must be a module procedure"_err_en_US,`.
  **L1918 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FINAL subroutine '%s' of derived type '%s' must be a module procedure"_err_en_US,`。
- **L1919 EN**: Executes a call or declaration centered on `subroutine.name`.
  **L1919 CN**: 执行以 `subroutine.name` 为核心的调用或声明。
- **L1920 EN**: Returns from the current function with `false`.
  **L1920 CN**: 以 `false` 从当前函数返回。

### Lines 1921-1944

````cpp
  }
  const Procedure *proc{Characterize(subroutine)};
  if (!proc) {
    return false; // error recovery
  }
  if (!proc->IsSubroutine()) {
    SayWithDeclaration(subroutine, finalName,
        "FINAL subroutine '%s' of derived type '%s' must be a subroutine"_err_en_US,
        subroutine.name(), derivedType.name());
    return false;
  }
  if (proc->dummyArguments.size() != 1) {
    SayWithDeclaration(subroutine, finalName,
        "FINAL subroutine '%s' of derived type '%s' must have a single dummy argument"_err_en_US,
        subroutine.name(), derivedType.name());
    return false;
  }
  const auto &arg{proc->dummyArguments[0]};
  const Symbol *errSym{&subroutine};
  if (const auto *details{subroutine.detailsIf<SubprogramDetails>()}) {
    if (!details->dummyArgs().empty()) {
      if (const Symbol *argSym{details->dummyArgs()[0]}) {
        errSym = argSym;
      }
````
- **L1921 EN**: Closes the current lexical scope or compound statement.
  **L1921 CN**: 结束当前词法作用域或复合语句块。
- **L1922 EN**: Executes a call or declaration centered on `*proc{Characterize`.
  **L1922 CN**: 执行以 `*proc{Characterize` 为核心的调用或声明。
- **L1923 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1923 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1924 EN**: Returns from the current function with `false; // error recovery`.
  **L1924 CN**: 以 `false; // error recovery` 从当前函数返回。
- **L1925 EN**: Closes the current lexical scope or compound statement.
  **L1925 CN**: 结束当前词法作用域或复合语句块。
- **L1926 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1926 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(subroutine, finalName,`.
  **L1927 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(subroutine, finalName,`。
- **L1928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FINAL subroutine '%s' of derived type '%s' must be a subroutine"_err_en_US,`.
  **L1928 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FINAL subroutine '%s' of derived type '%s' must be a subroutine"_err_en_US,`。
- **L1929 EN**: Executes a call or declaration centered on `subroutine.name`.
  **L1929 CN**: 执行以 `subroutine.name` 为核心的调用或声明。
- **L1930 EN**: Returns from the current function with `false`.
  **L1930 CN**: 以 `false` 从当前函数返回。
- **L1931 EN**: Closes the current lexical scope or compound statement.
  **L1931 CN**: 结束当前词法作用域或复合语句块。
- **L1932 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1932 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1933 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(subroutine, finalName,`.
  **L1933 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(subroutine, finalName,`。
- **L1934 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FINAL subroutine '%s' of derived type '%s' must have a single dummy argument"_err_en_US,`.
  **L1934 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FINAL subroutine '%s' of derived type '%s' must have a single dummy argument"_err_en_US,`。
- **L1935 EN**: Executes a call or declaration centered on `subroutine.name`.
  **L1935 CN**: 执行以 `subroutine.name` 为核心的调用或声明。
- **L1936 EN**: Returns from the current function with `false`.
  **L1936 CN**: 以 `false` 从当前函数返回。
- **L1937 EN**: Closes the current lexical scope or compound statement.
  **L1937 CN**: 结束当前词法作用域或复合语句块。
- **L1938 EN**: Executes a standalone statement or declaration: `const auto &arg{proc->dummyArguments[0]};`.
  **L1938 CN**: 执行一条独立语句或声明：`const auto &arg{proc->dummyArguments[0]};`。
- **L1939 EN**: Executes a standalone statement or declaration: `const Symbol *errSym{&subroutine};`.
  **L1939 CN**: 执行一条独立语句或声明：`const Symbol *errSym{&subroutine};`。
- **L1940 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1940 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1941 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1941 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1942 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1942 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1943 EN**: Executes a standalone statement or declaration: `errSym = argSym;`.
  **L1943 CN**: 执行一条独立语句或声明：`errSym = argSym;`。
- **L1944 EN**: Closes the current lexical scope or compound statement.
  **L1944 CN**: 结束当前词法作用域或复合语句块。

### Lines 1945-1968

````cpp
    }
  }
  const auto *ddo{std::get_if<DummyDataObject>(&arg.u)};
  if (!ddo) {
    SayWithDeclaration(subroutine, finalName,
        "FINAL subroutine '%s' of derived type '%s' must have a single dummy argument that is a data object"_err_en_US,
        subroutine.name(), derivedType.name());
    return false;
  }
  bool ok{true};
  if (arg.IsOptional()) {
    SayWithDeclaration(*errSym, finalName,
        "FINAL subroutine '%s' of derived type '%s' must not have an OPTIONAL dummy argument"_err_en_US,
        subroutine.name(), derivedType.name());
    ok = false;
  }
  if (ddo->attrs.test(DummyDataObject::Attr::Allocatable)) {
    SayWithDeclaration(*errSym, finalName,
        "FINAL subroutine '%s' of derived type '%s' must not have an ALLOCATABLE dummy argument"_err_en_US,
        subroutine.name(), derivedType.name());
    ok = false;
  }
  if (ddo->attrs.test(DummyDataObject::Attr::Pointer)) {
    SayWithDeclaration(*errSym, finalName,
````
- **L1945 EN**: Closes the current lexical scope or compound statement.
  **L1945 CN**: 结束当前词法作用域或复合语句块。
- **L1946 EN**: Closes the current lexical scope or compound statement.
  **L1946 CN**: 结束当前词法作用域或复合语句块。
- **L1947 EN**: Executes a call or declaration centered on `*ddo{std::get_if<DummyDataObject>`.
  **L1947 CN**: 执行以 `*ddo{std::get_if<DummyDataObject>` 为核心的调用或声明。
- **L1948 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1948 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1949 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(subroutine, finalName,`.
  **L1949 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(subroutine, finalName,`。
- **L1950 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FINAL subroutine '%s' of derived type '%s' must have a single dummy argument that is a data object"_err_en_US,`.
  **L1950 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FINAL subroutine '%s' of derived type '%s' must have a single dummy argument that is a data object"_err_en_US,`。
- **L1951 EN**: Executes a call or declaration centered on `subroutine.name`.
  **L1951 CN**: 执行以 `subroutine.name` 为核心的调用或声明。
- **L1952 EN**: Returns from the current function with `false`.
  **L1952 CN**: 以 `false` 从当前函数返回。
- **L1953 EN**: Closes the current lexical scope or compound statement.
  **L1953 CN**: 结束当前词法作用域或复合语句块。
- **L1954 EN**: Executes a standalone statement or declaration: `bool ok{true};`.
  **L1954 CN**: 执行一条独立语句或声明：`bool ok{true};`。
- **L1955 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1955 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1956 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*errSym, finalName,`.
  **L1956 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*errSym, finalName,`。
- **L1957 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FINAL subroutine '%s' of derived type '%s' must not have an OPTIONAL dummy argument"_err_en_US,`.
  **L1957 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FINAL subroutine '%s' of derived type '%s' must not have an OPTIONAL dummy argument"_err_en_US,`。
- **L1958 EN**: Executes a call or declaration centered on `subroutine.name`.
  **L1958 CN**: 执行以 `subroutine.name` 为核心的调用或声明。
- **L1959 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L1959 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L1960 EN**: Closes the current lexical scope or compound statement.
  **L1960 CN**: 结束当前词法作用域或复合语句块。
- **L1961 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1961 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1962 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*errSym, finalName,`.
  **L1962 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*errSym, finalName,`。
- **L1963 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FINAL subroutine '%s' of derived type '%s' must not have an ALLOCATABLE dummy argument"_err_en_US,`.
  **L1963 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FINAL subroutine '%s' of derived type '%s' must not have an ALLOCATABLE dummy argument"_err_en_US,`。
- **L1964 EN**: Executes a call or declaration centered on `subroutine.name`.
  **L1964 CN**: 执行以 `subroutine.name` 为核心的调用或声明。
- **L1965 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L1965 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L1966 EN**: Closes the current lexical scope or compound statement.
  **L1966 CN**: 结束当前词法作用域或复合语句块。
- **L1967 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1967 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1968 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*errSym, finalName,`.
  **L1968 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*errSym, finalName,`。

### Lines 1969-1992

````cpp
        "FINAL subroutine '%s' of derived type '%s' must not have a POINTER dummy argument"_err_en_US,
        subroutine.name(), derivedType.name());
    ok = false;
  }
  if (ddo->intent == common::Intent::Out) {
    SayWithDeclaration(*errSym, finalName,
        "FINAL subroutine '%s' of derived type '%s' must not have a dummy argument with INTENT(OUT)"_err_en_US,
        subroutine.name(), derivedType.name());
    ok = false;
  }
  if (ddo->attrs.test(DummyDataObject::Attr::Value)) {
    SayWithDeclaration(*errSym, finalName,
        "FINAL subroutine '%s' of derived type '%s' must not have a dummy argument with the VALUE attribute"_err_en_US,
        subroutine.name(), derivedType.name());
    ok = false;
  }
  if (ddo->type.corank() > 0) {
    SayWithDeclaration(*errSym, finalName,
        "FINAL subroutine '%s' of derived type '%s' must not have a coarray dummy argument"_err_en_US,
        subroutine.name(), derivedType.name());
    ok = false;
  }
  if (ddo->type.type().IsPolymorphic()) {
    SayWithDeclaration(*errSym, finalName,
````
- **L1969 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FINAL subroutine '%s' of derived type '%s' must not have a POINTER dummy argument"_err_en_US,`.
  **L1969 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FINAL subroutine '%s' of derived type '%s' must not have a POINTER dummy argument"_err_en_US,`。
- **L1970 EN**: Executes a call or declaration centered on `subroutine.name`.
  **L1970 CN**: 执行以 `subroutine.name` 为核心的调用或声明。
- **L1971 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L1971 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L1972 EN**: Closes the current lexical scope or compound statement.
  **L1972 CN**: 结束当前词法作用域或复合语句块。
- **L1973 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1973 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1974 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*errSym, finalName,`.
  **L1974 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*errSym, finalName,`。
- **L1975 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FINAL subroutine '%s' of derived type '%s' must not have a dummy argument with INTENT(OUT)"_err_en_US,`.
  **L1975 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FINAL subroutine '%s' of derived type '%s' must not have a dummy argument with INTENT(OUT)"_err_en_US,`。
- **L1976 EN**: Executes a call or declaration centered on `subroutine.name`.
  **L1976 CN**: 执行以 `subroutine.name` 为核心的调用或声明。
- **L1977 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L1977 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L1978 EN**: Closes the current lexical scope or compound statement.
  **L1978 CN**: 结束当前词法作用域或复合语句块。
- **L1979 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1979 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1980 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*errSym, finalName,`.
  **L1980 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*errSym, finalName,`。
- **L1981 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FINAL subroutine '%s' of derived type '%s' must not have a dummy argument with the VALUE attribute"_err_en_US,`.
  **L1981 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FINAL subroutine '%s' of derived type '%s' must not have a dummy argument with the VALUE attribute"_err_en_US,`。
- **L1982 EN**: Executes a call or declaration centered on `subroutine.name`.
  **L1982 CN**: 执行以 `subroutine.name` 为核心的调用或声明。
- **L1983 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L1983 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L1984 EN**: Closes the current lexical scope or compound statement.
  **L1984 CN**: 结束当前词法作用域或复合语句块。
- **L1985 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1985 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1986 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*errSym, finalName,`.
  **L1986 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*errSym, finalName,`。
- **L1987 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FINAL subroutine '%s' of derived type '%s' must not have a coarray dummy argument"_err_en_US,`.
  **L1987 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FINAL subroutine '%s' of derived type '%s' must not have a coarray dummy argument"_err_en_US,`。
- **L1988 EN**: Executes a call or declaration centered on `subroutine.name`.
  **L1988 CN**: 执行以 `subroutine.name` 为核心的调用或声明。
- **L1989 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L1989 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L1990 EN**: Closes the current lexical scope or compound statement.
  **L1990 CN**: 结束当前词法作用域或复合语句块。
- **L1991 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1991 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1992 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*errSym, finalName,`.
  **L1992 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*errSym, finalName,`。

### Lines 1993-2016

````cpp
        "FINAL subroutine '%s' of derived type '%s' must not have a polymorphic dummy argument"_err_en_US,
        subroutine.name(), derivedType.name());
    ok = false;
  } else if (ddo->type.type().category() != TypeCategory::Derived ||
      &ddo->type.type().GetDerivedTypeSpec().typeSymbol() != &derivedType) {
    SayWithDeclaration(*errSym, finalName,
        "FINAL subroutine '%s' of derived type '%s' must have a TYPE(%s) dummy argument"_err_en_US,
        subroutine.name(), derivedType.name(), derivedType.name());
    ok = false;
  } else { // check that all LEN type parameters are assumed
    for (auto ref : OrderParameterDeclarations(derivedType)) {
      if (IsLenTypeParameter(*ref)) {
        const auto *value{
            ddo->type.type().GetDerivedTypeSpec().FindParameter(ref->name())};
        if (!value || !value->isAssumed()) {
          SayWithDeclaration(*errSym, finalName,
              "FINAL subroutine '%s' of derived type '%s' must have a dummy argument with an assumed LEN type parameter '%s=*'"_err_en_US,
              subroutine.name(), derivedType.name(), ref->name());
          ok = false;
        }
      }
    }
  }
  return ok;
````
- **L1993 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FINAL subroutine '%s' of derived type '%s' must not have a polymorphic dummy argument"_err_en_US,`.
  **L1993 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FINAL subroutine '%s' of derived type '%s' must not have a polymorphic dummy argument"_err_en_US,`。
- **L1994 EN**: Executes a call or declaration centered on `subroutine.name`.
  **L1994 CN**: 执行以 `subroutine.name` 为核心的调用或声明。
- **L1995 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L1995 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L1996 EN**: Transitions from the previous branch into an `else if` condition.
  **L1996 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1997 EN**: Starts a function, method, lambda, or structured scope: `&ddo->type.type().GetDerivedTypeSpec().typeSymbol() != &derivedType) {`.
  **L1997 CN**: 开始一个函数、方法、lambda 或结构化作用域：`&ddo->type.type().GetDerivedTypeSpec().typeSymbol() != &derivedType) {`。
- **L1998 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*errSym, finalName,`.
  **L1998 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*errSym, finalName,`。
- **L1999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FINAL subroutine '%s' of derived type '%s' must have a TYPE(%s) dummy argument"_err_en_US,`.
  **L1999 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FINAL subroutine '%s' of derived type '%s' must have a TYPE(%s) dummy argument"_err_en_US,`。
- **L2000 EN**: Executes a call or declaration centered on `subroutine.name`.
  **L2000 CN**: 执行以 `subroutine.name` 为核心的调用或声明。
- **L2001 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L2001 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L2002 EN**: Transitions from the previous branch into the alternative path.
  **L2002 CN**: 从前一个分支过渡到备选路径。
- **L2003 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2003 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2004 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2004 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2005 EN**: Continues the surrounding expression or declaration: `const auto *value{`.
  **L2005 CN**: 继续构造周围的表达式或声明：`const auto *value{`。
- **L2006 EN**: Executes a call or declaration centered on `ddo->type.type`.
  **L2006 CN**: 执行以 `ddo->type.type` 为核心的调用或声明。
- **L2007 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2007 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2008 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*errSym, finalName,`.
  **L2008 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*errSym, finalName,`。
- **L2009 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FINAL subroutine '%s' of derived type '%s' must have a dummy argument with an assumed LEN type parameter '%s=*'"_err_en_US,`.
  **L2009 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FINAL subroutine '%s' of derived type '%s' must have a dummy argument with an assumed LEN type parameter '%s=*'"_err_en_US,`。
- **L2010 EN**: Executes a call or declaration centered on `subroutine.name`.
  **L2010 CN**: 执行以 `subroutine.name` 为核心的调用或声明。
- **L2011 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L2011 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L2012 EN**: Closes the current lexical scope or compound statement.
  **L2012 CN**: 结束当前词法作用域或复合语句块。
- **L2013 EN**: Closes the current lexical scope or compound statement.
  **L2013 CN**: 结束当前词法作用域或复合语句块。
- **L2014 EN**: Closes the current lexical scope or compound statement.
  **L2014 CN**: 结束当前词法作用域或复合语句块。
- **L2015 EN**: Closes the current lexical scope or compound statement.
  **L2015 CN**: 结束当前词法作用域或复合语句块。
- **L2016 EN**: Returns from the current function with `ok`.
  **L2016 CN**: 以 `ok` 从当前函数返回。

### Lines 2017-2040

````cpp
}

bool CheckHelper::CheckDistinguishableFinals(const Symbol &f1,
    SourceName f1Name, const Symbol &f2, SourceName f2Name,
    const Symbol &derivedType) {
  const Procedure *p1{Characterize(f1)};
  const Procedure *p2{Characterize(f2)};
  if (p1 && p2) {
    if (characteristics::Distinguishable(context_.languageFeatures(), *p1, *p2)
            .value_or(false)) {
      return true;
    }
    if (auto *msg{messages_.Say(f1Name,
            "FINAL subroutines '%s' and '%s' of derived type '%s' cannot be distinguished by rank or KIND type parameter value"_err_en_US,
            f1Name, f2Name, derivedType.name())}) {
      msg->Attach(f2Name, "FINAL declaration of '%s'"_en_US, f2.name())
          .Attach(f1.name(), "Definition of '%s'"_en_US, f1Name)
          .Attach(f2.name(), "Definition of '%s'"_en_US, f2Name);
    }
  }
  return false;
}

void CheckHelper::CheckHostAssoc(
````
- **L2017 EN**: Closes the current lexical scope or compound statement.
  **L2017 CN**: 结束当前词法作用域或复合语句块。
- **L2018 EN**: Blank line separating nearby declarations or logic blocks.
  **L2018 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2019 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool CheckHelper::CheckDistinguishableFinals(const Symbol &f1,`.
  **L2019 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool CheckHelper::CheckDistinguishableFinals(const Symbol &f1,`。
- **L2020 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SourceName f1Name, const Symbol &f2, SourceName f2Name,`.
  **L2020 CN**: 继续一个多行参数列表、初始化器或聚合项：`SourceName f1Name, const Symbol &f2, SourceName f2Name,`。
- **L2021 EN**: Continues the surrounding expression or declaration: `const Symbol &derivedType) {`.
  **L2021 CN**: 继续构造周围的表达式或声明：`const Symbol &derivedType) {`。
- **L2022 EN**: Executes a call or declaration centered on `*p1{Characterize`.
  **L2022 CN**: 执行以 `*p1{Characterize` 为核心的调用或声明。
- **L2023 EN**: Executes a call or declaration centered on `*p2{Characterize`.
  **L2023 CN**: 执行以 `*p2{Characterize` 为核心的调用或声明。
- **L2024 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2024 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2025 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2025 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2026 EN**: Starts a function, method, lambda, or structured scope: `.value_or(false)) {`.
  **L2026 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.value_or(false)) {`。
- **L2027 EN**: Returns from the current function with `true`.
  **L2027 CN**: 以 `true` 从当前函数返回。
- **L2028 EN**: Closes the current lexical scope or compound statement.
  **L2028 CN**: 结束当前词法作用域或复合语句块。
- **L2029 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2029 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2030 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FINAL subroutines '%s' and '%s' of derived type '%s' cannot be distinguished by rank or KIND type parameter value"_err_en_US,`.
  **L2030 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FINAL subroutines '%s' and '%s' of derived type '%s' cannot be distinguished by rank or KIND type parameter value"_err_en_US,`。
- **L2031 EN**: Starts a function, method, lambda, or structured scope: `f1Name, f2Name, derivedType.name())}) {`.
  **L2031 CN**: 开始一个函数、方法、lambda 或结构化作用域：`f1Name, f2Name, derivedType.name())}) {`。
- **L2032 EN**: Continues logic associated with callable symbol `Attach`.
  **L2032 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L2033 EN**: Continues logic associated with callable symbol `Attach`.
  **L2033 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L2034 EN**: Executes a call or declaration centered on `.Attach`.
  **L2034 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L2035 EN**: Closes the current lexical scope or compound statement.
  **L2035 CN**: 结束当前词法作用域或复合语句块。
- **L2036 EN**: Closes the current lexical scope or compound statement.
  **L2036 CN**: 结束当前词法作用域或复合语句块。
- **L2037 EN**: Returns from the current function with `false`.
  **L2037 CN**: 以 `false` 从当前函数返回。
- **L2038 EN**: Closes the current lexical scope or compound statement.
  **L2038 CN**: 结束当前词法作用域或复合语句块。
- **L2039 EN**: Blank line separating nearby declarations or logic blocks.
  **L2039 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2040 EN**: Continues logic associated with callable symbol `CheckHostAssoc`.
  **L2040 CN**: 继续与可调用符号 `CheckHostAssoc` 相关的逻辑。

### Lines 2041-2064

````cpp
    const Symbol &symbol, const HostAssocDetails &details) {
  const Symbol &hostSymbol{details.symbol()};
  if (hostSymbol.test(Symbol::Flag::ImplicitOrError)) {
    if (details.implicitOrSpecExprError) {
      messages_.Say("Implicitly typed local entity '%s' not allowed in"
                    " specification expression"_err_en_US,
          symbol.name());
    } else if (details.implicitOrExplicitTypeError) {
      messages_.Say(
          "No explicit type declared for '%s'"_err_en_US, symbol.name());
    }
  }
}

void CheckHelper::CheckGeneric(
    const Symbol &symbol, const GenericDetails &details) {
  CheckSpecifics(symbol, details);
  common::visit(common::visitors{
                    [&](const common::DefinedIo &io) {
                      CheckDefinedIoProc(symbol, details, io);
                    },
                    [&](const GenericKind::OtherKind &other) {
                      if (other == GenericKind::OtherKind::Name) {
                        CheckGenericVsIntrinsic(symbol, details);
````
- **L2041 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, const HostAssocDetails &details) {`.
  **L2041 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, const HostAssocDetails &details) {`。
- **L2042 EN**: Executes a call or declaration centered on `&hostSymbol{details.symbol`.
  **L2042 CN**: 执行以 `&hostSymbol{details.symbol` 为核心的调用或声明。
- **L2043 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2043 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2044 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2044 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2045 EN**: Continues logic associated with callable symbol `Say`.
  **L2045 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2046 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" specification expression"_err_en_US,`.
  **L2046 CN**: 继续一个多行参数列表、初始化器或聚合项：`" specification expression"_err_en_US,`。
- **L2047 EN**: Executes a call or declaration centered on `symbol.name`.
  **L2047 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L2048 EN**: Transitions from the previous branch into an `else if` condition.
  **L2048 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2049 EN**: Continues logic associated with callable symbol `Say`.
  **L2049 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2050 EN**: Executes a call or declaration centered on `symbol.name`.
  **L2050 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L2051 EN**: Closes the current lexical scope or compound statement.
  **L2051 CN**: 结束当前词法作用域或复合语句块。
- **L2052 EN**: Closes the current lexical scope or compound statement.
  **L2052 CN**: 结束当前词法作用域或复合语句块。
- **L2053 EN**: Closes the current lexical scope or compound statement.
  **L2053 CN**: 结束当前词法作用域或复合语句块。
- **L2054 EN**: Blank line separating nearby declarations or logic blocks.
  **L2054 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2055 EN**: Continues logic associated with callable symbol `CheckGeneric`.
  **L2055 CN**: 继续与可调用符号 `CheckGeneric` 相关的逻辑。
- **L2056 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, const GenericDetails &details) {`.
  **L2056 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, const GenericDetails &details) {`。
- **L2057 EN**: Executes a call or declaration centered on `CheckSpecifics`.
  **L2057 CN**: 执行以 `CheckSpecifics` 为核心的调用或声明。
- **L2058 EN**: Starts a function, method, lambda, or structured scope: `common::visit(common::visitors{`.
  **L2058 CN**: 开始一个函数、方法、lambda 或结构化作用域：`common::visit(common::visitors{`。
- **L2059 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::DefinedIo &io) {`.
  **L2059 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::DefinedIo &io) {`。
- **L2060 EN**: Executes a call or declaration centered on `CheckDefinedIoProc`.
  **L2060 CN**: 执行以 `CheckDefinedIoProc` 为核心的调用或声明。
- **L2061 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2061 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L2062 EN**: Starts a function, method, lambda, or structured scope: `[&](const GenericKind::OtherKind &other) {`.
  **L2062 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const GenericKind::OtherKind &other) {`。
- **L2063 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2063 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2064 EN**: Executes a call or declaration centered on `CheckGenericVsIntrinsic`.
  **L2064 CN**: 执行以 `CheckGenericVsIntrinsic` 为核心的调用或声明。

### Lines 2065-2088

````cpp
                      }
                    },
                    [](const auto &) {},
                },
      details.kind().u);
  // Ensure that shadowed symbols are checked
  if (details.specific()) {
    Check(*details.specific());
  }
  if (details.derivedType()) {
    Check(*details.derivedType());
  }
}

// Check that the specifics of this generic are distinguishable from each other
void CheckHelper::CollectSpecifics(DistinguishabilityHelper &helper,
    const Symbol &generic, const GenericDetails &details) {
  GenericKind kind{details.kind()};
  for (const Symbol &specific : details.specificProcs()) {
    if (specific.attrs().test(Attr::ABSTRACT)) {
      if (auto *msg{messages_.Say(generic.name(),
              "Generic interface '%s' must not use abstract interface '%s' as a specific procedure"_err_en_US,
              generic.name(), specific.name())}) {
        msg->Attach(
````
- **L2065 EN**: Closes the current lexical scope or compound statement.
  **L2065 CN**: 结束当前词法作用域或复合语句块。
- **L2066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2066 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L2067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[](const auto &) {},`.
  **L2067 CN**: 继续一个多行参数列表、初始化器或聚合项：`[](const auto &) {},`。
- **L2068 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2068 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L2069 EN**: Executes a call or declaration centered on `details.kind`.
  **L2069 CN**: 执行以 `details.kind` 为核心的调用或声明。
- **L2070 EN**: Comment explains nearby logic, intent, or metadata: `Ensure that shadowed symbols are checked`.
  **L2070 CN**: 注释说明附近代码的逻辑、意图或元数据：`Ensure that shadowed symbols are checked`。
- **L2071 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2071 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2072 EN**: Executes a call or declaration centered on `Check`.
  **L2072 CN**: 执行以 `Check` 为核心的调用或声明。
- **L2073 EN**: Closes the current lexical scope or compound statement.
  **L2073 CN**: 结束当前词法作用域或复合语句块。
- **L2074 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2074 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2075 EN**: Executes a call or declaration centered on `Check`.
  **L2075 CN**: 执行以 `Check` 为核心的调用或声明。
- **L2076 EN**: Closes the current lexical scope or compound statement.
  **L2076 CN**: 结束当前词法作用域或复合语句块。
- **L2077 EN**: Closes the current lexical scope or compound statement.
  **L2077 CN**: 结束当前词法作用域或复合语句块。
- **L2078 EN**: Blank line separating nearby declarations or logic blocks.
  **L2078 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2079 EN**: Comment explains nearby logic, intent, or metadata: `Check that the specifics of this generic are distinguishable from each other`.
  **L2079 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check that the specifics of this generic are distinguishable from each other`。
- **L2080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckHelper::CollectSpecifics(DistinguishabilityHelper &helper,`.
  **L2080 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckHelper::CollectSpecifics(DistinguishabilityHelper &helper,`。
- **L2081 EN**: Continues the surrounding expression or declaration: `const Symbol &generic, const GenericDetails &details) {`.
  **L2081 CN**: 继续构造周围的表达式或声明：`const Symbol &generic, const GenericDetails &details) {`。
- **L2082 EN**: Executes a call or declaration centered on `kind{details.kind`.
  **L2082 CN**: 执行以 `kind{details.kind` 为核心的调用或声明。
- **L2083 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2083 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2084 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2084 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2085 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2085 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2086 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Generic interface '%s' must not use abstract interface '%s' as a specific procedure"_err_en_US,`.
  **L2086 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Generic interface '%s' must not use abstract interface '%s' as a specific procedure"_err_en_US,`。
- **L2087 EN**: Starts a function, method, lambda, or structured scope: `generic.name(), specific.name())}) {`.
  **L2087 CN**: 开始一个函数、方法、lambda 或结构化作用域：`generic.name(), specific.name())}) {`。
- **L2088 EN**: Continues logic associated with callable symbol `Attach`.
  **L2088 CN**: 继续与可调用符号 `Attach` 相关的逻辑。

### Lines 2089-2112

````cpp
            specific.name(), "Definition of '%s'"_en_US, specific.name());
      }
      continue;
    }
    if (specific.attrs().test(Attr::INTRINSIC)) {
      // GNU Fortran allows INTRINSIC procedures in generics.
      auto intrinsic{context_.intrinsics().IsSpecificIntrinsicFunction(
          specific.name().ToString())};
      if (intrinsic && !intrinsic->isRestrictedSpecific) {
        if (auto *msg{Warn(common::LanguageFeature::IntrinsicAsSpecific,
                specific.name(),
                "Specific procedure '%s' of generic interface '%s' should not be INTRINSIC"_port_en_US,
                specific.name(), generic.name())}) {
          msg->Attach(
              generic.name(), "Definition of '%s'"_en_US, generic.name());
        }
      } else {
        if (auto *msg{Warn(common::LanguageFeature::IntrinsicAsSpecific,
                specific.name(),
                "Procedure '%s' of generic interface '%s' is INTRINSIC but not an unrestricted specific intrinsic function"_port_en_US,
                specific.name(), generic.name())}) {
          msg->Attach(
              generic.name(), "Definition of '%s'"_en_US, generic.name());
        }
````
- **L2089 EN**: Executes a call or declaration centered on `specific.name`.
  **L2089 CN**: 执行以 `specific.name` 为核心的调用或声明。
- **L2090 EN**: Closes the current lexical scope or compound statement.
  **L2090 CN**: 结束当前词法作用域或复合语句块。
- **L2091 EN**: Skips to the next loop iteration.
  **L2091 CN**: 跳到下一次循环迭代。
- **L2092 EN**: Closes the current lexical scope or compound statement.
  **L2092 CN**: 结束当前词法作用域或复合语句块。
- **L2093 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2093 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2094 EN**: Comment explains nearby logic, intent, or metadata: `GNU Fortran allows INTRINSIC procedures in generics.`.
  **L2094 CN**: 注释说明附近代码的逻辑、意图或元数据：`GNU Fortran allows INTRINSIC procedures in generics.`。
- **L2095 EN**: Continues logic associated with callable symbol `intrinsics`.
  **L2095 CN**: 继续与可调用符号 `intrinsics` 相关的逻辑。
- **L2096 EN**: Executes a call or declaration centered on `specific.name`.
  **L2096 CN**: 执行以 `specific.name` 为核心的调用或声明。
- **L2097 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2097 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2098 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2098 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2099 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `specific.name(),`.
  **L2099 CN**: 继续一个多行参数列表、初始化器或聚合项：`specific.name(),`。
- **L2100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Specific procedure '%s' of generic interface '%s' should not be INTRINSIC"_port_en_US,`.
  **L2100 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Specific procedure '%s' of generic interface '%s' should not be INTRINSIC"_port_en_US,`。
- **L2101 EN**: Starts a function, method, lambda, or structured scope: `specific.name(), generic.name())}) {`.
  **L2101 CN**: 开始一个函数、方法、lambda 或结构化作用域：`specific.name(), generic.name())}) {`。
- **L2102 EN**: Continues logic associated with callable symbol `Attach`.
  **L2102 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L2103 EN**: Executes a call or declaration centered on `generic.name`.
  **L2103 CN**: 执行以 `generic.name` 为核心的调用或声明。
- **L2104 EN**: Closes the current lexical scope or compound statement.
  **L2104 CN**: 结束当前词法作用域或复合语句块。
- **L2105 EN**: Transitions from the previous branch into the alternative path.
  **L2105 CN**: 从前一个分支过渡到备选路径。
- **L2106 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2106 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `specific.name(),`.
  **L2107 CN**: 继续一个多行参数列表、初始化器或聚合项：`specific.name(),`。
- **L2108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Procedure '%s' of generic interface '%s' is INTRINSIC but not an unrestricted specific intrinsic function"_port_en_US,`.
  **L2108 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Procedure '%s' of generic interface '%s' is INTRINSIC but not an unrestricted specific intrinsic function"_port_en_US,`。
- **L2109 EN**: Starts a function, method, lambda, or structured scope: `specific.name(), generic.name())}) {`.
  **L2109 CN**: 开始一个函数、方法、lambda 或结构化作用域：`specific.name(), generic.name())}) {`。
- **L2110 EN**: Continues logic associated with callable symbol `Attach`.
  **L2110 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L2111 EN**: Executes a call or declaration centered on `generic.name`.
  **L2111 CN**: 执行以 `generic.name` 为核心的调用或声明。
- **L2112 EN**: Closes the current lexical scope or compound statement.
  **L2112 CN**: 结束当前词法作用域或复合语句块。

### Lines 2113-2136

````cpp
        continue;
      }
    }
    if (IsStmtFunction(specific)) {
      if (auto *msg{messages_.Say(specific.name(),
              "Specific procedure '%s' of generic interface '%s' may not be a statement function"_err_en_US,
              specific.name(), generic.name())}) {
        msg->Attach(generic.name(), "Definition of '%s'"_en_US, generic.name());
      }
      continue;
    }
    if (const Procedure *procedure{Characterize(specific)}) {
      if (procedure->HasExplicitInterface()) {
        helper.Add(generic, kind, specific, *procedure);
      } else {
        if (auto *msg{messages_.Say(specific.name(),
                "Specific procedure '%s' of generic interface '%s' must have an explicit interface"_err_en_US,
                specific.name(), generic.name())}) {
          msg->Attach(
              generic.name(), "Definition of '%s'"_en_US, generic.name());
        }
      }
    }
  }
````
- **L2113 EN**: Skips to the next loop iteration.
  **L2113 CN**: 跳到下一次循环迭代。
- **L2114 EN**: Closes the current lexical scope or compound statement.
  **L2114 CN**: 结束当前词法作用域或复合语句块。
- **L2115 EN**: Closes the current lexical scope or compound statement.
  **L2115 CN**: 结束当前词法作用域或复合语句块。
- **L2116 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2116 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2117 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2117 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Specific procedure '%s' of generic interface '%s' may not be a statement function"_err_en_US,`.
  **L2118 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Specific procedure '%s' of generic interface '%s' may not be a statement function"_err_en_US,`。
- **L2119 EN**: Starts a function, method, lambda, or structured scope: `specific.name(), generic.name())}) {`.
  **L2119 CN**: 开始一个函数、方法、lambda 或结构化作用域：`specific.name(), generic.name())}) {`。
- **L2120 EN**: Executes a call or declaration centered on `msg->Attach`.
  **L2120 CN**: 执行以 `msg->Attach` 为核心的调用或声明。
- **L2121 EN**: Closes the current lexical scope or compound statement.
  **L2121 CN**: 结束当前词法作用域或复合语句块。
- **L2122 EN**: Skips to the next loop iteration.
  **L2122 CN**: 跳到下一次循环迭代。
- **L2123 EN**: Closes the current lexical scope or compound statement.
  **L2123 CN**: 结束当前词法作用域或复合语句块。
- **L2124 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2124 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2125 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2125 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2126 EN**: Executes a call or declaration centered on `helper.Add`.
  **L2126 CN**: 执行以 `helper.Add` 为核心的调用或声明。
- **L2127 EN**: Transitions from the previous branch into the alternative path.
  **L2127 CN**: 从前一个分支过渡到备选路径。
- **L2128 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2128 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Specific procedure '%s' of generic interface '%s' must have an explicit interface"_err_en_US,`.
  **L2129 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Specific procedure '%s' of generic interface '%s' must have an explicit interface"_err_en_US,`。
- **L2130 EN**: Starts a function, method, lambda, or structured scope: `specific.name(), generic.name())}) {`.
  **L2130 CN**: 开始一个函数、方法、lambda 或结构化作用域：`specific.name(), generic.name())}) {`。
- **L2131 EN**: Continues logic associated with callable symbol `Attach`.
  **L2131 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L2132 EN**: Executes a call or declaration centered on `generic.name`.
  **L2132 CN**: 执行以 `generic.name` 为核心的调用或声明。
- **L2133 EN**: Closes the current lexical scope or compound statement.
  **L2133 CN**: 结束当前词法作用域或复合语句块。
- **L2134 EN**: Closes the current lexical scope or compound statement.
  **L2134 CN**: 结束当前词法作用域或复合语句块。
- **L2135 EN**: Closes the current lexical scope or compound statement.
  **L2135 CN**: 结束当前词法作用域或复合语句块。
- **L2136 EN**: Closes the current lexical scope or compound statement.
  **L2136 CN**: 结束当前词法作用域或复合语句块。

### Lines 2137-2160

````cpp
  if (const Scope * parent{generic.owner().GetDerivedTypeParent()}) {
    if (const Symbol * inherited{parent->FindComponent(generic.name())}) {
      if (IsAccessible(*inherited, generic.owner().parent())) {
        if (const auto *details{inherited->detailsIf<GenericDetails>()}) {
          // Include specifics of inherited generic of the same name, too
          CollectSpecifics(helper, *inherited, *details);
        }
      }
    }
  }
}

void CheckHelper::CheckSpecifics(
    const Symbol &generic, const GenericDetails &details) {
  DistinguishabilityHelper helper{context_};
  CollectSpecifics(helper, generic, details);
  helper.Check(generic.owner());
}

static bool CUDAHostDeviceDiffer(
    const Procedure &proc, const DummyDataObject &arg) {
  auto procCUDA{
      proc.cudaSubprogramAttrs.value_or(common::CUDASubprogramAttrs::Host)};
  bool procIsHostOnly{procCUDA == common::CUDASubprogramAttrs::Host};
````
- **L2137 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2137 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2138 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2138 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2139 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2139 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2140 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2140 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2141 EN**: Comment explains nearby logic, intent, or metadata: `Include specifics of inherited generic of the same name, too`.
  **L2141 CN**: 注释说明附近代码的逻辑、意图或元数据：`Include specifics of inherited generic of the same name, too`。
- **L2142 EN**: Executes a call or declaration centered on `CollectSpecifics`.
  **L2142 CN**: 执行以 `CollectSpecifics` 为核心的调用或声明。
- **L2143 EN**: Closes the current lexical scope or compound statement.
  **L2143 CN**: 结束当前词法作用域或复合语句块。
- **L2144 EN**: Closes the current lexical scope or compound statement.
  **L2144 CN**: 结束当前词法作用域或复合语句块。
- **L2145 EN**: Closes the current lexical scope or compound statement.
  **L2145 CN**: 结束当前词法作用域或复合语句块。
- **L2146 EN**: Closes the current lexical scope or compound statement.
  **L2146 CN**: 结束当前词法作用域或复合语句块。
- **L2147 EN**: Closes the current lexical scope or compound statement.
  **L2147 CN**: 结束当前词法作用域或复合语句块。
- **L2148 EN**: Blank line separating nearby declarations or logic blocks.
  **L2148 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2149 EN**: Continues logic associated with callable symbol `CheckSpecifics`.
  **L2149 CN**: 继续与可调用符号 `CheckSpecifics` 相关的逻辑。
- **L2150 EN**: Continues the surrounding expression or declaration: `const Symbol &generic, const GenericDetails &details) {`.
  **L2150 CN**: 继续构造周围的表达式或声明：`const Symbol &generic, const GenericDetails &details) {`。
- **L2151 EN**: Executes a standalone statement or declaration: `DistinguishabilityHelper helper{context_};`.
  **L2151 CN**: 执行一条独立语句或声明：`DistinguishabilityHelper helper{context_};`。
- **L2152 EN**: Executes a call or declaration centered on `CollectSpecifics`.
  **L2152 CN**: 执行以 `CollectSpecifics` 为核心的调用或声明。
- **L2153 EN**: Executes a call or declaration centered on `helper.Check`.
  **L2153 CN**: 执行以 `helper.Check` 为核心的调用或声明。
- **L2154 EN**: Closes the current lexical scope or compound statement.
  **L2154 CN**: 结束当前词法作用域或复合语句块。
- **L2155 EN**: Blank line separating nearby declarations or logic blocks.
  **L2155 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2156 EN**: Continues logic associated with callable symbol `CUDAHostDeviceDiffer`.
  **L2156 CN**: 继续与可调用符号 `CUDAHostDeviceDiffer` 相关的逻辑。
- **L2157 EN**: Continues the surrounding expression or declaration: `const Procedure &proc, const DummyDataObject &arg) {`.
  **L2157 CN**: 继续构造周围的表达式或声明：`const Procedure &proc, const DummyDataObject &arg) {`。
- **L2158 EN**: Continues the surrounding expression or declaration: `auto procCUDA{`.
  **L2158 CN**: 继续构造周围的表达式或声明：`auto procCUDA{`。
- **L2159 EN**: Executes a call or declaration centered on `proc.cudaSubprogramAttrs.value_or`.
  **L2159 CN**: 执行以 `proc.cudaSubprogramAttrs.value_or` 为核心的调用或声明。
- **L2160 EN**: Executes a standalone statement or declaration: `bool procIsHostOnly{procCUDA == common::CUDASubprogramAttrs::Host};`.
  **L2160 CN**: 执行一条独立语句或声明：`bool procIsHostOnly{procCUDA == common::CUDASubprogramAttrs::Host};`。

### Lines 2161-2184

````cpp
  bool procIsDeviceOnly{
      !procIsHostOnly && procCUDA != common::CUDASubprogramAttrs::HostDevice};
  const auto &argCUDA{arg.cudaDataAttr};
  bool argIsHostOnly{!argCUDA || *argCUDA == common::CUDADataAttr::Pinned};
  bool argIsDeviceOnly{(!argCUDA && procIsDeviceOnly) ||
      (argCUDA &&
          (*argCUDA != common::CUDADataAttr::Managed &&
              *argCUDA != common::CUDADataAttr::Pinned &&
              *argCUDA != common::CUDADataAttr::Unified))};
  return (procIsHostOnly && argIsDeviceOnly) ||
      (procIsDeviceOnly && argIsHostOnly);
}

static bool ConflictsWithIntrinsicAssignment(const Procedure &proc) {
  const auto &lhsData{std::get<DummyDataObject>(proc.dummyArguments[0].u)};
  const auto &lhsTnS{lhsData.type};
  const auto &rhsData{std::get<DummyDataObject>(proc.dummyArguments[1].u)};
  const auto &rhsTnS{rhsData.type};
  return !CUDAHostDeviceDiffer(proc, lhsData) &&
      !CUDAHostDeviceDiffer(proc, rhsData) &&
      Tristate::No ==
      IsDefinedAssignment(
          lhsTnS.type(), lhsTnS.Rank(), rhsTnS.type(), rhsTnS.Rank());
}
````
- **L2161 EN**: Continues the surrounding expression or declaration: `bool procIsDeviceOnly{`.
  **L2161 CN**: 继续构造周围的表达式或声明：`bool procIsDeviceOnly{`。
- **L2162 EN**: Executes a standalone statement or declaration: `!procIsHostOnly && procCUDA != common::CUDASubprogramAttrs::HostDevice};`.
  **L2162 CN**: 执行一条独立语句或声明：`!procIsHostOnly && procCUDA != common::CUDASubprogramAttrs::HostDevice};`。
- **L2163 EN**: Executes a standalone statement or declaration: `const auto &argCUDA{arg.cudaDataAttr};`.
  **L2163 CN**: 执行一条独立语句或声明：`const auto &argCUDA{arg.cudaDataAttr};`。
- **L2164 EN**: Executes a standalone statement or declaration: `bool argIsHostOnly{!argCUDA || *argCUDA == common::CUDADataAttr::Pinned};`.
  **L2164 CN**: 执行一条独立语句或声明：`bool argIsHostOnly{!argCUDA || *argCUDA == common::CUDADataAttr::Pinned};`。
- **L2165 EN**: Continues the surrounding expression or declaration: `bool argIsDeviceOnly{(!argCUDA && procIsDeviceOnly) ||`.
  **L2165 CN**: 继续构造周围的表达式或声明：`bool argIsDeviceOnly{(!argCUDA && procIsDeviceOnly) ||`。
- **L2166 EN**: Continues the surrounding expression or declaration: `(argCUDA &&`.
  **L2166 CN**: 继续构造周围的表达式或声明：`(argCUDA &&`。
- **L2167 EN**: Continues the surrounding expression or declaration: `(*argCUDA != common::CUDADataAttr::Managed &&`.
  **L2167 CN**: 继续构造周围的表达式或声明：`(*argCUDA != common::CUDADataAttr::Managed &&`。
- **L2168 EN**: Comment explains nearby logic, intent, or metadata: `argCUDA != common::CUDADataAttr::Pinned &&`.
  **L2168 CN**: 注释说明附近代码的逻辑、意图或元数据：`argCUDA != common::CUDADataAttr::Pinned &&`。
- **L2169 EN**: Comment explains nearby logic, intent, or metadata: `argCUDA != common::CUDADataAttr::Unified))};`.
  **L2169 CN**: 注释说明附近代码的逻辑、意图或元数据：`argCUDA != common::CUDADataAttr::Unified))};`。
- **L2170 EN**: Returns from the current function with `(procIsHostOnly && argIsDeviceOnly) ||`.
  **L2170 CN**: 以 `(procIsHostOnly && argIsDeviceOnly) ||` 从当前函数返回。
- **L2171 EN**: Executes a call or declaration centered on `statement`.
  **L2171 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2172 EN**: Closes the current lexical scope or compound statement.
  **L2172 CN**: 结束当前词法作用域或复合语句块。
- **L2173 EN**: Blank line separating nearby declarations or logic blocks.
  **L2173 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2174 EN**: Starts a function, method, lambda, or structured scope: `static bool ConflictsWithIntrinsicAssignment(const Procedure &proc) {`.
  **L2174 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool ConflictsWithIntrinsicAssignment(const Procedure &proc) {`。
- **L2175 EN**: Executes a call or declaration centered on `&lhsData{std::get<DummyDataObject>`.
  **L2175 CN**: 执行以 `&lhsData{std::get<DummyDataObject>` 为核心的调用或声明。
- **L2176 EN**: Executes a standalone statement or declaration: `const auto &lhsTnS{lhsData.type};`.
  **L2176 CN**: 执行一条独立语句或声明：`const auto &lhsTnS{lhsData.type};`。
- **L2177 EN**: Executes a call or declaration centered on `&rhsData{std::get<DummyDataObject>`.
  **L2177 CN**: 执行以 `&rhsData{std::get<DummyDataObject>` 为核心的调用或声明。
- **L2178 EN**: Executes a standalone statement or declaration: `const auto &rhsTnS{rhsData.type};`.
  **L2178 CN**: 执行一条独立语句或声明：`const auto &rhsTnS{rhsData.type};`。
- **L2179 EN**: Returns from the current function with `!CUDAHostDeviceDiffer(proc, lhsData) &&`.
  **L2179 CN**: 以 `!CUDAHostDeviceDiffer(proc, lhsData) &&` 从当前函数返回。
- **L2180 EN**: Continues logic associated with callable symbol `CUDAHostDeviceDiffer`.
  **L2180 CN**: 继续与可调用符号 `CUDAHostDeviceDiffer` 相关的逻辑。
- **L2181 EN**: Continues the surrounding expression or declaration: `Tristate::No ==`.
  **L2181 CN**: 继续构造周围的表达式或声明：`Tristate::No ==`。
- **L2182 EN**: Continues logic associated with callable symbol `IsDefinedAssignment`.
  **L2182 CN**: 继续与可调用符号 `IsDefinedAssignment` 相关的逻辑。
- **L2183 EN**: Executes a call or declaration centered on `lhsTnS.type`.
  **L2183 CN**: 执行以 `lhsTnS.type` 为核心的调用或声明。
- **L2184 EN**: Closes the current lexical scope or compound statement.
  **L2184 CN**: 结束当前词法作用域或复合语句块。

### Lines 2185-2208

````cpp

static bool ConflictsWithIntrinsicOperator(
    const GenericKind &kind, const Procedure &proc, SemanticsContext &context) {
  if (!kind.IsIntrinsicOperator()) {
    return false;
  }
  const auto &arg0Data{std::get<DummyDataObject>(proc.dummyArguments[0].u)};
  if (CUDAHostDeviceDiffer(proc, arg0Data)) {
    return false;
  }
  const auto &arg0TnS{arg0Data.type};
  auto type0{arg0TnS.type()};
  if (proc.dummyArguments.size() == 1) { // unary
    return common::visit(
        common::visitors{
            [&](common::NumericOperator) { return IsIntrinsicNumeric(type0); },
            [&](common::LogicalOperator) { return IsIntrinsicLogical(type0); },
            [](const auto &) -> bool { DIE("bad generic kind"); },
        },
        kind.u);
  } else { // binary
    int rank0{arg0TnS.Rank()};
    const auto &arg1Data{std::get<DummyDataObject>(proc.dummyArguments[1].u)};
    if (CUDAHostDeviceDiffer(proc, arg1Data)) {
````
- **L2185 EN**: Blank line separating nearby declarations or logic blocks.
  **L2185 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2186 EN**: Continues logic associated with callable symbol `ConflictsWithIntrinsicOperator`.
  **L2186 CN**: 继续与可调用符号 `ConflictsWithIntrinsicOperator` 相关的逻辑。
- **L2187 EN**: Continues the surrounding expression or declaration: `const GenericKind &kind, const Procedure &proc, SemanticsContext &context) {`.
  **L2187 CN**: 继续构造周围的表达式或声明：`const GenericKind &kind, const Procedure &proc, SemanticsContext &context) {`。
- **L2188 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2188 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2189 EN**: Returns from the current function with `false`.
  **L2189 CN**: 以 `false` 从当前函数返回。
- **L2190 EN**: Closes the current lexical scope or compound statement.
  **L2190 CN**: 结束当前词法作用域或复合语句块。
- **L2191 EN**: Executes a call or declaration centered on `&arg0Data{std::get<DummyDataObject>`.
  **L2191 CN**: 执行以 `&arg0Data{std::get<DummyDataObject>` 为核心的调用或声明。
- **L2192 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2192 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2193 EN**: Returns from the current function with `false`.
  **L2193 CN**: 以 `false` 从当前函数返回。
- **L2194 EN**: Closes the current lexical scope or compound statement.
  **L2194 CN**: 结束当前词法作用域或复合语句块。
- **L2195 EN**: Executes a standalone statement or declaration: `const auto &arg0TnS{arg0Data.type};`.
  **L2195 CN**: 执行一条独立语句或声明：`const auto &arg0TnS{arg0Data.type};`。
- **L2196 EN**: Executes a call or declaration centered on `type0{arg0TnS.type`.
  **L2196 CN**: 执行以 `type0{arg0TnS.type` 为核心的调用或声明。
- **L2197 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2197 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2198 EN**: Returns from the current function with `common::visit(`.
  **L2198 CN**: 以 `common::visit(` 从当前函数返回。
- **L2199 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L2199 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L2200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](common::NumericOperator) { return IsIntrinsicNumeric(type0); },`.
  **L2200 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](common::NumericOperator) { return IsIntrinsicNumeric(type0); },`。
- **L2201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](common::LogicalOperator) { return IsIntrinsicLogical(type0); },`.
  **L2201 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](common::LogicalOperator) { return IsIntrinsicLogical(type0); },`。
- **L2202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[](const auto &) -> bool { DIE("bad generic kind"); },`.
  **L2202 CN**: 继续一个多行参数列表、初始化器或聚合项：`[](const auto &) -> bool { DIE("bad generic kind"); },`。
- **L2203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2203 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L2204 EN**: Executes a standalone statement or declaration: `kind.u);`.
  **L2204 CN**: 执行一条独立语句或声明：`kind.u);`。
- **L2205 EN**: Transitions from the previous branch into the alternative path.
  **L2205 CN**: 从前一个分支过渡到备选路径。
- **L2206 EN**: Executes a call or declaration centered on `rank0{arg0TnS.Rank`.
  **L2206 CN**: 执行以 `rank0{arg0TnS.Rank` 为核心的调用或声明。
- **L2207 EN**: Executes a call or declaration centered on `&arg1Data{std::get<DummyDataObject>`.
  **L2207 CN**: 执行以 `&arg1Data{std::get<DummyDataObject>` 为核心的调用或声明。
- **L2208 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2208 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2209-2232

````cpp
      return false;
    }
    const auto &arg1TnS{arg1Data.type};
    auto type1{arg1TnS.type()};
    int rank1{arg1TnS.Rank()};
    return common::visit(
        common::visitors{
            [&](common::NumericOperator) {
              return IsIntrinsicNumeric(type0, rank0, type1, rank1);
            },
            [&](common::LogicalOperator) {
              return IsIntrinsicLogical(type0, rank0, type1, rank1);
            },
            [&](common::RelationalOperator opr) {
              return IsIntrinsicRelational(opr, type0, rank0, type1, rank1);
            },
            [&](GenericKind::OtherKind x) {
              CHECK(x == GenericKind::OtherKind::Concat);
              return IsIntrinsicConcat(type0, rank0, type1, rank1);
            },
            [](const auto &) -> bool { DIE("bad generic kind"); },
        },
        kind.u);
  }
````
- **L2209 EN**: Returns from the current function with `false`.
  **L2209 CN**: 以 `false` 从当前函数返回。
- **L2210 EN**: Closes the current lexical scope or compound statement.
  **L2210 CN**: 结束当前词法作用域或复合语句块。
- **L2211 EN**: Executes a standalone statement or declaration: `const auto &arg1TnS{arg1Data.type};`.
  **L2211 CN**: 执行一条独立语句或声明：`const auto &arg1TnS{arg1Data.type};`。
- **L2212 EN**: Executes a call or declaration centered on `type1{arg1TnS.type`.
  **L2212 CN**: 执行以 `type1{arg1TnS.type` 为核心的调用或声明。
- **L2213 EN**: Executes a call or declaration centered on `rank1{arg1TnS.Rank`.
  **L2213 CN**: 执行以 `rank1{arg1TnS.Rank` 为核心的调用或声明。
- **L2214 EN**: Returns from the current function with `common::visit(`.
  **L2214 CN**: 以 `common::visit(` 从当前函数返回。
- **L2215 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L2215 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L2216 EN**: Starts a function, method, lambda, or structured scope: `[&](common::NumericOperator) {`.
  **L2216 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](common::NumericOperator) {`。
- **L2217 EN**: Returns from the current function with `IsIntrinsicNumeric(type0, rank0, type1, rank1)`.
  **L2217 CN**: 以 `IsIntrinsicNumeric(type0, rank0, type1, rank1)` 从当前函数返回。
- **L2218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2218 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L2219 EN**: Starts a function, method, lambda, or structured scope: `[&](common::LogicalOperator) {`.
  **L2219 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](common::LogicalOperator) {`。
- **L2220 EN**: Returns from the current function with `IsIntrinsicLogical(type0, rank0, type1, rank1)`.
  **L2220 CN**: 以 `IsIntrinsicLogical(type0, rank0, type1, rank1)` 从当前函数返回。
- **L2221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2221 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L2222 EN**: Starts a function, method, lambda, or structured scope: `[&](common::RelationalOperator opr) {`.
  **L2222 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](common::RelationalOperator opr) {`。
- **L2223 EN**: Returns from the current function with `IsIntrinsicRelational(opr, type0, rank0, type1, rank1)`.
  **L2223 CN**: 以 `IsIntrinsicRelational(opr, type0, rank0, type1, rank1)` 从当前函数返回。
- **L2224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2224 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L2225 EN**: Starts a function, method, lambda, or structured scope: `[&](GenericKind::OtherKind x) {`.
  **L2225 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](GenericKind::OtherKind x) {`。
- **L2226 EN**: Executes a call or declaration centered on `CHECK`.
  **L2226 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2227 EN**: Returns from the current function with `IsIntrinsicConcat(type0, rank0, type1, rank1)`.
  **L2227 CN**: 以 `IsIntrinsicConcat(type0, rank0, type1, rank1)` 从当前函数返回。
- **L2228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2228 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L2229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[](const auto &) -> bool { DIE("bad generic kind"); },`.
  **L2229 CN**: 继续一个多行参数列表、初始化器或聚合项：`[](const auto &) -> bool { DIE("bad generic kind"); },`。
- **L2230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2230 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L2231 EN**: Executes a standalone statement or declaration: `kind.u);`.
  **L2231 CN**: 执行一条独立语句或声明：`kind.u);`。
- **L2232 EN**: Closes the current lexical scope or compound statement.
  **L2232 CN**: 结束当前词法作用域或复合语句块。

### Lines 2233-2256

````cpp
}

// Check if this procedure can be used for defined operators (see 15.4.3.4.2).
bool CheckHelper::CheckDefinedOperator(SourceName opName, GenericKind kind,
    const Symbol &specific, const Procedure &proc) {
  if (context_.HasError(specific)) {
    return false;
  }
  std::optional<parser::MessageFixedText> msg;
  auto checkDefinedOperatorArgs{
      [&](SourceName opName, const Symbol &specific, const Procedure &proc) {
        bool arg0Defined{CheckDefinedOperatorArg(opName, specific, proc, 0)};
        bool arg1Defined{CheckDefinedOperatorArg(opName, specific, proc, 1)};
        return arg0Defined && arg1Defined;
      }};
  if (specific.attrs().test(Attr::NOPASS)) { // C774
    msg = "%s procedure '%s' may not have NOPASS attribute"_err_en_US;
  } else if (!proc.functionResult.has_value()) {
    msg = "%s procedure '%s' must be a function"_err_en_US;
  } else if (proc.functionResult->IsAssumedLengthCharacter()) {
    const auto *subpDetails{specific.detailsIf<SubprogramDetails>()};
    if (subpDetails && !subpDetails->isDummy() && subpDetails->isInterface()) {
      // Error is caught by more general test for interfaces with
      // assumed-length character function results
````
- **L2233 EN**: Closes the current lexical scope or compound statement.
  **L2233 CN**: 结束当前词法作用域或复合语句块。
- **L2234 EN**: Blank line separating nearby declarations or logic blocks.
  **L2234 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2235 EN**: Comment explains nearby logic, intent, or metadata: `Check if this procedure can be used for defined operators (see 15.4.3.4.2).`.
  **L2235 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check if this procedure can be used for defined operators (see 15.4.3.4.2).`。
- **L2236 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool CheckHelper::CheckDefinedOperator(SourceName opName, GenericKind kind,`.
  **L2236 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool CheckHelper::CheckDefinedOperator(SourceName opName, GenericKind kind,`。
- **L2237 EN**: Continues the surrounding expression or declaration: `const Symbol &specific, const Procedure &proc) {`.
  **L2237 CN**: 继续构造周围的表达式或声明：`const Symbol &specific, const Procedure &proc) {`。
- **L2238 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2238 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2239 EN**: Returns from the current function with `false`.
  **L2239 CN**: 以 `false` 从当前函数返回。
- **L2240 EN**: Closes the current lexical scope or compound statement.
  **L2240 CN**: 结束当前词法作用域或复合语句块。
- **L2241 EN**: Executes a standalone statement or declaration: `std::optional<parser::MessageFixedText> msg;`.
  **L2241 CN**: 执行一条独立语句或声明：`std::optional<parser::MessageFixedText> msg;`。
- **L2242 EN**: Continues the surrounding expression or declaration: `auto checkDefinedOperatorArgs{`.
  **L2242 CN**: 继续构造周围的表达式或声明：`auto checkDefinedOperatorArgs{`。
- **L2243 EN**: Starts a function, method, lambda, or structured scope: `[&](SourceName opName, const Symbol &specific, const Procedure &proc) {`.
  **L2243 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](SourceName opName, const Symbol &specific, const Procedure &proc) {`。
- **L2244 EN**: Executes a call or declaration centered on `arg0Defined{CheckDefinedOperatorArg`.
  **L2244 CN**: 执行以 `arg0Defined{CheckDefinedOperatorArg` 为核心的调用或声明。
- **L2245 EN**: Executes a call or declaration centered on `arg1Defined{CheckDefinedOperatorArg`.
  **L2245 CN**: 执行以 `arg1Defined{CheckDefinedOperatorArg` 为核心的调用或声明。
- **L2246 EN**: Returns from the current function with `arg0Defined && arg1Defined`.
  **L2246 CN**: 以 `arg0Defined && arg1Defined` 从当前函数返回。
- **L2247 EN**: Executes a standalone statement or declaration: `}};`.
  **L2247 CN**: 执行一条独立语句或声明：`}};`。
- **L2248 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2248 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2249 EN**: Executes a standalone statement or declaration: `msg = "%s procedure '%s' may not have NOPASS attribute"_err_en_US;`.
  **L2249 CN**: 执行一条独立语句或声明：`msg = "%s procedure '%s' may not have NOPASS attribute"_err_en_US;`。
- **L2250 EN**: Transitions from the previous branch into an `else if` condition.
  **L2250 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2251 EN**: Executes a standalone statement or declaration: `msg = "%s procedure '%s' must be a function"_err_en_US;`.
  **L2251 CN**: 执行一条独立语句或声明：`msg = "%s procedure '%s' must be a function"_err_en_US;`。
- **L2252 EN**: Transitions from the previous branch into an `else if` condition.
  **L2252 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2253 EN**: Executes a call or declaration centered on `*subpDetails{specific.detailsIf<SubprogramDetails>`.
  **L2253 CN**: 执行以 `*subpDetails{specific.detailsIf<SubprogramDetails>` 为核心的调用或声明。
- **L2254 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2254 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2255 EN**: Comment explains nearby logic, intent, or metadata: `Error is caught by more general test for interfaces with`.
  **L2255 CN**: 注释说明附近代码的逻辑、意图或元数据：`Error is caught by more general test for interfaces with`。
- **L2256 EN**: Comment explains nearby logic, intent, or metadata: `assumed-length character function results`.
  **L2256 CN**: 注释说明附近代码的逻辑、意图或元数据：`assumed-length character function results`。

### Lines 2257-2280

````cpp
      return true;
    }
    msg = "%s function '%s' may not have assumed-length CHARACTER(*)"
          " result"_err_en_US;
  } else if (auto m{CheckNumberOfArgs(kind, proc.dummyArguments.size())}) {
    if (m->IsFatal()) {
      msg = *m;
    } else {
      evaluate::AttachDeclaration(
          Warn(common::UsageWarning::DefinedOperatorArgs, specific.name(),
              std::move(*m), MakeOpName(opName), specific.name()),
          specific);
      return true;
    }
  } else if (!checkDefinedOperatorArgs(opName, specific, proc)) {
    return false; // error was reported
  } else if (ConflictsWithIntrinsicOperator(kind, proc, context_)) {
    msg = "%s function '%s' conflicts with intrinsic operator"_err_en_US;
  }
  if (msg) {
    SayWithDeclaration(
        specific, std::move(*msg), MakeOpName(opName), specific.name());
    context_.SetError(specific);
    return false;
````
- **L2257 EN**: Returns from the current function with `true`.
  **L2257 CN**: 以 `true` 从当前函数返回。
- **L2258 EN**: Closes the current lexical scope or compound statement.
  **L2258 CN**: 结束当前词法作用域或复合语句块。
- **L2259 EN**: Continues logic associated with callable symbol `CHARACTER`.
  **L2259 CN**: 继续与可调用符号 `CHARACTER` 相关的逻辑。
- **L2260 EN**: Executes a standalone statement or declaration: `" result"_err_en_US;`.
  **L2260 CN**: 执行一条独立语句或声明：`" result"_err_en_US;`。
- **L2261 EN**: Transitions from the previous branch into an `else if` condition.
  **L2261 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2262 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2262 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2263 EN**: Executes a standalone statement or declaration: `msg = *m;`.
  **L2263 CN**: 执行一条独立语句或声明：`msg = *m;`。
- **L2264 EN**: Transitions from the previous branch into the alternative path.
  **L2264 CN**: 从前一个分支过渡到备选路径。
- **L2265 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L2265 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L2266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::DefinedOperatorArgs, specific.name(),`.
  **L2266 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::DefinedOperatorArgs, specific.name(),`。
- **L2267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::move(*m), MakeOpName(opName), specific.name()),`.
  **L2267 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::move(*m), MakeOpName(opName), specific.name()),`。
- **L2268 EN**: Executes a standalone statement or declaration: `specific);`.
  **L2268 CN**: 执行一条独立语句或声明：`specific);`。
- **L2269 EN**: Returns from the current function with `true`.
  **L2269 CN**: 以 `true` 从当前函数返回。
- **L2270 EN**: Closes the current lexical scope or compound statement.
  **L2270 CN**: 结束当前词法作用域或复合语句块。
- **L2271 EN**: Transitions from the previous branch into an `else if` condition.
  **L2271 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2272 EN**: Returns from the current function with `false; // error was reported`.
  **L2272 CN**: 以 `false; // error was reported` 从当前函数返回。
- **L2273 EN**: Transitions from the previous branch into an `else if` condition.
  **L2273 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2274 EN**: Executes a standalone statement or declaration: `msg = "%s function '%s' conflicts with intrinsic operator"_err_en_US;`.
  **L2274 CN**: 执行一条独立语句或声明：`msg = "%s function '%s' conflicts with intrinsic operator"_err_en_US;`。
- **L2275 EN**: Closes the current lexical scope or compound statement.
  **L2275 CN**: 结束当前词法作用域或复合语句块。
- **L2276 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2276 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2277 EN**: Continues logic associated with callable symbol `SayWithDeclaration`.
  **L2277 CN**: 继续与可调用符号 `SayWithDeclaration` 相关的逻辑。
- **L2278 EN**: Executes a call or declaration centered on `std::move`.
  **L2278 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L2279 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L2279 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L2280 EN**: Returns from the current function with `false`.
  **L2280 CN**: 以 `false` 从当前函数返回。

### Lines 2281-2304

````cpp
  }
  return true;
}

// If the number of arguments is wrong for this intrinsic operator, return
// false and return the error message in msg.
std::optional<parser::MessageFixedText> CheckHelper::CheckNumberOfArgs(
    const GenericKind &kind, std::size_t nargs) {
  if (!kind.IsIntrinsicOperator()) {
    if (nargs < 1 || nargs > 2) {
      if (context_.ShouldWarn(common::UsageWarning::DefinedOperatorArgs)) {
        return "%s function '%s' should have 1 or 2 dummy arguments"_warn_en_US;
      }
    }
    return std::nullopt;
  }
  std::size_t min{2}, max{2}; // allowed number of args; default is binary
  common::visit(common::visitors{
                    [&](const common::NumericOperator &x) {
                      if (x == common::NumericOperator::Add ||
                          x == common::NumericOperator::Subtract) {
                        min = 1; // + and - are unary or binary
                      }
                    },
````
- **L2281 EN**: Closes the current lexical scope or compound statement.
  **L2281 CN**: 结束当前词法作用域或复合语句块。
- **L2282 EN**: Returns from the current function with `true`.
  **L2282 CN**: 以 `true` 从当前函数返回。
- **L2283 EN**: Closes the current lexical scope or compound statement.
  **L2283 CN**: 结束当前词法作用域或复合语句块。
- **L2284 EN**: Blank line separating nearby declarations or logic blocks.
  **L2284 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2285 EN**: Comment explains nearby logic, intent, or metadata: `If the number of arguments is wrong for this intrinsic operator, return`.
  **L2285 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the number of arguments is wrong for this intrinsic operator, return`。
- **L2286 EN**: Comment explains nearby logic, intent, or metadata: `false and return the error message in msg.`.
  **L2286 CN**: 注释说明附近代码的逻辑、意图或元数据：`false and return the error message in msg.`。
- **L2287 EN**: Continues logic associated with callable symbol `CheckNumberOfArgs`.
  **L2287 CN**: 继续与可调用符号 `CheckNumberOfArgs` 相关的逻辑。
- **L2288 EN**: Continues the surrounding expression or declaration: `const GenericKind &kind, std::size_t nargs) {`.
  **L2288 CN**: 继续构造周围的表达式或声明：`const GenericKind &kind, std::size_t nargs) {`。
- **L2289 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2289 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2290 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2290 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2291 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2291 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2292 EN**: Returns from the current function with `"%s function '%s' should have 1 or 2 dummy arguments"_warn_en_US`.
  **L2292 CN**: 以 `"%s function '%s' should have 1 or 2 dummy arguments"_warn_en_US` 从当前函数返回。
- **L2293 EN**: Closes the current lexical scope or compound statement.
  **L2293 CN**: 结束当前词法作用域或复合语句块。
- **L2294 EN**: Closes the current lexical scope or compound statement.
  **L2294 CN**: 结束当前词法作用域或复合语句块。
- **L2295 EN**: Returns from the current function with `std::nullopt`.
  **L2295 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2296 EN**: Closes the current lexical scope or compound statement.
  **L2296 CN**: 结束当前词法作用域或复合语句块。
- **L2297 EN**: Continues the surrounding expression or declaration: `std::size_t min{2}, max{2}; // allowed number of args; default is binary`.
  **L2297 CN**: 继续构造周围的表达式或声明：`std::size_t min{2}, max{2}; // allowed number of args; default is binary`。
- **L2298 EN**: Starts a function, method, lambda, or structured scope: `common::visit(common::visitors{`.
  **L2298 CN**: 开始一个函数、方法、lambda 或结构化作用域：`common::visit(common::visitors{`。
- **L2299 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::NumericOperator &x) {`.
  **L2299 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::NumericOperator &x) {`。
- **L2300 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2300 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2301 EN**: Continues the surrounding expression or declaration: `x == common::NumericOperator::Subtract) {`.
  **L2301 CN**: 继续构造周围的表达式或声明：`x == common::NumericOperator::Subtract) {`。
- **L2302 EN**: Continues the surrounding expression or declaration: `min = 1; // + and - are unary or binary`.
  **L2302 CN**: 继续构造周围的表达式或声明：`min = 1; // + and - are unary or binary`。
- **L2303 EN**: Closes the current lexical scope or compound statement.
  **L2303 CN**: 结束当前词法作用域或复合语句块。
- **L2304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2304 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。

### Lines 2305-2328

````cpp
                    [&](const common::LogicalOperator &x) {
                      if (x == common::LogicalOperator::Not) {
                        min = 1; // .NOT. is unary
                        max = 1;
                      }
                    },
                    [](const common::RelationalOperator &) {
                      // all are binary
                    },
                    [](const GenericKind::OtherKind &x) {
                      CHECK(x == GenericKind::OtherKind::Concat);
                    },
                    [](const auto &) { DIE("expected intrinsic operator"); },
                },
      kind.u);
  if (nargs >= min && nargs <= max) {
    return std::nullopt;
  } else if (max == 1) {
    return "%s function '%s' must have one dummy argument"_err_en_US;
  } else if (min == 2) {
    return "%s function '%s' must have two dummy arguments"_err_en_US;
  } else {
    return "%s function '%s' must have one or two dummy arguments"_err_en_US;
  }
````
- **L2305 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::LogicalOperator &x) {`.
  **L2305 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::LogicalOperator &x) {`。
- **L2306 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2306 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2307 EN**: Continues the surrounding expression or declaration: `min = 1; // .NOT. is unary`.
  **L2307 CN**: 继续构造周围的表达式或声明：`min = 1; // .NOT. is unary`。
- **L2308 EN**: Executes a standalone statement or declaration: `max = 1;`.
  **L2308 CN**: 执行一条独立语句或声明：`max = 1;`。
- **L2309 EN**: Closes the current lexical scope or compound statement.
  **L2309 CN**: 结束当前词法作用域或复合语句块。
- **L2310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2310 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L2311 EN**: Starts a function, method, lambda, or structured scope: `[](const common::RelationalOperator &) {`.
  **L2311 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[](const common::RelationalOperator &) {`。
- **L2312 EN**: Comment explains nearby logic, intent, or metadata: `all are binary`.
  **L2312 CN**: 注释说明附近代码的逻辑、意图或元数据：`all are binary`。
- **L2313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2313 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L2314 EN**: Starts a function, method, lambda, or structured scope: `[](const GenericKind::OtherKind &x) {`.
  **L2314 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[](const GenericKind::OtherKind &x) {`。
- **L2315 EN**: Executes a call or declaration centered on `CHECK`.
  **L2315 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2316 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L2317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[](const auto &) { DIE("expected intrinsic operator"); },`.
  **L2317 CN**: 继续一个多行参数列表、初始化器或聚合项：`[](const auto &) { DIE("expected intrinsic operator"); },`。
- **L2318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2318 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L2319 EN**: Executes a standalone statement or declaration: `kind.u);`.
  **L2319 CN**: 执行一条独立语句或声明：`kind.u);`。
- **L2320 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2320 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2321 EN**: Returns from the current function with `std::nullopt`.
  **L2321 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2322 EN**: Transitions from the previous branch into an `else if` condition.
  **L2322 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2323 EN**: Returns from the current function with `"%s function '%s' must have one dummy argument"_err_en_US`.
  **L2323 CN**: 以 `"%s function '%s' must have one dummy argument"_err_en_US` 从当前函数返回。
- **L2324 EN**: Transitions from the previous branch into an `else if` condition.
  **L2324 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2325 EN**: Returns from the current function with `"%s function '%s' must have two dummy arguments"_err_en_US`.
  **L2325 CN**: 以 `"%s function '%s' must have two dummy arguments"_err_en_US` 从当前函数返回。
- **L2326 EN**: Transitions from the previous branch into the alternative path.
  **L2326 CN**: 从前一个分支过渡到备选路径。
- **L2327 EN**: Returns from the current function with `"%s function '%s' must have one or two dummy arguments"_err_en_US`.
  **L2327 CN**: 以 `"%s function '%s' must have one or two dummy arguments"_err_en_US` 从当前函数返回。
- **L2328 EN**: Closes the current lexical scope or compound statement.
  **L2328 CN**: 结束当前词法作用域或复合语句块。

### Lines 2329-2352

````cpp
}

bool CheckHelper::CheckDefinedOperatorArg(const SourceName &opName,
    const Symbol &symbol, const Procedure &proc, std::size_t pos) {
  if (pos >= proc.dummyArguments.size()) {
    return true;
  }
  auto &arg{proc.dummyArguments.at(pos)};
  std::optional<parser::MessageFixedText> msg;
  if (arg.IsOptional()) {
    msg =
        "In %s function '%s', dummy argument '%s' may not be OPTIONAL"_err_en_US;
  } else if (const auto *dataObject{std::get_if<DummyDataObject>(&arg.u)};
             dataObject == nullptr) {
    msg =
        "In %s function '%s', dummy argument '%s' must be a data object"_err_en_US;
  } else if (dataObject->intent == common::Intent::Out) {
    msg =
        "In %s function '%s', dummy argument '%s' may not be INTENT(OUT)"_err_en_US;
  } else if (dataObject->intent != common::Intent::In &&
      !dataObject->attrs.test(DummyDataObject::Attr::Value)) {
    evaluate::AttachDeclaration(
        Warn(common::UsageWarning::DefinedOperatorArgs,
            "In %s function '%s', dummy argument '%s' should have INTENT(IN) or VALUE attribute"_warn_en_US,
````
- **L2329 EN**: Closes the current lexical scope or compound statement.
  **L2329 CN**: 结束当前词法作用域或复合语句块。
- **L2330 EN**: Blank line separating nearby declarations or logic blocks.
  **L2330 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool CheckHelper::CheckDefinedOperatorArg(const SourceName &opName,`.
  **L2331 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool CheckHelper::CheckDefinedOperatorArg(const SourceName &opName,`。
- **L2332 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, const Procedure &proc, std::size_t pos) {`.
  **L2332 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, const Procedure &proc, std::size_t pos) {`。
- **L2333 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2333 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2334 EN**: Returns from the current function with `true`.
  **L2334 CN**: 以 `true` 从当前函数返回。
- **L2335 EN**: Closes the current lexical scope or compound statement.
  **L2335 CN**: 结束当前词法作用域或复合语句块。
- **L2336 EN**: Executes a call or declaration centered on `&arg{proc.dummyArguments.at`.
  **L2336 CN**: 执行以 `&arg{proc.dummyArguments.at` 为核心的调用或声明。
- **L2337 EN**: Executes a standalone statement or declaration: `std::optional<parser::MessageFixedText> msg;`.
  **L2337 CN**: 执行一条独立语句或声明：`std::optional<parser::MessageFixedText> msg;`。
- **L2338 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2338 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2339 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2339 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2340 EN**: Executes a standalone statement or declaration: `"In %s function '%s', dummy argument '%s' may not be OPTIONAL"_err_en_US;`.
  **L2340 CN**: 执行一条独立语句或声明：`"In %s function '%s', dummy argument '%s' may not be OPTIONAL"_err_en_US;`。
- **L2341 EN**: Transitions from the previous branch into an `else if` condition.
  **L2341 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2342 EN**: Continues the surrounding expression or declaration: `dataObject == nullptr) {`.
  **L2342 CN**: 继续构造周围的表达式或声明：`dataObject == nullptr) {`。
- **L2343 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2343 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2344 EN**: Executes a standalone statement or declaration: `"In %s function '%s', dummy argument '%s' must be a data object"_err_en_US;`.
  **L2344 CN**: 执行一条独立语句或声明：`"In %s function '%s', dummy argument '%s' must be a data object"_err_en_US;`。
- **L2345 EN**: Transitions from the previous branch into an `else if` condition.
  **L2345 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2346 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2346 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2347 EN**: Executes a call or declaration centered on `INTENT`.
  **L2347 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L2348 EN**: Transitions from the previous branch into an `else if` condition.
  **L2348 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2349 EN**: Starts a function, method, lambda, or structured scope: `!dataObject->attrs.test(DummyDataObject::Attr::Value)) {`.
  **L2349 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!dataObject->attrs.test(DummyDataObject::Attr::Value)) {`。
- **L2350 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L2350 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L2351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::DefinedOperatorArgs,`.
  **L2351 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::DefinedOperatorArgs,`。
- **L2352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"In %s function '%s', dummy argument '%s' should have INTENT(IN) or VALUE attribute"_warn_en_US,`.
  **L2352 CN**: 继续一个多行参数列表、初始化器或聚合项：`"In %s function '%s', dummy argument '%s' should have INTENT(IN) or VALUE attribute"_warn_en_US,`。

### Lines 2353-2376

````cpp
            parser::ToUpperCaseLetters(opName.ToString()), symbol.name(),
            arg.name),
        symbol);
    return true;
  }
  if (msg) {
    SayWithDeclaration(symbol, std::move(*msg),
        parser::ToUpperCaseLetters(opName.ToString()), symbol.name(), arg.name);
    return false;
  }
  return true;
}

// Check if this procedure can be used for defined assignment (see 15.4.3.4.3).
bool CheckHelper::CheckDefinedAssignment(
    const Symbol &specific, const Procedure &proc) {
  if (context_.HasError(specific)) {
    return false;
  }
  std::optional<parser::MessageFixedText> msg;
  if (specific.attrs().test(Attr::NOPASS)) { // C774
    msg = "Defined assignment procedure '%s' may not have"
          " NOPASS attribute"_err_en_US;
  } else if (!proc.IsSubroutine()) {
````
- **L2353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(opName.ToString()), symbol.name(),`.
  **L2353 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(opName.ToString()), symbol.name(),`。
- **L2354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arg.name),`.
  **L2354 CN**: 继续一个多行参数列表、初始化器或聚合项：`arg.name),`。
- **L2355 EN**: Executes a standalone statement or declaration: `symbol);`.
  **L2355 CN**: 执行一条独立语句或声明：`symbol);`。
- **L2356 EN**: Returns from the current function with `true`.
  **L2356 CN**: 以 `true` 从当前函数返回。
- **L2357 EN**: Closes the current lexical scope or compound statement.
  **L2357 CN**: 结束当前词法作用域或复合语句块。
- **L2358 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2358 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(symbol, std::move(*msg),`.
  **L2359 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(symbol, std::move(*msg),`。
- **L2360 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L2360 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L2361 EN**: Returns from the current function with `false`.
  **L2361 CN**: 以 `false` 从当前函数返回。
- **L2362 EN**: Closes the current lexical scope or compound statement.
  **L2362 CN**: 结束当前词法作用域或复合语句块。
- **L2363 EN**: Returns from the current function with `true`.
  **L2363 CN**: 以 `true` 从当前函数返回。
- **L2364 EN**: Closes the current lexical scope or compound statement.
  **L2364 CN**: 结束当前词法作用域或复合语句块。
- **L2365 EN**: Blank line separating nearby declarations or logic blocks.
  **L2365 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2366 EN**: Comment explains nearby logic, intent, or metadata: `Check if this procedure can be used for defined assignment (see 15.4.3.4.3).`.
  **L2366 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check if this procedure can be used for defined assignment (see 15.4.3.4.3).`。
- **L2367 EN**: Continues logic associated with callable symbol `CheckDefinedAssignment`.
  **L2367 CN**: 继续与可调用符号 `CheckDefinedAssignment` 相关的逻辑。
- **L2368 EN**: Continues the surrounding expression or declaration: `const Symbol &specific, const Procedure &proc) {`.
  **L2368 CN**: 继续构造周围的表达式或声明：`const Symbol &specific, const Procedure &proc) {`。
- **L2369 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2369 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2370 EN**: Returns from the current function with `false`.
  **L2370 CN**: 以 `false` 从当前函数返回。
- **L2371 EN**: Closes the current lexical scope or compound statement.
  **L2371 CN**: 结束当前词法作用域或复合语句块。
- **L2372 EN**: Executes a standalone statement or declaration: `std::optional<parser::MessageFixedText> msg;`.
  **L2372 CN**: 执行一条独立语句或声明：`std::optional<parser::MessageFixedText> msg;`。
- **L2373 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2373 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2374 EN**: Continues the surrounding expression or declaration: `msg = "Defined assignment procedure '%s' may not have"`.
  **L2374 CN**: 继续构造周围的表达式或声明：`msg = "Defined assignment procedure '%s' may not have"`。
- **L2375 EN**: Executes a standalone statement or declaration: `" NOPASS attribute"_err_en_US;`.
  **L2375 CN**: 执行一条独立语句或声明：`" NOPASS attribute"_err_en_US;`。
- **L2376 EN**: Transitions from the previous branch into an `else if` condition.
  **L2376 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 2377-2400

````cpp
    msg = "Defined assignment procedure '%s' must be a subroutine"_err_en_US;
  } else if (proc.dummyArguments.size() != 2) {
    msg = "Defined assignment subroutine '%s' must have"
          " two dummy arguments"_err_en_US;
  } else {
    // Check both arguments even if the first has an error.
    bool ok0{CheckDefinedAssignmentArg(specific, proc.dummyArguments[0], 0)};
    bool ok1{CheckDefinedAssignmentArg(specific, proc.dummyArguments[1], 1)};
    if (!(ok0 && ok1)) {
      return false; // error was reported
    } else if (ConflictsWithIntrinsicAssignment(proc)) {
      msg =
          "Defined assignment subroutine '%s' conflicts with intrinsic assignment"_err_en_US;
    } else {
      return true; // OK
    }
  }
  SayWithDeclaration(specific, std::move(msg.value()), specific.name());
  context_.SetError(specific);
  return false;
}

bool CheckHelper::CheckDefinedAssignmentArg(
    const Symbol &symbol, const DummyArgument &arg, int pos) {
````
- **L2377 EN**: Executes a standalone statement or declaration: `msg = "Defined assignment procedure '%s' must be a subroutine"_err_en_US;`.
  **L2377 CN**: 执行一条独立语句或声明：`msg = "Defined assignment procedure '%s' must be a subroutine"_err_en_US;`。
- **L2378 EN**: Transitions from the previous branch into an `else if` condition.
  **L2378 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2379 EN**: Continues the surrounding expression or declaration: `msg = "Defined assignment subroutine '%s' must have"`.
  **L2379 CN**: 继续构造周围的表达式或声明：`msg = "Defined assignment subroutine '%s' must have"`。
- **L2380 EN**: Executes a standalone statement or declaration: `" two dummy arguments"_err_en_US;`.
  **L2380 CN**: 执行一条独立语句或声明：`" two dummy arguments"_err_en_US;`。
- **L2381 EN**: Transitions from the previous branch into the alternative path.
  **L2381 CN**: 从前一个分支过渡到备选路径。
- **L2382 EN**: Comment explains nearby logic, intent, or metadata: `Check both arguments even if the first has an error.`.
  **L2382 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check both arguments even if the first has an error.`。
- **L2383 EN**: Executes a call or declaration centered on `ok0{CheckDefinedAssignmentArg`.
  **L2383 CN**: 执行以 `ok0{CheckDefinedAssignmentArg` 为核心的调用或声明。
- **L2384 EN**: Executes a call or declaration centered on `ok1{CheckDefinedAssignmentArg`.
  **L2384 CN**: 执行以 `ok1{CheckDefinedAssignmentArg` 为核心的调用或声明。
- **L2385 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2385 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2386 EN**: Returns from the current function with `false; // error was reported`.
  **L2386 CN**: 以 `false; // error was reported` 从当前函数返回。
- **L2387 EN**: Transitions from the previous branch into an `else if` condition.
  **L2387 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2388 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2388 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2389 EN**: Executes a standalone statement or declaration: `"Defined assignment subroutine '%s' conflicts with intrinsic assignment"_err_en_US;`.
  **L2389 CN**: 执行一条独立语句或声明：`"Defined assignment subroutine '%s' conflicts with intrinsic assignment"_err_en_US;`。
- **L2390 EN**: Transitions from the previous branch into the alternative path.
  **L2390 CN**: 从前一个分支过渡到备选路径。
- **L2391 EN**: Returns from the current function with `true; // OK`.
  **L2391 CN**: 以 `true; // OK` 从当前函数返回。
- **L2392 EN**: Closes the current lexical scope or compound statement.
  **L2392 CN**: 结束当前词法作用域或复合语句块。
- **L2393 EN**: Closes the current lexical scope or compound statement.
  **L2393 CN**: 结束当前词法作用域或复合语句块。
- **L2394 EN**: Executes a call or declaration centered on `SayWithDeclaration`.
  **L2394 CN**: 执行以 `SayWithDeclaration` 为核心的调用或声明。
- **L2395 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L2395 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L2396 EN**: Returns from the current function with `false`.
  **L2396 CN**: 以 `false` 从当前函数返回。
- **L2397 EN**: Closes the current lexical scope or compound statement.
  **L2397 CN**: 结束当前词法作用域或复合语句块。
- **L2398 EN**: Blank line separating nearby declarations or logic blocks.
  **L2398 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2399 EN**: Continues logic associated with callable symbol `CheckDefinedAssignmentArg`.
  **L2399 CN**: 继续与可调用符号 `CheckDefinedAssignmentArg` 相关的逻辑。
- **L2400 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, const DummyArgument &arg, int pos) {`.
  **L2400 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, const DummyArgument &arg, int pos) {`。

### Lines 2401-2424

````cpp
  std::optional<parser::MessageFixedText> msg;
  if (arg.IsOptional()) {
    msg = "In defined assignment subroutine '%s', dummy argument '%s'"
          " may not be OPTIONAL"_err_en_US;
  } else if (const auto *dataObject{std::get_if<DummyDataObject>(&arg.u)}) {
    if (pos == 0) {
      if (dataObject->intent == common::Intent::In) {
        msg = "In defined assignment subroutine '%s', first dummy argument '%s'"
              " may not have INTENT(IN)"_err_en_US;
      } else if (dataObject->intent != common::Intent::Out &&
          dataObject->intent != common::Intent::InOut) {
        msg =
            "In defined assignment subroutine '%s', first dummy argument '%s' should have INTENT(OUT) or INTENT(INOUT)"_warn_en_US;
      }
    } else if (pos == 1) {
      if (dataObject->intent == common::Intent::Out) {
        msg = "In defined assignment subroutine '%s', second dummy"
              " argument '%s' may not have INTENT(OUT)"_err_en_US;
      } else if (dataObject->intent != common::Intent::In &&
          !dataObject->attrs.test(DummyDataObject::Attr::Value)) {
        msg =
            "In defined assignment subroutine '%s', second dummy argument '%s' should have INTENT(IN) or VALUE attribute"_warn_en_US;
      } else if (dataObject->attrs.test(DummyDataObject::Attr::Pointer)) {
        msg =
````
- **L2401 EN**: Executes a standalone statement or declaration: `std::optional<parser::MessageFixedText> msg;`.
  **L2401 CN**: 执行一条独立语句或声明：`std::optional<parser::MessageFixedText> msg;`。
- **L2402 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2402 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2403 EN**: Continues the surrounding expression or declaration: `msg = "In defined assignment subroutine '%s', dummy argument '%s'"`.
  **L2403 CN**: 继续构造周围的表达式或声明：`msg = "In defined assignment subroutine '%s', dummy argument '%s'"`。
- **L2404 EN**: Executes a standalone statement or declaration: `" may not be OPTIONAL"_err_en_US;`.
  **L2404 CN**: 执行一条独立语句或声明：`" may not be OPTIONAL"_err_en_US;`。
- **L2405 EN**: Transitions from the previous branch into an `else if` condition.
  **L2405 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2406 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2406 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2407 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2407 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2408 EN**: Continues the surrounding expression or declaration: `msg = "In defined assignment subroutine '%s', first dummy argument '%s'"`.
  **L2408 CN**: 继续构造周围的表达式或声明：`msg = "In defined assignment subroutine '%s', first dummy argument '%s'"`。
- **L2409 EN**: Executes a call or declaration centered on `INTENT`.
  **L2409 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L2410 EN**: Transitions from the previous branch into an `else if` condition.
  **L2410 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2411 EN**: Continues the surrounding expression or declaration: `dataObject->intent != common::Intent::InOut) {`.
  **L2411 CN**: 继续构造周围的表达式或声明：`dataObject->intent != common::Intent::InOut) {`。
- **L2412 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2412 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2413 EN**: Executes a call or declaration centered on `INTENT`.
  **L2413 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L2414 EN**: Closes the current lexical scope or compound statement.
  **L2414 CN**: 结束当前词法作用域或复合语句块。
- **L2415 EN**: Transitions from the previous branch into an `else if` condition.
  **L2415 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2416 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2416 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2417 EN**: Continues the surrounding expression or declaration: `msg = "In defined assignment subroutine '%s', second dummy"`.
  **L2417 CN**: 继续构造周围的表达式或声明：`msg = "In defined assignment subroutine '%s', second dummy"`。
- **L2418 EN**: Executes a call or declaration centered on `INTENT`.
  **L2418 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L2419 EN**: Transitions from the previous branch into an `else if` condition.
  **L2419 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2420 EN**: Starts a function, method, lambda, or structured scope: `!dataObject->attrs.test(DummyDataObject::Attr::Value)) {`.
  **L2420 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!dataObject->attrs.test(DummyDataObject::Attr::Value)) {`。
- **L2421 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2421 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2422 EN**: Executes a call or declaration centered on `INTENT`.
  **L2422 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L2423 EN**: Transitions from the previous branch into an `else if` condition.
  **L2423 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2424 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2424 CN**: 继续构造周围的表达式或声明：`msg =`。

### Lines 2425-2448

````cpp
            "In defined assignment subroutine '%s', second dummy argument '%s' must not be a pointer"_err_en_US;
      } else if (dataObject->attrs.test(DummyDataObject::Attr::Allocatable)) {
        msg =
            "In defined assignment subroutine '%s', second dummy argument '%s' must not be an allocatable"_err_en_US;
      }
    } else {
      DIE("pos must be 0 or 1");
    }
  } else {
    msg = "In defined assignment subroutine '%s', dummy argument '%s'"
          " must be a data object"_err_en_US;
  }
  if (msg) {
    if (msg->IsFatal()) {
      SayWithDeclaration(symbol, std::move(*msg), symbol.name(), arg.name);
      context_.SetError(symbol);
      return false;
    } else {
      evaluate::AttachDeclaration(
          Warn(common::UsageWarning::DefinedOperatorArgs, std::move(*msg),
              symbol.name(), arg.name),
          symbol);
    }
  }
````
- **L2425 EN**: Executes a standalone statement or declaration: `"In defined assignment subroutine '%s', second dummy argument '%s' must not be a pointer"_err_en_US;`.
  **L2425 CN**: 执行一条独立语句或声明：`"In defined assignment subroutine '%s', second dummy argument '%s' must not be a pointer"_err_en_US;`。
- **L2426 EN**: Transitions from the previous branch into an `else if` condition.
  **L2426 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2427 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2427 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2428 EN**: Executes a standalone statement or declaration: `"In defined assignment subroutine '%s', second dummy argument '%s' must not be an allocatable"_err_en_US;`.
  **L2428 CN**: 执行一条独立语句或声明：`"In defined assignment subroutine '%s', second dummy argument '%s' must not be an allocatable"_err_en_US;`。
- **L2429 EN**: Closes the current lexical scope or compound statement.
  **L2429 CN**: 结束当前词法作用域或复合语句块。
- **L2430 EN**: Transitions from the previous branch into the alternative path.
  **L2430 CN**: 从前一个分支过渡到备选路径。
- **L2431 EN**: Executes a call or declaration centered on `DIE`.
  **L2431 CN**: 执行以 `DIE` 为核心的调用或声明。
- **L2432 EN**: Closes the current lexical scope or compound statement.
  **L2432 CN**: 结束当前词法作用域或复合语句块。
- **L2433 EN**: Transitions from the previous branch into the alternative path.
  **L2433 CN**: 从前一个分支过渡到备选路径。
- **L2434 EN**: Continues the surrounding expression or declaration: `msg = "In defined assignment subroutine '%s', dummy argument '%s'"`.
  **L2434 CN**: 继续构造周围的表达式或声明：`msg = "In defined assignment subroutine '%s', dummy argument '%s'"`。
- **L2435 EN**: Executes a standalone statement or declaration: `" must be a data object"_err_en_US;`.
  **L2435 CN**: 执行一条独立语句或声明：`" must be a data object"_err_en_US;`。
- **L2436 EN**: Closes the current lexical scope or compound statement.
  **L2436 CN**: 结束当前词法作用域或复合语句块。
- **L2437 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2437 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2438 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2438 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2439 EN**: Executes a call or declaration centered on `SayWithDeclaration`.
  **L2439 CN**: 执行以 `SayWithDeclaration` 为核心的调用或声明。
- **L2440 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L2440 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L2441 EN**: Returns from the current function with `false`.
  **L2441 CN**: 以 `false` 从当前函数返回。
- **L2442 EN**: Transitions from the previous branch into the alternative path.
  **L2442 CN**: 从前一个分支过渡到备选路径。
- **L2443 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L2443 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L2444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::DefinedOperatorArgs, std::move(*msg),`.
  **L2444 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::DefinedOperatorArgs, std::move(*msg),`。
- **L2445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol.name(), arg.name),`.
  **L2445 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol.name(), arg.name),`。
- **L2446 EN**: Executes a standalone statement or declaration: `symbol);`.
  **L2446 CN**: 执行一条独立语句或声明：`symbol);`。
- **L2447 EN**: Closes the current lexical scope or compound statement.
  **L2447 CN**: 结束当前词法作用域或复合语句块。
- **L2448 EN**: Closes the current lexical scope or compound statement.
  **L2448 CN**: 结束当前词法作用域或复合语句块。

### Lines 2449-2472

````cpp
  return true;
}

// Report a conflicting attribute error if symbol has both of these attributes
bool CheckHelper::CheckConflicting(const Symbol &symbol, Attr a1, Attr a2) {
  if (symbol.attrs().test(a1) && symbol.attrs().test(a2)) {
    messages_.Say("'%s' may not have both the %s and %s attributes"_err_en_US,
        symbol.name(), AttrToString(a1), AttrToString(a2));
    return true;
  } else {
    return false;
  }
}

void CheckHelper::WarnMissingFinal(const Symbol &symbol) {
  const auto *object{symbol.detailsIf<ObjectEntityDetails>()};
  if (!object || object->IsAssumedRank() ||
      (!IsAutomaticallyDestroyed(symbol) &&
          symbol.owner().kind() != Scope::Kind::DerivedType)) {
    return;
  }
  const DeclTypeSpec *type{object->type()};
  const DerivedTypeSpec *derived{type ? type->AsDerived() : nullptr};
  const Symbol *derivedSym{derived ? &derived->typeSymbol() : nullptr};
````
- **L2449 EN**: Returns from the current function with `true`.
  **L2449 CN**: 以 `true` 从当前函数返回。
- **L2450 EN**: Closes the current lexical scope or compound statement.
  **L2450 CN**: 结束当前词法作用域或复合语句块。
- **L2451 EN**: Blank line separating nearby declarations or logic blocks.
  **L2451 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2452 EN**: Comment explains nearby logic, intent, or metadata: `Report a conflicting attribute error if symbol has both of these attributes`.
  **L2452 CN**: 注释说明附近代码的逻辑、意图或元数据：`Report a conflicting attribute error if symbol has both of these attributes`。
- **L2453 EN**: Starts a function, method, lambda, or structured scope: `bool CheckHelper::CheckConflicting(const Symbol &symbol, Attr a1, Attr a2) {`.
  **L2453 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool CheckHelper::CheckConflicting(const Symbol &symbol, Attr a1, Attr a2) {`。
- **L2454 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2454 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say("'%s' may not have both the %s and %s attributes"_err_en_US,`.
  **L2455 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say("'%s' may not have both the %s and %s attributes"_err_en_US,`。
- **L2456 EN**: Executes a call or declaration centered on `symbol.name`.
  **L2456 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L2457 EN**: Returns from the current function with `true`.
  **L2457 CN**: 以 `true` 从当前函数返回。
- **L2458 EN**: Transitions from the previous branch into the alternative path.
  **L2458 CN**: 从前一个分支过渡到备选路径。
- **L2459 EN**: Returns from the current function with `false`.
  **L2459 CN**: 以 `false` 从当前函数返回。
- **L2460 EN**: Closes the current lexical scope or compound statement.
  **L2460 CN**: 结束当前词法作用域或复合语句块。
- **L2461 EN**: Closes the current lexical scope or compound statement.
  **L2461 CN**: 结束当前词法作用域或复合语句块。
- **L2462 EN**: Blank line separating nearby declarations or logic blocks.
  **L2462 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2463 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::WarnMissingFinal(const Symbol &symbol) {`.
  **L2463 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::WarnMissingFinal(const Symbol &symbol) {`。
- **L2464 EN**: Executes a call or declaration centered on `*object{symbol.detailsIf<ObjectEntityDetails>`.
  **L2464 CN**: 执行以 `*object{symbol.detailsIf<ObjectEntityDetails>` 为核心的调用或声明。
- **L2465 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2465 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2466 EN**: Continues logic associated with callable symbol `IsAutomaticallyDestroyed`.
  **L2466 CN**: 继续与可调用符号 `IsAutomaticallyDestroyed` 相关的逻辑。
- **L2467 EN**: Starts a function, method, lambda, or structured scope: `symbol.owner().kind() != Scope::Kind::DerivedType)) {`.
  **L2467 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.owner().kind() != Scope::Kind::DerivedType)) {`。
- **L2468 EN**: Returns from the current function with `void`.
  **L2468 CN**: 以 `void` 从当前函数返回。
- **L2469 EN**: Closes the current lexical scope or compound statement.
  **L2469 CN**: 结束当前词法作用域或复合语句块。
- **L2470 EN**: Executes a call or declaration centered on `*type{object->type`.
  **L2470 CN**: 执行以 `*type{object->type` 为核心的调用或声明。
- **L2471 EN**: Executes a call or declaration centered on `type->AsDerived`.
  **L2471 CN**: 执行以 `type->AsDerived` 为核心的调用或声明。
- **L2472 EN**: Executes a call or declaration centered on `&derived->typeSymbol`.
  **L2472 CN**: 执行以 `&derived->typeSymbol` 为核心的调用或声明。

### Lines 2473-2496

````cpp
  int rank{object->shape().Rank()};
  const Symbol *initialDerivedSym{derivedSym};
  while (const auto *derivedDetails{
      derivedSym ? derivedSym->detailsIf<DerivedTypeDetails>() : nullptr}) {
    if (!derivedDetails->finals().empty() &&
        !derivedDetails->GetFinalForRank(rank)) {
      if (auto *msg{derivedSym == initialDerivedSym
                  ? Warn(common::UsageWarning::Final, symbol.name(),
                        "'%s' of derived type '%s' does not have a FINAL subroutine for its rank (%d)"_warn_en_US,
                        symbol.name(), derivedSym->name(), rank)
                  : Warn(common::UsageWarning::Final, symbol.name(),
                        "'%s' of derived type '%s' extended from '%s' does not have a FINAL subroutine for its rank (%d)"_warn_en_US,
                        symbol.name(), initialDerivedSym->name(),
                        derivedSym->name(), rank)}) {
        msg->Attach(derivedSym->name(),
            "Declaration of derived type '%s'"_en_US, derivedSym->name());
      }
      return;
    }
    derived = derivedSym->GetParentTypeSpec();
    derivedSym = derived ? &derived->typeSymbol() : nullptr;
  }
}

````
- **L2473 EN**: Executes a call or declaration centered on `rank{object->shape`.
  **L2473 CN**: 执行以 `rank{object->shape` 为核心的调用或声明。
- **L2474 EN**: Executes a standalone statement or declaration: `const Symbol *initialDerivedSym{derivedSym};`.
  **L2474 CN**: 执行一条独立语句或声明：`const Symbol *initialDerivedSym{derivedSym};`。
- **L2475 EN**: Begins a `while` control-flow statement and evaluates its condition.
  **L2475 CN**: 开始 `while` 控制流语句并计算其条件。
- **L2476 EN**: Starts a function, method, lambda, or structured scope: `derivedSym ? derivedSym->detailsIf<DerivedTypeDetails>() : nullptr}) {`.
  **L2476 CN**: 开始一个函数、方法、lambda 或结构化作用域：`derivedSym ? derivedSym->detailsIf<DerivedTypeDetails>() : nullptr}) {`。
- **L2477 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2477 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2478 EN**: Starts a function, method, lambda, or structured scope: `!derivedDetails->GetFinalForRank(rank)) {`.
  **L2478 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!derivedDetails->GetFinalForRank(rank)) {`。
- **L2479 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2479 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `? Warn(common::UsageWarning::Final, symbol.name(),`.
  **L2480 CN**: 继续一个多行参数列表、初始化器或聚合项：`? Warn(common::UsageWarning::Final, symbol.name(),`。
- **L2481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' of derived type '%s' does not have a FINAL subroutine for its rank (%d)"_warn_en_US,`.
  **L2481 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' of derived type '%s' does not have a FINAL subroutine for its rank (%d)"_warn_en_US,`。
- **L2482 EN**: Continues logic associated with callable symbol `name`.
  **L2482 CN**: 继续与可调用符号 `name` 相关的逻辑。
- **L2483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Warn(common::UsageWarning::Final, symbol.name(),`.
  **L2483 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Warn(common::UsageWarning::Final, symbol.name(),`。
- **L2484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' of derived type '%s' extended from '%s' does not have a FINAL subroutine for its rank (%d)"_warn_en_US,`.
  **L2484 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' of derived type '%s' extended from '%s' does not have a FINAL subroutine for its rank (%d)"_warn_en_US,`。
- **L2485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol.name(), initialDerivedSym->name(),`.
  **L2485 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol.name(), initialDerivedSym->name(),`。
- **L2486 EN**: Starts a function, method, lambda, or structured scope: `derivedSym->name(), rank)}) {`.
  **L2486 CN**: 开始一个函数、方法、lambda 或结构化作用域：`derivedSym->name(), rank)}) {`。
- **L2487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg->Attach(derivedSym->name(),`.
  **L2487 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg->Attach(derivedSym->name(),`。
- **L2488 EN**: Executes a call or declaration centered on `derivedSym->name`.
  **L2488 CN**: 执行以 `derivedSym->name` 为核心的调用或声明。
- **L2489 EN**: Closes the current lexical scope or compound statement.
  **L2489 CN**: 结束当前词法作用域或复合语句块。
- **L2490 EN**: Returns from the current function with `void`.
  **L2490 CN**: 以 `void` 从当前函数返回。
- **L2491 EN**: Closes the current lexical scope or compound statement.
  **L2491 CN**: 结束当前词法作用域或复合语句块。
- **L2492 EN**: Executes a call or declaration centered on `derivedSym->GetParentTypeSpec`.
  **L2492 CN**: 执行以 `derivedSym->GetParentTypeSpec` 为核心的调用或声明。
- **L2493 EN**: Executes a call or declaration centered on `&derived->typeSymbol`.
  **L2493 CN**: 执行以 `&derived->typeSymbol` 为核心的调用或声明。
- **L2494 EN**: Closes the current lexical scope or compound statement.
  **L2494 CN**: 结束当前词法作用域或复合语句块。
- **L2495 EN**: Closes the current lexical scope or compound statement.
  **L2495 CN**: 结束当前词法作用域或复合语句块。
- **L2496 EN**: Blank line separating nearby declarations or logic blocks.
  **L2496 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2497-2520

````cpp
const Procedure *CheckHelper::Characterize(const Symbol &symbol) {
  auto it{characterizeCache_.find(symbol)};
  if (it == characterizeCache_.end()) {
    auto pair{characterizeCache_.emplace(SymbolRef{symbol},
        Procedure::Characterize(symbol, context_.foldingContext()))};
    it = pair.first;
  }
  return common::GetPtrFromOptional(it->second);
}

void CheckHelper::CheckVolatile(const Symbol &symbol,
    const DerivedTypeSpec *derived) { // C866 - C868
  if (IsIntentIn(symbol)) {
    messages_.Say(
        "VOLATILE attribute may not apply to an INTENT(IN) argument"_err_en_US);
  }
  if (IsProcedure(symbol)) {
    messages_.Say("VOLATILE attribute may apply only to a variable"_err_en_US);
  }
  if (symbol.has<UseDetails>() || symbol.has<HostAssocDetails>()) {
    const Symbol &ultimate{symbol.GetUltimate()};
    if (evaluate::IsCoarray(ultimate)) {
      messages_.Say(
          "VOLATILE attribute may not apply to a coarray accessed by USE or host association"_err_en_US);
````
- **L2497 EN**: Starts a function, method, lambda, or structured scope: `const Procedure *CheckHelper::Characterize(const Symbol &symbol) {`.
  **L2497 CN**: 开始一个函数、方法、lambda 或结构化作用域：`const Procedure *CheckHelper::Characterize(const Symbol &symbol) {`。
- **L2498 EN**: Executes a call or declaration centered on `it{characterizeCache_.find`.
  **L2498 CN**: 执行以 `it{characterizeCache_.find` 为核心的调用或声明。
- **L2499 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2499 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto pair{characterizeCache_.emplace(SymbolRef{symbol},`.
  **L2500 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto pair{characterizeCache_.emplace(SymbolRef{symbol},`。
- **L2501 EN**: Executes a call or declaration centered on `Procedure::Characterize`.
  **L2501 CN**: 执行以 `Procedure::Characterize` 为核心的调用或声明。
- **L2502 EN**: Executes a standalone statement or declaration: `it = pair.first;`.
  **L2502 CN**: 执行一条独立语句或声明：`it = pair.first;`。
- **L2503 EN**: Closes the current lexical scope or compound statement.
  **L2503 CN**: 结束当前词法作用域或复合语句块。
- **L2504 EN**: Returns from the current function with `common::GetPtrFromOptional(it->second)`.
  **L2504 CN**: 以 `common::GetPtrFromOptional(it->second)` 从当前函数返回。
- **L2505 EN**: Closes the current lexical scope or compound statement.
  **L2505 CN**: 结束当前词法作用域或复合语句块。
- **L2506 EN**: Blank line separating nearby declarations or logic blocks.
  **L2506 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckHelper::CheckVolatile(const Symbol &symbol,`.
  **L2507 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckHelper::CheckVolatile(const Symbol &symbol,`。
- **L2508 EN**: Continues the surrounding expression or declaration: `const DerivedTypeSpec *derived) { // C866 - C868`.
  **L2508 CN**: 继续构造周围的表达式或声明：`const DerivedTypeSpec *derived) { // C866 - C868`。
- **L2509 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2509 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2510 EN**: Continues logic associated with callable symbol `Say`.
  **L2510 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2511 EN**: Executes a call or declaration centered on `INTENT`.
  **L2511 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L2512 EN**: Closes the current lexical scope or compound statement.
  **L2512 CN**: 结束当前词法作用域或复合语句块。
- **L2513 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2513 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2514 EN**: Executes a call or declaration centered on `messages_.Say`.
  **L2514 CN**: 执行以 `messages_.Say` 为核心的调用或声明。
- **L2515 EN**: Closes the current lexical scope or compound statement.
  **L2515 CN**: 结束当前词法作用域或复合语句块。
- **L2516 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2516 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2517 EN**: Executes a call or declaration centered on `&ultimate{symbol.GetUltimate`.
  **L2517 CN**: 执行以 `&ultimate{symbol.GetUltimate` 为核心的调用或声明。
- **L2518 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2518 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2519 EN**: Continues logic associated with callable symbol `Say`.
  **L2519 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2520 EN**: Executes a standalone statement or declaration: `"VOLATILE attribute may not apply to a coarray accessed by USE or host association"_err_en_US);`.
  **L2520 CN**: 执行一条独立语句或声明：`"VOLATILE attribute may not apply to a coarray accessed by USE or host association"_err_en_US);`。

### Lines 2521-2544

````cpp
    }
    if (derived) {
      if (FindCoarrayUltimateComponent(*derived)) {
        messages_.Say(
            "VOLATILE attribute may not apply to a type with a coarray ultimate component accessed by USE or host association"_err_en_US);
      }
    }
  }
}

void CheckHelper::CheckContiguous(const Symbol &symbol) {
  if (evaluate::IsVariable(symbol) &&
      ((IsPointer(symbol) && symbol.Rank() > 0) || IsAssumedShape(symbol) ||
          IsAssumedRank(symbol))) {
  } else {
    parser::MessageFixedText msg{symbol.owner().IsDerivedType()
            ? "CONTIGUOUS component '%s' should be an array with the POINTER attribute"_port_en_US
            : "CONTIGUOUS entity '%s' should be an array pointer, assumed-shape, or assumed-rank"_port_en_US};
    if (!context_.IsEnabled(common::LanguageFeature::RedundantContiguous)) {
      msg.set_severity(parser::Severity::Error);
      messages_.Say(std::move(msg), symbol.name());
    } else {
      Warn(common::LanguageFeature::RedundantContiguous, std::move(msg),
          symbol.name());
````
- **L2521 EN**: Closes the current lexical scope or compound statement.
  **L2521 CN**: 结束当前词法作用域或复合语句块。
- **L2522 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2522 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2523 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2523 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2524 EN**: Continues logic associated with callable symbol `Say`.
  **L2524 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2525 EN**: Executes a standalone statement or declaration: `"VOLATILE attribute may not apply to a type with a coarray ultimate component accessed by USE or host association"_err_en_US);`.
  **L2525 CN**: 执行一条独立语句或声明：`"VOLATILE attribute may not apply to a type with a coarray ultimate component accessed by USE or host association"_err_en_US);`。
- **L2526 EN**: Closes the current lexical scope or compound statement.
  **L2526 CN**: 结束当前词法作用域或复合语句块。
- **L2527 EN**: Closes the current lexical scope or compound statement.
  **L2527 CN**: 结束当前词法作用域或复合语句块。
- **L2528 EN**: Closes the current lexical scope or compound statement.
  **L2528 CN**: 结束当前词法作用域或复合语句块。
- **L2529 EN**: Closes the current lexical scope or compound statement.
  **L2529 CN**: 结束当前词法作用域或复合语句块。
- **L2530 EN**: Blank line separating nearby declarations or logic blocks.
  **L2530 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2531 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckContiguous(const Symbol &symbol) {`.
  **L2531 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckContiguous(const Symbol &symbol) {`。
- **L2532 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2532 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2533 EN**: Continues logic associated with callable symbol `IsPointer`.
  **L2533 CN**: 继续与可调用符号 `IsPointer` 相关的逻辑。
- **L2534 EN**: Starts a function, method, lambda, or structured scope: `IsAssumedRank(symbol))) {`.
  **L2534 CN**: 开始一个函数、方法、lambda 或结构化作用域：`IsAssumedRank(symbol))) {`。
- **L2535 EN**: Transitions from the previous branch into the alternative path.
  **L2535 CN**: 从前一个分支过渡到备选路径。
- **L2536 EN**: Continues logic associated with callable symbol `owner`.
  **L2536 CN**: 继续与可调用符号 `owner` 相关的逻辑。
- **L2537 EN**: Continues the surrounding expression or declaration: `? "CONTIGUOUS component '%s' should be an array with the POINTER attribute"_port_en_US`.
  **L2537 CN**: 继续构造周围的表达式或声明：`? "CONTIGUOUS component '%s' should be an array with the POINTER attribute"_port_en_US`。
- **L2538 EN**: Executes a standalone statement or declaration: `: "CONTIGUOUS entity '%s' should be an array pointer, assumed-shape, or assumed-rank"_port_en_US};`.
  **L2538 CN**: 执行一条独立语句或声明：`: "CONTIGUOUS entity '%s' should be an array pointer, assumed-shape, or assumed-rank"_port_en_US};`。
- **L2539 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2539 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2540 EN**: Executes a call or declaration centered on `msg.set_severity`.
  **L2540 CN**: 执行以 `msg.set_severity` 为核心的调用或声明。
- **L2541 EN**: Executes a call or declaration centered on `messages_.Say`.
  **L2541 CN**: 执行以 `messages_.Say` 为核心的调用或声明。
- **L2542 EN**: Transitions from the previous branch into the alternative path.
  **L2542 CN**: 从前一个分支过渡到备选路径。
- **L2543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::LanguageFeature::RedundantContiguous, std::move(msg),`.
  **L2543 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::LanguageFeature::RedundantContiguous, std::move(msg),`。
- **L2544 EN**: Executes a call or declaration centered on `symbol.name`.
  **L2544 CN**: 执行以 `symbol.name` 为核心的调用或声明。

### Lines 2545-2568

````cpp
    }
  }
}

void CheckHelper::CheckPointer(const Symbol &symbol) { // C852
  CheckConflicting(symbol, Attr::POINTER, Attr::TARGET);
  CheckConflicting(symbol, Attr::POINTER, Attr::ALLOCATABLE); // C751
  CheckConflicting(symbol, Attr::POINTER, Attr::INTRINSIC);
  // Prohibit constant pointers.  The standard does not explicitly prohibit
  // them, but the PARAMETER attribute requires a entity-decl to have an
  // initialization that is a constant-expr, and the only form of
  // initialization that allows a constant-expr is the one that's not a "=>"
  // pointer initialization.  See C811, C807, and section 8.5.13.
  CheckConflicting(symbol, Attr::POINTER, Attr::PARAMETER);
  if (symbol.Corank() > 0) {
    messages_.Say(
        "'%s' may not have the POINTER attribute because it is a coarray"_err_en_US,
        symbol.name());
  }
}

// C760 constraints on the passed-object dummy argument
// C757 constraints on procedure pointer components
void CheckHelper::CheckPassArg(
````
- **L2545 EN**: Closes the current lexical scope or compound statement.
  **L2545 CN**: 结束当前词法作用域或复合语句块。
- **L2546 EN**: Closes the current lexical scope or compound statement.
  **L2546 CN**: 结束当前词法作用域或复合语句块。
- **L2547 EN**: Closes the current lexical scope or compound statement.
  **L2547 CN**: 结束当前词法作用域或复合语句块。
- **L2548 EN**: Blank line separating nearby declarations or logic blocks.
  **L2548 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2549 EN**: Continues logic associated with callable symbol `CheckPointer`.
  **L2549 CN**: 继续与可调用符号 `CheckPointer` 相关的逻辑。
- **L2550 EN**: Executes a call or declaration centered on `CheckConflicting`.
  **L2550 CN**: 执行以 `CheckConflicting` 为核心的调用或声明。
- **L2551 EN**: Continues logic associated with callable symbol `CheckConflicting`.
  **L2551 CN**: 继续与可调用符号 `CheckConflicting` 相关的逻辑。
- **L2552 EN**: Executes a call or declaration centered on `CheckConflicting`.
  **L2552 CN**: 执行以 `CheckConflicting` 为核心的调用或声明。
- **L2553 EN**: Comment explains nearby logic, intent, or metadata: `Prohibit constant pointers.  The standard does not explicitly prohibit`.
  **L2553 CN**: 注释说明附近代码的逻辑、意图或元数据：`Prohibit constant pointers.  The standard does not explicitly prohibit`。
- **L2554 EN**: Comment explains nearby logic, intent, or metadata: `them, but the PARAMETER attribute requires a entity-decl to have an`.
  **L2554 CN**: 注释说明附近代码的逻辑、意图或元数据：`them, but the PARAMETER attribute requires a entity-decl to have an`。
- **L2555 EN**: Comment explains nearby logic, intent, or metadata: `initialization that is a constant-expr, and the only form of`.
  **L2555 CN**: 注释说明附近代码的逻辑、意图或元数据：`initialization that is a constant-expr, and the only form of`。
- **L2556 EN**: Comment explains nearby logic, intent, or metadata: `initialization that allows a constant-expr is the one that's not a "=>"`.
  **L2556 CN**: 注释说明附近代码的逻辑、意图或元数据：`initialization that allows a constant-expr is the one that's not a "=>"`。
- **L2557 EN**: Comment explains nearby logic, intent, or metadata: `pointer initialization.  See C811, C807, and section 8.5.13.`.
  **L2557 CN**: 注释说明附近代码的逻辑、意图或元数据：`pointer initialization.  See C811, C807, and section 8.5.13.`。
- **L2558 EN**: Executes a call or declaration centered on `CheckConflicting`.
  **L2558 CN**: 执行以 `CheckConflicting` 为核心的调用或声明。
- **L2559 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2559 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2560 EN**: Continues logic associated with callable symbol `Say`.
  **L2560 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' may not have the POINTER attribute because it is a coarray"_err_en_US,`.
  **L2561 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' may not have the POINTER attribute because it is a coarray"_err_en_US,`。
- **L2562 EN**: Executes a call or declaration centered on `symbol.name`.
  **L2562 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L2563 EN**: Closes the current lexical scope or compound statement.
  **L2563 CN**: 结束当前词法作用域或复合语句块。
- **L2564 EN**: Closes the current lexical scope or compound statement.
  **L2564 CN**: 结束当前词法作用域或复合语句块。
- **L2565 EN**: Blank line separating nearby declarations or logic blocks.
  **L2565 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2566 EN**: Comment explains nearby logic, intent, or metadata: `C760 constraints on the passed-object dummy argument`.
  **L2566 CN**: 注释说明附近代码的逻辑、意图或元数据：`C760 constraints on the passed-object dummy argument`。
- **L2567 EN**: Comment explains nearby logic, intent, or metadata: `C757 constraints on procedure pointer components`.
  **L2567 CN**: 注释说明附近代码的逻辑、意图或元数据：`C757 constraints on procedure pointer components`。
- **L2568 EN**: Continues logic associated with callable symbol `CheckPassArg`.
  **L2568 CN**: 继续与可调用符号 `CheckPassArg` 相关的逻辑。

### Lines 2569-2592

````cpp
    const Symbol &proc, const Symbol *interface0, const WithPassArg &details) {
  if (proc.attrs().test(Attr::NOPASS)) {
    return;
  }
  const auto &name{proc.name()};
  const Symbol *interface {
    interface0 ? FindInterface(*interface0) : nullptr
  };
  if (!interface) {
    messages_.Say(name,
        "Procedure component '%s' must have NOPASS attribute or explicit interface"_err_en_US,
        name);
    return;
  }
  const auto *subprogram{interface->detailsIf<SubprogramDetails>()};
  if (!subprogram) {
    messages_.Say(name,
        "Procedure component '%s' has invalid interface '%s'"_err_en_US, name,
        interface->name());
    return;
  }
  std::optional<SourceName> passName{details.passName()};
  const auto &dummyArgs{subprogram->dummyArgs()};
  if (!passName) {
````
- **L2569 EN**: Continues the surrounding expression or declaration: `const Symbol &proc, const Symbol *interface0, const WithPassArg &details) {`.
  **L2569 CN**: 继续构造周围的表达式或声明：`const Symbol &proc, const Symbol *interface0, const WithPassArg &details) {`。
- **L2570 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2570 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2571 EN**: Returns from the current function with `void`.
  **L2571 CN**: 以 `void` 从当前函数返回。
- **L2572 EN**: Closes the current lexical scope or compound statement.
  **L2572 CN**: 结束当前词法作用域或复合语句块。
- **L2573 EN**: Executes a call or declaration centered on `&name{proc.name`.
  **L2573 CN**: 执行以 `&name{proc.name` 为核心的调用或声明。
- **L2574 EN**: Continues the surrounding expression or declaration: `const Symbol *interface {`.
  **L2574 CN**: 继续构造周围的表达式或声明：`const Symbol *interface {`。
- **L2575 EN**: Continues logic associated with callable symbol `FindInterface`.
  **L2575 CN**: 继续与可调用符号 `FindInterface` 相关的逻辑。
- **L2576 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2576 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2577 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2577 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(name,`.
  **L2578 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(name,`。
- **L2579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Procedure component '%s' must have NOPASS attribute or explicit interface"_err_en_US,`.
  **L2579 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Procedure component '%s' must have NOPASS attribute or explicit interface"_err_en_US,`。
- **L2580 EN**: Executes a standalone statement or declaration: `name);`.
  **L2580 CN**: 执行一条独立语句或声明：`name);`。
- **L2581 EN**: Returns from the current function with `void`.
  **L2581 CN**: 以 `void` 从当前函数返回。
- **L2582 EN**: Closes the current lexical scope or compound statement.
  **L2582 CN**: 结束当前词法作用域或复合语句块。
- **L2583 EN**: Executes a call or declaration centered on `*subprogram{interface->detailsIf<SubprogramDetails>`.
  **L2583 CN**: 执行以 `*subprogram{interface->detailsIf<SubprogramDetails>` 为核心的调用或声明。
- **L2584 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2584 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(name,`.
  **L2585 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(name,`。
- **L2586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Procedure component '%s' has invalid interface '%s'"_err_en_US, name,`.
  **L2586 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Procedure component '%s' has invalid interface '%s'"_err_en_US, name,`。
- **L2587 EN**: Executes a call or declaration centered on `interface->name`.
  **L2587 CN**: 执行以 `interface->name` 为核心的调用或声明。
- **L2588 EN**: Returns from the current function with `void`.
  **L2588 CN**: 以 `void` 从当前函数返回。
- **L2589 EN**: Closes the current lexical scope or compound statement.
  **L2589 CN**: 结束当前词法作用域或复合语句块。
- **L2590 EN**: Executes a call or declaration centered on `passName{details.passName`.
  **L2590 CN**: 执行以 `passName{details.passName` 为核心的调用或声明。
- **L2591 EN**: Executes a call or declaration centered on `&dummyArgs{subprogram->dummyArgs`.
  **L2591 CN**: 执行以 `&dummyArgs{subprogram->dummyArgs` 为核心的调用或声明。
- **L2592 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2592 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2593-2616

````cpp
    if (dummyArgs.empty()) {
      messages_.Say(name,
          proc.has<ProcEntityDetails>()
              ? "Procedure component '%s' with no dummy arguments"
                " must have NOPASS attribute"_err_en_US
              : "Procedure binding '%s' with no dummy arguments"
                " must have NOPASS attribute"_err_en_US,
          name);
      context_.SetError(*interface);
      return;
    }
    Symbol *argSym{dummyArgs[0]};
    if (!argSym) {
      messages_.Say(interface->name(),
          "Cannot use an alternate return as the passed-object dummy "
          "argument"_err_en_US);
      return;
    }
    passName = dummyArgs[0]->name();
  }
  std::optional<int> passArgIndex{};
  for (std::size_t i{0}; i < dummyArgs.size(); ++i) {
    if (dummyArgs[i] && dummyArgs[i]->name() == *passName) {
      passArgIndex = i;
````
- **L2593 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2593 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(name,`.
  **L2594 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(name,`。
- **L2595 EN**: Continues logic associated with callable symbol `has<ProcEntityDetails>`.
  **L2595 CN**: 继续与可调用符号 `has<ProcEntityDetails>` 相关的逻辑。
- **L2596 EN**: Continues the surrounding expression or declaration: `? "Procedure component '%s' with no dummy arguments"`.
  **L2596 CN**: 继续构造周围的表达式或声明：`? "Procedure component '%s' with no dummy arguments"`。
- **L2597 EN**: Continues the surrounding expression or declaration: `" must have NOPASS attribute"_err_en_US`.
  **L2597 CN**: 继续构造周围的表达式或声明：`" must have NOPASS attribute"_err_en_US`。
- **L2598 EN**: Continues the surrounding expression or declaration: `: "Procedure binding '%s' with no dummy arguments"`.
  **L2598 CN**: 继续构造周围的表达式或声明：`: "Procedure binding '%s' with no dummy arguments"`。
- **L2599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" must have NOPASS attribute"_err_en_US,`.
  **L2599 CN**: 继续一个多行参数列表、初始化器或聚合项：`" must have NOPASS attribute"_err_en_US,`。
- **L2600 EN**: Executes a standalone statement or declaration: `name);`.
  **L2600 CN**: 执行一条独立语句或声明：`name);`。
- **L2601 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L2601 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L2602 EN**: Returns from the current function with `void`.
  **L2602 CN**: 以 `void` 从当前函数返回。
- **L2603 EN**: Closes the current lexical scope or compound statement.
  **L2603 CN**: 结束当前词法作用域或复合语句块。
- **L2604 EN**: Executes a standalone statement or declaration: `Symbol *argSym{dummyArgs[0]};`.
  **L2604 CN**: 执行一条独立语句或声明：`Symbol *argSym{dummyArgs[0]};`。
- **L2605 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2605 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(interface->name(),`.
  **L2606 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(interface->name(),`。
- **L2607 EN**: Continues the surrounding expression or declaration: `"Cannot use an alternate return as the passed-object dummy "`.
  **L2607 CN**: 继续构造周围的表达式或声明：`"Cannot use an alternate return as the passed-object dummy "`。
- **L2608 EN**: Executes a standalone statement or declaration: `"argument"_err_en_US);`.
  **L2608 CN**: 执行一条独立语句或声明：`"argument"_err_en_US);`。
- **L2609 EN**: Returns from the current function with `void`.
  **L2609 CN**: 以 `void` 从当前函数返回。
- **L2610 EN**: Closes the current lexical scope or compound statement.
  **L2610 CN**: 结束当前词法作用域或复合语句块。
- **L2611 EN**: Executes a call or declaration centered on `dummyArgs[0]->name`.
  **L2611 CN**: 执行以 `dummyArgs[0]->name` 为核心的调用或声明。
- **L2612 EN**: Closes the current lexical scope or compound statement.
  **L2612 CN**: 结束当前词法作用域或复合语句块。
- **L2613 EN**: Executes a standalone statement or declaration: `std::optional<int> passArgIndex{};`.
  **L2613 CN**: 执行一条独立语句或声明：`std::optional<int> passArgIndex{};`。
- **L2614 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2614 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2615 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2615 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2616 EN**: Executes a standalone statement or declaration: `passArgIndex = i;`.
  **L2616 CN**: 执行一条独立语句或声明：`passArgIndex = i;`。

### Lines 2617-2640

````cpp
      break;
    }
  }
  if (!passArgIndex) { // C758
    messages_.Say(*passName,
        "'%s' is not a dummy argument of procedure interface '%s'"_err_en_US,
        *passName, interface->name());
    return;
  }
  const Symbol &passArg{*dummyArgs[*passArgIndex]};
  std::optional<parser::MessageFixedText> msg;
  if (!passArg.has<ObjectEntityDetails>()) {
    msg = "Passed-object dummy argument '%s' of procedure '%s'"
          " must be a data object"_err_en_US;
  } else if (passArg.attrs().test(Attr::ALLOCATABLE)) {
    msg = "Passed-object dummy argument '%s' of procedure '%s'"
          " may not have the ALLOCATABLE attribute"_err_en_US;
  } else if (passArg.attrs().test(Attr::VALUE)) {
    msg = "Passed-object dummy argument '%s' of procedure '%s'"
          " may not have the VALUE attribute"_err_en_US;
  } else if (passArg.Rank() > 0) {
    msg = "Passed-object dummy argument '%s' of procedure '%s'"
          " must be scalar"_err_en_US;
  } else if (passArg.attrs().test(Attr::POINTER)) {
````
- **L2617 EN**: Exits the nearest loop or switch statement.
  **L2617 CN**: 退出最近的循环或 switch 语句。
- **L2618 EN**: Closes the current lexical scope or compound statement.
  **L2618 CN**: 结束当前词法作用域或复合语句块。
- **L2619 EN**: Closes the current lexical scope or compound statement.
  **L2619 CN**: 结束当前词法作用域或复合语句块。
- **L2620 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2620 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(*passName,`.
  **L2621 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(*passName,`。
- **L2622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' is not a dummy argument of procedure interface '%s'"_err_en_US,`.
  **L2622 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' is not a dummy argument of procedure interface '%s'"_err_en_US,`。
- **L2623 EN**: Comment explains nearby logic, intent, or metadata: `passName, interface->name());`.
  **L2623 CN**: 注释说明附近代码的逻辑、意图或元数据：`passName, interface->name());`。
- **L2624 EN**: Returns from the current function with `void`.
  **L2624 CN**: 以 `void` 从当前函数返回。
- **L2625 EN**: Closes the current lexical scope or compound statement.
  **L2625 CN**: 结束当前词法作用域或复合语句块。
- **L2626 EN**: Executes a standalone statement or declaration: `const Symbol &passArg{*dummyArgs[*passArgIndex]};`.
  **L2626 CN**: 执行一条独立语句或声明：`const Symbol &passArg{*dummyArgs[*passArgIndex]};`。
- **L2627 EN**: Executes a standalone statement or declaration: `std::optional<parser::MessageFixedText> msg;`.
  **L2627 CN**: 执行一条独立语句或声明：`std::optional<parser::MessageFixedText> msg;`。
- **L2628 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2628 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2629 EN**: Continues the surrounding expression or declaration: `msg = "Passed-object dummy argument '%s' of procedure '%s'"`.
  **L2629 CN**: 继续构造周围的表达式或声明：`msg = "Passed-object dummy argument '%s' of procedure '%s'"`。
- **L2630 EN**: Executes a standalone statement or declaration: `" must be a data object"_err_en_US;`.
  **L2630 CN**: 执行一条独立语句或声明：`" must be a data object"_err_en_US;`。
- **L2631 EN**: Transitions from the previous branch into an `else if` condition.
  **L2631 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2632 EN**: Continues the surrounding expression or declaration: `msg = "Passed-object dummy argument '%s' of procedure '%s'"`.
  **L2632 CN**: 继续构造周围的表达式或声明：`msg = "Passed-object dummy argument '%s' of procedure '%s'"`。
- **L2633 EN**: Executes a standalone statement or declaration: `" may not have the ALLOCATABLE attribute"_err_en_US;`.
  **L2633 CN**: 执行一条独立语句或声明：`" may not have the ALLOCATABLE attribute"_err_en_US;`。
- **L2634 EN**: Transitions from the previous branch into an `else if` condition.
  **L2634 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2635 EN**: Continues the surrounding expression or declaration: `msg = "Passed-object dummy argument '%s' of procedure '%s'"`.
  **L2635 CN**: 继续构造周围的表达式或声明：`msg = "Passed-object dummy argument '%s' of procedure '%s'"`。
- **L2636 EN**: Executes a standalone statement or declaration: `" may not have the VALUE attribute"_err_en_US;`.
  **L2636 CN**: 执行一条独立语句或声明：`" may not have the VALUE attribute"_err_en_US;`。
- **L2637 EN**: Transitions from the previous branch into an `else if` condition.
  **L2637 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2638 EN**: Continues the surrounding expression or declaration: `msg = "Passed-object dummy argument '%s' of procedure '%s'"`.
  **L2638 CN**: 继续构造周围的表达式或声明：`msg = "Passed-object dummy argument '%s' of procedure '%s'"`。
- **L2639 EN**: Executes a standalone statement or declaration: `" must be scalar"_err_en_US;`.
  **L2639 CN**: 执行一条独立语句或声明：`" must be scalar"_err_en_US;`。
- **L2640 EN**: Transitions from the previous branch into an `else if` condition.
  **L2640 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 2641-2664

````cpp
    if (context_.IsEnabled(common::LanguageFeature::PointerPassObject) &&
        IsIntentIn(passArg)) {
      if (proc.has<ProcBindingDetails>()) {
        // Extension: allow a passed object to be an INTENT(IN) POINTER.
        // Only works for TBPs, needs lowering work for proc ptr components.
        Warn(common::LanguageFeature::PointerPassObject, name,
            "Passed-object dummy argument '%s' of procedure '%s' that is an INTENT(IN) POINTER is not standard"_port_en_US,
            *passName, name);
      } else {
        msg =
            "Passed-object dummy argument '%s' of procedure '%s' used as procedure pointer component interface may not have the POINTER attribute"_err_en_US;
      }
    } else {
      msg =
          "Passed-object dummy argument '%s' of procedure '%s' may not have the POINTER attribute unless INTENT(IN)"_err_en_US;
    }
  }
  if (msg) {
    messages_.Say(name, std::move(*msg), passName.value(), name);
    return;
  }
  const DeclTypeSpec *type{passArg.GetType()};
  if (!type) {
    return; // an error already occurred
````
- **L2641 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2641 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2642 EN**: Starts a function, method, lambda, or structured scope: `IsIntentIn(passArg)) {`.
  **L2642 CN**: 开始一个函数、方法、lambda 或结构化作用域：`IsIntentIn(passArg)) {`。
- **L2643 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2643 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2644 EN**: Comment explains nearby logic, intent, or metadata: `Extension: allow a passed object to be an INTENT(IN) POINTER.`.
  **L2644 CN**: 注释说明附近代码的逻辑、意图或元数据：`Extension: allow a passed object to be an INTENT(IN) POINTER.`。
- **L2645 EN**: Comment explains nearby logic, intent, or metadata: `Only works for TBPs, needs lowering work for proc ptr components.`.
  **L2645 CN**: 注释说明附近代码的逻辑、意图或元数据：`Only works for TBPs, needs lowering work for proc ptr components.`。
- **L2646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::LanguageFeature::PointerPassObject, name,`.
  **L2646 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::LanguageFeature::PointerPassObject, name,`。
- **L2647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Passed-object dummy argument '%s' of procedure '%s' that is an INTENT(IN) POINTER is not standard"_port_en_US,`.
  **L2647 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Passed-object dummy argument '%s' of procedure '%s' that is an INTENT(IN) POINTER is not standard"_port_en_US,`。
- **L2648 EN**: Comment explains nearby logic, intent, or metadata: `passName, name);`.
  **L2648 CN**: 注释说明附近代码的逻辑、意图或元数据：`passName, name);`。
- **L2649 EN**: Transitions from the previous branch into the alternative path.
  **L2649 CN**: 从前一个分支过渡到备选路径。
- **L2650 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2650 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2651 EN**: Executes a standalone statement or declaration: `"Passed-object dummy argument '%s' of procedure '%s' used as procedure pointer component interface may not have the POINTER attribute"_err_en_US;`.
  **L2651 CN**: 执行一条独立语句或声明：`"Passed-object dummy argument '%s' of procedure '%s' used as procedure pointer component interface may not have the POINTER attribute"_err_en_US;`。
- **L2652 EN**: Closes the current lexical scope or compound statement.
  **L2652 CN**: 结束当前词法作用域或复合语句块。
- **L2653 EN**: Transitions from the previous branch into the alternative path.
  **L2653 CN**: 从前一个分支过渡到备选路径。
- **L2654 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2654 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2655 EN**: Executes a call or declaration centered on `INTENT`.
  **L2655 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L2656 EN**: Closes the current lexical scope or compound statement.
  **L2656 CN**: 结束当前词法作用域或复合语句块。
- **L2657 EN**: Closes the current lexical scope or compound statement.
  **L2657 CN**: 结束当前词法作用域或复合语句块。
- **L2658 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2658 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2659 EN**: Executes a call or declaration centered on `messages_.Say`.
  **L2659 CN**: 执行以 `messages_.Say` 为核心的调用或声明。
- **L2660 EN**: Returns from the current function with `void`.
  **L2660 CN**: 以 `void` 从当前函数返回。
- **L2661 EN**: Closes the current lexical scope or compound statement.
  **L2661 CN**: 结束当前词法作用域或复合语句块。
- **L2662 EN**: Executes a call or declaration centered on `*type{passArg.GetType`.
  **L2662 CN**: 执行以 `*type{passArg.GetType` 为核心的调用或声明。
- **L2663 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2663 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2664 EN**: Returns from the current function with `; // an error already occurred`.
  **L2664 CN**: 以 `; // an error already occurred` 从当前函数返回。

### Lines 2665-2688

````cpp
  }
  const Symbol &typeSymbol{*proc.owner().GetSymbol()};
  const DerivedTypeSpec *derived{type->AsDerived()};
  if (!derived || derived->typeSymbol() != typeSymbol) {
    messages_.Say(name,
        "Passed-object dummy argument '%s' of procedure '%s'"
        " must be of type '%s' but is '%s'"_err_en_US,
        passName.value(), name, typeSymbol.name(), type->AsFortran());
    return;
  }
  if (IsExtensibleType(derived) != type->IsPolymorphic()) {
    messages_.Say(name,
        type->IsPolymorphic()
            ? "Passed-object dummy argument '%s' of procedure '%s'"
              " may not be polymorphic because '%s' is not extensible"_err_en_US
            : "Passed-object dummy argument '%s' of procedure '%s'"
              " must be polymorphic because '%s' is extensible"_err_en_US,
        passName.value(), name, typeSymbol.name());
    return;
  }
  for (const auto &[paramName, paramValue] : derived->parameters()) {
    if (paramValue.isLen() && !paramValue.isAssumed()) {
      messages_.Say(name,
          "Passed-object dummy argument '%s' of procedure '%s'"
````
- **L2665 EN**: Closes the current lexical scope or compound statement.
  **L2665 CN**: 结束当前词法作用域或复合语句块。
- **L2666 EN**: Executes a call or declaration centered on `&typeSymbol{*proc.owner`.
  **L2666 CN**: 执行以 `&typeSymbol{*proc.owner` 为核心的调用或声明。
- **L2667 EN**: Executes a call or declaration centered on `*derived{type->AsDerived`.
  **L2667 CN**: 执行以 `*derived{type->AsDerived` 为核心的调用或声明。
- **L2668 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2668 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(name,`.
  **L2669 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(name,`。
- **L2670 EN**: Continues the surrounding expression or declaration: `"Passed-object dummy argument '%s' of procedure '%s'"`.
  **L2670 CN**: 继续构造周围的表达式或声明：`"Passed-object dummy argument '%s' of procedure '%s'"`。
- **L2671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" must be of type '%s' but is '%s'"_err_en_US,`.
  **L2671 CN**: 继续一个多行参数列表、初始化器或聚合项：`" must be of type '%s' but is '%s'"_err_en_US,`。
- **L2672 EN**: Executes a call or declaration centered on `passName.value`.
  **L2672 CN**: 执行以 `passName.value` 为核心的调用或声明。
- **L2673 EN**: Returns from the current function with `void`.
  **L2673 CN**: 以 `void` 从当前函数返回。
- **L2674 EN**: Closes the current lexical scope or compound statement.
  **L2674 CN**: 结束当前词法作用域或复合语句块。
- **L2675 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2675 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2676 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(name,`.
  **L2676 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(name,`。
- **L2677 EN**: Continues logic associated with callable symbol `IsPolymorphic`.
  **L2677 CN**: 继续与可调用符号 `IsPolymorphic` 相关的逻辑。
- **L2678 EN**: Continues the surrounding expression or declaration: `? "Passed-object dummy argument '%s' of procedure '%s'"`.
  **L2678 CN**: 继续构造周围的表达式或声明：`? "Passed-object dummy argument '%s' of procedure '%s'"`。
- **L2679 EN**: Continues the surrounding expression or declaration: `" may not be polymorphic because '%s' is not extensible"_err_en_US`.
  **L2679 CN**: 继续构造周围的表达式或声明：`" may not be polymorphic because '%s' is not extensible"_err_en_US`。
- **L2680 EN**: Continues the surrounding expression or declaration: `: "Passed-object dummy argument '%s' of procedure '%s'"`.
  **L2680 CN**: 继续构造周围的表达式或声明：`: "Passed-object dummy argument '%s' of procedure '%s'"`。
- **L2681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" must be polymorphic because '%s' is extensible"_err_en_US,`.
  **L2681 CN**: 继续一个多行参数列表、初始化器或聚合项：`" must be polymorphic because '%s' is extensible"_err_en_US,`。
- **L2682 EN**: Executes a call or declaration centered on `passName.value`.
  **L2682 CN**: 执行以 `passName.value` 为核心的调用或声明。
- **L2683 EN**: Returns from the current function with `void`.
  **L2683 CN**: 以 `void` 从当前函数返回。
- **L2684 EN**: Closes the current lexical scope or compound statement.
  **L2684 CN**: 结束当前词法作用域或复合语句块。
- **L2685 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2685 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2686 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2686 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2687 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(name,`.
  **L2687 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(name,`。
- **L2688 EN**: Continues the surrounding expression or declaration: `"Passed-object dummy argument '%s' of procedure '%s'"`.
  **L2688 CN**: 继续构造周围的表达式或声明：`"Passed-object dummy argument '%s' of procedure '%s'"`。

### Lines 2689-2712

````cpp
          " has non-assumed length parameter '%s'"_err_en_US,
          passName.value(), name, paramName);
    }
  }
}

void CheckHelper::CheckProcBinding(
    const Symbol &symbol, const ProcBindingDetails &binding) {
  const Scope &dtScope{symbol.owner()};
  CHECK(dtScope.kind() == Scope::Kind::DerivedType);
  bool isInaccessibleDeferred{false};
  const Symbol *overridden{
      FindOverriddenBinding(symbol, isInaccessibleDeferred)};
  if (symbol.attrs().test(Attr::DEFERRED)) {
    if (const Symbol *dtSymbol{dtScope.symbol()}) {
      if (!dtSymbol->attrs().test(Attr::ABSTRACT)) { // C733
        SayWithDeclaration(*dtSymbol,
            "Procedure bound to non-ABSTRACT derived type '%s' may not be DEFERRED"_err_en_US,
            dtSymbol->name());
      }
    }
    if (symbol.attrs().test(Attr::NON_OVERRIDABLE)) {
      messages_.Say(
          "Type-bound procedure '%s' may not be both DEFERRED and NON_OVERRIDABLE"_err_en_US,
````
- **L2689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" has non-assumed length parameter '%s'"_err_en_US,`.
  **L2689 CN**: 继续一个多行参数列表、初始化器或聚合项：`" has non-assumed length parameter '%s'"_err_en_US,`。
- **L2690 EN**: Executes a call or declaration centered on `passName.value`.
  **L2690 CN**: 执行以 `passName.value` 为核心的调用或声明。
- **L2691 EN**: Closes the current lexical scope or compound statement.
  **L2691 CN**: 结束当前词法作用域或复合语句块。
- **L2692 EN**: Closes the current lexical scope or compound statement.
  **L2692 CN**: 结束当前词法作用域或复合语句块。
- **L2693 EN**: Closes the current lexical scope or compound statement.
  **L2693 CN**: 结束当前词法作用域或复合语句块。
- **L2694 EN**: Blank line separating nearby declarations or logic blocks.
  **L2694 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2695 EN**: Continues logic associated with callable symbol `CheckProcBinding`.
  **L2695 CN**: 继续与可调用符号 `CheckProcBinding` 相关的逻辑。
- **L2696 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, const ProcBindingDetails &binding) {`.
  **L2696 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, const ProcBindingDetails &binding) {`。
- **L2697 EN**: Executes a call or declaration centered on `&dtScope{symbol.owner`.
  **L2697 CN**: 执行以 `&dtScope{symbol.owner` 为核心的调用或声明。
- **L2698 EN**: Executes a call or declaration centered on `CHECK`.
  **L2698 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2699 EN**: Executes a standalone statement or declaration: `bool isInaccessibleDeferred{false};`.
  **L2699 CN**: 执行一条独立语句或声明：`bool isInaccessibleDeferred{false};`。
- **L2700 EN**: Continues the surrounding expression or declaration: `const Symbol *overridden{`.
  **L2700 CN**: 继续构造周围的表达式或声明：`const Symbol *overridden{`。
- **L2701 EN**: Executes a call or declaration centered on `FindOverriddenBinding`.
  **L2701 CN**: 执行以 `FindOverriddenBinding` 为核心的调用或声明。
- **L2702 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2702 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2703 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2703 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2704 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2704 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*dtSymbol,`.
  **L2705 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*dtSymbol,`。
- **L2706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Procedure bound to non-ABSTRACT derived type '%s' may not be DEFERRED"_err_en_US,`.
  **L2706 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Procedure bound to non-ABSTRACT derived type '%s' may not be DEFERRED"_err_en_US,`。
- **L2707 EN**: Executes a call or declaration centered on `dtSymbol->name`.
  **L2707 CN**: 执行以 `dtSymbol->name` 为核心的调用或声明。
- **L2708 EN**: Closes the current lexical scope or compound statement.
  **L2708 CN**: 结束当前词法作用域或复合语句块。
- **L2709 EN**: Closes the current lexical scope or compound statement.
  **L2709 CN**: 结束当前词法作用域或复合语句块。
- **L2710 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2710 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2711 EN**: Continues logic associated with callable symbol `Say`.
  **L2711 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Type-bound procedure '%s' may not be both DEFERRED and NON_OVERRIDABLE"_err_en_US,`.
  **L2712 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Type-bound procedure '%s' may not be both DEFERRED and NON_OVERRIDABLE"_err_en_US,`。

### Lines 2713-2736

````cpp
          symbol.name());
    }
    if (overridden && !overridden->attrs().test(Attr::DEFERRED)) {
      SayWithDeclaration(*overridden,
          "Override of non-DEFERRED '%s' must not be DEFERRED"_err_en_US,
          symbol.name());
    }
  }
  if (binding.symbol().attrs().test(Attr::INTRINSIC) &&
      !context_.intrinsics().IsSpecificIntrinsicFunction(
          binding.symbol().name().ToString())) {
    messages_.Say(
        "Intrinsic procedure '%s' is not a specific intrinsic permitted for use in the definition of binding '%s'"_err_en_US,
        binding.symbol().name(), symbol.name());
  }
  if (overridden) {
    if (isInaccessibleDeferred) {
      evaluate::AttachDeclaration(
          Warn(common::LanguageFeature::InaccessibleDeferredOverride,
              symbol.name(),
              "Override of PRIVATE DEFERRED '%s' should appear in its module"_warn_en_US,
              symbol.name()),
          *overridden);
    }
````
- **L2713 EN**: Executes a call or declaration centered on `symbol.name`.
  **L2713 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L2714 EN**: Closes the current lexical scope or compound statement.
  **L2714 CN**: 结束当前词法作用域或复合语句块。
- **L2715 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2715 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*overridden,`.
  **L2716 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*overridden,`。
- **L2717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Override of non-DEFERRED '%s' must not be DEFERRED"_err_en_US,`.
  **L2717 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Override of non-DEFERRED '%s' must not be DEFERRED"_err_en_US,`。
- **L2718 EN**: Executes a call or declaration centered on `symbol.name`.
  **L2718 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L2719 EN**: Closes the current lexical scope or compound statement.
  **L2719 CN**: 结束当前词法作用域或复合语句块。
- **L2720 EN**: Closes the current lexical scope or compound statement.
  **L2720 CN**: 结束当前词法作用域或复合语句块。
- **L2721 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2721 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2722 EN**: Continues logic associated with callable symbol `intrinsics`.
  **L2722 CN**: 继续与可调用符号 `intrinsics` 相关的逻辑。
- **L2723 EN**: Starts a function, method, lambda, or structured scope: `binding.symbol().name().ToString())) {`.
  **L2723 CN**: 开始一个函数、方法、lambda 或结构化作用域：`binding.symbol().name().ToString())) {`。
- **L2724 EN**: Continues logic associated with callable symbol `Say`.
  **L2724 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Intrinsic procedure '%s' is not a specific intrinsic permitted for use in the definition of binding '%s'"_err_en_US,`.
  **L2725 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Intrinsic procedure '%s' is not a specific intrinsic permitted for use in the definition of binding '%s'"_err_en_US,`。
- **L2726 EN**: Executes a call or declaration centered on `binding.symbol`.
  **L2726 CN**: 执行以 `binding.symbol` 为核心的调用或声明。
- **L2727 EN**: Closes the current lexical scope or compound statement.
  **L2727 CN**: 结束当前词法作用域或复合语句块。
- **L2728 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2728 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2729 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2729 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2730 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L2730 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L2731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::LanguageFeature::InaccessibleDeferredOverride,`.
  **L2731 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::LanguageFeature::InaccessibleDeferredOverride,`。
- **L2732 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol.name(),`.
  **L2732 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol.name(),`。
- **L2733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Override of PRIVATE DEFERRED '%s' should appear in its module"_warn_en_US,`.
  **L2733 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Override of PRIVATE DEFERRED '%s' should appear in its module"_warn_en_US,`。
- **L2734 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol.name()),`.
  **L2734 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol.name()),`。
- **L2735 EN**: Comment explains nearby logic, intent, or metadata: `overridden);`.
  **L2735 CN**: 注释说明附近代码的逻辑、意图或元数据：`overridden);`。
- **L2736 EN**: Closes the current lexical scope or compound statement.
  **L2736 CN**: 结束当前词法作用域或复合语句块。

### Lines 2737-2760

````cpp
    if (overridden->attrs().test(Attr::NON_OVERRIDABLE)) {
      SayWithDeclaration(*overridden,
          "Override of NON_OVERRIDABLE '%s' is not permitted"_err_en_US,
          symbol.name());
    }
    if (const auto *overriddenBinding{
            overridden->detailsIf<ProcBindingDetails>()}) {
      if (!IsPureProcedure(symbol) && IsPureProcedure(*overridden)) {
        SayWithDeclaration(*overridden,
            "An overridden pure type-bound procedure binding must also be pure"_err_en_US);
        return;
      }
      if (!IsElementalProcedure(binding.symbol()) &&
          IsElementalProcedure(*overridden)) {
        SayWithDeclaration(*overridden,
            "A type-bound procedure and its override must both, or neither, be ELEMENTAL"_err_en_US);
        return;
      }
      bool isNopass{symbol.attrs().test(Attr::NOPASS)};
      if (isNopass != overridden->attrs().test(Attr::NOPASS)) {
        SayWithDeclaration(*overridden,
            isNopass
                ? "A NOPASS type-bound procedure may not override a passed-argument procedure"_err_en_US
                : "A passed-argument type-bound procedure may not override a NOPASS procedure"_err_en_US);
````
- **L2737 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2737 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*overridden,`.
  **L2738 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*overridden,`。
- **L2739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Override of NON_OVERRIDABLE '%s' is not permitted"_err_en_US,`.
  **L2739 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Override of NON_OVERRIDABLE '%s' is not permitted"_err_en_US,`。
- **L2740 EN**: Executes a call or declaration centered on `symbol.name`.
  **L2740 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L2741 EN**: Closes the current lexical scope or compound statement.
  **L2741 CN**: 结束当前词法作用域或复合语句块。
- **L2742 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2742 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2743 EN**: Starts a function, method, lambda, or structured scope: `overridden->detailsIf<ProcBindingDetails>()}) {`.
  **L2743 CN**: 开始一个函数、方法、lambda 或结构化作用域：`overridden->detailsIf<ProcBindingDetails>()}) {`。
- **L2744 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2744 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2745 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*overridden,`.
  **L2745 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*overridden,`。
- **L2746 EN**: Executes a standalone statement or declaration: `"An overridden pure type-bound procedure binding must also be pure"_err_en_US);`.
  **L2746 CN**: 执行一条独立语句或声明：`"An overridden pure type-bound procedure binding must also be pure"_err_en_US);`。
- **L2747 EN**: Returns from the current function with `void`.
  **L2747 CN**: 以 `void` 从当前函数返回。
- **L2748 EN**: Closes the current lexical scope or compound statement.
  **L2748 CN**: 结束当前词法作用域或复合语句块。
- **L2749 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2749 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2750 EN**: Starts a function, method, lambda, or structured scope: `IsElementalProcedure(*overridden)) {`.
  **L2750 CN**: 开始一个函数、方法、lambda 或结构化作用域：`IsElementalProcedure(*overridden)) {`。
- **L2751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*overridden,`.
  **L2751 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*overridden,`。
- **L2752 EN**: Executes a standalone statement or declaration: `"A type-bound procedure and its override must both, or neither, be ELEMENTAL"_err_en_US);`.
  **L2752 CN**: 执行一条独立语句或声明：`"A type-bound procedure and its override must both, or neither, be ELEMENTAL"_err_en_US);`。
- **L2753 EN**: Returns from the current function with `void`.
  **L2753 CN**: 以 `void` 从当前函数返回。
- **L2754 EN**: Closes the current lexical scope or compound statement.
  **L2754 CN**: 结束当前词法作用域或复合语句块。
- **L2755 EN**: Executes a call or declaration centered on `isNopass{symbol.attrs`.
  **L2755 CN**: 执行以 `isNopass{symbol.attrs` 为核心的调用或声明。
- **L2756 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2756 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*overridden,`.
  **L2757 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*overridden,`。
- **L2758 EN**: Continues the surrounding expression or declaration: `isNopass`.
  **L2758 CN**: 继续构造周围的表达式或声明：`isNopass`。
- **L2759 EN**: Continues the surrounding expression or declaration: `? "A NOPASS type-bound procedure may not override a passed-argument procedure"_err_en_US`.
  **L2759 CN**: 继续构造周围的表达式或声明：`? "A NOPASS type-bound procedure may not override a passed-argument procedure"_err_en_US`。
- **L2760 EN**: Executes a standalone statement or declaration: `: "A passed-argument type-bound procedure may not override a NOPASS procedure"_err_en_US);`.
  **L2760 CN**: 执行一条独立语句或声明：`: "A passed-argument type-bound procedure may not override a NOPASS procedure"_err_en_US);`。

### Lines 2761-2784

````cpp
      } else {
        const auto *bindingChars{Characterize(symbol)};
        const auto *overriddenChars{Characterize(*overridden)};
        if (bindingChars && overriddenChars) {
          if (isNopass) {
            if (!bindingChars->CanOverride(*overriddenChars, std::nullopt)) {
              SayWithDeclaration(*overridden,
                  "A NOPASS type-bound procedure and its override must have identical interfaces"_err_en_US);
            }
          } else if (!context_.HasError(binding.symbol())) {
            auto passIndex{bindingChars->FindPassIndex(binding.passName())};
            auto overriddenPassIndex{
                overriddenChars->FindPassIndex(overriddenBinding->passName())};
            if (passIndex && overriddenPassIndex) {
              if (*passIndex != *overriddenPassIndex) {
                SayWithDeclaration(*overridden,
                    "A type-bound procedure and its override must use the same PASS argument"_err_en_US);
              } else if (!bindingChars->CanOverride(
                             *overriddenChars, passIndex)) {
                SayWithDeclaration(*overridden,
                    "A type-bound procedure and its override must have compatible interfaces"_err_en_US);
              }
            }
          }
````
- **L2761 EN**: Transitions from the previous branch into the alternative path.
  **L2761 CN**: 从前一个分支过渡到备选路径。
- **L2762 EN**: Executes a call or declaration centered on `*bindingChars{Characterize`.
  **L2762 CN**: 执行以 `*bindingChars{Characterize` 为核心的调用或声明。
- **L2763 EN**: Executes a call or declaration centered on `*overriddenChars{Characterize`.
  **L2763 CN**: 执行以 `*overriddenChars{Characterize` 为核心的调用或声明。
- **L2764 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2764 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2765 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2765 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2766 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2766 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2767 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*overridden,`.
  **L2767 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*overridden,`。
- **L2768 EN**: Executes a standalone statement or declaration: `"A NOPASS type-bound procedure and its override must have identical interfaces"_err_en_US);`.
  **L2768 CN**: 执行一条独立语句或声明：`"A NOPASS type-bound procedure and its override must have identical interfaces"_err_en_US);`。
- **L2769 EN**: Closes the current lexical scope or compound statement.
  **L2769 CN**: 结束当前词法作用域或复合语句块。
- **L2770 EN**: Transitions from the previous branch into an `else if` condition.
  **L2770 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2771 EN**: Executes a call or declaration centered on `passIndex{bindingChars->FindPassIndex`.
  **L2771 CN**: 执行以 `passIndex{bindingChars->FindPassIndex` 为核心的调用或声明。
- **L2772 EN**: Continues the surrounding expression or declaration: `auto overriddenPassIndex{`.
  **L2772 CN**: 继续构造周围的表达式或声明：`auto overriddenPassIndex{`。
- **L2773 EN**: Executes a call or declaration centered on `overriddenChars->FindPassIndex`.
  **L2773 CN**: 执行以 `overriddenChars->FindPassIndex` 为核心的调用或声明。
- **L2774 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2774 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2775 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2775 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*overridden,`.
  **L2776 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*overridden,`。
- **L2777 EN**: Executes a standalone statement or declaration: `"A type-bound procedure and its override must use the same PASS argument"_err_en_US);`.
  **L2777 CN**: 执行一条独立语句或声明：`"A type-bound procedure and its override must use the same PASS argument"_err_en_US);`。
- **L2778 EN**: Transitions from the previous branch into an `else if` condition.
  **L2778 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2779 EN**: Comment explains nearby logic, intent, or metadata: `overriddenChars, passIndex)) {`.
  **L2779 CN**: 注释说明附近代码的逻辑、意图或元数据：`overriddenChars, passIndex)) {`。
- **L2780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*overridden,`.
  **L2780 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*overridden,`。
- **L2781 EN**: Executes a standalone statement or declaration: `"A type-bound procedure and its override must have compatible interfaces"_err_en_US);`.
  **L2781 CN**: 执行一条独立语句或声明：`"A type-bound procedure and its override must have compatible interfaces"_err_en_US);`。
- **L2782 EN**: Closes the current lexical scope or compound statement.
  **L2782 CN**: 结束当前词法作用域或复合语句块。
- **L2783 EN**: Closes the current lexical scope or compound statement.
  **L2783 CN**: 结束当前词法作用域或复合语句块。
- **L2784 EN**: Closes the current lexical scope or compound statement.
  **L2784 CN**: 结束当前词法作用域或复合语句块。

### Lines 2785-2808

````cpp
        }
      }
      if (symbol.attrs().test(Attr::PRIVATE)) {
        if (FindModuleContaining(dtScope) ==
            FindModuleContaining(overridden->owner())) {
          // types declared in same madule
          if (!overridden->attrs().test(Attr::PRIVATE)) {
            SayWithDeclaration(*overridden,
                "A PRIVATE procedure may not override a PUBLIC procedure"_err_en_US);
          }
        } else { // types declared in distinct madules
          if (!CheckAccessibleSymbol(dtScope.parent(), *overridden)) {
            SayWithDeclaration(*overridden,
                "A PRIVATE procedure may not override an accessible procedure"_err_en_US);
          }
        }
      }
    } else {
      SayWithDeclaration(*overridden,
          "A type-bound procedure binding may not have the same name as a parent component"_err_en_US);
    }
  }
  CheckPassArg(symbol, &binding.symbol(), binding);
}
````
- **L2785 EN**: Closes the current lexical scope or compound statement.
  **L2785 CN**: 结束当前词法作用域或复合语句块。
- **L2786 EN**: Closes the current lexical scope or compound statement.
  **L2786 CN**: 结束当前词法作用域或复合语句块。
- **L2787 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2787 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2788 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2788 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2789 EN**: Starts a function, method, lambda, or structured scope: `FindModuleContaining(overridden->owner())) {`.
  **L2789 CN**: 开始一个函数、方法、lambda 或结构化作用域：`FindModuleContaining(overridden->owner())) {`。
- **L2790 EN**: Comment explains nearby logic, intent, or metadata: `types declared in same madule`.
  **L2790 CN**: 注释说明附近代码的逻辑、意图或元数据：`types declared in same madule`。
- **L2791 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2791 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*overridden,`.
  **L2792 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*overridden,`。
- **L2793 EN**: Executes a standalone statement or declaration: `"A PRIVATE procedure may not override a PUBLIC procedure"_err_en_US);`.
  **L2793 CN**: 执行一条独立语句或声明：`"A PRIVATE procedure may not override a PUBLIC procedure"_err_en_US);`。
- **L2794 EN**: Closes the current lexical scope or compound statement.
  **L2794 CN**: 结束当前词法作用域或复合语句块。
- **L2795 EN**: Transitions from the previous branch into the alternative path.
  **L2795 CN**: 从前一个分支过渡到备选路径。
- **L2796 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2796 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2797 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*overridden,`.
  **L2797 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*overridden,`。
- **L2798 EN**: Executes a standalone statement or declaration: `"A PRIVATE procedure may not override an accessible procedure"_err_en_US);`.
  **L2798 CN**: 执行一条独立语句或声明：`"A PRIVATE procedure may not override an accessible procedure"_err_en_US);`。
- **L2799 EN**: Closes the current lexical scope or compound statement.
  **L2799 CN**: 结束当前词法作用域或复合语句块。
- **L2800 EN**: Closes the current lexical scope or compound statement.
  **L2800 CN**: 结束当前词法作用域或复合语句块。
- **L2801 EN**: Closes the current lexical scope or compound statement.
  **L2801 CN**: 结束当前词法作用域或复合语句块。
- **L2802 EN**: Transitions from the previous branch into the alternative path.
  **L2802 CN**: 从前一个分支过渡到备选路径。
- **L2803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*overridden,`.
  **L2803 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*overridden,`。
- **L2804 EN**: Executes a standalone statement or declaration: `"A type-bound procedure binding may not have the same name as a parent component"_err_en_US);`.
  **L2804 CN**: 执行一条独立语句或声明：`"A type-bound procedure binding may not have the same name as a parent component"_err_en_US);`。
- **L2805 EN**: Closes the current lexical scope or compound statement.
  **L2805 CN**: 结束当前词法作用域或复合语句块。
- **L2806 EN**: Closes the current lexical scope or compound statement.
  **L2806 CN**: 结束当前词法作用域或复合语句块。
- **L2807 EN**: Executes a call or declaration centered on `CheckPassArg`.
  **L2807 CN**: 执行以 `CheckPassArg` 为核心的调用或声明。
- **L2808 EN**: Closes the current lexical scope or compound statement.
  **L2808 CN**: 结束当前词法作用域或复合语句块。

### Lines 2809-2832

````cpp

void CheckHelper::Check(const Scope &scope) {
  scope_ = &scope;
  common::Restorer<const Symbol *> restorer{innermostSymbol_, innermostSymbol_};
  if (const Symbol *symbol{scope.symbol()}) {
    innermostSymbol_ = symbol;
  }
  if (scope.IsParameterizedDerivedTypeInstantiation()) {
    auto restorer{common::ScopedSet(scopeIsUninstantiatedPDT_, false)};
    auto restorer2{context_.foldingContext().messages().SetContext(
        scope.instantiationContext().get())};
    for (const auto &pair : scope) {
      CheckPointerInitialization(*pair.second);
    }
  } else {
    auto restorer{common::ScopedSet(
        scopeIsUninstantiatedPDT_, scope.IsParameterizedDerivedType())};
    for (const auto &set : scope.equivalenceSets()) {
      CheckEquivalenceSet(set);
    }
    for (const auto &pair : scope) {
      Check(*pair.second);
    }
    if (scope.IsSubmodule() && scope.symbol()) {
````
- **L2809 EN**: Blank line separating nearby declarations or logic blocks.
  **L2809 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2810 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::Check(const Scope &scope) {`.
  **L2810 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::Check(const Scope &scope) {`。
- **L2811 EN**: Executes a standalone statement or declaration: `scope_ = &scope;`.
  **L2811 CN**: 执行一条独立语句或声明：`scope_ = &scope;`。
- **L2812 EN**: Executes a standalone statement or declaration: `common::Restorer<const Symbol *> restorer{innermostSymbol_, innermostSymbol_};`.
  **L2812 CN**: 执行一条独立语句或声明：`common::Restorer<const Symbol *> restorer{innermostSymbol_, innermostSymbol_};`。
- **L2813 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2813 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2814 EN**: Executes a standalone statement or declaration: `innermostSymbol_ = symbol;`.
  **L2814 CN**: 执行一条独立语句或声明：`innermostSymbol_ = symbol;`。
- **L2815 EN**: Closes the current lexical scope or compound statement.
  **L2815 CN**: 结束当前词法作用域或复合语句块。
- **L2816 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2816 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2817 EN**: Executes a call or declaration centered on `restorer{common::ScopedSet`.
  **L2817 CN**: 执行以 `restorer{common::ScopedSet` 为核心的调用或声明。
- **L2818 EN**: Continues logic associated with callable symbol `foldingContext`.
  **L2818 CN**: 继续与可调用符号 `foldingContext` 相关的逻辑。
- **L2819 EN**: Executes a call or declaration centered on `scope.instantiationContext`.
  **L2819 CN**: 执行以 `scope.instantiationContext` 为核心的调用或声明。
- **L2820 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2820 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2821 EN**: Executes a call or declaration centered on `CheckPointerInitialization`.
  **L2821 CN**: 执行以 `CheckPointerInitialization` 为核心的调用或声明。
- **L2822 EN**: Closes the current lexical scope or compound statement.
  **L2822 CN**: 结束当前词法作用域或复合语句块。
- **L2823 EN**: Transitions from the previous branch into the alternative path.
  **L2823 CN**: 从前一个分支过渡到备选路径。
- **L2824 EN**: Continues logic associated with callable symbol `ScopedSet`.
  **L2824 CN**: 继续与可调用符号 `ScopedSet` 相关的逻辑。
- **L2825 EN**: Executes a call or declaration centered on `scope.IsParameterizedDerivedType`.
  **L2825 CN**: 执行以 `scope.IsParameterizedDerivedType` 为核心的调用或声明。
- **L2826 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2826 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2827 EN**: Executes a call or declaration centered on `CheckEquivalenceSet`.
  **L2827 CN**: 执行以 `CheckEquivalenceSet` 为核心的调用或声明。
- **L2828 EN**: Closes the current lexical scope or compound statement.
  **L2828 CN**: 结束当前词法作用域或复合语句块。
- **L2829 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2829 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2830 EN**: Executes a call or declaration centered on `Check`.
  **L2830 CN**: 执行以 `Check` 为核心的调用或声明。
- **L2831 EN**: Closes the current lexical scope or compound statement.
  **L2831 CN**: 结束当前词法作用域或复合语句块。
- **L2832 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2832 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2833-2856

````cpp
      // Submodule names are not in their parent's scopes
      Check(*scope.symbol());
    }
    for (const auto &pair : scope.commonBlocks()) {
      CheckCommonBlock(*pair.second);
    }
    int mainProgCnt{0};
    for (const Scope &child : scope.children()) {
      Check(child);
      // A program shall consist of exactly one main program (5.2.2).
      if (child.kind() == Scope::Kind::MainProgram) {
        ++mainProgCnt;
        if (mainProgCnt > 1) {
          messages_.Say(child.sourceRange(),
              "A source file cannot contain more than one main program"_err_en_US);
        }
      }
    }
    if (scope.kind() == Scope::Kind::BlockData) {
      CheckBlockData(scope);
    }
    if (auto name{scope.GetName()}) {
      auto iter{scope.find(*name)};
      if (iter != scope.end()) {
````
- **L2833 EN**: Comment explains nearby logic, intent, or metadata: `Submodule names are not in their parent's scopes`.
  **L2833 CN**: 注释说明附近代码的逻辑、意图或元数据：`Submodule names are not in their parent's scopes`。
- **L2834 EN**: Executes a call or declaration centered on `Check`.
  **L2834 CN**: 执行以 `Check` 为核心的调用或声明。
- **L2835 EN**: Closes the current lexical scope or compound statement.
  **L2835 CN**: 结束当前词法作用域或复合语句块。
- **L2836 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2836 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2837 EN**: Executes a call or declaration centered on `CheckCommonBlock`.
  **L2837 CN**: 执行以 `CheckCommonBlock` 为核心的调用或声明。
- **L2838 EN**: Closes the current lexical scope or compound statement.
  **L2838 CN**: 结束当前词法作用域或复合语句块。
- **L2839 EN**: Executes a standalone statement or declaration: `int mainProgCnt{0};`.
  **L2839 CN**: 执行一条独立语句或声明：`int mainProgCnt{0};`。
- **L2840 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2840 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2841 EN**: Executes a call or declaration centered on `Check`.
  **L2841 CN**: 执行以 `Check` 为核心的调用或声明。
- **L2842 EN**: Comment explains nearby logic, intent, or metadata: `A program shall consist of exactly one main program (5.2.2).`.
  **L2842 CN**: 注释说明附近代码的逻辑、意图或元数据：`A program shall consist of exactly one main program (5.2.2).`。
- **L2843 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2843 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2844 EN**: Executes a standalone statement or declaration: `++mainProgCnt;`.
  **L2844 CN**: 执行一条独立语句或声明：`++mainProgCnt;`。
- **L2845 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2845 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2846 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(child.sourceRange(),`.
  **L2846 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(child.sourceRange(),`。
- **L2847 EN**: Executes a standalone statement or declaration: `"A source file cannot contain more than one main program"_err_en_US);`.
  **L2847 CN**: 执行一条独立语句或声明：`"A source file cannot contain more than one main program"_err_en_US);`。
- **L2848 EN**: Closes the current lexical scope or compound statement.
  **L2848 CN**: 结束当前词法作用域或复合语句块。
- **L2849 EN**: Closes the current lexical scope or compound statement.
  **L2849 CN**: 结束当前词法作用域或复合语句块。
- **L2850 EN**: Closes the current lexical scope or compound statement.
  **L2850 CN**: 结束当前词法作用域或复合语句块。
- **L2851 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2851 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2852 EN**: Executes a call or declaration centered on `CheckBlockData`.
  **L2852 CN**: 执行以 `CheckBlockData` 为核心的调用或声明。
- **L2853 EN**: Closes the current lexical scope or compound statement.
  **L2853 CN**: 结束当前词法作用域或复合语句块。
- **L2854 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2854 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2855 EN**: Executes a call or declaration centered on `iter{scope.find`.
  **L2855 CN**: 执行以 `iter{scope.find` 为核心的调用或声明。
- **L2856 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2856 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2857-2880

````cpp
        const char *kind{nullptr};
        switch (scope.kind()) {
        case Scope::Kind::Module:
          kind = scope.symbol()->get<ModuleDetails>().isSubmodule()
              ? "submodule"
              : "module";
          break;
        case Scope::Kind::MainProgram:
          kind = "main program";
          break;
        case Scope::Kind::BlockData:
          kind = "BLOCK DATA subprogram";
          break;
        default:;
        }
        if (kind) {
          Warn(common::LanguageFeature::BenignNameClash, iter->second->name(),
              "Name '%s' declared in a %s should not have the same name as the %s"_port_en_US,
              *name, kind, kind);
        }
      }
    }
    CheckGenericOps(scope);
  }
````
- **L2857 EN**: Executes a standalone statement or declaration: `const char *kind{nullptr};`.
  **L2857 CN**: 执行一条独立语句或声明：`const char *kind{nullptr};`。
- **L2858 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2858 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2859 EN**: Introduces a switch dispatch label: `case Scope::Kind::Module:`.
  **L2859 CN**: 引入一个 switch 分发标签：`case Scope::Kind::Module:`。
- **L2860 EN**: Continues logic associated with callable symbol `symbol`.
  **L2860 CN**: 继续与可调用符号 `symbol` 相关的逻辑。
- **L2861 EN**: Continues the surrounding expression or declaration: `? "submodule"`.
  **L2861 CN**: 继续构造周围的表达式或声明：`? "submodule"`。
- **L2862 EN**: Executes a standalone statement or declaration: `: "module";`.
  **L2862 CN**: 执行一条独立语句或声明：`: "module";`。
- **L2863 EN**: Exits the nearest loop or switch statement.
  **L2863 CN**: 退出最近的循环或 switch 语句。
- **L2864 EN**: Introduces a switch dispatch label: `case Scope::Kind::MainProgram:`.
  **L2864 CN**: 引入一个 switch 分发标签：`case Scope::Kind::MainProgram:`。
- **L2865 EN**: Executes a standalone statement or declaration: `kind = "main program";`.
  **L2865 CN**: 执行一条独立语句或声明：`kind = "main program";`。
- **L2866 EN**: Exits the nearest loop or switch statement.
  **L2866 CN**: 退出最近的循环或 switch 语句。
- **L2867 EN**: Introduces a switch dispatch label: `case Scope::Kind::BlockData:`.
  **L2867 CN**: 引入一个 switch 分发标签：`case Scope::Kind::BlockData:`。
- **L2868 EN**: Executes a standalone statement or declaration: `kind = "BLOCK DATA subprogram";`.
  **L2868 CN**: 执行一条独立语句或声明：`kind = "BLOCK DATA subprogram";`。
- **L2869 EN**: Exits the nearest loop or switch statement.
  **L2869 CN**: 退出最近的循环或 switch 语句。
- **L2870 EN**: Introduces a switch dispatch label: `default:;`.
  **L2870 CN**: 引入一个 switch 分发标签：`default:;`。
- **L2871 EN**: Closes the current lexical scope or compound statement.
  **L2871 CN**: 结束当前词法作用域或复合语句块。
- **L2872 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2872 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2873 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::LanguageFeature::BenignNameClash, iter->second->name(),`.
  **L2873 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::LanguageFeature::BenignNameClash, iter->second->name(),`。
- **L2874 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Name '%s' declared in a %s should not have the same name as the %s"_port_en_US,`.
  **L2874 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Name '%s' declared in a %s should not have the same name as the %s"_port_en_US,`。
- **L2875 EN**: Comment explains nearby logic, intent, or metadata: `name, kind, kind);`.
  **L2875 CN**: 注释说明附近代码的逻辑、意图或元数据：`name, kind, kind);`。
- **L2876 EN**: Closes the current lexical scope or compound statement.
  **L2876 CN**: 结束当前词法作用域或复合语句块。
- **L2877 EN**: Closes the current lexical scope or compound statement.
  **L2877 CN**: 结束当前词法作用域或复合语句块。
- **L2878 EN**: Closes the current lexical scope or compound statement.
  **L2878 CN**: 结束当前词法作用域或复合语句块。
- **L2879 EN**: Executes a call or declaration centered on `CheckGenericOps`.
  **L2879 CN**: 执行以 `CheckGenericOps` 为核心的调用或声明。
- **L2880 EN**: Closes the current lexical scope or compound statement.
  **L2880 CN**: 结束当前词法作用域或复合语句块。

### Lines 2881-2904

````cpp
}

void CheckHelper::CheckEquivalenceSet(const EquivalenceSet &set) {
  auto iter{
      std::find_if(set.begin(), set.end(), [](const EquivalenceObject &object) {
        return FindCommonBlockContaining(object.symbol) != nullptr;
      })};
  if (iter != set.end()) {
    const Symbol &commonBlock{DEREF(FindCommonBlockContaining(iter->symbol))};
    for (auto &object : set) {
      if (&object != &*iter) {
        if (auto *details{object.symbol.detailsIf<ObjectEntityDetails>()}) {
          if (details->commonBlock()) {
            if (details->commonBlock() != &commonBlock) { // 8.10.3 paragraph 1
              if (auto *msg{messages_.Say(object.symbol.name(),
                      "Two objects in the same EQUIVALENCE set may not be members of distinct COMMON blocks"_err_en_US)}) {
                msg->Attach(iter->symbol.name(),
                       "Other object in EQUIVALENCE set"_en_US)
                    .Attach(details->commonBlock()->name(),
                        "COMMON block containing '%s'"_en_US,
                        object.symbol.name())
                    .Attach(commonBlock.name(),
                        "COMMON block containing '%s'"_en_US,
                        iter->symbol.name());
````
- **L2881 EN**: Closes the current lexical scope or compound statement.
  **L2881 CN**: 结束当前词法作用域或复合语句块。
- **L2882 EN**: Blank line separating nearby declarations or logic blocks.
  **L2882 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2883 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckEquivalenceSet(const EquivalenceSet &set) {`.
  **L2883 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckEquivalenceSet(const EquivalenceSet &set) {`。
- **L2884 EN**: Continues the surrounding expression or declaration: `auto iter{`.
  **L2884 CN**: 继续构造周围的表达式或声明：`auto iter{`。
- **L2885 EN**: Starts a function, method, lambda, or structured scope: `std::find_if(set.begin(), set.end(), [](const EquivalenceObject &object) {`.
  **L2885 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::find_if(set.begin(), set.end(), [](const EquivalenceObject &object) {`。
- **L2886 EN**: Returns from the current function with `FindCommonBlockContaining(object.symbol) != nullptr`.
  **L2886 CN**: 以 `FindCommonBlockContaining(object.symbol) != nullptr` 从当前函数返回。
- **L2887 EN**: Executes a standalone statement or declaration: `})};`.
  **L2887 CN**: 执行一条独立语句或声明：`})};`。
- **L2888 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2888 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2889 EN**: Executes a call or declaration centered on `&commonBlock{DEREF`.
  **L2889 CN**: 执行以 `&commonBlock{DEREF` 为核心的调用或声明。
- **L2890 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2890 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2891 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2891 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2892 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2892 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2893 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2893 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2894 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2894 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2895 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2895 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2896 EN**: Continues the surrounding expression or declaration: `"Two objects in the same EQUIVALENCE set may not be members of distinct COMMON blocks"_err_en_US)}) {`.
  **L2896 CN**: 继续构造周围的表达式或声明：`"Two objects in the same EQUIVALENCE set may not be members of distinct COMMON blocks"_err_en_US)}) {`。
- **L2897 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg->Attach(iter->symbol.name(),`.
  **L2897 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg->Attach(iter->symbol.name(),`。
- **L2898 EN**: Continues the surrounding expression or declaration: `"Other object in EQUIVALENCE set"_en_US)`.
  **L2898 CN**: 继续构造周围的表达式或声明：`"Other object in EQUIVALENCE set"_en_US)`。
- **L2899 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(details->commonBlock()->name(),`.
  **L2899 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(details->commonBlock()->name(),`。
- **L2900 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"COMMON block containing '%s'"_en_US,`.
  **L2900 CN**: 继续一个多行参数列表、初始化器或聚合项：`"COMMON block containing '%s'"_en_US,`。
- **L2901 EN**: Continues logic associated with callable symbol `name`.
  **L2901 CN**: 继续与可调用符号 `name` 相关的逻辑。
- **L2902 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(commonBlock.name(),`.
  **L2902 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(commonBlock.name(),`。
- **L2903 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"COMMON block containing '%s'"_en_US,`.
  **L2903 CN**: 继续一个多行参数列表、初始化器或聚合项：`"COMMON block containing '%s'"_en_US,`。
- **L2904 EN**: Executes a call or declaration centered on `iter->symbol.name`.
  **L2904 CN**: 执行以 `iter->symbol.name` 为核心的调用或声明。

### Lines 2905-2928

````cpp
              }
            }
          } else {
            // Mark all symbols in the equivalence set with the same COMMON
            // block to prevent spurious error messages about initialization
            // in BLOCK DATA outside COMMON
            details->set_commonBlock(commonBlock);
          }
        }
      }
    }
  }
  for (const EquivalenceObject &object : set) {
    CheckEquivalenceObject(object);
  }
}

static bool InCommonWithBind(const Symbol &symbol) {
  if (const auto *details{symbol.detailsIf<ObjectEntityDetails>()}) {
    const Symbol *commonBlock{details->commonBlock()};
    return commonBlock && commonBlock->attrs().test(Attr::BIND_C);
  } else {
    return false;
  }
````
- **L2905 EN**: Closes the current lexical scope or compound statement.
  **L2905 CN**: 结束当前词法作用域或复合语句块。
- **L2906 EN**: Closes the current lexical scope or compound statement.
  **L2906 CN**: 结束当前词法作用域或复合语句块。
- **L2907 EN**: Transitions from the previous branch into the alternative path.
  **L2907 CN**: 从前一个分支过渡到备选路径。
- **L2908 EN**: Comment explains nearby logic, intent, or metadata: `Mark all symbols in the equivalence set with the same COMMON`.
  **L2908 CN**: 注释说明附近代码的逻辑、意图或元数据：`Mark all symbols in the equivalence set with the same COMMON`。
- **L2909 EN**: Comment explains nearby logic, intent, or metadata: `block to prevent spurious error messages about initialization`.
  **L2909 CN**: 注释说明附近代码的逻辑、意图或元数据：`block to prevent spurious error messages about initialization`。
- **L2910 EN**: Comment explains nearby logic, intent, or metadata: `in BLOCK DATA outside COMMON`.
  **L2910 CN**: 注释说明附近代码的逻辑、意图或元数据：`in BLOCK DATA outside COMMON`。
- **L2911 EN**: Executes a call or declaration centered on `details->set_commonBlock`.
  **L2911 CN**: 执行以 `details->set_commonBlock` 为核心的调用或声明。
- **L2912 EN**: Closes the current lexical scope or compound statement.
  **L2912 CN**: 结束当前词法作用域或复合语句块。
- **L2913 EN**: Closes the current lexical scope or compound statement.
  **L2913 CN**: 结束当前词法作用域或复合语句块。
- **L2914 EN**: Closes the current lexical scope or compound statement.
  **L2914 CN**: 结束当前词法作用域或复合语句块。
- **L2915 EN**: Closes the current lexical scope or compound statement.
  **L2915 CN**: 结束当前词法作用域或复合语句块。
- **L2916 EN**: Closes the current lexical scope or compound statement.
  **L2916 CN**: 结束当前词法作用域或复合语句块。
- **L2917 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2917 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2918 EN**: Executes a call or declaration centered on `CheckEquivalenceObject`.
  **L2918 CN**: 执行以 `CheckEquivalenceObject` 为核心的调用或声明。
- **L2919 EN**: Closes the current lexical scope or compound statement.
  **L2919 CN**: 结束当前词法作用域或复合语句块。
- **L2920 EN**: Closes the current lexical scope or compound statement.
  **L2920 CN**: 结束当前词法作用域或复合语句块。
- **L2921 EN**: Blank line separating nearby declarations or logic blocks.
  **L2921 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2922 EN**: Starts a function, method, lambda, or structured scope: `static bool InCommonWithBind(const Symbol &symbol) {`.
  **L2922 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool InCommonWithBind(const Symbol &symbol) {`。
- **L2923 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2923 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2924 EN**: Executes a call or declaration centered on `*commonBlock{details->commonBlock`.
  **L2924 CN**: 执行以 `*commonBlock{details->commonBlock` 为核心的调用或声明。
- **L2925 EN**: Returns from the current function with `commonBlock && commonBlock->attrs().test(Attr::BIND_C)`.
  **L2925 CN**: 以 `commonBlock && commonBlock->attrs().test(Attr::BIND_C)` 从当前函数返回。
- **L2926 EN**: Transitions from the previous branch into the alternative path.
  **L2926 CN**: 从前一个分支过渡到备选路径。
- **L2927 EN**: Returns from the current function with `false`.
  **L2927 CN**: 以 `false` 从当前函数返回。
- **L2928 EN**: Closes the current lexical scope or compound statement.
  **L2928 CN**: 结束当前词法作用域或复合语句块。

### Lines 2929-2952

````cpp
}

void CheckHelper::CheckEquivalenceObject(const EquivalenceObject &object) {
  parser::MessageFixedText msg;
  const Symbol &symbol{object.symbol};
  if (symbol.owner().IsDerivedType()) {
    msg =
        "Derived type component '%s' is not allowed in an equivalence set"_err_en_US;
  } else if (IsDummy(symbol)) {
    msg = "Dummy argument '%s' is not allowed in an equivalence set"_err_en_US;
  } else if (symbol.IsFuncResult()) {
    msg = "Function result '%s' is not allow in an equivalence set"_err_en_US;
  } else if (IsPointer(symbol)) {
    msg = "Pointer '%s' is not allowed in an equivalence set"_err_en_US;
  } else if (IsAllocatable(symbol)) {
    msg =
        "Allocatable variable '%s' is not allowed in an equivalence set"_err_en_US;
  } else if (symbol.Corank() > 0) {
    msg = "Coarray '%s' is not allowed in an equivalence set"_err_en_US;
  } else if (symbol.has<UseDetails>()) {
    msg =
        "Use-associated variable '%s' is not allowed in an equivalence set"_err_en_US;
  } else if (symbol.attrs().test(Attr::BIND_C)) {
    msg =
````
- **L2929 EN**: Closes the current lexical scope or compound statement.
  **L2929 CN**: 结束当前词法作用域或复合语句块。
- **L2930 EN**: Blank line separating nearby declarations or logic blocks.
  **L2930 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2931 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckEquivalenceObject(const EquivalenceObject &object) {`.
  **L2931 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckEquivalenceObject(const EquivalenceObject &object) {`。
- **L2932 EN**: Executes a standalone statement or declaration: `parser::MessageFixedText msg;`.
  **L2932 CN**: 执行一条独立语句或声明：`parser::MessageFixedText msg;`。
- **L2933 EN**: Executes a standalone statement or declaration: `const Symbol &symbol{object.symbol};`.
  **L2933 CN**: 执行一条独立语句或声明：`const Symbol &symbol{object.symbol};`。
- **L2934 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2934 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2935 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2935 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2936 EN**: Executes a standalone statement or declaration: `"Derived type component '%s' is not allowed in an equivalence set"_err_en_US;`.
  **L2936 CN**: 执行一条独立语句或声明：`"Derived type component '%s' is not allowed in an equivalence set"_err_en_US;`。
- **L2937 EN**: Transitions from the previous branch into an `else if` condition.
  **L2937 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2938 EN**: Executes a standalone statement or declaration: `msg = "Dummy argument '%s' is not allowed in an equivalence set"_err_en_US;`.
  **L2938 CN**: 执行一条独立语句或声明：`msg = "Dummy argument '%s' is not allowed in an equivalence set"_err_en_US;`。
- **L2939 EN**: Transitions from the previous branch into an `else if` condition.
  **L2939 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2940 EN**: Executes a standalone statement or declaration: `msg = "Function result '%s' is not allow in an equivalence set"_err_en_US;`.
  **L2940 CN**: 执行一条独立语句或声明：`msg = "Function result '%s' is not allow in an equivalence set"_err_en_US;`。
- **L2941 EN**: Transitions from the previous branch into an `else if` condition.
  **L2941 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2942 EN**: Executes a standalone statement or declaration: `msg = "Pointer '%s' is not allowed in an equivalence set"_err_en_US;`.
  **L2942 CN**: 执行一条独立语句或声明：`msg = "Pointer '%s' is not allowed in an equivalence set"_err_en_US;`。
- **L2943 EN**: Transitions from the previous branch into an `else if` condition.
  **L2943 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2944 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2944 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2945 EN**: Executes a standalone statement or declaration: `"Allocatable variable '%s' is not allowed in an equivalence set"_err_en_US;`.
  **L2945 CN**: 执行一条独立语句或声明：`"Allocatable variable '%s' is not allowed in an equivalence set"_err_en_US;`。
- **L2946 EN**: Transitions from the previous branch into an `else if` condition.
  **L2946 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2947 EN**: Executes a standalone statement or declaration: `msg = "Coarray '%s' is not allowed in an equivalence set"_err_en_US;`.
  **L2947 CN**: 执行一条独立语句或声明：`msg = "Coarray '%s' is not allowed in an equivalence set"_err_en_US;`。
- **L2948 EN**: Transitions from the previous branch into an `else if` condition.
  **L2948 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2949 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2949 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2950 EN**: Executes a standalone statement or declaration: `"Use-associated variable '%s' is not allowed in an equivalence set"_err_en_US;`.
  **L2950 CN**: 执行一条独立语句或声明：`"Use-associated variable '%s' is not allowed in an equivalence set"_err_en_US;`。
- **L2951 EN**: Transitions from the previous branch into an `else if` condition.
  **L2951 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2952 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2952 CN**: 继续构造周围的表达式或声明：`msg =`。

### Lines 2953-2976

````cpp
        "Variable '%s' with BIND attribute is not allowed in an equivalence set"_err_en_US;
  } else if (symbol.attrs().test(Attr::TARGET)) {
    msg =
        "Variable '%s' with TARGET attribute is not allowed in an equivalence set"_err_en_US;
  } else if (IsNamedConstant(symbol)) {
    msg = "Named constant '%s' is not allowed in an equivalence set"_err_en_US;
  } else if (InCommonWithBind(symbol)) {
    msg =
        "Variable '%s' in common block with BIND attribute is not allowed in an equivalence set"_err_en_US;
  } else if (!symbol.has<ObjectEntityDetails>()) {
    msg = "'%s' in equivalence set is not a data object"_err_en_US;
  } else if (const auto *type{symbol.GetType()}) {
    const auto *derived{type->AsDerived()};
    if (derived && !derived->IsVectorType()) {
      if (const auto *comp{
              FindUltimateComponent(*derived, IsAllocatableOrPointer)}) {
        msg = IsPointer(*comp)
            ? "Derived type object '%s' with pointer ultimate component is not allowed in an equivalence set"_err_en_US
            : "Derived type object '%s' with allocatable ultimate component is not allowed in an equivalence set"_err_en_US;
      } else if (!derived->typeSymbol().get<DerivedTypeDetails>().sequence()) {
        msg =
            "Nonsequence derived type object '%s' is not allowed in an equivalence set"_err_en_US;
      }
    } else if (IsAutomatic(symbol)) {
````
- **L2953 EN**: Executes a standalone statement or declaration: `"Variable '%s' with BIND attribute is not allowed in an equivalence set"_err_en_US;`.
  **L2953 CN**: 执行一条独立语句或声明：`"Variable '%s' with BIND attribute is not allowed in an equivalence set"_err_en_US;`。
- **L2954 EN**: Transitions from the previous branch into an `else if` condition.
  **L2954 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2955 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2955 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2956 EN**: Executes a standalone statement or declaration: `"Variable '%s' with TARGET attribute is not allowed in an equivalence set"_err_en_US;`.
  **L2956 CN**: 执行一条独立语句或声明：`"Variable '%s' with TARGET attribute is not allowed in an equivalence set"_err_en_US;`。
- **L2957 EN**: Transitions from the previous branch into an `else if` condition.
  **L2957 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2958 EN**: Executes a standalone statement or declaration: `msg = "Named constant '%s' is not allowed in an equivalence set"_err_en_US;`.
  **L2958 CN**: 执行一条独立语句或声明：`msg = "Named constant '%s' is not allowed in an equivalence set"_err_en_US;`。
- **L2959 EN**: Transitions from the previous branch into an `else if` condition.
  **L2959 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2960 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2960 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2961 EN**: Executes a standalone statement or declaration: `"Variable '%s' in common block with BIND attribute is not allowed in an equivalence set"_err_en_US;`.
  **L2961 CN**: 执行一条独立语句或声明：`"Variable '%s' in common block with BIND attribute is not allowed in an equivalence set"_err_en_US;`。
- **L2962 EN**: Transitions from the previous branch into an `else if` condition.
  **L2962 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2963 EN**: Executes a standalone statement or declaration: `msg = "'%s' in equivalence set is not a data object"_err_en_US;`.
  **L2963 CN**: 执行一条独立语句或声明：`msg = "'%s' in equivalence set is not a data object"_err_en_US;`。
- **L2964 EN**: Transitions from the previous branch into an `else if` condition.
  **L2964 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2965 EN**: Executes a call or declaration centered on `*derived{type->AsDerived`.
  **L2965 CN**: 执行以 `*derived{type->AsDerived` 为核心的调用或声明。
- **L2966 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2966 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2967 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2967 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2968 EN**: Starts a function, method, lambda, or structured scope: `FindUltimateComponent(*derived, IsAllocatableOrPointer)}) {`.
  **L2968 CN**: 开始一个函数、方法、lambda 或结构化作用域：`FindUltimateComponent(*derived, IsAllocatableOrPointer)}) {`。
- **L2969 EN**: Continues logic associated with callable symbol `IsPointer`.
  **L2969 CN**: 继续与可调用符号 `IsPointer` 相关的逻辑。
- **L2970 EN**: Continues the surrounding expression or declaration: `? "Derived type object '%s' with pointer ultimate component is not allowed in an equivalence set"_err_en_US`.
  **L2970 CN**: 继续构造周围的表达式或声明：`? "Derived type object '%s' with pointer ultimate component is not allowed in an equivalence set"_err_en_US`。
- **L2971 EN**: Executes a standalone statement or declaration: `: "Derived type object '%s' with allocatable ultimate component is not allowed in an equivalence set"_err_en_US;`.
  **L2971 CN**: 执行一条独立语句或声明：`: "Derived type object '%s' with allocatable ultimate component is not allowed in an equivalence set"_err_en_US;`。
- **L2972 EN**: Transitions from the previous branch into an `else if` condition.
  **L2972 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2973 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2973 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2974 EN**: Executes a standalone statement or declaration: `"Nonsequence derived type object '%s' is not allowed in an equivalence set"_err_en_US;`.
  **L2974 CN**: 执行一条独立语句或声明：`"Nonsequence derived type object '%s' is not allowed in an equivalence set"_err_en_US;`。
- **L2975 EN**: Closes the current lexical scope or compound statement.
  **L2975 CN**: 结束当前词法作用域或复合语句块。
- **L2976 EN**: Transitions from the previous branch into an `else if` condition.
  **L2976 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 2977-3000

````cpp
      msg =
          "Automatic object '%s' is not allowed in an equivalence set"_err_en_US;
    } else if (symbol.test(Symbol::Flag::CrayPointee)) {
      messages_.Say(object.symbol.name(),
          "Cray pointee '%s' may not be a member of an EQUIVALENCE group"_err_en_US,
          object.symbol.name());
    }
  }
  if (!msg.text().empty()) {
    context_.Say(object.source, std::move(msg), symbol.name());
  }
}

void CheckHelper::CheckBlockData(const Scope &scope) {
  // BLOCK DATA subprograms should contain only named common blocks.
  // C1415 presents a list of statements that shouldn't appear in
  // BLOCK DATA, but so long as the subprogram contains no executable
  // code and allocates no storage outside named COMMON, we're happy
  // (e.g., an ENUM is strictly not allowed).
  for (const auto &pair : scope) {
    const Symbol &symbol{*pair.second};
    if (!(symbol.has<CommonBlockDetails>() || symbol.has<UseDetails>() ||
            symbol.has<UseErrorDetails>() || symbol.has<DerivedTypeDetails>() ||
            symbol.has<SubprogramDetails>() ||
````
- **L2977 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L2977 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L2978 EN**: Executes a standalone statement or declaration: `"Automatic object '%s' is not allowed in an equivalence set"_err_en_US;`.
  **L2978 CN**: 执行一条独立语句或声明：`"Automatic object '%s' is not allowed in an equivalence set"_err_en_US;`。
- **L2979 EN**: Transitions from the previous branch into an `else if` condition.
  **L2979 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2980 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(object.symbol.name(),`.
  **L2980 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(object.symbol.name(),`。
- **L2981 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Cray pointee '%s' may not be a member of an EQUIVALENCE group"_err_en_US,`.
  **L2981 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Cray pointee '%s' may not be a member of an EQUIVALENCE group"_err_en_US,`。
- **L2982 EN**: Executes a call or declaration centered on `object.symbol.name`.
  **L2982 CN**: 执行以 `object.symbol.name` 为核心的调用或声明。
- **L2983 EN**: Closes the current lexical scope or compound statement.
  **L2983 CN**: 结束当前词法作用域或复合语句块。
- **L2984 EN**: Closes the current lexical scope or compound statement.
  **L2984 CN**: 结束当前词法作用域或复合语句块。
- **L2985 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2985 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2986 EN**: Executes a call or declaration centered on `context_.Say`.
  **L2986 CN**: 执行以 `context_.Say` 为核心的调用或声明。
- **L2987 EN**: Closes the current lexical scope or compound statement.
  **L2987 CN**: 结束当前词法作用域或复合语句块。
- **L2988 EN**: Closes the current lexical scope or compound statement.
  **L2988 CN**: 结束当前词法作用域或复合语句块。
- **L2989 EN**: Blank line separating nearby declarations or logic blocks.
  **L2989 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2990 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckBlockData(const Scope &scope) {`.
  **L2990 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckBlockData(const Scope &scope) {`。
- **L2991 EN**: Comment explains nearby logic, intent, or metadata: `BLOCK DATA subprograms should contain only named common blocks.`.
  **L2991 CN**: 注释说明附近代码的逻辑、意图或元数据：`BLOCK DATA subprograms should contain only named common blocks.`。
- **L2992 EN**: Comment explains nearby logic, intent, or metadata: `C1415 presents a list of statements that shouldn't appear in`.
  **L2992 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1415 presents a list of statements that shouldn't appear in`。
- **L2993 EN**: Comment explains nearby logic, intent, or metadata: `BLOCK DATA, but so long as the subprogram contains no executable`.
  **L2993 CN**: 注释说明附近代码的逻辑、意图或元数据：`BLOCK DATA, but so long as the subprogram contains no executable`。
- **L2994 EN**: Comment explains nearby logic, intent, or metadata: `code and allocates no storage outside named COMMON, we're happy`.
  **L2994 CN**: 注释说明附近代码的逻辑、意图或元数据：`code and allocates no storage outside named COMMON, we're happy`。
- **L2995 EN**: Comment explains nearby logic, intent, or metadata: `(e.g., an ENUM is strictly not allowed).`.
  **L2995 CN**: 注释说明附近代码的逻辑、意图或元数据：`(e.g., an ENUM is strictly not allowed).`。
- **L2996 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2996 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2997 EN**: Executes a standalone statement or declaration: `const Symbol &symbol{*pair.second};`.
  **L2997 CN**: 执行一条独立语句或声明：`const Symbol &symbol{*pair.second};`。
- **L2998 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2998 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2999 EN**: Continues logic associated with callable symbol `has<UseErrorDetails>`.
  **L2999 CN**: 继续与可调用符号 `has<UseErrorDetails>` 相关的逻辑。
- **L3000 EN**: Continues logic associated with callable symbol `has<SubprogramDetails>`.
  **L3000 CN**: 继续与可调用符号 `has<SubprogramDetails>` 相关的逻辑。

### Lines 3001-3024

````cpp
            symbol.has<ObjectEntityDetails>() ||
            (symbol.has<ProcEntityDetails>() &&
                !symbol.attrs().test(Attr::POINTER)))) {
      messages_.Say(symbol.name(),
          "'%s' may not appear in a BLOCK DATA subprogram"_err_en_US,
          symbol.name());
    }
  }
}

// Check distinguishability of generic assignment and operators.
// For these, generics and generic bindings must be considered together.
void CheckHelper::CheckGenericOps(const Scope &scope) {
  DistinguishabilityHelper helper{context_};
  auto addSpecifics{[&](const Symbol &generic) {
    if (!IsAccessible(generic, scope)) {
      return;
    }
    const auto *details{generic.GetUltimate().detailsIf<GenericDetails>()};
    if (!details) {
      // Not a generic; ensure characteristics are defined if a function.
      auto restorer{messages_.SetLocation(generic.name())};
      if (IsFunction(generic) && !context_.HasError(generic)) {
        if (const Symbol *result{FindFunctionResult(generic)};
````
- **L3001 EN**: Continues logic associated with callable symbol `has<ObjectEntityDetails>`.
  **L3001 CN**: 继续与可调用符号 `has<ObjectEntityDetails>` 相关的逻辑。
- **L3002 EN**: Continues logic associated with callable symbol `has<ProcEntityDetails>`.
  **L3002 CN**: 继续与可调用符号 `has<ProcEntityDetails>` 相关的逻辑。
- **L3003 EN**: Starts a function, method, lambda, or structured scope: `!symbol.attrs().test(Attr::POINTER)))) {`.
  **L3003 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!symbol.attrs().test(Attr::POINTER)))) {`。
- **L3004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(symbol.name(),`.
  **L3004 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(symbol.name(),`。
- **L3005 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' may not appear in a BLOCK DATA subprogram"_err_en_US,`.
  **L3005 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' may not appear in a BLOCK DATA subprogram"_err_en_US,`。
- **L3006 EN**: Executes a call or declaration centered on `symbol.name`.
  **L3006 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L3007 EN**: Closes the current lexical scope or compound statement.
  **L3007 CN**: 结束当前词法作用域或复合语句块。
- **L3008 EN**: Closes the current lexical scope or compound statement.
  **L3008 CN**: 结束当前词法作用域或复合语句块。
- **L3009 EN**: Closes the current lexical scope or compound statement.
  **L3009 CN**: 结束当前词法作用域或复合语句块。
- **L3010 EN**: Blank line separating nearby declarations or logic blocks.
  **L3010 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3011 EN**: Comment explains nearby logic, intent, or metadata: `Check distinguishability of generic assignment and operators.`.
  **L3011 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check distinguishability of generic assignment and operators.`。
- **L3012 EN**: Comment explains nearby logic, intent, or metadata: `For these, generics and generic bindings must be considered together.`.
  **L3012 CN**: 注释说明附近代码的逻辑、意图或元数据：`For these, generics and generic bindings must be considered together.`。
- **L3013 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckGenericOps(const Scope &scope) {`.
  **L3013 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckGenericOps(const Scope &scope) {`。
- **L3014 EN**: Executes a standalone statement or declaration: `DistinguishabilityHelper helper{context_};`.
  **L3014 CN**: 执行一条独立语句或声明：`DistinguishabilityHelper helper{context_};`。
- **L3015 EN**: Starts a function, method, lambda, or structured scope: `auto addSpecifics{[&](const Symbol &generic) {`.
  **L3015 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto addSpecifics{[&](const Symbol &generic) {`。
- **L3016 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3016 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3017 EN**: Returns from the current function with `void`.
  **L3017 CN**: 以 `void` 从当前函数返回。
- **L3018 EN**: Closes the current lexical scope or compound statement.
  **L3018 CN**: 结束当前词法作用域或复合语句块。
- **L3019 EN**: Executes a call or declaration centered on `*details{generic.GetUltimate`.
  **L3019 CN**: 执行以 `*details{generic.GetUltimate` 为核心的调用或声明。
- **L3020 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3020 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3021 EN**: Comment explains nearby logic, intent, or metadata: `Not a generic; ensure characteristics are defined if a function.`.
  **L3021 CN**: 注释说明附近代码的逻辑、意图或元数据：`Not a generic; ensure characteristics are defined if a function.`。
- **L3022 EN**: Executes a call or declaration centered on `restorer{messages_.SetLocation`.
  **L3022 CN**: 执行以 `restorer{messages_.SetLocation` 为核心的调用或声明。
- **L3023 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3023 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3024 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3024 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3025-3048

````cpp
            result && !context_.HasError(*result)) {
          Characterize(generic);
        }
      }
      return;
    }
    GenericKind kind{details->kind()};
    if (!kind.IsAssignment() && !kind.IsOperator()) {
      return;
    }
    const SymbolVector &specifics{details->specificProcs()};
    const std::vector<SourceName> &bindingNames{details->bindingNames()};
    for (std::size_t i{0}; i < specifics.size(); ++i) {
      const Symbol &specific{*specifics[i]};
      auto restorer{messages_.SetLocation(bindingNames[i])};
      if (const Procedure *proc{Characterize(specific)}) {
        if (kind.IsAssignment()) {
          if (!CheckDefinedAssignment(specific, *proc)) {
            continue;
          }
        } else {
          if (!CheckDefinedOperator(generic.name(), kind, specific, *proc)) {
            continue;
          }
````
- **L3025 EN**: Starts a function, method, lambda, or structured scope: `result && !context_.HasError(*result)) {`.
  **L3025 CN**: 开始一个函数、方法、lambda 或结构化作用域：`result && !context_.HasError(*result)) {`。
- **L3026 EN**: Executes a call or declaration centered on `Characterize`.
  **L3026 CN**: 执行以 `Characterize` 为核心的调用或声明。
- **L3027 EN**: Closes the current lexical scope or compound statement.
  **L3027 CN**: 结束当前词法作用域或复合语句块。
- **L3028 EN**: Closes the current lexical scope or compound statement.
  **L3028 CN**: 结束当前词法作用域或复合语句块。
- **L3029 EN**: Returns from the current function with `void`.
  **L3029 CN**: 以 `void` 从当前函数返回。
- **L3030 EN**: Closes the current lexical scope or compound statement.
  **L3030 CN**: 结束当前词法作用域或复合语句块。
- **L3031 EN**: Executes a call or declaration centered on `kind{details->kind`.
  **L3031 CN**: 执行以 `kind{details->kind` 为核心的调用或声明。
- **L3032 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3032 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3033 EN**: Returns from the current function with `void`.
  **L3033 CN**: 以 `void` 从当前函数返回。
- **L3034 EN**: Closes the current lexical scope or compound statement.
  **L3034 CN**: 结束当前词法作用域或复合语句块。
- **L3035 EN**: Executes a call or declaration centered on `&specifics{details->specificProcs`.
  **L3035 CN**: 执行以 `&specifics{details->specificProcs` 为核心的调用或声明。
- **L3036 EN**: Executes a call or declaration centered on `&bindingNames{details->bindingNames`.
  **L3036 CN**: 执行以 `&bindingNames{details->bindingNames` 为核心的调用或声明。
- **L3037 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3037 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3038 EN**: Executes a standalone statement or declaration: `const Symbol &specific{*specifics[i]};`.
  **L3038 CN**: 执行一条独立语句或声明：`const Symbol &specific{*specifics[i]};`。
- **L3039 EN**: Executes a call or declaration centered on `restorer{messages_.SetLocation`.
  **L3039 CN**: 执行以 `restorer{messages_.SetLocation` 为核心的调用或声明。
- **L3040 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3040 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3041 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3041 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3042 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3042 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3043 EN**: Skips to the next loop iteration.
  **L3043 CN**: 跳到下一次循环迭代。
- **L3044 EN**: Closes the current lexical scope or compound statement.
  **L3044 CN**: 结束当前词法作用域或复合语句块。
- **L3045 EN**: Transitions from the previous branch into the alternative path.
  **L3045 CN**: 从前一个分支过渡到备选路径。
- **L3046 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3046 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3047 EN**: Skips to the next loop iteration.
  **L3047 CN**: 跳到下一次循环迭代。
- **L3048 EN**: Closes the current lexical scope or compound statement.
  **L3048 CN**: 结束当前词法作用域或复合语句块。

### Lines 3049-3072

````cpp
        }
        helper.Add(generic, kind, specific, *proc);
      }
    }
  }};
  for (const auto &pair : scope) {
    const Symbol &symbol{*pair.second};
    addSpecifics(symbol);
    const Symbol &ultimate{symbol.GetUltimate()};
    if (ultimate.has<DerivedTypeDetails>()) {
      if (const Scope *typeScope{ultimate.scope()}) {
        for (const auto &pair2 : *typeScope) {
          addSpecifics(*pair2.second);
        }
      }
    }
  }
  helper.Check(scope);
}

static bool IsSubprogramDefinition(const Symbol &symbol) {
  const auto *subp{symbol.detailsIf<SubprogramDetails>()};
  return subp && !subp->isInterface() && symbol.scope() &&
      symbol.scope()->kind() == Scope::Kind::Subprogram;
````
- **L3049 EN**: Closes the current lexical scope or compound statement.
  **L3049 CN**: 结束当前词法作用域或复合语句块。
- **L3050 EN**: Executes a call or declaration centered on `helper.Add`.
  **L3050 CN**: 执行以 `helper.Add` 为核心的调用或声明。
- **L3051 EN**: Closes the current lexical scope or compound statement.
  **L3051 CN**: 结束当前词法作用域或复合语句块。
- **L3052 EN**: Closes the current lexical scope or compound statement.
  **L3052 CN**: 结束当前词法作用域或复合语句块。
- **L3053 EN**: Executes a standalone statement or declaration: `}};`.
  **L3053 CN**: 执行一条独立语句或声明：`}};`。
- **L3054 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3054 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3055 EN**: Executes a standalone statement or declaration: `const Symbol &symbol{*pair.second};`.
  **L3055 CN**: 执行一条独立语句或声明：`const Symbol &symbol{*pair.second};`。
- **L3056 EN**: Executes a call or declaration centered on `addSpecifics`.
  **L3056 CN**: 执行以 `addSpecifics` 为核心的调用或声明。
- **L3057 EN**: Executes a call or declaration centered on `&ultimate{symbol.GetUltimate`.
  **L3057 CN**: 执行以 `&ultimate{symbol.GetUltimate` 为核心的调用或声明。
- **L3058 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3058 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3059 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3059 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3060 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3060 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3061 EN**: Executes a call or declaration centered on `addSpecifics`.
  **L3061 CN**: 执行以 `addSpecifics` 为核心的调用或声明。
- **L3062 EN**: Closes the current lexical scope or compound statement.
  **L3062 CN**: 结束当前词法作用域或复合语句块。
- **L3063 EN**: Closes the current lexical scope or compound statement.
  **L3063 CN**: 结束当前词法作用域或复合语句块。
- **L3064 EN**: Closes the current lexical scope or compound statement.
  **L3064 CN**: 结束当前词法作用域或复合语句块。
- **L3065 EN**: Closes the current lexical scope or compound statement.
  **L3065 CN**: 结束当前词法作用域或复合语句块。
- **L3066 EN**: Executes a call or declaration centered on `helper.Check`.
  **L3066 CN**: 执行以 `helper.Check` 为核心的调用或声明。
- **L3067 EN**: Closes the current lexical scope or compound statement.
  **L3067 CN**: 结束当前词法作用域或复合语句块。
- **L3068 EN**: Blank line separating nearby declarations or logic blocks.
  **L3068 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3069 EN**: Starts a function, method, lambda, or structured scope: `static bool IsSubprogramDefinition(const Symbol &symbol) {`.
  **L3069 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool IsSubprogramDefinition(const Symbol &symbol) {`。
- **L3070 EN**: Executes a call or declaration centered on `*subp{symbol.detailsIf<SubprogramDetails>`.
  **L3070 CN**: 执行以 `*subp{symbol.detailsIf<SubprogramDetails>` 为核心的调用或声明。
- **L3071 EN**: Returns from the current function with `subp && !subp->isInterface() && symbol.scope() &&`.
  **L3071 CN**: 以 `subp && !subp->isInterface() && symbol.scope() &&` 从当前函数返回。
- **L3072 EN**: Executes a call or declaration centered on `symbol.scope`.
  **L3072 CN**: 执行以 `symbol.scope` 为核心的调用或声明。

### Lines 3073-3096

````cpp
}

static bool IsExternalProcedureDefinition(const Symbol &symbol) {
  return IsBlockData(symbol) ||
      ((IsSubprogramDefinition(symbol) || IsAlternateEntry(&symbol)) &&
          (IsExternal(symbol) || symbol.GetBindName()));
}

static std::optional<std::string> DefinesGlobalName(const Symbol &symbol) {
  if (const auto *module{symbol.detailsIf<ModuleDetails>()}) {
    if (!module->isSubmodule() && !symbol.owner().IsIntrinsicModules()) {
      return symbol.name().ToString();
    }
  } else if (IsBlockData(symbol)) {
    return symbol.name().ToString();
  } else {
    const std::string *bindC{symbol.GetBindName()};
    if (symbol.has<CommonBlockDetails>() ||
        IsExternalProcedureDefinition(symbol) ||
        (symbol.owner().IsGlobal() && IsExternal(symbol))) {
      return bindC ? *bindC : symbol.name().ToString();
    } else if (bindC &&
        (symbol.has<ObjectEntityDetails>() || IsModuleProcedure(symbol))) {
      return *bindC;
````
- **L3073 EN**: Closes the current lexical scope or compound statement.
  **L3073 CN**: 结束当前词法作用域或复合语句块。
- **L3074 EN**: Blank line separating nearby declarations or logic blocks.
  **L3074 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3075 EN**: Starts a function, method, lambda, or structured scope: `static bool IsExternalProcedureDefinition(const Symbol &symbol) {`.
  **L3075 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool IsExternalProcedureDefinition(const Symbol &symbol) {`。
- **L3076 EN**: Returns from the current function with `IsBlockData(symbol) ||`.
  **L3076 CN**: 以 `IsBlockData(symbol) ||` 从当前函数返回。
- **L3077 EN**: Continues logic associated with callable symbol `IsSubprogramDefinition`.
  **L3077 CN**: 继续与可调用符号 `IsSubprogramDefinition` 相关的逻辑。
- **L3078 EN**: Executes a call or declaration centered on `statement`.
  **L3078 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3079 EN**: Closes the current lexical scope or compound statement.
  **L3079 CN**: 结束当前词法作用域或复合语句块。
- **L3080 EN**: Blank line separating nearby declarations or logic blocks.
  **L3080 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3081 EN**: Starts a function, method, lambda, or structured scope: `static std::optional<std::string> DefinesGlobalName(const Symbol &symbol) {`.
  **L3081 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static std::optional<std::string> DefinesGlobalName(const Symbol &symbol) {`。
- **L3082 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3082 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3083 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3083 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3084 EN**: Returns from the current function with `symbol.name().ToString()`.
  **L3084 CN**: 以 `symbol.name().ToString()` 从当前函数返回。
- **L3085 EN**: Closes the current lexical scope or compound statement.
  **L3085 CN**: 结束当前词法作用域或复合语句块。
- **L3086 EN**: Transitions from the previous branch into an `else if` condition.
  **L3086 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3087 EN**: Returns from the current function with `symbol.name().ToString()`.
  **L3087 CN**: 以 `symbol.name().ToString()` 从当前函数返回。
- **L3088 EN**: Transitions from the previous branch into the alternative path.
  **L3088 CN**: 从前一个分支过渡到备选路径。
- **L3089 EN**: Executes a call or declaration centered on `*bindC{symbol.GetBindName`.
  **L3089 CN**: 执行以 `*bindC{symbol.GetBindName` 为核心的调用或声明。
- **L3090 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3090 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3091 EN**: Continues logic associated with callable symbol `IsExternalProcedureDefinition`.
  **L3091 CN**: 继续与可调用符号 `IsExternalProcedureDefinition` 相关的逻辑。
- **L3092 EN**: Starts a function, method, lambda, or structured scope: `(symbol.owner().IsGlobal() && IsExternal(symbol))) {`.
  **L3092 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(symbol.owner().IsGlobal() && IsExternal(symbol))) {`。
- **L3093 EN**: Returns from the current function with `bindC ? *bindC : symbol.name().ToString()`.
  **L3093 CN**: 以 `bindC ? *bindC : symbol.name().ToString()` 从当前函数返回。
- **L3094 EN**: Transitions from the previous branch into an `else if` condition.
  **L3094 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3095 EN**: Starts a function, method, lambda, or structured scope: `(symbol.has<ObjectEntityDetails>() || IsModuleProcedure(symbol))) {`.
  **L3095 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(symbol.has<ObjectEntityDetails>() || IsModuleProcedure(symbol))) {`。
- **L3096 EN**: Returns from the current function with `*bindC`.
  **L3096 CN**: 以 `*bindC` 从当前函数返回。

### Lines 3097-3120

````cpp
    }
  }
  return std::nullopt;
}

// 19.2 p2
void CheckHelper::CheckGlobalName(const Symbol &symbol) {
  if (auto global{DefinesGlobalName(symbol)}) {
    auto pair{globalNames_.emplace(std::move(*global), symbol)};
    if (!pair.second) {
      const Symbol &other{*pair.first->second};
      if (context_.HasError(symbol) || context_.HasError(other)) {
        // don't pile on
      } else if (symbol.has<CommonBlockDetails>() &&
          other.has<CommonBlockDetails>() && symbol.name() == other.name()) {
        // Two common blocks can have the same global name so long as
        // they're not in the same scope.
      } else if ((IsProcedure(symbol) || IsBlockData(symbol)) &&
          (IsProcedure(other) || IsBlockData(other)) &&
          (!IsExternalProcedureDefinition(symbol) ||
              !IsExternalProcedureDefinition(other))) {
        // both are procedures/BLOCK DATA, not both definitions
      } else if (AreSameModuleSymbol(symbol, other)) {
        // Both symbols are the same thing.
````
- **L3097 EN**: Closes the current lexical scope or compound statement.
  **L3097 CN**: 结束当前词法作用域或复合语句块。
- **L3098 EN**: Closes the current lexical scope or compound statement.
  **L3098 CN**: 结束当前词法作用域或复合语句块。
- **L3099 EN**: Returns from the current function with `std::nullopt`.
  **L3099 CN**: 以 `std::nullopt` 从当前函数返回。
- **L3100 EN**: Closes the current lexical scope or compound statement.
  **L3100 CN**: 结束当前词法作用域或复合语句块。
- **L3101 EN**: Blank line separating nearby declarations or logic blocks.
  **L3101 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3102 EN**: Comment explains nearby logic, intent, or metadata: `19.2 p2`.
  **L3102 CN**: 注释说明附近代码的逻辑、意图或元数据：`19.2 p2`。
- **L3103 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckGlobalName(const Symbol &symbol) {`.
  **L3103 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckGlobalName(const Symbol &symbol) {`。
- **L3104 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3104 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3105 EN**: Executes a call or declaration centered on `pair{globalNames_.emplace`.
  **L3105 CN**: 执行以 `pair{globalNames_.emplace` 为核心的调用或声明。
- **L3106 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3106 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3107 EN**: Executes a standalone statement or declaration: `const Symbol &other{*pair.first->second};`.
  **L3107 CN**: 执行一条独立语句或声明：`const Symbol &other{*pair.first->second};`。
- **L3108 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3108 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3109 EN**: Comment explains nearby logic, intent, or metadata: `don't pile on`.
  **L3109 CN**: 注释说明附近代码的逻辑、意图或元数据：`don't pile on`。
- **L3110 EN**: Transitions from the previous branch into an `else if` condition.
  **L3110 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3111 EN**: Starts a function, method, lambda, or structured scope: `other.has<CommonBlockDetails>() && symbol.name() == other.name()) {`.
  **L3111 CN**: 开始一个函数、方法、lambda 或结构化作用域：`other.has<CommonBlockDetails>() && symbol.name() == other.name()) {`。
- **L3112 EN**: Comment explains nearby logic, intent, or metadata: `Two common blocks can have the same global name so long as`.
  **L3112 CN**: 注释说明附近代码的逻辑、意图或元数据：`Two common blocks can have the same global name so long as`。
- **L3113 EN**: Comment explains nearby logic, intent, or metadata: `they're not in the same scope.`.
  **L3113 CN**: 注释说明附近代码的逻辑、意图或元数据：`they're not in the same scope.`。
- **L3114 EN**: Transitions from the previous branch into an `else if` condition.
  **L3114 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3115 EN**: Continues logic associated with callable symbol `IsProcedure`.
  **L3115 CN**: 继续与可调用符号 `IsProcedure` 相关的逻辑。
- **L3116 EN**: Continues logic associated with callable symbol `IsExternalProcedureDefinition`.
  **L3116 CN**: 继续与可调用符号 `IsExternalProcedureDefinition` 相关的逻辑。
- **L3117 EN**: Starts a function, method, lambda, or structured scope: `!IsExternalProcedureDefinition(other))) {`.
  **L3117 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!IsExternalProcedureDefinition(other))) {`。
- **L3118 EN**: Comment explains nearby logic, intent, or metadata: `both are procedures/BLOCK DATA, not both definitions`.
  **L3118 CN**: 注释说明附近代码的逻辑、意图或元数据：`both are procedures/BLOCK DATA, not both definitions`。
- **L3119 EN**: Transitions from the previous branch into an `else if` condition.
  **L3119 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3120 EN**: Comment explains nearby logic, intent, or metadata: `Both symbols are the same thing.`.
  **L3120 CN**: 注释说明附近代码的逻辑、意图或元数据：`Both symbols are the same thing.`。

### Lines 3121-3144

````cpp
      } else if (symbol.has<ModuleDetails>()) {
        Warn(common::LanguageFeature::BenignNameClash, symbol.name(),
            "Module '%s' conflicts with a global name"_port_en_US,
            pair.first->first);
      } else if (other.has<ModuleDetails>()) {
        Warn(common::LanguageFeature::BenignNameClash, symbol.name(),
            "Global name '%s' conflicts with a module"_port_en_US,
            pair.first->first);
      } else if (auto *msg{messages_.Say(symbol.name(),
                     "Two entities have the same global name '%s'"_err_en_US,
                     pair.first->first)}) {
        msg->Attach(other.name(), "Conflicting declaration"_en_US);
        context_.SetError(symbol);
        context_.SetError(other);
      }
    }
  }
}

void CheckHelper::CheckProcedureAssemblyName(const Symbol &symbol) {
  if (!IsProcedure(symbol) || symbol != symbol.GetUltimate())
    return;
  const std::string *bindName{symbol.GetBindName()};
  const bool hasExplicitBindingLabel{
````
- **L3121 EN**: Transitions from the previous branch into an `else if` condition.
  **L3121 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::LanguageFeature::BenignNameClash, symbol.name(),`.
  **L3122 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::LanguageFeature::BenignNameClash, symbol.name(),`。
- **L3123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Module '%s' conflicts with a global name"_port_en_US,`.
  **L3123 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Module '%s' conflicts with a global name"_port_en_US,`。
- **L3124 EN**: Executes a standalone statement or declaration: `pair.first->first);`.
  **L3124 CN**: 执行一条独立语句或声明：`pair.first->first);`。
- **L3125 EN**: Transitions from the previous branch into an `else if` condition.
  **L3125 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::LanguageFeature::BenignNameClash, symbol.name(),`.
  **L3126 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::LanguageFeature::BenignNameClash, symbol.name(),`。
- **L3127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Global name '%s' conflicts with a module"_port_en_US,`.
  **L3127 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Global name '%s' conflicts with a module"_port_en_US,`。
- **L3128 EN**: Executes a standalone statement or declaration: `pair.first->first);`.
  **L3128 CN**: 执行一条独立语句或声明：`pair.first->first);`。
- **L3129 EN**: Transitions from the previous branch into an `else if` condition.
  **L3129 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Two entities have the same global name '%s'"_err_en_US,`.
  **L3130 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Two entities have the same global name '%s'"_err_en_US,`。
- **L3131 EN**: Continues the surrounding expression or declaration: `pair.first->first)}) {`.
  **L3131 CN**: 继续构造周围的表达式或声明：`pair.first->first)}) {`。
- **L3132 EN**: Executes a call or declaration centered on `msg->Attach`.
  **L3132 CN**: 执行以 `msg->Attach` 为核心的调用或声明。
- **L3133 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L3133 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L3134 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L3134 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L3135 EN**: Closes the current lexical scope or compound statement.
  **L3135 CN**: 结束当前词法作用域或复合语句块。
- **L3136 EN**: Closes the current lexical scope or compound statement.
  **L3136 CN**: 结束当前词法作用域或复合语句块。
- **L3137 EN**: Closes the current lexical scope or compound statement.
  **L3137 CN**: 结束当前词法作用域或复合语句块。
- **L3138 EN**: Closes the current lexical scope or compound statement.
  **L3138 CN**: 结束当前词法作用域或复合语句块。
- **L3139 EN**: Blank line separating nearby declarations or logic blocks.
  **L3139 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3140 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckProcedureAssemblyName(const Symbol &symbol) {`.
  **L3140 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckProcedureAssemblyName(const Symbol &symbol) {`。
- **L3141 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3141 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3142 EN**: Returns from the current function with `void`.
  **L3142 CN**: 以 `void` 从当前函数返回。
- **L3143 EN**: Executes a call or declaration centered on `*bindName{symbol.GetBindName`.
  **L3143 CN**: 执行以 `*bindName{symbol.GetBindName` 为核心的调用或声明。
- **L3144 EN**: Continues the surrounding expression or declaration: `const bool hasExplicitBindingLabel{`.
  **L3144 CN**: 继续构造周围的表达式或声明：`const bool hasExplicitBindingLabel{`。

### Lines 3145-3168

````cpp
      symbol.GetIsExplicitBindName() && bindName};
  if (hasExplicitBindingLabel || IsExternal(symbol)) {
    const std::string assemblyName{hasExplicitBindingLabel
            ? *bindName
            : common::GetExternalAssemblyName(
                  symbol.name().ToString(), context_.underscoring())};
    auto pair{procedureAssemblyNames_.emplace(std::move(assemblyName), symbol)};
    if (!pair.second) {
      const Symbol &other{*pair.first->second};
      const bool otherHasExplicitBindingLabel{
          other.GetIsExplicitBindName() && other.GetBindName()};
      if (otherHasExplicitBindingLabel != hasExplicitBindingLabel) {
        // The BIND(C,NAME="...") binding label is the same as the name that
        // will be used in LLVM IR for an external procedure declared without
        // BIND(C) in the same file. While this is not forbidden by the
        // standard, this name collision would lead to a crash when producing
        // the IR.
        if (auto *msg{messages_.Say(symbol.name(),
                "%s procedure assembly name conflicts with %s procedure assembly name"_err_en_US,
                hasExplicitBindingLabel ? "BIND(C)" : "Non BIND(C)",
                hasExplicitBindingLabel ? "non BIND(C)" : "BIND(C)")}) {
          msg->Attach(other.name(), "Conflicting declaration"_en_US);
        }
        context_.SetError(symbol);
````
- **L3145 EN**: Executes a call or declaration centered on `symbol.GetIsExplicitBindName`.
  **L3145 CN**: 执行以 `symbol.GetIsExplicitBindName` 为核心的调用或声明。
- **L3146 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3146 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3147 EN**: Continues the surrounding expression or declaration: `const std::string assemblyName{hasExplicitBindingLabel`.
  **L3147 CN**: 继续构造周围的表达式或声明：`const std::string assemblyName{hasExplicitBindingLabel`。
- **L3148 EN**: Continues the surrounding expression or declaration: `? *bindName`.
  **L3148 CN**: 继续构造周围的表达式或声明：`? *bindName`。
- **L3149 EN**: Continues logic associated with callable symbol `GetExternalAssemblyName`.
  **L3149 CN**: 继续与可调用符号 `GetExternalAssemblyName` 相关的逻辑。
- **L3150 EN**: Executes a call or declaration centered on `symbol.name`.
  **L3150 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L3151 EN**: Executes a call or declaration centered on `pair{procedureAssemblyNames_.emplace`.
  **L3151 CN**: 执行以 `pair{procedureAssemblyNames_.emplace` 为核心的调用或声明。
- **L3152 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3152 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3153 EN**: Executes a standalone statement or declaration: `const Symbol &other{*pair.first->second};`.
  **L3153 CN**: 执行一条独立语句或声明：`const Symbol &other{*pair.first->second};`。
- **L3154 EN**: Continues the surrounding expression or declaration: `const bool otherHasExplicitBindingLabel{`.
  **L3154 CN**: 继续构造周围的表达式或声明：`const bool otherHasExplicitBindingLabel{`。
- **L3155 EN**: Executes a call or declaration centered on `other.GetIsExplicitBindName`.
  **L3155 CN**: 执行以 `other.GetIsExplicitBindName` 为核心的调用或声明。
- **L3156 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3156 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3157 EN**: Comment explains nearby logic, intent, or metadata: `The BIND(C,NAME="...") binding label is the same as the name that`.
  **L3157 CN**: 注释说明附近代码的逻辑、意图或元数据：`The BIND(C,NAME="...") binding label is the same as the name that`。
- **L3158 EN**: Comment explains nearby logic, intent, or metadata: `will be used in LLVM IR for an external procedure declared without`.
  **L3158 CN**: 注释说明附近代码的逻辑、意图或元数据：`will be used in LLVM IR for an external procedure declared without`。
- **L3159 EN**: Comment explains nearby logic, intent, or metadata: `BIND(C) in the same file. While this is not forbidden by the`.
  **L3159 CN**: 注释说明附近代码的逻辑、意图或元数据：`BIND(C) in the same file. While this is not forbidden by the`。
- **L3160 EN**: Comment explains nearby logic, intent, or metadata: `standard, this name collision would lead to a crash when producing`.
  **L3160 CN**: 注释说明附近代码的逻辑、意图或元数据：`standard, this name collision would lead to a crash when producing`。
- **L3161 EN**: Comment explains nearby logic, intent, or metadata: `the IR.`.
  **L3161 CN**: 注释说明附近代码的逻辑、意图或元数据：`the IR.`。
- **L3162 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3162 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s procedure assembly name conflicts with %s procedure assembly name"_err_en_US,`.
  **L3163 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s procedure assembly name conflicts with %s procedure assembly name"_err_en_US,`。
- **L3164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `hasExplicitBindingLabel ? "BIND(C)" : "Non BIND(C)",`.
  **L3164 CN**: 继续一个多行参数列表、初始化器或聚合项：`hasExplicitBindingLabel ? "BIND(C)" : "Non BIND(C)",`。
- **L3165 EN**: Starts a function, method, lambda, or structured scope: `hasExplicitBindingLabel ? "non BIND(C)" : "BIND(C)")}) {`.
  **L3165 CN**: 开始一个函数、方法、lambda 或结构化作用域：`hasExplicitBindingLabel ? "non BIND(C)" : "BIND(C)")}) {`。
- **L3166 EN**: Executes a call or declaration centered on `msg->Attach`.
  **L3166 CN**: 执行以 `msg->Attach` 为核心的调用或声明。
- **L3167 EN**: Closes the current lexical scope or compound statement.
  **L3167 CN**: 结束当前词法作用域或复合语句块。
- **L3168 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L3168 CN**: 执行以 `context_.SetError` 为核心的调用或声明。

### Lines 3169-3192

````cpp
        context_.SetError(other);
      }
      // Otherwise, the global names also match and the conflict is analyzed
      // by CheckGlobalName.
    }
  }
}

parser::Messages CheckHelper::WhyNotInteroperableDerivedType(
    const Symbol &symbol) {
  parser::Messages msgs;
  if (examinedByWhyNotInteroperable_.find(symbol) !=
      examinedByWhyNotInteroperable_.end()) {
    return msgs;
  }
  examinedByWhyNotInteroperable_.insert(symbol);
  if (const auto *derived{symbol.detailsIf<DerivedTypeDetails>()}) {
    if (derived->sequence()) { // C1801
      msgs.Say(symbol.name(),
          "An interoperable derived type cannot have the SEQUENCE attribute"_err_en_US);
    } else if (!derived->paramNameOrder().empty()) { // C1802
      msgs.Say(symbol.name(),
          "An interoperable derived type cannot have a type parameter"_err_en_US);
    } else if (const auto *parent{
````
- **L3169 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L3169 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L3170 EN**: Closes the current lexical scope or compound statement.
  **L3170 CN**: 结束当前词法作用域或复合语句块。
- **L3171 EN**: Comment explains nearby logic, intent, or metadata: `Otherwise, the global names also match and the conflict is analyzed`.
  **L3171 CN**: 注释说明附近代码的逻辑、意图或元数据：`Otherwise, the global names also match and the conflict is analyzed`。
- **L3172 EN**: Comment explains nearby logic, intent, or metadata: `by CheckGlobalName.`.
  **L3172 CN**: 注释说明附近代码的逻辑、意图或元数据：`by CheckGlobalName.`。
- **L3173 EN**: Closes the current lexical scope or compound statement.
  **L3173 CN**: 结束当前词法作用域或复合语句块。
- **L3174 EN**: Closes the current lexical scope or compound statement.
  **L3174 CN**: 结束当前词法作用域或复合语句块。
- **L3175 EN**: Closes the current lexical scope or compound statement.
  **L3175 CN**: 结束当前词法作用域或复合语句块。
- **L3176 EN**: Blank line separating nearby declarations or logic blocks.
  **L3176 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3177 EN**: Continues logic associated with callable symbol `WhyNotInteroperableDerivedType`.
  **L3177 CN**: 继续与可调用符号 `WhyNotInteroperableDerivedType` 相关的逻辑。
- **L3178 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol) {`.
  **L3178 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol) {`。
- **L3179 EN**: Executes a standalone statement or declaration: `parser::Messages msgs;`.
  **L3179 CN**: 执行一条独立语句或声明：`parser::Messages msgs;`。
- **L3180 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3180 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3181 EN**: Starts a function, method, lambda, or structured scope: `examinedByWhyNotInteroperable_.end()) {`.
  **L3181 CN**: 开始一个函数、方法、lambda 或结构化作用域：`examinedByWhyNotInteroperable_.end()) {`。
- **L3182 EN**: Returns from the current function with `msgs`.
  **L3182 CN**: 以 `msgs` 从当前函数返回。
- **L3183 EN**: Closes the current lexical scope or compound statement.
  **L3183 CN**: 结束当前词法作用域或复合语句块。
- **L3184 EN**: Executes a call or declaration centered on `examinedByWhyNotInteroperable_.insert`.
  **L3184 CN**: 执行以 `examinedByWhyNotInteroperable_.insert` 为核心的调用或声明。
- **L3185 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3185 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3186 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3186 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3187 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3188 EN**: Executes a standalone statement or declaration: `"An interoperable derived type cannot have the SEQUENCE attribute"_err_en_US);`.
  **L3188 CN**: 执行一条独立语句或声明：`"An interoperable derived type cannot have the SEQUENCE attribute"_err_en_US);`。
- **L3189 EN**: Transitions from the previous branch into an `else if` condition.
  **L3189 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3190 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3191 EN**: Executes a standalone statement or declaration: `"An interoperable derived type cannot have a type parameter"_err_en_US);`.
  **L3191 CN**: 执行一条独立语句或声明：`"An interoperable derived type cannot have a type parameter"_err_en_US);`。
- **L3192 EN**: Transitions from the previous branch into an `else if` condition.
  **L3192 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 3193-3216

````cpp
                   symbol.scope()->GetDerivedTypeParent()}) { // C1803
      if (symbol.attrs().test(Attr::BIND_C)) {
        msgs.Say(symbol.name(),
            "A derived type with the BIND attribute cannot be an extended derived type"_err_en_US);
      } else {
        bool interoperableParent{true};
        if (parent->symbol()) {
          auto bad{WhyNotInteroperableDerivedType(*parent->symbol())};
          if (bad.AnyFatalError()) {
            auto &msg{msgs.Say(symbol.name(),
                "The parent of an interoperable type is not interoperable"_err_en_US)};
            bad.AttachTo(msg, parser::Severity::None);
            interoperableParent = false;
          }
        }
        if (interoperableParent) {
          msgs.Say(symbol.name(),
              "An interoperable type should not be an extended derived type"_warn_en_US);
        }
      }
    }
    const Symbol *parentComponent{symbol.scope()
            ? derived->GetParentComponent(*symbol.scope())
            : nullptr};
````
- **L3193 EN**: Continues logic associated with callable symbol `scope`.
  **L3193 CN**: 继续与可调用符号 `scope` 相关的逻辑。
- **L3194 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3194 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3195 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3196 EN**: Executes a standalone statement or declaration: `"A derived type with the BIND attribute cannot be an extended derived type"_err_en_US);`.
  **L3196 CN**: 执行一条独立语句或声明：`"A derived type with the BIND attribute cannot be an extended derived type"_err_en_US);`。
- **L3197 EN**: Transitions from the previous branch into the alternative path.
  **L3197 CN**: 从前一个分支过渡到备选路径。
- **L3198 EN**: Executes a standalone statement or declaration: `bool interoperableParent{true};`.
  **L3198 CN**: 执行一条独立语句或声明：`bool interoperableParent{true};`。
- **L3199 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3199 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3200 EN**: Executes a call or declaration centered on `bad{WhyNotInteroperableDerivedType`.
  **L3200 CN**: 执行以 `bad{WhyNotInteroperableDerivedType` 为核心的调用或声明。
- **L3201 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3201 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto &msg{msgs.Say(symbol.name(),`.
  **L3202 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto &msg{msgs.Say(symbol.name(),`。
- **L3203 EN**: Executes a standalone statement or declaration: `"The parent of an interoperable type is not interoperable"_err_en_US)};`.
  **L3203 CN**: 执行一条独立语句或声明：`"The parent of an interoperable type is not interoperable"_err_en_US)};`。
- **L3204 EN**: Executes a call or declaration centered on `bad.AttachTo`.
  **L3204 CN**: 执行以 `bad.AttachTo` 为核心的调用或声明。
- **L3205 EN**: Executes a standalone statement or declaration: `interoperableParent = false;`.
  **L3205 CN**: 执行一条独立语句或声明：`interoperableParent = false;`。
- **L3206 EN**: Closes the current lexical scope or compound statement.
  **L3206 CN**: 结束当前词法作用域或复合语句块。
- **L3207 EN**: Closes the current lexical scope or compound statement.
  **L3207 CN**: 结束当前词法作用域或复合语句块。
- **L3208 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3208 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3209 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3210 EN**: Executes a standalone statement or declaration: `"An interoperable type should not be an extended derived type"_warn_en_US);`.
  **L3210 CN**: 执行一条独立语句或声明：`"An interoperable type should not be an extended derived type"_warn_en_US);`。
- **L3211 EN**: Closes the current lexical scope or compound statement.
  **L3211 CN**: 结束当前词法作用域或复合语句块。
- **L3212 EN**: Closes the current lexical scope or compound statement.
  **L3212 CN**: 结束当前词法作用域或复合语句块。
- **L3213 EN**: Closes the current lexical scope or compound statement.
  **L3213 CN**: 结束当前词法作用域或复合语句块。
- **L3214 EN**: Continues logic associated with callable symbol `scope`.
  **L3214 CN**: 继续与可调用符号 `scope` 相关的逻辑。
- **L3215 EN**: Continues logic associated with callable symbol `GetParentComponent`.
  **L3215 CN**: 继续与可调用符号 `GetParentComponent` 相关的逻辑。
- **L3216 EN**: Executes a standalone statement or declaration: `: nullptr};`.
  **L3216 CN**: 执行一条独立语句或声明：`: nullptr};`。

### Lines 3217-3240

````cpp
    for (const auto &pair : *symbol.scope()) {
      const Symbol &component{*pair.second};
      if (&component == parentComponent) {
        continue; // was checked above
      }
      if (IsProcedure(component)) { // C1804
        msgs.Say(component.name(),
            "An interoperable derived type cannot have a type bound procedure"_err_en_US);
      } else if (IsAllocatableOrPointer(component)) { // C1806
        msgs.Say(component.name(),
            "An interoperable derived type cannot have a pointer or allocatable component"_err_en_US);
      } else if (const auto *type{component.GetType()}) {
        if (const auto *derived{type->AsDerived()}) {
          auto bad{WhyNotInteroperableDerivedType(derived->typeSymbol())};
          if (bad.AnyFatalError()) {
            auto &msg{msgs.Say(component.name(),
                "Component '%s' of an interoperable derived type must have an interoperable type but does not"_err_en_US,
                component.name())};
            bad.AttachTo(msg, parser::Severity::None);
          } else if (!derived->typeSymbol().GetUltimate().attrs().test(
                         Attr::BIND_C)) {
            auto &msg{
                msgs.Say(component.name(),
                        "Derived type of component '%s' of an interoperable derived type should have the BIND attribute"_warn_en_US,
````
- **L3217 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3217 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3218 EN**: Executes a standalone statement or declaration: `const Symbol &component{*pair.second};`.
  **L3218 CN**: 执行一条独立语句或声明：`const Symbol &component{*pair.second};`。
- **L3219 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3219 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3220 EN**: Skips to the next loop iteration.
  **L3220 CN**: 跳到下一次循环迭代。
- **L3221 EN**: Closes the current lexical scope or compound statement.
  **L3221 CN**: 结束当前词法作用域或复合语句块。
- **L3222 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3222 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(component.name(),`.
  **L3223 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(component.name(),`。
- **L3224 EN**: Executes a standalone statement or declaration: `"An interoperable derived type cannot have a type bound procedure"_err_en_US);`.
  **L3224 CN**: 执行一条独立语句或声明：`"An interoperable derived type cannot have a type bound procedure"_err_en_US);`。
- **L3225 EN**: Transitions from the previous branch into an `else if` condition.
  **L3225 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(component.name(),`.
  **L3226 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(component.name(),`。
- **L3227 EN**: Executes a standalone statement or declaration: `"An interoperable derived type cannot have a pointer or allocatable component"_err_en_US);`.
  **L3227 CN**: 执行一条独立语句或声明：`"An interoperable derived type cannot have a pointer or allocatable component"_err_en_US);`。
- **L3228 EN**: Transitions from the previous branch into an `else if` condition.
  **L3228 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3229 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3229 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3230 EN**: Executes a call or declaration centered on `bad{WhyNotInteroperableDerivedType`.
  **L3230 CN**: 执行以 `bad{WhyNotInteroperableDerivedType` 为核心的调用或声明。
- **L3231 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3231 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto &msg{msgs.Say(component.name(),`.
  **L3232 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto &msg{msgs.Say(component.name(),`。
- **L3233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Component '%s' of an interoperable derived type must have an interoperable type but does not"_err_en_US,`.
  **L3233 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Component '%s' of an interoperable derived type must have an interoperable type but does not"_err_en_US,`。
- **L3234 EN**: Executes a call or declaration centered on `component.name`.
  **L3234 CN**: 执行以 `component.name` 为核心的调用或声明。
- **L3235 EN**: Executes a call or declaration centered on `bad.AttachTo`.
  **L3235 CN**: 执行以 `bad.AttachTo` 为核心的调用或声明。
- **L3236 EN**: Transitions from the previous branch into an `else if` condition.
  **L3236 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3237 EN**: Continues the surrounding expression or declaration: `Attr::BIND_C)) {`.
  **L3237 CN**: 继续构造周围的表达式或声明：`Attr::BIND_C)) {`。
- **L3238 EN**: Continues the surrounding expression or declaration: `auto &msg{`.
  **L3238 CN**: 继续构造周围的表达式或声明：`auto &msg{`。
- **L3239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(component.name(),`.
  **L3239 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(component.name(),`。
- **L3240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Derived type of component '%s' of an interoperable derived type should have the BIND attribute"_warn_en_US,`.
  **L3240 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Derived type of component '%s' of an interoperable derived type should have the BIND attribute"_warn_en_US,`。

### Lines 3241-3264

````cpp
                        component.name())
                    .Attach(derived->typeSymbol().name(),
                        "Non-BIND(C) component type"_en_US)};
            bad.AttachTo(msg, parser::Severity::None);
          } else {
            msgs.Annex(std::move(bad));
          }
        } else if (auto dyType{evaluate::DynamicType::From(*type)}; dyType &&
                   !evaluate::IsInteroperableIntrinsicType(
                       *dyType, &context_.languageFeatures())
                        .value_or(false)) {
          if (type->category() == DeclTypeSpec::Logical) {
            context().Warn(msgs, common::UsageWarning::LogicalVsCBool,
                component.name(),
                "A LOGICAL component of an interoperable type should have the interoperable KIND=C_BOOL"_port_en_US);
          } else if (type->category() == DeclTypeSpec::Character && dyType &&
              dyType->kind() == 1) {
            context().Warn(msgs, common::UsageWarning::BindCCharLength,
                component.name(),
                "A CHARACTER component of an interoperable type should have length 1"_port_en_US);
          } else {
            msgs.Say(component.name(),
                "Each component of an interoperable derived type must have an interoperable type"_err_en_US);
          }
````
- **L3241 EN**: Continues logic associated with callable symbol `name`.
  **L3241 CN**: 继续与可调用符号 `name` 相关的逻辑。
- **L3242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(derived->typeSymbol().name(),`.
  **L3242 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(derived->typeSymbol().name(),`。
- **L3243 EN**: Executes a call or declaration centered on `"Non-BIND`.
  **L3243 CN**: 执行以 `"Non-BIND` 为核心的调用或声明。
- **L3244 EN**: Executes a call or declaration centered on `bad.AttachTo`.
  **L3244 CN**: 执行以 `bad.AttachTo` 为核心的调用或声明。
- **L3245 EN**: Transitions from the previous branch into the alternative path.
  **L3245 CN**: 从前一个分支过渡到备选路径。
- **L3246 EN**: Executes a call or declaration centered on `msgs.Annex`.
  **L3246 CN**: 执行以 `msgs.Annex` 为核心的调用或声明。
- **L3247 EN**: Closes the current lexical scope or compound statement.
  **L3247 CN**: 结束当前词法作用域或复合语句块。
- **L3248 EN**: Transitions from the previous branch into an `else if` condition.
  **L3248 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3249 EN**: Continues logic associated with callable symbol `IsInteroperableIntrinsicType`.
  **L3249 CN**: 继续与可调用符号 `IsInteroperableIntrinsicType` 相关的逻辑。
- **L3250 EN**: Comment explains nearby logic, intent, or metadata: `dyType, &context_.languageFeatures())`.
  **L3250 CN**: 注释说明附近代码的逻辑、意图或元数据：`dyType, &context_.languageFeatures())`。
- **L3251 EN**: Starts a function, method, lambda, or structured scope: `.value_or(false)) {`.
  **L3251 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.value_or(false)) {`。
- **L3252 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3252 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context().Warn(msgs, common::UsageWarning::LogicalVsCBool,`.
  **L3253 CN**: 继续一个多行参数列表、初始化器或聚合项：`context().Warn(msgs, common::UsageWarning::LogicalVsCBool,`。
- **L3254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `component.name(),`.
  **L3254 CN**: 继续一个多行参数列表、初始化器或聚合项：`component.name(),`。
- **L3255 EN**: Executes a standalone statement or declaration: `"A LOGICAL component of an interoperable type should have the interoperable KIND=C_BOOL"_port_en_US);`.
  **L3255 CN**: 执行一条独立语句或声明：`"A LOGICAL component of an interoperable type should have the interoperable KIND=C_BOOL"_port_en_US);`。
- **L3256 EN**: Transitions from the previous branch into an `else if` condition.
  **L3256 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3257 EN**: Starts a function, method, lambda, or structured scope: `dyType->kind() == 1) {`.
  **L3257 CN**: 开始一个函数、方法、lambda 或结构化作用域：`dyType->kind() == 1) {`。
- **L3258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context().Warn(msgs, common::UsageWarning::BindCCharLength,`.
  **L3258 CN**: 继续一个多行参数列表、初始化器或聚合项：`context().Warn(msgs, common::UsageWarning::BindCCharLength,`。
- **L3259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `component.name(),`.
  **L3259 CN**: 继续一个多行参数列表、初始化器或聚合项：`component.name(),`。
- **L3260 EN**: Executes a standalone statement or declaration: `"A CHARACTER component of an interoperable type should have length 1"_port_en_US);`.
  **L3260 CN**: 执行一条独立语句或声明：`"A CHARACTER component of an interoperable type should have length 1"_port_en_US);`。
- **L3261 EN**: Transitions from the previous branch into the alternative path.
  **L3261 CN**: 从前一个分支过渡到备选路径。
- **L3262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(component.name(),`.
  **L3262 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(component.name(),`。
- **L3263 EN**: Executes a standalone statement or declaration: `"Each component of an interoperable derived type must have an interoperable type"_err_en_US);`.
  **L3263 CN**: 执行一条独立语句或声明：`"Each component of an interoperable derived type must have an interoperable type"_err_en_US);`。
- **L3264 EN**: Closes the current lexical scope or compound statement.
  **L3264 CN**: 结束当前词法作用域或复合语句块。

### Lines 3265-3288

````cpp
        }
      }
      if (auto extents{
              evaluate::GetConstantExtents(foldingContext_, &component)};
          extents && evaluate::GetSize(*extents) == 0) {
        msgs.Say(component.name(),
            "An array component of an interoperable type must have at least one element"_err_en_US);
      }
    }
    if (derived->componentNames().empty()) { // F'2023 C1805
      context().Warn(msgs, common::LanguageFeature::EmptyBindCDerivedType,
          symbol.name(),
          "A derived type with the BIND attribute should not be empty"_warn_en_US);
    }
  }
  if (msgs.AnyFatalError()) {
    examinedByWhyNotInteroperable_.erase(symbol);
  }
  return msgs;
}

parser::Messages CheckHelper::WhyNotInteroperableObject(
    const Symbol &symbol, bool allowNonInteroperableType, bool forCommonBlock) {
  parser::Messages msgs;
````
- **L3265 EN**: Closes the current lexical scope or compound statement.
  **L3265 CN**: 结束当前词法作用域或复合语句块。
- **L3266 EN**: Closes the current lexical scope or compound statement.
  **L3266 CN**: 结束当前词法作用域或复合语句块。
- **L3267 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3267 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3268 EN**: Executes a call or declaration centered on `evaluate::GetConstantExtents`.
  **L3268 CN**: 执行以 `evaluate::GetConstantExtents` 为核心的调用或声明。
- **L3269 EN**: Starts a function, method, lambda, or structured scope: `extents && evaluate::GetSize(*extents) == 0) {`.
  **L3269 CN**: 开始一个函数、方法、lambda 或结构化作用域：`extents && evaluate::GetSize(*extents) == 0) {`。
- **L3270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(component.name(),`.
  **L3270 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(component.name(),`。
- **L3271 EN**: Executes a standalone statement or declaration: `"An array component of an interoperable type must have at least one element"_err_en_US);`.
  **L3271 CN**: 执行一条独立语句或声明：`"An array component of an interoperable type must have at least one element"_err_en_US);`。
- **L3272 EN**: Closes the current lexical scope or compound statement.
  **L3272 CN**: 结束当前词法作用域或复合语句块。
- **L3273 EN**: Closes the current lexical scope or compound statement.
  **L3273 CN**: 结束当前词法作用域或复合语句块。
- **L3274 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3274 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context().Warn(msgs, common::LanguageFeature::EmptyBindCDerivedType,`.
  **L3275 CN**: 继续一个多行参数列表、初始化器或聚合项：`context().Warn(msgs, common::LanguageFeature::EmptyBindCDerivedType,`。
- **L3276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol.name(),`.
  **L3276 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol.name(),`。
- **L3277 EN**: Executes a standalone statement or declaration: `"A derived type with the BIND attribute should not be empty"_warn_en_US);`.
  **L3277 CN**: 执行一条独立语句或声明：`"A derived type with the BIND attribute should not be empty"_warn_en_US);`。
- **L3278 EN**: Closes the current lexical scope or compound statement.
  **L3278 CN**: 结束当前词法作用域或复合语句块。
- **L3279 EN**: Closes the current lexical scope or compound statement.
  **L3279 CN**: 结束当前词法作用域或复合语句块。
- **L3280 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3280 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3281 EN**: Executes a call or declaration centered on `examinedByWhyNotInteroperable_.erase`.
  **L3281 CN**: 执行以 `examinedByWhyNotInteroperable_.erase` 为核心的调用或声明。
- **L3282 EN**: Closes the current lexical scope or compound statement.
  **L3282 CN**: 结束当前词法作用域或复合语句块。
- **L3283 EN**: Returns from the current function with `msgs`.
  **L3283 CN**: 以 `msgs` 从当前函数返回。
- **L3284 EN**: Closes the current lexical scope or compound statement.
  **L3284 CN**: 结束当前词法作用域或复合语句块。
- **L3285 EN**: Blank line separating nearby declarations or logic blocks.
  **L3285 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3286 EN**: Continues logic associated with callable symbol `WhyNotInteroperableObject`.
  **L3286 CN**: 继续与可调用符号 `WhyNotInteroperableObject` 相关的逻辑。
- **L3287 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, bool allowNonInteroperableType, bool forCommonBlock) {`.
  **L3287 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, bool allowNonInteroperableType, bool forCommonBlock) {`。
- **L3288 EN**: Executes a standalone statement or declaration: `parser::Messages msgs;`.
  **L3288 CN**: 执行一条独立语句或声明：`parser::Messages msgs;`。

### Lines 3289-3312

````cpp
  if (!forCommonBlock) {
    if (examinedByWhyNotInteroperable_.find(symbol) !=
        examinedByWhyNotInteroperable_.end()) {
      return msgs;
    }
    examinedByWhyNotInteroperable_.insert(symbol);
  }
  bool isExplicitBindC{symbol.attrs().test(Attr::BIND_C)};
  CHECK(symbol.has<ObjectEntityDetails>());
  if (isExplicitBindC && !symbol.owner().IsModule()) {
    msgs.Say(symbol.name(),
        "A variable with BIND(C) attribute may only appear in the specification part of a module"_err_en_US);
  }
  auto shape{evaluate::GetShape(foldingContext_, symbol)};
  if (shape) {
    if (evaluate::GetRank(*shape) == 0) { // 18.3.4
      if (IsAllocatableOrPointer(symbol) && !IsDummy(symbol)) {
        msgs.Say(symbol.name(),
            "A scalar interoperable variable may not be ALLOCATABLE or POINTER"_err_en_US);
      }
    } else if (auto extents{
                   evaluate::AsConstantExtents(foldingContext_, *shape)}) {
      if (evaluate::GetSize(*extents) == 0) {
        msgs.Say(symbol.name(),
````
- **L3289 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3289 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3290 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3290 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3291 EN**: Starts a function, method, lambda, or structured scope: `examinedByWhyNotInteroperable_.end()) {`.
  **L3291 CN**: 开始一个函数、方法、lambda 或结构化作用域：`examinedByWhyNotInteroperable_.end()) {`。
- **L3292 EN**: Returns from the current function with `msgs`.
  **L3292 CN**: 以 `msgs` 从当前函数返回。
- **L3293 EN**: Closes the current lexical scope or compound statement.
  **L3293 CN**: 结束当前词法作用域或复合语句块。
- **L3294 EN**: Executes a call or declaration centered on `examinedByWhyNotInteroperable_.insert`.
  **L3294 CN**: 执行以 `examinedByWhyNotInteroperable_.insert` 为核心的调用或声明。
- **L3295 EN**: Closes the current lexical scope or compound statement.
  **L3295 CN**: 结束当前词法作用域或复合语句块。
- **L3296 EN**: Executes a call or declaration centered on `isExplicitBindC{symbol.attrs`.
  **L3296 CN**: 执行以 `isExplicitBindC{symbol.attrs` 为核心的调用或声明。
- **L3297 EN**: Executes a call or declaration centered on `CHECK`.
  **L3297 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L3298 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3298 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3299 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3300 EN**: Executes a call or declaration centered on `BIND`.
  **L3300 CN**: 执行以 `BIND` 为核心的调用或声明。
- **L3301 EN**: Closes the current lexical scope or compound statement.
  **L3301 CN**: 结束当前词法作用域或复合语句块。
- **L3302 EN**: Executes a call or declaration centered on `shape{evaluate::GetShape`.
  **L3302 CN**: 执行以 `shape{evaluate::GetShape` 为核心的调用或声明。
- **L3303 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3303 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3304 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3304 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3305 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3305 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3306 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3307 EN**: Executes a standalone statement or declaration: `"A scalar interoperable variable may not be ALLOCATABLE or POINTER"_err_en_US);`.
  **L3307 CN**: 执行一条独立语句或声明：`"A scalar interoperable variable may not be ALLOCATABLE or POINTER"_err_en_US);`。
- **L3308 EN**: Closes the current lexical scope or compound statement.
  **L3308 CN**: 结束当前词法作用域或复合语句块。
- **L3309 EN**: Transitions from the previous branch into an `else if` condition.
  **L3309 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3310 EN**: Starts a function, method, lambda, or structured scope: `evaluate::AsConstantExtents(foldingContext_, *shape)}) {`.
  **L3310 CN**: 开始一个函数、方法、lambda 或结构化作用域：`evaluate::AsConstantExtents(foldingContext_, *shape)}) {`。
- **L3311 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3311 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3312 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。

### Lines 3313-3336

````cpp
            "Interoperable array must have at least one element"_err_en_US);
      }
    } else if (!evaluate::IsExplicitShape(symbol) &&
        !IsAssumedSizeArray(symbol) &&
        !(IsDummy(symbol) && !symbol.attrs().test(Attr::VALUE))) {
      msgs.Say(symbol.name(),
          "BIND(C) array must have explicit shape or be assumed-size unless a dummy argument without the VALUE attribute"_err_en_US);
    }
  }
  if (const auto *type{symbol.GetType()}) {
    const auto *derived{type->AsDerived()};
    if (derived && !derived->typeSymbol().attrs().test(Attr::BIND_C)) {
      if (allowNonInteroperableType) { // portability warning only
        evaluate::AttachDeclaration(
            Warn(common::UsageWarning::Portability, symbol.name(),
                "The derived type of this interoperable object should be BIND(C)"_port_en_US),
            derived->typeSymbol());
      } else if (!context_.IsEnabled(
                     common::LanguageFeature::NonBindCInteroperability)) {
        msgs.Say(symbol.name(),
                "The derived type of an interoperable object must be BIND(C)"_err_en_US)
            .Attach(derived->typeSymbol().name(), "Non-BIND(C) type"_en_US);
      } else if (auto bad{
                     WhyNotInteroperableDerivedType(derived->typeSymbol())};
````
- **L3313 EN**: Executes a standalone statement or declaration: `"Interoperable array must have at least one element"_err_en_US);`.
  **L3313 CN**: 执行一条独立语句或声明：`"Interoperable array must have at least one element"_err_en_US);`。
- **L3314 EN**: Closes the current lexical scope or compound statement.
  **L3314 CN**: 结束当前词法作用域或复合语句块。
- **L3315 EN**: Transitions from the previous branch into an `else if` condition.
  **L3315 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3316 EN**: Continues logic associated with callable symbol `IsAssumedSizeArray`.
  **L3316 CN**: 继续与可调用符号 `IsAssumedSizeArray` 相关的逻辑。
- **L3317 EN**: Starts a function, method, lambda, or structured scope: `!(IsDummy(symbol) && !symbol.attrs().test(Attr::VALUE))) {`.
  **L3317 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!(IsDummy(symbol) && !symbol.attrs().test(Attr::VALUE))) {`。
- **L3318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3318 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3319 EN**: Executes a call or declaration centered on `"BIND`.
  **L3319 CN**: 执行以 `"BIND` 为核心的调用或声明。
- **L3320 EN**: Closes the current lexical scope or compound statement.
  **L3320 CN**: 结束当前词法作用域或复合语句块。
- **L3321 EN**: Closes the current lexical scope or compound statement.
  **L3321 CN**: 结束当前词法作用域或复合语句块。
- **L3322 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3322 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3323 EN**: Executes a call or declaration centered on `*derived{type->AsDerived`.
  **L3323 CN**: 执行以 `*derived{type->AsDerived` 为核心的调用或声明。
- **L3324 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3324 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3325 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3325 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3326 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L3326 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L3327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::Portability, symbol.name(),`.
  **L3327 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::Portability, symbol.name(),`。
- **L3328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The derived type of this interoperable object should be BIND(C)"_port_en_US),`.
  **L3328 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The derived type of this interoperable object should be BIND(C)"_port_en_US),`。
- **L3329 EN**: Executes a call or declaration centered on `derived->typeSymbol`.
  **L3329 CN**: 执行以 `derived->typeSymbol` 为核心的调用或声明。
- **L3330 EN**: Transitions from the previous branch into an `else if` condition.
  **L3330 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3331 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::NonBindCInteroperability)) {`.
  **L3331 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::NonBindCInteroperability)) {`。
- **L3332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3332 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3333 EN**: Continues logic associated with callable symbol `BIND`.
  **L3333 CN**: 继续与可调用符号 `BIND` 相关的逻辑。
- **L3334 EN**: Executes a call or declaration centered on `.Attach`.
  **L3334 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L3335 EN**: Transitions from the previous branch into an `else if` condition.
  **L3335 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3336 EN**: Executes a call or declaration centered on `WhyNotInteroperableDerivedType`.
  **L3336 CN**: 执行以 `WhyNotInteroperableDerivedType` 为核心的调用或声明。

### Lines 3337-3360

````cpp
                 bad.AnyFatalError()) {
        bad.AttachTo(
            msgs.Say(symbol.name(),
                    "The derived type of an interoperable object must be interoperable, but is not"_err_en_US)
                .Attach(derived->typeSymbol().name(),
                    "Non-interoperable type"_en_US),
            parser::Severity::None);
      } else {
        msgs.Say(symbol.name(),
                "The derived type of an interoperable object should be BIND(C)"_warn_en_US)
            .Attach(derived->typeSymbol().name(), "Non-BIND(C) type"_en_US);
      }
    }
    if (type->IsAssumedType()) { // ok
    } else if (IsAssumedLengthCharacter(symbol) &&
        !IsAllocatableOrPointer(symbol)) {
    } else if (IsAllocatableOrPointer(symbol) &&
        type->category() == DeclTypeSpec::Character &&
        type->characterTypeSpec().length().isDeferred()) {
      // ok; F'2023 18.3.7 p2(6)
    } else if (derived) { // type has been checked
    } else if (auto dyType{evaluate::DynamicType::From(*type)}; dyType &&
        evaluate::IsInteroperableIntrinsicType(
            *dyType, InModuleFile() ? nullptr : &context_.languageFeatures())
````
- **L3337 EN**: Starts a function, method, lambda, or structured scope: `bad.AnyFatalError()) {`.
  **L3337 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bad.AnyFatalError()) {`。
- **L3338 EN**: Continues logic associated with callable symbol `AttachTo`.
  **L3338 CN**: 继续与可调用符号 `AttachTo` 相关的逻辑。
- **L3339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3339 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3340 EN**: Continues the surrounding expression or declaration: `"The derived type of an interoperable object must be interoperable, but is not"_err_en_US)`.
  **L3340 CN**: 继续构造周围的表达式或声明：`"The derived type of an interoperable object must be interoperable, but is not"_err_en_US)`。
- **L3341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(derived->typeSymbol().name(),`.
  **L3341 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(derived->typeSymbol().name(),`。
- **L3342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Non-interoperable type"_en_US),`.
  **L3342 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Non-interoperable type"_en_US),`。
- **L3343 EN**: Executes a standalone statement or declaration: `parser::Severity::None);`.
  **L3343 CN**: 执行一条独立语句或声明：`parser::Severity::None);`。
- **L3344 EN**: Transitions from the previous branch into the alternative path.
  **L3344 CN**: 从前一个分支过渡到备选路径。
- **L3345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3345 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3346 EN**: Continues logic associated with callable symbol `BIND`.
  **L3346 CN**: 继续与可调用符号 `BIND` 相关的逻辑。
- **L3347 EN**: Executes a call or declaration centered on `.Attach`.
  **L3347 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L3348 EN**: Closes the current lexical scope or compound statement.
  **L3348 CN**: 结束当前词法作用域或复合语句块。
- **L3349 EN**: Closes the current lexical scope or compound statement.
  **L3349 CN**: 结束当前词法作用域或复合语句块。
- **L3350 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3350 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3351 EN**: Transitions from the previous branch into an `else if` condition.
  **L3351 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3352 EN**: Starts a function, method, lambda, or structured scope: `!IsAllocatableOrPointer(symbol)) {`.
  **L3352 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!IsAllocatableOrPointer(symbol)) {`。
- **L3353 EN**: Transitions from the previous branch into an `else if` condition.
  **L3353 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3354 EN**: Continues logic associated with callable symbol `category`.
  **L3354 CN**: 继续与可调用符号 `category` 相关的逻辑。
- **L3355 EN**: Starts a function, method, lambda, or structured scope: `type->characterTypeSpec().length().isDeferred()) {`.
  **L3355 CN**: 开始一个函数、方法、lambda 或结构化作用域：`type->characterTypeSpec().length().isDeferred()) {`。
- **L3356 EN**: Comment explains nearby logic, intent, or metadata: `ok; F'2023 18.3.7 p2(6)`.
  **L3356 CN**: 注释说明附近代码的逻辑、意图或元数据：`ok; F'2023 18.3.7 p2(6)`。
- **L3357 EN**: Transitions from the previous branch into an `else if` condition.
  **L3357 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3358 EN**: Transitions from the previous branch into an `else if` condition.
  **L3358 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3359 EN**: Continues logic associated with callable symbol `IsInteroperableIntrinsicType`.
  **L3359 CN**: 继续与可调用符号 `IsInteroperableIntrinsicType` 相关的逻辑。
- **L3360 EN**: Comment explains nearby logic, intent, or metadata: `dyType, InModuleFile() ? nullptr : &context_.languageFeatures())`.
  **L3360 CN**: 注释说明附近代码的逻辑、意图或元数据：`dyType, InModuleFile() ? nullptr : &context_.languageFeatures())`。

### Lines 3361-3384

````cpp
            .value_or(false)) {
      // F'2023 18.3.7 p2(4,5)
      // N.B. Language features are not passed to IsInteroperableIntrinsicType
      // when processing a module file, since the module file might have been
      // compiled with CUDA while the client is not.
    } else if (type->category() == DeclTypeSpec::Logical) {
      if (context_.ShouldWarn(common::UsageWarning::LogicalVsCBool)) {
        if (IsDummy(symbol)) {
          Warn(common::UsageWarning::LogicalVsCBool, symbol.name(),
              "A BIND(C) LOGICAL dummy argument should have the interoperable KIND=C_BOOL"_port_en_US);
        } else {
          Warn(common::UsageWarning::LogicalVsCBool, symbol.name(),
              "A BIND(C) LOGICAL object should have the interoperable KIND=C_BOOL"_port_en_US);
        }
      }
    } else if (symbol.attrs().test(Attr::VALUE)) {
      msgs.Say(symbol.name(),
          "A BIND(C) VALUE dummy argument must have an interoperable type"_err_en_US);
    } else {
      msgs.Say(symbol.name(),
          "A BIND(C) object must have an interoperable type"_err_en_US);
    }
  }
  if (IsOptional(symbol) && !symbol.attrs().test(Attr::VALUE)) {
````
- **L3361 EN**: Starts a function, method, lambda, or structured scope: `.value_or(false)) {`.
  **L3361 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.value_or(false)) {`。
- **L3362 EN**: Comment explains nearby logic, intent, or metadata: `F'2023 18.3.7 p2(4,5)`.
  **L3362 CN**: 注释说明附近代码的逻辑、意图或元数据：`F'2023 18.3.7 p2(4,5)`。
- **L3363 EN**: Comment explains nearby logic, intent, or metadata: `N.B. Language features are not passed to IsInteroperableIntrinsicType`.
  **L3363 CN**: 注释说明附近代码的逻辑、意图或元数据：`N.B. Language features are not passed to IsInteroperableIntrinsicType`。
- **L3364 EN**: Comment explains nearby logic, intent, or metadata: `when processing a module file, since the module file might have been`.
  **L3364 CN**: 注释说明附近代码的逻辑、意图或元数据：`when processing a module file, since the module file might have been`。
- **L3365 EN**: Comment explains nearby logic, intent, or metadata: `compiled with CUDA while the client is not.`.
  **L3365 CN**: 注释说明附近代码的逻辑、意图或元数据：`compiled with CUDA while the client is not.`。
- **L3366 EN**: Transitions from the previous branch into an `else if` condition.
  **L3366 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3367 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3367 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3368 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3368 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::LogicalVsCBool, symbol.name(),`.
  **L3369 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::LogicalVsCBool, symbol.name(),`。
- **L3370 EN**: Executes a call or declaration centered on `BIND`.
  **L3370 CN**: 执行以 `BIND` 为核心的调用或声明。
- **L3371 EN**: Transitions from the previous branch into the alternative path.
  **L3371 CN**: 从前一个分支过渡到备选路径。
- **L3372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warn(common::UsageWarning::LogicalVsCBool, symbol.name(),`.
  **L3372 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warn(common::UsageWarning::LogicalVsCBool, symbol.name(),`。
- **L3373 EN**: Executes a call or declaration centered on `BIND`.
  **L3373 CN**: 执行以 `BIND` 为核心的调用或声明。
- **L3374 EN**: Closes the current lexical scope or compound statement.
  **L3374 CN**: 结束当前词法作用域或复合语句块。
- **L3375 EN**: Closes the current lexical scope or compound statement.
  **L3375 CN**: 结束当前词法作用域或复合语句块。
- **L3376 EN**: Transitions from the previous branch into an `else if` condition.
  **L3376 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3377 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3378 EN**: Executes a call or declaration centered on `BIND`.
  **L3378 CN**: 执行以 `BIND` 为核心的调用或声明。
- **L3379 EN**: Transitions from the previous branch into the alternative path.
  **L3379 CN**: 从前一个分支过渡到备选路径。
- **L3380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3380 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3381 EN**: Executes a call or declaration centered on `BIND`.
  **L3381 CN**: 执行以 `BIND` 为核心的调用或声明。
- **L3382 EN**: Closes the current lexical scope or compound statement.
  **L3382 CN**: 结束当前词法作用域或复合语句块。
- **L3383 EN**: Closes the current lexical scope or compound statement.
  **L3383 CN**: 结束当前词法作用域或复合语句块。
- **L3384 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3384 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3385-3408

````cpp
    msgs.Say(symbol.name(),
        "An interoperable procedure with an OPTIONAL dummy argument might not be portable"_port_en_US);
  }
  if (IsDescriptor(symbol) && IsPointer(symbol) &&
      symbol.attrs().test(Attr::CONTIGUOUS)) {
    msgs.Say(symbol.name(),
        "An interoperable pointer must not be CONTIGUOUS"_err_en_US);
  }
  if (!forCommonBlock && msgs.AnyFatalError()) {
    examinedByWhyNotInteroperable_.erase(symbol);
  }
  return msgs;
}

parser::Messages CheckHelper::WhyNotInteroperableFunctionResult(
    const Symbol &symbol) {
  parser::Messages msgs;
  if (IsPointer(symbol) || IsAllocatable(symbol)) {
    msgs.Say(symbol.name(),
        "Interoperable function result may not have ALLOCATABLE or POINTER attribute"_err_en_US);
  }
  if (const DeclTypeSpec * type{symbol.GetType()};
      type && type->category() == DeclTypeSpec::Character) {
    bool isConstOne{false}; // 18.3.1(1)
````
- **L3385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3385 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3386 EN**: Executes a standalone statement or declaration: `"An interoperable procedure with an OPTIONAL dummy argument might not be portable"_port_en_US);`.
  **L3386 CN**: 执行一条独立语句或声明：`"An interoperable procedure with an OPTIONAL dummy argument might not be portable"_port_en_US);`。
- **L3387 EN**: Closes the current lexical scope or compound statement.
  **L3387 CN**: 结束当前词法作用域或复合语句块。
- **L3388 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3388 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3389 EN**: Starts a function, method, lambda, or structured scope: `symbol.attrs().test(Attr::CONTIGUOUS)) {`.
  **L3389 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.attrs().test(Attr::CONTIGUOUS)) {`。
- **L3390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3390 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3391 EN**: Executes a standalone statement or declaration: `"An interoperable pointer must not be CONTIGUOUS"_err_en_US);`.
  **L3391 CN**: 执行一条独立语句或声明：`"An interoperable pointer must not be CONTIGUOUS"_err_en_US);`。
- **L3392 EN**: Closes the current lexical scope or compound statement.
  **L3392 CN**: 结束当前词法作用域或复合语句块。
- **L3393 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3393 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3394 EN**: Executes a call or declaration centered on `examinedByWhyNotInteroperable_.erase`.
  **L3394 CN**: 执行以 `examinedByWhyNotInteroperable_.erase` 为核心的调用或声明。
- **L3395 EN**: Closes the current lexical scope or compound statement.
  **L3395 CN**: 结束当前词法作用域或复合语句块。
- **L3396 EN**: Returns from the current function with `msgs`.
  **L3396 CN**: 以 `msgs` 从当前函数返回。
- **L3397 EN**: Closes the current lexical scope or compound statement.
  **L3397 CN**: 结束当前词法作用域或复合语句块。
- **L3398 EN**: Blank line separating nearby declarations or logic blocks.
  **L3398 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3399 EN**: Continues logic associated with callable symbol `WhyNotInteroperableFunctionResult`.
  **L3399 CN**: 继续与可调用符号 `WhyNotInteroperableFunctionResult` 相关的逻辑。
- **L3400 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol) {`.
  **L3400 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol) {`。
- **L3401 EN**: Executes a standalone statement or declaration: `parser::Messages msgs;`.
  **L3401 CN**: 执行一条独立语句或声明：`parser::Messages msgs;`。
- **L3402 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3402 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3403 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3404 EN**: Executes a standalone statement or declaration: `"Interoperable function result may not have ALLOCATABLE or POINTER attribute"_err_en_US);`.
  **L3404 CN**: 执行一条独立语句或声明：`"Interoperable function result may not have ALLOCATABLE or POINTER attribute"_err_en_US);`。
- **L3405 EN**: Closes the current lexical scope or compound statement.
  **L3405 CN**: 结束当前词法作用域或复合语句块。
- **L3406 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3406 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3407 EN**: Starts a function, method, lambda, or structured scope: `type && type->category() == DeclTypeSpec::Character) {`.
  **L3407 CN**: 开始一个函数、方法、lambda 或结构化作用域：`type && type->category() == DeclTypeSpec::Character) {`。
- **L3408 EN**: Continues the surrounding expression or declaration: `bool isConstOne{false}; // 18.3.1(1)`.
  **L3408 CN**: 继续构造周围的表达式或声明：`bool isConstOne{false}; // 18.3.1(1)`。

### Lines 3409-3432

````cpp
    if (const auto &len{type->characterTypeSpec().length().GetExplicit()}) {
      if (auto constLen{evaluate::ToInt64(*len)}) {
        isConstOne = constLen == 1;
      }
    }
    if (!isConstOne) {
      msgs.Say(symbol.name(),
          "Interoperable character function result must have length one"_err_en_US);
    }
  }
  if (symbol.Rank() > 0) {
    msgs.Say(symbol.name(),
        "Interoperable function result must be scalar"_err_en_US);
  }
  return msgs;
}

parser::Messages CheckHelper::WhyNotInteroperableProcedure(
    const Symbol &symbol, bool isError) {
  parser::Messages msgs;
  if (examinedByWhyNotInteroperable_.find(symbol) !=
      examinedByWhyNotInteroperable_.end()) {
    return msgs;
  }
````
- **L3409 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3409 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3410 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3410 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3411 EN**: Executes a standalone statement or declaration: `isConstOne = constLen == 1;`.
  **L3411 CN**: 执行一条独立语句或声明：`isConstOne = constLen == 1;`。
- **L3412 EN**: Closes the current lexical scope or compound statement.
  **L3412 CN**: 结束当前词法作用域或复合语句块。
- **L3413 EN**: Closes the current lexical scope or compound statement.
  **L3413 CN**: 结束当前词法作用域或复合语句块。
- **L3414 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3414 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3415 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3416 EN**: Executes a standalone statement or declaration: `"Interoperable character function result must have length one"_err_en_US);`.
  **L3416 CN**: 执行一条独立语句或声明：`"Interoperable character function result must have length one"_err_en_US);`。
- **L3417 EN**: Closes the current lexical scope or compound statement.
  **L3417 CN**: 结束当前词法作用域或复合语句块。
- **L3418 EN**: Closes the current lexical scope or compound statement.
  **L3418 CN**: 结束当前词法作用域或复合语句块。
- **L3419 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3419 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3420 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3421 EN**: Executes a standalone statement or declaration: `"Interoperable function result must be scalar"_err_en_US);`.
  **L3421 CN**: 执行一条独立语句或声明：`"Interoperable function result must be scalar"_err_en_US);`。
- **L3422 EN**: Closes the current lexical scope or compound statement.
  **L3422 CN**: 结束当前词法作用域或复合语句块。
- **L3423 EN**: Returns from the current function with `msgs`.
  **L3423 CN**: 以 `msgs` 从当前函数返回。
- **L3424 EN**: Closes the current lexical scope or compound statement.
  **L3424 CN**: 结束当前词法作用域或复合语句块。
- **L3425 EN**: Blank line separating nearby declarations or logic blocks.
  **L3425 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3426 EN**: Continues logic associated with callable symbol `WhyNotInteroperableProcedure`.
  **L3426 CN**: 继续与可调用符号 `WhyNotInteroperableProcedure` 相关的逻辑。
- **L3427 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, bool isError) {`.
  **L3427 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, bool isError) {`。
- **L3428 EN**: Executes a standalone statement or declaration: `parser::Messages msgs;`.
  **L3428 CN**: 执行一条独立语句或声明：`parser::Messages msgs;`。
- **L3429 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3429 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3430 EN**: Starts a function, method, lambda, or structured scope: `examinedByWhyNotInteroperable_.end()) {`.
  **L3430 CN**: 开始一个函数、方法、lambda 或结构化作用域：`examinedByWhyNotInteroperable_.end()) {`。
- **L3431 EN**: Returns from the current function with `msgs`.
  **L3431 CN**: 以 `msgs` 从当前函数返回。
- **L3432 EN**: Closes the current lexical scope or compound statement.
  **L3432 CN**: 结束当前词法作用域或复合语句块。

### Lines 3433-3456

````cpp
  isError |= symbol.attrs().test(Attr::BIND_C);
  examinedByWhyNotInteroperable_.insert(symbol);
  if (const auto *proc{symbol.detailsIf<ProcEntityDetails>()}) {
    if (isError) {
      if (!proc->procInterface() ||
          !proc->procInterface()->attrs().test(Attr::BIND_C)) {
        msgs.Say(symbol.name(),
            "An interface name with the BIND attribute must appear if the BIND attribute appears in a procedure declaration"_err_en_US);
      }
    } else if (!proc->procInterface()) {
      msgs.Say(symbol.name(),
          "An interoperable procedure should have an interface"_port_en_US);
    } else if (!proc->procInterface()->attrs().test(Attr::BIND_C)) {
      auto bad{WhyNotInteroperableProcedure(
          *proc->procInterface(), /*isError=*/false)};
      if (bad.AnyFatalError()) {
        bad.AttachTo(msgs.Say(symbol.name(),
            "An interoperable procedure must have an interoperable interface"_err_en_US));
      } else {
        msgs.Say(symbol.name(),
            "An interoperable procedure should have an interface with the BIND attribute"_warn_en_US);
      }
    }
  } else if (const auto *subp{symbol.detailsIf<SubprogramDetails>()}) {
````
- **L3433 EN**: Executes a call or declaration centered on `symbol.attrs`.
  **L3433 CN**: 执行以 `symbol.attrs` 为核心的调用或声明。
- **L3434 EN**: Executes a call or declaration centered on `examinedByWhyNotInteroperable_.insert`.
  **L3434 CN**: 执行以 `examinedByWhyNotInteroperable_.insert` 为核心的调用或声明。
- **L3435 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3435 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3436 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3436 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3437 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3437 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3438 EN**: Starts a function, method, lambda, or structured scope: `!proc->procInterface()->attrs().test(Attr::BIND_C)) {`.
  **L3438 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!proc->procInterface()->attrs().test(Attr::BIND_C)) {`。
- **L3439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3439 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3440 EN**: Executes a standalone statement or declaration: `"An interface name with the BIND attribute must appear if the BIND attribute appears in a procedure declaration"_err_en_US);`.
  **L3440 CN**: 执行一条独立语句或声明：`"An interface name with the BIND attribute must appear if the BIND attribute appears in a procedure declaration"_err_en_US);`。
- **L3441 EN**: Closes the current lexical scope or compound statement.
  **L3441 CN**: 结束当前词法作用域或复合语句块。
- **L3442 EN**: Transitions from the previous branch into an `else if` condition.
  **L3442 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3443 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3444 EN**: Executes a standalone statement or declaration: `"An interoperable procedure should have an interface"_port_en_US);`.
  **L3444 CN**: 执行一条独立语句或声明：`"An interoperable procedure should have an interface"_port_en_US);`。
- **L3445 EN**: Transitions from the previous branch into an `else if` condition.
  **L3445 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3446 EN**: Continues logic associated with callable symbol `WhyNotInteroperableProcedure`.
  **L3446 CN**: 继续与可调用符号 `WhyNotInteroperableProcedure` 相关的逻辑。
- **L3447 EN**: Comment explains nearby logic, intent, or metadata: `proc->procInterface(), /*isError=*/false)};`.
  **L3447 CN**: 注释说明附近代码的逻辑、意图或元数据：`proc->procInterface(), /*isError=*/false)};`。
- **L3448 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3448 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bad.AttachTo(msgs.Say(symbol.name(),`.
  **L3449 CN**: 继续一个多行参数列表、初始化器或聚合项：`bad.AttachTo(msgs.Say(symbol.name(),`。
- **L3450 EN**: Executes a standalone statement or declaration: `"An interoperable procedure must have an interoperable interface"_err_en_US));`.
  **L3450 CN**: 执行一条独立语句或声明：`"An interoperable procedure must have an interoperable interface"_err_en_US));`。
- **L3451 EN**: Transitions from the previous branch into the alternative path.
  **L3451 CN**: 从前一个分支过渡到备选路径。
- **L3452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3452 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。
- **L3453 EN**: Executes a standalone statement or declaration: `"An interoperable procedure should have an interface with the BIND attribute"_warn_en_US);`.
  **L3453 CN**: 执行一条独立语句或声明：`"An interoperable procedure should have an interface with the BIND attribute"_warn_en_US);`。
- **L3454 EN**: Closes the current lexical scope or compound statement.
  **L3454 CN**: 结束当前词法作用域或复合语句块。
- **L3455 EN**: Closes the current lexical scope or compound statement.
  **L3455 CN**: 结束当前词法作用域或复合语句块。
- **L3456 EN**: Transitions from the previous branch into an `else if` condition.
  **L3456 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 3457-3480

````cpp
    for (const Symbol *dummy : subp->dummyArgs()) {
      if (dummy) {
        parser::Messages dummyMsgs;
        if (dummy->has<ProcEntityDetails>() ||
            dummy->has<SubprogramDetails>()) {
          dummyMsgs = WhyNotInteroperableProcedure(*dummy, /*isError=*/false);
          if (dummyMsgs.empty() && !dummy->attrs().test(Attr::BIND_C)) {
            dummyMsgs.Say(dummy->name(),
                "A dummy procedure of an interoperable procedure should be BIND(C)"_warn_en_US);
          }
        } else if (dummy->has<ObjectEntityDetails>()) {
          // Emit only optional portability warnings for non-interoperable
          // types when the dummy argument is not VALUE and will be implemented
          // on the C side by either a cdesc_t * or a void *.  F'2023 18.3.7 (5)
          bool allowNonInteroperableType{!dummy->attrs().test(Attr::VALUE) &&
              (IsDescriptor(*dummy) || IsAssumedType(*dummy))};
          dummyMsgs = WhyNotInteroperableObject(
              *dummy, allowNonInteroperableType, /*forCommonBlock=*/false);
        } else {
          CheckBindC(*dummy);
        }
        msgs.Annex(std::move(dummyMsgs));
      } else {
        msgs.Say(symbol.name(),
````
- **L3457 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3457 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3458 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3458 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3459 EN**: Executes a standalone statement or declaration: `parser::Messages dummyMsgs;`.
  **L3459 CN**: 执行一条独立语句或声明：`parser::Messages dummyMsgs;`。
- **L3460 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3460 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3461 EN**: Starts a function, method, lambda, or structured scope: `dummy->has<SubprogramDetails>()) {`.
  **L3461 CN**: 开始一个函数、方法、lambda 或结构化作用域：`dummy->has<SubprogramDetails>()) {`。
- **L3462 EN**: Executes a call or declaration centered on `WhyNotInteroperableProcedure`.
  **L3462 CN**: 执行以 `WhyNotInteroperableProcedure` 为核心的调用或声明。
- **L3463 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3463 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dummyMsgs.Say(dummy->name(),`.
  **L3464 CN**: 继续一个多行参数列表、初始化器或聚合项：`dummyMsgs.Say(dummy->name(),`。
- **L3465 EN**: Executes a call or declaration centered on `BIND`.
  **L3465 CN**: 执行以 `BIND` 为核心的调用或声明。
- **L3466 EN**: Closes the current lexical scope or compound statement.
  **L3466 CN**: 结束当前词法作用域或复合语句块。
- **L3467 EN**: Transitions from the previous branch into an `else if` condition.
  **L3467 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3468 EN**: Comment explains nearby logic, intent, or metadata: `Emit only optional portability warnings for non-interoperable`.
  **L3468 CN**: 注释说明附近代码的逻辑、意图或元数据：`Emit only optional portability warnings for non-interoperable`。
- **L3469 EN**: Comment explains nearby logic, intent, or metadata: `types when the dummy argument is not VALUE and will be implemented`.
  **L3469 CN**: 注释说明附近代码的逻辑、意图或元数据：`types when the dummy argument is not VALUE and will be implemented`。
- **L3470 EN**: Comment explains nearby logic, intent, or metadata: `on the C side by either a cdesc_t * or a void *.  F'2023 18.3.7 (5)`.
  **L3470 CN**: 注释说明附近代码的逻辑、意图或元数据：`on the C side by either a cdesc_t * or a void *.  F'2023 18.3.7 (5)`。
- **L3471 EN**: Continues logic associated with callable symbol `attrs`.
  **L3471 CN**: 继续与可调用符号 `attrs` 相关的逻辑。
- **L3472 EN**: Executes a call or declaration centered on `statement`.
  **L3472 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3473 EN**: Continues logic associated with callable symbol `WhyNotInteroperableObject`.
  **L3473 CN**: 继续与可调用符号 `WhyNotInteroperableObject` 相关的逻辑。
- **L3474 EN**: Comment explains nearby logic, intent, or metadata: `dummy, allowNonInteroperableType, /*forCommonBlock=*/false);`.
  **L3474 CN**: 注释说明附近代码的逻辑、意图或元数据：`dummy, allowNonInteroperableType, /*forCommonBlock=*/false);`。
- **L3475 EN**: Transitions from the previous branch into the alternative path.
  **L3475 CN**: 从前一个分支过渡到备选路径。
- **L3476 EN**: Executes a call or declaration centered on `CheckBindC`.
  **L3476 CN**: 执行以 `CheckBindC` 为核心的调用或声明。
- **L3477 EN**: Closes the current lexical scope or compound statement.
  **L3477 CN**: 结束当前词法作用域或复合语句块。
- **L3478 EN**: Executes a call or declaration centered on `msgs.Annex`.
  **L3478 CN**: 执行以 `msgs.Annex` 为核心的调用或声明。
- **L3479 EN**: Transitions from the previous branch into the alternative path.
  **L3479 CN**: 从前一个分支过渡到备选路径。
- **L3480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(symbol.name(),`.
  **L3480 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(symbol.name(),`。

### Lines 3481-3504

````cpp
            "A subprogram interface with the BIND attribute may not have an alternate return argument"_err_en_US);
      }
    }
    if (subp->isFunction()) {
      if (subp->result().has<ObjectEntityDetails>()) {
        msgs.Annex(WhyNotInteroperableFunctionResult(subp->result()));
      } else {
        msgs.Say(subp->result().name(),
            "The result of an interoperable function must be a data object"_err_en_US);
      }
    }
  }
  if (msgs.AnyFatalError()) {
    examinedByWhyNotInteroperable_.erase(symbol);
  }
  return msgs;
}

void CheckHelper::CheckBindC(const Symbol &symbol) {
  bool isExplicitBindC{symbol.attrs().test(Attr::BIND_C)};
  if (isExplicitBindC) {
    CheckConflicting(symbol, Attr::BIND_C, Attr::ELEMENTAL);
    CheckConflicting(symbol, Attr::BIND_C, Attr::INTRINSIC);
    CheckConflicting(symbol, Attr::BIND_C, Attr::PARAMETER);
````
- **L3481 EN**: Executes a standalone statement or declaration: `"A subprogram interface with the BIND attribute may not have an alternate return argument"_err_en_US);`.
  **L3481 CN**: 执行一条独立语句或声明：`"A subprogram interface with the BIND attribute may not have an alternate return argument"_err_en_US);`。
- **L3482 EN**: Closes the current lexical scope or compound statement.
  **L3482 CN**: 结束当前词法作用域或复合语句块。
- **L3483 EN**: Closes the current lexical scope or compound statement.
  **L3483 CN**: 结束当前词法作用域或复合语句块。
- **L3484 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3484 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3485 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3485 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3486 EN**: Executes a call or declaration centered on `msgs.Annex`.
  **L3486 CN**: 执行以 `msgs.Annex` 为核心的调用或声明。
- **L3487 EN**: Transitions from the previous branch into the alternative path.
  **L3487 CN**: 从前一个分支过渡到备选路径。
- **L3488 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msgs.Say(subp->result().name(),`.
  **L3488 CN**: 继续一个多行参数列表、初始化器或聚合项：`msgs.Say(subp->result().name(),`。
- **L3489 EN**: Executes a standalone statement or declaration: `"The result of an interoperable function must be a data object"_err_en_US);`.
  **L3489 CN**: 执行一条独立语句或声明：`"The result of an interoperable function must be a data object"_err_en_US);`。
- **L3490 EN**: Closes the current lexical scope or compound statement.
  **L3490 CN**: 结束当前词法作用域或复合语句块。
- **L3491 EN**: Closes the current lexical scope or compound statement.
  **L3491 CN**: 结束当前词法作用域或复合语句块。
- **L3492 EN**: Closes the current lexical scope or compound statement.
  **L3492 CN**: 结束当前词法作用域或复合语句块。
- **L3493 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3493 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3494 EN**: Executes a call or declaration centered on `examinedByWhyNotInteroperable_.erase`.
  **L3494 CN**: 执行以 `examinedByWhyNotInteroperable_.erase` 为核心的调用或声明。
- **L3495 EN**: Closes the current lexical scope or compound statement.
  **L3495 CN**: 结束当前词法作用域或复合语句块。
- **L3496 EN**: Returns from the current function with `msgs`.
  **L3496 CN**: 以 `msgs` 从当前函数返回。
- **L3497 EN**: Closes the current lexical scope or compound statement.
  **L3497 CN**: 结束当前词法作用域或复合语句块。
- **L3498 EN**: Blank line separating nearby declarations or logic blocks.
  **L3498 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3499 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckBindC(const Symbol &symbol) {`.
  **L3499 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckBindC(const Symbol &symbol) {`。
- **L3500 EN**: Executes a call or declaration centered on `isExplicitBindC{symbol.attrs`.
  **L3500 CN**: 执行以 `isExplicitBindC{symbol.attrs` 为核心的调用或声明。
- **L3501 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3501 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3502 EN**: Executes a call or declaration centered on `CheckConflicting`.
  **L3502 CN**: 执行以 `CheckConflicting` 为核心的调用或声明。
- **L3503 EN**: Executes a call or declaration centered on `CheckConflicting`.
  **L3503 CN**: 执行以 `CheckConflicting` 为核心的调用或声明。
- **L3504 EN**: Executes a call or declaration centered on `CheckConflicting`.
  **L3504 CN**: 执行以 `CheckConflicting` 为核心的调用或声明。

### Lines 3505-3528

````cpp
  } else {
    // symbol must be interoperable (e.g., dummy argument of interoperable
    // procedure interface) but is not itself BIND(C).
  }
  parser::Messages whyNot;
  if (const std::string * bindName{symbol.GetBindName()};
      bindName) { // has a binding name
    if (!bindName->empty()) {
      bool ok{bindName->front() == '_' || parser::IsLetter(bindName->front())};
      for (char ch : *bindName) {
        ok &= ch == '_' || parser::IsLetter(ch) || parser::IsDecimalDigit(ch);
      }
      if (!ok) {
        messages_.Say(symbol.name(),
            "Symbol has a BIND(C) name that is not a valid C language identifier"_err_en_US);
        context_.SetError(symbol);
      }
    }
  }
  if (symbol.GetIsExplicitBindName()) { // BIND(C,NAME=...); C1552, C1529
    auto defClass{ClassifyProcedure(symbol)};
    if (IsProcedurePointer(symbol)) {
      messages_.Say(symbol.name(),
          "A procedure pointer may not have a BIND attribute with a name"_err_en_US);
````
- **L3505 EN**: Transitions from the previous branch into the alternative path.
  **L3505 CN**: 从前一个分支过渡到备选路径。
- **L3506 EN**: Comment explains nearby logic, intent, or metadata: `symbol must be interoperable (e.g., dummy argument of interoperable`.
  **L3506 CN**: 注释说明附近代码的逻辑、意图或元数据：`symbol must be interoperable (e.g., dummy argument of interoperable`。
- **L3507 EN**: Comment explains nearby logic, intent, or metadata: `procedure interface) but is not itself BIND(C).`.
  **L3507 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedure interface) but is not itself BIND(C).`。
- **L3508 EN**: Closes the current lexical scope or compound statement.
  **L3508 CN**: 结束当前词法作用域或复合语句块。
- **L3509 EN**: Executes a standalone statement or declaration: `parser::Messages whyNot;`.
  **L3509 CN**: 执行一条独立语句或声明：`parser::Messages whyNot;`。
- **L3510 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3510 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3511 EN**: Continues the surrounding expression or declaration: `bindName) { // has a binding name`.
  **L3511 CN**: 继续构造周围的表达式或声明：`bindName) { // has a binding name`。
- **L3512 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3512 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3513 EN**: Executes a call or declaration centered on `ok{bindName->front`.
  **L3513 CN**: 执行以 `ok{bindName->front` 为核心的调用或声明。
- **L3514 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3514 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3515 EN**: Executes a call or declaration centered on `parser::IsLetter`.
  **L3515 CN**: 执行以 `parser::IsLetter` 为核心的调用或声明。
- **L3516 EN**: Closes the current lexical scope or compound statement.
  **L3516 CN**: 结束当前词法作用域或复合语句块。
- **L3517 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3517 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(symbol.name(),`.
  **L3518 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(symbol.name(),`。
- **L3519 EN**: Executes a call or declaration centered on `BIND`.
  **L3519 CN**: 执行以 `BIND` 为核心的调用或声明。
- **L3520 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L3520 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L3521 EN**: Closes the current lexical scope or compound statement.
  **L3521 CN**: 结束当前词法作用域或复合语句块。
- **L3522 EN**: Closes the current lexical scope or compound statement.
  **L3522 CN**: 结束当前词法作用域或复合语句块。
- **L3523 EN**: Closes the current lexical scope or compound statement.
  **L3523 CN**: 结束当前词法作用域或复合语句块。
- **L3524 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3524 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3525 EN**: Executes a call or declaration centered on `defClass{ClassifyProcedure`.
  **L3525 CN**: 执行以 `defClass{ClassifyProcedure` 为核心的调用或声明。
- **L3526 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3526 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(symbol.name(),`.
  **L3527 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(symbol.name(),`。
- **L3528 EN**: Executes a standalone statement or declaration: `"A procedure pointer may not have a BIND attribute with a name"_err_en_US);`.
  **L3528 CN**: 执行一条独立语句或声明：`"A procedure pointer may not have a BIND attribute with a name"_err_en_US);`。

### Lines 3529-3552

````cpp
      context_.SetError(symbol);
    } else if (defClass == ProcedureDefinitionClass::None ||
        IsExternal(symbol)) {
    } else if (symbol.attrs().test(Attr::ABSTRACT)) {
      messages_.Say(symbol.name(),
          "An ABSTRACT interface may not have a BIND attribute with a name"_err_en_US);
      context_.SetError(symbol);
    } else if (defClass == ProcedureDefinitionClass::Internal ||
        defClass == ProcedureDefinitionClass::Dummy) {
      messages_.Say(symbol.name(),
          "An internal or dummy procedure may not have a BIND(C,NAME=) binding label"_err_en_US);
      context_.SetError(symbol);
    }
  }
  // F2023 C1807 - a procedure defined in a submodule shall not have a binding
  // label unless its interface is declared in the ancestor module.
  const std::string *bindName{symbol.GetBindName()};
  if (symbol.has<SubprogramDetails>() &&
      !symbol.get<SubprogramDetails>().isInterface() && bindName &&
      !bindName->empty() && symbol.owner().IsSubmodule()) {
    const Symbol *iface{FindSeparateModuleSubprogramInterface(&symbol)};
    bool ok{false};
    if (iface) {
      const Scope *ifaceModule{FindModuleOrSubmoduleContaining(iface->owner())};
````
- **L3529 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L3529 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L3530 EN**: Transitions from the previous branch into an `else if` condition.
  **L3530 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3531 EN**: Starts a function, method, lambda, or structured scope: `IsExternal(symbol)) {`.
  **L3531 CN**: 开始一个函数、方法、lambda 或结构化作用域：`IsExternal(symbol)) {`。
- **L3532 EN**: Transitions from the previous branch into an `else if` condition.
  **L3532 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3533 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(symbol.name(),`.
  **L3533 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(symbol.name(),`。
- **L3534 EN**: Executes a standalone statement or declaration: `"An ABSTRACT interface may not have a BIND attribute with a name"_err_en_US);`.
  **L3534 CN**: 执行一条独立语句或声明：`"An ABSTRACT interface may not have a BIND attribute with a name"_err_en_US);`。
- **L3535 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L3535 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L3536 EN**: Transitions from the previous branch into an `else if` condition.
  **L3536 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3537 EN**: Continues the surrounding expression or declaration: `defClass == ProcedureDefinitionClass::Dummy) {`.
  **L3537 CN**: 继续构造周围的表达式或声明：`defClass == ProcedureDefinitionClass::Dummy) {`。
- **L3538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(symbol.name(),`.
  **L3538 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(symbol.name(),`。
- **L3539 EN**: Executes a call or declaration centered on `BIND`.
  **L3539 CN**: 执行以 `BIND` 为核心的调用或声明。
- **L3540 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L3540 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L3541 EN**: Closes the current lexical scope or compound statement.
  **L3541 CN**: 结束当前词法作用域或复合语句块。
- **L3542 EN**: Closes the current lexical scope or compound statement.
  **L3542 CN**: 结束当前词法作用域或复合语句块。
- **L3543 EN**: Comment explains nearby logic, intent, or metadata: `F2023 C1807 - a procedure defined in a submodule shall not have a binding`.
  **L3543 CN**: 注释说明附近代码的逻辑、意图或元数据：`F2023 C1807 - a procedure defined in a submodule shall not have a binding`。
- **L3544 EN**: Comment explains nearby logic, intent, or metadata: `label unless its interface is declared in the ancestor module.`.
  **L3544 CN**: 注释说明附近代码的逻辑、意图或元数据：`label unless its interface is declared in the ancestor module.`。
- **L3545 EN**: Executes a call or declaration centered on `*bindName{symbol.GetBindName`.
  **L3545 CN**: 执行以 `*bindName{symbol.GetBindName` 为核心的调用或声明。
- **L3546 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3546 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3547 EN**: Continues logic associated with callable symbol `get<SubprogramDetails>`.
  **L3547 CN**: 继续与可调用符号 `get<SubprogramDetails>` 相关的逻辑。
- **L3548 EN**: Starts a function, method, lambda, or structured scope: `!bindName->empty() && symbol.owner().IsSubmodule()) {`.
  **L3548 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!bindName->empty() && symbol.owner().IsSubmodule()) {`。
- **L3549 EN**: Executes a call or declaration centered on `*iface{FindSeparateModuleSubprogramInterface`.
  **L3549 CN**: 执行以 `*iface{FindSeparateModuleSubprogramInterface` 为核心的调用或声明。
- **L3550 EN**: Executes a standalone statement or declaration: `bool ok{false};`.
  **L3550 CN**: 执行一条独立语句或声明：`bool ok{false};`。
- **L3551 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3551 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3552 EN**: Executes a call or declaration centered on `*ifaceModule{FindModuleOrSubmoduleContaining`.
  **L3552 CN**: 执行以 `*ifaceModule{FindModuleOrSubmoduleContaining` 为核心的调用或声明。

### Lines 3553-3576

````cpp
      ok = ifaceModule && ifaceModule->IsModule();
    }
    if (!ok) {
      messages_.Say(symbol.name(),
          "A procedure defined in a submodule shall not have a binding label unless its interface is declared in the ancestor module"_err_en_US);
      context_.SetError(symbol);
    }
  }
  if (symbol.has<ObjectEntityDetails>()) {
    whyNot = WhyNotInteroperableObject(symbol);
  } else if (symbol.has<ProcEntityDetails>() ||
      symbol.has<SubprogramDetails>()) {
    whyNot = WhyNotInteroperableProcedure(symbol, /*isError=*/isExplicitBindC);
  } else if (symbol.has<DerivedTypeDetails>()) {
    whyNot = WhyNotInteroperableDerivedType(symbol);
  }
  if (!whyNot.empty()) {
    bool anyFatal{whyNot.AnyFatalError()};
    if (anyFatal ||
        (!InModuleFile() &&
            context_.ShouldWarn(
                common::LanguageFeature::NonBindCInteroperability))) {
      context_.messages().Annex(std::move(whyNot));
    }
````
- **L3553 EN**: Executes a call or declaration centered on `ifaceModule->IsModule`.
  **L3553 CN**: 执行以 `ifaceModule->IsModule` 为核心的调用或声明。
- **L3554 EN**: Closes the current lexical scope or compound statement.
  **L3554 CN**: 结束当前词法作用域或复合语句块。
- **L3555 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3555 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(symbol.name(),`.
  **L3556 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(symbol.name(),`。
- **L3557 EN**: Executes a standalone statement or declaration: `"A procedure defined in a submodule shall not have a binding label unless its interface is declared in the ancestor module"_err_en_US);`.
  **L3557 CN**: 执行一条独立语句或声明：`"A procedure defined in a submodule shall not have a binding label unless its interface is declared in the ancestor module"_err_en_US);`。
- **L3558 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L3558 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L3559 EN**: Closes the current lexical scope or compound statement.
  **L3559 CN**: 结束当前词法作用域或复合语句块。
- **L3560 EN**: Closes the current lexical scope or compound statement.
  **L3560 CN**: 结束当前词法作用域或复合语句块。
- **L3561 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3561 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3562 EN**: Executes a call or declaration centered on `WhyNotInteroperableObject`.
  **L3562 CN**: 执行以 `WhyNotInteroperableObject` 为核心的调用或声明。
- **L3563 EN**: Transitions from the previous branch into an `else if` condition.
  **L3563 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3564 EN**: Starts a function, method, lambda, or structured scope: `symbol.has<SubprogramDetails>()) {`.
  **L3564 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.has<SubprogramDetails>()) {`。
- **L3565 EN**: Executes a call or declaration centered on `WhyNotInteroperableProcedure`.
  **L3565 CN**: 执行以 `WhyNotInteroperableProcedure` 为核心的调用或声明。
- **L3566 EN**: Transitions from the previous branch into an `else if` condition.
  **L3566 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3567 EN**: Executes a call or declaration centered on `WhyNotInteroperableDerivedType`.
  **L3567 CN**: 执行以 `WhyNotInteroperableDerivedType` 为核心的调用或声明。
- **L3568 EN**: Closes the current lexical scope or compound statement.
  **L3568 CN**: 结束当前词法作用域或复合语句块。
- **L3569 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3569 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3570 EN**: Executes a call or declaration centered on `anyFatal{whyNot.AnyFatalError`.
  **L3570 CN**: 执行以 `anyFatal{whyNot.AnyFatalError` 为核心的调用或声明。
- **L3571 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3571 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3572 EN**: Continues logic associated with callable symbol `InModuleFile`.
  **L3572 CN**: 继续与可调用符号 `InModuleFile` 相关的逻辑。
- **L3573 EN**: Continues logic associated with callable symbol `ShouldWarn`.
  **L3573 CN**: 继续与可调用符号 `ShouldWarn` 相关的逻辑。
- **L3574 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::NonBindCInteroperability))) {`.
  **L3574 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::NonBindCInteroperability))) {`。
- **L3575 EN**: Executes a call or declaration centered on `context_.messages`.
  **L3575 CN**: 执行以 `context_.messages` 为核心的调用或声明。
- **L3576 EN**: Closes the current lexical scope or compound statement.
  **L3576 CN**: 结束当前词法作用域或复合语句块。

### Lines 3577-3600

````cpp
    if (anyFatal) {
      context_.SetError(symbol);
    }
  }
}

bool CheckHelper::CheckDioDummyIsData(
    const Symbol &subp, const Symbol *arg, std::size_t position) {
  if (arg && arg->detailsIf<ObjectEntityDetails>()) {
    if (IsAssumedRank(*arg)) {
      messages_.Say(arg->name(),
          "Dummy argument '%s' may not be assumed-rank"_err_en_US, arg->name());
      return false;
    } else {
      return true;
    }
  } else {
    if (arg) {
      messages_.Say(arg->name(),
          "Dummy argument '%s' must be a data object"_err_en_US, arg->name());
    } else {
      messages_.Say(subp.name(),
          "Dummy argument %d of '%s' must be a data object"_err_en_US, position,
          subp.name());
````
- **L3577 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3577 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3578 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L3578 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L3579 EN**: Closes the current lexical scope or compound statement.
  **L3579 CN**: 结束当前词法作用域或复合语句块。
- **L3580 EN**: Closes the current lexical scope or compound statement.
  **L3580 CN**: 结束当前词法作用域或复合语句块。
- **L3581 EN**: Closes the current lexical scope or compound statement.
  **L3581 CN**: 结束当前词法作用域或复合语句块。
- **L3582 EN**: Blank line separating nearby declarations or logic blocks.
  **L3582 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3583 EN**: Continues logic associated with callable symbol `CheckDioDummyIsData`.
  **L3583 CN**: 继续与可调用符号 `CheckDioDummyIsData` 相关的逻辑。
- **L3584 EN**: Continues the surrounding expression or declaration: `const Symbol &subp, const Symbol *arg, std::size_t position) {`.
  **L3584 CN**: 继续构造周围的表达式或声明：`const Symbol &subp, const Symbol *arg, std::size_t position) {`。
- **L3585 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3585 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3586 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3586 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(arg->name(),`.
  **L3587 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(arg->name(),`。
- **L3588 EN**: Executes a call or declaration centered on `arg->name`.
  **L3588 CN**: 执行以 `arg->name` 为核心的调用或声明。
- **L3589 EN**: Returns from the current function with `false`.
  **L3589 CN**: 以 `false` 从当前函数返回。
- **L3590 EN**: Transitions from the previous branch into the alternative path.
  **L3590 CN**: 从前一个分支过渡到备选路径。
- **L3591 EN**: Returns from the current function with `true`.
  **L3591 CN**: 以 `true` 从当前函数返回。
- **L3592 EN**: Closes the current lexical scope or compound statement.
  **L3592 CN**: 结束当前词法作用域或复合语句块。
- **L3593 EN**: Transitions from the previous branch into the alternative path.
  **L3593 CN**: 从前一个分支过渡到备选路径。
- **L3594 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3594 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(arg->name(),`.
  **L3595 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(arg->name(),`。
- **L3596 EN**: Executes a call or declaration centered on `arg->name`.
  **L3596 CN**: 执行以 `arg->name` 为核心的调用或声明。
- **L3597 EN**: Transitions from the previous branch into the alternative path.
  **L3597 CN**: 从前一个分支过渡到备选路径。
- **L3598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(subp.name(),`.
  **L3598 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(subp.name(),`。
- **L3599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy argument %d of '%s' must be a data object"_err_en_US, position,`.
  **L3599 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy argument %d of '%s' must be a data object"_err_en_US, position,`。
- **L3600 EN**: Executes a call or declaration centered on `subp.name`.
  **L3600 CN**: 执行以 `subp.name` 为核心的调用或声明。

### Lines 3601-3624

````cpp
    }
    return false;
  }
}

void CheckHelper::CheckAlreadySeenDefinedIo(const DerivedTypeSpec &derivedType,
    common::DefinedIo ioKind, const Symbol &proc, const Symbol &generic) {
  // Check for conflict between non-type-bound defined I/O and type-bound
  // generics. It's okay to have two or more distinct defined I/O procedures for
  // the same type if they're coming from distinct non-type-bound interfaces.
  // (The non-type-bound interfaces would have been merged into a single generic
  //  -- with errors where indistinguishable --  when both were visible from the
  // same scope.)
  if (generic.owner().IsDerivedType()) {
    return;
  }
  if (const Scope * dtScope{derivedType.scope()}) {
    if (auto iter{dtScope->find(generic.name())}; iter != dtScope->end() &&
        IsAccessible(*iter->second, generic.owner())) {
      for (auto specRef : iter->second->get<GenericDetails>().specificProcs()) {
        const Symbol *specific{&specRef->get<ProcBindingDetails>().symbol()};
        if (specific == &proc) {
          continue; // unambiguous, accept
        }
````
- **L3601 EN**: Closes the current lexical scope or compound statement.
  **L3601 CN**: 结束当前词法作用域或复合语句块。
- **L3602 EN**: Returns from the current function with `false`.
  **L3602 CN**: 以 `false` 从当前函数返回。
- **L3603 EN**: Closes the current lexical scope or compound statement.
  **L3603 CN**: 结束当前词法作用域或复合语句块。
- **L3604 EN**: Closes the current lexical scope or compound statement.
  **L3604 CN**: 结束当前词法作用域或复合语句块。
- **L3605 EN**: Blank line separating nearby declarations or logic blocks.
  **L3605 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckHelper::CheckAlreadySeenDefinedIo(const DerivedTypeSpec &derivedType,`.
  **L3606 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckHelper::CheckAlreadySeenDefinedIo(const DerivedTypeSpec &derivedType,`。
- **L3607 EN**: Continues the surrounding expression or declaration: `common::DefinedIo ioKind, const Symbol &proc, const Symbol &generic) {`.
  **L3607 CN**: 继续构造周围的表达式或声明：`common::DefinedIo ioKind, const Symbol &proc, const Symbol &generic) {`。
- **L3608 EN**: Comment explains nearby logic, intent, or metadata: `Check for conflict between non-type-bound defined I/O and type-bound`.
  **L3608 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check for conflict between non-type-bound defined I/O and type-bound`。
- **L3609 EN**: Comment explains nearby logic, intent, or metadata: `generics. It's okay to have two or more distinct defined I/O procedures for`.
  **L3609 CN**: 注释说明附近代码的逻辑、意图或元数据：`generics. It's okay to have two or more distinct defined I/O procedures for`。
- **L3610 EN**: Comment explains nearby logic, intent, or metadata: `the same type if they're coming from distinct non-type-bound interfaces.`.
  **L3610 CN**: 注释说明附近代码的逻辑、意图或元数据：`the same type if they're coming from distinct non-type-bound interfaces.`。
- **L3611 EN**: Comment explains nearby logic, intent, or metadata: `(The non-type-bound interfaces would have been merged into a single generic`.
  **L3611 CN**: 注释说明附近代码的逻辑、意图或元数据：`(The non-type-bound interfaces would have been merged into a single generic`。
- **L3612 EN**: Comment explains nearby logic, intent, or metadata: `-- with errors where indistinguishable --  when both were visible from the`.
  **L3612 CN**: 注释说明附近代码的逻辑、意图或元数据：`-- with errors where indistinguishable --  when both were visible from the`。
- **L3613 EN**: Comment explains nearby logic, intent, or metadata: `same scope.)`.
  **L3613 CN**: 注释说明附近代码的逻辑、意图或元数据：`same scope.)`。
- **L3614 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3614 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3615 EN**: Returns from the current function with `void`.
  **L3615 CN**: 以 `void` 从当前函数返回。
- **L3616 EN**: Closes the current lexical scope or compound statement.
  **L3616 CN**: 结束当前词法作用域或复合语句块。
- **L3617 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3617 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3618 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3618 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3619 EN**: Starts a function, method, lambda, or structured scope: `IsAccessible(*iter->second, generic.owner())) {`.
  **L3619 CN**: 开始一个函数、方法、lambda 或结构化作用域：`IsAccessible(*iter->second, generic.owner())) {`。
- **L3620 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3620 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3621 EN**: Executes a call or declaration centered on `*specific{&specRef->get<ProcBindingDetails>`.
  **L3621 CN**: 执行以 `*specific{&specRef->get<ProcBindingDetails>` 为核心的调用或声明。
- **L3622 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3622 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3623 EN**: Skips to the next loop iteration.
  **L3623 CN**: 跳到下一次循环迭代。
- **L3624 EN**: Closes the current lexical scope or compound statement.
  **L3624 CN**: 结束当前词法作用域或复合语句块。

### Lines 3625-3648

````cpp
        if (const auto *peDetails{specific->detailsIf<ProcEntityDetails>()}) {
          specific = peDetails->procInterface();
          if (!specific) {
            continue;
          }
        }
        if (const auto *specDT{GetDtvArgDerivedType(*specific)};
            specDT && evaluate::AreSameDerivedType(derivedType, *specDT)) {
          SayWithDeclaration(*specRef, proc.name(),
              "Derived type '%s' has conflicting type-bound input/output procedure '%s'"_err_en_US,
              derivedType.name(), GenericKind::AsFortran(ioKind));
          return;
        }
      }
    }
  }
}

void CheckHelper::CheckDioDummyIsDerived(const Symbol &proc, const Symbol &arg,
    common::DefinedIo ioKind, const Symbol &generic) {
  if (const DeclTypeSpec *type{arg.GetType()}) {
    if (const DerivedTypeSpec *derivedType{type->AsDerived()}) {
      CheckAlreadySeenDefinedIo(*derivedType, ioKind, proc, generic);
      bool isPolymorphic{type->IsPolymorphic()};
````
- **L3625 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3625 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3626 EN**: Executes a call or declaration centered on `peDetails->procInterface`.
  **L3626 CN**: 执行以 `peDetails->procInterface` 为核心的调用或声明。
- **L3627 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3627 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3628 EN**: Skips to the next loop iteration.
  **L3628 CN**: 跳到下一次循环迭代。
- **L3629 EN**: Closes the current lexical scope or compound statement.
  **L3629 CN**: 结束当前词法作用域或复合语句块。
- **L3630 EN**: Closes the current lexical scope or compound statement.
  **L3630 CN**: 结束当前词法作用域或复合语句块。
- **L3631 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3631 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3632 EN**: Starts a function, method, lambda, or structured scope: `specDT && evaluate::AreSameDerivedType(derivedType, *specDT)) {`.
  **L3632 CN**: 开始一个函数、方法、lambda 或结构化作用域：`specDT && evaluate::AreSameDerivedType(derivedType, *specDT)) {`。
- **L3633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(*specRef, proc.name(),`.
  **L3633 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(*specRef, proc.name(),`。
- **L3634 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Derived type '%s' has conflicting type-bound input/output procedure '%s'"_err_en_US,`.
  **L3634 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Derived type '%s' has conflicting type-bound input/output procedure '%s'"_err_en_US,`。
- **L3635 EN**: Executes a call or declaration centered on `derivedType.name`.
  **L3635 CN**: 执行以 `derivedType.name` 为核心的调用或声明。
- **L3636 EN**: Returns from the current function with `void`.
  **L3636 CN**: 以 `void` 从当前函数返回。
- **L3637 EN**: Closes the current lexical scope or compound statement.
  **L3637 CN**: 结束当前词法作用域或复合语句块。
- **L3638 EN**: Closes the current lexical scope or compound statement.
  **L3638 CN**: 结束当前词法作用域或复合语句块。
- **L3639 EN**: Closes the current lexical scope or compound statement.
  **L3639 CN**: 结束当前词法作用域或复合语句块。
- **L3640 EN**: Closes the current lexical scope or compound statement.
  **L3640 CN**: 结束当前词法作用域或复合语句块。
- **L3641 EN**: Closes the current lexical scope or compound statement.
  **L3641 CN**: 结束当前词法作用域或复合语句块。
- **L3642 EN**: Blank line separating nearby declarations or logic blocks.
  **L3642 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckHelper::CheckDioDummyIsDerived(const Symbol &proc, const Symbol &arg,`.
  **L3643 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckHelper::CheckDioDummyIsDerived(const Symbol &proc, const Symbol &arg,`。
- **L3644 EN**: Continues the surrounding expression or declaration: `common::DefinedIo ioKind, const Symbol &generic) {`.
  **L3644 CN**: 继续构造周围的表达式或声明：`common::DefinedIo ioKind, const Symbol &generic) {`。
- **L3645 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3645 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3646 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3646 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3647 EN**: Executes a call or declaration centered on `CheckAlreadySeenDefinedIo`.
  **L3647 CN**: 执行以 `CheckAlreadySeenDefinedIo` 为核心的调用或声明。
- **L3648 EN**: Executes a call or declaration centered on `isPolymorphic{type->IsPolymorphic`.
  **L3648 CN**: 执行以 `isPolymorphic{type->IsPolymorphic` 为核心的调用或声明。

### Lines 3649-3672

````cpp
      if (isPolymorphic != IsExtensibleType(derivedType)) {
        messages_.Say(arg.name(),
            "Dummy argument '%s' of a defined input/output procedure must be %s when the derived type is %s"_err_en_US,
            arg.name(), isPolymorphic ? "TYPE()" : "CLASS()",
            isPolymorphic ? "not extensible" : "extensible");
      }
    } else {
      messages_.Say(arg.name(),
          "Dummy argument '%s' of a defined input/output procedure must have a derived type"_err_en_US,
          arg.name());
    }
  }
}

void CheckHelper::CheckDioDummyIsDefaultInteger(
    const Symbol &subp, const Symbol &arg) {
  if (const DeclTypeSpec *type{arg.GetType()};
      type && type->IsNumeric(TypeCategory::Integer)) {
    if (const auto kind{evaluate::ToInt64(type->numericTypeSpec().kind())};
        kind && *kind == context_.GetDefaultKind(TypeCategory::Integer)) {
      return;
    }
  }
  messages_.Say(arg.name(),
````
- **L3649 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3649 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(arg.name(),`.
  **L3650 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(arg.name(),`。
- **L3651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy argument '%s' of a defined input/output procedure must be %s when the derived type is %s"_err_en_US,`.
  **L3651 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy argument '%s' of a defined input/output procedure must be %s when the derived type is %s"_err_en_US,`。
- **L3652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arg.name(), isPolymorphic ? "TYPE()" : "CLASS()",`.
  **L3652 CN**: 继续一个多行参数列表、初始化器或聚合项：`arg.name(), isPolymorphic ? "TYPE()" : "CLASS()",`。
- **L3653 EN**: Executes a standalone statement or declaration: `isPolymorphic ? "not extensible" : "extensible");`.
  **L3653 CN**: 执行一条独立语句或声明：`isPolymorphic ? "not extensible" : "extensible");`。
- **L3654 EN**: Closes the current lexical scope or compound statement.
  **L3654 CN**: 结束当前词法作用域或复合语句块。
- **L3655 EN**: Transitions from the previous branch into the alternative path.
  **L3655 CN**: 从前一个分支过渡到备选路径。
- **L3656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(arg.name(),`.
  **L3656 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(arg.name(),`。
- **L3657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy argument '%s' of a defined input/output procedure must have a derived type"_err_en_US,`.
  **L3657 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy argument '%s' of a defined input/output procedure must have a derived type"_err_en_US,`。
- **L3658 EN**: Executes a call or declaration centered on `arg.name`.
  **L3658 CN**: 执行以 `arg.name` 为核心的调用或声明。
- **L3659 EN**: Closes the current lexical scope or compound statement.
  **L3659 CN**: 结束当前词法作用域或复合语句块。
- **L3660 EN**: Closes the current lexical scope or compound statement.
  **L3660 CN**: 结束当前词法作用域或复合语句块。
- **L3661 EN**: Closes the current lexical scope or compound statement.
  **L3661 CN**: 结束当前词法作用域或复合语句块。
- **L3662 EN**: Blank line separating nearby declarations or logic blocks.
  **L3662 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3663 EN**: Continues logic associated with callable symbol `CheckDioDummyIsDefaultInteger`.
  **L3663 CN**: 继续与可调用符号 `CheckDioDummyIsDefaultInteger` 相关的逻辑。
- **L3664 EN**: Continues the surrounding expression or declaration: `const Symbol &subp, const Symbol &arg) {`.
  **L3664 CN**: 继续构造周围的表达式或声明：`const Symbol &subp, const Symbol &arg) {`。
- **L3665 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3665 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3666 EN**: Starts a function, method, lambda, or structured scope: `type && type->IsNumeric(TypeCategory::Integer)) {`.
  **L3666 CN**: 开始一个函数、方法、lambda 或结构化作用域：`type && type->IsNumeric(TypeCategory::Integer)) {`。
- **L3667 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3667 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3668 EN**: Starts a function, method, lambda, or structured scope: `kind && *kind == context_.GetDefaultKind(TypeCategory::Integer)) {`.
  **L3668 CN**: 开始一个函数、方法、lambda 或结构化作用域：`kind && *kind == context_.GetDefaultKind(TypeCategory::Integer)) {`。
- **L3669 EN**: Returns from the current function with `void`.
  **L3669 CN**: 以 `void` 从当前函数返回。
- **L3670 EN**: Closes the current lexical scope or compound statement.
  **L3670 CN**: 结束当前词法作用域或复合语句块。
- **L3671 EN**: Closes the current lexical scope or compound statement.
  **L3671 CN**: 结束当前词法作用域或复合语句块。
- **L3672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(arg.name(),`.
  **L3672 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(arg.name(),`。

### Lines 3673-3696

````cpp
      "Dummy argument '%s' of a defined input/output procedure must be an INTEGER of default KIND"_err_en_US,
      arg.name());
}

void CheckHelper::CheckDioDummyIsScalar(const Symbol &subp, const Symbol &arg) {
  if (arg.Rank() > 0) {
    messages_.Say(arg.name(),
        "Dummy argument '%s' of a defined input/output procedure must be a scalar"_err_en_US,
        arg.name());
  }
}

void CheckHelper::CheckDioDtvArg(const Symbol &proc, const Symbol &subp,
    const Symbol *arg, common::DefinedIo ioKind, const Symbol &generic) {
  // Dtv argument looks like: dtv-type-spec, INTENT(INOUT) :: dtv
  if (CheckDioDummyIsData(subp, arg, 0)) {
    CheckDioDummyIsDerived(proc, *arg, ioKind, generic);
    CheckDioDummyAttrs(subp, *arg,
        ioKind == common::DefinedIo::ReadFormatted ||
                ioKind == common::DefinedIo::ReadUnformatted
            ? Attr::INTENT_INOUT
            : Attr::INTENT_IN);
    CheckDioDummyIsScalar(subp, *arg);
  }
````
- **L3673 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy argument '%s' of a defined input/output procedure must be an INTEGER of default KIND"_err_en_US,`.
  **L3673 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy argument '%s' of a defined input/output procedure must be an INTEGER of default KIND"_err_en_US,`。
- **L3674 EN**: Executes a call or declaration centered on `arg.name`.
  **L3674 CN**: 执行以 `arg.name` 为核心的调用或声明。
- **L3675 EN**: Closes the current lexical scope or compound statement.
  **L3675 CN**: 结束当前词法作用域或复合语句块。
- **L3676 EN**: Blank line separating nearby declarations or logic blocks.
  **L3676 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3677 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckDioDummyIsScalar(const Symbol &subp, const Symbol &arg) {`.
  **L3677 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckDioDummyIsScalar(const Symbol &subp, const Symbol &arg) {`。
- **L3678 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3678 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3679 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(arg.name(),`.
  **L3679 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(arg.name(),`。
- **L3680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy argument '%s' of a defined input/output procedure must be a scalar"_err_en_US,`.
  **L3680 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy argument '%s' of a defined input/output procedure must be a scalar"_err_en_US,`。
- **L3681 EN**: Executes a call or declaration centered on `arg.name`.
  **L3681 CN**: 执行以 `arg.name` 为核心的调用或声明。
- **L3682 EN**: Closes the current lexical scope or compound statement.
  **L3682 CN**: 结束当前词法作用域或复合语句块。
- **L3683 EN**: Closes the current lexical scope or compound statement.
  **L3683 CN**: 结束当前词法作用域或复合语句块。
- **L3684 EN**: Blank line separating nearby declarations or logic blocks.
  **L3684 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckHelper::CheckDioDtvArg(const Symbol &proc, const Symbol &subp,`.
  **L3685 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckHelper::CheckDioDtvArg(const Symbol &proc, const Symbol &subp,`。
- **L3686 EN**: Continues the surrounding expression or declaration: `const Symbol *arg, common::DefinedIo ioKind, const Symbol &generic) {`.
  **L3686 CN**: 继续构造周围的表达式或声明：`const Symbol *arg, common::DefinedIo ioKind, const Symbol &generic) {`。
- **L3687 EN**: Comment explains nearby logic, intent, or metadata: `Dtv argument looks like: dtv-type-spec, INTENT(INOUT) :: dtv`.
  **L3687 CN**: 注释说明附近代码的逻辑、意图或元数据：`Dtv argument looks like: dtv-type-spec, INTENT(INOUT) :: dtv`。
- **L3688 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3688 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3689 EN**: Executes a call or declaration centered on `CheckDioDummyIsDerived`.
  **L3689 CN**: 执行以 `CheckDioDummyIsDerived` 为核心的调用或声明。
- **L3690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckDioDummyAttrs(subp, *arg,`.
  **L3690 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckDioDummyAttrs(subp, *arg,`。
- **L3691 EN**: Continues the surrounding expression or declaration: `ioKind == common::DefinedIo::ReadFormatted ||`.
  **L3691 CN**: 继续构造周围的表达式或声明：`ioKind == common::DefinedIo::ReadFormatted ||`。
- **L3692 EN**: Continues the surrounding expression or declaration: `ioKind == common::DefinedIo::ReadUnformatted`.
  **L3692 CN**: 继续构造周围的表达式或声明：`ioKind == common::DefinedIo::ReadUnformatted`。
- **L3693 EN**: Continues the surrounding expression or declaration: `? Attr::INTENT_INOUT`.
  **L3693 CN**: 继续构造周围的表达式或声明：`? Attr::INTENT_INOUT`。
- **L3694 EN**: Executes a standalone statement or declaration: `: Attr::INTENT_IN);`.
  **L3694 CN**: 执行一条独立语句或声明：`: Attr::INTENT_IN);`。
- **L3695 EN**: Executes a call or declaration centered on `CheckDioDummyIsScalar`.
  **L3695 CN**: 执行以 `CheckDioDummyIsScalar` 为核心的调用或声明。
- **L3696 EN**: Closes the current lexical scope or compound statement.
  **L3696 CN**: 结束当前词法作用域或复合语句块。

### Lines 3697-3720

````cpp
}

// If an explicit INTRINSIC name is a function, so must all the specifics be,
// and similarly for subroutines
void CheckHelper::CheckGenericVsIntrinsic(
    const Symbol &symbol, const GenericDetails &generic) {
  if (symbol.attrs().test(Attr::INTRINSIC)) {
    const evaluate::IntrinsicProcTable &table{
        context_.foldingContext().intrinsics()};
    bool isSubroutine{table.IsIntrinsicSubroutine(symbol.name().ToString())};
    if (isSubroutine || table.IsIntrinsicFunction(symbol.name().ToString())) {
      for (const SymbolRef &ref : generic.specificProcs()) {
        const Symbol &ultimate{ref->GetUltimate()};
        bool specificFunc{ultimate.test(Symbol::Flag::Function)};
        bool specificSubr{ultimate.test(Symbol::Flag::Subroutine)};
        if (!specificFunc && !specificSubr) {
          if (const auto *proc{ultimate.detailsIf<SubprogramDetails>()}) {
            if (proc->isFunction()) {
              specificFunc = true;
            } else {
              specificSubr = true;
            }
          }
        }
````
- **L3697 EN**: Closes the current lexical scope or compound statement.
  **L3697 CN**: 结束当前词法作用域或复合语句块。
- **L3698 EN**: Blank line separating nearby declarations or logic blocks.
  **L3698 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3699 EN**: Comment explains nearby logic, intent, or metadata: `If an explicit INTRINSIC name is a function, so must all the specifics be,`.
  **L3699 CN**: 注释说明附近代码的逻辑、意图或元数据：`If an explicit INTRINSIC name is a function, so must all the specifics be,`。
- **L3700 EN**: Comment explains nearby logic, intent, or metadata: `and similarly for subroutines`.
  **L3700 CN**: 注释说明附近代码的逻辑、意图或元数据：`and similarly for subroutines`。
- **L3701 EN**: Continues logic associated with callable symbol `CheckGenericVsIntrinsic`.
  **L3701 CN**: 继续与可调用符号 `CheckGenericVsIntrinsic` 相关的逻辑。
- **L3702 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, const GenericDetails &generic) {`.
  **L3702 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, const GenericDetails &generic) {`。
- **L3703 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3703 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3704 EN**: Continues the surrounding expression or declaration: `const evaluate::IntrinsicProcTable &table{`.
  **L3704 CN**: 继续构造周围的表达式或声明：`const evaluate::IntrinsicProcTable &table{`。
- **L3705 EN**: Executes a call or declaration centered on `context_.foldingContext`.
  **L3705 CN**: 执行以 `context_.foldingContext` 为核心的调用或声明。
- **L3706 EN**: Executes a call or declaration centered on `isSubroutine{table.IsIntrinsicSubroutine`.
  **L3706 CN**: 执行以 `isSubroutine{table.IsIntrinsicSubroutine` 为核心的调用或声明。
- **L3707 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3707 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3708 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3708 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3709 EN**: Executes a call or declaration centered on `&ultimate{ref->GetUltimate`.
  **L3709 CN**: 执行以 `&ultimate{ref->GetUltimate` 为核心的调用或声明。
- **L3710 EN**: Executes a call or declaration centered on `specificFunc{ultimate.test`.
  **L3710 CN**: 执行以 `specificFunc{ultimate.test` 为核心的调用或声明。
- **L3711 EN**: Executes a call or declaration centered on `specificSubr{ultimate.test`.
  **L3711 CN**: 执行以 `specificSubr{ultimate.test` 为核心的调用或声明。
- **L3712 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3712 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3713 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3713 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3714 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3714 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3715 EN**: Executes a standalone statement or declaration: `specificFunc = true;`.
  **L3715 CN**: 执行一条独立语句或声明：`specificFunc = true;`。
- **L3716 EN**: Transitions from the previous branch into the alternative path.
  **L3716 CN**: 从前一个分支过渡到备选路径。
- **L3717 EN**: Executes a standalone statement or declaration: `specificSubr = true;`.
  **L3717 CN**: 执行一条独立语句或声明：`specificSubr = true;`。
- **L3718 EN**: Closes the current lexical scope or compound statement.
  **L3718 CN**: 结束当前词法作用域或复合语句块。
- **L3719 EN**: Closes the current lexical scope or compound statement.
  **L3719 CN**: 结束当前词法作用域或复合语句块。
- **L3720 EN**: Closes the current lexical scope or compound statement.
  **L3720 CN**: 结束当前词法作用域或复合语句块。

### Lines 3721-3744

````cpp
        if ((specificFunc || specificSubr) &&
            isSubroutine != specificSubr) { // C848
          messages_.Say(symbol.name(),
              "Generic interface '%s' with explicit intrinsic %s of the same name may not have specific procedure '%s' that is a %s"_err_en_US,
              symbol.name(), isSubroutine ? "subroutine" : "function",
              ref->name(), isSubroutine ? "function" : "subroutine");
        }
      }
    }
  }
}

void CheckHelper::CheckDefaultIntegerArg(
    const Symbol &subp, const Symbol *arg, Attr intent) {
  // Argument looks like: INTEGER, INTENT(intent) :: arg
  if (CheckDioDummyIsData(subp, arg, 1)) {
    CheckDioDummyIsDefaultInteger(subp, *arg);
    CheckDioDummyIsScalar(subp, *arg);
    CheckDioDummyAttrs(subp, *arg, intent);
  }
}

void CheckHelper::CheckDioAssumedLenCharacterArg(const Symbol &subp,
    const Symbol *arg, std::size_t argPosition, Attr intent) {
````
- **L3721 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3721 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3722 EN**: Continues the surrounding expression or declaration: `isSubroutine != specificSubr) { // C848`.
  **L3722 CN**: 继续构造周围的表达式或声明：`isSubroutine != specificSubr) { // C848`。
- **L3723 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(symbol.name(),`.
  **L3723 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(symbol.name(),`。
- **L3724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Generic interface '%s' with explicit intrinsic %s of the same name may not have specific procedure '%s' that is a %s"_err_en_US,`.
  **L3724 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Generic interface '%s' with explicit intrinsic %s of the same name may not have specific procedure '%s' that is a %s"_err_en_US,`。
- **L3725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol.name(), isSubroutine ? "subroutine" : "function",`.
  **L3725 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol.name(), isSubroutine ? "subroutine" : "function",`。
- **L3726 EN**: Executes a call or declaration centered on `ref->name`.
  **L3726 CN**: 执行以 `ref->name` 为核心的调用或声明。
- **L3727 EN**: Closes the current lexical scope or compound statement.
  **L3727 CN**: 结束当前词法作用域或复合语句块。
- **L3728 EN**: Closes the current lexical scope or compound statement.
  **L3728 CN**: 结束当前词法作用域或复合语句块。
- **L3729 EN**: Closes the current lexical scope or compound statement.
  **L3729 CN**: 结束当前词法作用域或复合语句块。
- **L3730 EN**: Closes the current lexical scope or compound statement.
  **L3730 CN**: 结束当前词法作用域或复合语句块。
- **L3731 EN**: Closes the current lexical scope or compound statement.
  **L3731 CN**: 结束当前词法作用域或复合语句块。
- **L3732 EN**: Blank line separating nearby declarations or logic blocks.
  **L3732 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3733 EN**: Continues logic associated with callable symbol `CheckDefaultIntegerArg`.
  **L3733 CN**: 继续与可调用符号 `CheckDefaultIntegerArg` 相关的逻辑。
- **L3734 EN**: Continues the surrounding expression or declaration: `const Symbol &subp, const Symbol *arg, Attr intent) {`.
  **L3734 CN**: 继续构造周围的表达式或声明：`const Symbol &subp, const Symbol *arg, Attr intent) {`。
- **L3735 EN**: Comment explains nearby logic, intent, or metadata: `Argument looks like: INTEGER, INTENT(intent) :: arg`.
  **L3735 CN**: 注释说明附近代码的逻辑、意图或元数据：`Argument looks like: INTEGER, INTENT(intent) :: arg`。
- **L3736 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3736 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3737 EN**: Executes a call or declaration centered on `CheckDioDummyIsDefaultInteger`.
  **L3737 CN**: 执行以 `CheckDioDummyIsDefaultInteger` 为核心的调用或声明。
- **L3738 EN**: Executes a call or declaration centered on `CheckDioDummyIsScalar`.
  **L3738 CN**: 执行以 `CheckDioDummyIsScalar` 为核心的调用或声明。
- **L3739 EN**: Executes a call or declaration centered on `CheckDioDummyAttrs`.
  **L3739 CN**: 执行以 `CheckDioDummyAttrs` 为核心的调用或声明。
- **L3740 EN**: Closes the current lexical scope or compound statement.
  **L3740 CN**: 结束当前词法作用域或复合语句块。
- **L3741 EN**: Closes the current lexical scope or compound statement.
  **L3741 CN**: 结束当前词法作用域或复合语句块。
- **L3742 EN**: Blank line separating nearby declarations or logic blocks.
  **L3742 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckHelper::CheckDioAssumedLenCharacterArg(const Symbol &subp,`.
  **L3743 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckHelper::CheckDioAssumedLenCharacterArg(const Symbol &subp,`。
- **L3744 EN**: Continues the surrounding expression or declaration: `const Symbol *arg, std::size_t argPosition, Attr intent) {`.
  **L3744 CN**: 继续构造周围的表达式或声明：`const Symbol *arg, std::size_t argPosition, Attr intent) {`。

### Lines 3745-3768

````cpp
  // Argument looks like: CHARACTER (LEN=*), INTENT(intent) :: (iotype OR iomsg)
  if (CheckDioDummyIsData(subp, arg, argPosition)) {
    CheckDioDummyAttrs(subp, *arg, intent);
    const DeclTypeSpec *type{arg ? arg->GetType() : nullptr};
    const IntrinsicTypeSpec *intrinsic{type ? type->AsIntrinsic() : nullptr};
    const auto kind{
        intrinsic ? evaluate::ToInt64(intrinsic->kind()) : std::nullopt};
    if (!IsAssumedLengthCharacter(*arg) ||
        (!kind ||
            *kind !=
                context_.defaultKinds().GetDefaultKind(
                    TypeCategory::Character))) {
      messages_.Say(arg->name(),
          "Dummy argument '%s' of a defined input/output procedure must be assumed-length CHARACTER of default kind"_err_en_US,
          arg->name());
    }
    CheckDioDummyIsScalar(subp, *arg);
  }
}

void CheckHelper::CheckDioVlistArg(
    const Symbol &subp, const Symbol *arg, std::size_t argPosition) {
  // Vlist argument looks like: INTEGER, INTENT(IN) :: v_list(:)
  if (CheckDioDummyIsData(subp, arg, argPosition)) {
````
- **L3745 EN**: Comment explains nearby logic, intent, or metadata: `Argument looks like: CHARACTER (LEN=*), INTENT(intent) :: (iotype OR iomsg)`.
  **L3745 CN**: 注释说明附近代码的逻辑、意图或元数据：`Argument looks like: CHARACTER (LEN=*), INTENT(intent) :: (iotype OR iomsg)`。
- **L3746 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3746 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3747 EN**: Executes a call or declaration centered on `CheckDioDummyAttrs`.
  **L3747 CN**: 执行以 `CheckDioDummyAttrs` 为核心的调用或声明。
- **L3748 EN**: Executes a call or declaration centered on `arg->GetType`.
  **L3748 CN**: 执行以 `arg->GetType` 为核心的调用或声明。
- **L3749 EN**: Executes a call or declaration centered on `type->AsIntrinsic`.
  **L3749 CN**: 执行以 `type->AsIntrinsic` 为核心的调用或声明。
- **L3750 EN**: Continues the surrounding expression or declaration: `const auto kind{`.
  **L3750 CN**: 继续构造周围的表达式或声明：`const auto kind{`。
- **L3751 EN**: Executes a call or declaration centered on `evaluate::ToInt64`.
  **L3751 CN**: 执行以 `evaluate::ToInt64` 为核心的调用或声明。
- **L3752 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3752 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3753 EN**: Continues the surrounding expression or declaration: `(!kind ||`.
  **L3753 CN**: 继续构造周围的表达式或声明：`(!kind ||`。
- **L3754 EN**: Comment explains nearby logic, intent, or metadata: `kind !=`.
  **L3754 CN**: 注释说明附近代码的逻辑、意图或元数据：`kind !=`。
- **L3755 EN**: Continues logic associated with callable symbol `defaultKinds`.
  **L3755 CN**: 继续与可调用符号 `defaultKinds` 相关的逻辑。
- **L3756 EN**: Continues the surrounding expression or declaration: `TypeCategory::Character))) {`.
  **L3756 CN**: 继续构造周围的表达式或声明：`TypeCategory::Character))) {`。
- **L3757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(arg->name(),`.
  **L3757 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(arg->name(),`。
- **L3758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy argument '%s' of a defined input/output procedure must be assumed-length CHARACTER of default kind"_err_en_US,`.
  **L3758 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy argument '%s' of a defined input/output procedure must be assumed-length CHARACTER of default kind"_err_en_US,`。
- **L3759 EN**: Executes a call or declaration centered on `arg->name`.
  **L3759 CN**: 执行以 `arg->name` 为核心的调用或声明。
- **L3760 EN**: Closes the current lexical scope or compound statement.
  **L3760 CN**: 结束当前词法作用域或复合语句块。
- **L3761 EN**: Executes a call or declaration centered on `CheckDioDummyIsScalar`.
  **L3761 CN**: 执行以 `CheckDioDummyIsScalar` 为核心的调用或声明。
- **L3762 EN**: Closes the current lexical scope or compound statement.
  **L3762 CN**: 结束当前词法作用域或复合语句块。
- **L3763 EN**: Closes the current lexical scope or compound statement.
  **L3763 CN**: 结束当前词法作用域或复合语句块。
- **L3764 EN**: Blank line separating nearby declarations or logic blocks.
  **L3764 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3765 EN**: Continues logic associated with callable symbol `CheckDioVlistArg`.
  **L3765 CN**: 继续与可调用符号 `CheckDioVlistArg` 相关的逻辑。
- **L3766 EN**: Continues the surrounding expression or declaration: `const Symbol &subp, const Symbol *arg, std::size_t argPosition) {`.
  **L3766 CN**: 继续构造周围的表达式或声明：`const Symbol &subp, const Symbol *arg, std::size_t argPosition) {`。
- **L3767 EN**: Comment explains nearby logic, intent, or metadata: `Vlist argument looks like: INTEGER, INTENT(IN) :: v_list(:)`.
  **L3767 CN**: 注释说明附近代码的逻辑、意图或元数据：`Vlist argument looks like: INTEGER, INTENT(IN) :: v_list(:)`。
- **L3768 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3768 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3769-3792

````cpp
    CheckDioDummyIsDefaultInteger(subp, *arg);
    CheckDioDummyAttrs(subp, *arg, Attr::INTENT_IN);
    const auto *objectDetails{arg->detailsIf<ObjectEntityDetails>()};
    if (!objectDetails || !objectDetails->shape().CanBeAssumedShape() ||
        objectDetails->shape().Rank() != 1) {
      messages_.Say(arg->name(),
          "Dummy argument '%s' of a defined input/output procedure must be assumed shape vector"_err_en_US,
          arg->name());
    }
  }
}

void CheckHelper::CheckDioArgCount(
    const Symbol &subp, common::DefinedIo ioKind, std::size_t argCount) {
  const std::size_t requiredArgCount{
      (std::size_t)(ioKind == common::DefinedIo::ReadFormatted ||
                  ioKind == common::DefinedIo::WriteFormatted
              ? 6
              : 4)};
  if (argCount != requiredArgCount) {
    SayWithDeclaration(subp,
        "Defined input/output procedure '%s' must have %d dummy arguments rather than %d"_err_en_US,
        subp.name(), requiredArgCount, argCount);
    context_.SetError(subp);
````
- **L3769 EN**: Executes a call or declaration centered on `CheckDioDummyIsDefaultInteger`.
  **L3769 CN**: 执行以 `CheckDioDummyIsDefaultInteger` 为核心的调用或声明。
- **L3770 EN**: Executes a call or declaration centered on `CheckDioDummyAttrs`.
  **L3770 CN**: 执行以 `CheckDioDummyAttrs` 为核心的调用或声明。
- **L3771 EN**: Executes a call or declaration centered on `*objectDetails{arg->detailsIf<ObjectEntityDetails>`.
  **L3771 CN**: 执行以 `*objectDetails{arg->detailsIf<ObjectEntityDetails>` 为核心的调用或声明。
- **L3772 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3772 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3773 EN**: Starts a function, method, lambda, or structured scope: `objectDetails->shape().Rank() != 1) {`.
  **L3773 CN**: 开始一个函数、方法、lambda 或结构化作用域：`objectDetails->shape().Rank() != 1) {`。
- **L3774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(arg->name(),`.
  **L3774 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(arg->name(),`。
- **L3775 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy argument '%s' of a defined input/output procedure must be assumed shape vector"_err_en_US,`.
  **L3775 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy argument '%s' of a defined input/output procedure must be assumed shape vector"_err_en_US,`。
- **L3776 EN**: Executes a call or declaration centered on `arg->name`.
  **L3776 CN**: 执行以 `arg->name` 为核心的调用或声明。
- **L3777 EN**: Closes the current lexical scope or compound statement.
  **L3777 CN**: 结束当前词法作用域或复合语句块。
- **L3778 EN**: Closes the current lexical scope or compound statement.
  **L3778 CN**: 结束当前词法作用域或复合语句块。
- **L3779 EN**: Closes the current lexical scope or compound statement.
  **L3779 CN**: 结束当前词法作用域或复合语句块。
- **L3780 EN**: Blank line separating nearby declarations or logic blocks.
  **L3780 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3781 EN**: Continues logic associated with callable symbol `CheckDioArgCount`.
  **L3781 CN**: 继续与可调用符号 `CheckDioArgCount` 相关的逻辑。
- **L3782 EN**: Continues the surrounding expression or declaration: `const Symbol &subp, common::DefinedIo ioKind, std::size_t argCount) {`.
  **L3782 CN**: 继续构造周围的表达式或声明：`const Symbol &subp, common::DefinedIo ioKind, std::size_t argCount) {`。
- **L3783 EN**: Continues the surrounding expression or declaration: `const std::size_t requiredArgCount{`.
  **L3783 CN**: 继续构造周围的表达式或声明：`const std::size_t requiredArgCount{`。
- **L3784 EN**: Continues the surrounding expression or declaration: `(std::size_t)(ioKind == common::DefinedIo::ReadFormatted ||`.
  **L3784 CN**: 继续构造周围的表达式或声明：`(std::size_t)(ioKind == common::DefinedIo::ReadFormatted ||`。
- **L3785 EN**: Continues the surrounding expression or declaration: `ioKind == common::DefinedIo::WriteFormatted`.
  **L3785 CN**: 继续构造周围的表达式或声明：`ioKind == common::DefinedIo::WriteFormatted`。
- **L3786 EN**: Continues the surrounding expression or declaration: `? 6`.
  **L3786 CN**: 继续构造周围的表达式或声明：`? 6`。
- **L3787 EN**: Executes a standalone statement or declaration: `: 4)};`.
  **L3787 CN**: 执行一条独立语句或声明：`: 4)};`。
- **L3788 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3788 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3789 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayWithDeclaration(subp,`.
  **L3789 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayWithDeclaration(subp,`。
- **L3790 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Defined input/output procedure '%s' must have %d dummy arguments rather than %d"_err_en_US,`.
  **L3790 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Defined input/output procedure '%s' must have %d dummy arguments rather than %d"_err_en_US,`。
- **L3791 EN**: Executes a call or declaration centered on `subp.name`.
  **L3791 CN**: 执行以 `subp.name` 为核心的调用或声明。
- **L3792 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L3792 CN**: 执行以 `context_.SetError` 为核心的调用或声明。

### Lines 3793-3816

````cpp
  }
}

void CheckHelper::CheckDioDummyAttrs(
    const Symbol &subp, const Symbol &arg, Attr goodIntent) {
  // Defined I/O procedures can't have attributes other than INTENT
  Attrs attrs{arg.attrs()};
  if (!attrs.test(goodIntent)) {
    messages_.Say(arg.name(),
        "Dummy argument '%s' of a defined input/output procedure must have intent '%s'"_err_en_US,
        arg.name(), AttrToString(goodIntent));
  }
  attrs = attrs - Attr::INTENT_IN - Attr::INTENT_OUT - Attr::INTENT_INOUT;
  if (!attrs.empty()) {
    messages_.Say(arg.name(),
        "Dummy argument '%s' of a defined input/output procedure may not have any attributes"_err_en_US,
        arg.name());
  }
}

// Enforce semantics for defined input/output procedures (12.6.4.8.2) and C777
void CheckHelper::CheckDefinedIoProc(const Symbol &symbol,
    const GenericDetails &details, common::DefinedIo ioKind) {
  for (auto ref : details.specificProcs()) {
````
- **L3793 EN**: Closes the current lexical scope or compound statement.
  **L3793 CN**: 结束当前词法作用域或复合语句块。
- **L3794 EN**: Closes the current lexical scope or compound statement.
  **L3794 CN**: 结束当前词法作用域或复合语句块。
- **L3795 EN**: Blank line separating nearby declarations or logic blocks.
  **L3795 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3796 EN**: Continues logic associated with callable symbol `CheckDioDummyAttrs`.
  **L3796 CN**: 继续与可调用符号 `CheckDioDummyAttrs` 相关的逻辑。
- **L3797 EN**: Continues the surrounding expression or declaration: `const Symbol &subp, const Symbol &arg, Attr goodIntent) {`.
  **L3797 CN**: 继续构造周围的表达式或声明：`const Symbol &subp, const Symbol &arg, Attr goodIntent) {`。
- **L3798 EN**: Comment explains nearby logic, intent, or metadata: `Defined I/O procedures can't have attributes other than INTENT`.
  **L3798 CN**: 注释说明附近代码的逻辑、意图或元数据：`Defined I/O procedures can't have attributes other than INTENT`。
- **L3799 EN**: Executes a call or declaration centered on `attrs{arg.attrs`.
  **L3799 CN**: 执行以 `attrs{arg.attrs` 为核心的调用或声明。
- **L3800 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3800 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(arg.name(),`.
  **L3801 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(arg.name(),`。
- **L3802 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy argument '%s' of a defined input/output procedure must have intent '%s'"_err_en_US,`.
  **L3802 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy argument '%s' of a defined input/output procedure must have intent '%s'"_err_en_US,`。
- **L3803 EN**: Executes a call or declaration centered on `arg.name`.
  **L3803 CN**: 执行以 `arg.name` 为核心的调用或声明。
- **L3804 EN**: Closes the current lexical scope or compound statement.
  **L3804 CN**: 结束当前词法作用域或复合语句块。
- **L3805 EN**: Executes a standalone statement or declaration: `attrs = attrs - Attr::INTENT_IN - Attr::INTENT_OUT - Attr::INTENT_INOUT;`.
  **L3805 CN**: 执行一条独立语句或声明：`attrs = attrs - Attr::INTENT_IN - Attr::INTENT_OUT - Attr::INTENT_INOUT;`。
- **L3806 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3806 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3807 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(arg.name(),`.
  **L3807 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(arg.name(),`。
- **L3808 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy argument '%s' of a defined input/output procedure may not have any attributes"_err_en_US,`.
  **L3808 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy argument '%s' of a defined input/output procedure may not have any attributes"_err_en_US,`。
- **L3809 EN**: Executes a call or declaration centered on `arg.name`.
  **L3809 CN**: 执行以 `arg.name` 为核心的调用或声明。
- **L3810 EN**: Closes the current lexical scope or compound statement.
  **L3810 CN**: 结束当前词法作用域或复合语句块。
- **L3811 EN**: Closes the current lexical scope or compound statement.
  **L3811 CN**: 结束当前词法作用域或复合语句块。
- **L3812 EN**: Blank line separating nearby declarations or logic blocks.
  **L3812 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3813 EN**: Comment explains nearby logic, intent, or metadata: `Enforce semantics for defined input/output procedures (12.6.4.8.2) and C777`.
  **L3813 CN**: 注释说明附近代码的逻辑、意图或元数据：`Enforce semantics for defined input/output procedures (12.6.4.8.2) and C777`。
- **L3814 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckHelper::CheckDefinedIoProc(const Symbol &symbol,`.
  **L3814 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckHelper::CheckDefinedIoProc(const Symbol &symbol,`。
- **L3815 EN**: Continues the surrounding expression or declaration: `const GenericDetails &details, common::DefinedIo ioKind) {`.
  **L3815 CN**: 继续构造周围的表达式或声明：`const GenericDetails &details, common::DefinedIo ioKind) {`。
- **L3816 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3816 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 3817-3840

````cpp
    const Symbol &ultimate{ref->GetUltimate()};
    const auto *binding{ultimate.detailsIf<ProcBindingDetails>()};
    if (ultimate.attrs().test(Attr::NOPASS)) { // C774
      messages_.Say(
          "Defined input/output procedure '%s' may not have NOPASS attribute"_err_en_US,
          ultimate.name());
      context_.SetError(ultimate);
    }
    const Symbol *specificProc{binding ? &binding->symbol() : &ultimate};
    const Symbol *specificSubp{specificProc};
    if (const auto *peDetails{specificSubp->detailsIf<ProcEntityDetails>()}) {
      specificSubp = peDetails->procInterface();
      if (!specificSubp) {
        continue;
      }
    }
    if (const auto *subpDetails{specificSubp->detailsIf<SubprogramDetails>()}) {
      const std::vector<Symbol *> &dummyArgs{subpDetails->dummyArgs()};
      CheckDioArgCount(*specificSubp, ioKind, dummyArgs.size());
      int argCount{0};
      for (auto *arg : dummyArgs) {
        if (arg && arg->Corank() > 0) {
          evaluate::AttachDeclaration(
              messages_.Say(arg->name(),
````
- **L3817 EN**: Executes a call or declaration centered on `&ultimate{ref->GetUltimate`.
  **L3817 CN**: 执行以 `&ultimate{ref->GetUltimate` 为核心的调用或声明。
- **L3818 EN**: Executes a call or declaration centered on `*binding{ultimate.detailsIf<ProcBindingDetails>`.
  **L3818 CN**: 执行以 `*binding{ultimate.detailsIf<ProcBindingDetails>` 为核心的调用或声明。
- **L3819 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3819 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3820 EN**: Continues logic associated with callable symbol `Say`.
  **L3820 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L3821 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Defined input/output procedure '%s' may not have NOPASS attribute"_err_en_US,`.
  **L3821 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Defined input/output procedure '%s' may not have NOPASS attribute"_err_en_US,`。
- **L3822 EN**: Executes a call or declaration centered on `ultimate.name`.
  **L3822 CN**: 执行以 `ultimate.name` 为核心的调用或声明。
- **L3823 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L3823 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L3824 EN**: Closes the current lexical scope or compound statement.
  **L3824 CN**: 结束当前词法作用域或复合语句块。
- **L3825 EN**: Executes a call or declaration centered on `&binding->symbol`.
  **L3825 CN**: 执行以 `&binding->symbol` 为核心的调用或声明。
- **L3826 EN**: Executes a standalone statement or declaration: `const Symbol *specificSubp{specificProc};`.
  **L3826 CN**: 执行一条独立语句或声明：`const Symbol *specificSubp{specificProc};`。
- **L3827 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3827 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3828 EN**: Executes a call or declaration centered on `peDetails->procInterface`.
  **L3828 CN**: 执行以 `peDetails->procInterface` 为核心的调用或声明。
- **L3829 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3829 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3830 EN**: Skips to the next loop iteration.
  **L3830 CN**: 跳到下一次循环迭代。
- **L3831 EN**: Closes the current lexical scope or compound statement.
  **L3831 CN**: 结束当前词法作用域或复合语句块。
- **L3832 EN**: Closes the current lexical scope or compound statement.
  **L3832 CN**: 结束当前词法作用域或复合语句块。
- **L3833 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3833 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3834 EN**: Executes a call or declaration centered on `&dummyArgs{subpDetails->dummyArgs`.
  **L3834 CN**: 执行以 `&dummyArgs{subpDetails->dummyArgs` 为核心的调用或声明。
- **L3835 EN**: Executes a call or declaration centered on `CheckDioArgCount`.
  **L3835 CN**: 执行以 `CheckDioArgCount` 为核心的调用或声明。
- **L3836 EN**: Executes a standalone statement or declaration: `int argCount{0};`.
  **L3836 CN**: 执行一条独立语句或声明：`int argCount{0};`。
- **L3837 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3837 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3838 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3838 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3839 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L3839 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L3840 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages_.Say(arg->name(),`.
  **L3840 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages_.Say(arg->name(),`。

### Lines 3841-3864

````cpp
                  "Dummy argument '%s' of defined input/output procedure '%s' may not be a coarray"_err_en_US,
                  arg->name(), ultimate.name()),
              *arg);
        }
        switch (argCount++) {
        case 0:
          // dtv-type-spec, INTENT(INOUT) :: dtv
          CheckDioDtvArg(*specificProc, *specificSubp, arg, ioKind, symbol);
          break;
        case 1:
          // INTEGER, INTENT(IN) :: unit
          CheckDefaultIntegerArg(*specificSubp, arg, Attr::INTENT_IN);
          break;
        case 2:
          if (ioKind == common::DefinedIo::ReadFormatted ||
              ioKind == common::DefinedIo::WriteFormatted) {
            // CHARACTER (LEN=*), INTENT(IN) :: iotype
            CheckDioAssumedLenCharacterArg(
                *specificSubp, arg, argCount, Attr::INTENT_IN);
          } else {
            // INTEGER, INTENT(OUT) :: iostat
            CheckDefaultIntegerArg(*specificSubp, arg, Attr::INTENT_OUT);
          }
          break;
````
- **L3841 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy argument '%s' of defined input/output procedure '%s' may not be a coarray"_err_en_US,`.
  **L3841 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy argument '%s' of defined input/output procedure '%s' may not be a coarray"_err_en_US,`。
- **L3842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arg->name(), ultimate.name()),`.
  **L3842 CN**: 继续一个多行参数列表、初始化器或聚合项：`arg->name(), ultimate.name()),`。
- **L3843 EN**: Comment explains nearby logic, intent, or metadata: `arg);`.
  **L3843 CN**: 注释说明附近代码的逻辑、意图或元数据：`arg);`。
- **L3844 EN**: Closes the current lexical scope or compound statement.
  **L3844 CN**: 结束当前词法作用域或复合语句块。
- **L3845 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L3845 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L3846 EN**: Introduces a switch dispatch label: `case 0:`.
  **L3846 CN**: 引入一个 switch 分发标签：`case 0:`。
- **L3847 EN**: Comment explains nearby logic, intent, or metadata: `dtv-type-spec, INTENT(INOUT) :: dtv`.
  **L3847 CN**: 注释说明附近代码的逻辑、意图或元数据：`dtv-type-spec, INTENT(INOUT) :: dtv`。
- **L3848 EN**: Executes a call or declaration centered on `CheckDioDtvArg`.
  **L3848 CN**: 执行以 `CheckDioDtvArg` 为核心的调用或声明。
- **L3849 EN**: Exits the nearest loop or switch statement.
  **L3849 CN**: 退出最近的循环或 switch 语句。
- **L3850 EN**: Introduces a switch dispatch label: `case 1:`.
  **L3850 CN**: 引入一个 switch 分发标签：`case 1:`。
- **L3851 EN**: Comment explains nearby logic, intent, or metadata: `INTEGER, INTENT(IN) :: unit`.
  **L3851 CN**: 注释说明附近代码的逻辑、意图或元数据：`INTEGER, INTENT(IN) :: unit`。
- **L3852 EN**: Executes a call or declaration centered on `CheckDefaultIntegerArg`.
  **L3852 CN**: 执行以 `CheckDefaultIntegerArg` 为核心的调用或声明。
- **L3853 EN**: Exits the nearest loop or switch statement.
  **L3853 CN**: 退出最近的循环或 switch 语句。
- **L3854 EN**: Introduces a switch dispatch label: `case 2:`.
  **L3854 CN**: 引入一个 switch 分发标签：`case 2:`。
- **L3855 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3855 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3856 EN**: Continues the surrounding expression or declaration: `ioKind == common::DefinedIo::WriteFormatted) {`.
  **L3856 CN**: 继续构造周围的表达式或声明：`ioKind == common::DefinedIo::WriteFormatted) {`。
- **L3857 EN**: Comment explains nearby logic, intent, or metadata: `CHARACTER (LEN=*), INTENT(IN) :: iotype`.
  **L3857 CN**: 注释说明附近代码的逻辑、意图或元数据：`CHARACTER (LEN=*), INTENT(IN) :: iotype`。
- **L3858 EN**: Continues logic associated with callable symbol `CheckDioAssumedLenCharacterArg`.
  **L3858 CN**: 继续与可调用符号 `CheckDioAssumedLenCharacterArg` 相关的逻辑。
- **L3859 EN**: Comment explains nearby logic, intent, or metadata: `specificSubp, arg, argCount, Attr::INTENT_IN);`.
  **L3859 CN**: 注释说明附近代码的逻辑、意图或元数据：`specificSubp, arg, argCount, Attr::INTENT_IN);`。
- **L3860 EN**: Transitions from the previous branch into the alternative path.
  **L3860 CN**: 从前一个分支过渡到备选路径。
- **L3861 EN**: Comment explains nearby logic, intent, or metadata: `INTEGER, INTENT(OUT) :: iostat`.
  **L3861 CN**: 注释说明附近代码的逻辑、意图或元数据：`INTEGER, INTENT(OUT) :: iostat`。
- **L3862 EN**: Executes a call or declaration centered on `CheckDefaultIntegerArg`.
  **L3862 CN**: 执行以 `CheckDefaultIntegerArg` 为核心的调用或声明。
- **L3863 EN**: Closes the current lexical scope or compound statement.
  **L3863 CN**: 结束当前词法作用域或复合语句块。
- **L3864 EN**: Exits the nearest loop or switch statement.
  **L3864 CN**: 退出最近的循环或 switch 语句。

### Lines 3865-3888

````cpp
        case 3:
          if (ioKind == common::DefinedIo::ReadFormatted ||
              ioKind == common::DefinedIo::WriteFormatted) {
            // INTEGER, INTENT(IN) :: v_list(:)
            CheckDioVlistArg(*specificSubp, arg, argCount);
          } else {
            // CHARACTER (LEN=*), INTENT(INOUT) :: iomsg
            CheckDioAssumedLenCharacterArg(
                *specificSubp, arg, argCount, Attr::INTENT_INOUT);
          }
          break;
        case 4:
          // INTEGER, INTENT(OUT) :: iostat
          CheckDefaultIntegerArg(*specificSubp, arg, Attr::INTENT_OUT);
          break;
        case 5:
          // CHARACTER (LEN=*), INTENT(INOUT) :: iomsg
          CheckDioAssumedLenCharacterArg(
              *specificSubp, arg, argCount, Attr::INTENT_INOUT);
          break;
        default:;
        }
      }
    }
````
- **L3865 EN**: Introduces a switch dispatch label: `case 3:`.
  **L3865 CN**: 引入一个 switch 分发标签：`case 3:`。
- **L3866 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3866 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3867 EN**: Continues the surrounding expression or declaration: `ioKind == common::DefinedIo::WriteFormatted) {`.
  **L3867 CN**: 继续构造周围的表达式或声明：`ioKind == common::DefinedIo::WriteFormatted) {`。
- **L3868 EN**: Comment explains nearby logic, intent, or metadata: `INTEGER, INTENT(IN) :: v_list(:)`.
  **L3868 CN**: 注释说明附近代码的逻辑、意图或元数据：`INTEGER, INTENT(IN) :: v_list(:)`。
- **L3869 EN**: Executes a call or declaration centered on `CheckDioVlistArg`.
  **L3869 CN**: 执行以 `CheckDioVlistArg` 为核心的调用或声明。
- **L3870 EN**: Transitions from the previous branch into the alternative path.
  **L3870 CN**: 从前一个分支过渡到备选路径。
- **L3871 EN**: Comment explains nearby logic, intent, or metadata: `CHARACTER (LEN=*), INTENT(INOUT) :: iomsg`.
  **L3871 CN**: 注释说明附近代码的逻辑、意图或元数据：`CHARACTER (LEN=*), INTENT(INOUT) :: iomsg`。
- **L3872 EN**: Continues logic associated with callable symbol `CheckDioAssumedLenCharacterArg`.
  **L3872 CN**: 继续与可调用符号 `CheckDioAssumedLenCharacterArg` 相关的逻辑。
- **L3873 EN**: Comment explains nearby logic, intent, or metadata: `specificSubp, arg, argCount, Attr::INTENT_INOUT);`.
  **L3873 CN**: 注释说明附近代码的逻辑、意图或元数据：`specificSubp, arg, argCount, Attr::INTENT_INOUT);`。
- **L3874 EN**: Closes the current lexical scope or compound statement.
  **L3874 CN**: 结束当前词法作用域或复合语句块。
- **L3875 EN**: Exits the nearest loop or switch statement.
  **L3875 CN**: 退出最近的循环或 switch 语句。
- **L3876 EN**: Introduces a switch dispatch label: `case 4:`.
  **L3876 CN**: 引入一个 switch 分发标签：`case 4:`。
- **L3877 EN**: Comment explains nearby logic, intent, or metadata: `INTEGER, INTENT(OUT) :: iostat`.
  **L3877 CN**: 注释说明附近代码的逻辑、意图或元数据：`INTEGER, INTENT(OUT) :: iostat`。
- **L3878 EN**: Executes a call or declaration centered on `CheckDefaultIntegerArg`.
  **L3878 CN**: 执行以 `CheckDefaultIntegerArg` 为核心的调用或声明。
- **L3879 EN**: Exits the nearest loop or switch statement.
  **L3879 CN**: 退出最近的循环或 switch 语句。
- **L3880 EN**: Introduces a switch dispatch label: `case 5:`.
  **L3880 CN**: 引入一个 switch 分发标签：`case 5:`。
- **L3881 EN**: Comment explains nearby logic, intent, or metadata: `CHARACTER (LEN=*), INTENT(INOUT) :: iomsg`.
  **L3881 CN**: 注释说明附近代码的逻辑、意图或元数据：`CHARACTER (LEN=*), INTENT(INOUT) :: iomsg`。
- **L3882 EN**: Continues logic associated with callable symbol `CheckDioAssumedLenCharacterArg`.
  **L3882 CN**: 继续与可调用符号 `CheckDioAssumedLenCharacterArg` 相关的逻辑。
- **L3883 EN**: Comment explains nearby logic, intent, or metadata: `specificSubp, arg, argCount, Attr::INTENT_INOUT);`.
  **L3883 CN**: 注释说明附近代码的逻辑、意图或元数据：`specificSubp, arg, argCount, Attr::INTENT_INOUT);`。
- **L3884 EN**: Exits the nearest loop or switch statement.
  **L3884 CN**: 退出最近的循环或 switch 语句。
- **L3885 EN**: Introduces a switch dispatch label: `default:;`.
  **L3885 CN**: 引入一个 switch 分发标签：`default:;`。
- **L3886 EN**: Closes the current lexical scope or compound statement.
  **L3886 CN**: 结束当前词法作用域或复合语句块。
- **L3887 EN**: Closes the current lexical scope or compound statement.
  **L3887 CN**: 结束当前词法作用域或复合语句块。
- **L3888 EN**: Closes the current lexical scope or compound statement.
  **L3888 CN**: 结束当前词法作用域或复合语句块。

### Lines 3889-3912

````cpp
  }
}

void CheckHelper::CheckSymbolType(const Symbol &symbol) {
  const Symbol *result{FindFunctionResult(symbol)};
  const Symbol &relevant{result ? *result : symbol};
  if (IsAllocatable(relevant)) { // always ok
  } else if (IsProcedurePointer(symbol) && result && IsPointer(*result)) {
    // procedure pointer returning allocatable or pointer: ok
  } else if (IsPointer(relevant) && !IsProcedure(relevant)) {
    // object pointers are always ok
  } else if (auto dyType{evaluate::DynamicType::From(relevant)}) {
    if (dyType->IsPolymorphic() && !dyType->IsAssumedType() &&
        !(IsDummy(symbol) && !IsProcedure(relevant))) { // C708
      if (IsProcedure(symbol)) {
        messages_.Say(
            "Polymorphic function%s '%s' must have an explicit interface whose result is ALLOCATABLE or POINTER"_err_en_US,
            IsPointer(symbol) ? " pointer" : "", symbol.name());
      } else {
        messages_.Say(
            "CLASS entity '%s' must be a dummy argument, allocatable, or object pointer"_err_en_US,
            symbol.name());
      }
    }
````
- **L3889 EN**: Closes the current lexical scope or compound statement.
  **L3889 CN**: 结束当前词法作用域或复合语句块。
- **L3890 EN**: Closes the current lexical scope or compound statement.
  **L3890 CN**: 结束当前词法作用域或复合语句块。
- **L3891 EN**: Blank line separating nearby declarations or logic blocks.
  **L3891 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3892 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckSymbolType(const Symbol &symbol) {`.
  **L3892 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckSymbolType(const Symbol &symbol) {`。
- **L3893 EN**: Executes a call or declaration centered on `*result{FindFunctionResult`.
  **L3893 CN**: 执行以 `*result{FindFunctionResult` 为核心的调用或声明。
- **L3894 EN**: Executes a standalone statement or declaration: `const Symbol &relevant{result ? *result : symbol};`.
  **L3894 CN**: 执行一条独立语句或声明：`const Symbol &relevant{result ? *result : symbol};`。
- **L3895 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3895 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3896 EN**: Transitions from the previous branch into an `else if` condition.
  **L3896 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3897 EN**: Comment explains nearby logic, intent, or metadata: `procedure pointer returning allocatable or pointer: ok`.
  **L3897 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedure pointer returning allocatable or pointer: ok`。
- **L3898 EN**: Transitions from the previous branch into an `else if` condition.
  **L3898 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3899 EN**: Comment explains nearby logic, intent, or metadata: `object pointers are always ok`.
  **L3899 CN**: 注释说明附近代码的逻辑、意图或元数据：`object pointers are always ok`。
- **L3900 EN**: Transitions from the previous branch into an `else if` condition.
  **L3900 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3901 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3901 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3902 EN**: Continues logic associated with callable symbol `IsDummy`.
  **L3902 CN**: 继续与可调用符号 `IsDummy` 相关的逻辑。
- **L3903 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3903 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3904 EN**: Continues logic associated with callable symbol `Say`.
  **L3904 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L3905 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Polymorphic function%s '%s' must have an explicit interface whose result is ALLOCATABLE or POINTER"_err_en_US,`.
  **L3905 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Polymorphic function%s '%s' must have an explicit interface whose result is ALLOCATABLE or POINTER"_err_en_US,`。
- **L3906 EN**: Executes a call or declaration centered on `IsPointer`.
  **L3906 CN**: 执行以 `IsPointer` 为核心的调用或声明。
- **L3907 EN**: Transitions from the previous branch into the alternative path.
  **L3907 CN**: 从前一个分支过渡到备选路径。
- **L3908 EN**: Continues logic associated with callable symbol `Say`.
  **L3908 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L3909 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"CLASS entity '%s' must be a dummy argument, allocatable, or object pointer"_err_en_US,`.
  **L3909 CN**: 继续一个多行参数列表、初始化器或聚合项：`"CLASS entity '%s' must be a dummy argument, allocatable, or object pointer"_err_en_US,`。
- **L3910 EN**: Executes a call or declaration centered on `symbol.name`.
  **L3910 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L3911 EN**: Closes the current lexical scope or compound statement.
  **L3911 CN**: 结束当前词法作用域或复合语句块。
- **L3912 EN**: Closes the current lexical scope or compound statement.
  **L3912 CN**: 结束当前词法作用域或复合语句块。

### Lines 3913-3936

````cpp
    if (dyType->HasDeferredTypeParameter()) { // C702
      messages_.Say(
          "'%s' has a type %s with a deferred type parameter but is neither an allocatable nor an object pointer"_err_en_US,
          symbol.name(), dyType->AsFortran());
    }
    if (!symbol.has<ObjectEntityDetails>()) {
      if (const DerivedTypeSpec *
          derived{evaluate::GetDerivedTypeSpec(*dyType)}) {
        if (IsEventTypeOrLockType(derived)) {
          messages_.Say(
              "Entity '%s' with EVENT_TYPE or LOCK_TYPE must be an object"_err_en_US,
              symbol.name());
        } else if (auto iter{FindEventOrLockPotentialComponent(*derived)}) {
          messages_.Say(
              "Entity '%s' with EVENT_TYPE or LOCK_TYPE potential subobject component '%s' must be an object"_err_en_US,
              symbol.name(), iter.BuildResultDesignatorName());
        }
      }
    }
  }
}

void CheckHelper::CheckModuleProcedureDef(const Symbol &symbol) {
  auto procClass{ClassifyProcedure(symbol)};
````
- **L3913 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3913 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3914 EN**: Continues logic associated with callable symbol `Say`.
  **L3914 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L3915 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' has a type %s with a deferred type parameter but is neither an allocatable nor an object pointer"_err_en_US,`.
  **L3915 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' has a type %s with a deferred type parameter but is neither an allocatable nor an object pointer"_err_en_US,`。
- **L3916 EN**: Executes a call or declaration centered on `symbol.name`.
  **L3916 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L3917 EN**: Closes the current lexical scope or compound statement.
  **L3917 CN**: 结束当前词法作用域或复合语句块。
- **L3918 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3918 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3919 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3919 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3920 EN**: Starts a function, method, lambda, or structured scope: `derived{evaluate::GetDerivedTypeSpec(*dyType)}) {`.
  **L3920 CN**: 开始一个函数、方法、lambda 或结构化作用域：`derived{evaluate::GetDerivedTypeSpec(*dyType)}) {`。
- **L3921 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3921 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3922 EN**: Continues logic associated with callable symbol `Say`.
  **L3922 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L3923 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Entity '%s' with EVENT_TYPE or LOCK_TYPE must be an object"_err_en_US,`.
  **L3923 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Entity '%s' with EVENT_TYPE or LOCK_TYPE must be an object"_err_en_US,`。
- **L3924 EN**: Executes a call or declaration centered on `symbol.name`.
  **L3924 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L3925 EN**: Transitions from the previous branch into an `else if` condition.
  **L3925 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3926 EN**: Continues logic associated with callable symbol `Say`.
  **L3926 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L3927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Entity '%s' with EVENT_TYPE or LOCK_TYPE potential subobject component '%s' must be an object"_err_en_US,`.
  **L3927 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Entity '%s' with EVENT_TYPE or LOCK_TYPE potential subobject component '%s' must be an object"_err_en_US,`。
- **L3928 EN**: Executes a call or declaration centered on `symbol.name`.
  **L3928 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L3929 EN**: Closes the current lexical scope or compound statement.
  **L3929 CN**: 结束当前词法作用域或复合语句块。
- **L3930 EN**: Closes the current lexical scope or compound statement.
  **L3930 CN**: 结束当前词法作用域或复合语句块。
- **L3931 EN**: Closes the current lexical scope or compound statement.
  **L3931 CN**: 结束当前词法作用域或复合语句块。
- **L3932 EN**: Closes the current lexical scope or compound statement.
  **L3932 CN**: 结束当前词法作用域或复合语句块。
- **L3933 EN**: Closes the current lexical scope or compound statement.
  **L3933 CN**: 结束当前词法作用域或复合语句块。
- **L3934 EN**: Blank line separating nearby declarations or logic blocks.
  **L3934 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3935 EN**: Starts a function, method, lambda, or structured scope: `void CheckHelper::CheckModuleProcedureDef(const Symbol &symbol) {`.
  **L3935 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckHelper::CheckModuleProcedureDef(const Symbol &symbol) {`。
- **L3936 EN**: Executes a call or declaration centered on `procClass{ClassifyProcedure`.
  **L3936 CN**: 执行以 `procClass{ClassifyProcedure` 为核心的调用或声明。

### Lines 3937-3960

````cpp
  if (const auto *subprogram{symbol.detailsIf<SubprogramDetails>()};
      subprogram &&
      (procClass == ProcedureDefinitionClass::Module &&
          symbol.attrs().test(Attr::MODULE)) &&
      !subprogram->bindName() && !subprogram->isInterface()) {
    const Symbol &interface {
      subprogram->moduleInterface() ? *subprogram->moduleInterface() : symbol
    };
    if (const Symbol *
            module{interface.owner().kind() == Scope::Kind::Module
                    ? interface.owner().symbol()
                    : nullptr};
        module && module->has<ModuleDetails>()) {
      std::pair<SourceName, const Symbol *> key{symbol.name(), module};
      auto iter{moduleProcs_.find(key)};
      if (iter == moduleProcs_.end()) {
        moduleProcs_.emplace(std::move(key), symbol);
      } else if (
          auto *msg{messages_.Say(symbol.name(),
              "Module procedure '%s' in '%s' has multiple definitions"_err_en_US,
              symbol.name(), GetModuleOrSubmoduleName(*module))}) {
        msg->Attach(iter->second->name(), "Previous definition of '%s'"_en_US,
            symbol.name());
      }
````
- **L3937 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3937 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3938 EN**: Continues the surrounding expression or declaration: `subprogram &&`.
  **L3938 CN**: 继续构造周围的表达式或声明：`subprogram &&`。
- **L3939 EN**: Continues the surrounding expression or declaration: `(procClass == ProcedureDefinitionClass::Module &&`.
  **L3939 CN**: 继续构造周围的表达式或声明：`(procClass == ProcedureDefinitionClass::Module &&`。
- **L3940 EN**: Continues logic associated with callable symbol `attrs`.
  **L3940 CN**: 继续与可调用符号 `attrs` 相关的逻辑。
- **L3941 EN**: Starts a function, method, lambda, or structured scope: `!subprogram->bindName() && !subprogram->isInterface()) {`.
  **L3941 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!subprogram->bindName() && !subprogram->isInterface()) {`。
- **L3942 EN**: Continues the surrounding expression or declaration: `const Symbol &interface {`.
  **L3942 CN**: 继续构造周围的表达式或声明：`const Symbol &interface {`。
- **L3943 EN**: Continues logic associated with callable symbol `moduleInterface`.
  **L3943 CN**: 继续与可调用符号 `moduleInterface` 相关的逻辑。
- **L3944 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L3944 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L3945 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3945 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3946 EN**: Continues logic associated with callable symbol `owner`.
  **L3946 CN**: 继续与可调用符号 `owner` 相关的逻辑。
- **L3947 EN**: Continues logic associated with callable symbol `owner`.
  **L3947 CN**: 继续与可调用符号 `owner` 相关的逻辑。
- **L3948 EN**: Executes a standalone statement or declaration: `: nullptr};`.
  **L3948 CN**: 执行一条独立语句或声明：`: nullptr};`。
- **L3949 EN**: Starts a function, method, lambda, or structured scope: `module && module->has<ModuleDetails>()) {`.
  **L3949 CN**: 开始一个函数、方法、lambda 或结构化作用域：`module && module->has<ModuleDetails>()) {`。
- **L3950 EN**: Executes a call or declaration centered on `key{symbol.name`.
  **L3950 CN**: 执行以 `key{symbol.name` 为核心的调用或声明。
- **L3951 EN**: Executes a call or declaration centered on `iter{moduleProcs_.find`.
  **L3951 CN**: 执行以 `iter{moduleProcs_.find` 为核心的调用或声明。
- **L3952 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3952 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3953 EN**: Executes a call or declaration centered on `moduleProcs_.emplace`.
  **L3953 CN**: 执行以 `moduleProcs_.emplace` 为核心的调用或声明。
- **L3954 EN**: Transitions from the previous branch into an `else if` condition.
  **L3954 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L3955 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto *msg{messages_.Say(symbol.name(),`.
  **L3955 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto *msg{messages_.Say(symbol.name(),`。
- **L3956 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Module procedure '%s' in '%s' has multiple definitions"_err_en_US,`.
  **L3956 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Module procedure '%s' in '%s' has multiple definitions"_err_en_US,`。
- **L3957 EN**: Starts a function, method, lambda, or structured scope: `symbol.name(), GetModuleOrSubmoduleName(*module))}) {`.
  **L3957 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.name(), GetModuleOrSubmoduleName(*module))}) {`。
- **L3958 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg->Attach(iter->second->name(), "Previous definition of '%s'"_en_US,`.
  **L3958 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg->Attach(iter->second->name(), "Previous definition of '%s'"_en_US,`。
- **L3959 EN**: Executes a call or declaration centered on `symbol.name`.
  **L3959 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L3960 EN**: Closes the current lexical scope or compound statement.
  **L3960 CN**: 结束当前词法作用域或复合语句块。

### Lines 3961-3984

````cpp
    }
  }
}

void SubprogramMatchHelper::Check(
    const Symbol &symbol1, const Symbol &symbol2) {
  const auto details1{symbol1.get<SubprogramDetails>()};
  const auto details2{symbol2.get<SubprogramDetails>()};
  if (details1.isFunction() != details2.isFunction()) {
    Say(symbol1, symbol2,
        details1.isFunction()
            ? "Module function '%s' was declared as a subroutine in the"
              " corresponding interface body"_err_en_US
            : "Module subroutine '%s' was declared as a function in the"
              " corresponding interface body"_err_en_US);
    return;
  }
  const auto &args1{details1.dummyArgs()};
  const auto &args2{details2.dummyArgs()};
  int nargs1{static_cast<int>(args1.size())};
  int nargs2{static_cast<int>(args2.size())};
  if (nargs1 != nargs2) {
    Say(symbol1, symbol2,
        "Module subprogram '%s' has %d args but the corresponding interface"
````
- **L3961 EN**: Closes the current lexical scope or compound statement.
  **L3961 CN**: 结束当前词法作用域或复合语句块。
- **L3962 EN**: Closes the current lexical scope or compound statement.
  **L3962 CN**: 结束当前词法作用域或复合语句块。
- **L3963 EN**: Closes the current lexical scope or compound statement.
  **L3963 CN**: 结束当前词法作用域或复合语句块。
- **L3964 EN**: Blank line separating nearby declarations or logic blocks.
  **L3964 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3965 EN**: Continues logic associated with callable symbol `Check`.
  **L3965 CN**: 继续与可调用符号 `Check` 相关的逻辑。
- **L3966 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol1, const Symbol &symbol2) {`.
  **L3966 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol1, const Symbol &symbol2) {`。
- **L3967 EN**: Executes a call or declaration centered on `details1{symbol1.get<SubprogramDetails>`.
  **L3967 CN**: 执行以 `details1{symbol1.get<SubprogramDetails>` 为核心的调用或声明。
- **L3968 EN**: Executes a call or declaration centered on `details2{symbol2.get<SubprogramDetails>`.
  **L3968 CN**: 执行以 `details2{symbol2.get<SubprogramDetails>` 为核心的调用或声明。
- **L3969 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3969 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3970 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L3970 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L3971 EN**: Continues logic associated with callable symbol `isFunction`.
  **L3971 CN**: 继续与可调用符号 `isFunction` 相关的逻辑。
- **L3972 EN**: Continues the surrounding expression or declaration: `? "Module function '%s' was declared as a subroutine in the"`.
  **L3972 CN**: 继续构造周围的表达式或声明：`? "Module function '%s' was declared as a subroutine in the"`。
- **L3973 EN**: Continues the surrounding expression or declaration: `" corresponding interface body"_err_en_US`.
  **L3973 CN**: 继续构造周围的表达式或声明：`" corresponding interface body"_err_en_US`。
- **L3974 EN**: Continues the surrounding expression or declaration: `: "Module subroutine '%s' was declared as a function in the"`.
  **L3974 CN**: 继续构造周围的表达式或声明：`: "Module subroutine '%s' was declared as a function in the"`。
- **L3975 EN**: Executes a standalone statement or declaration: `" corresponding interface body"_err_en_US);`.
  **L3975 CN**: 执行一条独立语句或声明：`" corresponding interface body"_err_en_US);`。
- **L3976 EN**: Returns from the current function with `void`.
  **L3976 CN**: 以 `void` 从当前函数返回。
- **L3977 EN**: Closes the current lexical scope or compound statement.
  **L3977 CN**: 结束当前词法作用域或复合语句块。
- **L3978 EN**: Executes a call or declaration centered on `&args1{details1.dummyArgs`.
  **L3978 CN**: 执行以 `&args1{details1.dummyArgs` 为核心的调用或声明。
- **L3979 EN**: Executes a call or declaration centered on `&args2{details2.dummyArgs`.
  **L3979 CN**: 执行以 `&args2{details2.dummyArgs` 为核心的调用或声明。
- **L3980 EN**: Executes a call or declaration centered on `nargs1{static_cast<int>`.
  **L3980 CN**: 执行以 `nargs1{static_cast<int>` 为核心的调用或声明。
- **L3981 EN**: Executes a call or declaration centered on `nargs2{static_cast<int>`.
  **L3981 CN**: 执行以 `nargs2{static_cast<int>` 为核心的调用或声明。
- **L3982 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3982 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3983 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L3983 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L3984 EN**: Continues the surrounding expression or declaration: `"Module subprogram '%s' has %d args but the corresponding interface"`.
  **L3984 CN**: 继续构造周围的表达式或声明：`"Module subprogram '%s' has %d args but the corresponding interface"`。

### Lines 3985-4008

````cpp
        " body has %d"_err_en_US,
        nargs1, nargs2);
    return;
  }
  bool nonRecursive1{symbol1.attrs().test(Attr::NON_RECURSIVE)};
  if (nonRecursive1 != symbol2.attrs().test(Attr::NON_RECURSIVE)) { // C1551
    Say(symbol1, symbol2,
        nonRecursive1
            ? "Module subprogram '%s' has NON_RECURSIVE prefix but"
              " the corresponding interface body does not"_err_en_US
            : "Module subprogram '%s' does not have NON_RECURSIVE prefix but "
              "the corresponding interface body does"_err_en_US);
  }
  const std::string *bindName1{details1.bindName()};
  const std::string *bindName2{details2.bindName()};
  if (!bindName1 && !bindName2) {
    // OK - neither has a binding label
  } else if (!bindName1) {
    Say(symbol1, symbol2,
        "Module subprogram '%s' does not have a binding label but the"
        " corresponding interface body does"_err_en_US);
  } else if (!bindName2) {
    Say(symbol1, symbol2,
        "Module subprogram '%s' has a binding label but the"
````
- **L3985 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" body has %d"_err_en_US,`.
  **L3985 CN**: 继续一个多行参数列表、初始化器或聚合项：`" body has %d"_err_en_US,`。
- **L3986 EN**: Executes a standalone statement or declaration: `nargs1, nargs2);`.
  **L3986 CN**: 执行一条独立语句或声明：`nargs1, nargs2);`。
- **L3987 EN**: Returns from the current function with `void`.
  **L3987 CN**: 以 `void` 从当前函数返回。
- **L3988 EN**: Closes the current lexical scope or compound statement.
  **L3988 CN**: 结束当前词法作用域或复合语句块。
- **L3989 EN**: Executes a call or declaration centered on `nonRecursive1{symbol1.attrs`.
  **L3989 CN**: 执行以 `nonRecursive1{symbol1.attrs` 为核心的调用或声明。
- **L3990 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3990 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3991 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L3991 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L3992 EN**: Continues the surrounding expression or declaration: `nonRecursive1`.
  **L3992 CN**: 继续构造周围的表达式或声明：`nonRecursive1`。
- **L3993 EN**: Continues the surrounding expression or declaration: `? "Module subprogram '%s' has NON_RECURSIVE prefix but"`.
  **L3993 CN**: 继续构造周围的表达式或声明：`? "Module subprogram '%s' has NON_RECURSIVE prefix but"`。
- **L3994 EN**: Continues the surrounding expression or declaration: `" the corresponding interface body does not"_err_en_US`.
  **L3994 CN**: 继续构造周围的表达式或声明：`" the corresponding interface body does not"_err_en_US`。
- **L3995 EN**: Continues the surrounding expression or declaration: `: "Module subprogram '%s' does not have NON_RECURSIVE prefix but "`.
  **L3995 CN**: 继续构造周围的表达式或声明：`: "Module subprogram '%s' does not have NON_RECURSIVE prefix but "`。
- **L3996 EN**: Executes a standalone statement or declaration: `"the corresponding interface body does"_err_en_US);`.
  **L3996 CN**: 执行一条独立语句或声明：`"the corresponding interface body does"_err_en_US);`。
- **L3997 EN**: Closes the current lexical scope or compound statement.
  **L3997 CN**: 结束当前词法作用域或复合语句块。
- **L3998 EN**: Executes a call or declaration centered on `*bindName1{details1.bindName`.
  **L3998 CN**: 执行以 `*bindName1{details1.bindName` 为核心的调用或声明。
- **L3999 EN**: Executes a call or declaration centered on `*bindName2{details2.bindName`.
  **L3999 CN**: 执行以 `*bindName2{details2.bindName` 为核心的调用或声明。
- **L4000 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4000 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4001 EN**: Comment explains nearby logic, intent, or metadata: `OK - neither has a binding label`.
  **L4001 CN**: 注释说明附近代码的逻辑、意图或元数据：`OK - neither has a binding label`。
- **L4002 EN**: Transitions from the previous branch into an `else if` condition.
  **L4002 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4003 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4004 EN**: Continues the surrounding expression or declaration: `"Module subprogram '%s' does not have a binding label but the"`.
  **L4004 CN**: 继续构造周围的表达式或声明：`"Module subprogram '%s' does not have a binding label but the"`。
- **L4005 EN**: Executes a standalone statement or declaration: `" corresponding interface body does"_err_en_US);`.
  **L4005 CN**: 执行一条独立语句或声明：`" corresponding interface body does"_err_en_US);`。
- **L4006 EN**: Transitions from the previous branch into an `else if` condition.
  **L4006 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4007 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4007 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4008 EN**: Continues the surrounding expression or declaration: `"Module subprogram '%s' has a binding label but the"`.
  **L4008 CN**: 继续构造周围的表达式或声明：`"Module subprogram '%s' has a binding label but the"`。

### Lines 4009-4032

````cpp
        " corresponding interface body does not"_err_en_US);
  } else if (*bindName1 != *bindName2) {
    Say(symbol1, symbol2,
        "Module subprogram '%s' has binding label '%s' but the corresponding"
        " interface body has '%s'"_err_en_US,
        *details1.bindName(), *details2.bindName());
  }
  const Procedure *proc1{checkHelper.Characterize(symbol1)};
  const Procedure *proc2{checkHelper.Characterize(symbol2)};
  if (!proc1 || !proc2) {
    return;
  }
  if (proc1->attrs.test(Procedure::Attr::Pure) !=
      proc2->attrs.test(Procedure::Attr::Pure)) {
    Say(symbol1, symbol2,
        "Module subprogram '%s' and its corresponding interface body are not both PURE"_err_en_US);
  }
  if (proc1->attrs.test(Procedure::Attr::Elemental) !=
      proc2->attrs.test(Procedure::Attr::Elemental)) {
    Say(symbol1, symbol2,
        "Module subprogram '%s' and its corresponding interface body are not both ELEMENTAL"_err_en_US);
  }
  if (proc1->attrs.test(Procedure::Attr::BindC) !=
      proc2->attrs.test(Procedure::Attr::BindC)) {
````
- **L4009 EN**: Executes a standalone statement or declaration: `" corresponding interface body does not"_err_en_US);`.
  **L4009 CN**: 执行一条独立语句或声明：`" corresponding interface body does not"_err_en_US);`。
- **L4010 EN**: Transitions from the previous branch into an `else if` condition.
  **L4010 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4011 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4011 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4012 EN**: Continues the surrounding expression or declaration: `"Module subprogram '%s' has binding label '%s' but the corresponding"`.
  **L4012 CN**: 继续构造周围的表达式或声明：`"Module subprogram '%s' has binding label '%s' but the corresponding"`。
- **L4013 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" interface body has '%s'"_err_en_US,`.
  **L4013 CN**: 继续一个多行参数列表、初始化器或聚合项：`" interface body has '%s'"_err_en_US,`。
- **L4014 EN**: Comment explains nearby logic, intent, or metadata: `details1.bindName(), *details2.bindName());`.
  **L4014 CN**: 注释说明附近代码的逻辑、意图或元数据：`details1.bindName(), *details2.bindName());`。
- **L4015 EN**: Closes the current lexical scope or compound statement.
  **L4015 CN**: 结束当前词法作用域或复合语句块。
- **L4016 EN**: Executes a call or declaration centered on `*proc1{checkHelper.Characterize`.
  **L4016 CN**: 执行以 `*proc1{checkHelper.Characterize` 为核心的调用或声明。
- **L4017 EN**: Executes a call or declaration centered on `*proc2{checkHelper.Characterize`.
  **L4017 CN**: 执行以 `*proc2{checkHelper.Characterize` 为核心的调用或声明。
- **L4018 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4018 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4019 EN**: Returns from the current function with `void`.
  **L4019 CN**: 以 `void` 从当前函数返回。
- **L4020 EN**: Closes the current lexical scope or compound statement.
  **L4020 CN**: 结束当前词法作用域或复合语句块。
- **L4021 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4021 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4022 EN**: Starts a function, method, lambda, or structured scope: `proc2->attrs.test(Procedure::Attr::Pure)) {`.
  **L4022 CN**: 开始一个函数、方法、lambda 或结构化作用域：`proc2->attrs.test(Procedure::Attr::Pure)) {`。
- **L4023 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4023 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4024 EN**: Executes a standalone statement or declaration: `"Module subprogram '%s' and its corresponding interface body are not both PURE"_err_en_US);`.
  **L4024 CN**: 执行一条独立语句或声明：`"Module subprogram '%s' and its corresponding interface body are not both PURE"_err_en_US);`。
- **L4025 EN**: Closes the current lexical scope or compound statement.
  **L4025 CN**: 结束当前词法作用域或复合语句块。
- **L4026 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4026 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4027 EN**: Starts a function, method, lambda, or structured scope: `proc2->attrs.test(Procedure::Attr::Elemental)) {`.
  **L4027 CN**: 开始一个函数、方法、lambda 或结构化作用域：`proc2->attrs.test(Procedure::Attr::Elemental)) {`。
- **L4028 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4028 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4029 EN**: Executes a standalone statement or declaration: `"Module subprogram '%s' and its corresponding interface body are not both ELEMENTAL"_err_en_US);`.
  **L4029 CN**: 执行一条独立语句或声明：`"Module subprogram '%s' and its corresponding interface body are not both ELEMENTAL"_err_en_US);`。
- **L4030 EN**: Closes the current lexical scope or compound statement.
  **L4030 CN**: 结束当前词法作用域或复合语句块。
- **L4031 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4031 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4032 EN**: Starts a function, method, lambda, or structured scope: `proc2->attrs.test(Procedure::Attr::BindC)) {`.
  **L4032 CN**: 开始一个函数、方法、lambda 或结构化作用域：`proc2->attrs.test(Procedure::Attr::BindC)) {`。

### Lines 4033-4056

````cpp
    Say(symbol1, symbol2,
        "Module subprogram '%s' and its corresponding interface body are not both BIND(C)"_err_en_US);
  }
  if (proc1->functionResult && proc2->functionResult) {
    std::string whyNot;
    if (!proc1->functionResult->IsCompatibleWith(
            *proc2->functionResult, &whyNot)) {
      Say(symbol1, symbol2,
          "Result of function '%s' is not compatible with the result of the corresponding interface body: %s"_err_en_US,
          whyNot);
    }
  }
  for (int i{0}; i < nargs1; ++i) {
    const Symbol *arg1{args1[i]};
    const Symbol *arg2{args2[i]};
    if (arg1 && !arg2) {
      Say(symbol1, symbol2,
          "Dummy argument %2$d of '%1$s' is not an alternate return indicator"
          " but the corresponding argument in the interface body is"_err_en_US,
          i + 1);
    } else if (!arg1 && arg2) {
      Say(symbol1, symbol2,
          "Dummy argument %2$d of '%1$s' is an alternate return indicator but"
          " the corresponding argument in the interface body is not"_err_en_US,
````
- **L4033 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4033 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4034 EN**: Executes a call or declaration centered on `BIND`.
  **L4034 CN**: 执行以 `BIND` 为核心的调用或声明。
- **L4035 EN**: Closes the current lexical scope or compound statement.
  **L4035 CN**: 结束当前词法作用域或复合语句块。
- **L4036 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4036 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4037 EN**: Executes a standalone statement or declaration: `std::string whyNot;`.
  **L4037 CN**: 执行一条独立语句或声明：`std::string whyNot;`。
- **L4038 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4038 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4039 EN**: Comment explains nearby logic, intent, or metadata: `proc2->functionResult, &whyNot)) {`.
  **L4039 CN**: 注释说明附近代码的逻辑、意图或元数据：`proc2->functionResult, &whyNot)) {`。
- **L4040 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4040 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4041 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Result of function '%s' is not compatible with the result of the corresponding interface body: %s"_err_en_US,`.
  **L4041 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Result of function '%s' is not compatible with the result of the corresponding interface body: %s"_err_en_US,`。
- **L4042 EN**: Executes a standalone statement or declaration: `whyNot);`.
  **L4042 CN**: 执行一条独立语句或声明：`whyNot);`。
- **L4043 EN**: Closes the current lexical scope or compound statement.
  **L4043 CN**: 结束当前词法作用域或复合语句块。
- **L4044 EN**: Closes the current lexical scope or compound statement.
  **L4044 CN**: 结束当前词法作用域或复合语句块。
- **L4045 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4045 CN**: 开始 `for` 控制流语句并计算其条件。
- **L4046 EN**: Executes a standalone statement or declaration: `const Symbol *arg1{args1[i]};`.
  **L4046 CN**: 执行一条独立语句或声明：`const Symbol *arg1{args1[i]};`。
- **L4047 EN**: Executes a standalone statement or declaration: `const Symbol *arg2{args2[i]};`.
  **L4047 CN**: 执行一条独立语句或声明：`const Symbol *arg2{args2[i]};`。
- **L4048 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4048 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4049 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4049 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4050 EN**: Continues the surrounding expression or declaration: `"Dummy argument %2$d of '%1$s' is not an alternate return indicator"`.
  **L4050 CN**: 继续构造周围的表达式或声明：`"Dummy argument %2$d of '%1$s' is not an alternate return indicator"`。
- **L4051 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" but the corresponding argument in the interface body is"_err_en_US,`.
  **L4051 CN**: 继续一个多行参数列表、初始化器或聚合项：`" but the corresponding argument in the interface body is"_err_en_US,`。
- **L4052 EN**: Executes a standalone statement or declaration: `i + 1);`.
  **L4052 CN**: 执行一条独立语句或声明：`i + 1);`。
- **L4053 EN**: Transitions from the previous branch into an `else if` condition.
  **L4053 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4054 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4054 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4055 EN**: Continues the surrounding expression or declaration: `"Dummy argument %2$d of '%1$s' is an alternate return indicator but"`.
  **L4055 CN**: 继续构造周围的表达式或声明：`"Dummy argument %2$d of '%1$s' is an alternate return indicator but"`。
- **L4056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" the corresponding argument in the interface body is not"_err_en_US,`.
  **L4056 CN**: 继续一个多行参数列表、初始化器或聚合项：`" the corresponding argument in the interface body is not"_err_en_US,`。

### Lines 4057-4080

````cpp
          i + 1);
    } else if (arg1 && arg2) {
      SourceName name1{arg1->name()};
      SourceName name2{arg2->name()};
      if (name1 != name2) {
        Say(*arg1, *arg2,
            "Dummy argument name '%s' does not match corresponding name '%s'"
            " in interface body"_err_en_US,
            name2);
      } else {
        CheckDummyArg(
            *arg1, *arg2, proc1->dummyArguments[i], proc2->dummyArguments[i]);
      }
    }
  }
}

void SubprogramMatchHelper::CheckDummyArg(const Symbol &symbol1,
    const Symbol &symbol2, const DummyArgument &arg1,
    const DummyArgument &arg2) {
  common::visit(
      common::visitors{
          [&](const DummyDataObject &obj1, const DummyDataObject &obj2) {
            CheckDummyDataObject(symbol1, symbol2, obj1, obj2);
````
- **L4057 EN**: Executes a standalone statement or declaration: `i + 1);`.
  **L4057 CN**: 执行一条独立语句或声明：`i + 1);`。
- **L4058 EN**: Transitions from the previous branch into an `else if` condition.
  **L4058 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4059 EN**: Executes a call or declaration centered on `name1{arg1->name`.
  **L4059 CN**: 执行以 `name1{arg1->name` 为核心的调用或声明。
- **L4060 EN**: Executes a call or declaration centered on `name2{arg2->name`.
  **L4060 CN**: 执行以 `name2{arg2->name` 为核心的调用或声明。
- **L4061 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4061 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4062 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(*arg1, *arg2,`.
  **L4062 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(*arg1, *arg2,`。
- **L4063 EN**: Continues the surrounding expression or declaration: `"Dummy argument name '%s' does not match corresponding name '%s'"`.
  **L4063 CN**: 继续构造周围的表达式或声明：`"Dummy argument name '%s' does not match corresponding name '%s'"`。
- **L4064 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" in interface body"_err_en_US,`.
  **L4064 CN**: 继续一个多行参数列表、初始化器或聚合项：`" in interface body"_err_en_US,`。
- **L4065 EN**: Executes a standalone statement or declaration: `name2);`.
  **L4065 CN**: 执行一条独立语句或声明：`name2);`。
- **L4066 EN**: Transitions from the previous branch into the alternative path.
  **L4066 CN**: 从前一个分支过渡到备选路径。
- **L4067 EN**: Continues logic associated with callable symbol `CheckDummyArg`.
  **L4067 CN**: 继续与可调用符号 `CheckDummyArg` 相关的逻辑。
- **L4068 EN**: Comment explains nearby logic, intent, or metadata: `arg1, *arg2, proc1->dummyArguments[i], proc2->dummyArguments[i]);`.
  **L4068 CN**: 注释说明附近代码的逻辑、意图或元数据：`arg1, *arg2, proc1->dummyArguments[i], proc2->dummyArguments[i]);`。
- **L4069 EN**: Closes the current lexical scope or compound statement.
  **L4069 CN**: 结束当前词法作用域或复合语句块。
- **L4070 EN**: Closes the current lexical scope or compound statement.
  **L4070 CN**: 结束当前词法作用域或复合语句块。
- **L4071 EN**: Closes the current lexical scope or compound statement.
  **L4071 CN**: 结束当前词法作用域或复合语句块。
- **L4072 EN**: Closes the current lexical scope or compound statement.
  **L4072 CN**: 结束当前词法作用域或复合语句块。
- **L4073 EN**: Blank line separating nearby declarations or logic blocks.
  **L4073 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4074 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void SubprogramMatchHelper::CheckDummyArg(const Symbol &symbol1,`.
  **L4074 CN**: 继续一个多行参数列表、初始化器或聚合项：`void SubprogramMatchHelper::CheckDummyArg(const Symbol &symbol1,`。
- **L4075 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Symbol &symbol2, const DummyArgument &arg1,`.
  **L4075 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Symbol &symbol2, const DummyArgument &arg1,`。
- **L4076 EN**: Continues the surrounding expression or declaration: `const DummyArgument &arg2) {`.
  **L4076 CN**: 继续构造周围的表达式或声明：`const DummyArgument &arg2) {`。
- **L4077 EN**: Continues logic associated with callable symbol `visit`.
  **L4077 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L4078 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L4078 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L4079 EN**: Starts a function, method, lambda, or structured scope: `[&](const DummyDataObject &obj1, const DummyDataObject &obj2) {`.
  **L4079 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const DummyDataObject &obj1, const DummyDataObject &obj2) {`。
- **L4080 EN**: Executes a call or declaration centered on `CheckDummyDataObject`.
  **L4080 CN**: 执行以 `CheckDummyDataObject` 为核心的调用或声明。

### Lines 4081-4104

````cpp
          },
          [&](const DummyProcedure &proc1, const DummyProcedure &proc2) {
            CheckDummyProcedure(symbol1, symbol2, proc1, proc2);
          },
          [&](const DummyDataObject &, const auto &) {
            Say(symbol1, symbol2,
                "Dummy argument '%s' is a data object; the corresponding"
                " argument in the interface body is not"_err_en_US);
          },
          [&](const DummyProcedure &, const auto &) {
            Say(symbol1, symbol2,
                "Dummy argument '%s' is a procedure; the corresponding"
                " argument in the interface body is not"_err_en_US);
          },
          [&](const auto &, const auto &) {
            llvm_unreachable("Dummy arguments are not data objects or"
                             "procedures");
          },
      },
      arg1.u, arg2.u);
}

void SubprogramMatchHelper::CheckDummyDataObject(const Symbol &symbol1,
    const Symbol &symbol2, const DummyDataObject &obj1,
````
- **L4081 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L4081 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L4082 EN**: Starts a function, method, lambda, or structured scope: `[&](const DummyProcedure &proc1, const DummyProcedure &proc2) {`.
  **L4082 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const DummyProcedure &proc1, const DummyProcedure &proc2) {`。
- **L4083 EN**: Executes a call or declaration centered on `CheckDummyProcedure`.
  **L4083 CN**: 执行以 `CheckDummyProcedure` 为核心的调用或声明。
- **L4084 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L4084 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L4085 EN**: Starts a function, method, lambda, or structured scope: `[&](const DummyDataObject &, const auto &) {`.
  **L4085 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const DummyDataObject &, const auto &) {`。
- **L4086 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4086 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4087 EN**: Continues the surrounding expression or declaration: `"Dummy argument '%s' is a data object; the corresponding"`.
  **L4087 CN**: 继续构造周围的表达式或声明：`"Dummy argument '%s' is a data object; the corresponding"`。
- **L4088 EN**: Executes a standalone statement or declaration: `" argument in the interface body is not"_err_en_US);`.
  **L4088 CN**: 执行一条独立语句或声明：`" argument in the interface body is not"_err_en_US);`。
- **L4089 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L4089 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L4090 EN**: Starts a function, method, lambda, or structured scope: `[&](const DummyProcedure &, const auto &) {`.
  **L4090 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const DummyProcedure &, const auto &) {`。
- **L4091 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4091 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4092 EN**: Continues the surrounding expression or declaration: `"Dummy argument '%s' is a procedure; the corresponding"`.
  **L4092 CN**: 继续构造周围的表达式或声明：`"Dummy argument '%s' is a procedure; the corresponding"`。
- **L4093 EN**: Executes a standalone statement or declaration: `" argument in the interface body is not"_err_en_US);`.
  **L4093 CN**: 执行一条独立语句或声明：`" argument in the interface body is not"_err_en_US);`。
- **L4094 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L4094 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L4095 EN**: Starts a function, method, lambda, or structured scope: `[&](const auto &, const auto &) {`.
  **L4095 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const auto &, const auto &) {`。
- **L4096 EN**: Marks this control path as unreachable to LLVM.
  **L4096 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L4097 EN**: Executes a standalone statement or declaration: `"procedures");`.
  **L4097 CN**: 执行一条独立语句或声明：`"procedures");`。
- **L4098 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L4098 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L4099 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L4099 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L4100 EN**: Executes a standalone statement or declaration: `arg1.u, arg2.u);`.
  **L4100 CN**: 执行一条独立语句或声明：`arg1.u, arg2.u);`。
- **L4101 EN**: Closes the current lexical scope or compound statement.
  **L4101 CN**: 结束当前词法作用域或复合语句块。
- **L4102 EN**: Blank line separating nearby declarations or logic blocks.
  **L4102 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void SubprogramMatchHelper::CheckDummyDataObject(const Symbol &symbol1,`.
  **L4103 CN**: 继续一个多行参数列表、初始化器或聚合项：`void SubprogramMatchHelper::CheckDummyDataObject(const Symbol &symbol1,`。
- **L4104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Symbol &symbol2, const DummyDataObject &obj1,`.
  **L4104 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Symbol &symbol2, const DummyDataObject &obj1,`。

### Lines 4105-4128

````cpp
    const DummyDataObject &obj2) {
  if (!CheckSameIntent(symbol1, symbol2, obj1.intent, obj2.intent)) {
  } else if (!CheckSameAttrs(symbol1, symbol2, obj1.attrs, obj2.attrs)) {
  } else if (!obj1.type.type().IsEquivalentTo(obj2.type.type())) {
    Say(symbol1, symbol2,
        "Dummy argument '%s' has type %s; the corresponding argument in the interface body has distinct type %s"_err_en_US,
        obj1.type.type().AsFortran(), obj2.type.type().AsFortran());
  } else if (!ShapesAreCompatible(obj1, obj2)) {
    Say(symbol1, symbol2,
        "The shape of dummy argument '%s' does not match the shape of the"
        " corresponding argument in the interface body"_err_en_US);
  }
  // TODO: coshape
}

void SubprogramMatchHelper::CheckDummyProcedure(const Symbol &symbol1,
    const Symbol &symbol2, const DummyProcedure &proc1,
    const DummyProcedure &proc2) {
  std::string whyNot;
  if (!CheckSameIntent(symbol1, symbol2, proc1.intent, proc2.intent)) {
  } else if (!CheckSameAttrs(symbol1, symbol2, proc1.attrs, proc2.attrs)) {
  } else if (!proc2.IsCompatibleWith(proc1, &whyNot)) {
    Say(symbol1, symbol2,
        "Dummy procedure '%s' is not compatible with the corresponding argument in the interface body: %s"_err_en_US,
````
- **L4105 EN**: Continues the surrounding expression or declaration: `const DummyDataObject &obj2) {`.
  **L4105 CN**: 继续构造周围的表达式或声明：`const DummyDataObject &obj2) {`。
- **L4106 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4106 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4107 EN**: Transitions from the previous branch into an `else if` condition.
  **L4107 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4108 EN**: Transitions from the previous branch into an `else if` condition.
  **L4108 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4109 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy argument '%s' has type %s; the corresponding argument in the interface body has distinct type %s"_err_en_US,`.
  **L4110 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy argument '%s' has type %s; the corresponding argument in the interface body has distinct type %s"_err_en_US,`。
- **L4111 EN**: Executes a call or declaration centered on `obj1.type.type`.
  **L4111 CN**: 执行以 `obj1.type.type` 为核心的调用或声明。
- **L4112 EN**: Transitions from the previous branch into an `else if` condition.
  **L4112 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4113 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4114 EN**: Continues the surrounding expression or declaration: `"The shape of dummy argument '%s' does not match the shape of the"`.
  **L4114 CN**: 继续构造周围的表达式或声明：`"The shape of dummy argument '%s' does not match the shape of the"`。
- **L4115 EN**: Executes a standalone statement or declaration: `" corresponding argument in the interface body"_err_en_US);`.
  **L4115 CN**: 执行一条独立语句或声明：`" corresponding argument in the interface body"_err_en_US);`。
- **L4116 EN**: Closes the current lexical scope or compound statement.
  **L4116 CN**: 结束当前词法作用域或复合语句块。
- **L4117 EN**: Comment records a pending task or caution: `TODO: coshape`.
  **L4117 CN**: 注释记录待办事项或注意点：`TODO: coshape`。
- **L4118 EN**: Closes the current lexical scope or compound statement.
  **L4118 CN**: 结束当前词法作用域或复合语句块。
- **L4119 EN**: Blank line separating nearby declarations or logic blocks.
  **L4119 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void SubprogramMatchHelper::CheckDummyProcedure(const Symbol &symbol1,`.
  **L4120 CN**: 继续一个多行参数列表、初始化器或聚合项：`void SubprogramMatchHelper::CheckDummyProcedure(const Symbol &symbol1,`。
- **L4121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Symbol &symbol2, const DummyProcedure &proc1,`.
  **L4121 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Symbol &symbol2, const DummyProcedure &proc1,`。
- **L4122 EN**: Continues the surrounding expression or declaration: `const DummyProcedure &proc2) {`.
  **L4122 CN**: 继续构造周围的表达式或声明：`const DummyProcedure &proc2) {`。
- **L4123 EN**: Executes a standalone statement or declaration: `std::string whyNot;`.
  **L4123 CN**: 执行一条独立语句或声明：`std::string whyNot;`。
- **L4124 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4124 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4125 EN**: Transitions from the previous branch into an `else if` condition.
  **L4125 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4126 EN**: Transitions from the previous branch into an `else if` condition.
  **L4126 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4127 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy procedure '%s' is not compatible with the corresponding argument in the interface body: %s"_err_en_US,`.
  **L4128 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy procedure '%s' is not compatible with the corresponding argument in the interface body: %s"_err_en_US,`。

### Lines 4129-4152

````cpp
        whyNot);
  } else if (proc1 != proc2) {
    evaluate::AttachDeclaration(
        symbol1.owner().context().Warn(
            common::UsageWarning::MismatchingDummyProcedure,
            "Dummy procedure '%s' does not exactly match the corresponding argument in the interface body"_warn_en_US,
            symbol1.name()),
        symbol2);
  }
}

bool SubprogramMatchHelper::CheckSameIntent(const Symbol &symbol1,
    const Symbol &symbol2, common::Intent intent1, common::Intent intent2) {
  if (intent1 == intent2) {
    return true;
  } else {
    Say(symbol1, symbol2,
        "The intent of dummy argument '%s' does not match the intent"
        " of the corresponding argument in the interface body"_err_en_US);
    return false;
  }
}

// Report an error referring to first symbol with declaration of second symbol
````
- **L4129 EN**: Executes a standalone statement or declaration: `whyNot);`.
  **L4129 CN**: 执行一条独立语句或声明：`whyNot);`。
- **L4130 EN**: Transitions from the previous branch into an `else if` condition.
  **L4130 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4131 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L4131 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L4132 EN**: Continues logic associated with callable symbol `owner`.
  **L4132 CN**: 继续与可调用符号 `owner` 相关的逻辑。
- **L4133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::UsageWarning::MismatchingDummyProcedure,`.
  **L4133 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::UsageWarning::MismatchingDummyProcedure,`。
- **L4134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Dummy procedure '%s' does not exactly match the corresponding argument in the interface body"_warn_en_US,`.
  **L4134 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Dummy procedure '%s' does not exactly match the corresponding argument in the interface body"_warn_en_US,`。
- **L4135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol1.name()),`.
  **L4135 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol1.name()),`。
- **L4136 EN**: Executes a standalone statement or declaration: `symbol2);`.
  **L4136 CN**: 执行一条独立语句或声明：`symbol2);`。
- **L4137 EN**: Closes the current lexical scope or compound statement.
  **L4137 CN**: 结束当前词法作用域或复合语句块。
- **L4138 EN**: Closes the current lexical scope or compound statement.
  **L4138 CN**: 结束当前词法作用域或复合语句块。
- **L4139 EN**: Blank line separating nearby declarations or logic blocks.
  **L4139 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool SubprogramMatchHelper::CheckSameIntent(const Symbol &symbol1,`.
  **L4140 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool SubprogramMatchHelper::CheckSameIntent(const Symbol &symbol1,`。
- **L4141 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol2, common::Intent intent1, common::Intent intent2) {`.
  **L4141 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol2, common::Intent intent1, common::Intent intent2) {`。
- **L4142 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4142 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4143 EN**: Returns from the current function with `true`.
  **L4143 CN**: 以 `true` 从当前函数返回。
- **L4144 EN**: Transitions from the previous branch into the alternative path.
  **L4144 CN**: 从前一个分支过渡到备选路径。
- **L4145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4145 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4146 EN**: Continues the surrounding expression or declaration: `"The intent of dummy argument '%s' does not match the intent"`.
  **L4146 CN**: 继续构造周围的表达式或声明：`"The intent of dummy argument '%s' does not match the intent"`。
- **L4147 EN**: Executes a standalone statement or declaration: `" of the corresponding argument in the interface body"_err_en_US);`.
  **L4147 CN**: 执行一条独立语句或声明：`" of the corresponding argument in the interface body"_err_en_US);`。
- **L4148 EN**: Returns from the current function with `false`.
  **L4148 CN**: 以 `false` 从当前函数返回。
- **L4149 EN**: Closes the current lexical scope or compound statement.
  **L4149 CN**: 结束当前词法作用域或复合语句块。
- **L4150 EN**: Closes the current lexical scope or compound statement.
  **L4150 CN**: 结束当前词法作用域或复合语句块。
- **L4151 EN**: Blank line separating nearby declarations or logic blocks.
  **L4151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4152 EN**: Comment explains nearby logic, intent, or metadata: `Report an error referring to first symbol with declaration of second symbol`.
  **L4152 CN**: 注释说明附近代码的逻辑、意图或元数据：`Report an error referring to first symbol with declaration of second symbol`。

### Lines 4153-4176

````cpp
template <typename... A>
void SubprogramMatchHelper::Say(const Symbol &symbol1, const Symbol &symbol2,
    parser::MessageFixedText &&text, A &&...args) {
  auto &message{context().Say(symbol1.name(), std::move(text), symbol1.name(),
      std::forward<A>(args)...)};
  evaluate::AttachDeclaration(message, symbol2);
}

template <typename ATTRS>
bool SubprogramMatchHelper::CheckSameAttrs(
    const Symbol &symbol1, const Symbol &symbol2, ATTRS attrs1, ATTRS attrs2) {
  if (attrs1 == attrs2) {
    return true;
  }
  attrs1.IterateOverMembers([&](auto attr) {
    if (!attrs2.test(attr)) {
      Say(symbol1, symbol2,
          "Dummy argument '%s' has the %s attribute; the corresponding"
          " argument in the interface body does not"_err_en_US,
          AsFortran(attr));
    }
  });
  attrs2.IterateOverMembers([&](auto attr) {
    if (!attrs1.test(attr)) {
````
- **L4153 EN**: Introduces template parameters or specialization context: `template <typename... A>`.
  **L4153 CN**: 为后续声明引入模板参数或特化上下文：`template <typename... A>`。
- **L4154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void SubprogramMatchHelper::Say(const Symbol &symbol1, const Symbol &symbol2,`.
  **L4154 CN**: 继续一个多行参数列表、初始化器或聚合项：`void SubprogramMatchHelper::Say(const Symbol &symbol1, const Symbol &symbol2,`。
- **L4155 EN**: Continues the surrounding expression or declaration: `parser::MessageFixedText &&text, A &&...args) {`.
  **L4155 CN**: 继续构造周围的表达式或声明：`parser::MessageFixedText &&text, A &&...args) {`。
- **L4156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto &message{context().Say(symbol1.name(), std::move(text), symbol1.name(),`.
  **L4156 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto &message{context().Say(symbol1.name(), std::move(text), symbol1.name(),`。
- **L4157 EN**: Executes a call or declaration centered on `std::forward<A>`.
  **L4157 CN**: 执行以 `std::forward<A>` 为核心的调用或声明。
- **L4158 EN**: Executes a call or declaration centered on `evaluate::AttachDeclaration`.
  **L4158 CN**: 执行以 `evaluate::AttachDeclaration` 为核心的调用或声明。
- **L4159 EN**: Closes the current lexical scope or compound statement.
  **L4159 CN**: 结束当前词法作用域或复合语句块。
- **L4160 EN**: Blank line separating nearby declarations or logic blocks.
  **L4160 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4161 EN**: Introduces template parameters or specialization context: `template <typename ATTRS>`.
  **L4161 CN**: 为后续声明引入模板参数或特化上下文：`template <typename ATTRS>`。
- **L4162 EN**: Continues logic associated with callable symbol `CheckSameAttrs`.
  **L4162 CN**: 继续与可调用符号 `CheckSameAttrs` 相关的逻辑。
- **L4163 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol1, const Symbol &symbol2, ATTRS attrs1, ATTRS attrs2) {`.
  **L4163 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol1, const Symbol &symbol2, ATTRS attrs1, ATTRS attrs2) {`。
- **L4164 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4164 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4165 EN**: Returns from the current function with `true`.
  **L4165 CN**: 以 `true` 从当前函数返回。
- **L4166 EN**: Closes the current lexical scope or compound statement.
  **L4166 CN**: 结束当前词法作用域或复合语句块。
- **L4167 EN**: Starts a function, method, lambda, or structured scope: `attrs1.IterateOverMembers([&](auto attr) {`.
  **L4167 CN**: 开始一个函数、方法、lambda 或结构化作用域：`attrs1.IterateOverMembers([&](auto attr) {`。
- **L4168 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4168 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4169 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4170 EN**: Continues the surrounding expression or declaration: `"Dummy argument '%s' has the %s attribute; the corresponding"`.
  **L4170 CN**: 继续构造周围的表达式或声明：`"Dummy argument '%s' has the %s attribute; the corresponding"`。
- **L4171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" argument in the interface body does not"_err_en_US,`.
  **L4171 CN**: 继续一个多行参数列表、初始化器或聚合项：`" argument in the interface body does not"_err_en_US,`。
- **L4172 EN**: Executes a call or declaration centered on `AsFortran`.
  **L4172 CN**: 执行以 `AsFortran` 为核心的调用或声明。
- **L4173 EN**: Closes the current lexical scope or compound statement.
  **L4173 CN**: 结束当前词法作用域或复合语句块。
- **L4174 EN**: Executes a standalone statement or declaration: `});`.
  **L4174 CN**: 执行一条独立语句或声明：`});`。
- **L4175 EN**: Starts a function, method, lambda, or structured scope: `attrs2.IterateOverMembers([&](auto attr) {`.
  **L4175 CN**: 开始一个函数、方法、lambda 或结构化作用域：`attrs2.IterateOverMembers([&](auto attr) {`。
- **L4176 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4176 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 4177-4200

````cpp
      Say(symbol1, symbol2,
          "Dummy argument '%s' does not have the %s attribute; the"
          " corresponding argument in the interface body does"_err_en_US,
          AsFortran(attr));
    }
  });
  return false;
}

bool SubprogramMatchHelper::ShapesAreCompatible(
    const DummyDataObject &obj1, const DummyDataObject &obj2) {
  return characteristics::ShapesAreCompatible(
      FoldShape(obj1.type.shape()), FoldShape(obj2.type.shape()));
}

evaluate::Shape SubprogramMatchHelper::FoldShape(const evaluate::Shape &shape) {
  evaluate::Shape result;
  for (const auto &extent : shape) {
    result.emplace_back(
        evaluate::Fold(context().foldingContext(), common::Clone(extent)));
  }
  return result;
}

````
- **L4177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Say(symbol1, symbol2,`.
  **L4177 CN**: 继续一个多行参数列表、初始化器或聚合项：`Say(symbol1, symbol2,`。
- **L4178 EN**: Continues the surrounding expression or declaration: `"Dummy argument '%s' does not have the %s attribute; the"`.
  **L4178 CN**: 继续构造周围的表达式或声明：`"Dummy argument '%s' does not have the %s attribute; the"`。
- **L4179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" corresponding argument in the interface body does"_err_en_US,`.
  **L4179 CN**: 继续一个多行参数列表、初始化器或聚合项：`" corresponding argument in the interface body does"_err_en_US,`。
- **L4180 EN**: Executes a call or declaration centered on `AsFortran`.
  **L4180 CN**: 执行以 `AsFortran` 为核心的调用或声明。
- **L4181 EN**: Closes the current lexical scope or compound statement.
  **L4181 CN**: 结束当前词法作用域或复合语句块。
- **L4182 EN**: Executes a standalone statement or declaration: `});`.
  **L4182 CN**: 执行一条独立语句或声明：`});`。
- **L4183 EN**: Returns from the current function with `false`.
  **L4183 CN**: 以 `false` 从当前函数返回。
- **L4184 EN**: Closes the current lexical scope or compound statement.
  **L4184 CN**: 结束当前词法作用域或复合语句块。
- **L4185 EN**: Blank line separating nearby declarations or logic blocks.
  **L4185 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4186 EN**: Continues logic associated with callable symbol `ShapesAreCompatible`.
  **L4186 CN**: 继续与可调用符号 `ShapesAreCompatible` 相关的逻辑。
- **L4187 EN**: Continues the surrounding expression or declaration: `const DummyDataObject &obj1, const DummyDataObject &obj2) {`.
  **L4187 CN**: 继续构造周围的表达式或声明：`const DummyDataObject &obj1, const DummyDataObject &obj2) {`。
- **L4188 EN**: Returns from the current function with `characteristics::ShapesAreCompatible(`.
  **L4188 CN**: 以 `characteristics::ShapesAreCompatible(` 从当前函数返回。
- **L4189 EN**: Executes a call or declaration centered on `FoldShape`.
  **L4189 CN**: 执行以 `FoldShape` 为核心的调用或声明。
- **L4190 EN**: Closes the current lexical scope or compound statement.
  **L4190 CN**: 结束当前词法作用域或复合语句块。
- **L4191 EN**: Blank line separating nearby declarations or logic blocks.
  **L4191 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4192 EN**: Starts a function, method, lambda, or structured scope: `evaluate::Shape SubprogramMatchHelper::FoldShape(const evaluate::Shape &shape) {`.
  **L4192 CN**: 开始一个函数、方法、lambda 或结构化作用域：`evaluate::Shape SubprogramMatchHelper::FoldShape(const evaluate::Shape &shape) {`。
- **L4193 EN**: Executes a standalone statement or declaration: `evaluate::Shape result;`.
  **L4193 CN**: 执行一条独立语句或声明：`evaluate::Shape result;`。
- **L4194 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4194 CN**: 开始 `for` 控制流语句并计算其条件。
- **L4195 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L4195 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L4196 EN**: Executes a call or declaration centered on `evaluate::Fold`.
  **L4196 CN**: 执行以 `evaluate::Fold` 为核心的调用或声明。
- **L4197 EN**: Closes the current lexical scope or compound statement.
  **L4197 CN**: 结束当前词法作用域或复合语句块。
- **L4198 EN**: Returns from the current function with `result`.
  **L4198 CN**: 以 `result` 从当前函数返回。
- **L4199 EN**: Closes the current lexical scope or compound statement.
  **L4199 CN**: 结束当前词法作用域或复合语句块。
- **L4200 EN**: Blank line separating nearby declarations or logic blocks.
  **L4200 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 4201-4224

````cpp
void DistinguishabilityHelper::Add(const Symbol &generic, GenericKind kind,
    const Symbol &specific, const Procedure &procedure) {
  const Symbol &ultimate{specific.GetUltimate()};
  if (!context_.HasError(ultimate)) {
    nameToSpecifics_[generic.name()].emplace(
        &ultimate, ProcedureInfo{kind, procedure});
  }
}

void DistinguishabilityHelper::Check(const Scope &scope) {
  if (FindModuleFileContaining(scope)) {
    // Distinguishability was checked when the module was created;
    // don't let optional warnings then become errors now.
    return;
  }
  for (const auto &[name, info] : nameToSpecifics_) {
    for (auto iter1{info.begin()}; iter1 != info.end(); ++iter1) {
      const auto &[ultimate, procInfo]{*iter1};
      const auto &[kind, proc]{procInfo};
      for (auto iter2{iter1}; ++iter2 != info.end();) {
        const auto &[ultimate2, procInfo2]{*iter2};
        if (&*ultimate == &*ultimate2) {
          continue; // ok, actually the same procedure/binding
        } else if (const auto *binding1{
````
- **L4201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void DistinguishabilityHelper::Add(const Symbol &generic, GenericKind kind,`.
  **L4201 CN**: 继续一个多行参数列表、初始化器或聚合项：`void DistinguishabilityHelper::Add(const Symbol &generic, GenericKind kind,`。
- **L4202 EN**: Continues the surrounding expression or declaration: `const Symbol &specific, const Procedure &procedure) {`.
  **L4202 CN**: 继续构造周围的表达式或声明：`const Symbol &specific, const Procedure &procedure) {`。
- **L4203 EN**: Executes a call or declaration centered on `&ultimate{specific.GetUltimate`.
  **L4203 CN**: 执行以 `&ultimate{specific.GetUltimate` 为核心的调用或声明。
- **L4204 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4204 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4205 EN**: Continues logic associated with callable symbol `name`.
  **L4205 CN**: 继续与可调用符号 `name` 相关的逻辑。
- **L4206 EN**: Executes a standalone statement or declaration: `&ultimate, ProcedureInfo{kind, procedure});`.
  **L4206 CN**: 执行一条独立语句或声明：`&ultimate, ProcedureInfo{kind, procedure});`。
- **L4207 EN**: Closes the current lexical scope or compound statement.
  **L4207 CN**: 结束当前词法作用域或复合语句块。
- **L4208 EN**: Closes the current lexical scope or compound statement.
  **L4208 CN**: 结束当前词法作用域或复合语句块。
- **L4209 EN**: Blank line separating nearby declarations or logic blocks.
  **L4209 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4210 EN**: Starts a function, method, lambda, or structured scope: `void DistinguishabilityHelper::Check(const Scope &scope) {`.
  **L4210 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void DistinguishabilityHelper::Check(const Scope &scope) {`。
- **L4211 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4211 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4212 EN**: Comment explains nearby logic, intent, or metadata: `Distinguishability was checked when the module was created;`.
  **L4212 CN**: 注释说明附近代码的逻辑、意图或元数据：`Distinguishability was checked when the module was created;`。
- **L4213 EN**: Comment explains nearby logic, intent, or metadata: `don't let optional warnings then become errors now.`.
  **L4213 CN**: 注释说明附近代码的逻辑、意图或元数据：`don't let optional warnings then become errors now.`。
- **L4214 EN**: Returns from the current function with `void`.
  **L4214 CN**: 以 `void` 从当前函数返回。
- **L4215 EN**: Closes the current lexical scope or compound statement.
  **L4215 CN**: 结束当前词法作用域或复合语句块。
- **L4216 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4216 CN**: 开始 `for` 控制流语句并计算其条件。
- **L4217 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4217 CN**: 开始 `for` 控制流语句并计算其条件。
- **L4218 EN**: Executes a standalone statement or declaration: `const auto &[ultimate, procInfo]{*iter1};`.
  **L4218 CN**: 执行一条独立语句或声明：`const auto &[ultimate, procInfo]{*iter1};`。
- **L4219 EN**: Executes a standalone statement or declaration: `const auto &[kind, proc]{procInfo};`.
  **L4219 CN**: 执行一条独立语句或声明：`const auto &[kind, proc]{procInfo};`。
- **L4220 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L4220 CN**: 开始 `for` 控制流语句并计算其条件。
- **L4221 EN**: Executes a standalone statement or declaration: `const auto &[ultimate2, procInfo2]{*iter2};`.
  **L4221 CN**: 执行一条独立语句或声明：`const auto &[ultimate2, procInfo2]{*iter2};`。
- **L4222 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4222 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4223 EN**: Skips to the next loop iteration.
  **L4223 CN**: 跳到下一次循环迭代。
- **L4224 EN**: Transitions from the previous branch into an `else if` condition.
  **L4224 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 4225-4248

````cpp
                       ultimate->detailsIf<ProcBindingDetails>()}) {
          if (const auto *binding2{
                  ultimate2->detailsIf<ProcBindingDetails>()}) {
            if (&binding1->symbol().GetUltimate() ==
                &binding2->symbol().GetUltimate()) {
              continue; // ok, (NOPASS) bindings resolve identically
            } else if (ultimate->name() == ultimate2->name()) {
              continue; // override, possibly of DEFERRED
            }
          }
        } else if (ultimate->has<ProcBindingDetails>() &&
            ultimate2->has<ProcBindingDetails>() &&
            ultimate->name() == ultimate2->name()) {
          continue; // override, possibly of DEFERRED
        }
        auto distinguishable{kind.IsName()
                ? evaluate::characteristics::Distinguishable
                : evaluate::characteristics::DistinguishableOpOrAssign};
        std::optional<bool> distinct{distinguishable(
            context_.languageFeatures(), proc, procInfo2.procedure)};
        if (!distinct.value_or(false)) {
          SayNotDistinguishable(GetTopLevelUnitContaining(scope), name, kind,
              *ultimate, *ultimate2, distinct.has_value());
        }
````
- **L4225 EN**: Starts a function, method, lambda, or structured scope: `ultimate->detailsIf<ProcBindingDetails>()}) {`.
  **L4225 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ultimate->detailsIf<ProcBindingDetails>()}) {`。
- **L4226 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4226 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4227 EN**: Starts a function, method, lambda, or structured scope: `ultimate2->detailsIf<ProcBindingDetails>()}) {`.
  **L4227 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ultimate2->detailsIf<ProcBindingDetails>()}) {`。
- **L4228 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4228 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4229 EN**: Starts a function, method, lambda, or structured scope: `&binding2->symbol().GetUltimate()) {`.
  **L4229 CN**: 开始一个函数、方法、lambda 或结构化作用域：`&binding2->symbol().GetUltimate()) {`。
- **L4230 EN**: Skips to the next loop iteration.
  **L4230 CN**: 跳到下一次循环迭代。
- **L4231 EN**: Transitions from the previous branch into an `else if` condition.
  **L4231 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4232 EN**: Skips to the next loop iteration.
  **L4232 CN**: 跳到下一次循环迭代。
- **L4233 EN**: Closes the current lexical scope or compound statement.
  **L4233 CN**: 结束当前词法作用域或复合语句块。
- **L4234 EN**: Closes the current lexical scope or compound statement.
  **L4234 CN**: 结束当前词法作用域或复合语句块。
- **L4235 EN**: Transitions from the previous branch into an `else if` condition.
  **L4235 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L4236 EN**: Continues logic associated with callable symbol `has<ProcBindingDetails>`.
  **L4236 CN**: 继续与可调用符号 `has<ProcBindingDetails>` 相关的逻辑。
- **L4237 EN**: Starts a function, method, lambda, or structured scope: `ultimate->name() == ultimate2->name()) {`.
  **L4237 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ultimate->name() == ultimate2->name()) {`。
- **L4238 EN**: Skips to the next loop iteration.
  **L4238 CN**: 跳到下一次循环迭代。
- **L4239 EN**: Closes the current lexical scope or compound statement.
  **L4239 CN**: 结束当前词法作用域或复合语句块。
- **L4240 EN**: Continues logic associated with callable symbol `IsName`.
  **L4240 CN**: 继续与可调用符号 `IsName` 相关的逻辑。
- **L4241 EN**: Continues the surrounding expression or declaration: `? evaluate::characteristics::Distinguishable`.
  **L4241 CN**: 继续构造周围的表达式或声明：`? evaluate::characteristics::Distinguishable`。
- **L4242 EN**: Executes a standalone statement or declaration: `: evaluate::characteristics::DistinguishableOpOrAssign};`.
  **L4242 CN**: 执行一条独立语句或声明：`: evaluate::characteristics::DistinguishableOpOrAssign};`。
- **L4243 EN**: Continues logic associated with callable symbol `distinguishable`.
  **L4243 CN**: 继续与可调用符号 `distinguishable` 相关的逻辑。
- **L4244 EN**: Executes a call or declaration centered on `context_.languageFeatures`.
  **L4244 CN**: 执行以 `context_.languageFeatures` 为核心的调用或声明。
- **L4245 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4245 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SayNotDistinguishable(GetTopLevelUnitContaining(scope), name, kind,`.
  **L4246 CN**: 继续一个多行参数列表、初始化器或聚合项：`SayNotDistinguishable(GetTopLevelUnitContaining(scope), name, kind,`。
- **L4247 EN**: Comment explains nearby logic, intent, or metadata: `ultimate, *ultimate2, distinct.has_value());`.
  **L4247 CN**: 注释说明附近代码的逻辑、意图或元数据：`ultimate, *ultimate2, distinct.has_value());`。
- **L4248 EN**: Closes the current lexical scope or compound statement.
  **L4248 CN**: 结束当前词法作用域或复合语句块。

### Lines 4249-4272

````cpp
      }
    }
  }
}

void DistinguishabilityHelper::SayNotDistinguishable(const Scope &scope,
    const SourceName &name, GenericKind kind, const Symbol &proc1,
    const Symbol &proc2, bool isHardConflict) {
  bool isUseAssociated{!scope.sourceRange().Contains(name)};
  // The rules for distinguishing specific procedures (F'2023 15.4.3.4.5)
  // are inadequate for some real-world cases like pFUnit.
  // When there are optional dummy arguments or unlimited polymorphic
  // dummy data object arguments, the best that we can do is emit an optional
  // portability warning.  Also, named generics created by USE association
  // merging shouldn't receive hard errors for ambiguity.
  // (Non-named generics might be defined I/O procedures or defined
  // assignments that need to be used by the runtime.)
  bool isWarning{!isHardConflict || (isUseAssociated && kind.IsName())};
  if (isWarning &&
      (!context_.ShouldWarn(
           common::LanguageFeature::IndistinguishableSpecifics) ||
          FindModuleFileContaining(scope))) {
    return;
  }
````
- **L4249 EN**: Closes the current lexical scope or compound statement.
  **L4249 CN**: 结束当前词法作用域或复合语句块。
- **L4250 EN**: Closes the current lexical scope or compound statement.
  **L4250 CN**: 结束当前词法作用域或复合语句块。
- **L4251 EN**: Closes the current lexical scope or compound statement.
  **L4251 CN**: 结束当前词法作用域或复合语句块。
- **L4252 EN**: Closes the current lexical scope or compound statement.
  **L4252 CN**: 结束当前词法作用域或复合语句块。
- **L4253 EN**: Blank line separating nearby declarations or logic blocks.
  **L4253 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void DistinguishabilityHelper::SayNotDistinguishable(const Scope &scope,`.
  **L4254 CN**: 继续一个多行参数列表、初始化器或聚合项：`void DistinguishabilityHelper::SayNotDistinguishable(const Scope &scope,`。
- **L4255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const SourceName &name, GenericKind kind, const Symbol &proc1,`.
  **L4255 CN**: 继续一个多行参数列表、初始化器或聚合项：`const SourceName &name, GenericKind kind, const Symbol &proc1,`。
- **L4256 EN**: Continues the surrounding expression or declaration: `const Symbol &proc2, bool isHardConflict) {`.
  **L4256 CN**: 继续构造周围的表达式或声明：`const Symbol &proc2, bool isHardConflict) {`。
- **L4257 EN**: Executes a call or declaration centered on `isUseAssociated{!scope.sourceRange`.
  **L4257 CN**: 执行以 `isUseAssociated{!scope.sourceRange` 为核心的调用或声明。
- **L4258 EN**: Comment explains nearby logic, intent, or metadata: `The rules for distinguishing specific procedures (F'2023 15.4.3.4.5)`.
  **L4258 CN**: 注释说明附近代码的逻辑、意图或元数据：`The rules for distinguishing specific procedures (F'2023 15.4.3.4.5)`。
- **L4259 EN**: Comment explains nearby logic, intent, or metadata: `are inadequate for some real-world cases like pFUnit.`.
  **L4259 CN**: 注释说明附近代码的逻辑、意图或元数据：`are inadequate for some real-world cases like pFUnit.`。
- **L4260 EN**: Comment explains nearby logic, intent, or metadata: `When there are optional dummy arguments or unlimited polymorphic`.
  **L4260 CN**: 注释说明附近代码的逻辑、意图或元数据：`When there are optional dummy arguments or unlimited polymorphic`。
- **L4261 EN**: Comment explains nearby logic, intent, or metadata: `dummy data object arguments, the best that we can do is emit an optional`.
  **L4261 CN**: 注释说明附近代码的逻辑、意图或元数据：`dummy data object arguments, the best that we can do is emit an optional`。
- **L4262 EN**: Comment explains nearby logic, intent, or metadata: `portability warning.  Also, named generics created by USE association`.
  **L4262 CN**: 注释说明附近代码的逻辑、意图或元数据：`portability warning.  Also, named generics created by USE association`。
- **L4263 EN**: Comment explains nearby logic, intent, or metadata: `merging shouldn't receive hard errors for ambiguity.`.
  **L4263 CN**: 注释说明附近代码的逻辑、意图或元数据：`merging shouldn't receive hard errors for ambiguity.`。
- **L4264 EN**: Comment explains nearby logic, intent, or metadata: `(Non-named generics might be defined I/O procedures or defined`.
  **L4264 CN**: 注释说明附近代码的逻辑、意图或元数据：`(Non-named generics might be defined I/O procedures or defined`。
- **L4265 EN**: Comment explains nearby logic, intent, or metadata: `assignments that need to be used by the runtime.)`.
  **L4265 CN**: 注释说明附近代码的逻辑、意图或元数据：`assignments that need to be used by the runtime.)`。
- **L4266 EN**: Executes a call or declaration centered on `||`.
  **L4266 CN**: 执行以 `||` 为核心的调用或声明。
- **L4267 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4267 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4268 EN**: Continues logic associated with callable symbol `ShouldWarn`.
  **L4268 CN**: 继续与可调用符号 `ShouldWarn` 相关的逻辑。
- **L4269 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::IndistinguishableSpecifics) ||`.
  **L4269 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::IndistinguishableSpecifics) ||`。
- **L4270 EN**: Starts a function, method, lambda, or structured scope: `FindModuleFileContaining(scope))) {`.
  **L4270 CN**: 开始一个函数、方法、lambda 或结构化作用域：`FindModuleFileContaining(scope))) {`。
- **L4271 EN**: Returns from the current function with `void`.
  **L4271 CN**: 以 `void` 从当前函数返回。
- **L4272 EN**: Closes the current lexical scope or compound statement.
  **L4272 CN**: 结束当前词法作用域或复合语句块。

### Lines 4273-4296

````cpp
  std::string name1{proc1.name().ToString()};
  std::string name2{proc2.name().ToString()};
  if (kind.IsOperator() || kind.IsAssignment()) {
    // proc1 and proc2 may come from different scopes so qualify their names
    if (proc1.owner().IsDerivedType()) {
      name1 = proc1.owner().GetName()->ToString() + '%' + name1;
    }
    if (proc2.owner().IsDerivedType()) {
      name2 = proc2.owner().GetName()->ToString() + '%' + name2;
    }
  }
  parser::Message *msg;
  if (!isUseAssociated) {
    CHECK(isWarning == !isHardConflict);
    msg = &context_.Say(name,
        isHardConflict
            ? "Generic '%s' may not have specific procedures '%s' and '%s' as their interfaces are not distinguishable"_err_en_US
            : "Generic '%s' should not have specific procedures '%s' and '%s' as their interfaces are not distinguishable by the rules in the standard"_port_en_US,
        MakeOpName(name), name1, name2);
  } else {
    msg = &context_.Say(*GetTopLevelUnitContaining(proc1).GetName(),
        isHardConflict
            ? (isWarning
                      ? "USE-associated generic '%s' should not have specific procedures '%s' and '%s' as their interfaces are not distinguishable"_warn_en_US
````
- **L4273 EN**: Executes a call or declaration centered on `name1{proc1.name`.
  **L4273 CN**: 执行以 `name1{proc1.name` 为核心的调用或声明。
- **L4274 EN**: Executes a call or declaration centered on `name2{proc2.name`.
  **L4274 CN**: 执行以 `name2{proc2.name` 为核心的调用或声明。
- **L4275 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4275 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4276 EN**: Comment explains nearby logic, intent, or metadata: `proc1 and proc2 may come from different scopes so qualify their names`.
  **L4276 CN**: 注释说明附近代码的逻辑、意图或元数据：`proc1 and proc2 may come from different scopes so qualify their names`。
- **L4277 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4277 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4278 EN**: Executes a call or declaration centered on `proc1.owner`.
  **L4278 CN**: 执行以 `proc1.owner` 为核心的调用或声明。
- **L4279 EN**: Closes the current lexical scope or compound statement.
  **L4279 CN**: 结束当前词法作用域或复合语句块。
- **L4280 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4280 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4281 EN**: Executes a call or declaration centered on `proc2.owner`.
  **L4281 CN**: 执行以 `proc2.owner` 为核心的调用或声明。
- **L4282 EN**: Closes the current lexical scope or compound statement.
  **L4282 CN**: 结束当前词法作用域或复合语句块。
- **L4283 EN**: Closes the current lexical scope or compound statement.
  **L4283 CN**: 结束当前词法作用域或复合语句块。
- **L4284 EN**: Executes a standalone statement or declaration: `parser::Message *msg;`.
  **L4284 CN**: 执行一条独立语句或声明：`parser::Message *msg;`。
- **L4285 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4285 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4286 EN**: Executes a call or declaration centered on `CHECK`.
  **L4286 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L4287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg = &context_.Say(name,`.
  **L4287 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg = &context_.Say(name,`。
- **L4288 EN**: Continues the surrounding expression or declaration: `isHardConflict`.
  **L4288 CN**: 继续构造周围的表达式或声明：`isHardConflict`。
- **L4289 EN**: Continues the surrounding expression or declaration: `? "Generic '%s' may not have specific procedures '%s' and '%s' as their interfaces are not distinguishable"_err_en_US`.
  **L4289 CN**: 继续构造周围的表达式或声明：`? "Generic '%s' may not have specific procedures '%s' and '%s' as their interfaces are not distinguishable"_err_en_US`。
- **L4290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: "Generic '%s' should not have specific procedures '%s' and '%s' as their interfaces are not distinguishable by the rules in the standard"_port_en_US,`.
  **L4290 CN**: 继续一个多行参数列表、初始化器或聚合项：`: "Generic '%s' should not have specific procedures '%s' and '%s' as their interfaces are not distinguishable by the rules in the standard"_port_en_US,`。
- **L4291 EN**: Executes a call or declaration centered on `MakeOpName`.
  **L4291 CN**: 执行以 `MakeOpName` 为核心的调用或声明。
- **L4292 EN**: Transitions from the previous branch into the alternative path.
  **L4292 CN**: 从前一个分支过渡到备选路径。
- **L4293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg = &context_.Say(*GetTopLevelUnitContaining(proc1).GetName(),`.
  **L4293 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg = &context_.Say(*GetTopLevelUnitContaining(proc1).GetName(),`。
- **L4294 EN**: Continues the surrounding expression or declaration: `isHardConflict`.
  **L4294 CN**: 继续构造周围的表达式或声明：`isHardConflict`。
- **L4295 EN**: Continues the surrounding expression or declaration: `? (isWarning`.
  **L4295 CN**: 继续构造周围的表达式或声明：`? (isWarning`。
- **L4296 EN**: Continues the surrounding expression or declaration: `? "USE-associated generic '%s' should not have specific procedures '%s' and '%s' as their interfaces are not distinguishable"_warn_en_US`.
  **L4296 CN**: 继续构造周围的表达式或声明：`? "USE-associated generic '%s' should not have specific procedures '%s' and '%s' as their interfaces are not distinguishable"_warn_en_US`。

### Lines 4297-4320

````cpp
                      : "USE-associated generic '%s' may not have specific procedures '%s' and '%s' as their interfaces are not distinguishable"_err_en_US)
            : "USE-associated generic '%s' should not have specific procedures '%s' and '%s' as their interfaces are not distinguishable by the rules in the standard"_port_en_US,
        MakeOpName(name), name1, name2);
  }
  AttachDeclaration(*msg, scope, proc1);
  AttachDeclaration(*msg, scope, proc2);
}

// `evaluate::AttachDeclaration` doesn't handle the generic case where `proc`
// comes from a different module but is not necessarily use-associated.
void DistinguishabilityHelper::AttachDeclaration(
    parser::Message &msg, const Scope &scope, const Symbol &proc) {
  if (proc.owner().IsTopLevel()) {
    evaluate::AttachDeclaration(msg, proc);
  } else {
    const Scope &unit{GetTopLevelUnitContaining(proc)};
    if (unit == scope) {
      evaluate::AttachDeclaration(msg, proc);
    } else {
      msg.Attach(unit.GetName().value(),
          "'%s' is USE-associated from module '%s'"_en_US, proc.name(),
          unit.GetName().value());
    }
  }
````
- **L4297 EN**: Continues the surrounding expression or declaration: `: "USE-associated generic '%s' may not have specific procedures '%s' and '%s' as their interfaces are not distinguishable"_err_en_US)`.
  **L4297 CN**: 继续构造周围的表达式或声明：`: "USE-associated generic '%s' may not have specific procedures '%s' and '%s' as their interfaces are not distinguishable"_err_en_US)`。
- **L4298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: "USE-associated generic '%s' should not have specific procedures '%s' and '%s' as their interfaces are not distinguishable by the rules in the standard"_port_en_US,`.
  **L4298 CN**: 继续一个多行参数列表、初始化器或聚合项：`: "USE-associated generic '%s' should not have specific procedures '%s' and '%s' as their interfaces are not distinguishable by the rules in the standard"_port_en_US,`。
- **L4299 EN**: Executes a call or declaration centered on `MakeOpName`.
  **L4299 CN**: 执行以 `MakeOpName` 为核心的调用或声明。
- **L4300 EN**: Closes the current lexical scope or compound statement.
  **L4300 CN**: 结束当前词法作用域或复合语句块。
- **L4301 EN**: Executes a call or declaration centered on `AttachDeclaration`.
  **L4301 CN**: 执行以 `AttachDeclaration` 为核心的调用或声明。
- **L4302 EN**: Executes a call or declaration centered on `AttachDeclaration`.
  **L4302 CN**: 执行以 `AttachDeclaration` 为核心的调用或声明。
- **L4303 EN**: Closes the current lexical scope or compound statement.
  **L4303 CN**: 结束当前词法作用域或复合语句块。
- **L4304 EN**: Blank line separating nearby declarations or logic blocks.
  **L4304 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4305 EN**: Comment explains nearby logic, intent, or metadata: ``evaluate::AttachDeclaration` doesn't handle the generic case where `proc``.
  **L4305 CN**: 注释说明附近代码的逻辑、意图或元数据：``evaluate::AttachDeclaration` doesn't handle the generic case where `proc``。
- **L4306 EN**: Comment explains nearby logic, intent, or metadata: `comes from a different module but is not necessarily use-associated.`.
  **L4306 CN**: 注释说明附近代码的逻辑、意图或元数据：`comes from a different module but is not necessarily use-associated.`。
- **L4307 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L4307 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L4308 EN**: Continues the surrounding expression or declaration: `parser::Message &msg, const Scope &scope, const Symbol &proc) {`.
  **L4308 CN**: 继续构造周围的表达式或声明：`parser::Message &msg, const Scope &scope, const Symbol &proc) {`。
- **L4309 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4309 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4310 EN**: Executes a call or declaration centered on `evaluate::AttachDeclaration`.
  **L4310 CN**: 执行以 `evaluate::AttachDeclaration` 为核心的调用或声明。
- **L4311 EN**: Transitions from the previous branch into the alternative path.
  **L4311 CN**: 从前一个分支过渡到备选路径。
- **L4312 EN**: Executes a call or declaration centered on `&unit{GetTopLevelUnitContaining`.
  **L4312 CN**: 执行以 `&unit{GetTopLevelUnitContaining` 为核心的调用或声明。
- **L4313 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L4313 CN**: 开始 `if` 控制流语句并计算其条件。
- **L4314 EN**: Executes a call or declaration centered on `evaluate::AttachDeclaration`.
  **L4314 CN**: 执行以 `evaluate::AttachDeclaration` 为核心的调用或声明。
- **L4315 EN**: Transitions from the previous branch into the alternative path.
  **L4315 CN**: 从前一个分支过渡到备选路径。
- **L4316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg.Attach(unit.GetName().value(),`.
  **L4316 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg.Attach(unit.GetName().value(),`。
- **L4317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' is USE-associated from module '%s'"_en_US, proc.name(),`.
  **L4317 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' is USE-associated from module '%s'"_en_US, proc.name(),`。
- **L4318 EN**: Executes a call or declaration centered on `unit.GetName`.
  **L4318 CN**: 执行以 `unit.GetName` 为核心的调用或声明。
- **L4319 EN**: Closes the current lexical scope or compound statement.
  **L4319 CN**: 结束当前词法作用域或复合语句块。
- **L4320 EN**: Closes the current lexical scope or compound statement.
  **L4320 CN**: 结束当前词法作用域或复合语句块。

### Lines 4321-4326

````cpp
}

void CheckDeclarations(SemanticsContext &context) {
  CheckHelper{context}.Check();
}
} // namespace Fortran::semantics
````
- **L4321 EN**: Closes the current lexical scope or compound statement.
  **L4321 CN**: 结束当前词法作用域或复合语句块。
- **L4322 EN**: Blank line separating nearby declarations or logic blocks.
  **L4322 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4323 EN**: Starts a function, method, lambda, or structured scope: `void CheckDeclarations(SemanticsContext &context) {`.
  **L4323 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckDeclarations(SemanticsContext &context) {`。
- **L4324 EN**: Executes a call or declaration centered on `CheckHelper{context}.Check`.
  **L4324 CN**: 执行以 `CheckHelper{context}.Check` 为核心的调用或声明。
- **L4325 EN**: Closes the current lexical scope or compound statement.
  **L4325 CN**: 结束当前词法作用域或复合语句块。
- **L4326 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::semantics`.
  **L4326 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::semantics`。

## Key Concepts / 关键概念

- **Fortran semantic checking / Fortran 语义检查**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Semantic-context management / 语义上下文管理**
- **Scope and symbol resolution / 作用域与符号解析**
- **Symbol modeling and lookup / 符号建模与查找**
- **Fortran type-spec reasoning / Fortran 类型说明推理**
- **Compile-time evaluation helpers / 编译期求值辅助**
- **Constant folding / 常量折叠**
- **Evaluation context management / 求值上下文管理**
- **OpenACC handling / OpenACC 处理**

## Dependencies / 依赖关系

- `check-declarations.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `definable.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `pointer-assignment.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Evaluate/check-expression.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/fold.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/tools.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Parser/characters.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Semantics/scope.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/semantics.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/symbol.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/type.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `algorithm`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
- `map`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
- `string`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
