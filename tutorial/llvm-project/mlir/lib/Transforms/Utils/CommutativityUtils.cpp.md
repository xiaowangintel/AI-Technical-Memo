# CommutativityUtils.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Transforms/Utils/CommutativityUtils.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements a commutativity utility pattern and a function to populate this pattern. The function is intended to be used inside passes to simplify the matching of commutative operations by fixing the order of their operands.
  - **CN**: 实现核心 MLIR 变换 Pass 与变换辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- CommutativityUtils.cpp - Commutativity utilities ---------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-14
```cpp
//
// This file implements a commutativity utility pattern and a function to
// populate this pattern. The function is intended to be used inside passes to
// simplify the matching of commutative operations by fixing the order of their
// operands.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 15-21
```cpp

#include "mlir/Transforms/CommutativityUtils.h"

#include <queue>

using namespace mlir;

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Transforms/CommutativityUtils.h`, `queue`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Transforms/CommutativityUtils.h`, `queue`。

### Lines 22-28
```cpp
/// The possible "types" of ancestors. Here, an ancestor is an op or a block
/// argument present in the backward slice of a value.
enum AncestorType {
  /// Pertains to a block argument.
  BLOCK_ARGUMENT,

  /// Pertains to a non-constant-like op.
```
- **EN**: Introduces declarations for `AncestorType`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `AncestorType` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 29-35
```cpp
  NON_CONSTANT_OP,

  /// Pertains to a constant-like op.
  CONSTANT_OP
};

/// Stores the "key" associated with an ancestor.
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 36-42
```cpp
struct AncestorKey {
  /// Holds `BLOCK_ARGUMENT`, `NON_CONSTANT_OP`, or `CONSTANT_OP`, depending on
  /// the ancestor.
  AncestorType type;

  /// Holds the op name of the ancestor if its `type` is `NON_CONSTANT_OP` or
  /// `CONSTANT_OP`. Else, holds "".
```
- **EN**: Introduces declarations for `AncestorKey`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `AncestorKey` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 43-55
```cpp
  StringRef opName;

