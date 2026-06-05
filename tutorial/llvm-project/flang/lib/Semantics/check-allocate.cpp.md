# check-allocate.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Semantics/check-allocate.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements semantic checking, symbol reasoning, and language-rule enforcement for check allocate.
- **Purpose (CN)**: 实现 check allocate 相关的语义检查、符号推理与语言规则约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- lib/Semantics/check-allocate.cpp ----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "check-allocate.h"
#include "assignment.h"
#include "definable.h"
#include "flang/Evaluate/fold.h"
#include "flang/Evaluate/shape.h"
#include "flang/Evaluate/type.h"
#include "flang/Parser/parse-tree.h"
#include "flang/Parser/tools.h"
#include "flang/Semantics/attr.h"
#include "flang/Semantics/expression.h"
#include "flang/Semantics/tools.h"
#include "flang/Semantics/type.h"

namespace Fortran::semantics {

struct AllocateCheckerInfo {
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
- **L9 EN**: Includes "check-allocate.h" to access local declarations paired with this implementation.
  **L9 CN**: 引入 "check-allocate.h" 以使用与该实现配套的本地声明。
- **L10 EN**: Includes "assignment.h" to access local declarations paired with this implementation.
  **L10 CN**: 引入 "assignment.h" 以使用与该实现配套的本地声明。
- **L11 EN**: Includes "definable.h" to access local declarations paired with this implementation.
  **L11 CN**: 引入 "definable.h" 以使用与该实现配套的本地声明。
- **L12 EN**: Includes "flang/Evaluate/fold.h" to access Fortran constant-folding and evaluation facilities.
  **L12 CN**: 引入 "flang/Evaluate/fold.h" 以使用Fortran 常量折叠与求值能力。
- **L13 EN**: Includes "flang/Evaluate/shape.h" to access Fortran constant-folding and evaluation facilities.
  **L13 CN**: 引入 "flang/Evaluate/shape.h" 以使用Fortran 常量折叠与求值能力。
- **L14 EN**: Includes "flang/Evaluate/type.h" to access Fortran constant-folding and evaluation facilities.
  **L14 CN**: 引入 "flang/Evaluate/type.h" 以使用Fortran 常量折叠与求值能力。
- **L15 EN**: Includes "flang/Parser/parse-tree.h" to access parse-tree, token, or source representation support.
  **L15 CN**: 引入 "flang/Parser/parse-tree.h" 以使用语法树、词法单元或源码表示支持。
- **L16 EN**: Includes "flang/Parser/tools.h" to access parse-tree, token, or source representation support.
  **L16 CN**: 引入 "flang/Parser/tools.h" 以使用语法树、词法单元或源码表示支持。
- **L17 EN**: Includes "flang/Semantics/attr.h" to access Fortran semantic analysis, symbol, and type information.
  **L17 CN**: 引入 "flang/Semantics/attr.h" 以使用Fortran 语义分析、符号与类型信息。
- **L18 EN**: Includes "flang/Semantics/expression.h" to access Fortran semantic analysis, symbol, and type information.
  **L18 CN**: 引入 "flang/Semantics/expression.h" 以使用Fortran 语义分析、符号与类型信息。
- **L19 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L19 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L20 EN**: Includes "flang/Semantics/type.h" to access Fortran semantic analysis, symbol, and type information.
  **L20 CN**: 引入 "flang/Semantics/type.h" 以使用Fortran 语义分析、符号与类型信息。
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L22 EN**: Opens namespace scope `Fortran::semantics`.
  **L22 CN**: 打开命名空间作用域 `Fortran::semantics`。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Declares struct `AllocateCheckerInfo`.
  **L24 CN**: 声明 struct `AllocateCheckerInfo`。

### Lines 25-48

````cpp
  const DeclTypeSpec *typeSpec{nullptr};
  std::optional<evaluate::DynamicType> sourceExprType;
  std::optional<parser::CharBlock> sourceExprLoc;
  std::optional<parser::CharBlock> typeSpecLoc;
  std::optional<parser::CharBlock> statSource;
  std::optional<parser::CharBlock> msgSource;
  const SomeExpr *statVar{nullptr};
  const SomeExpr *msgVar{nullptr};
  int sourceExprRank{0}; // only valid if gotMold || gotSource
  bool gotStat{false};
  bool gotMsg{false};
  bool gotTypeSpec{false};
  bool gotSource{false};
  bool gotMold{false};
  bool gotStream{false};
  bool gotPinned{false};
  std::optional<evaluate::ConstantSubscripts> sourceExprShape;
};

class AllocationCheckerHelper {
public:
  AllocationCheckerHelper(
      const parser::Allocation &alloc, AllocateCheckerInfo &info)
      : allocateInfo_{info}, allocation_{alloc},
````
- **L25 EN**: Executes a standalone statement or declaration: `const DeclTypeSpec *typeSpec{nullptr};`.
  **L25 CN**: 执行一条独立语句或声明：`const DeclTypeSpec *typeSpec{nullptr};`。
- **L26 EN**: Executes a standalone statement or declaration: `std::optional<evaluate::DynamicType> sourceExprType;`.
  **L26 CN**: 执行一条独立语句或声明：`std::optional<evaluate::DynamicType> sourceExprType;`。
- **L27 EN**: Executes a standalone statement or declaration: `std::optional<parser::CharBlock> sourceExprLoc;`.
  **L27 CN**: 执行一条独立语句或声明：`std::optional<parser::CharBlock> sourceExprLoc;`。
- **L28 EN**: Executes a standalone statement or declaration: `std::optional<parser::CharBlock> typeSpecLoc;`.
  **L28 CN**: 执行一条独立语句或声明：`std::optional<parser::CharBlock> typeSpecLoc;`。
- **L29 EN**: Executes a standalone statement or declaration: `std::optional<parser::CharBlock> statSource;`.
  **L29 CN**: 执行一条独立语句或声明：`std::optional<parser::CharBlock> statSource;`。
- **L30 EN**: Executes a standalone statement or declaration: `std::optional<parser::CharBlock> msgSource;`.
  **L30 CN**: 执行一条独立语句或声明：`std::optional<parser::CharBlock> msgSource;`。
- **L31 EN**: Executes a standalone statement or declaration: `const SomeExpr *statVar{nullptr};`.
  **L31 CN**: 执行一条独立语句或声明：`const SomeExpr *statVar{nullptr};`。
- **L32 EN**: Executes a standalone statement or declaration: `const SomeExpr *msgVar{nullptr};`.
  **L32 CN**: 执行一条独立语句或声明：`const SomeExpr *msgVar{nullptr};`。
- **L33 EN**: Continues the surrounding expression or declaration: `int sourceExprRank{0}; // only valid if gotMold || gotSource`.
  **L33 CN**: 继续构造周围的表达式或声明：`int sourceExprRank{0}; // only valid if gotMold || gotSource`。
- **L34 EN**: Executes a standalone statement or declaration: `bool gotStat{false};`.
  **L34 CN**: 执行一条独立语句或声明：`bool gotStat{false};`。
- **L35 EN**: Executes a standalone statement or declaration: `bool gotMsg{false};`.
  **L35 CN**: 执行一条独立语句或声明：`bool gotMsg{false};`。
- **L36 EN**: Executes a standalone statement or declaration: `bool gotTypeSpec{false};`.
  **L36 CN**: 执行一条独立语句或声明：`bool gotTypeSpec{false};`。
- **L37 EN**: Executes a standalone statement or declaration: `bool gotSource{false};`.
  **L37 CN**: 执行一条独立语句或声明：`bool gotSource{false};`。
- **L38 EN**: Executes a standalone statement or declaration: `bool gotMold{false};`.
  **L38 CN**: 执行一条独立语句或声明：`bool gotMold{false};`。
- **L39 EN**: Executes a standalone statement or declaration: `bool gotStream{false};`.
  **L39 CN**: 执行一条独立语句或声明：`bool gotStream{false};`。
- **L40 EN**: Executes a standalone statement or declaration: `bool gotPinned{false};`.
  **L40 CN**: 执行一条独立语句或声明：`bool gotPinned{false};`。
- **L41 EN**: Executes a standalone statement or declaration: `std::optional<evaluate::ConstantSubscripts> sourceExprShape;`.
  **L41 CN**: 执行一条独立语句或声明：`std::optional<evaluate::ConstantSubscripts> sourceExprShape;`。
- **L42 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L42 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L44 EN**: Declares class `AllocationCheckerHelper`.
  **L44 CN**: 声明 class `AllocationCheckerHelper`。
- **L45 EN**: Sets the following members to `public` access.
  **L45 CN**: 将后续成员的访问级别设为 `public`。
- **L46 EN**: Continues logic associated with callable symbol `AllocationCheckerHelper`.
  **L46 CN**: 继续与可调用符号 `AllocationCheckerHelper` 相关的逻辑。
- **L47 EN**: Continues the surrounding expression or declaration: `const parser::Allocation &alloc, AllocateCheckerInfo &info)`.
  **L47 CN**: 继续构造周围的表达式或声明：`const parser::Allocation &alloc, AllocateCheckerInfo &info)`。
- **L48 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: allocateInfo_{info}, allocation_{alloc},`.
  **L48 CN**: 继续一个多行参数列表、初始化器或聚合项：`: allocateInfo_{info}, allocation_{alloc},`。

### Lines 49-72

````cpp
        allocateObject_{std::get<parser::AllocateObject>(alloc.t)},
        allocateShapeSpecRank_{ShapeSpecRank(alloc)},
        allocateCoarraySpecRank_{CoarraySpecRank(alloc)} {}

  bool RunChecks(SemanticsContext &context);

private:
  bool hasAllocateShapeSpecList() const { return allocateShapeSpecRank_ != 0; }
  bool hasAllocateCoarraySpec() const { return allocateCoarraySpecRank_ != 0; }
  bool RunCoarrayRelatedChecks(SemanticsContext &) const;

  static int ShapeSpecRank(const parser::Allocation &allocation) {
    return static_cast<int>(
        std::get<std::list<parser::AllocateShapeSpec>>(allocation.t).size());
  }

  static int CoarraySpecRank(const parser::Allocation &allocation) {
    if (const auto &coarraySpec{
            std::get<std::optional<parser::AllocateCoarraySpec>>(
                allocation.t)}) {
      return std::get<std::list<parser::AllocateCoshapeSpec>>(coarraySpec->t)
                 .size() +
          1;
    } else {
````
- **L49 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `allocateObject_{std::get<parser::AllocateObject>(alloc.t)},`.
  **L49 CN**: 继续一个多行参数列表、初始化器或聚合项：`allocateObject_{std::get<parser::AllocateObject>(alloc.t)},`。
- **L50 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `allocateShapeSpecRank_{ShapeSpecRank(alloc)},`.
  **L50 CN**: 继续一个多行参数列表、初始化器或聚合项：`allocateShapeSpecRank_{ShapeSpecRank(alloc)},`。
- **L51 EN**: Continues logic associated with callable symbol `CoarraySpecRank`.
  **L51 CN**: 继续与可调用符号 `CoarraySpecRank` 相关的逻辑。
- **L52 EN**: Blank line separating nearby declarations or logic blocks.
  **L52 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L53 EN**: Executes a call or declaration centered on `RunChecks`.
  **L53 CN**: 执行以 `RunChecks` 为核心的调用或声明。
- **L54 EN**: Blank line separating nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L55 EN**: Sets the following members to `private` access.
  **L55 CN**: 将后续成员的访问级别设为 `private`。
- **L56 EN**: Continues logic associated with callable symbol `hasAllocateShapeSpecList`.
  **L56 CN**: 继续与可调用符号 `hasAllocateShapeSpecList` 相关的逻辑。
- **L57 EN**: Continues logic associated with callable symbol `hasAllocateCoarraySpec`.
  **L57 CN**: 继续与可调用符号 `hasAllocateCoarraySpec` 相关的逻辑。
- **L58 EN**: Executes a call or declaration centered on `RunCoarrayRelatedChecks`.
  **L58 CN**: 执行以 `RunCoarrayRelatedChecks` 为核心的调用或声明。
- **L59 EN**: Blank line separating nearby declarations or logic blocks.
  **L59 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L60 EN**: Starts a function, method, lambda, or structured scope: `static int ShapeSpecRank(const parser::Allocation &allocation) {`.
  **L60 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static int ShapeSpecRank(const parser::Allocation &allocation) {`。
- **L61 EN**: Returns from the current function with `static_cast<int>(`.
  **L61 CN**: 以 `static_cast<int>(` 从当前函数返回。
- **L62 EN**: Executes a call or declaration centered on `std::get<std::list<parser::AllocateShapeSpec>>`.
  **L62 CN**: 执行以 `std::get<std::list<parser::AllocateShapeSpec>>` 为核心的调用或声明。
- **L63 EN**: Closes the current lexical scope or compound statement.
  **L63 CN**: 结束当前词法作用域或复合语句块。
- **L64 EN**: Blank line separating nearby declarations or logic blocks.
  **L64 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L65 EN**: Starts a function, method, lambda, or structured scope: `static int CoarraySpecRank(const parser::Allocation &allocation) {`.
  **L65 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static int CoarraySpecRank(const parser::Allocation &allocation) {`。
- **L66 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L66 CN**: 开始 `if` 控制流语句并计算其条件。
- **L67 EN**: Continues logic associated with callable symbol `AllocateCoarraySpec>>`.
  **L67 CN**: 继续与可调用符号 `AllocateCoarraySpec>>` 相关的逻辑。
- **L68 EN**: Continues the surrounding expression or declaration: `allocation.t)}) {`.
  **L68 CN**: 继续构造周围的表达式或声明：`allocation.t)}) {`。
- **L69 EN**: Returns from the current function with `std::get<std::list<parser::AllocateCoshapeSpec>>(coarraySpec->t)`.
  **L69 CN**: 以 `std::get<std::list<parser::AllocateCoshapeSpec>>(coarraySpec->t)` 从当前函数返回。
- **L70 EN**: Continues logic associated with callable symbol `size`.
  **L70 CN**: 继续与可调用符号 `size` 相关的逻辑。
- **L71 EN**: Executes a standalone statement or declaration: `1;`.
  **L71 CN**: 执行一条独立语句或声明：`1;`。
- **L72 EN**: Transitions from the previous branch into the alternative path.
  **L72 CN**: 从前一个分支过渡到备选路径。

### Lines 73-96

````cpp
      return 0;
    }
  }

  void GatherAllocationBasicInfo() {
    if (type_->category() == DeclTypeSpec::Category::Character) {
      hasDeferredTypeParameter_ =
          type_->characterTypeSpec().length().isDeferred();
    } else if (const DerivedTypeSpec * derivedTypeSpec{type_->AsDerived()}) {
      for (const auto &pair : derivedTypeSpec->parameters()) {
        hasDeferredTypeParameter_ |= pair.second.isDeferred();
      }
      isAbstract_ = derivedTypeSpec->typeSymbol().attrs().test(Attr::ABSTRACT);
    }
    isUnlimitedPolymorphic_ =
        type_->category() == DeclTypeSpec::Category::ClassStar;
  }

  AllocateCheckerInfo &allocateInfo_;
  const parser::Allocation &allocation_;
  const parser::AllocateObject &allocateObject_;
  const int allocateShapeSpecRank_{0};
  const int allocateCoarraySpecRank_{0};
  const parser::Name &name_{parser::GetLastName(allocateObject_)};
````
- **L73 EN**: Returns from the current function with `0`.
  **L73 CN**: 以 `0` 从当前函数返回。
- **L74 EN**: Closes the current lexical scope or compound statement.
  **L74 CN**: 结束当前词法作用域或复合语句块。
- **L75 EN**: Closes the current lexical scope or compound statement.
  **L75 CN**: 结束当前词法作用域或复合语句块。
- **L76 EN**: Blank line separating nearby declarations or logic blocks.
  **L76 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L77 EN**: Starts a function, method, lambda, or structured scope: `void GatherAllocationBasicInfo() {`.
  **L77 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void GatherAllocationBasicInfo() {`。
- **L78 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L78 CN**: 开始 `if` 控制流语句并计算其条件。
- **L79 EN**: Continues the surrounding expression or declaration: `hasDeferredTypeParameter_ =`.
  **L79 CN**: 继续构造周围的表达式或声明：`hasDeferredTypeParameter_ =`。
- **L80 EN**: Executes a call or declaration centered on `type_->characterTypeSpec`.
  **L80 CN**: 执行以 `type_->characterTypeSpec` 为核心的调用或声明。
- **L81 EN**: Transitions from the previous branch into an `else if` condition.
  **L81 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L82 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L82 CN**: 开始 `for` 控制流语句并计算其条件。
- **L83 EN**: Executes a call or declaration centered on `pair.second.isDeferred`.
  **L83 CN**: 执行以 `pair.second.isDeferred` 为核心的调用或声明。
- **L84 EN**: Closes the current lexical scope or compound statement.
  **L84 CN**: 结束当前词法作用域或复合语句块。
- **L85 EN**: Executes a call or declaration centered on `derivedTypeSpec->typeSymbol`.
  **L85 CN**: 执行以 `derivedTypeSpec->typeSymbol` 为核心的调用或声明。
- **L86 EN**: Closes the current lexical scope or compound statement.
  **L86 CN**: 结束当前词法作用域或复合语句块。
- **L87 EN**: Continues the surrounding expression or declaration: `isUnlimitedPolymorphic_ =`.
  **L87 CN**: 继续构造周围的表达式或声明：`isUnlimitedPolymorphic_ =`。
- **L88 EN**: Executes a call or declaration centered on `type_->category`.
  **L88 CN**: 执行以 `type_->category` 为核心的调用或声明。
- **L89 EN**: Closes the current lexical scope or compound statement.
  **L89 CN**: 结束当前词法作用域或复合语句块。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L91 EN**: Executes a standalone statement or declaration: `AllocateCheckerInfo &allocateInfo_;`.
  **L91 CN**: 执行一条独立语句或声明：`AllocateCheckerInfo &allocateInfo_;`。
- **L92 EN**: Executes a standalone statement or declaration: `const parser::Allocation &allocation_;`.
  **L92 CN**: 执行一条独立语句或声明：`const parser::Allocation &allocation_;`。
- **L93 EN**: Executes a standalone statement or declaration: `const parser::AllocateObject &allocateObject_;`.
  **L93 CN**: 执行一条独立语句或声明：`const parser::AllocateObject &allocateObject_;`。
- **L94 EN**: Executes a standalone statement or declaration: `const int allocateShapeSpecRank_{0};`.
  **L94 CN**: 执行一条独立语句或声明：`const int allocateShapeSpecRank_{0};`。
- **L95 EN**: Executes a standalone statement or declaration: `const int allocateCoarraySpecRank_{0};`.
  **L95 CN**: 执行一条独立语句或声明：`const int allocateCoarraySpecRank_{0};`。
- **L96 EN**: Executes a call or declaration centered on `&name_{parser::GetLastName`.
  **L96 CN**: 执行以 `&name_{parser::GetLastName` 为核心的调用或声明。

### Lines 97-120

````cpp
  // no USE or host association
  const Symbol *ultimate_{
      name_.symbol ? &name_.symbol->GetUltimate() : nullptr};
  const DeclTypeSpec *type_{ultimate_ ? ultimate_->GetType() : nullptr};
  const int rank_{ultimate_ ? ultimate_->Rank() : 0};
  const int corank_{ultimate_ ? ultimate_->Corank() : 0};
  bool hasDeferredTypeParameter_{false};
  bool isUnlimitedPolymorphic_{false};
  bool isAbstract_{false};
};

static std::optional<AllocateCheckerInfo> CheckAllocateOptions(
    const parser::AllocateStmt &allocateStmt, SemanticsContext &context) {
  AllocateCheckerInfo info;
  bool stopCheckingAllocate{false}; // for errors that would lead to ambiguity
  if (const auto &typeSpec{
          std::get<std::optional<parser::TypeSpec>>(allocateStmt.t)}) {
    info.typeSpec = typeSpec->declTypeSpec;
    if (!info.typeSpec) {
      CHECK(context.AnyFatalError());
      return std::nullopt;
    }
    info.gotTypeSpec = true;
    info.typeSpecLoc = parser::FindSourceLocation(*typeSpec);
````
- **L97 EN**: Comment explains nearby logic, intent, or metadata: `no USE or host association`.
  **L97 CN**: 注释说明附近代码的逻辑、意图或元数据：`no USE or host association`。
- **L98 EN**: Continues the surrounding expression or declaration: `const Symbol *ultimate_{`.
  **L98 CN**: 继续构造周围的表达式或声明：`const Symbol *ultimate_{`。
- **L99 EN**: Executes a call or declaration centered on `&name_.symbol->GetUltimate`.
  **L99 CN**: 执行以 `&name_.symbol->GetUltimate` 为核心的调用或声明。
- **L100 EN**: Executes a call or declaration centered on `ultimate_->GetType`.
  **L100 CN**: 执行以 `ultimate_->GetType` 为核心的调用或声明。
- **L101 EN**: Executes a call or declaration centered on `ultimate_->Rank`.
  **L101 CN**: 执行以 `ultimate_->Rank` 为核心的调用或声明。
- **L102 EN**: Executes a call or declaration centered on `ultimate_->Corank`.
  **L102 CN**: 执行以 `ultimate_->Corank` 为核心的调用或声明。
- **L103 EN**: Executes a standalone statement or declaration: `bool hasDeferredTypeParameter_{false};`.
  **L103 CN**: 执行一条独立语句或声明：`bool hasDeferredTypeParameter_{false};`。
- **L104 EN**: Executes a standalone statement or declaration: `bool isUnlimitedPolymorphic_{false};`.
  **L104 CN**: 执行一条独立语句或声明：`bool isUnlimitedPolymorphic_{false};`。
- **L105 EN**: Executes a standalone statement or declaration: `bool isAbstract_{false};`.
  **L105 CN**: 执行一条独立语句或声明：`bool isAbstract_{false};`。
- **L106 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L106 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L107 EN**: Blank line separating nearby declarations or logic blocks.
  **L107 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L108 EN**: Continues logic associated with callable symbol `CheckAllocateOptions`.
  **L108 CN**: 继续与可调用符号 `CheckAllocateOptions` 相关的逻辑。
- **L109 EN**: Continues the surrounding expression or declaration: `const parser::AllocateStmt &allocateStmt, SemanticsContext &context) {`.
  **L109 CN**: 继续构造周围的表达式或声明：`const parser::AllocateStmt &allocateStmt, SemanticsContext &context) {`。
- **L110 EN**: Executes a standalone statement or declaration: `AllocateCheckerInfo info;`.
  **L110 CN**: 执行一条独立语句或声明：`AllocateCheckerInfo info;`。
- **L111 EN**: Continues the surrounding expression or declaration: `bool stopCheckingAllocate{false}; // for errors that would lead to ambiguity`.
  **L111 CN**: 继续构造周围的表达式或声明：`bool stopCheckingAllocate{false}; // for errors that would lead to ambiguity`。
- **L112 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L112 CN**: 开始 `if` 控制流语句并计算其条件。
- **L113 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::optional<parser::TypeSpec>>(allocateStmt.t)}) {`.
  **L113 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::optional<parser::TypeSpec>>(allocateStmt.t)}) {`。
- **L114 EN**: Executes a standalone statement or declaration: `info.typeSpec = typeSpec->declTypeSpec;`.
  **L114 CN**: 执行一条独立语句或声明：`info.typeSpec = typeSpec->declTypeSpec;`。
- **L115 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L115 CN**: 开始 `if` 控制流语句并计算其条件。
- **L116 EN**: Executes a call or declaration centered on `CHECK`.
  **L116 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L117 EN**: Returns from the current function with `std::nullopt`.
  **L117 CN**: 以 `std::nullopt` 从当前函数返回。
- **L118 EN**: Closes the current lexical scope or compound statement.
  **L118 CN**: 结束当前词法作用域或复合语句块。
- **L119 EN**: Executes a standalone statement or declaration: `info.gotTypeSpec = true;`.
  **L119 CN**: 执行一条独立语句或声明：`info.gotTypeSpec = true;`。
- **L120 EN**: Executes a call or declaration centered on `parser::FindSourceLocation`.
  **L120 CN**: 执行以 `parser::FindSourceLocation` 为核心的调用或声明。

### Lines 121-144

````cpp
    if (const DerivedTypeSpec * derived{info.typeSpec->AsDerived()}) {
      // C937
      if (auto it{FindCoarrayUltimateComponent(*derived)}) {
        context
            .Say(
                "Type-spec in ALLOCATE must not specify a type with a coarray ultimate component"_err_en_US)
            .Attach(it->name(),
                "Type '%s' has coarray ultimate component '%s' declared here"_en_US,
                info.typeSpec->AsFortran(), it.BuildResultDesignatorName());
      }
    }
    if (auto dyType{evaluate::DynamicType::From(*info.typeSpec)}) {
      if (dyType->HasDeferredTypeParameter()) {
        context.Say(
            "Type-spec in ALLOCATE must not have a deferred type parameter"_err_en_US);
      }
    }
  }

  const parser::Expr *parserSourceExpr{nullptr};
  for (const parser::AllocOpt &allocOpt :
      std::get<std::list<parser::AllocOpt>>(allocateStmt.t)) {
    common::visit(
        common::visitors{
````
- **L121 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L121 CN**: 开始 `if` 控制流语句并计算其条件。
- **L122 EN**: Comment explains nearby logic, intent, or metadata: `C937`.
  **L122 CN**: 注释说明附近代码的逻辑、意图或元数据：`C937`。
- **L123 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L123 CN**: 开始 `if` 控制流语句并计算其条件。
- **L124 EN**: Continues the surrounding expression or declaration: `context`.
  **L124 CN**: 继续构造周围的表达式或声明：`context`。
- **L125 EN**: Continues logic associated with callable symbol `Say`.
  **L125 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L126 EN**: Continues the surrounding expression or declaration: `"Type-spec in ALLOCATE must not specify a type with a coarray ultimate component"_err_en_US)`.
  **L126 CN**: 继续构造周围的表达式或声明：`"Type-spec in ALLOCATE must not specify a type with a coarray ultimate component"_err_en_US)`。
- **L127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(it->name(),`.
  **L127 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(it->name(),`。
- **L128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Type '%s' has coarray ultimate component '%s' declared here"_en_US,`.
  **L128 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Type '%s' has coarray ultimate component '%s' declared here"_en_US,`。
- **L129 EN**: Executes a call or declaration centered on `info.typeSpec->AsFortran`.
  **L129 CN**: 执行以 `info.typeSpec->AsFortran` 为核心的调用或声明。
- **L130 EN**: Closes the current lexical scope or compound statement.
  **L130 CN**: 结束当前词法作用域或复合语句块。
- **L131 EN**: Closes the current lexical scope or compound statement.
  **L131 CN**: 结束当前词法作用域或复合语句块。
- **L132 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L132 CN**: 开始 `if` 控制流语句并计算其条件。
- **L133 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L133 CN**: 开始 `if` 控制流语句并计算其条件。
- **L134 EN**: Continues logic associated with callable symbol `Say`.
  **L134 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L135 EN**: Executes a standalone statement or declaration: `"Type-spec in ALLOCATE must not have a deferred type parameter"_err_en_US);`.
  **L135 CN**: 执行一条独立语句或声明：`"Type-spec in ALLOCATE must not have a deferred type parameter"_err_en_US);`。
- **L136 EN**: Closes the current lexical scope or compound statement.
  **L136 CN**: 结束当前词法作用域或复合语句块。
- **L137 EN**: Closes the current lexical scope or compound statement.
  **L137 CN**: 结束当前词法作用域或复合语句块。
- **L138 EN**: Closes the current lexical scope or compound statement.
  **L138 CN**: 结束当前词法作用域或复合语句块。
- **L139 EN**: Blank line separating nearby declarations or logic blocks.
  **L139 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L140 EN**: Executes a standalone statement or declaration: `const parser::Expr *parserSourceExpr{nullptr};`.
  **L140 CN**: 执行一条独立语句或声明：`const parser::Expr *parserSourceExpr{nullptr};`。
- **L141 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L141 CN**: 开始 `for` 控制流语句并计算其条件。
- **L142 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::list<parser::AllocOpt>>(allocateStmt.t)) {`.
  **L142 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::list<parser::AllocOpt>>(allocateStmt.t)) {`。
- **L143 EN**: Continues logic associated with callable symbol `visit`.
  **L143 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L144 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L144 CN**: 继续构造周围的表达式或声明：`common::visitors{`。

### Lines 145-168

````cpp
            [&](const parser::StatOrErrmsg &statOrErr) {
              common::visit(
                  common::visitors{
                      [&](const parser::StatVariable &var) {
                        if (info.gotStat) { // C943
                          context.Say(
                              "STAT may not be duplicated in a ALLOCATE statement"_err_en_US);
                        }
                        info.gotStat = true;
                        info.statVar = GetExpr(context, var);
                        info.statSource =
                            parser::Unwrap<parser::Variable>(var)->GetSource();
                      },
                      [&](const parser::MsgVariable &var) {
                        WarnOnDeferredLengthCharacterScalar(context,
                            GetExpr(context, var),
                            parser::UnwrapRef<parser::Variable>(var)
                                .GetSource(),
                            "ERRMSG=");
                        if (info.gotMsg) { // C943
                          context.Say(
                              "ERRMSG may not be duplicated in a ALLOCATE statement"_err_en_US);
                        }
                        info.gotMsg = true;
````
- **L145 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::StatOrErrmsg &statOrErr) {`.
  **L145 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::StatOrErrmsg &statOrErr) {`。
- **L146 EN**: Continues logic associated with callable symbol `visit`.
  **L146 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L147 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L147 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L148 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::StatVariable &var) {`.
  **L148 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::StatVariable &var) {`。
- **L149 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L149 CN**: 开始 `if` 控制流语句并计算其条件。
- **L150 EN**: Continues logic associated with callable symbol `Say`.
  **L150 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L151 EN**: Executes a standalone statement or declaration: `"STAT may not be duplicated in a ALLOCATE statement"_err_en_US);`.
  **L151 CN**: 执行一条独立语句或声明：`"STAT may not be duplicated in a ALLOCATE statement"_err_en_US);`。
- **L152 EN**: Closes the current lexical scope or compound statement.
  **L152 CN**: 结束当前词法作用域或复合语句块。
- **L153 EN**: Executes a standalone statement or declaration: `info.gotStat = true;`.
  **L153 CN**: 执行一条独立语句或声明：`info.gotStat = true;`。
- **L154 EN**: Executes a call or declaration centered on `GetExpr`.
  **L154 CN**: 执行以 `GetExpr` 为核心的调用或声明。
- **L155 EN**: Continues the surrounding expression or declaration: `info.statSource =`.
  **L155 CN**: 继续构造周围的表达式或声明：`info.statSource =`。
- **L156 EN**: Executes a call or declaration centered on `parser::Unwrap<parser::Variable>`.
  **L156 CN**: 执行以 `parser::Unwrap<parser::Variable>` 为核心的调用或声明。
- **L157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L157 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L158 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::MsgVariable &var) {`.
  **L158 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::MsgVariable &var) {`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WarnOnDeferredLengthCharacterScalar(context,`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`WarnOnDeferredLengthCharacterScalar(context,`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `GetExpr(context, var),`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`GetExpr(context, var),`。
- **L161 EN**: Continues logic associated with callable symbol `Variable>`.
  **L161 CN**: 继续与可调用符号 `Variable>` 相关的逻辑。
- **L162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.GetSource(),`.
  **L162 CN**: 继续一个多行参数列表、初始化器或聚合项：`.GetSource(),`。
- **L163 EN**: Executes a standalone statement or declaration: `"ERRMSG=");`.
  **L163 CN**: 执行一条独立语句或声明：`"ERRMSG=");`。
- **L164 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L164 CN**: 开始 `if` 控制流语句并计算其条件。
- **L165 EN**: Continues logic associated with callable symbol `Say`.
  **L165 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L166 EN**: Executes a standalone statement or declaration: `"ERRMSG may not be duplicated in a ALLOCATE statement"_err_en_US);`.
  **L166 CN**: 执行一条独立语句或声明：`"ERRMSG may not be duplicated in a ALLOCATE statement"_err_en_US);`。
- **L167 EN**: Closes the current lexical scope or compound statement.
  **L167 CN**: 结束当前词法作用域或复合语句块。
- **L168 EN**: Executes a standalone statement or declaration: `info.gotMsg = true;`.
  **L168 CN**: 执行一条独立语句或声明：`info.gotMsg = true;`。

### Lines 169-192

````cpp
                        info.msgVar = GetExpr(context, var);
                        info.msgSource =
                            parser::Unwrap<parser::Variable>(var)->GetSource();
                      },
                  },
                  statOrErr.u);
            },
            [&](const parser::AllocOpt::Source &source) {
              if (info.gotSource) { // C943
                context.Say(
                    "SOURCE may not be duplicated in a ALLOCATE statement"_err_en_US);
                stopCheckingAllocate = true;
              }
              if (info.gotMold || info.gotTypeSpec) { // C944
                context.Say(
                    "At most one of source-expr and type-spec may appear in a ALLOCATE statement"_err_en_US);
                stopCheckingAllocate = true;
              }
              parserSourceExpr = &source.v.value();
              info.gotSource = true;
            },
            [&](const parser::AllocOpt::Mold &mold) {
              if (info.gotMold) { // C943
                context.Say(
````
- **L169 EN**: Executes a call or declaration centered on `GetExpr`.
  **L169 CN**: 执行以 `GetExpr` 为核心的调用或声明。
- **L170 EN**: Continues the surrounding expression or declaration: `info.msgSource =`.
  **L170 CN**: 继续构造周围的表达式或声明：`info.msgSource =`。
- **L171 EN**: Executes a call or declaration centered on `parser::Unwrap<parser::Variable>`.
  **L171 CN**: 执行以 `parser::Unwrap<parser::Variable>` 为核心的调用或声明。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L174 EN**: Executes a standalone statement or declaration: `statOrErr.u);`.
  **L174 CN**: 执行一条独立语句或声明：`statOrErr.u);`。
- **L175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L175 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L176 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::AllocOpt::Source &source) {`.
  **L176 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::AllocOpt::Source &source) {`。
- **L177 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L177 CN**: 开始 `if` 控制流语句并计算其条件。
- **L178 EN**: Continues logic associated with callable symbol `Say`.
  **L178 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L179 EN**: Executes a standalone statement or declaration: `"SOURCE may not be duplicated in a ALLOCATE statement"_err_en_US);`.
  **L179 CN**: 执行一条独立语句或声明：`"SOURCE may not be duplicated in a ALLOCATE statement"_err_en_US);`。
- **L180 EN**: Executes a standalone statement or declaration: `stopCheckingAllocate = true;`.
  **L180 CN**: 执行一条独立语句或声明：`stopCheckingAllocate = true;`。
- **L181 EN**: Closes the current lexical scope or compound statement.
  **L181 CN**: 结束当前词法作用域或复合语句块。
- **L182 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L182 CN**: 开始 `if` 控制流语句并计算其条件。
- **L183 EN**: Continues logic associated with callable symbol `Say`.
  **L183 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L184 EN**: Executes a standalone statement or declaration: `"At most one of source-expr and type-spec may appear in a ALLOCATE statement"_err_en_US);`.
  **L184 CN**: 执行一条独立语句或声明：`"At most one of source-expr and type-spec may appear in a ALLOCATE statement"_err_en_US);`。
- **L185 EN**: Executes a standalone statement or declaration: `stopCheckingAllocate = true;`.
  **L185 CN**: 执行一条独立语句或声明：`stopCheckingAllocate = true;`。
- **L186 EN**: Closes the current lexical scope or compound statement.
  **L186 CN**: 结束当前词法作用域或复合语句块。
- **L187 EN**: Executes a call or declaration centered on `&source.v.value`.
  **L187 CN**: 执行以 `&source.v.value` 为核心的调用或声明。
- **L188 EN**: Executes a standalone statement or declaration: `info.gotSource = true;`.
  **L188 CN**: 执行一条独立语句或声明：`info.gotSource = true;`。
- **L189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L189 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L190 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::AllocOpt::Mold &mold) {`.
  **L190 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::AllocOpt::Mold &mold) {`。
- **L191 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L191 CN**: 开始 `if` 控制流语句并计算其条件。
- **L192 EN**: Continues logic associated with callable symbol `Say`.
  **L192 CN**: 继续与可调用符号 `Say` 相关的逻辑。

### Lines 193-216

````cpp
                    "MOLD may not be duplicated in a ALLOCATE statement"_err_en_US);
                stopCheckingAllocate = true;
              }
              if (info.gotSource || info.gotTypeSpec) { // C944
                context.Say(
                    "At most one of source-expr and type-spec may appear in a ALLOCATE statement"_err_en_US);
                stopCheckingAllocate = true;
              }
              parserSourceExpr = &mold.v.value();
              info.gotMold = true;
            },
            [&](const parser::AllocOpt::Stream &stream) { // CUDA
              if (info.gotStream) {
                context.Say(
                    "STREAM may not be duplicated in a ALLOCATE statement"_err_en_US);
                stopCheckingAllocate = true;
              }
              info.gotStream = true;
            },
            [&](const parser::AllocOpt::Pinned &pinned) { // CUDA
              if (info.gotPinned) {
                context.Say(
                    "PINNED may not be duplicated in a ALLOCATE statement"_err_en_US);
                stopCheckingAllocate = true;
````
- **L193 EN**: Executes a standalone statement or declaration: `"MOLD may not be duplicated in a ALLOCATE statement"_err_en_US);`.
  **L193 CN**: 执行一条独立语句或声明：`"MOLD may not be duplicated in a ALLOCATE statement"_err_en_US);`。
- **L194 EN**: Executes a standalone statement or declaration: `stopCheckingAllocate = true;`.
  **L194 CN**: 执行一条独立语句或声明：`stopCheckingAllocate = true;`。
- **L195 EN**: Closes the current lexical scope or compound statement.
  **L195 CN**: 结束当前词法作用域或复合语句块。
- **L196 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L196 CN**: 开始 `if` 控制流语句并计算其条件。
- **L197 EN**: Continues logic associated with callable symbol `Say`.
  **L197 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L198 EN**: Executes a standalone statement or declaration: `"At most one of source-expr and type-spec may appear in a ALLOCATE statement"_err_en_US);`.
  **L198 CN**: 执行一条独立语句或声明：`"At most one of source-expr and type-spec may appear in a ALLOCATE statement"_err_en_US);`。
- **L199 EN**: Executes a standalone statement or declaration: `stopCheckingAllocate = true;`.
  **L199 CN**: 执行一条独立语句或声明：`stopCheckingAllocate = true;`。
- **L200 EN**: Closes the current lexical scope or compound statement.
  **L200 CN**: 结束当前词法作用域或复合语句块。
- **L201 EN**: Executes a call or declaration centered on `&mold.v.value`.
  **L201 CN**: 执行以 `&mold.v.value` 为核心的调用或声明。
- **L202 EN**: Executes a standalone statement or declaration: `info.gotMold = true;`.
  **L202 CN**: 执行一条独立语句或声明：`info.gotMold = true;`。
- **L203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L203 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L204 EN**: Continues the surrounding expression or declaration: `[&](const parser::AllocOpt::Stream &stream) { // CUDA`.
  **L204 CN**: 继续构造周围的表达式或声明：`[&](const parser::AllocOpt::Stream &stream) { // CUDA`。
- **L205 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L205 CN**: 开始 `if` 控制流语句并计算其条件。
- **L206 EN**: Continues logic associated with callable symbol `Say`.
  **L206 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L207 EN**: Executes a standalone statement or declaration: `"STREAM may not be duplicated in a ALLOCATE statement"_err_en_US);`.
  **L207 CN**: 执行一条独立语句或声明：`"STREAM may not be duplicated in a ALLOCATE statement"_err_en_US);`。
- **L208 EN**: Executes a standalone statement or declaration: `stopCheckingAllocate = true;`.
  **L208 CN**: 执行一条独立语句或声明：`stopCheckingAllocate = true;`。
- **L209 EN**: Closes the current lexical scope or compound statement.
  **L209 CN**: 结束当前词法作用域或复合语句块。
- **L210 EN**: Executes a standalone statement or declaration: `info.gotStream = true;`.
  **L210 CN**: 执行一条独立语句或声明：`info.gotStream = true;`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L212 EN**: Continues the surrounding expression or declaration: `[&](const parser::AllocOpt::Pinned &pinned) { // CUDA`.
  **L212 CN**: 继续构造周围的表达式或声明：`[&](const parser::AllocOpt::Pinned &pinned) { // CUDA`。
- **L213 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L213 CN**: 开始 `if` 控制流语句并计算其条件。
- **L214 EN**: Continues logic associated with callable symbol `Say`.
  **L214 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L215 EN**: Executes a standalone statement or declaration: `"PINNED may not be duplicated in a ALLOCATE statement"_err_en_US);`.
  **L215 CN**: 执行一条独立语句或声明：`"PINNED may not be duplicated in a ALLOCATE statement"_err_en_US);`。
- **L216 EN**: Executes a standalone statement or declaration: `stopCheckingAllocate = true;`.
  **L216 CN**: 执行一条独立语句或声明：`stopCheckingAllocate = true;`。

### Lines 217-240

````cpp
              }
              info.gotPinned = true;
            },
        },
        allocOpt.u);
  }

  if (stopCheckingAllocate) {
    return std::nullopt;
  }

  if (info.gotSource || info.gotMold) {
    if (const auto *expr{GetExpr(context, DEREF(parserSourceExpr))}) {
      parser::CharBlock at{parserSourceExpr->source};
      info.sourceExprType = expr->GetType();
      if (!info.sourceExprType) {
        context.Say(at,
            "Typeless item not allowed as SOURCE or MOLD in ALLOCATE"_err_en_US);
        return std::nullopt;
      }
      info.sourceExprRank = expr->Rank();
      info.sourceExprLoc = parserSourceExpr->source;
      if (const DerivedTypeSpec *
          derived{evaluate::GetDerivedTypeSpec(info.sourceExprType)}) {
````
- **L217 EN**: Closes the current lexical scope or compound statement.
  **L217 CN**: 结束当前词法作用域或复合语句块。
- **L218 EN**: Executes a standalone statement or declaration: `info.gotPinned = true;`.
  **L218 CN**: 执行一条独立语句或声明：`info.gotPinned = true;`。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L221 EN**: Executes a standalone statement or declaration: `allocOpt.u);`.
  **L221 CN**: 执行一条独立语句或声明：`allocOpt.u);`。
- **L222 EN**: Closes the current lexical scope or compound statement.
  **L222 CN**: 结束当前词法作用域或复合语句块。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L224 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L224 CN**: 开始 `if` 控制流语句并计算其条件。
- **L225 EN**: Returns from the current function with `std::nullopt`.
  **L225 CN**: 以 `std::nullopt` 从当前函数返回。
- **L226 EN**: Closes the current lexical scope or compound statement.
  **L226 CN**: 结束当前词法作用域或复合语句块。
- **L227 EN**: Blank line separating nearby declarations or logic blocks.
  **L227 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L228 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L228 CN**: 开始 `if` 控制流语句并计算其条件。
- **L229 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L229 CN**: 开始 `if` 控制流语句并计算其条件。
- **L230 EN**: Executes a standalone statement or declaration: `parser::CharBlock at{parserSourceExpr->source};`.
  **L230 CN**: 执行一条独立语句或声明：`parser::CharBlock at{parserSourceExpr->source};`。
- **L231 EN**: Executes a call or declaration centered on `expr->GetType`.
  **L231 CN**: 执行以 `expr->GetType` 为核心的调用或声明。
- **L232 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L232 CN**: 开始 `if` 控制流语句并计算其条件。
- **L233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(at,`.
  **L233 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(at,`。
- **L234 EN**: Executes a standalone statement or declaration: `"Typeless item not allowed as SOURCE or MOLD in ALLOCATE"_err_en_US);`.
  **L234 CN**: 执行一条独立语句或声明：`"Typeless item not allowed as SOURCE or MOLD in ALLOCATE"_err_en_US);`。
- **L235 EN**: Returns from the current function with `std::nullopt`.
  **L235 CN**: 以 `std::nullopt` 从当前函数返回。
- **L236 EN**: Closes the current lexical scope or compound statement.
  **L236 CN**: 结束当前词法作用域或复合语句块。
- **L237 EN**: Executes a call or declaration centered on `expr->Rank`.
  **L237 CN**: 执行以 `expr->Rank` 为核心的调用或声明。
- **L238 EN**: Executes a standalone statement or declaration: `info.sourceExprLoc = parserSourceExpr->source;`.
  **L238 CN**: 执行一条独立语句或声明：`info.sourceExprLoc = parserSourceExpr->source;`。
- **L239 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L239 CN**: 开始 `if` 控制流语句并计算其条件。
- **L240 EN**: Starts a function, method, lambda, or structured scope: `derived{evaluate::GetDerivedTypeSpec(info.sourceExprType)}) {`.
  **L240 CN**: 开始一个函数、方法、lambda 或结构化作用域：`derived{evaluate::GetDerivedTypeSpec(info.sourceExprType)}) {`。

### Lines 241-264

````cpp
        // C949
        if (auto it{FindCoarrayUltimateComponent(*derived)}) {
          context
              .Say(at,
                  "SOURCE or MOLD expression must not have a type with a coarray ultimate component"_err_en_US)
              .Attach(it->name(),
                  "Type '%s' has coarray ultimate component '%s' declared here"_en_US,
                  info.sourceExprType.value().AsFortran(),
                  it.BuildResultDesignatorName());
        }
        if (info.gotSource) {
          // C948
          if (IsEventTypeOrLockType(derived)) {
            context.Say(at,
                "SOURCE expression type must not be EVENT_TYPE or LOCK_TYPE from ISO_FORTRAN_ENV"_err_en_US);
          } else if (auto it{FindEventOrLockPotentialComponent(*derived)}) {
            context
                .Say(at,
                    "SOURCE expression type must not have potential subobject "
                    "component"
                    " of type EVENT_TYPE or LOCK_TYPE from ISO_FORTRAN_ENV"_err_en_US)
                .Attach(it->name(),
                    "Type '%s' has potential ultimate component '%s' declared here"_en_US,
                    info.sourceExprType.value().AsFortran(),
````
- **L241 EN**: Comment explains nearby logic, intent, or metadata: `C949`.
  **L241 CN**: 注释说明附近代码的逻辑、意图或元数据：`C949`。
- **L242 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L242 CN**: 开始 `if` 控制流语句并计算其条件。
- **L243 EN**: Continues the surrounding expression or declaration: `context`.
  **L243 CN**: 继续构造周围的表达式或声明：`context`。
- **L244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(at,`.
  **L244 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(at,`。
- **L245 EN**: Continues the surrounding expression or declaration: `"SOURCE or MOLD expression must not have a type with a coarray ultimate component"_err_en_US)`.
  **L245 CN**: 继续构造周围的表达式或声明：`"SOURCE or MOLD expression must not have a type with a coarray ultimate component"_err_en_US)`。
- **L246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(it->name(),`.
  **L246 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(it->name(),`。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Type '%s' has coarray ultimate component '%s' declared here"_en_US,`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Type '%s' has coarray ultimate component '%s' declared here"_en_US,`。
- **L248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `info.sourceExprType.value().AsFortran(),`.
  **L248 CN**: 继续一个多行参数列表、初始化器或聚合项：`info.sourceExprType.value().AsFortran(),`。
- **L249 EN**: Executes a call or declaration centered on `it.BuildResultDesignatorName`.
  **L249 CN**: 执行以 `it.BuildResultDesignatorName` 为核心的调用或声明。
- **L250 EN**: Closes the current lexical scope or compound statement.
  **L250 CN**: 结束当前词法作用域或复合语句块。
- **L251 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L251 CN**: 开始 `if` 控制流语句并计算其条件。
- **L252 EN**: Comment explains nearby logic, intent, or metadata: `C948`.
  **L252 CN**: 注释说明附近代码的逻辑、意图或元数据：`C948`。
- **L253 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L253 CN**: 开始 `if` 控制流语句并计算其条件。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(at,`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(at,`。
- **L255 EN**: Executes a standalone statement or declaration: `"SOURCE expression type must not be EVENT_TYPE or LOCK_TYPE from ISO_FORTRAN_ENV"_err_en_US);`.
  **L255 CN**: 执行一条独立语句或声明：`"SOURCE expression type must not be EVENT_TYPE or LOCK_TYPE from ISO_FORTRAN_ENV"_err_en_US);`。
- **L256 EN**: Transitions from the previous branch into an `else if` condition.
  **L256 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L257 EN**: Continues the surrounding expression or declaration: `context`.
  **L257 CN**: 继续构造周围的表达式或声明：`context`。
- **L258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(at,`.
  **L258 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(at,`。
- **L259 EN**: Continues the surrounding expression or declaration: `"SOURCE expression type must not have potential subobject "`.
  **L259 CN**: 继续构造周围的表达式或声明：`"SOURCE expression type must not have potential subobject "`。
- **L260 EN**: Continues the surrounding expression or declaration: `"component"`.
  **L260 CN**: 继续构造周围的表达式或声明：`"component"`。
- **L261 EN**: Continues the surrounding expression or declaration: `" of type EVENT_TYPE or LOCK_TYPE from ISO_FORTRAN_ENV"_err_en_US)`.
  **L261 CN**: 继续构造周围的表达式或声明：`" of type EVENT_TYPE or LOCK_TYPE from ISO_FORTRAN_ENV"_err_en_US)`。
- **L262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(it->name(),`.
  **L262 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(it->name(),`。
- **L263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Type '%s' has potential ultimate component '%s' declared here"_en_US,`.
  **L263 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Type '%s' has potential ultimate component '%s' declared here"_en_US,`。
- **L264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `info.sourceExprType.value().AsFortran(),`.
  **L264 CN**: 继续一个多行参数列表、初始化器或聚合项：`info.sourceExprType.value().AsFortran(),`。

### Lines 265-288

````cpp
                    it.BuildResultDesignatorName());
          }
        }
      }
      if (info.gotSource) { // C1594(6) - SOURCE= restrictions when pure
        const Scope &scope{context.FindScope(at)};
        if (FindPureProcedureContaining(scope)) {
          parser::ContextualMessages messages{at, &context.messages()};
          CheckCopyabilityInPureScope(messages, *expr, scope);
        }
      }
      auto maybeShape{evaluate::GetShape(context.foldingContext(), *expr)};
      info.sourceExprShape =
          evaluate::AsConstantExtents(context.foldingContext(), maybeShape);
    } else {
      // Error already reported on source expression.
      // Do not continue allocate checks.
      return std::nullopt;
    }
  }

  return info;
}

````
- **L265 EN**: Executes a call or declaration centered on `it.BuildResultDesignatorName`.
  **L265 CN**: 执行以 `it.BuildResultDesignatorName` 为核心的调用或声明。
- **L266 EN**: Closes the current lexical scope or compound statement.
  **L266 CN**: 结束当前词法作用域或复合语句块。
- **L267 EN**: Closes the current lexical scope or compound statement.
  **L267 CN**: 结束当前词法作用域或复合语句块。
- **L268 EN**: Closes the current lexical scope or compound statement.
  **L268 CN**: 结束当前词法作用域或复合语句块。
- **L269 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L269 CN**: 开始 `if` 控制流语句并计算其条件。
- **L270 EN**: Executes a call or declaration centered on `&scope{context.FindScope`.
  **L270 CN**: 执行以 `&scope{context.FindScope` 为核心的调用或声明。
- **L271 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L271 CN**: 开始 `if` 控制流语句并计算其条件。
- **L272 EN**: Executes a call or declaration centered on `&context.messages`.
  **L272 CN**: 执行以 `&context.messages` 为核心的调用或声明。
- **L273 EN**: Executes a call or declaration centered on `CheckCopyabilityInPureScope`.
  **L273 CN**: 执行以 `CheckCopyabilityInPureScope` 为核心的调用或声明。
- **L274 EN**: Closes the current lexical scope or compound statement.
  **L274 CN**: 结束当前词法作用域或复合语句块。
- **L275 EN**: Closes the current lexical scope or compound statement.
  **L275 CN**: 结束当前词法作用域或复合语句块。
- **L276 EN**: Executes a call or declaration centered on `maybeShape{evaluate::GetShape`.
  **L276 CN**: 执行以 `maybeShape{evaluate::GetShape` 为核心的调用或声明。
- **L277 EN**: Continues the surrounding expression or declaration: `info.sourceExprShape =`.
  **L277 CN**: 继续构造周围的表达式或声明：`info.sourceExprShape =`。
- **L278 EN**: Executes a call or declaration centered on `evaluate::AsConstantExtents`.
  **L278 CN**: 执行以 `evaluate::AsConstantExtents` 为核心的调用或声明。
- **L279 EN**: Transitions from the previous branch into the alternative path.
  **L279 CN**: 从前一个分支过渡到备选路径。
- **L280 EN**: Comment explains nearby logic, intent, or metadata: `Error already reported on source expression.`.
  **L280 CN**: 注释说明附近代码的逻辑、意图或元数据：`Error already reported on source expression.`。
- **L281 EN**: Comment explains nearby logic, intent, or metadata: `Do not continue allocate checks.`.
  **L281 CN**: 注释说明附近代码的逻辑、意图或元数据：`Do not continue allocate checks.`。
- **L282 EN**: Returns from the current function with `std::nullopt`.
  **L282 CN**: 以 `std::nullopt` 从当前函数返回。
- **L283 EN**: Closes the current lexical scope or compound statement.
  **L283 CN**: 结束当前词法作用域或复合语句块。
- **L284 EN**: Closes the current lexical scope or compound statement.
  **L284 CN**: 结束当前词法作用域或复合语句块。
- **L285 EN**: Blank line separating nearby declarations or logic blocks.
  **L285 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L286 EN**: Returns from the current function with `info`.
  **L286 CN**: 以 `info` 从当前函数返回。
- **L287 EN**: Closes the current lexical scope or compound statement.
  **L287 CN**: 结束当前词法作用域或复合语句块。
- **L288 EN**: Blank line separating nearby declarations or logic blocks.
  **L288 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 289-312

````cpp
// Beware, type compatibility is not symmetric, IsTypeCompatible checks that
// type1 is type compatible with type2. Note: type parameters are not considered
// in this test.
static bool IsTypeCompatible(
    const DeclTypeSpec &type1, const DerivedTypeSpec &derivedType2) {
  if (const DerivedTypeSpec * derivedType1{type1.AsDerived()}) {
    if (type1.category() == DeclTypeSpec::Category::TypeDerived) {
      return evaluate::AreSameDerivedTypeIgnoringTypeParameters(
          *derivedType1, derivedType2);
    } else if (type1.category() == DeclTypeSpec::Category::ClassDerived) {
      for (const DerivedTypeSpec *parent{&derivedType2}; parent;
           parent = parent->typeSymbol().GetParentTypeSpec()) {
        if (evaluate::AreSameDerivedTypeIgnoringTypeParameters(
                *derivedType1, *parent)) {
          return true;
        }
      }
    }
  }
  return false;
}

static bool IsTypeCompatible(
    const DeclTypeSpec &type1, const DeclTypeSpec &type2) {
````
- **L289 EN**: Comment explains nearby logic, intent, or metadata: `Beware, type compatibility is not symmetric, IsTypeCompatible checks that`.
  **L289 CN**: 注释说明附近代码的逻辑、意图或元数据：`Beware, type compatibility is not symmetric, IsTypeCompatible checks that`。
- **L290 EN**: Comment explains nearby logic, intent, or metadata: `type1 is type compatible with type2. Note: type parameters are not considered`.
  **L290 CN**: 注释说明附近代码的逻辑、意图或元数据：`type1 is type compatible with type2. Note: type parameters are not considered`。
- **L291 EN**: Comment explains nearby logic, intent, or metadata: `in this test.`.
  **L291 CN**: 注释说明附近代码的逻辑、意图或元数据：`in this test.`。
- **L292 EN**: Continues logic associated with callable symbol `IsTypeCompatible`.
  **L292 CN**: 继续与可调用符号 `IsTypeCompatible` 相关的逻辑。
- **L293 EN**: Continues the surrounding expression or declaration: `const DeclTypeSpec &type1, const DerivedTypeSpec &derivedType2) {`.
  **L293 CN**: 继续构造周围的表达式或声明：`const DeclTypeSpec &type1, const DerivedTypeSpec &derivedType2) {`。
- **L294 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L294 CN**: 开始 `if` 控制流语句并计算其条件。
- **L295 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L295 CN**: 开始 `if` 控制流语句并计算其条件。
- **L296 EN**: Returns from the current function with `evaluate::AreSameDerivedTypeIgnoringTypeParameters(`.
  **L296 CN**: 以 `evaluate::AreSameDerivedTypeIgnoringTypeParameters(` 从当前函数返回。
- **L297 EN**: Comment explains nearby logic, intent, or metadata: `derivedType1, derivedType2);`.
  **L297 CN**: 注释说明附近代码的逻辑、意图或元数据：`derivedType1, derivedType2);`。
- **L298 EN**: Transitions from the previous branch into an `else if` condition.
  **L298 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L299 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L299 CN**: 开始 `for` 控制流语句并计算其条件。
- **L300 EN**: Starts a function, method, lambda, or structured scope: `parent = parent->typeSymbol().GetParentTypeSpec()) {`.
  **L300 CN**: 开始一个函数、方法、lambda 或结构化作用域：`parent = parent->typeSymbol().GetParentTypeSpec()) {`。
- **L301 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L301 CN**: 开始 `if` 控制流语句并计算其条件。
- **L302 EN**: Comment explains nearby logic, intent, or metadata: `derivedType1, *parent)) {`.
  **L302 CN**: 注释说明附近代码的逻辑、意图或元数据：`derivedType1, *parent)) {`。
- **L303 EN**: Returns from the current function with `true`.
  **L303 CN**: 以 `true` 从当前函数返回。
- **L304 EN**: Closes the current lexical scope or compound statement.
  **L304 CN**: 结束当前词法作用域或复合语句块。
- **L305 EN**: Closes the current lexical scope or compound statement.
  **L305 CN**: 结束当前词法作用域或复合语句块。
- **L306 EN**: Closes the current lexical scope or compound statement.
  **L306 CN**: 结束当前词法作用域或复合语句块。
- **L307 EN**: Closes the current lexical scope or compound statement.
  **L307 CN**: 结束当前词法作用域或复合语句块。
- **L308 EN**: Returns from the current function with `false`.
  **L308 CN**: 以 `false` 从当前函数返回。
- **L309 EN**: Closes the current lexical scope or compound statement.
  **L309 CN**: 结束当前词法作用域或复合语句块。
- **L310 EN**: Blank line separating nearby declarations or logic blocks.
  **L310 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L311 EN**: Continues logic associated with callable symbol `IsTypeCompatible`.
  **L311 CN**: 继续与可调用符号 `IsTypeCompatible` 相关的逻辑。
- **L312 EN**: Continues the surrounding expression or declaration: `const DeclTypeSpec &type1, const DeclTypeSpec &type2) {`.
  **L312 CN**: 继续构造周围的表达式或声明：`const DeclTypeSpec &type1, const DeclTypeSpec &type2) {`。

### Lines 313-336

````cpp
  if (type1.category() == DeclTypeSpec::Category::ClassStar) {
    // TypeStar does not make sense in allocate context because assumed type
    // cannot be allocatable (C709)
    return true;
  }
  if (const IntrinsicTypeSpec * intrinsicType2{type2.AsIntrinsic()}) {
    if (const IntrinsicTypeSpec * intrinsicType1{type1.AsIntrinsic()}) {
      return intrinsicType1->category() == intrinsicType2->category();
    } else {
      return false;
    }
  } else if (const DerivedTypeSpec * derivedType2{type2.AsDerived()}) {
    return IsTypeCompatible(type1, *derivedType2);
  }
  return false;
}

static bool IsTypeCompatible(
    const DeclTypeSpec &type1, const evaluate::DynamicType &type2) {
  if (type1.category() == DeclTypeSpec::Category::ClassStar) {
    // TypeStar does not make sense in allocate context because assumed type
    // cannot be allocatable (C709)
    return true;
  }
````
- **L313 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L313 CN**: 开始 `if` 控制流语句并计算其条件。
- **L314 EN**: Comment explains nearby logic, intent, or metadata: `TypeStar does not make sense in allocate context because assumed type`.
  **L314 CN**: 注释说明附近代码的逻辑、意图或元数据：`TypeStar does not make sense in allocate context because assumed type`。
- **L315 EN**: Comment explains nearby logic, intent, or metadata: `cannot be allocatable (C709)`.
  **L315 CN**: 注释说明附近代码的逻辑、意图或元数据：`cannot be allocatable (C709)`。
- **L316 EN**: Returns from the current function with `true`.
  **L316 CN**: 以 `true` 从当前函数返回。
- **L317 EN**: Closes the current lexical scope or compound statement.
  **L317 CN**: 结束当前词法作用域或复合语句块。
- **L318 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L318 CN**: 开始 `if` 控制流语句并计算其条件。
- **L319 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L319 CN**: 开始 `if` 控制流语句并计算其条件。
- **L320 EN**: Returns from the current function with `intrinsicType1->category() == intrinsicType2->category()`.
  **L320 CN**: 以 `intrinsicType1->category() == intrinsicType2->category()` 从当前函数返回。
- **L321 EN**: Transitions from the previous branch into the alternative path.
  **L321 CN**: 从前一个分支过渡到备选路径。
- **L322 EN**: Returns from the current function with `false`.
  **L322 CN**: 以 `false` 从当前函数返回。
- **L323 EN**: Closes the current lexical scope or compound statement.
  **L323 CN**: 结束当前词法作用域或复合语句块。
- **L324 EN**: Transitions from the previous branch into an `else if` condition.
  **L324 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L325 EN**: Returns from the current function with `IsTypeCompatible(type1, *derivedType2)`.
  **L325 CN**: 以 `IsTypeCompatible(type1, *derivedType2)` 从当前函数返回。
- **L326 EN**: Closes the current lexical scope or compound statement.
  **L326 CN**: 结束当前词法作用域或复合语句块。
- **L327 EN**: Returns from the current function with `false`.
  **L327 CN**: 以 `false` 从当前函数返回。
- **L328 EN**: Closes the current lexical scope or compound statement.
  **L328 CN**: 结束当前词法作用域或复合语句块。
- **L329 EN**: Blank line separating nearby declarations or logic blocks.
  **L329 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L330 EN**: Continues logic associated with callable symbol `IsTypeCompatible`.
  **L330 CN**: 继续与可调用符号 `IsTypeCompatible` 相关的逻辑。
- **L331 EN**: Continues the surrounding expression or declaration: `const DeclTypeSpec &type1, const evaluate::DynamicType &type2) {`.
  **L331 CN**: 继续构造周围的表达式或声明：`const DeclTypeSpec &type1, const evaluate::DynamicType &type2) {`。
- **L332 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L332 CN**: 开始 `if` 控制流语句并计算其条件。
- **L333 EN**: Comment explains nearby logic, intent, or metadata: `TypeStar does not make sense in allocate context because assumed type`.
  **L333 CN**: 注释说明附近代码的逻辑、意图或元数据：`TypeStar does not make sense in allocate context because assumed type`。
- **L334 EN**: Comment explains nearby logic, intent, or metadata: `cannot be allocatable (C709)`.
  **L334 CN**: 注释说明附近代码的逻辑、意图或元数据：`cannot be allocatable (C709)`。
- **L335 EN**: Returns from the current function with `true`.
  **L335 CN**: 以 `true` 从当前函数返回。
- **L336 EN**: Closes the current lexical scope or compound statement.
  **L336 CN**: 结束当前词法作用域或复合语句块。

### Lines 337-360

````cpp
  if (type2.category() != evaluate::TypeCategory::Derived) {
    if (const IntrinsicTypeSpec * intrinsicType1{type1.AsIntrinsic()}) {
      return intrinsicType1->category() == type2.category();
    } else {
      return false;
    }
  } else if (!type2.IsUnlimitedPolymorphic()) {
    return IsTypeCompatible(type1, type2.GetDerivedTypeSpec());
  }
  return false;
}

// Note: Check assumes  type1 is compatible with type2. type2 may have more type
// parameters than type1 but if a type2 type parameter is assumed, then this
// check enforce that type1 has it. type1 can be unlimited polymorphic, but not
// type2.
static bool HaveSameAssumedTypeParameters(
    const DeclTypeSpec &type1, const DeclTypeSpec &type2) {
  if (type2.category() == DeclTypeSpec::Category::Character) {
    bool type2LengthIsAssumed{type2.characterTypeSpec().length().isAssumed()};
    if (type1.category() == DeclTypeSpec::Category::Character) {
      return type1.characterTypeSpec().length().isAssumed() ==
          type2LengthIsAssumed;
    }
````
- **L337 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L337 CN**: 开始 `if` 控制流语句并计算其条件。
- **L338 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L338 CN**: 开始 `if` 控制流语句并计算其条件。
- **L339 EN**: Returns from the current function with `intrinsicType1->category() == type2.category()`.
  **L339 CN**: 以 `intrinsicType1->category() == type2.category()` 从当前函数返回。
- **L340 EN**: Transitions from the previous branch into the alternative path.
  **L340 CN**: 从前一个分支过渡到备选路径。
- **L341 EN**: Returns from the current function with `false`.
  **L341 CN**: 以 `false` 从当前函数返回。
- **L342 EN**: Closes the current lexical scope or compound statement.
  **L342 CN**: 结束当前词法作用域或复合语句块。
- **L343 EN**: Transitions from the previous branch into an `else if` condition.
  **L343 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L344 EN**: Returns from the current function with `IsTypeCompatible(type1, type2.GetDerivedTypeSpec())`.
  **L344 CN**: 以 `IsTypeCompatible(type1, type2.GetDerivedTypeSpec())` 从当前函数返回。
- **L345 EN**: Closes the current lexical scope or compound statement.
  **L345 CN**: 结束当前词法作用域或复合语句块。
- **L346 EN**: Returns from the current function with `false`.
  **L346 CN**: 以 `false` 从当前函数返回。
- **L347 EN**: Closes the current lexical scope or compound statement.
  **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Blank line separating nearby declarations or logic blocks.
  **L348 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L349 EN**: Comment explains nearby logic, intent, or metadata: `Note: Check assumes  type1 is compatible with type2. type2 may have more type`.
  **L349 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note: Check assumes  type1 is compatible with type2. type2 may have more type`。
- **L350 EN**: Comment explains nearby logic, intent, or metadata: `parameters than type1 but if a type2 type parameter is assumed, then this`.
  **L350 CN**: 注释说明附近代码的逻辑、意图或元数据：`parameters than type1 but if a type2 type parameter is assumed, then this`。
- **L351 EN**: Comment explains nearby logic, intent, or metadata: `check enforce that type1 has it. type1 can be unlimited polymorphic, but not`.
  **L351 CN**: 注释说明附近代码的逻辑、意图或元数据：`check enforce that type1 has it. type1 can be unlimited polymorphic, but not`。
- **L352 EN**: Comment explains nearby logic, intent, or metadata: `type2.`.
  **L352 CN**: 注释说明附近代码的逻辑、意图或元数据：`type2.`。
- **L353 EN**: Continues logic associated with callable symbol `HaveSameAssumedTypeParameters`.
  **L353 CN**: 继续与可调用符号 `HaveSameAssumedTypeParameters` 相关的逻辑。
- **L354 EN**: Continues the surrounding expression or declaration: `const DeclTypeSpec &type1, const DeclTypeSpec &type2) {`.
  **L354 CN**: 继续构造周围的表达式或声明：`const DeclTypeSpec &type1, const DeclTypeSpec &type2) {`。
- **L355 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L355 CN**: 开始 `if` 控制流语句并计算其条件。
- **L356 EN**: Executes a call or declaration centered on `type2LengthIsAssumed{type2.characterTypeSpec`.
  **L356 CN**: 执行以 `type2LengthIsAssumed{type2.characterTypeSpec` 为核心的调用或声明。
- **L357 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L357 CN**: 开始 `if` 控制流语句并计算其条件。
- **L358 EN**: Returns from the current function with `type1.characterTypeSpec().length().isAssumed() ==`.
  **L358 CN**: 以 `type1.characterTypeSpec().length().isAssumed() ==` 从当前函数返回。
- **L359 EN**: Executes a standalone statement or declaration: `type2LengthIsAssumed;`.
  **L359 CN**: 执行一条独立语句或声明：`type2LengthIsAssumed;`。
- **L360 EN**: Closes the current lexical scope or compound statement.
  **L360 CN**: 结束当前词法作用域或复合语句块。

### Lines 361-384

````cpp
    // It is possible to reach this if type1 is unlimited polymorphic
    return !type2LengthIsAssumed;
  } else if (const DerivedTypeSpec * derivedType2{type2.AsDerived()}) {
    int type2AssumedParametersCount{0};
    int type1AssumedParametersCount{0};
    for (const auto &pair : derivedType2->parameters()) {
      type2AssumedParametersCount += pair.second.isAssumed();
    }
    // type1 may be unlimited polymorphic
    if (const DerivedTypeSpec * derivedType1{type1.AsDerived()}) {
      for (auto it{derivedType1->parameters().begin()};
           it != derivedType1->parameters().end(); ++it) {
        if (it->second.isAssumed()) {
          ++type1AssumedParametersCount;
          const ParamValue *param{derivedType2->FindParameter(it->first)};
          if (!param || !param->isAssumed()) {
            // type1 has an assumed parameter that is not a type parameter of
            // type2 or not assumed in type2.
            return false;
          }
        }
      }
    }
    // Will return false if type2 has type parameters that are not assumed in
````
- **L361 EN**: Comment explains nearby logic, intent, or metadata: `It is possible to reach this if type1 is unlimited polymorphic`.
  **L361 CN**: 注释说明附近代码的逻辑、意图或元数据：`It is possible to reach this if type1 is unlimited polymorphic`。
- **L362 EN**: Returns from the current function with `!type2LengthIsAssumed`.
  **L362 CN**: 以 `!type2LengthIsAssumed` 从当前函数返回。
- **L363 EN**: Transitions from the previous branch into an `else if` condition.
  **L363 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L364 EN**: Executes a standalone statement or declaration: `int type2AssumedParametersCount{0};`.
  **L364 CN**: 执行一条独立语句或声明：`int type2AssumedParametersCount{0};`。
- **L365 EN**: Executes a standalone statement or declaration: `int type1AssumedParametersCount{0};`.
  **L365 CN**: 执行一条独立语句或声明：`int type1AssumedParametersCount{0};`。
- **L366 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L366 CN**: 开始 `for` 控制流语句并计算其条件。
- **L367 EN**: Executes a call or declaration centered on `pair.second.isAssumed`.
  **L367 CN**: 执行以 `pair.second.isAssumed` 为核心的调用或声明。
- **L368 EN**: Closes the current lexical scope or compound statement.
  **L368 CN**: 结束当前词法作用域或复合语句块。
- **L369 EN**: Comment explains nearby logic, intent, or metadata: `type1 may be unlimited polymorphic`.
  **L369 CN**: 注释说明附近代码的逻辑、意图或元数据：`type1 may be unlimited polymorphic`。
- **L370 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L370 CN**: 开始 `if` 控制流语句并计算其条件。
- **L371 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L371 CN**: 开始 `for` 控制流语句并计算其条件。
- **L372 EN**: Starts a function, method, lambda, or structured scope: `it != derivedType1->parameters().end(); ++it) {`.
  **L372 CN**: 开始一个函数、方法、lambda 或结构化作用域：`it != derivedType1->parameters().end(); ++it) {`。
- **L373 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L373 CN**: 开始 `if` 控制流语句并计算其条件。
- **L374 EN**: Executes a standalone statement or declaration: `++type1AssumedParametersCount;`.
  **L374 CN**: 执行一条独立语句或声明：`++type1AssumedParametersCount;`。
- **L375 EN**: Executes a call or declaration centered on `*param{derivedType2->FindParameter`.
  **L375 CN**: 执行以 `*param{derivedType2->FindParameter` 为核心的调用或声明。
- **L376 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L376 CN**: 开始 `if` 控制流语句并计算其条件。
- **L377 EN**: Comment explains nearby logic, intent, or metadata: `type1 has an assumed parameter that is not a type parameter of`.
  **L377 CN**: 注释说明附近代码的逻辑、意图或元数据：`type1 has an assumed parameter that is not a type parameter of`。
- **L378 EN**: Comment explains nearby logic, intent, or metadata: `type2 or not assumed in type2.`.
  **L378 CN**: 注释说明附近代码的逻辑、意图或元数据：`type2 or not assumed in type2.`。
- **L379 EN**: Returns from the current function with `false`.
  **L379 CN**: 以 `false` 从当前函数返回。
- **L380 EN**: Closes the current lexical scope or compound statement.
  **L380 CN**: 结束当前词法作用域或复合语句块。
- **L381 EN**: Closes the current lexical scope or compound statement.
  **L381 CN**: 结束当前词法作用域或复合语句块。
- **L382 EN**: Closes the current lexical scope or compound statement.
  **L382 CN**: 结束当前词法作用域或复合语句块。
- **L383 EN**: Closes the current lexical scope or compound statement.
  **L383 CN**: 结束当前词法作用域或复合语句块。
- **L384 EN**: Comment explains nearby logic, intent, or metadata: `Will return false if type2 has type parameters that are not assumed in`.
  **L384 CN**: 注释说明附近代码的逻辑、意图或元数据：`Will return false if type2 has type parameters that are not assumed in`。

### Lines 385-408

````cpp
    // type1 or do not exist in type1
    return type1AssumedParametersCount == type2AssumedParametersCount;
  }
  return true; // other intrinsic types have no length type parameters
}

static std::optional<std::int64_t> GetTypeParameterInt64Value(
    const Symbol &parameterSymbol, const DerivedTypeSpec &derivedType) {
  if (const ParamValue *
      paramValue{derivedType.FindParameter(parameterSymbol.name())}) {
    return evaluate::ToInt64(paramValue->GetExplicit());
  }
  return std::nullopt;
}

static bool HaveCompatibleTypeParameters(
    const DerivedTypeSpec &derivedType1, const DerivedTypeSpec &derivedType2) {
  for (const Symbol &symbol :
      OrderParameterDeclarations(derivedType1.typeSymbol())) {
    auto v1{GetTypeParameterInt64Value(symbol, derivedType1)};
    auto v2{GetTypeParameterInt64Value(symbol, derivedType2)};
    if (v1 && v2 && *v1 != *v2) {
      return false;
    }
````
- **L385 EN**: Comment explains nearby logic, intent, or metadata: `type1 or do not exist in type1`.
  **L385 CN**: 注释说明附近代码的逻辑、意图或元数据：`type1 or do not exist in type1`。
- **L386 EN**: Returns from the current function with `type1AssumedParametersCount == type2AssumedParametersCount`.
  **L386 CN**: 以 `type1AssumedParametersCount == type2AssumedParametersCount` 从当前函数返回。
- **L387 EN**: Closes the current lexical scope or compound statement.
  **L387 CN**: 结束当前词法作用域或复合语句块。
- **L388 EN**: Returns from the current function with `true; // other intrinsic types have no length type parameters`.
  **L388 CN**: 以 `true; // other intrinsic types have no length type parameters` 从当前函数返回。
- **L389 EN**: Closes the current lexical scope or compound statement.
  **L389 CN**: 结束当前词法作用域或复合语句块。
- **L390 EN**: Blank line separating nearby declarations or logic blocks.
  **L390 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L391 EN**: Continues logic associated with callable symbol `GetTypeParameterInt64Value`.
  **L391 CN**: 继续与可调用符号 `GetTypeParameterInt64Value` 相关的逻辑。
- **L392 EN**: Continues the surrounding expression or declaration: `const Symbol &parameterSymbol, const DerivedTypeSpec &derivedType) {`.
  **L392 CN**: 继续构造周围的表达式或声明：`const Symbol &parameterSymbol, const DerivedTypeSpec &derivedType) {`。
- **L393 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L393 CN**: 开始 `if` 控制流语句并计算其条件。
- **L394 EN**: Starts a function, method, lambda, or structured scope: `paramValue{derivedType.FindParameter(parameterSymbol.name())}) {`.
  **L394 CN**: 开始一个函数、方法、lambda 或结构化作用域：`paramValue{derivedType.FindParameter(parameterSymbol.name())}) {`。
- **L395 EN**: Returns from the current function with `evaluate::ToInt64(paramValue->GetExplicit())`.
  **L395 CN**: 以 `evaluate::ToInt64(paramValue->GetExplicit())` 从当前函数返回。
- **L396 EN**: Closes the current lexical scope or compound statement.
  **L396 CN**: 结束当前词法作用域或复合语句块。
- **L397 EN**: Returns from the current function with `std::nullopt`.
  **L397 CN**: 以 `std::nullopt` 从当前函数返回。
- **L398 EN**: Closes the current lexical scope or compound statement.
  **L398 CN**: 结束当前词法作用域或复合语句块。
- **L399 EN**: Blank line separating nearby declarations or logic blocks.
  **L399 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L400 EN**: Continues logic associated with callable symbol `HaveCompatibleTypeParameters`.
  **L400 CN**: 继续与可调用符号 `HaveCompatibleTypeParameters` 相关的逻辑。
- **L401 EN**: Continues the surrounding expression or declaration: `const DerivedTypeSpec &derivedType1, const DerivedTypeSpec &derivedType2) {`.
  **L401 CN**: 继续构造周围的表达式或声明：`const DerivedTypeSpec &derivedType1, const DerivedTypeSpec &derivedType2) {`。
- **L402 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L402 CN**: 开始 `for` 控制流语句并计算其条件。
- **L403 EN**: Starts a function, method, lambda, or structured scope: `OrderParameterDeclarations(derivedType1.typeSymbol())) {`.
  **L403 CN**: 开始一个函数、方法、lambda 或结构化作用域：`OrderParameterDeclarations(derivedType1.typeSymbol())) {`。
- **L404 EN**: Executes a call or declaration centered on `v1{GetTypeParameterInt64Value`.
  **L404 CN**: 执行以 `v1{GetTypeParameterInt64Value` 为核心的调用或声明。
- **L405 EN**: Executes a call or declaration centered on `v2{GetTypeParameterInt64Value`.
  **L405 CN**: 执行以 `v2{GetTypeParameterInt64Value` 为核心的调用或声明。
- **L406 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L406 CN**: 开始 `if` 控制流语句并计算其条件。
- **L407 EN**: Returns from the current function with `false`.
  **L407 CN**: 以 `false` 从当前函数返回。
- **L408 EN**: Closes the current lexical scope or compound statement.
  **L408 CN**: 结束当前词法作用域或复合语句块。

### Lines 409-432

````cpp
  }
  return true;
}

static bool HaveCompatibleTypeParameters(
    const DeclTypeSpec &type1, const evaluate::DynamicType &type2) {
  if (type1.category() == DeclTypeSpec::Category::ClassStar) {
    return true;
  }
  if (const IntrinsicTypeSpec * intrinsicType1{type1.AsIntrinsic()}) {
    return evaluate::ToInt64(intrinsicType1->kind()).value() == type2.kind();
  } else if (type2.IsUnlimitedPolymorphic()) {
    return false;
  } else if (const DerivedTypeSpec * derivedType1{type1.AsDerived()}) {
    return HaveCompatibleTypeParameters(
        *derivedType1, type2.GetDerivedTypeSpec());
  } else {
    common::die("unexpected type1 category");
  }
}

static bool HaveCompatibleTypeParameters(
    const DeclTypeSpec &type1, const DeclTypeSpec &type2) {
  if (type1.category() == DeclTypeSpec::Category::ClassStar) {
````
- **L409 EN**: Closes the current lexical scope or compound statement.
  **L409 CN**: 结束当前词法作用域或复合语句块。
- **L410 EN**: Returns from the current function with `true`.
  **L410 CN**: 以 `true` 从当前函数返回。
- **L411 EN**: Closes the current lexical scope or compound statement.
  **L411 CN**: 结束当前词法作用域或复合语句块。
- **L412 EN**: Blank line separating nearby declarations or logic blocks.
  **L412 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L413 EN**: Continues logic associated with callable symbol `HaveCompatibleTypeParameters`.
  **L413 CN**: 继续与可调用符号 `HaveCompatibleTypeParameters` 相关的逻辑。
- **L414 EN**: Continues the surrounding expression or declaration: `const DeclTypeSpec &type1, const evaluate::DynamicType &type2) {`.
  **L414 CN**: 继续构造周围的表达式或声明：`const DeclTypeSpec &type1, const evaluate::DynamicType &type2) {`。
- **L415 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L415 CN**: 开始 `if` 控制流语句并计算其条件。
- **L416 EN**: Returns from the current function with `true`.
  **L416 CN**: 以 `true` 从当前函数返回。
- **L417 EN**: Closes the current lexical scope or compound statement.
  **L417 CN**: 结束当前词法作用域或复合语句块。
- **L418 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L418 CN**: 开始 `if` 控制流语句并计算其条件。
- **L419 EN**: Returns from the current function with `evaluate::ToInt64(intrinsicType1->kind()).value() == type2.kind()`.
  **L419 CN**: 以 `evaluate::ToInt64(intrinsicType1->kind()).value() == type2.kind()` 从当前函数返回。
- **L420 EN**: Transitions from the previous branch into an `else if` condition.
  **L420 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L421 EN**: Returns from the current function with `false`.
  **L421 CN**: 以 `false` 从当前函数返回。
- **L422 EN**: Transitions from the previous branch into an `else if` condition.
  **L422 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L423 EN**: Returns from the current function with `HaveCompatibleTypeParameters(`.
  **L423 CN**: 以 `HaveCompatibleTypeParameters(` 从当前函数返回。
- **L424 EN**: Comment explains nearby logic, intent, or metadata: `derivedType1, type2.GetDerivedTypeSpec());`.
  **L424 CN**: 注释说明附近代码的逻辑、意图或元数据：`derivedType1, type2.GetDerivedTypeSpec());`。
- **L425 EN**: Transitions from the previous branch into the alternative path.
  **L425 CN**: 从前一个分支过渡到备选路径。
- **L426 EN**: Executes a call or declaration centered on `common::die`.
  **L426 CN**: 执行以 `common::die` 为核心的调用或声明。
- **L427 EN**: Closes the current lexical scope or compound statement.
  **L427 CN**: 结束当前词法作用域或复合语句块。
- **L428 EN**: Closes the current lexical scope or compound statement.
  **L428 CN**: 结束当前词法作用域或复合语句块。
- **L429 EN**: Blank line separating nearby declarations or logic blocks.
  **L429 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L430 EN**: Continues logic associated with callable symbol `HaveCompatibleTypeParameters`.
  **L430 CN**: 继续与可调用符号 `HaveCompatibleTypeParameters` 相关的逻辑。
- **L431 EN**: Continues the surrounding expression or declaration: `const DeclTypeSpec &type1, const DeclTypeSpec &type2) {`.
  **L431 CN**: 继续构造周围的表达式或声明：`const DeclTypeSpec &type1, const DeclTypeSpec &type2) {`。
- **L432 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L432 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 433-456

````cpp
    return true;
  } else if (const IntrinsicTypeSpec * intrinsicType1{type1.AsIntrinsic()}) {
    const IntrinsicTypeSpec *intrinsicType2{type2.AsIntrinsic()};
    return !intrinsicType2 || intrinsicType1->kind() == intrinsicType2->kind();
  } else if (const DerivedTypeSpec * derivedType1{type1.AsDerived()}) {
    const DerivedTypeSpec *derivedType2{type2.AsDerived()};
    return !derivedType2 ||
        HaveCompatibleTypeParameters(*derivedType1, *derivedType2);
  } else {
    common::die("unexpected type1 category");
  }
}

static bool HaveCompatibleLengths(
    const DeclTypeSpec &type1, const DeclTypeSpec &type2) {
  if (type1.category() == DeclTypeSpec::Character &&
      type2.category() == DeclTypeSpec::Character) {
    auto v1{
        evaluate::ToInt64(type1.characterTypeSpec().length().GetExplicit())};
    auto v2{
        evaluate::ToInt64(type2.characterTypeSpec().length().GetExplicit())};
    return !v1 || !v2 || (*v1 >= 0 ? *v1 : 0) == (*v2 >= 0 ? *v2 : 0);
  } else {
    return true;
````
- **L433 EN**: Returns from the current function with `true`.
  **L433 CN**: 以 `true` 从当前函数返回。
- **L434 EN**: Transitions from the previous branch into an `else if` condition.
  **L434 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L435 EN**: Executes a call or declaration centered on `*intrinsicType2{type2.AsIntrinsic`.
  **L435 CN**: 执行以 `*intrinsicType2{type2.AsIntrinsic` 为核心的调用或声明。
- **L436 EN**: Returns from the current function with `!intrinsicType2 || intrinsicType1->kind() == intrinsicType2->kind()`.
  **L436 CN**: 以 `!intrinsicType2 || intrinsicType1->kind() == intrinsicType2->kind()` 从当前函数返回。
- **L437 EN**: Transitions from the previous branch into an `else if` condition.
  **L437 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L438 EN**: Executes a call or declaration centered on `*derivedType2{type2.AsDerived`.
  **L438 CN**: 执行以 `*derivedType2{type2.AsDerived` 为核心的调用或声明。
- **L439 EN**: Returns from the current function with `!derivedType2 ||`.
  **L439 CN**: 以 `!derivedType2 ||` 从当前函数返回。
- **L440 EN**: Executes a call or declaration centered on `HaveCompatibleTypeParameters`.
  **L440 CN**: 执行以 `HaveCompatibleTypeParameters` 为核心的调用或声明。
- **L441 EN**: Transitions from the previous branch into the alternative path.
  **L441 CN**: 从前一个分支过渡到备选路径。
- **L442 EN**: Executes a call or declaration centered on `common::die`.
  **L442 CN**: 执行以 `common::die` 为核心的调用或声明。
- **L443 EN**: Closes the current lexical scope or compound statement.
  **L443 CN**: 结束当前词法作用域或复合语句块。
- **L444 EN**: Closes the current lexical scope or compound statement.
  **L444 CN**: 结束当前词法作用域或复合语句块。
- **L445 EN**: Blank line separating nearby declarations or logic blocks.
  **L445 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L446 EN**: Continues logic associated with callable symbol `HaveCompatibleLengths`.
  **L446 CN**: 继续与可调用符号 `HaveCompatibleLengths` 相关的逻辑。
- **L447 EN**: Continues the surrounding expression or declaration: `const DeclTypeSpec &type1, const DeclTypeSpec &type2) {`.
  **L447 CN**: 继续构造周围的表达式或声明：`const DeclTypeSpec &type1, const DeclTypeSpec &type2) {`。
- **L448 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L448 CN**: 开始 `if` 控制流语句并计算其条件。
- **L449 EN**: Starts a function, method, lambda, or structured scope: `type2.category() == DeclTypeSpec::Character) {`.
  **L449 CN**: 开始一个函数、方法、lambda 或结构化作用域：`type2.category() == DeclTypeSpec::Character) {`。
- **L450 EN**: Continues the surrounding expression or declaration: `auto v1{`.
  **L450 CN**: 继续构造周围的表达式或声明：`auto v1{`。
- **L451 EN**: Executes a call or declaration centered on `evaluate::ToInt64`.
  **L451 CN**: 执行以 `evaluate::ToInt64` 为核心的调用或声明。
- **L452 EN**: Continues the surrounding expression or declaration: `auto v2{`.
  **L452 CN**: 继续构造周围的表达式或声明：`auto v2{`。
- **L453 EN**: Executes a call or declaration centered on `evaluate::ToInt64`.
  **L453 CN**: 执行以 `evaluate::ToInt64` 为核心的调用或声明。
- **L454 EN**: Returns from the current function with `!v1 || !v2 || (*v1 >= 0 ? *v1 : 0) == (*v2 >= 0 ? *v2 : 0)`.
  **L454 CN**: 以 `!v1 || !v2 || (*v1 >= 0 ? *v1 : 0) == (*v2 >= 0 ? *v2 : 0)` 从当前函数返回。
- **L455 EN**: Transitions from the previous branch into the alternative path.
  **L455 CN**: 从前一个分支过渡到备选路径。
- **L456 EN**: Returns from the current function with `true`.
  **L456 CN**: 以 `true` 从当前函数返回。

### Lines 457-480

````cpp
  }
}

static bool HaveCompatibleLengths(
    const DeclTypeSpec &type1, const evaluate::DynamicType &type2) {
  if (type1.category() == DeclTypeSpec::Character &&
      type2.category() == TypeCategory::Character) {
    auto v1{
        evaluate::ToInt64(type1.characterTypeSpec().length().GetExplicit())};
    auto v2{type2.knownLength()};
    return !v1 || !v2 || (*v1 >= 0 ? *v1 : 0) == (*v2 >= 0 ? *v2 : 0);
  } else {
    return true;
  }
}

bool AreSameAllocation(const SomeExpr *root, const SomeExpr *path) {
  if (root && path) {
    // For now we just use equality of expressions. If we implement a more
    // sophisticated alias analysis we should use it here.
    return *root == *path;
  } else {
    return false;
  }
````
- **L457 EN**: Closes the current lexical scope or compound statement.
  **L457 CN**: 结束当前词法作用域或复合语句块。
- **L458 EN**: Closes the current lexical scope or compound statement.
  **L458 CN**: 结束当前词法作用域或复合语句块。
- **L459 EN**: Blank line separating nearby declarations or logic blocks.
  **L459 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L460 EN**: Continues logic associated with callable symbol `HaveCompatibleLengths`.
  **L460 CN**: 继续与可调用符号 `HaveCompatibleLengths` 相关的逻辑。
- **L461 EN**: Continues the surrounding expression or declaration: `const DeclTypeSpec &type1, const evaluate::DynamicType &type2) {`.
  **L461 CN**: 继续构造周围的表达式或声明：`const DeclTypeSpec &type1, const evaluate::DynamicType &type2) {`。
- **L462 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L462 CN**: 开始 `if` 控制流语句并计算其条件。
- **L463 EN**: Starts a function, method, lambda, or structured scope: `type2.category() == TypeCategory::Character) {`.
  **L463 CN**: 开始一个函数、方法、lambda 或结构化作用域：`type2.category() == TypeCategory::Character) {`。
- **L464 EN**: Continues the surrounding expression or declaration: `auto v1{`.
  **L464 CN**: 继续构造周围的表达式或声明：`auto v1{`。
- **L465 EN**: Executes a call or declaration centered on `evaluate::ToInt64`.
  **L465 CN**: 执行以 `evaluate::ToInt64` 为核心的调用或声明。
- **L466 EN**: Executes a call or declaration centered on `v2{type2.knownLength`.
  **L466 CN**: 执行以 `v2{type2.knownLength` 为核心的调用或声明。
- **L467 EN**: Returns from the current function with `!v1 || !v2 || (*v1 >= 0 ? *v1 : 0) == (*v2 >= 0 ? *v2 : 0)`.
  **L467 CN**: 以 `!v1 || !v2 || (*v1 >= 0 ? *v1 : 0) == (*v2 >= 0 ? *v2 : 0)` 从当前函数返回。
- **L468 EN**: Transitions from the previous branch into the alternative path.
  **L468 CN**: 从前一个分支过渡到备选路径。
- **L469 EN**: Returns from the current function with `true`.
  **L469 CN**: 以 `true` 从当前函数返回。
- **L470 EN**: Closes the current lexical scope or compound statement.
  **L470 CN**: 结束当前词法作用域或复合语句块。
- **L471 EN**: Closes the current lexical scope or compound statement.
  **L471 CN**: 结束当前词法作用域或复合语句块。
- **L472 EN**: Blank line separating nearby declarations or logic blocks.
  **L472 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L473 EN**: Starts a function, method, lambda, or structured scope: `bool AreSameAllocation(const SomeExpr *root, const SomeExpr *path) {`.
  **L473 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool AreSameAllocation(const SomeExpr *root, const SomeExpr *path) {`。
- **L474 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L474 CN**: 开始 `if` 控制流语句并计算其条件。
- **L475 EN**: Comment explains nearby logic, intent, or metadata: `For now we just use equality of expressions. If we implement a more`.
  **L475 CN**: 注释说明附近代码的逻辑、意图或元数据：`For now we just use equality of expressions. If we implement a more`。
- **L476 EN**: Comment explains nearby logic, intent, or metadata: `sophisticated alias analysis we should use it here.`.
  **L476 CN**: 注释说明附近代码的逻辑、意图或元数据：`sophisticated alias analysis we should use it here.`。
- **L477 EN**: Returns from the current function with `*root == *path`.
  **L477 CN**: 以 `*root == *path` 从当前函数返回。
- **L478 EN**: Transitions from the previous branch into the alternative path.
  **L478 CN**: 从前一个分支过渡到备选路径。
- **L479 EN**: Returns from the current function with `false`.
  **L479 CN**: 以 `false` 从当前函数返回。
- **L480 EN**: Closes the current lexical scope or compound statement.
  **L480 CN**: 结束当前词法作用域或复合语句块。

### Lines 481-504

````cpp
}

bool AllocationCheckerHelper::RunChecks(SemanticsContext &context) {
  if (!ultimate_) {
    CHECK(context.AnyFatalError());
    return false;
  }
  if (!IsVariableName(*ultimate_)) { // C932 pre-requisite
    context.Say(name_.source,
        "Name in ALLOCATE statement must be a variable name"_err_en_US);
    return false;
  }
  if (!type_) {
    // This is done after variable check because a user could have put
    // a subroutine name in allocate for instance which is a symbol with
    // no type.
    CHECK(context.AnyFatalError());
    return false;
  }
  GatherAllocationBasicInfo();
  if (!IsAllocatableOrObjectPointer(ultimate_)) { // C932
    context.Say(name_.source,
        "Entity in ALLOCATE statement must have the ALLOCATABLE or POINTER attribute"_err_en_US);
    return false;
````
- **L481 EN**: Closes the current lexical scope or compound statement.
  **L481 CN**: 结束当前词法作用域或复合语句块。
- **L482 EN**: Blank line separating nearby declarations or logic blocks.
  **L482 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L483 EN**: Starts a function, method, lambda, or structured scope: `bool AllocationCheckerHelper::RunChecks(SemanticsContext &context) {`.
  **L483 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool AllocationCheckerHelper::RunChecks(SemanticsContext &context) {`。
- **L484 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L484 CN**: 开始 `if` 控制流语句并计算其条件。
- **L485 EN**: Executes a call or declaration centered on `CHECK`.
  **L485 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L486 EN**: Returns from the current function with `false`.
  **L486 CN**: 以 `false` 从当前函数返回。
- **L487 EN**: Closes the current lexical scope or compound statement.
  **L487 CN**: 结束当前词法作用域或复合语句块。
- **L488 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L488 CN**: 开始 `if` 控制流语句并计算其条件。
- **L489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L489 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L490 EN**: Executes a standalone statement or declaration: `"Name in ALLOCATE statement must be a variable name"_err_en_US);`.
  **L490 CN**: 执行一条独立语句或声明：`"Name in ALLOCATE statement must be a variable name"_err_en_US);`。
- **L491 EN**: Returns from the current function with `false`.
  **L491 CN**: 以 `false` 从当前函数返回。
- **L492 EN**: Closes the current lexical scope or compound statement.
  **L492 CN**: 结束当前词法作用域或复合语句块。
- **L493 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L493 CN**: 开始 `if` 控制流语句并计算其条件。
- **L494 EN**: Comment explains nearby logic, intent, or metadata: `This is done after variable check because a user could have put`.
  **L494 CN**: 注释说明附近代码的逻辑、意图或元数据：`This is done after variable check because a user could have put`。
- **L495 EN**: Comment explains nearby logic, intent, or metadata: `a subroutine name in allocate for instance which is a symbol with`.
  **L495 CN**: 注释说明附近代码的逻辑、意图或元数据：`a subroutine name in allocate for instance which is a symbol with`。
- **L496 EN**: Comment explains nearby logic, intent, or metadata: `no type.`.
  **L496 CN**: 注释说明附近代码的逻辑、意图或元数据：`no type.`。
- **L497 EN**: Executes a call or declaration centered on `CHECK`.
  **L497 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L498 EN**: Returns from the current function with `false`.
  **L498 CN**: 以 `false` 从当前函数返回。
- **L499 EN**: Closes the current lexical scope or compound statement.
  **L499 CN**: 结束当前词法作用域或复合语句块。
- **L500 EN**: Executes a call or declaration centered on `GatherAllocationBasicInfo`.
  **L500 CN**: 执行以 `GatherAllocationBasicInfo` 为核心的调用或声明。
- **L501 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L501 CN**: 开始 `if` 控制流语句并计算其条件。
- **L502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L502 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L503 EN**: Executes a standalone statement or declaration: `"Entity in ALLOCATE statement must have the ALLOCATABLE or POINTER attribute"_err_en_US);`.
  **L503 CN**: 执行一条独立语句或声明：`"Entity in ALLOCATE statement must have the ALLOCATABLE or POINTER attribute"_err_en_US);`。
- **L504 EN**: Returns from the current function with `false`.
  **L504 CN**: 以 `false` 从当前函数返回。

### Lines 505-528

````cpp
  }
  bool gotSourceExprOrTypeSpec{allocateInfo_.gotMold ||
      allocateInfo_.gotTypeSpec || allocateInfo_.gotSource};
  if (hasDeferredTypeParameter_ && !gotSourceExprOrTypeSpec) {
    // C933
    context.Say(name_.source,
        "Either type-spec or source-expr must appear in ALLOCATE when allocatable object has a deferred type parameters"_err_en_US);
    return false;
  }
  if (isUnlimitedPolymorphic_ && !gotSourceExprOrTypeSpec) {
    // C933
    context.Say(name_.source,
        "Either type-spec or source-expr must appear in ALLOCATE when allocatable object is unlimited polymorphic"_err_en_US);
    return false;
  }
  if (isAbstract_ && !gotSourceExprOrTypeSpec) {
    // C933
    context.Say(name_.source,
        "Either type-spec or source-expr must appear in ALLOCATE when allocatable object is of abstract type"_err_en_US);
    return false;
  }
  if (allocateInfo_.gotTypeSpec) {
    if (!IsTypeCompatible(*type_, *allocateInfo_.typeSpec)) {
      // C934
````
- **L505 EN**: Closes the current lexical scope or compound statement.
  **L505 CN**: 结束当前词法作用域或复合语句块。
- **L506 EN**: Continues the surrounding expression or declaration: `bool gotSourceExprOrTypeSpec{allocateInfo_.gotMold ||`.
  **L506 CN**: 继续构造周围的表达式或声明：`bool gotSourceExprOrTypeSpec{allocateInfo_.gotMold ||`。
- **L507 EN**: Executes a standalone statement or declaration: `allocateInfo_.gotTypeSpec || allocateInfo_.gotSource};`.
  **L507 CN**: 执行一条独立语句或声明：`allocateInfo_.gotTypeSpec || allocateInfo_.gotSource};`。
- **L508 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L508 CN**: 开始 `if` 控制流语句并计算其条件。
- **L509 EN**: Comment explains nearby logic, intent, or metadata: `C933`.
  **L509 CN**: 注释说明附近代码的逻辑、意图或元数据：`C933`。
- **L510 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L510 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L511 EN**: Executes a standalone statement or declaration: `"Either type-spec or source-expr must appear in ALLOCATE when allocatable object has a deferred type parameters"_err_en_US);`.
  **L511 CN**: 执行一条独立语句或声明：`"Either type-spec or source-expr must appear in ALLOCATE when allocatable object has a deferred type parameters"_err_en_US);`。
- **L512 EN**: Returns from the current function with `false`.
  **L512 CN**: 以 `false` 从当前函数返回。
- **L513 EN**: Closes the current lexical scope or compound statement.
  **L513 CN**: 结束当前词法作用域或复合语句块。
- **L514 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L514 CN**: 开始 `if` 控制流语句并计算其条件。
- **L515 EN**: Comment explains nearby logic, intent, or metadata: `C933`.
  **L515 CN**: 注释说明附近代码的逻辑、意图或元数据：`C933`。
- **L516 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L516 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L517 EN**: Executes a standalone statement or declaration: `"Either type-spec or source-expr must appear in ALLOCATE when allocatable object is unlimited polymorphic"_err_en_US);`.
  **L517 CN**: 执行一条独立语句或声明：`"Either type-spec or source-expr must appear in ALLOCATE when allocatable object is unlimited polymorphic"_err_en_US);`。
- **L518 EN**: Returns from the current function with `false`.
  **L518 CN**: 以 `false` 从当前函数返回。
- **L519 EN**: Closes the current lexical scope or compound statement.
  **L519 CN**: 结束当前词法作用域或复合语句块。
- **L520 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L520 CN**: 开始 `if` 控制流语句并计算其条件。
- **L521 EN**: Comment explains nearby logic, intent, or metadata: `C933`.
  **L521 CN**: 注释说明附近代码的逻辑、意图或元数据：`C933`。
- **L522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L522 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L523 EN**: Executes a standalone statement or declaration: `"Either type-spec or source-expr must appear in ALLOCATE when allocatable object is of abstract type"_err_en_US);`.
  **L523 CN**: 执行一条独立语句或声明：`"Either type-spec or source-expr must appear in ALLOCATE when allocatable object is of abstract type"_err_en_US);`。
- **L524 EN**: Returns from the current function with `false`.
  **L524 CN**: 以 `false` 从当前函数返回。
- **L525 EN**: Closes the current lexical scope or compound statement.
  **L525 CN**: 结束当前词法作用域或复合语句块。
- **L526 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L526 CN**: 开始 `if` 控制流语句并计算其条件。
- **L527 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L527 CN**: 开始 `if` 控制流语句并计算其条件。
- **L528 EN**: Comment explains nearby logic, intent, or metadata: `C934`.
  **L528 CN**: 注释说明附近代码的逻辑、意图或元数据：`C934`。

### Lines 529-552

````cpp
      context.Say(name_.source,
          "Allocatable object in ALLOCATE must be type compatible with type-spec"_err_en_US);
      return false;
    }
    if (!HaveCompatibleTypeParameters(*type_, *allocateInfo_.typeSpec)) {
      context.Say(name_.source,
          // C936
          "Type parameters of allocatable object in ALLOCATE must be the same as the corresponding ones in type-spec"_err_en_US);
      return false;
    }
    if (!HaveCompatibleLengths(*type_, *allocateInfo_.typeSpec)) { // C934
      context.Say(name_.source,
          "Character length of allocatable object in ALLOCATE must be the same as the type-spec"_err_en_US);
      return false;
    }
    if (!HaveSameAssumedTypeParameters(*type_, *allocateInfo_.typeSpec)) {
      // C935
      context.Say(name_.source,
          "Type parameters in type-spec must be assumed if and only if they are assumed for allocatable object in ALLOCATE"_err_en_US);
      return false;
    }
  } else if (allocateInfo_.gotSource || allocateInfo_.gotMold) {
    if (!IsTypeCompatible(*type_, allocateInfo_.sourceExprType.value())) {
      // first part of C945
````
- **L529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L529 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L530 EN**: Executes a standalone statement or declaration: `"Allocatable object in ALLOCATE must be type compatible with type-spec"_err_en_US);`.
  **L530 CN**: 执行一条独立语句或声明：`"Allocatable object in ALLOCATE must be type compatible with type-spec"_err_en_US);`。
- **L531 EN**: Returns from the current function with `false`.
  **L531 CN**: 以 `false` 从当前函数返回。
- **L532 EN**: Closes the current lexical scope or compound statement.
  **L532 CN**: 结束当前词法作用域或复合语句块。
- **L533 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L533 CN**: 开始 `if` 控制流语句并计算其条件。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L535 EN**: Comment explains nearby logic, intent, or metadata: `C936`.
  **L535 CN**: 注释说明附近代码的逻辑、意图或元数据：`C936`。
- **L536 EN**: Executes a standalone statement or declaration: `"Type parameters of allocatable object in ALLOCATE must be the same as the corresponding ones in type-spec"_err_en_US);`.
  **L536 CN**: 执行一条独立语句或声明：`"Type parameters of allocatable object in ALLOCATE must be the same as the corresponding ones in type-spec"_err_en_US);`。
- **L537 EN**: Returns from the current function with `false`.
  **L537 CN**: 以 `false` 从当前函数返回。
- **L538 EN**: Closes the current lexical scope or compound statement.
  **L538 CN**: 结束当前词法作用域或复合语句块。
- **L539 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L539 CN**: 开始 `if` 控制流语句并计算其条件。
- **L540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L540 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L541 EN**: Executes a standalone statement or declaration: `"Character length of allocatable object in ALLOCATE must be the same as the type-spec"_err_en_US);`.
  **L541 CN**: 执行一条独立语句或声明：`"Character length of allocatable object in ALLOCATE must be the same as the type-spec"_err_en_US);`。
- **L542 EN**: Returns from the current function with `false`.
  **L542 CN**: 以 `false` 从当前函数返回。
- **L543 EN**: Closes the current lexical scope or compound statement.
  **L543 CN**: 结束当前词法作用域或复合语句块。
- **L544 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L544 CN**: 开始 `if` 控制流语句并计算其条件。
- **L545 EN**: Comment explains nearby logic, intent, or metadata: `C935`.
  **L545 CN**: 注释说明附近代码的逻辑、意图或元数据：`C935`。
- **L546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L546 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L547 EN**: Executes a standalone statement or declaration: `"Type parameters in type-spec must be assumed if and only if they are assumed for allocatable object in ALLOCATE"_err_en_US);`.
  **L547 CN**: 执行一条独立语句或声明：`"Type parameters in type-spec must be assumed if and only if they are assumed for allocatable object in ALLOCATE"_err_en_US);`。
- **L548 EN**: Returns from the current function with `false`.
  **L548 CN**: 以 `false` 从当前函数返回。
- **L549 EN**: Closes the current lexical scope or compound statement.
  **L549 CN**: 结束当前词法作用域或复合语句块。
- **L550 EN**: Transitions from the previous branch into an `else if` condition.
  **L550 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L551 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L551 CN**: 开始 `if` 控制流语句并计算其条件。
- **L552 EN**: Comment explains nearby logic, intent, or metadata: `first part of C945`.
  **L552 CN**: 注释说明附近代码的逻辑、意图或元数据：`first part of C945`。

### Lines 553-576

````cpp
      context.Say(name_.source,
          "Allocatable object in ALLOCATE must be type compatible with source expression from MOLD or SOURCE"_err_en_US);
      return false;
    }
    if (!HaveCompatibleTypeParameters(
            *type_, allocateInfo_.sourceExprType.value())) {
      // C946
      context.Say(name_.source,
          "Derived type parameters of allocatable object must be the same as the corresponding ones of SOURCE or MOLD expression"_err_en_US);
      return false;
    }
    // Character length distinction is allowed, with a warning
    if (!HaveCompatibleLengths(
            *type_, allocateInfo_.sourceExprType.value())) { // F'2023 C950
      context.Warn(common::LanguageFeature::AllocateToOtherLength, name_.source,
          "Character length of allocatable object in ALLOCATE should be the same as the SOURCE or MOLD"_port_en_US);
    }
  }
  // Shape related checks
  if (ultimate_ && IsAssumedRank(*ultimate_)) {
    context.Say(name_.source,
        "An assumed-rank dummy argument may not appear in an ALLOCATE statement"_err_en_US);
    return false;
  }
````
- **L553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L553 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L554 EN**: Executes a standalone statement or declaration: `"Allocatable object in ALLOCATE must be type compatible with source expression from MOLD or SOURCE"_err_en_US);`.
  **L554 CN**: 执行一条独立语句或声明：`"Allocatable object in ALLOCATE must be type compatible with source expression from MOLD or SOURCE"_err_en_US);`。
- **L555 EN**: Returns from the current function with `false`.
  **L555 CN**: 以 `false` 从当前函数返回。
- **L556 EN**: Closes the current lexical scope or compound statement.
  **L556 CN**: 结束当前词法作用域或复合语句块。
- **L557 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L557 CN**: 开始 `if` 控制流语句并计算其条件。
- **L558 EN**: Comment explains nearby logic, intent, or metadata: `type_, allocateInfo_.sourceExprType.value())) {`.
  **L558 CN**: 注释说明附近代码的逻辑、意图或元数据：`type_, allocateInfo_.sourceExprType.value())) {`。
- **L559 EN**: Comment explains nearby logic, intent, or metadata: `C946`.
  **L559 CN**: 注释说明附近代码的逻辑、意图或元数据：`C946`。
- **L560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L560 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L561 EN**: Executes a standalone statement or declaration: `"Derived type parameters of allocatable object must be the same as the corresponding ones of SOURCE or MOLD expression"_err_en_US);`.
  **L561 CN**: 执行一条独立语句或声明：`"Derived type parameters of allocatable object must be the same as the corresponding ones of SOURCE or MOLD expression"_err_en_US);`。
- **L562 EN**: Returns from the current function with `false`.
  **L562 CN**: 以 `false` 从当前函数返回。
- **L563 EN**: Closes the current lexical scope or compound statement.
  **L563 CN**: 结束当前词法作用域或复合语句块。
- **L564 EN**: Comment explains nearby logic, intent, or metadata: `Character length distinction is allowed, with a warning`.
  **L564 CN**: 注释说明附近代码的逻辑、意图或元数据：`Character length distinction is allowed, with a warning`。
- **L565 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L565 CN**: 开始 `if` 控制流语句并计算其条件。
- **L566 EN**: Comment explains nearby logic, intent, or metadata: `type_, allocateInfo_.sourceExprType.value())) { // F'2023 C950`.
  **L566 CN**: 注释说明附近代码的逻辑、意图或元数据：`type_, allocateInfo_.sourceExprType.value())) { // F'2023 C950`。
- **L567 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Warn(common::LanguageFeature::AllocateToOtherLength, name_.source,`.
  **L567 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Warn(common::LanguageFeature::AllocateToOtherLength, name_.source,`。
- **L568 EN**: Executes a standalone statement or declaration: `"Character length of allocatable object in ALLOCATE should be the same as the SOURCE or MOLD"_port_en_US);`.
  **L568 CN**: 执行一条独立语句或声明：`"Character length of allocatable object in ALLOCATE should be the same as the SOURCE or MOLD"_port_en_US);`。
- **L569 EN**: Closes the current lexical scope or compound statement.
  **L569 CN**: 结束当前词法作用域或复合语句块。
- **L570 EN**: Closes the current lexical scope or compound statement.
  **L570 CN**: 结束当前词法作用域或复合语句块。
- **L571 EN**: Comment explains nearby logic, intent, or metadata: `Shape related checks`.
  **L571 CN**: 注释说明附近代码的逻辑、意图或元数据：`Shape related checks`。
- **L572 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L572 CN**: 开始 `if` 控制流语句并计算其条件。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L574 EN**: Executes a standalone statement or declaration: `"An assumed-rank dummy argument may not appear in an ALLOCATE statement"_err_en_US);`.
  **L574 CN**: 执行一条独立语句或声明：`"An assumed-rank dummy argument may not appear in an ALLOCATE statement"_err_en_US);`。
- **L575 EN**: Returns from the current function with `false`.
  **L575 CN**: 以 `false` 从当前函数返回。
- **L576 EN**: Closes the current lexical scope or compound statement.
  **L576 CN**: 结束当前词法作用域或复合语句块。

### Lines 577-600

````cpp
  if (ultimate_ && IsAssumedSizeArray(*ultimate_) && context.AnyFatalError()) {
    // An assumed-size dummy array or RANK(*) case of SELECT RANK will have
    // already been diagnosed; don't pile on.
    return false;
  }
  if (rank_ > 0) {
    if (!hasAllocateShapeSpecList()) {
      // C939
      if (!(allocateInfo_.gotSource || allocateInfo_.gotMold)) {
        context.Say(name_.source,
            "Arrays in ALLOCATE must have a shape specification or an expression of the same rank must appear in SOURCE or MOLD"_err_en_US);
        return false;
      } else {
        if (allocateInfo_.sourceExprRank != rank_) {
          context
              .Say(name_.source,
                  "Arrays in ALLOCATE must have a shape specification or an expression of the same rank must appear in SOURCE or MOLD"_err_en_US)
              .Attach(allocateInfo_.sourceExprLoc.value(),
                  "Expression in %s has rank %d but allocatable object has rank %d"_en_US,
                  allocateInfo_.gotSource ? "SOURCE" : "MOLD",
                  allocateInfo_.sourceExprRank, rank_);
          return false;
        }
      }
````
- **L577 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L577 CN**: 开始 `if` 控制流语句并计算其条件。
- **L578 EN**: Comment explains nearby logic, intent, or metadata: `An assumed-size dummy array or RANK(*) case of SELECT RANK will have`.
  **L578 CN**: 注释说明附近代码的逻辑、意图或元数据：`An assumed-size dummy array or RANK(*) case of SELECT RANK will have`。
- **L579 EN**: Comment explains nearby logic, intent, or metadata: `already been diagnosed; don't pile on.`.
  **L579 CN**: 注释说明附近代码的逻辑、意图或元数据：`already been diagnosed; don't pile on.`。
- **L580 EN**: Returns from the current function with `false`.
  **L580 CN**: 以 `false` 从当前函数返回。
- **L581 EN**: Closes the current lexical scope or compound statement.
  **L581 CN**: 结束当前词法作用域或复合语句块。
- **L582 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L582 CN**: 开始 `if` 控制流语句并计算其条件。
- **L583 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L583 CN**: 开始 `if` 控制流语句并计算其条件。
- **L584 EN**: Comment explains nearby logic, intent, or metadata: `C939`.
  **L584 CN**: 注释说明附近代码的逻辑、意图或元数据：`C939`。
- **L585 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L585 CN**: 开始 `if` 控制流语句并计算其条件。
- **L586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L586 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L587 EN**: Executes a standalone statement or declaration: `"Arrays in ALLOCATE must have a shape specification or an expression of the same rank must appear in SOURCE or MOLD"_err_en_US);`.
  **L587 CN**: 执行一条独立语句或声明：`"Arrays in ALLOCATE must have a shape specification or an expression of the same rank must appear in SOURCE or MOLD"_err_en_US);`。
- **L588 EN**: Returns from the current function with `false`.
  **L588 CN**: 以 `false` 从当前函数返回。
- **L589 EN**: Transitions from the previous branch into the alternative path.
  **L589 CN**: 从前一个分支过渡到备选路径。
- **L590 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L590 CN**: 开始 `if` 控制流语句并计算其条件。
- **L591 EN**: Continues the surrounding expression or declaration: `context`.
  **L591 CN**: 继续构造周围的表达式或声明：`context`。
- **L592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(name_.source,`.
  **L592 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(name_.source,`。
- **L593 EN**: Continues the surrounding expression or declaration: `"Arrays in ALLOCATE must have a shape specification or an expression of the same rank must appear in SOURCE or MOLD"_err_en_US)`.
  **L593 CN**: 继续构造周围的表达式或声明：`"Arrays in ALLOCATE must have a shape specification or an expression of the same rank must appear in SOURCE or MOLD"_err_en_US)`。
- **L594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(allocateInfo_.sourceExprLoc.value(),`.
  **L594 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(allocateInfo_.sourceExprLoc.value(),`。
- **L595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Expression in %s has rank %d but allocatable object has rank %d"_en_US,`.
  **L595 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Expression in %s has rank %d but allocatable object has rank %d"_en_US,`。
- **L596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `allocateInfo_.gotSource ? "SOURCE" : "MOLD",`.
  **L596 CN**: 继续一个多行参数列表、初始化器或聚合项：`allocateInfo_.gotSource ? "SOURCE" : "MOLD",`。
- **L597 EN**: Executes a standalone statement or declaration: `allocateInfo_.sourceExprRank, rank_);`.
  **L597 CN**: 执行一条独立语句或声明：`allocateInfo_.sourceExprRank, rank_);`。
- **L598 EN**: Returns from the current function with `false`.
  **L598 CN**: 以 `false` 从当前函数返回。
- **L599 EN**: Closes the current lexical scope or compound statement.
  **L599 CN**: 结束当前词法作用域或复合语句块。
- **L600 EN**: Closes the current lexical scope or compound statement.
  **L600 CN**: 结束当前词法作用域或复合语句块。

### Lines 601-624

````cpp
    } else {
      // explicit shape-spec-list
      if (allocateShapeSpecRank_ != rank_) {
        context
            .Say(name_.source,
                "The number of shape specifications, when they appear, must match the rank of allocatable object"_err_en_US)
            .Attach(
                ultimate_->name(), "Declared here with rank %d"_en_US, rank_);
        return false;
      } else if (allocateInfo_.gotSource && allocateInfo_.sourceExprShape &&
          allocateInfo_.sourceExprShape->size() ==
              static_cast<std::size_t>(allocateShapeSpecRank_)) {
        std::size_t j{0};
        for (const auto &shapeSpec :
            std::get<std::list<parser::AllocateShapeSpec>>(allocation_.t)) {
          if (j >= allocateInfo_.sourceExprShape->size()) {
            break;
          }
          std::optional<evaluate::ConstantSubscript> lbound;
          if (const auto &lb{std::get<0>(shapeSpec.t)}) {
            lbound.reset();
            const auto &lbExpr{parser::UnwrapRef<parser::Expr>(lb)};
            if (const auto *expr{GetExpr(context, lbExpr)}) {
              auto folded{
````
- **L601 EN**: Transitions from the previous branch into the alternative path.
  **L601 CN**: 从前一个分支过渡到备选路径。
- **L602 EN**: Comment explains nearby logic, intent, or metadata: `explicit shape-spec-list`.
  **L602 CN**: 注释说明附近代码的逻辑、意图或元数据：`explicit shape-spec-list`。
- **L603 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L603 CN**: 开始 `if` 控制流语句并计算其条件。
- **L604 EN**: Continues the surrounding expression or declaration: `context`.
  **L604 CN**: 继续构造周围的表达式或声明：`context`。
- **L605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(name_.source,`.
  **L605 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(name_.source,`。
- **L606 EN**: Continues the surrounding expression or declaration: `"The number of shape specifications, when they appear, must match the rank of allocatable object"_err_en_US)`.
  **L606 CN**: 继续构造周围的表达式或声明：`"The number of shape specifications, when they appear, must match the rank of allocatable object"_err_en_US)`。
- **L607 EN**: Continues logic associated with callable symbol `Attach`.
  **L607 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L608 EN**: Executes a call or declaration centered on `ultimate_->name`.
  **L608 CN**: 执行以 `ultimate_->name` 为核心的调用或声明。
- **L609 EN**: Returns from the current function with `false`.
  **L609 CN**: 以 `false` 从当前函数返回。
- **L610 EN**: Transitions from the previous branch into an `else if` condition.
  **L610 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L611 EN**: Continues logic associated with callable symbol `size`.
  **L611 CN**: 继续与可调用符号 `size` 相关的逻辑。
- **L612 EN**: Starts a function, method, lambda, or structured scope: `static_cast<std::size_t>(allocateShapeSpecRank_)) {`.
  **L612 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static_cast<std::size_t>(allocateShapeSpecRank_)) {`。
- **L613 EN**: Executes a standalone statement or declaration: `std::size_t j{0};`.
  **L613 CN**: 执行一条独立语句或声明：`std::size_t j{0};`。
- **L614 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L614 CN**: 开始 `for` 控制流语句并计算其条件。
- **L615 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::list<parser::AllocateShapeSpec>>(allocation_.t)) {`.
  **L615 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::list<parser::AllocateShapeSpec>>(allocation_.t)) {`。
- **L616 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L616 CN**: 开始 `if` 控制流语句并计算其条件。
- **L617 EN**: Exits the nearest loop or switch statement.
  **L617 CN**: 退出最近的循环或 switch 语句。
- **L618 EN**: Closes the current lexical scope or compound statement.
  **L618 CN**: 结束当前词法作用域或复合语句块。
- **L619 EN**: Executes a standalone statement or declaration: `std::optional<evaluate::ConstantSubscript> lbound;`.
  **L619 CN**: 执行一条独立语句或声明：`std::optional<evaluate::ConstantSubscript> lbound;`。
- **L620 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L620 CN**: 开始 `if` 控制流语句并计算其条件。
- **L621 EN**: Executes a call or declaration centered on `lbound.reset`.
  **L621 CN**: 执行以 `lbound.reset` 为核心的调用或声明。
- **L622 EN**: Executes a call or declaration centered on `&lbExpr{parser::UnwrapRef<parser::Expr>`.
  **L622 CN**: 执行以 `&lbExpr{parser::UnwrapRef<parser::Expr>` 为核心的调用或声明。
- **L623 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L623 CN**: 开始 `if` 控制流语句并计算其条件。
- **L624 EN**: Continues the surrounding expression or declaration: `auto folded{`.
  **L624 CN**: 继续构造周围的表达式或声明：`auto folded{`。

### Lines 625-648

````cpp
                  evaluate::Fold(context.foldingContext(), SomeExpr(*expr))};
              lbound = evaluate::ToInt64(folded);
              evaluate::SetExpr(lbExpr, std::move(folded));
            }
          } else {
            lbound = 1;
          }
          if (lbound) {
            const auto &ubExpr{
                parser::UnwrapRef<parser::Expr>(std::get<1>(shapeSpec.t))};
            if (const auto *expr{GetExpr(context, ubExpr)}) {
              auto folded{
                  evaluate::Fold(context.foldingContext(), SomeExpr(*expr))};
              auto ubound{evaluate::ToInt64(folded)};
              evaluate::SetExpr(ubExpr, std::move(folded));
              if (ubound) {
                auto extent{*ubound - *lbound + 1};
                if (extent < 0) {
                  extent = 0;
                }
                if (extent != allocateInfo_.sourceExprShape->at(j)) {
                  context.Say(name_.source,
                      "Allocation has extent %jd on dimension %d, but SOURCE= has extent %jd"_err_en_US,
                      static_cast<std::intmax_t>(extent), j + 1,
````
- **L625 EN**: Executes a call or declaration centered on `evaluate::Fold`.
  **L625 CN**: 执行以 `evaluate::Fold` 为核心的调用或声明。
- **L626 EN**: Executes a call or declaration centered on `evaluate::ToInt64`.
  **L626 CN**: 执行以 `evaluate::ToInt64` 为核心的调用或声明。
- **L627 EN**: Executes a call or declaration centered on `evaluate::SetExpr`.
  **L627 CN**: 执行以 `evaluate::SetExpr` 为核心的调用或声明。
- **L628 EN**: Closes the current lexical scope or compound statement.
  **L628 CN**: 结束当前词法作用域或复合语句块。
- **L629 EN**: Transitions from the previous branch into the alternative path.
  **L629 CN**: 从前一个分支过渡到备选路径。
- **L630 EN**: Executes a standalone statement or declaration: `lbound = 1;`.
  **L630 CN**: 执行一条独立语句或声明：`lbound = 1;`。
- **L631 EN**: Closes the current lexical scope or compound statement.
  **L631 CN**: 结束当前词法作用域或复合语句块。
- **L632 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L632 CN**: 开始 `if` 控制流语句并计算其条件。
- **L633 EN**: Continues the surrounding expression or declaration: `const auto &ubExpr{`.
  **L633 CN**: 继续构造周围的表达式或声明：`const auto &ubExpr{`。
- **L634 EN**: Executes a call or declaration centered on `parser::UnwrapRef<parser::Expr>`.
  **L634 CN**: 执行以 `parser::UnwrapRef<parser::Expr>` 为核心的调用或声明。
- **L635 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L635 CN**: 开始 `if` 控制流语句并计算其条件。
- **L636 EN**: Continues the surrounding expression or declaration: `auto folded{`.
  **L636 CN**: 继续构造周围的表达式或声明：`auto folded{`。
- **L637 EN**: Executes a call or declaration centered on `evaluate::Fold`.
  **L637 CN**: 执行以 `evaluate::Fold` 为核心的调用或声明。
- **L638 EN**: Executes a call or declaration centered on `ubound{evaluate::ToInt64`.
  **L638 CN**: 执行以 `ubound{evaluate::ToInt64` 为核心的调用或声明。
- **L639 EN**: Executes a call or declaration centered on `evaluate::SetExpr`.
  **L639 CN**: 执行以 `evaluate::SetExpr` 为核心的调用或声明。
- **L640 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L640 CN**: 开始 `if` 控制流语句并计算其条件。
- **L641 EN**: Executes a standalone statement or declaration: `auto extent{*ubound - *lbound + 1};`.
  **L641 CN**: 执行一条独立语句或声明：`auto extent{*ubound - *lbound + 1};`。
- **L642 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L642 CN**: 开始 `if` 控制流语句并计算其条件。
- **L643 EN**: Executes a standalone statement or declaration: `extent = 0;`.
  **L643 CN**: 执行一条独立语句或声明：`extent = 0;`。
- **L644 EN**: Closes the current lexical scope or compound statement.
  **L644 CN**: 结束当前词法作用域或复合语句块。
- **L645 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L645 CN**: 开始 `if` 控制流语句并计算其条件。
- **L646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L646 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Allocation has extent %jd on dimension %d, but SOURCE= has extent %jd"_err_en_US,`.
  **L647 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Allocation has extent %jd on dimension %d, but SOURCE= has extent %jd"_err_en_US,`。
- **L648 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<std::intmax_t>(extent), j + 1,`.
  **L648 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<std::intmax_t>(extent), j + 1,`。

### Lines 649-672

````cpp
                      static_cast<std::intmax_t>(
                          allocateInfo_.sourceExprShape->at(j)));
                }
              }
            }
          }
          ++j;
        }
      }
    }
  } else { // allocating a scalar object
    if (hasAllocateShapeSpecList()) {
      context.Say(name_.source,
          "Shape specifications must not appear when allocatable object is scalar"_err_en_US);
      return false;
    }
  }
  // second and last part of C945
  if (allocateInfo_.gotSource && allocateInfo_.sourceExprRank &&
      allocateInfo_.sourceExprRank != rank_) {
    context
        .Say(name_.source,
            "If SOURCE appears, the related expression must be scalar or have the same rank as each allocatable object in ALLOCATE"_err_en_US)
        .Attach(allocateInfo_.sourceExprLoc.value(),
````
- **L649 EN**: Continues logic associated with callable symbol `intmax_t>`.
  **L649 CN**: 继续与可调用符号 `intmax_t>` 相关的逻辑。
- **L650 EN**: Executes a call or declaration centered on `allocateInfo_.sourceExprShape->at`.
  **L650 CN**: 执行以 `allocateInfo_.sourceExprShape->at` 为核心的调用或声明。
- **L651 EN**: Closes the current lexical scope or compound statement.
  **L651 CN**: 结束当前词法作用域或复合语句块。
- **L652 EN**: Closes the current lexical scope or compound statement.
  **L652 CN**: 结束当前词法作用域或复合语句块。
- **L653 EN**: Closes the current lexical scope or compound statement.
  **L653 CN**: 结束当前词法作用域或复合语句块。
- **L654 EN**: Closes the current lexical scope or compound statement.
  **L654 CN**: 结束当前词法作用域或复合语句块。
- **L655 EN**: Executes a standalone statement or declaration: `++j;`.
  **L655 CN**: 执行一条独立语句或声明：`++j;`。
- **L656 EN**: Closes the current lexical scope or compound statement.
  **L656 CN**: 结束当前词法作用域或复合语句块。
- **L657 EN**: Closes the current lexical scope or compound statement.
  **L657 CN**: 结束当前词法作用域或复合语句块。
- **L658 EN**: Closes the current lexical scope or compound statement.
  **L658 CN**: 结束当前词法作用域或复合语句块。
- **L659 EN**: Transitions from the previous branch into the alternative path.
  **L659 CN**: 从前一个分支过渡到备选路径。
- **L660 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L660 CN**: 开始 `if` 控制流语句并计算其条件。
- **L661 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L661 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L662 EN**: Executes a standalone statement or declaration: `"Shape specifications must not appear when allocatable object is scalar"_err_en_US);`.
  **L662 CN**: 执行一条独立语句或声明：`"Shape specifications must not appear when allocatable object is scalar"_err_en_US);`。
- **L663 EN**: Returns from the current function with `false`.
  **L663 CN**: 以 `false` 从当前函数返回。
- **L664 EN**: Closes the current lexical scope or compound statement.
  **L664 CN**: 结束当前词法作用域或复合语句块。
- **L665 EN**: Closes the current lexical scope or compound statement.
  **L665 CN**: 结束当前词法作用域或复合语句块。
- **L666 EN**: Comment explains nearby logic, intent, or metadata: `second and last part of C945`.
  **L666 CN**: 注释说明附近代码的逻辑、意图或元数据：`second and last part of C945`。
- **L667 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L667 CN**: 开始 `if` 控制流语句并计算其条件。
- **L668 EN**: Continues the surrounding expression or declaration: `allocateInfo_.sourceExprRank != rank_) {`.
  **L668 CN**: 继续构造周围的表达式或声明：`allocateInfo_.sourceExprRank != rank_) {`。
- **L669 EN**: Continues the surrounding expression or declaration: `context`.
  **L669 CN**: 继续构造周围的表达式或声明：`context`。
- **L670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(name_.source,`.
  **L670 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(name_.source,`。
- **L671 EN**: Continues the surrounding expression or declaration: `"If SOURCE appears, the related expression must be scalar or have the same rank as each allocatable object in ALLOCATE"_err_en_US)`.
  **L671 CN**: 继续构造周围的表达式或声明：`"If SOURCE appears, the related expression must be scalar or have the same rank as each allocatable object in ALLOCATE"_err_en_US)`。
- **L672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(allocateInfo_.sourceExprLoc.value(),`.
  **L672 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(allocateInfo_.sourceExprLoc.value(),`。

### Lines 673-696

````cpp
            "SOURCE expression has rank %d"_en_US, allocateInfo_.sourceExprRank)
        .Attach(ultimate_->name(),
            "Allocatable object declared here with rank %d"_en_US, rank_);
    return false;
  }
  context.CheckIndexVarRedefine(name_);
  const Scope &subpScope{
      GetProgramUnitContaining(context.FindScope(name_.source))};
  if (allocateObject_.typedExpr && allocateObject_.typedExpr->v) {
    DefinabilityFlags flags{DefinabilityFlag::PointerDefinition,
        DefinabilityFlag::AcceptAllocatable};
    if (allocateInfo_.gotSource) {
      flags.set(DefinabilityFlag::SourcedAllocation);
    }
    if (auto whyNot{WhyNotDefinable(
            name_.source, subpScope, flags, *allocateObject_.typedExpr->v)}) {
      context
          .Say(name_.source,
              "Name in ALLOCATE statement is not definable"_err_en_US)
          .Attach(std::move(whyNot->set_severity(parser::Severity::Because)));
      return false;
    }
  }
  if (allocateInfo_.gotPinned) {
````
- **L673 EN**: Continues the surrounding expression or declaration: `"SOURCE expression has rank %d"_en_US, allocateInfo_.sourceExprRank)`.
  **L673 CN**: 继续构造周围的表达式或声明：`"SOURCE expression has rank %d"_en_US, allocateInfo_.sourceExprRank)`。
- **L674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(ultimate_->name(),`.
  **L674 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(ultimate_->name(),`。
- **L675 EN**: Executes a standalone statement or declaration: `"Allocatable object declared here with rank %d"_en_US, rank_);`.
  **L675 CN**: 执行一条独立语句或声明：`"Allocatable object declared here with rank %d"_en_US, rank_);`。
- **L676 EN**: Returns from the current function with `false`.
  **L676 CN**: 以 `false` 从当前函数返回。
- **L677 EN**: Closes the current lexical scope or compound statement.
  **L677 CN**: 结束当前词法作用域或复合语句块。
- **L678 EN**: Executes a call or declaration centered on `context.CheckIndexVarRedefine`.
  **L678 CN**: 执行以 `context.CheckIndexVarRedefine` 为核心的调用或声明。
- **L679 EN**: Continues the surrounding expression or declaration: `const Scope &subpScope{`.
  **L679 CN**: 继续构造周围的表达式或声明：`const Scope &subpScope{`。
- **L680 EN**: Executes a call or declaration centered on `GetProgramUnitContaining`.
  **L680 CN**: 执行以 `GetProgramUnitContaining` 为核心的调用或声明。
- **L681 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L681 CN**: 开始 `if` 控制流语句并计算其条件。
- **L682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefinabilityFlags flags{DefinabilityFlag::PointerDefinition,`.
  **L682 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefinabilityFlags flags{DefinabilityFlag::PointerDefinition,`。
- **L683 EN**: Executes a standalone statement or declaration: `DefinabilityFlag::AcceptAllocatable};`.
  **L683 CN**: 执行一条独立语句或声明：`DefinabilityFlag::AcceptAllocatable};`。
- **L684 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L684 CN**: 开始 `if` 控制流语句并计算其条件。
- **L685 EN**: Executes a call or declaration centered on `flags.set`.
  **L685 CN**: 执行以 `flags.set` 为核心的调用或声明。
- **L686 EN**: Closes the current lexical scope or compound statement.
  **L686 CN**: 结束当前词法作用域或复合语句块。
- **L687 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L687 CN**: 开始 `if` 控制流语句并计算其条件。
- **L688 EN**: Continues the surrounding expression or declaration: `name_.source, subpScope, flags, *allocateObject_.typedExpr->v)}) {`.
  **L688 CN**: 继续构造周围的表达式或声明：`name_.source, subpScope, flags, *allocateObject_.typedExpr->v)}) {`。
- **L689 EN**: Continues the surrounding expression or declaration: `context`.
  **L689 CN**: 继续构造周围的表达式或声明：`context`。
- **L690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(name_.source,`.
  **L690 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(name_.source,`。
- **L691 EN**: Continues the surrounding expression or declaration: `"Name in ALLOCATE statement is not definable"_err_en_US)`.
  **L691 CN**: 继续构造周围的表达式或声明：`"Name in ALLOCATE statement is not definable"_err_en_US)`。
- **L692 EN**: Executes a call or declaration centered on `.Attach`.
  **L692 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L693 EN**: Returns from the current function with `false`.
  **L693 CN**: 以 `false` 从当前函数返回。
- **L694 EN**: Closes the current lexical scope or compound statement.
  **L694 CN**: 结束当前词法作用域或复合语句块。
- **L695 EN**: Closes the current lexical scope or compound statement.
  **L695 CN**: 结束当前词法作用域或复合语句块。
- **L696 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L696 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 697-720

````cpp
    std::optional<common::CUDADataAttr> cudaAttr{GetCUDADataAttr(ultimate_)};
    if ((!cudaAttr || *cudaAttr != common::CUDADataAttr::Pinned) &&
        context.languageFeatures().ShouldWarn(
            common::UsageWarning::CUDAUsage)) {
      context.Say(name_.source,
          "Object in ALLOCATE should have PINNED attribute when PINNED option is specified"_warn_en_US);
    }
  }
  if (allocateInfo_.gotStream) {
    std::optional<common::CUDADataAttr> cudaAttr{GetCUDADataAttr(ultimate_)};
    if (!cudaAttr || *cudaAttr != common::CUDADataAttr::Device) {
      context.Say(name_.source,
          "Object in ALLOCATE must have DEVICE attribute when STREAM option is specified"_err_en_US);
    }
  }

  if (const SomeExpr *allocObj{GetExpr(context, allocateObject_)}) {
    if (AreSameAllocation(allocObj, allocateInfo_.statVar)) {
      context.Say(allocateInfo_.statSource.value_or(name_.source),
          "STAT variable in ALLOCATE must not be the variable being allocated"_err_en_US);
    }
    if (AreSameAllocation(allocObj, allocateInfo_.msgVar)) {
      context.Say(allocateInfo_.msgSource.value_or(name_.source),
          "ERRMSG variable in ALLOCATE must not be the variable being allocated"_err_en_US);
````
- **L697 EN**: Executes a call or declaration centered on `cudaAttr{GetCUDADataAttr`.
  **L697 CN**: 执行以 `cudaAttr{GetCUDADataAttr` 为核心的调用或声明。
- **L698 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L698 CN**: 开始 `if` 控制流语句并计算其条件。
- **L699 EN**: Continues logic associated with callable symbol `languageFeatures`.
  **L699 CN**: 继续与可调用符号 `languageFeatures` 相关的逻辑。
- **L700 EN**: Continues the surrounding expression or declaration: `common::UsageWarning::CUDAUsage)) {`.
  **L700 CN**: 继续构造周围的表达式或声明：`common::UsageWarning::CUDAUsage)) {`。
- **L701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L701 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L702 EN**: Executes a standalone statement or declaration: `"Object in ALLOCATE should have PINNED attribute when PINNED option is specified"_warn_en_US);`.
  **L702 CN**: 执行一条独立语句或声明：`"Object in ALLOCATE should have PINNED attribute when PINNED option is specified"_warn_en_US);`。
- **L703 EN**: Closes the current lexical scope or compound statement.
  **L703 CN**: 结束当前词法作用域或复合语句块。
- **L704 EN**: Closes the current lexical scope or compound statement.
  **L704 CN**: 结束当前词法作用域或复合语句块。
- **L705 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L705 CN**: 开始 `if` 控制流语句并计算其条件。
- **L706 EN**: Executes a call or declaration centered on `cudaAttr{GetCUDADataAttr`.
  **L706 CN**: 执行以 `cudaAttr{GetCUDADataAttr` 为核心的调用或声明。
- **L707 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L707 CN**: 开始 `if` 控制流语句并计算其条件。
- **L708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L708 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L709 EN**: Executes a standalone statement or declaration: `"Object in ALLOCATE must have DEVICE attribute when STREAM option is specified"_err_en_US);`.
  **L709 CN**: 执行一条独立语句或声明：`"Object in ALLOCATE must have DEVICE attribute when STREAM option is specified"_err_en_US);`。
- **L710 EN**: Closes the current lexical scope or compound statement.
  **L710 CN**: 结束当前词法作用域或复合语句块。
- **L711 EN**: Closes the current lexical scope or compound statement.
  **L711 CN**: 结束当前词法作用域或复合语句块。
- **L712 EN**: Blank line separating nearby declarations or logic blocks.
  **L712 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L713 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L713 CN**: 开始 `if` 控制流语句并计算其条件。
- **L714 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L714 CN**: 开始 `if` 控制流语句并计算其条件。
- **L715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(allocateInfo_.statSource.value_or(name_.source),`.
  **L715 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(allocateInfo_.statSource.value_or(name_.source),`。
- **L716 EN**: Executes a standalone statement or declaration: `"STAT variable in ALLOCATE must not be the variable being allocated"_err_en_US);`.
  **L716 CN**: 执行一条独立语句或声明：`"STAT variable in ALLOCATE must not be the variable being allocated"_err_en_US);`。
- **L717 EN**: Closes the current lexical scope or compound statement.
  **L717 CN**: 结束当前词法作用域或复合语句块。
- **L718 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L718 CN**: 开始 `if` 控制流语句并计算其条件。
- **L719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(allocateInfo_.msgSource.value_or(name_.source),`.
  **L719 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(allocateInfo_.msgSource.value_or(name_.source),`。
- **L720 EN**: Executes a standalone statement or declaration: `"ERRMSG variable in ALLOCATE must not be the variable being allocated"_err_en_US);`.
  **L720 CN**: 执行一条独立语句或声明：`"ERRMSG variable in ALLOCATE must not be the variable being allocated"_err_en_US);`。

### Lines 721-744

````cpp
    }
  }
  return RunCoarrayRelatedChecks(context);
}

bool AllocationCheckerHelper::RunCoarrayRelatedChecks(
    SemanticsContext &context) const {
  if (!ultimate_) {
    CHECK(context.AnyFatalError());
    return false;
  }
  if (evaluate::IsCoarray(*ultimate_)) {
    if (allocateInfo_.gotTypeSpec) {
      // C938
      if (const DerivedTypeSpec *
          derived{allocateInfo_.typeSpec->AsDerived()}) {
        if (IsTeamType(derived)) {
          context
              .Say(allocateInfo_.typeSpecLoc.value(),
                  "Type-Spec in ALLOCATE must not be TEAM_TYPE from ISO_FORTRAN_ENV when an allocatable object is a coarray"_err_en_US)
              .Attach(name_.source, "'%s' is a coarray"_en_US, name_.source);
          return false;
        } else if (IsIsoCType(derived)) {
          context
````
- **L721 EN**: Closes the current lexical scope or compound statement.
  **L721 CN**: 结束当前词法作用域或复合语句块。
- **L722 EN**: Closes the current lexical scope or compound statement.
  **L722 CN**: 结束当前词法作用域或复合语句块。
- **L723 EN**: Returns from the current function with `RunCoarrayRelatedChecks(context)`.
  **L723 CN**: 以 `RunCoarrayRelatedChecks(context)` 从当前函数返回。
- **L724 EN**: Closes the current lexical scope or compound statement.
  **L724 CN**: 结束当前词法作用域或复合语句块。
- **L725 EN**: Blank line separating nearby declarations or logic blocks.
  **L725 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L726 EN**: Continues logic associated with callable symbol `RunCoarrayRelatedChecks`.
  **L726 CN**: 继续与可调用符号 `RunCoarrayRelatedChecks` 相关的逻辑。
- **L727 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context) const {`.
  **L727 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context) const {`。
- **L728 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L728 CN**: 开始 `if` 控制流语句并计算其条件。
- **L729 EN**: Executes a call or declaration centered on `CHECK`.
  **L729 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L730 EN**: Returns from the current function with `false`.
  **L730 CN**: 以 `false` 从当前函数返回。
- **L731 EN**: Closes the current lexical scope or compound statement.
  **L731 CN**: 结束当前词法作用域或复合语句块。
- **L732 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L732 CN**: 开始 `if` 控制流语句并计算其条件。
- **L733 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L733 CN**: 开始 `if` 控制流语句并计算其条件。
- **L734 EN**: Comment explains nearby logic, intent, or metadata: `C938`.
  **L734 CN**: 注释说明附近代码的逻辑、意图或元数据：`C938`。
- **L735 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L735 CN**: 开始 `if` 控制流语句并计算其条件。
- **L736 EN**: Starts a function, method, lambda, or structured scope: `derived{allocateInfo_.typeSpec->AsDerived()}) {`.
  **L736 CN**: 开始一个函数、方法、lambda 或结构化作用域：`derived{allocateInfo_.typeSpec->AsDerived()}) {`。
- **L737 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L737 CN**: 开始 `if` 控制流语句并计算其条件。
- **L738 EN**: Continues the surrounding expression or declaration: `context`.
  **L738 CN**: 继续构造周围的表达式或声明：`context`。
- **L739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(allocateInfo_.typeSpecLoc.value(),`.
  **L739 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(allocateInfo_.typeSpecLoc.value(),`。
- **L740 EN**: Continues the surrounding expression or declaration: `"Type-Spec in ALLOCATE must not be TEAM_TYPE from ISO_FORTRAN_ENV when an allocatable object is a coarray"_err_en_US)`.
  **L740 CN**: 继续构造周围的表达式或声明：`"Type-Spec in ALLOCATE must not be TEAM_TYPE from ISO_FORTRAN_ENV when an allocatable object is a coarray"_err_en_US)`。
- **L741 EN**: Executes a call or declaration centered on `.Attach`.
  **L741 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L742 EN**: Returns from the current function with `false`.
  **L742 CN**: 以 `false` 从当前函数返回。
- **L743 EN**: Transitions from the previous branch into an `else if` condition.
  **L743 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L744 EN**: Continues the surrounding expression or declaration: `context`.
  **L744 CN**: 继续构造周围的表达式或声明：`context`。

### Lines 745-768

````cpp
              .Say(allocateInfo_.typeSpecLoc.value(),
                  "Type-Spec in ALLOCATE must not be C_PTR or C_FUNPTR from ISO_C_BINDING when an allocatable object is a coarray"_err_en_US)
              .Attach(name_.source, "'%s' is a coarray"_en_US, name_.source);
          return false;
        }
      }
    } else if (allocateInfo_.gotSource || allocateInfo_.gotMold) {
      // C948
      const evaluate::DynamicType &sourceType{
          allocateInfo_.sourceExprType.value()};
      if (const auto *derived{evaluate::GetDerivedTypeSpec(sourceType)}) {
        if (IsTeamType(derived)) {
          context
              .Say(allocateInfo_.sourceExprLoc.value(),
                  "SOURCE or MOLD expression type must not be TEAM_TYPE from ISO_FORTRAN_ENV when an allocatable object is a coarray"_err_en_US)
              .Attach(name_.source, "'%s' is a coarray"_en_US, name_.source);
          return false;
        } else if (IsIsoCType(derived)) {
          context
              .Say(allocateInfo_.sourceExprLoc.value(),
                  "SOURCE or MOLD expression type must not be C_PTR or C_FUNPTR from ISO_C_BINDING when an allocatable object is a coarray"_err_en_US)
              .Attach(name_.source, "'%s' is a coarray"_en_US, name_.source);
          return false;
        }
````
- **L745 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(allocateInfo_.typeSpecLoc.value(),`.
  **L745 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(allocateInfo_.typeSpecLoc.value(),`。
- **L746 EN**: Continues the surrounding expression or declaration: `"Type-Spec in ALLOCATE must not be C_PTR or C_FUNPTR from ISO_C_BINDING when an allocatable object is a coarray"_err_en_US)`.
  **L746 CN**: 继续构造周围的表达式或声明：`"Type-Spec in ALLOCATE must not be C_PTR or C_FUNPTR from ISO_C_BINDING when an allocatable object is a coarray"_err_en_US)`。
- **L747 EN**: Executes a call or declaration centered on `.Attach`.
  **L747 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L748 EN**: Returns from the current function with `false`.
  **L748 CN**: 以 `false` 从当前函数返回。
- **L749 EN**: Closes the current lexical scope or compound statement.
  **L749 CN**: 结束当前词法作用域或复合语句块。
- **L750 EN**: Closes the current lexical scope or compound statement.
  **L750 CN**: 结束当前词法作用域或复合语句块。
- **L751 EN**: Transitions from the previous branch into an `else if` condition.
  **L751 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L752 EN**: Comment explains nearby logic, intent, or metadata: `C948`.
  **L752 CN**: 注释说明附近代码的逻辑、意图或元数据：`C948`。
- **L753 EN**: Continues the surrounding expression or declaration: `const evaluate::DynamicType &sourceType{`.
  **L753 CN**: 继续构造周围的表达式或声明：`const evaluate::DynamicType &sourceType{`。
- **L754 EN**: Executes a call or declaration centered on `allocateInfo_.sourceExprType.value`.
  **L754 CN**: 执行以 `allocateInfo_.sourceExprType.value` 为核心的调用或声明。
- **L755 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L755 CN**: 开始 `if` 控制流语句并计算其条件。
- **L756 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L756 CN**: 开始 `if` 控制流语句并计算其条件。
- **L757 EN**: Continues the surrounding expression or declaration: `context`.
  **L757 CN**: 继续构造周围的表达式或声明：`context`。
- **L758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(allocateInfo_.sourceExprLoc.value(),`.
  **L758 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(allocateInfo_.sourceExprLoc.value(),`。
- **L759 EN**: Continues the surrounding expression or declaration: `"SOURCE or MOLD expression type must not be TEAM_TYPE from ISO_FORTRAN_ENV when an allocatable object is a coarray"_err_en_US)`.
  **L759 CN**: 继续构造周围的表达式或声明：`"SOURCE or MOLD expression type must not be TEAM_TYPE from ISO_FORTRAN_ENV when an allocatable object is a coarray"_err_en_US)`。
- **L760 EN**: Executes a call or declaration centered on `.Attach`.
  **L760 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L761 EN**: Returns from the current function with `false`.
  **L761 CN**: 以 `false` 从当前函数返回。
- **L762 EN**: Transitions from the previous branch into an `else if` condition.
  **L762 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L763 EN**: Continues the surrounding expression or declaration: `context`.
  **L763 CN**: 继续构造周围的表达式或声明：`context`。
- **L764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(allocateInfo_.sourceExprLoc.value(),`.
  **L764 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(allocateInfo_.sourceExprLoc.value(),`。
- **L765 EN**: Continues the surrounding expression or declaration: `"SOURCE or MOLD expression type must not be C_PTR or C_FUNPTR from ISO_C_BINDING when an allocatable object is a coarray"_err_en_US)`.
  **L765 CN**: 继续构造周围的表达式或声明：`"SOURCE or MOLD expression type must not be C_PTR or C_FUNPTR from ISO_C_BINDING when an allocatable object is a coarray"_err_en_US)`。
- **L766 EN**: Executes a call or declaration centered on `.Attach`.
  **L766 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L767 EN**: Returns from the current function with `false`.
  **L767 CN**: 以 `false` 从当前函数返回。
- **L768 EN**: Closes the current lexical scope or compound statement.
  **L768 CN**: 结束当前词法作用域或复合语句块。

### Lines 769-792

````cpp
      }
    }
    if (!hasAllocateCoarraySpec()) {
      // C941
      context.Say(name_.source,
          "Coarray specification must appear in ALLOCATE when allocatable object is a coarray"_err_en_US);
      return false;
    } else {
      if (allocateCoarraySpecRank_ != corank_) {
        // Second and last part of C942
        context
            .Say(name_.source,
                "Corank of coarray specification in ALLOCATE must match corank of alloctable coarray"_err_en_US)
            .Attach(ultimate_->name(), "Declared here with corank %d"_en_US,
                corank_);
        return false;
      }
      if (const auto &coarraySpec{
              std::get<std::optional<parser::AllocateCoarraySpec>>(
                  allocation_.t)}) {
        int dim{0};
        for (const auto &spec :
            std::get<std::list<parser::AllocateCoshapeSpec>>(coarraySpec->t)) {
          if (auto ubv{evaluate::ToInt64(
````
- **L769 EN**: Closes the current lexical scope or compound statement.
  **L769 CN**: 结束当前词法作用域或复合语句块。
- **L770 EN**: Closes the current lexical scope or compound statement.
  **L770 CN**: 结束当前词法作用域或复合语句块。
- **L771 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L771 CN**: 开始 `if` 控制流语句并计算其条件。
- **L772 EN**: Comment explains nearby logic, intent, or metadata: `C941`.
  **L772 CN**: 注释说明附近代码的逻辑、意图或元数据：`C941`。
- **L773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L773 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L774 EN**: Executes a standalone statement or declaration: `"Coarray specification must appear in ALLOCATE when allocatable object is a coarray"_err_en_US);`.
  **L774 CN**: 执行一条独立语句或声明：`"Coarray specification must appear in ALLOCATE when allocatable object is a coarray"_err_en_US);`。
- **L775 EN**: Returns from the current function with `false`.
  **L775 CN**: 以 `false` 从当前函数返回。
- **L776 EN**: Transitions from the previous branch into the alternative path.
  **L776 CN**: 从前一个分支过渡到备选路径。
- **L777 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L777 CN**: 开始 `if` 控制流语句并计算其条件。
- **L778 EN**: Comment explains nearby logic, intent, or metadata: `Second and last part of C942`.
  **L778 CN**: 注释说明附近代码的逻辑、意图或元数据：`Second and last part of C942`。
- **L779 EN**: Continues the surrounding expression or declaration: `context`.
  **L779 CN**: 继续构造周围的表达式或声明：`context`。
- **L780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(name_.source,`.
  **L780 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(name_.source,`。
- **L781 EN**: Continues the surrounding expression or declaration: `"Corank of coarray specification in ALLOCATE must match corank of alloctable coarray"_err_en_US)`.
  **L781 CN**: 继续构造周围的表达式或声明：`"Corank of coarray specification in ALLOCATE must match corank of alloctable coarray"_err_en_US)`。
- **L782 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(ultimate_->name(), "Declared here with corank %d"_en_US,`.
  **L782 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(ultimate_->name(), "Declared here with corank %d"_en_US,`。
- **L783 EN**: Executes a standalone statement or declaration: `corank_);`.
  **L783 CN**: 执行一条独立语句或声明：`corank_);`。
- **L784 EN**: Returns from the current function with `false`.
  **L784 CN**: 以 `false` 从当前函数返回。
- **L785 EN**: Closes the current lexical scope or compound statement.
  **L785 CN**: 结束当前词法作用域或复合语句块。
- **L786 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L786 CN**: 开始 `if` 控制流语句并计算其条件。
- **L787 EN**: Continues logic associated with callable symbol `AllocateCoarraySpec>>`.
  **L787 CN**: 继续与可调用符号 `AllocateCoarraySpec>>` 相关的逻辑。
- **L788 EN**: Continues the surrounding expression or declaration: `allocation_.t)}) {`.
  **L788 CN**: 继续构造周围的表达式或声明：`allocation_.t)}) {`。
- **L789 EN**: Executes a standalone statement or declaration: `int dim{0};`.
  **L789 CN**: 执行一条独立语句或声明：`int dim{0};`。
- **L790 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L790 CN**: 开始 `for` 控制流语句并计算其条件。
- **L791 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::list<parser::AllocateCoshapeSpec>>(coarraySpec->t)) {`.
  **L791 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::list<parser::AllocateCoshapeSpec>>(coarraySpec->t)) {`。
- **L792 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L792 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 793-816

````cpp
                  GetExpr(context, std::get<parser::BoundExpr>(spec.t)))}) {
            if (auto *lbx{GetExpr(context,
                    std::get<std::optional<parser::BoundExpr>>(spec.t))}) {
              auto lbv{evaluate::ToInt64(*lbx)};
              if (lbv && *ubv < *lbv) {
                context.Say(name_.source,
                    "Upper cobound %jd is less than lower cobound %jd of codimension %d"_err_en_US,
                    std::intmax_t{*ubv}, std::intmax_t{*lbv}, dim + 1);
              }
            } else if (*ubv < 1) {
              context.Say(name_.source,
                  "Upper cobound %jd of codimension %d is less than 1"_err_en_US,
                  std::intmax_t{*ubv}, dim + 1);
            }
          }
          ++dim;
        }
      }
    }
  } else { // Not a coarray
    if (hasAllocateCoarraySpec()) {
      // C941
      context.Say(name_.source,
          "Coarray specification must not appear in ALLOCATE when allocatable object is not a coarray"_err_en_US);
````
- **L793 EN**: Starts a function, method, lambda, or structured scope: `GetExpr(context, std::get<parser::BoundExpr>(spec.t)))}) {`.
  **L793 CN**: 开始一个函数、方法、lambda 或结构化作用域：`GetExpr(context, std::get<parser::BoundExpr>(spec.t)))}) {`。
- **L794 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L794 CN**: 开始 `if` 控制流语句并计算其条件。
- **L795 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::optional<parser::BoundExpr>>(spec.t))}) {`.
  **L795 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::optional<parser::BoundExpr>>(spec.t))}) {`。
- **L796 EN**: Executes a call or declaration centered on `lbv{evaluate::ToInt64`.
  **L796 CN**: 执行以 `lbv{evaluate::ToInt64` 为核心的调用或声明。
- **L797 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L797 CN**: 开始 `if` 控制流语句并计算其条件。
- **L798 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L798 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L799 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Upper cobound %jd is less than lower cobound %jd of codimension %d"_err_en_US,`.
  **L799 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Upper cobound %jd is less than lower cobound %jd of codimension %d"_err_en_US,`。
- **L800 EN**: Executes a standalone statement or declaration: `std::intmax_t{*ubv}, std::intmax_t{*lbv}, dim + 1);`.
  **L800 CN**: 执行一条独立语句或声明：`std::intmax_t{*ubv}, std::intmax_t{*lbv}, dim + 1);`。
- **L801 EN**: Closes the current lexical scope or compound statement.
  **L801 CN**: 结束当前词法作用域或复合语句块。
- **L802 EN**: Transitions from the previous branch into an `else if` condition.
  **L802 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L803 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Upper cobound %jd of codimension %d is less than 1"_err_en_US,`.
  **L804 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Upper cobound %jd of codimension %d is less than 1"_err_en_US,`。
- **L805 EN**: Executes a standalone statement or declaration: `std::intmax_t{*ubv}, dim + 1);`.
  **L805 CN**: 执行一条独立语句或声明：`std::intmax_t{*ubv}, dim + 1);`。
- **L806 EN**: Closes the current lexical scope or compound statement.
  **L806 CN**: 结束当前词法作用域或复合语句块。
- **L807 EN**: Closes the current lexical scope or compound statement.
  **L807 CN**: 结束当前词法作用域或复合语句块。
- **L808 EN**: Executes a standalone statement or declaration: `++dim;`.
  **L808 CN**: 执行一条独立语句或声明：`++dim;`。
- **L809 EN**: Closes the current lexical scope or compound statement.
  **L809 CN**: 结束当前词法作用域或复合语句块。
- **L810 EN**: Closes the current lexical scope or compound statement.
  **L810 CN**: 结束当前词法作用域或复合语句块。
- **L811 EN**: Closes the current lexical scope or compound statement.
  **L811 CN**: 结束当前词法作用域或复合语句块。
- **L812 EN**: Transitions from the previous branch into the alternative path.
  **L812 CN**: 从前一个分支过渡到备选路径。
- **L813 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L813 CN**: 开始 `if` 控制流语句并计算其条件。
- **L814 EN**: Comment explains nearby logic, intent, or metadata: `C941`.
  **L814 CN**: 注释说明附近代码的逻辑、意图或元数据：`C941`。
- **L815 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(name_.source,`.
  **L815 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(name_.source,`。
- **L816 EN**: Executes a standalone statement or declaration: `"Coarray specification must not appear in ALLOCATE when allocatable object is not a coarray"_err_en_US);`.
  **L816 CN**: 执行一条独立语句或声明：`"Coarray specification must not appear in ALLOCATE when allocatable object is not a coarray"_err_en_US);`。

### Lines 817-838

````cpp
      return false;
    }
  }
  if (const parser::CoindexedNamedObject *
      coindexedObject{parser::GetCoindexedNamedObject(allocateObject_)}) {
    // C950
    context.Say(parser::FindSourceLocation(*coindexedObject),
        "Allocatable object must not be coindexed in ALLOCATE"_err_en_US);
    return false;
  }
  return true;
}

void AllocateChecker::Leave(const parser::AllocateStmt &allocateStmt) {
  if (auto info{CheckAllocateOptions(allocateStmt, context_)}) {
    for (const parser::Allocation &allocation :
        std::get<std::list<parser::Allocation>>(allocateStmt.t)) {
      AllocationCheckerHelper{allocation, *info}.RunChecks(context_);
    }
  }
}
} // namespace Fortran::semantics
````
- **L817 EN**: Returns from the current function with `false`.
  **L817 CN**: 以 `false` 从当前函数返回。
- **L818 EN**: Closes the current lexical scope or compound statement.
  **L818 CN**: 结束当前词法作用域或复合语句块。
- **L819 EN**: Closes the current lexical scope or compound statement.
  **L819 CN**: 结束当前词法作用域或复合语句块。
- **L820 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L820 CN**: 开始 `if` 控制流语句并计算其条件。
- **L821 EN**: Starts a function, method, lambda, or structured scope: `coindexedObject{parser::GetCoindexedNamedObject(allocateObject_)}) {`.
  **L821 CN**: 开始一个函数、方法、lambda 或结构化作用域：`coindexedObject{parser::GetCoindexedNamedObject(allocateObject_)}) {`。
- **L822 EN**: Comment explains nearby logic, intent, or metadata: `C950`.
  **L822 CN**: 注释说明附近代码的逻辑、意图或元数据：`C950`。
- **L823 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(parser::FindSourceLocation(*coindexedObject),`.
  **L823 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(parser::FindSourceLocation(*coindexedObject),`。
- **L824 EN**: Executes a standalone statement or declaration: `"Allocatable object must not be coindexed in ALLOCATE"_err_en_US);`.
  **L824 CN**: 执行一条独立语句或声明：`"Allocatable object must not be coindexed in ALLOCATE"_err_en_US);`。
- **L825 EN**: Returns from the current function with `false`.
  **L825 CN**: 以 `false` 从当前函数返回。
- **L826 EN**: Closes the current lexical scope or compound statement.
  **L826 CN**: 结束当前词法作用域或复合语句块。
- **L827 EN**: Returns from the current function with `true`.
  **L827 CN**: 以 `true` 从当前函数返回。
- **L828 EN**: Closes the current lexical scope or compound statement.
  **L828 CN**: 结束当前词法作用域或复合语句块。
- **L829 EN**: Blank line separating nearby declarations or logic blocks.
  **L829 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L830 EN**: Starts a function, method, lambda, or structured scope: `void AllocateChecker::Leave(const parser::AllocateStmt &allocateStmt) {`.
  **L830 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AllocateChecker::Leave(const parser::AllocateStmt &allocateStmt) {`。
- **L831 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L831 CN**: 开始 `if` 控制流语句并计算其条件。
- **L832 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L832 CN**: 开始 `for` 控制流语句并计算其条件。
- **L833 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::list<parser::Allocation>>(allocateStmt.t)) {`.
  **L833 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::list<parser::Allocation>>(allocateStmt.t)) {`。
- **L834 EN**: Executes a call or declaration centered on `*info}.RunChecks`.
  **L834 CN**: 执行以 `*info}.RunChecks` 为核心的调用或声明。
- **L835 EN**: Closes the current lexical scope or compound statement.
  **L835 CN**: 结束当前词法作用域或复合语句块。
- **L836 EN**: Closes the current lexical scope or compound statement.
  **L836 CN**: 结束当前词法作用域或复合语句块。
- **L837 EN**: Closes the current lexical scope or compound statement.
  **L837 CN**: 结束当前词法作用域或复合语句块。
- **L838 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::semantics`.
  **L838 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::semantics`。

## Key Concepts / 关键概念

- **Fortran semantic checking / Fortran 语义检查**
- **Fortran parse tree handling / Fortran 语法树处理**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Semantic-context management / 语义上下文管理**
- **Scope and symbol resolution / 作用域与符号解析**
- **Symbol modeling and lookup / 符号建模与查找**
- **Fortran type-spec reasoning / Fortran 类型说明推理**
- **Compile-time evaluation helpers / 编译期求值辅助**
- **Constant folding / 常量折叠**
- **CUDA-specific lowering or runtime handling / CUDA 专用 lowering 或运行时处理**

## Dependencies / 依赖关系

- `check-allocate.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `assignment.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `definable.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Evaluate/fold.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/shape.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/type.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Parser/parse-tree.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Parser/tools.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Semantics/attr.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/expression.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/type.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
