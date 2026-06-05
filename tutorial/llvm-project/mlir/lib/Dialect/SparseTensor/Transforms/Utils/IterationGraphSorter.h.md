# IterationGraphSorter.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SparseTensor/Transforms/Utils/IterationGraphSorter.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This header file defines the iteration graph sorter (top-sort scheduling).
- **Purpose (CN)**: 声明稀疏张量优化、lowering 与重写 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

````cpp
//===- IterationGraphSorter.h -----------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This header file defines the iteration graph sorter (top-sort scheduling).
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_SPARSETENSOR_TRANSFORMS_UTILS_ITERATIONGRAPHSORTER_H_
#define MLIR_DIALECT_SPARSETENSOR_TRANSFORMS_UTILS_ITERATIONGRAPHSORTER_H_

#include "mlir/Dialect/SparseTensor/Transforms/Passes.h"
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
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This header file defines the iteration graph sorter (top-sort scheduling).`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This header file defines the iteration graph sorter (top-sort scheduling).`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Starts a preprocessor conditional block: `#ifndef MLIR_DIALECT_SPARSETENSOR_TRANSFORMS_UTILS_ITERATIONGRAPHSORTER_H_`.
  **L13 CN**: 开始一个预处理条件块：`#ifndef MLIR_DIALECT_SPARSETENSOR_TRANSFORMS_UTILS_ITERATIONGRAPHSORTER_H_`。
- **L14 EN**: Defines macro `MLIR_DIALECT_SPARSETENSOR_TRANSFORMS_UTILS_ITERATIONGRAPHSORTER_H_` for generated declarations, local shorthand, or conditional logic.
  **L14 CN**: 定义宏 `MLIR_DIALECT_SPARSETENSOR_TRANSFORMS_UTILS_ITERATIONGRAPHSORTER_H_`，供生成式声明、本地简写或条件逻辑使用。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L16 EN**: Includes "mlir/Dialect/SparseTensor/Transforms/Passes.h" to access dialect-specific IR, transforms, or shared utilities.
  **L16 CN**: 引入 "mlir/Dialect/SparseTensor/Transforms/Passes.h" 以使用方言专用 IR、变换或共享工具。

### Lines 17-32

````cpp
#include "mlir/IR/AffineMap.h"