  /// Constructor for `AncestorKey`.
  AncestorKey(Operation *op) {
    if (!op) {
      type = BLOCK_ARGUMENT;
    } else {
      type =
          op->hasTrait<OpTrait::ConstantLike>() ? CONSTANT_OP : NON_CONSTANT_OP;
      opName = op->getName().getStringRef();
    }
  }

```
- **EN**: Implements logic around `AncestorKey`, `ConstantLike>`, `getName`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `AncestorKey`、`ConstantLike>`、`getName` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 56-62
```cpp
  /// Overloaded operator `<` for `AncestorKey`.
  ///
  /// AncestorKeys of type `BLOCK_ARGUMENT` are considered the smallest, those
  /// of type `CONSTANT_OP`, the largest, and `NON_CONSTANT_OP` types come in
  /// between. Within the types `NON_CONSTANT_OP` and `CONSTANT_OP`, the smaller
  /// ones are the ones with smaller op names (lexicographically).
  ///
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 63-72
```cpp
  /// TODO: Include other information like attributes, value type, etc., to
  /// enhance this comparison. For example, currently this comparison doesn't
  /// differentiate between `cmpi sle` and `cmpi sgt` or `addi (in i32)` and
  /// `addi (in i64)`. Such an enhancement should only be done if the need
  /// arises.
  bool operator<(const AncestorKey &key) const {
    return std::tie(type, opName) < std::tie(key.type, key.opName);
  }
};

```
- **EN**: Implements logic around `operator`, `tie`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `operator`、`tie` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 73-79
```cpp
/// Stores a commutative operand along with its BFS traversal information.
struct CommutativeOperand {
  /// Stores the operand.
  Value operand;

  /// Stores the queue of ancestors of the operand's BFS traversal at a
  /// particular point in time.
```
- **EN**: Introduces declarations for `CommutativeOperand`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `CommutativeOperand` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 80-86
```cpp
  std::queue<Operation *> ancestorQueue;

  /// Stores the list of ancestors that have been visited by the BFS traversal
  /// at a particular point in time.
  DenseSet<Operation *> visitedAncestors;

  /// Stores the operand's "key". This "key" is defined as a list of the
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 87-93
```cpp
  /// "AncestorKeys" associated with the ancestors of this operand, in a
  /// breadth-first order.
  ///
  /// So, if an operand, say `A`, was produced as follows:
  ///
  /// `<block argument>`  `<block argument>`
  ///             \          /
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 94-100
```cpp
  ///              \        /
  ///             `arith.subi`           `arith.constant`
  ///                       \            /
  ///                        `arith.addi`
  ///                              |
  ///                         returns `A`
  ///
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 101-107
```cpp
  /// Then, the ancestors of `A`, in the breadth-first order are:
  /// `arith.addi`, `arith.subi`, `arith.constant`, `<block argument>`, and
  /// `<block argument>`.
  ///
  /// Thus, the "key" associated with operand `A` is:
  /// {
  ///  {type: `NON_CONSTANT_OP`, opName: "arith.addi"},
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 108-114
```cpp
  ///  {type: `NON_CONSTANT_OP`, opName: "arith.subi"},
  ///  {type: `CONSTANT_OP`, opName: "arith.constant"},
  ///  {type: `BLOCK_ARGUMENT`, opName: ""},
  ///  {type: `BLOCK_ARGUMENT`, opName: ""}
  /// }
  SmallVector<AncestorKey, 4> key;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 115-123
```cpp
  /// Push an ancestor into the operand's BFS information structure. This
  /// entails it being pushed into the queue (always) and inserted into the
  /// "visited ancestors" list (iff it is an op rather than a block argument).
  void pushAncestor(Operation *op) {
    ancestorQueue.push(op);
    if (op)
      visitedAncestors.insert(op);
  }

```
- **EN**: Implements logic around `pushAncestor`, `push`, `insert`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `pushAncestor`、`push`、`insert` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 124-130
```cpp
  /// Refresh the key.
  ///
  /// Refreshing a key entails making it up-to-date with the operand's BFS
  /// traversal that has happened till that point in time, i.e, appending the
  /// existing key with the front ancestor's "AncestorKey". Note that a key
  /// directly reflects the BFS and thus needs to be refreshed during the
  /// progression of the traversal.
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 131-139
```cpp
  void refreshKey() {
    if (ancestorQueue.empty())
      return;

    Operation *frontAncestor = ancestorQueue.front();
    AncestorKey frontAncestorKey(frontAncestor);
    key.push_back(frontAncestorKey);
  }

```
- **EN**: Implements logic around `refreshKey`, `empty`, `front`, `frontAncestorKey`, and 1 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `refreshKey`、`empty`、`front`、`frontAncestorKey` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 140-153
```cpp
  /// Pop the front ancestor, if any, from the queue and then push its adjacent
  /// unvisited ancestors, if any, to the queue (this is the main body of the
  /// BFS algorithm).
  void popFrontAndPushAdjacentUnvisitedAncestors() {
    if (ancestorQueue.empty())
      return;
    Operation *frontAncestor = ancestorQueue.front();
    ancestorQueue.pop();
    if (!frontAncestor)
      return;
    for (Value operand : frontAncestor->getOperands()) {
      Operation *operandDefOp = operand.getDefiningOp();
      if (!operandDefOp || !visitedAncestors.contains(operandDefOp))
        pushAncestor(operandDefOp);
```
- **EN**: Implements logic around `popFrontAndPushAdjacentUnvisitedAncestors`, `empty`, `front`, `pop`, and 4 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `popFrontAndPushAdjacentUnvisitedAncestors`、`empty`、`front`、`pop` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 154-160
```cpp
    }
  }
};

/// Sorts the operands of `op` in ascending order of the "key" associated with
/// each operand iff `op` is commutative. This is a stable sort.
///
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 161-167
```cpp
/// After the application of this pattern, since the commutative operands now
/// have a deterministic order in which they occur in an op, the matching of
/// large DAGs becomes much simpler, i.e., requires much less number of checks
/// to be written by a user in her/his pattern matching function.
///
/// Some examples of such a sorting:
///
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 168-174
```cpp
/// Assume that the sorting is being applied to `foo.commutative`, which is a
/// commutative op.
///
/// Example 1:
///
/// %1 = foo.const 0
/// %2 = foo.mul <block argument>, <block argument>
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 175-181
```cpp
/// %3 = foo.commutative %1, %2
///
/// Here,
/// 1. The key associated with %1 is:
///     `{
///       {CONSTANT_OP, "foo.const"}
///      }`
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 182-188
```cpp
/// 2. The key associated with %2 is:
///     `{
///       {NON_CONSTANT_OP, "foo.mul"},
///       {BLOCK_ARGUMENT, ""},
///       {BLOCK_ARGUMENT, ""}
///      }`
///
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 189-195
```cpp
/// The key of %2 < the key of %1
/// Thus, the sorted `foo.commutative` is:
/// %3 = foo.commutative %2, %1
///
/// Example 2:
///
/// %1 = foo.const 0
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 196-202
```cpp
/// %2 = foo.mul <block argument>, <block argument>
/// %3 = foo.mul %2, %1
/// %4 = foo.add %2, %1
/// %5 = foo.commutative %1, %2, %3, %4
///
/// Here,
/// 1. The key associated with %1 is:
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 203-209
```cpp
///     `{
///       {CONSTANT_OP, "foo.const"}
///      }`
/// 2. The key associated with %2 is:
///     `{
///       {NON_CONSTANT_OP, "foo.mul"},
///       {BLOCK_ARGUMENT, ""}
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 210-216
```cpp
///      }`
/// 3. The key associated with %3 is:
///     `{
///       {NON_CONSTANT_OP, "foo.mul"},
///       {NON_CONSTANT_OP, "foo.mul"},
///       {CONSTANT_OP, "foo.const"},
///       {BLOCK_ARGUMENT, ""},
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 217-223
```cpp
///       {BLOCK_ARGUMENT, ""}
///      }`
/// 4. The key associated with %4 is:
///     `{
///       {NON_CONSTANT_OP, "foo.add"},
///       {NON_CONSTANT_OP, "foo.mul"},
///       {CONSTANT_OP, "foo.const"},
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 224-230
```cpp
///       {BLOCK_ARGUMENT, ""},
///       {BLOCK_ARGUMENT, ""}
///      }`
///
/// Thus, the sorted `foo.commutative` is:
/// %5 = foo.commutative %4, %3, %2, %1
class SortCommutativeOperands : public RewritePattern {
```
- **EN**: Introduces declarations for `SortCommutativeOperands`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `SortCommutativeOperands` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 231-244
```cpp
public:
  SortCommutativeOperands(MLIRContext *context)
      : RewritePattern(MatchAnyOpTypeTag(), /*benefit=*/5, context) {}
  LogicalResult matchAndRewrite(Operation *op,
                                PatternRewriter &rewriter) const override {
    // Custom comparator for two commutative operands, which returns true iff
    // the "key" of `constCommOperandA` < the "key" of `constCommOperandB`,
    // i.e.,
    // 1. In the first unequal pair of corresponding AncestorKeys, the
    // AncestorKey in `constCommOperandA` is smaller, or,
    // 2. Both the AncestorKeys in every pair are the same and the size of
    // `constCommOperandA`'s "key" is smaller.
    auto commutativeOperandComparator =
        [](const std::unique_ptr<CommutativeOperand> &constCommOperandA,
```
- **EN**: Implements logic around `SortCommutativeOperands`, `RewritePattern`, `matchAndRewrite`; this block makes success/failure or diagnostics explicit through MLIR result utilities; relies on MLIR pattern rewriting infrastructure; manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `SortCommutativeOperands`、`RewritePattern`、`matchAndRewrite` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并依赖 MLIR 模式重写基础设施，并操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 245-255
```cpp
           const std::unique_ptr<CommutativeOperand> &constCommOperandB) {
          if (constCommOperandA->operand == constCommOperandB->operand)
            return false;

          auto &commOperandA =
              const_cast<std::unique_ptr<CommutativeOperand> &>(
                  constCommOperandA);
          auto &commOperandB =
              const_cast<std::unique_ptr<CommutativeOperand> &>(
                  constCommOperandB);

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 256-269
```cpp
          // Iteratively perform the BFS's of both operands until an order among
          // them can be determined.
          unsigned keyIndex = 0;
          while (true) {
            if (commOperandA->key.size() <= keyIndex) {
              if (commOperandA->ancestorQueue.empty())
                return true;
              commOperandA->popFrontAndPushAdjacentUnvisitedAncestors();
              commOperandA->refreshKey();
            }
            if (commOperandB->key.size() <= keyIndex) {
              if (commOperandB->ancestorQueue.empty())
                return false;
              commOperandB->popFrontAndPushAdjacentUnvisitedAncestors();
```
- **EN**: Implements logic around `size`, `empty`, `popFrontAndPushAdjacentUnvisitedAncestors`, `refreshKey`; this block implements transformation or simplification logic.
- **CN**: 围绕 `size`、`empty`、`popFrontAndPushAdjacentUnvisitedAncestors`、`refreshKey` 实现具体逻辑；该代码块实现变换或简化逻辑。

### Lines 270-282
```cpp
              commOperandB->refreshKey();
            }
            if (commOperandA->ancestorQueue.empty() ||
                commOperandB->ancestorQueue.empty())
              return commOperandA->key.size() < commOperandB->key.size();
            if (commOperandA->key[keyIndex] < commOperandB->key[keyIndex])
              return true;
            if (commOperandB->key[keyIndex] < commOperandA->key[keyIndex])
              return false;
            keyIndex++;
          }
        };

```
- **EN**: Implements logic around `refreshKey`, `empty`, `size`; this block implements transformation or simplification logic.
- **CN**: 围绕 `refreshKey`、`empty`、`size` 实现具体逻辑；该代码块实现变换或简化逻辑。

### Lines 283-296
```cpp
    // If `op` is not commutative, do nothing.
    if (!op->hasTrait<OpTrait::IsCommutative>())
      return failure();

    // Populate the list of commutative operands.
    SmallVector<Value, 2> operands = op->getOperands();
    SmallVector<std::unique_ptr<CommutativeOperand>, 2> commOperands;
    for (Value operand : operands) {
      std::unique_ptr<CommutativeOperand> commOperand =
          std::make_unique<CommutativeOperand>();
      commOperand->operand = operand;
      commOperand->pushAncestor(operand.getDefiningOp());
      commOperand->refreshKey();
      commOperands.push_back(std::move(commOperand));
```
- **EN**: Implements logic around `IsCommutative>`, `failure`, `getOperands`, `make_unique`, and 3 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `IsCommutative>`、`failure`、`getOperands`、`make_unique` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 297-310
```cpp
    }

    // Sort the operands.
    llvm::stable_sort(commOperands, commutativeOperandComparator);
    SmallVector<Value, 2> sortedOperands;
    for (const std::unique_ptr<CommutativeOperand> &commOperand : commOperands)
      sortedOperands.push_back(commOperand->operand);
    if (sortedOperands == operands)
      return failure();
    rewriter.modifyOpInPlace(op, [&] { op->setOperands(sortedOperands); });
    return success();
  }
};

```
- **EN**: Implements logic around `stable_sort`, `push_back`, `failure`, `modifyOpInPlace`, and 1 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `stable_sort`、`push_back`、`failure`、`modifyOpInPlace` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 311-313
```cpp
void mlir::populateCommutativityUtilsPatterns(RewritePatternSet &patterns) {
  patterns.add<SortCommutativeOperands>(patterns.getContext());
}
```
- **EN**: Implements logic around `populateCommutativityUtilsPatterns`, `add`; this block implements transformation or simplification logic.
- **CN**: 围绕 `populateCommutativityUtilsPatterns`、`add` 实现具体逻辑；该代码块实现变换或简化逻辑。

## Key Concepts / 关键概念

- **IR transformation passes / IR 变换 Pass**:
  - **EN**: Applies canonicalization, dataflow-aware simplification, or structural rewrites to MLIR.
  - **CN**: 对 MLIR 应用规范化、数据流感知简化或结构性重写。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **Pattern rewriting / 模式重写**:
  - **EN**: The implementation uses rewrite patterns to match and transform IR.
  - **CN**: 该实现使用重写模式来匹配并变换 IR。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Transforms/CommutativityUtils.h`
- **Standard-library headers / 标准库头文件**: `<queue>`
- **Subsystem categories / 子系统类别**: core transformation utilities / 核心变换工具 (1)
