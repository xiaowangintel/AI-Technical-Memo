# IterationGraphSorter.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SparseTensor/Transforms/Utils/IterationGraphSorter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements sparse tensor optimization, lowering, and rewrite passes.
- **Purpose (CN)**: 实现稀疏张量优化、lowering 与重写 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- IterationGraphSorter.cpp -------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "IterationGraphSorter.h"

#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/SparseTensor/IR/SparseTensor.h"
#include "mlir/Dialect/Utils/StructuredOpsUtils.h"
#include "mlir/IR/AffineExprVisitor.h"
#include "mlir/IR/BuiltinTypes.h"

using namespace mlir;
using namespace mlir::sparse_tensor;

namespace {
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L9 EN**: Includes "IterationGraphSorter.h" to access local declarations paired with this implementation unit.
  **L9 CN**: 引入 "IterationGraphSorter.h" 以使用与该实现单元配套的本地声明。
- **L10 EN**: Blank line separating nearby declarations or logic blocks.
  **L10 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L11 EN**: Includes "mlir/Dialect/Linalg/IR/Linalg.h" to access dialect-specific IR, transforms, or shared utilities.
  **L11 CN**: 引入 "mlir/Dialect/Linalg/IR/Linalg.h" 以使用方言专用 IR、变换或共享工具。
- **L12 EN**: Includes "mlir/Dialect/SparseTensor/IR/SparseTensor.h" to access dialect-specific IR, transforms, or shared utilities.
  **L12 CN**: 引入 "mlir/Dialect/SparseTensor/IR/SparseTensor.h" 以使用方言专用 IR、变换或共享工具。
- **L13 EN**: Includes "mlir/Dialect/Utils/StructuredOpsUtils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L13 CN**: 引入 "mlir/Dialect/Utils/StructuredOpsUtils.h" 以使用方言专用 IR、变换或共享工具。
- **L14 EN**: Includes "mlir/IR/AffineExprVisitor.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L14 CN**: 引入 "mlir/IR/AffineExprVisitor.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L15 EN**: Includes "mlir/IR/BuiltinTypes.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L15 CN**: 引入 "mlir/IR/BuiltinTypes.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L17 EN**: Brings namespace `mlir` into local scope.
  **L17 CN**: 将命名空间 `mlir` 引入当前作用域。
- **L18 EN**: Brings namespace `mlir::sparse_tensor` into local scope.
  **L18 CN**: 将命名空间 `mlir::sparse_tensor` 引入当前作用域。
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L20 EN**: Opens namespace scope ``.
  **L20 CN**: 打开命名空间作用域 ``。

### Lines 21-40

````cpp

/// A helper class that visits an affine expression and tries to find
/// an AffineDimExpr to which the corresponding iterator from a GenericOp
/// matches the desired iterator type. If there is no matched iterator
/// type, the method returns the first DimExpr in the expression.
class AffineDimFinder : public AffineExprVisitor<AffineDimFinder> {
public:
  explicit AffineDimFinder(ArrayRef<utils::IteratorType> itTypes)
      : iterTypes(itTypes) {}

  /// Overrides the visit method from AffineExprVisitor.
  void visitDimExpr(AffineDimExpr expr) {
    if (pickedDim == nullptr || pickIterType == iterTypes[expr.getPosition()])
      pickedDim = expr;
  }