namespace mlir {

// Forward declarations.
class Value;
namespace utils {
enum class IteratorType : uint32_t;
} // namespace utils
namespace linalg {
class GenericOp;
} // namespace linalg

namespace sparse_tensor {

/// Iteration graph sorting mask,
````
- **L17 EN**: Includes "mlir/IR/AffineMap.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L17 CN**: 引入 "mlir/IR/AffineMap.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L19 EN**: Opens namespace scope `mlir`.
  **L19 CN**: 打开命名空间作用域 `mlir`。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L21 EN**: Comment explains nearby logic, invariants, or intent: `Forward declarations.`.
  **L21 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Forward declarations.`。
- **L22 EN**: Declares class `Value;`.
  **L22 CN**: 声明 class `Value;`。
- **L23 EN**: Opens namespace scope `utils`.
  **L23 CN**: 打开命名空间作用域 `utils`。
- **L24 EN**: Declares enum `class`.
  **L24 CN**: 声明 enum `class`。
- **L25 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace utils`.
  **L25 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace utils`。
- **L26 EN**: Opens namespace scope `linalg`.
  **L26 CN**: 打开命名空间作用域 `linalg`。
- **L27 EN**: Declares class `GenericOp;`.
  **L27 CN**: 声明 class `GenericOp;`。
- **L28 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace linalg`.
  **L28 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace linalg`。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L30 EN**: Opens namespace scope `sparse_tensor`.
  **L30 CN**: 打开命名空间作用域 `sparse_tensor`。
- **L31 EN**: Blank line separating nearby declarations or logic blocks.
  **L31 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L32 EN**: Comment explains nearby logic, invariants, or intent: `Iteration graph sorting mask,`.
  **L32 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Iteration graph sorting mask,`。

### Lines 33-48

````cpp
enum class SortMask : unsigned {
  // The individual mask bits.
  kIncludeDenseOutput = 0x1, // b001
  kIncludeDenseInput = 0x2,  // b010
  // The subsets of mask bits.
  kIncludeAll = 0x7,   // b111
  kIncludeDense = 0x3, // b011
  kSparseOnly = 0x0,   // b000
};

class IterationGraphSorter {
public:
  /// Factory method that constructs an iteration graph sorter
  /// for the given linalg.generic operation with a specific loop ordering
  /// strategy.
  static IterationGraphSorter
````
- **L33 EN**: Declares enum `class`.
  **L33 CN**: 声明 enum `class`。
- **L34 EN**: Comment explains nearby logic, invariants, or intent: `The individual mask bits.`.
  **L34 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The individual mask bits.`。
- **L35 EN**: Continues the surrounding expression or declaration: `kIncludeDenseOutput = 0x1, // b001`.
  **L35 CN**: 继续构造周围的表达式或声明：`kIncludeDenseOutput = 0x1, // b001`。
- **L36 EN**: Continues the surrounding expression or declaration: `kIncludeDenseInput = 0x2,  // b010`.
  **L36 CN**: 继续构造周围的表达式或声明：`kIncludeDenseInput = 0x2,  // b010`。
- **L37 EN**: Comment explains nearby logic, invariants, or intent: `The subsets of mask bits.`.
  **L37 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The subsets of mask bits.`。
- **L38 EN**: Continues the surrounding expression or declaration: `kIncludeAll = 0x7,   // b111`.
  **L38 CN**: 继续构造周围的表达式或声明：`kIncludeAll = 0x7,   // b111`。
- **L39 EN**: Continues the surrounding expression or declaration: `kIncludeDense = 0x3, // b011`.
  **L39 CN**: 继续构造周围的表达式或声明：`kIncludeDense = 0x3, // b011`。
- **L40 EN**: Continues the surrounding expression or declaration: `kSparseOnly = 0x0,   // b000`.
  **L40 CN**: 继续构造周围的表达式或声明：`kSparseOnly = 0x0,   // b000`。
- **L41 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L41 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L42 EN**: Blank line separating nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L43 EN**: Declares class `IterationGraphSorter`.
  **L43 CN**: 声明 class `IterationGraphSorter`。
- **L44 EN**: Sets the following members to `public` access.
  **L44 CN**: 将后续成员的访问级别设为 `public`。
- **L45 EN**: Comment explains nearby logic, invariants, or intent: `Factory method that constructs an iteration graph sorter`.
  **L45 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Factory method that constructs an iteration graph sorter`。
- **L46 EN**: Comment explains nearby logic, invariants, or intent: `for the given linalg.generic operation with a specific loop ordering`.
  **L46 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`for the given linalg.generic operation with a specific loop ordering`。
- **L47 EN**: Comment explains nearby logic, invariants, or intent: `strategy.`.
  **L47 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`strategy.`。
- **L48 EN**: Continues the surrounding expression or declaration: `static IterationGraphSorter`.
  **L48 CN**: 继续构造周围的表达式或声明：`static IterationGraphSorter`。

### Lines 49-64

````cpp
  fromGenericOp(linalg::GenericOp genericOp,
                sparse_tensor::LoopOrderingStrategy strategy);

  /// Returns a permutation that represents the scheduled loop order.
  /// Note that the returned AffineMap could be null if the kernel
  /// cannot be scheduled due to cyclic iteration graph.
  [[nodiscard]] AffineMap sort(SortMask mask, Value ignored = nullptr);

  /// Returns the number of loops in the iteration graph.
  unsigned getNumLoops() const { return loop2OutLvl.getNumDims(); }

private:
  // Private constructor.
  IterationGraphSorter(SmallVector<Value> &&insArg,
                       SmallVector<AffineMap> &&loop2InsLvlArg, Value out,
                       AffineMap loop2OutLvl,
````
- **L49 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fromGenericOp(linalg::GenericOp genericOp,`.
  **L49 CN**: 继续一个多行参数列表、初始化器或聚合项：`fromGenericOp(linalg::GenericOp genericOp,`。
- **L50 EN**: Executes a standalone statement or declaration: `sparse_tensor::LoopOrderingStrategy strategy);`.
  **L50 CN**: 执行一条独立语句或声明：`sparse_tensor::LoopOrderingStrategy strategy);`。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L52 EN**: Comment explains nearby logic, invariants, or intent: `Returns a permutation that represents the scheduled loop order.`.
  **L52 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns a permutation that represents the scheduled loop order.`。
- **L53 EN**: Comment explains nearby logic, invariants, or intent: `Note that the returned AffineMap could be null if the kernel`.
  **L53 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note that the returned AffineMap could be null if the kernel`。
- **L54 EN**: Comment explains nearby logic, invariants, or intent: `cannot be scheduled due to cyclic iteration graph.`.
  **L54 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`cannot be scheduled due to cyclic iteration graph.`。
- **L55 EN**: Executes a call or declaration centered on `sort`.
  **L55 CN**: 执行以 `sort` 为核心的调用或声明。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L57 EN**: Comment explains nearby logic, invariants, or intent: `Returns the number of loops in the iteration graph.`.
  **L57 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns the number of loops in the iteration graph.`。
- **L58 EN**: Continues logic associated with callable symbol `getNumLoops`.
  **L58 CN**: 继续与可调用符号 `getNumLoops` 相关的逻辑。
- **L59 EN**: Blank line separating nearby declarations or logic blocks.
  **L59 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L60 EN**: Sets the following members to `private` access.
  **L60 CN**: 将后续成员的访问级别设为 `private`。
- **L61 EN**: Comment explains nearby logic, invariants, or intent: `Private constructor.`.
  **L61 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Private constructor.`。
- **L62 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IterationGraphSorter(SmallVector<Value> &&insArg,`.
  **L62 CN**: 继续一个多行参数列表、初始化器或聚合项：`IterationGraphSorter(SmallVector<Value> &&insArg,`。
- **L63 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<AffineMap> &&loop2InsLvlArg, Value out,`.
  **L63 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<AffineMap> &&loop2InsLvlArg, Value out,`。
- **L64 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AffineMap loop2OutLvl,`.
  **L64 CN**: 继续一个多行参数列表、初始化器或聚合项：`AffineMap loop2OutLvl,`。

### Lines 65-80

````cpp
                       SmallVector<utils::IteratorType> &&iterTypesArg,
                       sparse_tensor::LoopOrderingStrategy strategy =
                           sparse_tensor::LoopOrderingStrategy::kDefault);

  // Adds all the constraints in the given loop to level map.
  void addConstraints(Value t, AffineMap loop2LvlMap);

  /// A helper to compute a topological sort. The method has an
  /// O(n^2) time complexity since we use an adjacency matrix
  /// representation for the iteration graph.
  AffineMap topoSort();

  // Input tensors and associated loop to level maps.
  SmallVector<Value> ins;
  SmallVector<AffineMap> loop2InsLvl;

````
- **L65 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<utils::IteratorType> &&iterTypesArg,`.
  **L65 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<utils::IteratorType> &&iterTypesArg,`。
- **L66 EN**: Continues the surrounding expression or declaration: `sparse_tensor::LoopOrderingStrategy strategy =`.
  **L66 CN**: 继续构造周围的表达式或声明：`sparse_tensor::LoopOrderingStrategy strategy =`。
- **L67 EN**: Executes a standalone statement or declaration: `sparse_tensor::LoopOrderingStrategy::kDefault);`.
  **L67 CN**: 执行一条独立语句或声明：`sparse_tensor::LoopOrderingStrategy::kDefault);`。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L69 EN**: Comment explains nearby logic, invariants, or intent: `Adds all the constraints in the given loop to level map.`.
  **L69 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Adds all the constraints in the given loop to level map.`。
- **L70 EN**: Executes a call or declaration centered on `addConstraints`.
  **L70 CN**: 执行以 `addConstraints` 为核心的调用或声明。
- **L71 EN**: Blank line separating nearby declarations or logic blocks.
  **L71 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L72 EN**: Comment explains nearby logic, invariants, or intent: `A helper to compute a topological sort. The method has an`.
  **L72 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A helper to compute a topological sort. The method has an`。
- **L73 EN**: Comment explains nearby logic, invariants, or intent: `O(n^2) time complexity since we use an adjacency matrix`.
  **L73 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`O(n^2) time complexity since we use an adjacency matrix`。
- **L74 EN**: Comment explains nearby logic, invariants, or intent: `representation for the iteration graph.`.
  **L74 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`representation for the iteration graph.`。
- **L75 EN**: Executes a call or declaration centered on `topoSort`.
  **L75 CN**: 执行以 `topoSort` 为核心的调用或声明。
- **L76 EN**: Blank line separating nearby declarations or logic blocks.
  **L76 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L77 EN**: Comment explains nearby logic, invariants, or intent: `Input tensors and associated loop to level maps.`.
  **L77 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Input tensors and associated loop to level maps.`。
- **L78 EN**: Executes a standalone statement or declaration: `SmallVector<Value> ins;`.
  **L78 CN**: 执行一条独立语句或声明：`SmallVector<Value> ins;`。
- **L79 EN**: Executes a standalone statement or declaration: `SmallVector<AffineMap> loop2InsLvl;`.
  **L79 CN**: 执行一条独立语句或声明：`SmallVector<AffineMap> loop2InsLvl;`。
- **L80 EN**: Blank line separating nearby declarations or logic blocks.
  **L80 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 81-96

````cpp
  // Output tensor and associated loop to level map.
  Value out;
  AffineMap loop2OutLvl;

  // Loop itation types;
  SmallVector<utils::IteratorType> iterTypes;

  // Adjacency matrix that represents the iteration graph.
  std::vector<std::vector<bool>> itGraph;

  // InDegree used for topo sort.
  std::vector<unsigned> inDegree;

  // Loop ordering strategy.
  sparse_tensor::LoopOrderingStrategy strategy;
};
````
- **L81 EN**: Comment explains nearby logic, invariants, or intent: `Output tensor and associated loop to level map.`.
  **L81 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output tensor and associated loop to level map.`。
- **L82 EN**: Executes a standalone statement or declaration: `Value out;`.
  **L82 CN**: 执行一条独立语句或声明：`Value out;`。
- **L83 EN**: Executes a standalone statement or declaration: `AffineMap loop2OutLvl;`.
  **L83 CN**: 执行一条独立语句或声明：`AffineMap loop2OutLvl;`。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L85 EN**: Comment explains nearby logic, invariants, or intent: `Loop itation types;`.
  **L85 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Loop itation types;`。
- **L86 EN**: Executes a standalone statement or declaration: `SmallVector<utils::IteratorType> iterTypes;`.
  **L86 CN**: 执行一条独立语句或声明：`SmallVector<utils::IteratorType> iterTypes;`。
- **L87 EN**: Blank line separating nearby declarations or logic blocks.
  **L87 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L88 EN**: Comment explains nearby logic, invariants, or intent: `Adjacency matrix that represents the iteration graph.`.
  **L88 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Adjacency matrix that represents the iteration graph.`。
- **L89 EN**: Executes a standalone statement or declaration: `std::vector<std::vector<bool>> itGraph;`.
  **L89 CN**: 执行一条独立语句或声明：`std::vector<std::vector<bool>> itGraph;`。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L91 EN**: Comment explains nearby logic, invariants, or intent: `InDegree used for topo sort.`.
  **L91 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`InDegree used for topo sort.`。
- **L92 EN**: Executes a standalone statement or declaration: `std::vector<unsigned> inDegree;`.
  **L92 CN**: 执行一条独立语句或声明：`std::vector<unsigned> inDegree;`。
- **L93 EN**: Blank line separating nearby declarations or logic blocks.
  **L93 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L94 EN**: Comment explains nearby logic, invariants, or intent: `Loop ordering strategy.`.
  **L94 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Loop ordering strategy.`。
- **L95 EN**: Executes a standalone statement or declaration: `sparse_tensor::LoopOrderingStrategy strategy;`.
  **L95 CN**: 执行一条独立语句或声明：`sparse_tensor::LoopOrderingStrategy strategy;`。
- **L96 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L96 CN**: 结束当前声明作用域，例如类、结构体或枚举。

### Lines 97-101

````cpp

} // namespace sparse_tensor
} // namespace mlir

#endif // MLIR_DIALECT_SPARSETENSOR_TRANSFORMS_UTILS_ITERATIONGRAPHSORTER_H_
````
- **L97 EN**: Blank line separating nearby declarations or logic blocks.
  **L97 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L98 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace sparse_tensor`.
  **L98 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace sparse_tensor`。
- **L99 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace mlir`.
  **L99 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace mlir`。
- **L100 EN**: Blank line separating nearby declarations or logic blocks.
  **L100 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L101 EN**: Closes the current preprocessor conditional block.
  **L101 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Sparse tensor compilation / 稀疏张量编译**
- **Rewrite-driven lowering / 基于重写的 lowering**
- **Shared helper utilities / 共享辅助工具**
- **Dialect definition and registration / 方言定义与注册**
- **Type-system modeling / 类型系统建模**
- **SSA value representation / SSA 值表示**
- **Tensor-level abstraction / 张量层抽象**
- **Transform dialect orchestration / Transform 方言编排**
- **Sparse tensor abstraction / 稀疏张量抽象**
- **Vector-level optimization and lowering / 向量级优化与 lowering**

## Dependencies / 依赖关系

- `mlir/Dialect/SparseTensor/Transforms/Passes.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/IR/AffineMap.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