  /// Sets the desired iterator type that we want to pick.
  void setPickedIterType(utils::IteratorType iterType) {
    pickIterType = iterType;
  }
````
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L22 EN**: Comment explains nearby logic, invariants, or intent: `A helper class that visits an affine expression and tries to find`.
  **L22 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A helper class that visits an affine expression and tries to find`。
- **L23 EN**: Comment explains nearby logic, invariants, or intent: `an AffineDimExpr to which the corresponding iterator from a GenericOp`.
  **L23 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`an AffineDimExpr to which the corresponding iterator from a GenericOp`。
- **L24 EN**: Comment explains nearby logic, invariants, or intent: `matches the desired iterator type. If there is no matched iterator`.
  **L24 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`matches the desired iterator type. If there is no matched iterator`。
- **L25 EN**: Comment explains nearby logic, invariants, or intent: `type, the method returns the first DimExpr in the expression.`.
  **L25 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`type, the method returns the first DimExpr in the expression.`。
- **L26 EN**: Declares class `AffineDimFinder`.
  **L26 CN**: 声明 class `AffineDimFinder`。
- **L27 EN**: Sets the following members to `public` access.
  **L27 CN**: 将后续成员的访问级别设为 `public`。
- **L28 EN**: Continues logic associated with callable symbol `AffineDimFinder`.
  **L28 CN**: 继续与可调用符号 `AffineDimFinder` 相关的逻辑。
- **L29 EN**: Continues logic associated with callable symbol `iterTypes`.
  **L29 CN**: 继续与可调用符号 `iterTypes` 相关的逻辑。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L31 EN**: Comment explains nearby logic, invariants, or intent: `Overrides the visit method from AffineExprVisitor.`.
  **L31 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Overrides the visit method from AffineExprVisitor.`。
- **L32 EN**: Starts a function, method, lambda, or structured scope: `void visitDimExpr(AffineDimExpr expr) {`.
  **L32 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void visitDimExpr(AffineDimExpr expr) {`。
- **L33 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L33 CN**: 开始 `if` 控制流语句并计算其条件。
- **L34 EN**: Executes a standalone statement or declaration: `pickedDim = expr;`.
  **L34 CN**: 执行一条独立语句或声明：`pickedDim = expr;`。
- **L35 EN**: Closes the current lexical scope or compound statement.
  **L35 CN**: 结束当前词法作用域或复合语句块。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L37 EN**: Comment explains nearby logic, invariants, or intent: `Sets the desired iterator type that we want to pick.`.
  **L37 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sets the desired iterator type that we want to pick.`。
- **L38 EN**: Starts a function, method, lambda, or structured scope: `void setPickedIterType(utils::IteratorType iterType) {`.
  **L38 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void setPickedIterType(utils::IteratorType iterType) {`。
- **L39 EN**: Executes a standalone statement or declaration: `pickIterType = iterType;`.
  **L39 CN**: 执行一条独立语句或声明：`pickIterType = iterType;`。
- **L40 EN**: Closes the current lexical scope or compound statement.
  **L40 CN**: 结束当前词法作用域或复合语句块。

### Lines 41-60

````cpp

  /// Gets the desired AffineDimExpr.
  AffineDimExpr getDimExpr() const {
    return llvm::cast<AffineDimExpr>(pickedDim);
  }

  /// Walks the graph in post order to find dim expr.
  void walkPostOrder(AffineExpr expr) {
    pickedDim = nullptr;
    AffineExprVisitor<AffineDimFinder>::walkPostOrder(expr);
  }

private:
  /// The picked AffineDimExpr after visit.
  AffineExpr pickedDim;
  /// The iterator type that we want.
  utils::IteratorType pickIterType;
  /// The mapping between levels and iterator types.
  ArrayRef<utils::IteratorType> iterTypes;
};
````
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L42 EN**: Comment explains nearby logic, invariants, or intent: `Gets the desired AffineDimExpr.`.
  **L42 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Gets the desired AffineDimExpr.`。
- **L43 EN**: Starts a function, method, lambda, or structured scope: `AffineDimExpr getDimExpr() const {`.
  **L43 CN**: 开始一个函数、方法、lambda 或结构化作用域：`AffineDimExpr getDimExpr() const {`。
- **L44 EN**: Returns from the current function with `llvm::cast<AffineDimExpr>(pickedDim)`.
  **L44 CN**: 以 `llvm::cast<AffineDimExpr>(pickedDim)` 从当前函数返回。
- **L45 EN**: Closes the current lexical scope or compound statement.
  **L45 CN**: 结束当前词法作用域或复合语句块。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L47 EN**: Comment explains nearby logic, invariants, or intent: `Walks the graph in post order to find dim expr.`.
  **L47 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Walks the graph in post order to find dim expr.`。
- **L48 EN**: Starts a function, method, lambda, or structured scope: `void walkPostOrder(AffineExpr expr) {`.
  **L48 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void walkPostOrder(AffineExpr expr) {`。
- **L49 EN**: Executes a standalone statement or declaration: `pickedDim = nullptr;`.
  **L49 CN**: 执行一条独立语句或声明：`pickedDim = nullptr;`。
- **L50 EN**: Executes a call or declaration centered on `AffineExprVisitor<AffineDimFinder>::walkPostOrder`.
  **L50 CN**: 执行以 `AffineExprVisitor<AffineDimFinder>::walkPostOrder` 为核心的调用或声明。
- **L51 EN**: Closes the current lexical scope or compound statement.
  **L51 CN**: 结束当前词法作用域或复合语句块。
- **L52 EN**: Blank line separating nearby declarations or logic blocks.
  **L52 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L53 EN**: Sets the following members to `private` access.
  **L53 CN**: 将后续成员的访问级别设为 `private`。
- **L54 EN**: Comment explains nearby logic, invariants, or intent: `The picked AffineDimExpr after visit.`.
  **L54 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The picked AffineDimExpr after visit.`。
- **L55 EN**: Executes a standalone statement or declaration: `AffineExpr pickedDim;`.
  **L55 CN**: 执行一条独立语句或声明：`AffineExpr pickedDim;`。
- **L56 EN**: Comment explains nearby logic, invariants, or intent: `The iterator type that we want.`.
  **L56 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The iterator type that we want.`。
- **L57 EN**: Executes a standalone statement or declaration: `utils::IteratorType pickIterType;`.
  **L57 CN**: 执行一条独立语句或声明：`utils::IteratorType pickIterType;`。
- **L58 EN**: Comment explains nearby logic, invariants, or intent: `The mapping between levels and iterator types.`.
  **L58 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The mapping between levels and iterator types.`。
- **L59 EN**: Executes a standalone statement or declaration: `ArrayRef<utils::IteratorType> iterTypes;`.
  **L59 CN**: 执行一条独立语句或声明：`ArrayRef<utils::IteratorType> iterTypes;`。
- **L60 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L60 CN**: 结束当前声明作用域，例如类、结构体或枚举。

### Lines 61-80

````cpp

/// Flattens an affine expression into a list of AffineDimExprs.
struct AffineDimCollector : public AffineExprVisitor<AffineDimCollector> {
  // Overrides method from AffineExprVisitor.
  void visitDimExpr(AffineDimExpr expr) { dims.push_back(expr); }
  SmallVector<AffineDimExpr> dims;
};

} // namespace

inline static bool includesAny(SortMask mask1, SortMask mask2) {
  return static_cast<unsigned>(mask1) & static_cast<unsigned>(mask2);
}

inline static bool includesDenseInput(SortMask mask) {
  return includesAny(mask, SortMask::kIncludeDenseInput);
}

inline static bool includesDenseOutput(SortMask mask) {
  return includesAny(mask, SortMask::kIncludeDenseOutput);
````
- **L61 EN**: Blank line separating nearby declarations or logic blocks.
  **L61 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L62 EN**: Comment explains nearby logic, invariants, or intent: `Flattens an affine expression into a list of AffineDimExprs.`.
  **L62 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Flattens an affine expression into a list of AffineDimExprs.`。
- **L63 EN**: Declares struct `AffineDimCollector`.
  **L63 CN**: 声明 struct `AffineDimCollector`。
- **L64 EN**: Comment explains nearby logic, invariants, or intent: `Overrides method from AffineExprVisitor.`.
  **L64 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Overrides method from AffineExprVisitor.`。
- **L65 EN**: Continues logic associated with callable symbol `visitDimExpr`.
  **L65 CN**: 继续与可调用符号 `visitDimExpr` 相关的逻辑。
- **L66 EN**: Executes a standalone statement or declaration: `SmallVector<AffineDimExpr> dims;`.
  **L66 CN**: 执行一条独立语句或声明：`SmallVector<AffineDimExpr> dims;`。
- **L67 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L67 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L69 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace`.
  **L69 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L70 EN**: Blank line separating nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L71 EN**: Starts a function, method, lambda, or structured scope: `inline static bool includesAny(SortMask mask1, SortMask mask2) {`.
  **L71 CN**: 开始一个函数、方法、lambda 或结构化作用域：`inline static bool includesAny(SortMask mask1, SortMask mask2) {`。
- **L72 EN**: Returns from the current function with `static_cast<unsigned>(mask1) & static_cast<unsigned>(mask2)`.
  **L72 CN**: 以 `static_cast<unsigned>(mask1) & static_cast<unsigned>(mask2)` 从当前函数返回。
- **L73 EN**: Closes the current lexical scope or compound statement.
  **L73 CN**: 结束当前词法作用域或复合语句块。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L75 EN**: Starts a function, method, lambda, or structured scope: `inline static bool includesDenseInput(SortMask mask) {`.
  **L75 CN**: 开始一个函数、方法、lambda 或结构化作用域：`inline static bool includesDenseInput(SortMask mask) {`。
- **L76 EN**: Returns from the current function with `includesAny(mask, SortMask::kIncludeDenseInput)`.
  **L76 CN**: 以 `includesAny(mask, SortMask::kIncludeDenseInput)` 从当前函数返回。
- **L77 EN**: Closes the current lexical scope or compound statement.
  **L77 CN**: 结束当前词法作用域或复合语句块。
- **L78 EN**: Blank line separating nearby declarations or logic blocks.
  **L78 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L79 EN**: Starts a function, method, lambda, or structured scope: `inline static bool includesDenseOutput(SortMask mask) {`.
  **L79 CN**: 开始一个函数、方法、lambda 或结构化作用域：`inline static bool includesDenseOutput(SortMask mask) {`。
- **L80 EN**: Returns from the current function with `includesAny(mask, SortMask::kIncludeDenseOutput)`.
  **L80 CN**: 以 `includesAny(mask, SortMask::kIncludeDenseOutput)` 从当前函数返回。

### Lines 81-100

````cpp
}

/// Returns a sparsity rank for loop ordering: lower values indicate
/// dimensions that should be placed in outer loops.
/// 0 = Dense, 1 = Compressed, 2 = Singleton, 3 = Other/Unknown.
static unsigned getLoopSparsityRank(unsigned loop, ArrayRef<Value> allTensors,
                                    ArrayRef<AffineMap> allMaps) {
  // Start with highest rank.
  unsigned minRank = 3;

  for (auto [tensor, map] : llvm::zip(allTensors, allMaps)) {
    // Check if this loop accesses this tensor.
    bool loopAccessesTensor = false;
    unsigned tensorDim = 0;
    for (AffineExpr expr : map.getResults()) {
      if (auto dimExpr = dyn_cast<AffineDimExpr>(expr)) {
        if (dimExpr.getPosition() == loop) {
          loopAccessesTensor = true;
          break;
        }
````
- **L81 EN**: Closes the current lexical scope or compound statement.
  **L81 CN**: 结束当前词法作用域或复合语句块。
- **L82 EN**: Blank line separating nearby declarations or logic blocks.
  **L82 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L83 EN**: Comment explains nearby logic, invariants, or intent: `Returns a sparsity rank for loop ordering: lower values indicate`.
  **L83 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns a sparsity rank for loop ordering: lower values indicate`。
- **L84 EN**: Comment explains nearby logic, invariants, or intent: `dimensions that should be placed in outer loops.`.
  **L84 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`dimensions that should be placed in outer loops.`。
- **L85 EN**: Comment explains nearby logic, invariants, or intent: `0 = Dense, 1 = Compressed, 2 = Singleton, 3 = Other/Unknown.`.
  **L85 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`0 = Dense, 1 = Compressed, 2 = Singleton, 3 = Other/Unknown.`。
- **L86 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static unsigned getLoopSparsityRank(unsigned loop, ArrayRef<Value> allTensors,`.
  **L86 CN**: 继续一个多行参数列表、初始化器或聚合项：`static unsigned getLoopSparsityRank(unsigned loop, ArrayRef<Value> allTensors,`。
- **L87 EN**: Continues the surrounding expression or declaration: `ArrayRef<AffineMap> allMaps) {`.
  **L87 CN**: 继续构造周围的表达式或声明：`ArrayRef<AffineMap> allMaps) {`。
- **L88 EN**: Comment explains nearby logic, invariants, or intent: `Start with highest rank.`.
  **L88 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Start with highest rank.`。
- **L89 EN**: Initializes variable `minRank` from the right-hand expression.
  **L89 CN**: 使用右侧表达式初始化变量 `minRank`。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L91 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L91 CN**: 开始 `for` 控制流语句并计算其条件。
- **L92 EN**: Comment explains nearby logic, invariants, or intent: `Check if this loop accesses this tensor.`.
  **L92 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check if this loop accesses this tensor.`。
- **L93 EN**: Initializes variable `loopAccessesTensor` from the right-hand expression.
  **L93 CN**: 使用右侧表达式初始化变量 `loopAccessesTensor`。
- **L94 EN**: Initializes variable `tensorDim` from the right-hand expression.
  **L94 CN**: 使用右侧表达式初始化变量 `tensorDim`。
- **L95 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L95 CN**: 开始 `for` 控制流语句并计算其条件。
- **L96 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L96 CN**: 开始 `if` 控制流语句并计算其条件。
- **L97 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L97 CN**: 开始 `if` 控制流语句并计算其条件。
- **L98 EN**: Executes a standalone statement or declaration: `loopAccessesTensor = true;`.
  **L98 CN**: 执行一条独立语句或声明：`loopAccessesTensor = true;`。
- **L99 EN**: Exits the nearest loop or switch statement.
  **L99 CN**: 退出最近的循环或 switch 语句。
- **L100 EN**: Closes the current lexical scope or compound statement.
  **L100 CN**: 结束当前词法作用域或复合语句块。

### Lines 101-120

````cpp
      }
      tensorDim++;
    }

    if (loopAccessesTensor) {
      const auto enc = getSparseTensorEncoding(tensor.getType());
      if (!enc) {
        // Dense tensor - lowest rank.
        return 0;
      } else {
        // Sparse tensor - check the level type for this dimension.
        auto lvlTypes = enc.getLvlTypes();
        if (tensorDim < lvlTypes.size()) {
          auto lvlType = lvlTypes[tensorDim];
          if (isDenseLT(lvlType)) {
            return 0; // Dense level.
          } else if (isCompressedLT(lvlType)) {
            minRank = std::min(minRank, 1u); // Compressed level.
          } else if (isSingletonLT(lvlType)) {
            minRank = std::min(minRank, 2u); // Singleton level.
````
- **L101 EN**: Closes the current lexical scope or compound statement.
  **L101 CN**: 结束当前词法作用域或复合语句块。
- **L102 EN**: Executes a standalone statement or declaration: `tensorDim++;`.
  **L102 CN**: 执行一条独立语句或声明：`tensorDim++;`。
- **L103 EN**: Closes the current lexical scope or compound statement.
  **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Blank line separating nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L105 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L105 CN**: 开始 `if` 控制流语句并计算其条件。
- **L106 EN**: Initializes variable `enc` from the right-hand expression.
  **L106 CN**: 使用右侧表达式初始化变量 `enc`。
- **L107 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L107 CN**: 开始 `if` 控制流语句并计算其条件。
- **L108 EN**: Comment explains nearby logic, invariants, or intent: `Dense tensor - lowest rank.`.
  **L108 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Dense tensor - lowest rank.`。
- **L109 EN**: Returns from the current function with `0`.
  **L109 CN**: 以 `0` 从当前函数返回。
- **L110 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L110 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L111 EN**: Comment explains nearby logic, invariants, or intent: `Sparse tensor - check the level type for this dimension.`.
  **L111 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sparse tensor - check the level type for this dimension.`。
- **L112 EN**: Initializes variable `lvlTypes` from the right-hand expression.
  **L112 CN**: 使用右侧表达式初始化变量 `lvlTypes`。
- **L113 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L113 CN**: 开始 `if` 控制流语句并计算其条件。
- **L114 EN**: Initializes variable `lvlType` from the right-hand expression.
  **L114 CN**: 使用右侧表达式初始化变量 `lvlType`。
- **L115 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L115 CN**: 开始 `if` 控制流语句并计算其条件。
- **L116 EN**: Returns from the current function with `0; // Dense level.`.
  **L116 CN**: 以 `0; // Dense level.` 从当前函数返回。
- **L117 EN**: Starts a function, method, lambda, or structured scope: `} else if (isCompressedLT(lvlType)) {`.
  **L117 CN**: 开始一个函数、方法、lambda 或结构化作用域：`} else if (isCompressedLT(lvlType)) {`。
- **L118 EN**: Continues logic associated with callable symbol `min`.
  **L118 CN**: 继续与可调用符号 `min` 相关的逻辑。
- **L119 EN**: Starts a function, method, lambda, or structured scope: `} else if (isSingletonLT(lvlType)) {`.
  **L119 CN**: 开始一个函数、方法、lambda 或结构化作用域：`} else if (isSingletonLT(lvlType)) {`。
- **L120 EN**: Continues logic associated with callable symbol `min`.
  **L120 CN**: 继续与可调用符号 `min` 相关的逻辑。

### Lines 121-140

````cpp
          }
        }
      }
    }
  }

  return minRank;
}

AffineMap IterationGraphSorter::topoSort() {
  // The sorted result will put the first Reduction iterator to the
  // latest possible position.
  std::vector<unsigned> redIt; // reduce iterator with 0 degree
  std::vector<unsigned> parIt; // parallel iterator with 0 degree
  const unsigned numLoops = getNumLoops();
  for (unsigned i = 0; i < numLoops; i++) {
    if (inDegree[i] == 0) {
      if (iterTypes[i] == utils::IteratorType::reduction)
        redIt.push_back(i);
      else
````
- **L121 EN**: Closes the current lexical scope or compound statement.
  **L121 CN**: 结束当前词法作用域或复合语句块。
- **L122 EN**: Closes the current lexical scope or compound statement.
  **L122 CN**: 结束当前词法作用域或复合语句块。
- **L123 EN**: Closes the current lexical scope or compound statement.
  **L123 CN**: 结束当前词法作用域或复合语句块。
- **L124 EN**: Closes the current lexical scope or compound statement.
  **L124 CN**: 结束当前词法作用域或复合语句块。
- **L125 EN**: Closes the current lexical scope or compound statement.
  **L125 CN**: 结束当前词法作用域或复合语句块。
- **L126 EN**: Blank line separating nearby declarations or logic blocks.
  **L126 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L127 EN**: Returns from the current function with `minRank`.
  **L127 CN**: 以 `minRank` 从当前函数返回。
- **L128 EN**: Closes the current lexical scope or compound statement.
  **L128 CN**: 结束当前词法作用域或复合语句块。
- **L129 EN**: Blank line separating nearby declarations or logic blocks.
  **L129 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L130 EN**: Starts a function, method, lambda, or structured scope: `AffineMap IterationGraphSorter::topoSort() {`.
  **L130 CN**: 开始一个函数、方法、lambda 或结构化作用域：`AffineMap IterationGraphSorter::topoSort() {`。
- **L131 EN**: Comment explains nearby logic, invariants, or intent: `The sorted result will put the first Reduction iterator to the`.
  **L131 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The sorted result will put the first Reduction iterator to the`。
- **L132 EN**: Comment explains nearby logic, invariants, or intent: `latest possible position.`.
  **L132 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`latest possible position.`。
- **L133 EN**: Continues the surrounding expression or declaration: `std::vector<unsigned> redIt; // reduce iterator with 0 degree`.
  **L133 CN**: 继续构造周围的表达式或声明：`std::vector<unsigned> redIt; // reduce iterator with 0 degree`。
- **L134 EN**: Continues the surrounding expression or declaration: `std::vector<unsigned> parIt; // parallel iterator with 0 degree`.
  **L134 CN**: 继续构造周围的表达式或声明：`std::vector<unsigned> parIt; // parallel iterator with 0 degree`。
- **L135 EN**: Initializes variable `numLoops` from the right-hand expression.
  **L135 CN**: 使用右侧表达式初始化变量 `numLoops`。
- **L136 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L136 CN**: 开始 `for` 控制流语句并计算其条件。
- **L137 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L137 CN**: 开始 `if` 控制流语句并计算其条件。
- **L138 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L138 CN**: 开始 `if` 控制流语句并计算其条件。
- **L139 EN**: Executes a call or declaration centered on `redIt.push_back`.
  **L139 CN**: 执行以 `redIt.push_back` 为核心的调用或声明。
- **L140 EN**: Starts the alternative branch of the preceding conditional.
  **L140 CN**: 开始前一个条件语句的备选分支。

### Lines 141-160

````cpp
        parIt.push_back(i);
    }
  }

  SmallVector<unsigned> loopOrder;
  while (!redIt.empty() || !parIt.empty()) {
    // We always prefer a parallel loop over a reduction loop because putting
    // a reduction loop early might make the loop sequence inadmissible.
    auto &it = !parIt.empty() ? parIt : redIt;

    // Select loop based on strategy.
    unsigned src;
    switch (strategy) {
    case sparse_tensor::LoopOrderingStrategy::kDefault:
      src = it.back();
      break;
    case sparse_tensor::LoopOrderingStrategy::kDenseOuter: {
      // Prefer dense, then compressed, then singleton dimensions outermost.
      // Create combined tensor and map lists for analysis.
      SmallVector<Value> allTensors = ins;
````
- **L141 EN**: Executes a call or declaration centered on `parIt.push_back`.
  **L141 CN**: 执行以 `parIt.push_back` 为核心的调用或声明。
- **L142 EN**: Closes the current lexical scope or compound statement.
  **L142 CN**: 结束当前词法作用域或复合语句块。
- **L143 EN**: Closes the current lexical scope or compound statement.
  **L143 CN**: 结束当前词法作用域或复合语句块。
- **L144 EN**: Blank line separating nearby declarations or logic blocks.
  **L144 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L145 EN**: Executes a standalone statement or declaration: `SmallVector<unsigned> loopOrder;`.
  **L145 CN**: 执行一条独立语句或声明：`SmallVector<unsigned> loopOrder;`。
- **L146 EN**: Begins a `while` control-flow statement and evaluates its condition.
  **L146 CN**: 开始 `while` 控制流语句并计算其条件。
- **L147 EN**: Comment explains nearby logic, invariants, or intent: `We always prefer a parallel loop over a reduction loop because putting`.
  **L147 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`We always prefer a parallel loop over a reduction loop because putting`。
- **L148 EN**: Comment explains nearby logic, invariants, or intent: `a reduction loop early might make the loop sequence inadmissible.`.
  **L148 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a reduction loop early might make the loop sequence inadmissible.`。
- **L149 EN**: Executes a call or declaration centered on `!parIt.empty`.
  **L149 CN**: 执行以 `!parIt.empty` 为核心的调用或声明。
- **L150 EN**: Blank line separating nearby declarations or logic blocks.
  **L150 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L151 EN**: Comment explains nearby logic, invariants, or intent: `Select loop based on strategy.`.
  **L151 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Select loop based on strategy.`。
- **L152 EN**: Executes a standalone statement or declaration: `unsigned src;`.
  **L152 CN**: 执行一条独立语句或声明：`unsigned src;`。
- **L153 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L153 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L154 EN**: Introduces a switch dispatch label: `case sparse_tensor::LoopOrderingStrategy::kDefault:`.
  **L154 CN**: 引入一个 switch 分发标签：`case sparse_tensor::LoopOrderingStrategy::kDefault:`。
- **L155 EN**: Executes a call or declaration centered on `it.back`.
  **L155 CN**: 执行以 `it.back` 为核心的调用或声明。
- **L156 EN**: Exits the nearest loop or switch statement.
  **L156 CN**: 退出最近的循环或 switch 语句。
- **L157 EN**: Introduces a switch dispatch label: `case sparse_tensor::LoopOrderingStrategy::kDenseOuter: {`.
  **L157 CN**: 引入一个 switch 分发标签：`case sparse_tensor::LoopOrderingStrategy::kDenseOuter: {`。
- **L158 EN**: Comment explains nearby logic, invariants, or intent: `Prefer dense, then compressed, then singleton dimensions outermost.`.
  **L158 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Prefer dense, then compressed, then singleton dimensions outermost.`。
- **L159 EN**: Comment explains nearby logic, invariants, or intent: `Create combined tensor and map lists for analysis.`.
  **L159 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create combined tensor and map lists for analysis.`。
- **L160 EN**: Initializes variable `allTensors` from the right-hand expression.
  **L160 CN**: 使用右侧表达式初始化变量 `allTensors`。

### Lines 161-180

````cpp
      allTensors.push_back(out);
      SmallVector<AffineMap> allMaps = loop2InsLvl;
      allMaps.push_back(loop2OutLvl);

      // Find loop with minimum (lowest) sparsity rank.
      unsigned minLoop = it[0];
      unsigned minRank = getLoopSparsityRank(minLoop, allTensors, allMaps);

      for (auto candidateLoop : it) {
        unsigned rank = getLoopSparsityRank(candidateLoop, allTensors, allMaps);
        if (rank < minRank || (rank == minRank && candidateLoop < minLoop)) {
          minLoop = candidateLoop;
          minRank = rank;
        }
      }
      src = minLoop;
      break;
    }
    }

````
- **L161 EN**: Executes a call or declaration centered on `allTensors.push_back`.
  **L161 CN**: 执行以 `allTensors.push_back` 为核心的调用或声明。
- **L162 EN**: Initializes variable `allMaps` from the right-hand expression.
  **L162 CN**: 使用右侧表达式初始化变量 `allMaps`。
- **L163 EN**: Executes a call or declaration centered on `allMaps.push_back`.
  **L163 CN**: 执行以 `allMaps.push_back` 为核心的调用或声明。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L165 EN**: Comment explains nearby logic, invariants, or intent: `Find loop with minimum (lowest) sparsity rank.`.
  **L165 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Find loop with minimum (lowest) sparsity rank.`。
- **L166 EN**: Initializes variable `minLoop` from the right-hand expression.
  **L166 CN**: 使用右侧表达式初始化变量 `minLoop`。
- **L167 EN**: Initializes variable `minRank` from the right-hand expression.
  **L167 CN**: 使用右侧表达式初始化变量 `minRank`。
- **L168 EN**: Blank line separating nearby declarations or logic blocks.
  **L168 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L169 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L169 CN**: 开始 `for` 控制流语句并计算其条件。
- **L170 EN**: Initializes variable `rank` from the right-hand expression.
  **L170 CN**: 使用右侧表达式初始化变量 `rank`。
- **L171 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L171 CN**: 开始 `if` 控制流语句并计算其条件。
- **L172 EN**: Executes a standalone statement or declaration: `minLoop = candidateLoop;`.
  **L172 CN**: 执行一条独立语句或声明：`minLoop = candidateLoop;`。
- **L173 EN**: Executes a standalone statement or declaration: `minRank = rank;`.
  **L173 CN**: 执行一条独立语句或声明：`minRank = rank;`。
- **L174 EN**: Closes the current lexical scope or compound statement.
  **L174 CN**: 结束当前词法作用域或复合语句块。
- **L175 EN**: Closes the current lexical scope or compound statement.
  **L175 CN**: 结束当前词法作用域或复合语句块。
- **L176 EN**: Executes a standalone statement or declaration: `src = minLoop;`.
  **L176 CN**: 执行一条独立语句或声明：`src = minLoop;`。
- **L177 EN**: Exits the nearest loop or switch statement.
  **L177 CN**: 退出最近的循环或 switch 语句。
- **L178 EN**: Closes the current lexical scope or compound statement.
  **L178 CN**: 结束当前词法作用域或复合语句块。
- **L179 EN**: Closes the current lexical scope or compound statement.
  **L179 CN**: 结束当前词法作用域或复合语句块。
- **L180 EN**: Blank line separating nearby declarations or logic blocks.
  **L180 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 181-200

````cpp
    loopOrder.push_back(src);
    // Remove the selected loop from the worklist.
    it.erase(std::find(it.begin(), it.end(), src));
    // Update in-degree, and push 0-degree node into worklist.
    for (unsigned dst = 0; dst < numLoops; dst++) {
      if (itGraph[src][dst] && --inDegree[dst] == 0) {
        if (iterTypes[dst] == utils::IteratorType::reduction)
          redIt.push_back(dst);
        else
          parIt.push_back(dst);
      }
    }
  }

  // Return the topological sort on success.
  if (loopOrder.size() == numLoops)
    return AffineMap::getPermutationMap(loopOrder, out.getContext());

  // Cycle detected.
  return AffineMap();
````
- **L181 EN**: Executes a call or declaration centered on `loopOrder.push_back`.
  **L181 CN**: 执行以 `loopOrder.push_back` 为核心的调用或声明。
- **L182 EN**: Comment explains nearby logic, invariants, or intent: `Remove the selected loop from the worklist.`.
  **L182 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Remove the selected loop from the worklist.`。
- **L183 EN**: Executes a call or declaration centered on `it.erase`.
  **L183 CN**: 执行以 `it.erase` 为核心的调用或声明。
- **L184 EN**: Comment explains nearby logic, invariants, or intent: `Update in-degree, and push 0-degree node into worklist.`.
  **L184 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Update in-degree, and push 0-degree node into worklist.`。
- **L185 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L185 CN**: 开始 `for` 控制流语句并计算其条件。
- **L186 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L186 CN**: 开始 `if` 控制流语句并计算其条件。
- **L187 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L187 CN**: 开始 `if` 控制流语句并计算其条件。
- **L188 EN**: Executes a call or declaration centered on `redIt.push_back`.
  **L188 CN**: 执行以 `redIt.push_back` 为核心的调用或声明。
- **L189 EN**: Starts the alternative branch of the preceding conditional.
  **L189 CN**: 开始前一个条件语句的备选分支。
- **L190 EN**: Executes a call or declaration centered on `parIt.push_back`.
  **L190 CN**: 执行以 `parIt.push_back` 为核心的调用或声明。
- **L191 EN**: Closes the current lexical scope or compound statement.
  **L191 CN**: 结束当前词法作用域或复合语句块。
- **L192 EN**: Closes the current lexical scope or compound statement.
  **L192 CN**: 结束当前词法作用域或复合语句块。
- **L193 EN**: Closes the current lexical scope or compound statement.
  **L193 CN**: 结束当前词法作用域或复合语句块。
- **L194 EN**: Blank line separating nearby declarations or logic blocks.
  **L194 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L195 EN**: Comment explains nearby logic, invariants, or intent: `Return the topological sort on success.`.
  **L195 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Return the topological sort on success.`。
- **L196 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L196 CN**: 开始 `if` 控制流语句并计算其条件。
- **L197 EN**: Returns from the current function with `AffineMap::getPermutationMap(loopOrder, out.getContext())`.
  **L197 CN**: 以 `AffineMap::getPermutationMap(loopOrder, out.getContext())` 从当前函数返回。
- **L198 EN**: Blank line separating nearby declarations or logic blocks.
  **L198 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L199 EN**: Comment explains nearby logic, invariants, or intent: `Cycle detected.`.
  **L199 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Cycle detected.`。
- **L200 EN**: Returns from the current function with `AffineMap()`.
  **L200 CN**: 以 `AffineMap()` 从当前函数返回。

### Lines 201-220

````cpp
}

IterationGraphSorter IterationGraphSorter::fromGenericOp(
    linalg::GenericOp genericOp, sparse_tensor::LoopOrderingStrategy strategy) {
  // Must be a demapped sparse kernel.
  assert(!hasAnyNonIdentityOperandsOrResults(genericOp) &&
         hasAnySparseOperandOrResult(genericOp) &&
         genericOp.getNumDpsInits() == 1);

  SmallVector<AffineMap> loopMap = genericOp.getIndexingMapsArray();
  SmallVector<Value> ins = genericOp.getDpsInputs();

  AffineMap outMap = loopMap.back();
  loopMap.pop_back();

  Value out = genericOp.getDpsInitOperand(0)->get();
  SmallVector<utils::IteratorType> iterTypes =
      genericOp.getIteratorTypesArray();

  return IterationGraphSorter(std::move(ins), std::move(loopMap), out, outMap,
````
- **L201 EN**: Closes the current lexical scope or compound statement.
  **L201 CN**: 结束当前词法作用域或复合语句块。
- **L202 EN**: Blank line separating nearby declarations or logic blocks.
  **L202 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L203 EN**: Continues logic associated with callable symbol `fromGenericOp`.
  **L203 CN**: 继续与可调用符号 `fromGenericOp` 相关的逻辑。
- **L204 EN**: Continues the surrounding expression or declaration: `linalg::GenericOp genericOp, sparse_tensor::LoopOrderingStrategy strategy) {`.
  **L204 CN**: 继续构造周围的表达式或声明：`linalg::GenericOp genericOp, sparse_tensor::LoopOrderingStrategy strategy) {`。
- **L205 EN**: Comment explains nearby logic, invariants, or intent: `Must be a demapped sparse kernel.`.
  **L205 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Must be a demapped sparse kernel.`。
- **L206 EN**: Checks an internal invariant in debug builds.
  **L206 CN**: 在调试构建中检查内部不变式。
- **L207 EN**: Continues logic associated with callable symbol `hasAnySparseOperandOrResult`.
  **L207 CN**: 继续与可调用符号 `hasAnySparseOperandOrResult` 相关的逻辑。
- **L208 EN**: Executes a call or declaration centered on `genericOp.getNumDpsInits`.
  **L208 CN**: 执行以 `genericOp.getNumDpsInits` 为核心的调用或声明。
- **L209 EN**: Blank line separating nearby declarations or logic blocks.
  **L209 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L210 EN**: Initializes variable `loopMap` from the right-hand expression.
  **L210 CN**: 使用右侧表达式初始化变量 `loopMap`。
- **L211 EN**: Initializes variable `ins` from the right-hand expression.
  **L211 CN**: 使用右侧表达式初始化变量 `ins`。
- **L212 EN**: Blank line separating nearby declarations or logic blocks.
  **L212 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L213 EN**: Initializes variable `outMap` from the right-hand expression.
  **L213 CN**: 使用右侧表达式初始化变量 `outMap`。
- **L214 EN**: Executes a call or declaration centered on `loopMap.pop_back`.
  **L214 CN**: 执行以 `loopMap.pop_back` 为核心的调用或声明。
- **L215 EN**: Blank line separating nearby declarations or logic blocks.
  **L215 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L216 EN**: Initializes variable `out` from the right-hand expression.
  **L216 CN**: 使用右侧表达式初始化变量 `out`。
- **L217 EN**: Continues the surrounding expression or declaration: `SmallVector<utils::IteratorType> iterTypes =`.
  **L217 CN**: 继续构造周围的表达式或声明：`SmallVector<utils::IteratorType> iterTypes =`。
- **L218 EN**: Executes a call or declaration centered on `genericOp.getIteratorTypesArray`.
  **L218 CN**: 执行以 `genericOp.getIteratorTypesArray` 为核心的调用或声明。
- **L219 EN**: Blank line separating nearby declarations or logic blocks.
  **L219 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L220 EN**: Returns from the current function with `IterationGraphSorter(std::move(ins), std::move(loopMap), out, outMap,`.
  **L220 CN**: 以 `IterationGraphSorter(std::move(ins), std::move(loopMap), out, outMap,` 从当前函数返回。

### Lines 221-240

````cpp
                              std::move(iterTypes), strategy);
}

IterationGraphSorter::IterationGraphSorter(
    SmallVector<Value> &&insArg, SmallVector<AffineMap> &&loop2InsLvlArg,
    Value out, AffineMap loop2OutLvl,
    SmallVector<utils::IteratorType> &&iterTypesArg,
    sparse_tensor::LoopOrderingStrategy strategy)
    : ins(std::move(insArg)), loop2InsLvl(std::move(loop2InsLvlArg)), out(out),
      loop2OutLvl(loop2OutLvl), iterTypes(std::move(iterTypesArg)),
      strategy(strategy) {
  // One map per tensor.
  assert(loop2InsLvl.size() == ins.size());
  // All the affine maps have the same number of dimensions (loops).
  assert(llvm::all_equal(llvm::map_range(
      loop2InsLvl, [](AffineMap m) { return m.getNumDims(); })));
  // The number of results of the map should match the rank of the tensor.
  assert(llvm::all_of(llvm::zip(loop2InsLvl, ins), [](auto mvPair) {
    auto [m, v] = mvPair;

````
- **L221 EN**: Executes a call or declaration centered on `std::move`.
  **L221 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L222 EN**: Closes the current lexical scope or compound statement.
  **L222 CN**: 结束当前词法作用域或复合语句块。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L224 EN**: Continues logic associated with callable symbol `IterationGraphSorter`.
  **L224 CN**: 继续与可调用符号 `IterationGraphSorter` 相关的逻辑。
- **L225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<Value> &&insArg, SmallVector<AffineMap> &&loop2InsLvlArg,`.
  **L225 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<Value> &&insArg, SmallVector<AffineMap> &&loop2InsLvlArg,`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value out, AffineMap loop2OutLvl,`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value out, AffineMap loop2OutLvl,`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<utils::IteratorType> &&iterTypesArg,`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<utils::IteratorType> &&iterTypesArg,`。
- **L228 EN**: Continues the surrounding expression or declaration: `sparse_tensor::LoopOrderingStrategy strategy)`.
  **L228 CN**: 继续构造周围的表达式或声明：`sparse_tensor::LoopOrderingStrategy strategy)`。
- **L229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ins(std::move(insArg)), loop2InsLvl(std::move(loop2InsLvlArg)), out(out),`.
  **L229 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ins(std::move(insArg)), loop2InsLvl(std::move(loop2InsLvlArg)), out(out),`。
- **L230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loop2OutLvl(loop2OutLvl), iterTypes(std::move(iterTypesArg)),`.
  **L230 CN**: 继续一个多行参数列表、初始化器或聚合项：`loop2OutLvl(loop2OutLvl), iterTypes(std::move(iterTypesArg)),`。
- **L231 EN**: Starts a function, method, lambda, or structured scope: `strategy(strategy) {`.
  **L231 CN**: 开始一个函数、方法、lambda 或结构化作用域：`strategy(strategy) {`。
- **L232 EN**: Comment explains nearby logic, invariants, or intent: `One map per tensor.`.
  **L232 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`One map per tensor.`。
- **L233 EN**: Checks an internal invariant in debug builds.
  **L233 CN**: 在调试构建中检查内部不变式。
- **L234 EN**: Comment explains nearby logic, invariants, or intent: `All the affine maps have the same number of dimensions (loops).`.
  **L234 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`All the affine maps have the same number of dimensions (loops).`。
- **L235 EN**: Checks an internal invariant in debug builds.
  **L235 CN**: 在调试构建中检查内部不变式。
- **L236 EN**: Executes a call or declaration centered on `[]`.
  **L236 CN**: 执行以 `[]` 为核心的调用或声明。
- **L237 EN**: Comment explains nearby logic, invariants, or intent: `The number of results of the map should match the rank of the tensor.`.
  **L237 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The number of results of the map should match the rank of the tensor.`。
- **L238 EN**: Checks an internal invariant in debug builds.
  **L238 CN**: 在调试构建中检查内部不变式。
- **L239 EN**: Executes a standalone statement or declaration: `auto [m, v] = mvPair;`.
  **L239 CN**: 执行一条独立语句或声明：`auto [m, v] = mvPair;`。
- **L240 EN**: Blank line separating nearby declarations or logic blocks.
  **L240 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 241-260

````cpp
    // For ranked types the rank must match.
    // Simply return true for UnrankedTensorType
    if (auto shapedType = llvm::dyn_cast<ShapedType>(v.getType())) {
      return !shapedType.hasRank() ||
             (m.getNumResults() == shapedType.getRank());
    }
    // Non-shaped (scalar) types behave like rank-0.
    return m.getNumResults() == 0;
  }));

  itGraph.resize(getNumLoops(), std::vector<bool>(getNumLoops(), false));
  inDegree.resize(getNumLoops());
}

AffineMap IterationGraphSorter::sort(SortMask mask, Value ignored) {
  // Reset the adjacency matrix that represents the iteration graph.
  for (auto &row : itGraph)
    llvm::fill(row, false);

  // Reset in-degree.
````
- **L241 EN**: Comment explains nearby logic, invariants, or intent: `For ranked types the rank must match.`.
  **L241 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For ranked types the rank must match.`。
- **L242 EN**: Comment explains nearby logic, invariants, or intent: `Simply return true for UnrankedTensorType`.
  **L242 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Simply return true for UnrankedTensorType`。
- **L243 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L243 CN**: 开始 `if` 控制流语句并计算其条件。
- **L244 EN**: Returns from the current function with `!shapedType.hasRank() ||`.
  **L244 CN**: 以 `!shapedType.hasRank() ||` 从当前函数返回。
- **L245 EN**: Executes a call or declaration centered on `statement`.
  **L245 CN**: 执行以 `statement` 为核心的调用或声明。
- **L246 EN**: Closes the current lexical scope or compound statement.
  **L246 CN**: 结束当前词法作用域或复合语句块。
- **L247 EN**: Comment explains nearby logic, invariants, or intent: `Non-shaped (scalar) types behave like rank-0.`.
  **L247 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Non-shaped (scalar) types behave like rank-0.`。
- **L248 EN**: Returns from the current function with `m.getNumResults() == 0`.
  **L248 CN**: 以 `m.getNumResults() == 0` 从当前函数返回。
- **L249 EN**: Executes a standalone statement or declaration: `}));`.
  **L249 CN**: 执行一条独立语句或声明：`}));`。
- **L250 EN**: Blank line separating nearby declarations or logic blocks.
  **L250 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L251 EN**: Executes a call or declaration centered on `itGraph.resize`.
  **L251 CN**: 执行以 `itGraph.resize` 为核心的调用或声明。
- **L252 EN**: Executes a call or declaration centered on `inDegree.resize`.
  **L252 CN**: 执行以 `inDegree.resize` 为核心的调用或声明。
- **L253 EN**: Closes the current lexical scope or compound statement.
  **L253 CN**: 结束当前词法作用域或复合语句块。
- **L254 EN**: Blank line separating nearby declarations or logic blocks.
  **L254 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L255 EN**: Starts a function, method, lambda, or structured scope: `AffineMap IterationGraphSorter::sort(SortMask mask, Value ignored) {`.
  **L255 CN**: 开始一个函数、方法、lambda 或结构化作用域：`AffineMap IterationGraphSorter::sort(SortMask mask, Value ignored) {`。
- **L256 EN**: Comment explains nearby logic, invariants, or intent: `Reset the adjacency matrix that represents the iteration graph.`.
  **L256 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Reset the adjacency matrix that represents the iteration graph.`。
- **L257 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L257 CN**: 开始 `for` 控制流语句并计算其条件。
- **L258 EN**: Executes a call or declaration centered on `llvm::fill`.
  **L258 CN**: 执行以 `llvm::fill` 为核心的调用或声明。
- **L259 EN**: Blank line separating nearby declarations or logic blocks.
  **L259 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L260 EN**: Comment explains nearby logic, invariants, or intent: `Reset in-degree.`.
  **L260 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Reset in-degree.`。

### Lines 261-280

````cpp
  llvm::fill(inDegree, 0);

  // Add the constraints for the loop to level map.
  for (auto [in, map] : llvm::zip(ins, loop2InsLvl)) {
    // Get map and encoding.
    const auto enc = getSparseTensorEncoding(in.getType());
    // Skip dense inputs when not requested.
    if ((!enc && !includesDenseInput(mask)) || in == ignored)
      continue;
    addConstraints(in, map);
  }

  // Add the constraints for the output map.
  const auto enc = getSparseTensorEncoding(out.getType());
  if ((enc || includesDenseOutput(mask)) && out != ignored)
    addConstraints(out, loop2OutLvl);

  // Return the topological sort (empty for cyclic).
  return topoSort();
}
````
- **L261 EN**: Executes a call or declaration centered on `llvm::fill`.
  **L261 CN**: 执行以 `llvm::fill` 为核心的调用或声明。
- **L262 EN**: Blank line separating nearby declarations or logic blocks.
  **L262 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L263 EN**: Comment explains nearby logic, invariants, or intent: `Add the constraints for the loop to level map.`.
  **L263 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Add the constraints for the loop to level map.`。
- **L264 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L264 CN**: 开始 `for` 控制流语句并计算其条件。
- **L265 EN**: Comment explains nearby logic, invariants, or intent: `Get map and encoding.`.
  **L265 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Get map and encoding.`。
- **L266 EN**: Initializes variable `enc` from the right-hand expression.
  **L266 CN**: 使用右侧表达式初始化变量 `enc`。
- **L267 EN**: Comment explains nearby logic, invariants, or intent: `Skip dense inputs when not requested.`.
  **L267 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Skip dense inputs when not requested.`。
- **L268 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L268 CN**: 开始 `if` 控制流语句并计算其条件。
- **L269 EN**: Skips to the next loop iteration.
  **L269 CN**: 跳到下一次循环迭代。
- **L270 EN**: Executes a call or declaration centered on `addConstraints`.
  **L270 CN**: 执行以 `addConstraints` 为核心的调用或声明。
- **L271 EN**: Closes the current lexical scope or compound statement.
  **L271 CN**: 结束当前词法作用域或复合语句块。
- **L272 EN**: Blank line separating nearby declarations or logic blocks.
  **L272 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L273 EN**: Comment explains nearby logic, invariants, or intent: `Add the constraints for the output map.`.
  **L273 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Add the constraints for the output map.`。
- **L274 EN**: Initializes variable `enc` from the right-hand expression.
  **L274 CN**: 使用右侧表达式初始化变量 `enc`。
- **L275 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L275 CN**: 开始 `if` 控制流语句并计算其条件。
- **L276 EN**: Executes a call or declaration centered on `addConstraints`.
  **L276 CN**: 执行以 `addConstraints` 为核心的调用或声明。
- **L277 EN**: Blank line separating nearby declarations or logic blocks.
  **L277 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L278 EN**: Comment explains nearby logic, invariants, or intent: `Return the topological sort (empty for cyclic).`.
  **L278 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Return the topological sort (empty for cyclic).`。
- **L279 EN**: Returns from the current function with `topoSort()`.
  **L279 CN**: 以 `topoSort()` 从当前函数返回。
- **L280 EN**: Closes the current lexical scope or compound statement.
  **L280 CN**: 结束当前词法作用域或复合语句块。

### Lines 281-300

````cpp

void IterationGraphSorter::addConstraints(Value t, AffineMap loop2LvlMap) {
  auto addIterOrdering = [this](unsigned f, unsigned t) {
    if (!itGraph[f][t] && f != t) {
      itGraph[f][t] = true;
      inDegree[t]++;
    }
  };

  // Set up a reduction finder.
  AffineDimFinder finder(iterTypes);
  finder.setPickedIterType(utils::IteratorType::reduction);

  // To compute iteration graph for tensor[d0 + d1 + d3, d4 + d5 + d6],
  // we require there exist d_x \in {d0, d1, d3} and d_y \in {d4, d5, d6},
  // and d_x > d_y && {d0, d1, d3} - d_x > {d4, d5, d6} - d_y
  const Level lvlRank = loop2LvlMap.getNumResults();
  for (Level lvl = 1; lvl < lvlRank; lvl++) {
    const AffineExpr fa = loop2LvlMap.getResult(lvl - 1);
    const AffineExpr ta = loop2LvlMap.getResult(lvl);
````
- **L281 EN**: Blank line separating nearby declarations or logic blocks.
  **L281 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L282 EN**: Starts a function, method, lambda, or structured scope: `void IterationGraphSorter::addConstraints(Value t, AffineMap loop2LvlMap) {`.
  **L282 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IterationGraphSorter::addConstraints(Value t, AffineMap loop2LvlMap) {`。
- **L283 EN**: Starts a function, method, lambda, or structured scope: `auto addIterOrdering = [this](unsigned f, unsigned t) {`.
  **L283 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto addIterOrdering = [this](unsigned f, unsigned t) {`。
- **L284 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L284 CN**: 开始 `if` 控制流语句并计算其条件。
- **L285 EN**: Executes a standalone statement or declaration: `itGraph[f][t] = true;`.
  **L285 CN**: 执行一条独立语句或声明：`itGraph[f][t] = true;`。
- **L286 EN**: Executes a standalone statement or declaration: `inDegree[t]++;`.
  **L286 CN**: 执行一条独立语句或声明：`inDegree[t]++;`。
- **L287 EN**: Closes the current lexical scope or compound statement.
  **L287 CN**: 结束当前词法作用域或复合语句块。
- **L288 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L288 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L289 EN**: Blank line separating nearby declarations or logic blocks.
  **L289 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L290 EN**: Comment explains nearby logic, invariants, or intent: `Set up a reduction finder.`.
  **L290 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Set up a reduction finder.`。
- **L291 EN**: Executes a call or declaration centered on `finder`.
  **L291 CN**: 执行以 `finder` 为核心的调用或声明。
- **L292 EN**: Executes a call or declaration centered on `finder.setPickedIterType`.
  **L292 CN**: 执行以 `finder.setPickedIterType` 为核心的调用或声明。
- **L293 EN**: Blank line separating nearby declarations or logic blocks.
  **L293 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L294 EN**: Comment explains nearby logic, invariants, or intent: `To compute iteration graph for tensor[d0 + d1 + d3, d4 + d5 + d6],`.
  **L294 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`To compute iteration graph for tensor[d0 + d1 + d3, d4 + d5 + d6],`。
- **L295 EN**: Comment explains nearby logic, invariants, or intent: `we require there exist d_x \in {d0, d1, d3} and d_y \in {d4, d5, d6},`.
  **L295 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`we require there exist d_x \in {d0, d1, d3} and d_y \in {d4, d5, d6},`。
- **L296 EN**: Comment explains nearby logic, invariants, or intent: `and d_x > d_y && {d0, d1, d3} - d_x > {d4, d5, d6} - d_y`.
  **L296 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and d_x > d_y && {d0, d1, d3} - d_x > {d4, d5, d6} - d_y`。
- **L297 EN**: Initializes variable `lvlRank` from the right-hand expression.
  **L297 CN**: 使用右侧表达式初始化变量 `lvlRank`。
- **L298 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L298 CN**: 开始 `for` 控制流语句并计算其条件。
- **L299 EN**: Initializes variable `fa` from the right-hand expression.
  **L299 CN**: 使用右侧表达式初始化变量 `fa`。
- **L300 EN**: Initializes variable `ta` from the right-hand expression.
  **L300 CN**: 使用右侧表达式初始化变量 `ta`。

### Lines 301-320

````cpp

    if (llvm::isa<AffineDimExpr>(fa) || llvm::isa<AffineDimExpr>(ta)) {
      // Special case when at least one loop2LvlExp is a simple AffineDimExpr
      // (say, d0) and we require d0 > {d1, d2, ...} or {d1, d2, ...} > d0
      AffineDimCollector fCollector;
      fCollector.walkPostOrder(fa);
      AffineDimCollector tCollector;
      tCollector.walkPostOrder(ta);

      for (auto fd : fCollector.dims) {
        for (auto td : tCollector.dims) {
          const unsigned f = fd.getPosition();
          const unsigned t = td.getPosition();
          addIterOrdering(f, t);
        }
      }
      continue;
    }

    // When both loop2LvlExpr is compound, we pick an abitrary reduction loop
````
- **L301 EN**: Blank line separating nearby declarations or logic blocks.
  **L301 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L302 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L302 CN**: 开始 `if` 控制流语句并计算其条件。
- **L303 EN**: Comment explains nearby logic, invariants, or intent: `Special case when at least one loop2LvlExp is a simple AffineDimExpr`.
  **L303 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Special case when at least one loop2LvlExp is a simple AffineDimExpr`。
- **L304 EN**: Comment explains nearby logic, invariants, or intent: `(say, d0) and we require d0 > {d1, d2, ...} or {d1, d2, ...} > d0`.
  **L304 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(say, d0) and we require d0 > {d1, d2, ...} or {d1, d2, ...} > d0`。
- **L305 EN**: Executes a standalone statement or declaration: `AffineDimCollector fCollector;`.
  **L305 CN**: 执行一条独立语句或声明：`AffineDimCollector fCollector;`。
- **L306 EN**: Executes a call or declaration centered on `fCollector.walkPostOrder`.
  **L306 CN**: 执行以 `fCollector.walkPostOrder` 为核心的调用或声明。
- **L307 EN**: Executes a standalone statement or declaration: `AffineDimCollector tCollector;`.
  **L307 CN**: 执行一条独立语句或声明：`AffineDimCollector tCollector;`。
- **L308 EN**: Executes a call or declaration centered on `tCollector.walkPostOrder`.
  **L308 CN**: 执行以 `tCollector.walkPostOrder` 为核心的调用或声明。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L310 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L310 CN**: 开始 `for` 控制流语句并计算其条件。
- **L311 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L311 CN**: 开始 `for` 控制流语句并计算其条件。
- **L312 EN**: Initializes variable `f` from the right-hand expression.
  **L312 CN**: 使用右侧表达式初始化变量 `f`。
- **L313 EN**: Initializes variable `t` from the right-hand expression.
  **L313 CN**: 使用右侧表达式初始化变量 `t`。
- **L314 EN**: Executes a call or declaration centered on `addIterOrdering`.
  **L314 CN**: 执行以 `addIterOrdering` 为核心的调用或声明。
- **L315 EN**: Closes the current lexical scope or compound statement.
  **L315 CN**: 结束当前词法作用域或复合语句块。
- **L316 EN**: Closes the current lexical scope or compound statement.
  **L316 CN**: 结束当前词法作用域或复合语句块。
- **L317 EN**: Skips to the next loop iteration.
  **L317 CN**: 跳到下一次循环迭代。
- **L318 EN**: Closes the current lexical scope or compound statement.
  **L318 CN**: 结束当前词法作用域或复合语句块。
- **L319 EN**: Blank line separating nearby declarations or logic blocks.
  **L319 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L320 EN**: Comment explains nearby logic, invariants, or intent: `When both loop2LvlExpr is compound, we pick an abitrary reduction loop`.
  **L320 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`When both loop2LvlExpr is compound, we pick an abitrary reduction loop`。

### Lines 321-340

````cpp
    // from lhs and rhs and use them as d_x and d_y.
    finder.walkPostOrder(fa);
    const AffineDimExpr fexp = finder.getDimExpr();
    const unsigned fldx = fexp.getPosition();

    finder.walkPostOrder(ta);
    const AffineDimExpr texp = finder.getDimExpr();
    const unsigned tldx = texp.getPosition();

    // d_x > d_y
    addIterOrdering(fldx, tldx);

    AffineDimCollector fCollector;
    fCollector.walkPostOrder(fa);
    AffineDimCollector tCollector;
    tCollector.walkPostOrder(ta);

    // Make sure dx and dy is the last.
    for (auto fd : fCollector.dims) {
      const unsigned f = fd.getPosition();
````
- **L321 EN**: Comment explains nearby logic, invariants, or intent: `from lhs and rhs and use them as d_x and d_y.`.
  **L321 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`from lhs and rhs and use them as d_x and d_y.`。
- **L322 EN**: Executes a call or declaration centered on `finder.walkPostOrder`.
  **L322 CN**: 执行以 `finder.walkPostOrder` 为核心的调用或声明。
- **L323 EN**: Initializes variable `fexp` from the right-hand expression.
  **L323 CN**: 使用右侧表达式初始化变量 `fexp`。
- **L324 EN**: Initializes variable `fldx` from the right-hand expression.
  **L324 CN**: 使用右侧表达式初始化变量 `fldx`。
- **L325 EN**: Blank line separating nearby declarations or logic blocks.
  **L325 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L326 EN**: Executes a call or declaration centered on `finder.walkPostOrder`.
  **L326 CN**: 执行以 `finder.walkPostOrder` 为核心的调用或声明。
- **L327 EN**: Initializes variable `texp` from the right-hand expression.
  **L327 CN**: 使用右侧表达式初始化变量 `texp`。
- **L328 EN**: Initializes variable `tldx` from the right-hand expression.
  **L328 CN**: 使用右侧表达式初始化变量 `tldx`。
- **L329 EN**: Blank line separating nearby declarations or logic blocks.
  **L329 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L330 EN**: Comment explains nearby logic, invariants, or intent: `d_x > d_y`.
  **L330 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`d_x > d_y`。
- **L331 EN**: Executes a call or declaration centered on `addIterOrdering`.
  **L331 CN**: 执行以 `addIterOrdering` 为核心的调用或声明。
- **L332 EN**: Blank line separating nearby declarations or logic blocks.
  **L332 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L333 EN**: Executes a standalone statement or declaration: `AffineDimCollector fCollector;`.
  **L333 CN**: 执行一条独立语句或声明：`AffineDimCollector fCollector;`。
- **L334 EN**: Executes a call or declaration centered on `fCollector.walkPostOrder`.
  **L334 CN**: 执行以 `fCollector.walkPostOrder` 为核心的调用或声明。
- **L335 EN**: Executes a standalone statement or declaration: `AffineDimCollector tCollector;`.
  **L335 CN**: 执行一条独立语句或声明：`AffineDimCollector tCollector;`。
- **L336 EN**: Executes a call or declaration centered on `tCollector.walkPostOrder`.
  **L336 CN**: 执行以 `tCollector.walkPostOrder` 为核心的调用或声明。
- **L337 EN**: Blank line separating nearby declarations or logic blocks.
  **L337 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L338 EN**: Comment explains nearby logic, invariants, or intent: `Make sure dx and dy is the last.`.
  **L338 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Make sure dx and dy is the last.`。
- **L339 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L339 CN**: 开始 `for` 控制流语句并计算其条件。
- **L340 EN**: Initializes variable `f` from the right-hand expression.
  **L340 CN**: 使用右侧表达式初始化变量 `f`。

### Lines 341-360

````cpp
      addIterOrdering(f, fldx);
    }
    for (auto td : tCollector.dims) {
      const unsigned t = td.getPosition();
      addIterOrdering(t, tldx);
    }
    // {d0, d1, d3} - d_x > {d4, d5, d6} - d_y
    // This is to ensure that the affine expressions are reduced in sparse
    // tensor level ordering.
    for (auto fd : fCollector.dims) {
      const unsigned f = fd.getPosition();
      if (f == fldx) // skip d_x
        continue;
      for (auto td : tCollector.dims) {
        const unsigned t = td.getPosition();
        if (t == tldx) // skip d_y
          continue;
        addIterOrdering(f, t);
      }
    }
````
- **L341 EN**: Executes a call or declaration centered on `addIterOrdering`.
  **L341 CN**: 执行以 `addIterOrdering` 为核心的调用或声明。
- **L342 EN**: Closes the current lexical scope or compound statement.
  **L342 CN**: 结束当前词法作用域或复合语句块。
- **L343 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L343 CN**: 开始 `for` 控制流语句并计算其条件。
- **L344 EN**: Initializes variable `t` from the right-hand expression.
  **L344 CN**: 使用右侧表达式初始化变量 `t`。
- **L345 EN**: Executes a call or declaration centered on `addIterOrdering`.
  **L345 CN**: 执行以 `addIterOrdering` 为核心的调用或声明。
- **L346 EN**: Closes the current lexical scope or compound statement.
  **L346 CN**: 结束当前词法作用域或复合语句块。
- **L347 EN**: Comment explains nearby logic, invariants, or intent: `{d0, d1, d3} - d_x > {d4, d5, d6} - d_y`.
  **L347 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`{d0, d1, d3} - d_x > {d4, d5, d6} - d_y`。
- **L348 EN**: Comment explains nearby logic, invariants, or intent: `This is to ensure that the affine expressions are reduced in sparse`.
  **L348 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This is to ensure that the affine expressions are reduced in sparse`。
- **L349 EN**: Comment explains nearby logic, invariants, or intent: `tensor level ordering.`.
  **L349 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tensor level ordering.`。
- **L350 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L350 CN**: 开始 `for` 控制流语句并计算其条件。
- **L351 EN**: Initializes variable `f` from the right-hand expression.
  **L351 CN**: 使用右侧表达式初始化变量 `f`。
- **L352 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L352 CN**: 开始 `if` 控制流语句并计算其条件。
- **L353 EN**: Skips to the next loop iteration.
  **L353 CN**: 跳到下一次循环迭代。
- **L354 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L354 CN**: 开始 `for` 控制流语句并计算其条件。
- **L355 EN**: Initializes variable `t` from the right-hand expression.
  **L355 CN**: 使用右侧表达式初始化变量 `t`。
- **L356 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L356 CN**: 开始 `if` 控制流语句并计算其条件。
- **L357 EN**: Skips to the next loop iteration.
  **L357 CN**: 跳到下一次循环迭代。
- **L358 EN**: Executes a call or declaration centered on `addIterOrdering`.
  **L358 CN**: 执行以 `addIterOrdering` 为核心的调用或声明。
- **L359 EN**: Closes the current lexical scope or compound statement.
  **L359 CN**: 结束当前词法作用域或复合语句块。
- **L360 EN**: Closes the current lexical scope or compound statement.
  **L360 CN**: 结束当前词法作用域或复合语句块。

### Lines 361-362

````cpp
  }
}
````
- **L361 EN**: Closes the current lexical scope or compound statement.
  **L361 CN**: 结束当前词法作用域或复合语句块。
- **L362 EN**: Closes the current lexical scope or compound statement.
  **L362 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Sparse tensor compilation / 稀疏张量编译**
- **Rewrite-driven lowering / 基于重写的 lowering**
- **Shared helper utilities / 共享辅助工具**
- **Dialect definition and registration / 方言定义与注册**
- **Type-system modeling / 类型系统建模**
- **SSA value representation / SSA 值表示**
- **Shaped type reasoning / 形状类型推理**
- **Tensor-level abstraction / 张量层抽象**
- **Sparse tensor abstraction / 稀疏张量抽象**
- **Vector-level optimization and lowering / 向量级优化与 lowering**

## Dependencies / 依赖关系

- `IterationGraphSorter.h`: Provides local declarations paired with this implementation unit. / 提供与该实现单元配套的本地声明。
- `mlir/Dialect/Linalg/IR/Linalg.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/SparseTensor/IR/SparseTensor.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Utils/StructuredOpsUtils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/IR/AffineExprVisitor.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
- `mlir/IR/BuiltinTypes.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
