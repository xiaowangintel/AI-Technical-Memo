# loopnest_randomization.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/loopnest_randomization.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements loop-nest scheduling and transformation utilities for Tensor Expression IR.
- **Purpose (CN)**: 实现 Tensor Expression IR 的循环嵌套调度与变换工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```cpp
#include <algorithm>
#include <iostream>
#include <random>
#include <stdexcept>
#include <typeinfo>
#include <unordered_map>
#include <unordered_set>
#include <vector>

#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/jit_opt_limit.h>
#include <torch/csrc/jit/tensorexpr/ir_simplifier.h>
#include <torch/csrc/jit/tensorexpr/loopnest.h>
#include <torch/csrc/jit/tensorexpr/loopnest_randomization.h>

namespace torch::jit::tensorexpr {

namespace randomization_helper {
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/jit_log.h, torch/csrc/jit/jit_opt_limit.h, torch/csrc/jit/tensorexpr/ir_simplifier.h, and 2 more; standard-library headers such as algorithm, iostream, random, and 5 more. The namespace declarations place the code inside torch::jit::tensorexpr, randomization_helper, matching the surrounding JIT subsystem.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/jit_log.h、torch/csrc/jit/jit_opt_limit.h、torch/csrc/jit/tensorexpr/ir_simplifier.h 等共 5 项；标准库头文件，如 algorithm、iostream、random 等共 8 项。 命名空间声明把代码放入 torch::jit::tensorexpr、randomization_helper 中，与周边 JIT 子系统保持一致。

### Lines 20-34
```cpp
static int64_t max_transformations(int n_max_transforms) {
  // Reuse the env variable PYTORCH_JIT_OPT_LIMIT to control the max number of
  // transformations.  Example - set the env variable
  // PYTORCH_JIT_OPT_LIMIT="loopnest_randomization=10" to set max
  // transformations to 10.  This can be helpful in gradually reducing the
  // number of transformations when we see an error.
  if (!JIT_OPT_ALLOWED) {
    return n_max_transforms;
  }
  int max_transforms = 1;
  while (JIT_OPT_ALLOWED && max_transforms < n_max_transforms) {
    max_transforms++;
  }
  return max_transforms;
}
```
- **EN**: This chunk defines `max_transformations`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `max_transformations`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 36-59
```cpp
static std::vector<std::vector<ForPtr>> GetAllPerfectlyNestedLoopNests(
    std::vector<ForPtr> loops) {
  // Find the first set of loops that can be reordered
  std::vector<std::vector<ForPtr>> all_nested_loops;
  std::vector<ForPtr> nested_loops;
  if (loops.empty()) {
    return all_nested_loops;
  }
  nested_loops.push_back(loops[0]);
  for (size_t i = 1; i < loops.size(); i++) {
    auto last_loop = nested_loops.back();
    auto next_loop = loops[i];
    if (last_loop->body()->nstmts() == 1 &&
        last_loop->body()->front() == next_loop) {
      nested_loops.push_back(next_loop);
    } else {
      if (nested_loops.size() > 1) {
        all_nested_loops.push_back(nested_loops);
      }
      nested_loops.clear();
      nested_loops.push_back(next_loop);
    }
  }
  return all_nested_loops;
```
- **EN**: This chunk defines `GetAllPerfectlyNestedLoopNests`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `GetAllPerfectlyNestedLoopNests`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 60-82
```cpp
}

template <typename T>
static std::tuple<std::vector<T>, std::vector<int>> select_n_randomly(
    std::vector<T>& objects,
    int n,
    std::default_random_engine& random_engine) {
  std::vector<int> indices(objects.size());
  std::iota(indices.begin(), indices.end(), 0);
  std::shuffle(indices.begin(), indices.end(), random_engine);

  std::vector<T> selected_objects;
  std::vector<int> selected_indices;
  if (static_cast<int>(indices.size()) < n) {
    return std::make_tuple(selected_objects, selected_indices);
  }
  for (int i = 0; i < n; i++) {
    int index = indices[i];
    selected_indices.push_back(index);
    selected_objects.push_back(objects[index]);
  }
  return std::make_tuple(selected_objects, selected_indices);
}
```
- **EN**: This chunk defines `indices`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `indices`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 84-100
```cpp
static int find_factor(const ForPtr& loop) {
  // Find valid factors
  ExprPtr loop_stop = loop->stop();
  auto loop_imm = intValue(loop_stop);
  if (loop_imm) {
    int loop_bound = *loop_imm;
    int factor = rand() % (loop_bound - 1) + 1;
    return factor;
  }
  return -1;
}

static void printHistory(int index, std::string message) {
  message = "Random Transform Sequence - Transformations[" +
      std::to_string(index) + "] = " + message;
  GRAPH_DEBUG(message);
}
```
- **EN**: This chunk defines `printHistory`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `printHistory`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 102-124
```cpp
template <typename T>
static std::string join(std::vector<T> indices, char sep = ',') {
  std::string s;
  for (const auto& index : indices) {
    s += std::to_string(index) + sep;
  }
  return s;
}

static std::string join(
    const std::vector<std::string>& indices,
    char sep = ',') {
  std::string s;
  for (const auto& index : indices) {
    s += index + sep;
  }
  return s;
}
template <typename T>
static std::string indexOf(const std::vector<T>& objects, const T& object) {
  return std::to_string(std::distance(
      objects.begin(), std::find(objects.begin(), objects.end(), object)));
}
```
- **EN**: This chunk defines `indexOf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `indexOf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 126-149
```cpp
} // namespace randomization_helper

void loopnestRandomization(int64_t seed, LoopNest& l) {
  // This is to help with deterministic testing of randomized infrastructure.
  // When seed value is 1, we perform preset loop transformations. This allows
  // testing of interface.
  if (seed == 1) {
    l.simplify();
    return;
  }

  std::default_random_engine random_engine(seed);
  std::srand(seed);
  // Set the maximum allowed number of transformations beyond which it is hard
  // to track and debug. Arbitrarily choosing 20 as maximum number.
  int max_allowed_transformations = 20;
  int n_transforms = randomization_helper::max_transformations(
      std::rand() % max_allowed_transformations);
  std::string message;
  // clang-format off
  //   Transformations list:
  //
  //       StmtPtr simplify();
  //       bool computeInline(BufPtr b);
```
- **EN**: This chunk defines `random_engine`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `random_engine`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 150-173
```cpp
  //       void inlineIntermediateBufs(bool allow_duplicated_work);
  //       bool optimizeConditionals();
  //       static void splitWithTail(ForPtr f, int factor);
  //       static void splitWithMask(ForPtr f, int factor);
  //       static std::vector<ForPtr> distributeLoop(ForPtr loop, const std::unordered_set<StmtPtr>& pivots);
  //       static std::vector<ForPtr> distributeLoop(ForPtr loop);
  //       static std::vector<ForPtr> distributeLoopAndParents(ForPtr loop);
  //       static std::vector<ForPtr> distributeLoopOverInnerLoops(ForPtr loop);
  //       static std::vector<ForPtr> distributeLoopAndParentsOverInnerLoops(ForPtr loop);
  //       static bool fuseLoops(const std::vector<ForPtr>& loops, ForPtr* fused);
  //       static void reorderAxis(ForPtr a, ForPtr b);
  //       static std::vector<ForPtr> reorder(const std::vector<ForPtr>& loops, const std::vector<size_t>& permutation);
  //       ForPtr tile(ForPtr x, ForPtr y, int x_factor, int y_factor);
  //       static void fullUnroll(ForPtr f);
  //       static bool normalize(ForPtr f);
  //       static bool flatten(const std::vector<ForPtr>& f, ForPtr* flattened);
  //       static void compressBuffer(BufPtr buf, StmtPtr stmt);
  //       static void compressAllBuffers(StmtPtr stmt);
  //       static void sliceHead(ForPtr f, int factor, ForPtr* head, ForPtr* tail);
  //       static void sliceHead(ForPtr f, int factor);
  //       static void sliceTail(ForPtr f, int factor, ForPtr* head, ForPtr* tail);
  //       static void sliceTail(ForPtr f, int factor);
  //       static AccessResult cacheAccesses(BufPtr producer, const std::string& name, StmtPtr consumer);
  //       static void computeAt(StmtPtr s, ForPtr at);
```
- **EN**: This comment block documents assumptions, constraints, or generated-code provenance that shape the implementation below.
- **CN**: 这一段注释说明了后续实现依赖的假设、约束或生成来源。

### Lines 174-197
```cpp
  //       static bool rfactor(StmtPtr s, ForPtr outer_reduction_for);
  //       static bool vectorize(ForPtr);
  //       void vectorizeInnerLoops();
  //       void eliminateDeadStores();
  //       void prepareForCodegen();
  // clang-format on
  enum TransformKind {
    SIMPLIFY = 0,
    COMPUTE_INLINE,
    INLINE_ALL,
    OPT_COND,
    SPLIT_TAIL,
    SPLIT_MASK,
    DIST1,
    DIST2,
    DIST3,
    DIST4,
    DIST5,
    FUSE_LOOPS,
    REORDER_AXIS,
    REORDER,
    TILE,
    FULL_UNROLL,
    NORMALIZE,
```
- **EN**: It introduces or extends TransformKind, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 它引入或扩展了 TransformKind，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 198-221
```cpp
    FLATTEN,
    COMPRESS_BUFFER,
    COMPRESS_ALL_BUFFERS,
    SLICE_HEAD,
    SLICE_TAIL,
    CACHE_ACCESSES,
    COMPUTE_AT,
    RFACTOR,
    VECTORIZE,
    VECTORIZE_INNER_LOOPS,
    ELIMINATE_DEAD_STORES,
    MAX_TRANSFORM,
  };
  bool can_inline = true;
  try {
    for (int n_transform = 0; n_transform < n_transforms; n_transform++) {
      int transform = std::rand() % MAX_TRANSFORM;
      switch (transform) {
        case SIMPLIFY: {
          message = "simplify();\n";
          randomization_helper::printHistory(n_transform, message);
          l.simplify();
          break;
        }
```
- **EN**: This chunk continues `TransformKind` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case.
- **CN**: 这一段延续了 `TransformKind`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。

### Lines 222-245
```cpp
        case COMPUTE_INLINE: {
          if (can_inline) {
            auto bufs = NodeFinder<Buf>::find(l.root_stmt());
            if (!bufs.empty()) {
              int buf_number = std::rand() % (int)bufs.size();
              message =
                  "computeInline(" + bufs[buf_number]->name_hint() + ");\n";
              randomization_helper::printHistory(n_transform, message);
              l.computeInline(bufs[buf_number]);
            }
          }
          break;
        }
        case INLINE_ALL: {
          if (can_inline) {
            int allow_dup = std::rand() % 2;
            message =
                "inlineIntermediateBufs(" + std::to_string(allow_dup) + ");\n";
            randomization_helper::printHistory(n_transform, message);
            l.inlineIntermediateBufs(allow_dup);
            can_inline = false;
          }
          break;
        }
```
- **EN**: This chunk continues `TransformKind` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `TransformKind`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 246-269
```cpp
        case OPT_COND: {
          message = "optimizeConditionals();\n";
          randomization_helper::printHistory(n_transform, message);
          l.optimizeConditionals();
          break;
        }
        case SPLIT_TAIL: {
          auto loops = NodeFinder<For>::find(l.root_stmt());
          if (loops.empty()) {
            break;
          }
          int loop_n = std::rand() % (int)loops.size();
          auto loop = loops[loop_n];
          int factor = (std::rand() % 20) + 1;
          message = "splitWithTail(loops[" + std::to_string(loop_n) + "], " +
              std::to_string(factor) + ");\n";
          randomization_helper::printHistory(n_transform, message);
          l.splitWithTail(loop, factor);
          break;
        }
        case SPLIT_MASK: {
          auto loops = NodeFinder<For>::find(l.root_stmt());
          if (loops.empty()) {
            break;
```
- **EN**: This chunk continues `TransformKind` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `TransformKind`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 270-293
```cpp
          }
          int loop_n = std::rand() % (int)loops.size();
          auto loop = loops[loop_n];
          int factor = (std::rand() % 20) + 1;
          message = "splitWithMask(loops[" + std::to_string(loop_n) + "], " +
              std::to_string(factor) + ")\n";
          randomization_helper::printHistory(n_transform, message);
          l.splitWithMask(loop, factor);
          break;
        }
        case DIST1: {
          auto loops = NodeFinder<For>::find(l.root_stmt());
          if (loops.empty()) {
            break;
          }
          int loop_n = std::rand() % (int)loops.size();
          auto loop = loops[loop_n];
          std::vector<StmtPtr> stmts(
              loop->body()->begin(), loop->body()->end());
          if (stmts.empty()) {
            break;
          }
          int n_pivots = (std::rand() % (int)stmts.size()) + 1;
          auto [pivots, chosen_indices] =
```
- **EN**: This chunk defines `stmts`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `stmts`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 294-311
```cpp
              randomization_helper::select_n_randomly<StmtPtr>(
                  stmts, n_pivots, random_engine);
          std::unordered_set<StmtPtr> pivots_set(pivots.begin(), pivots.end());
          message = "distributeLoop(loops[" + std::to_string(loop_n) +
              "], pivots=stmts(" + randomization_helper::join(chosen_indices) +
              "))\n";
          randomization_helper::printHistory(n_transform, message);
          l.distributeLoop(loop, pivots_set);
          break;
        }
        case DIST2: {
          auto loops = NodeFinder<For>::find(l.root_stmt());

          if (loops.empty()) {
            break;
          }
          int loop_n = std::rand() % (int)loops.size();
          auto loop = loops[loop_n];
```
- **EN**: This chunk defines `pivots_set`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `pivots_set`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 313-334
```cpp
          message = "distributeLoop(loops[" + std::to_string(loop_n) + "])\n";
          randomization_helper::printHistory(n_transform, message);
          l.distributeLoop(loop);
          break;
        }
        case DIST3: {
          auto loops = NodeFinder<For>::find(l.root_stmt());

          if (loops.empty()) {
            break;
          }
          int loop_n = std::rand() % (int)loops.size();
          auto loop = loops[loop_n];

          message = "distributeLoopAndParents(loops[" + std::to_string(loop_n) +
              "])\n";
          randomization_helper::printHistory(n_transform, message);
          l.distributeLoopAndParents(loop);
          break;
        }
        case DIST4: {
          auto loops = NodeFinder<For>::find(l.root_stmt());
```
- **EN**: This chunk continues `pivots_set` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `pivots_set`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 336-355
```cpp
          if (loops.empty()) {
            break;
          }
          int loop_n = std::rand() % (int)loops.size();
          auto loop = loops[loop_n];

          message = "distributeLoopOverInnerLoops(loops[" +
              std::to_string(loop_n) + "])\n";
          randomization_helper::printHistory(n_transform, message);
          l.distributeLoopOverInnerLoops(loop);
          break;
        }
        case DIST5: {
          auto loops = NodeFinder<For>::find(l.root_stmt());

          if (loops.empty()) {
            break;
          }
          int loop_n = std::rand() % (int)loops.size();
          auto loop = loops[loop_n];
```
- **EN**: This chunk continues `pivots_set` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `pivots_set`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 357-376
```cpp
          message = "distributeLoopAndParentsOverInnerLoops(loops[" +
              std::to_string(loop_n) + "])\n";
          randomization_helper::printHistory(n_transform, message);
          l.distributeLoopAndParentsOverInnerLoops(loop);
          break;
        }
        case FUSE_LOOPS: {
          // Get all the loops
          auto loops = NodeFinder<For>::find(l.root_stmt());
          if (loops.size() <= 1) {
            break;
          }

          // Find a random number of loops to fuse
          int num_loops_to_fuse =
              std::max(2, (std::rand() % (int)loops.size()));

          auto [loops_to_fuse, chosen_indices] =
              randomization_helper::select_n_randomly<ForPtr>(
                  loops, num_loops_to_fuse, random_engine);
```
- **EN**: This chunk continues `pivots_set` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `pivots_set`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 378-392
```cpp
          message = "fuseLoops(loops[" +
              randomization_helper::join(chosen_indices) + "], &fused_loop);\n";
          randomization_helper::printHistory(n_transform, message);
          // Fuse the loops
          ForPtr fused_loop;
          l.fuseLoops(loops_to_fuse, &fused_loop);
          break;
        }

        case REORDER_AXIS: {
          // Get all the loops
          auto loops = NodeFinder<For>::find(l.root_stmt());
          if (loops.size() <= 1) {
            break;
          }
```
- **EN**: This chunk continues `pivots_set` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `pivots_set`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 394-411
```cpp
          // Find pairs of axes that can be reordered
          std::vector<std::pair<ForPtr, ForPtr>> valid_pairs;
          for (const auto i : c10::irange(loops.size())) {
            for (const auto j : c10::irange(i + 1, loops.size())) {
              if (LoopNest::findOuterFor(loops[i], loops[j])) {
                valid_pairs.emplace_back(loops[i], loops[j]);
              }
            }
          }

          // Choose a pair randomly
          if (valid_pairs.empty()) {
            break;
          }
          int valid_pair_n = std::rand() % (int)valid_pairs.size();
          auto loop_pair = valid_pairs.at(valid_pair_n);
          auto first_loop = std::get<0>(loop_pair);
          auto second_loop = std::get<1>(loop_pair);
```
- **EN**: This chunk continues `pivots_set` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `pivots_set`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 413-432
```cpp
          std::string first_index =
              randomization_helper::indexOf(loops, first_loop);
          std::string second_index =
              randomization_helper::indexOf(loops, second_loop);
          message = "reorderAxis(loops[";
          message += first_index;
          message += "], loops[";
          message += second_index + "]);\n";
          randomization_helper::printHistory(n_transform, message);
          // reorder the axis
          l.reorderAxis(first_loop, second_loop);
          break;
        }

        case REORDER: {
          // Get all the loops
          auto loops = NodeFinder<For>::find(l.root_stmt());
          if (loops.size() <= 1) {
            break;
          }
```
- **EN**: This chunk continues `pivots_set` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `pivots_set`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 434-448
```cpp
          // Find all perfectly nested loop nests
          auto all_nested_loops =
              randomization_helper::GetAllPerfectlyNestedLoopNests(loops);
          if (all_nested_loops.empty()) {
            break;
          }

          // Randomly pick a set of consecutive loops to reorder
          int index = rand() % (int)all_nested_loops.size();
          auto nested_loops = all_nested_loops.at(index);

          // Create a random permutation for reordering
          std::vector<size_t> permutation(nested_loops.size());
          std::iota(permutation.begin(), permutation.end(), 0);
          std::shuffle(permutation.begin(), permutation.end(), random_engine);
```
- **EN**: This chunk defines `permutation`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `permutation`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 450-470
```cpp
          // Generate a good history message
          std::vector<std::string> indices;
          indices.reserve(nested_loops.size());
          for (const auto& l : nested_loops) {
            indices.push_back(randomization_helper::indexOf(loops, l));
          }
          message = "reorder(loops[" + randomization_helper::join(indices) +
              "], permutation=[" + randomization_helper::join(permutation) +
              "]);\n";
          randomization_helper::printHistory(n_transform, message);
          // reorder
          l.reorder(nested_loops, permutation);
          break;
        }

        case TILE: {
          // Get all the loops
          auto loops = NodeFinder<For>::find(l.root_stmt());
          if (loops.size() <= 1) {
            break;
          }
```
- **EN**: This chunk continues `permutation` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `permutation`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 472-489
```cpp
          // Tile needs two perfectly nested loops. To find such loops, we find
          // all perfectly nested loop nests, randomly pick one of them, and
          // randomly pick 2 consecutive loops in that loop nest.
          // Find all perfectly nested loop nests
          auto all_nested_loops =
              randomization_helper::GetAllPerfectlyNestedLoopNests(loops);
          if (all_nested_loops.empty()) {
            break;
          }

          int index = rand() % (int)all_nested_loops.size();
          auto const& nested_loops = all_nested_loops.at(index);
          if (nested_loops.size() < 2) {
            break;
          }
          int loop_number = rand() % ((int)nested_loops.size() - 1);
          auto x_loop = nested_loops.at(loop_number);
          auto y_loop = nested_loops.at(loop_number + 1);
```
- **EN**: This chunk continues `permutation` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `permutation`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 491-511
```cpp
          int x_factor = randomization_helper::find_factor(x_loop);
          int y_factor = randomization_helper::find_factor(y_loop);
          if (x_factor == -1 || y_factor == -1) {
            break;
          }

          std::string x_loop_index =
              randomization_helper::indexOf(loops, x_loop);
          std::string y_loop_index =
              randomization_helper::indexOf(loops, y_loop);
          message = "tile(loops[";
          message += x_loop_index;
          message += "], loops[";
          message += y_loop_index + "], ";
          message += std::to_string(x_factor);
          message += ", " + std::to_string(y_factor) + ");\n";
          randomization_helper::printHistory(n_transform, message);
          // tile
          l.tile(x_loop, y_loop, x_factor, y_factor);
          break;
        }
```
- **EN**: This chunk continues `permutation` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `permutation`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 513-533
```cpp
        case FULL_UNROLL: {
          auto loops = NodeFinder<For>::find(l.root_stmt());
          if (loops.empty()) {
            break;
          }
          int loop_n = std::rand() % (int)loops.size();
          auto loop = loops[loop_n];

          message = "fullUnroll(loops[" + std::to_string(loop_n) + "]);\n";
          randomization_helper::printHistory(n_transform, message);
          LoopNest::fullUnroll(loop);
          break;
        }

        case NORMALIZE: {
          auto loops = NodeFinder<For>::find(l.root_stmt());
          if (loops.empty()) {
            break;
          }
          int loop_n = std::rand() % (int)loops.size();
          auto loop = loops[loop_n];
```
- **EN**: This chunk continues `permutation` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `permutation`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 535-557
```cpp
          message = "normalize(loops[" + std::to_string(loop_n) + "]);\n";
          randomization_helper::printHistory(n_transform, message);
          l.normalize(loop);
          break;
        }

        case FLATTEN: {
          // Get all the loops
          auto loops = NodeFinder<For>::find(l.root_stmt());
          if (loops.size() <= 1) {
            break;
          }

          // Find all perfectly nested loop nests
          auto all_nested_loops =
              randomization_helper::GetAllPerfectlyNestedLoopNests(loops);
          if (all_nested_loops.empty()) {
            break;
          }

          // Randomly pick a set of consecutive loops to flatten
          int index = rand() % (int)all_nested_loops.size();
          auto const& nested_loops = all_nested_loops.at(index);
```
- **EN**: This chunk continues `permutation` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `permutation`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 559-576
```cpp
          // Generate a good history message
          std::vector<std::string> indices;
          indices.reserve(nested_loops.size());
          for (const auto& l : nested_loops) {
            indices.push_back(randomization_helper::indexOf(loops, l));
          }
          message =
              "flatten(loops[" + randomization_helper::join(indices) + "]);\n";
          randomization_helper::printHistory(n_transform, message);
          // flatten
          l.flatten(nested_loops);
          break;
        }

        case COMPRESS_BUFFER: {
          auto buffers = NodeFinder<Buf>::find(l.root_stmt());
          int buffer_n = std::rand() % (int)buffers.size();
          auto buffer = buffers[buffer_n];
```
- **EN**: This chunk continues `permutation` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段延续了 `permutation`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 578-599
```cpp
          message = "compressBuffer(buffers[" + std::to_string(buffer_n) +
              "], l.root_stmt());\n";
          randomization_helper::printHistory(n_transform, message);
          l.compressBuffer(buffer, l.root_stmt());
          break;
        }

        case COMPRESS_ALL_BUFFERS: {
          message = "compressAllBuffers(l.root_stmt());\n";
          randomization_helper::printHistory(n_transform, message);
          l.compressAllBuffers(l.root_stmt());
          break;
        }

        case SLICE_HEAD: {
          // Get all the loops
          auto loops = NodeFinder<For>::find(l.root_stmt());
          if (loops.empty()) {
            break;
          }
          int loop_n = std::rand() % (int)loops.size();
          auto loop = loops[loop_n];
```
- **EN**: This chunk continues `permutation` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `permutation`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 601-618
```cpp
          int factor = randomization_helper::find_factor(loop);
          if (factor == -1) {
            break;
          }
          message = "sliceHead(loops[" + std::to_string(loop_n) + "]);\n";
          randomization_helper::printHistory(n_transform, message);
          l.sliceHead(loop, factor);
          break;
        }

        case SLICE_TAIL: {
          // Get all the loops
          auto loops = NodeFinder<For>::find(l.root_stmt());
          if (loops.empty()) {
            break;
          }
          int loop_n = std::rand() % (int)loops.size();
          auto loop = loops[loop_n];
```
- **EN**: This chunk continues `permutation` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `permutation`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 620-643
```cpp
          int factor = randomization_helper::find_factor(loop);
          if (factor == -1) {
            break;
          }
          message = "sliceTail(loops[" + std::to_string(loop_n) + "]);\n";
          randomization_helper::printHistory(n_transform, message);
          l.sliceTail(loop, factor);
          break;
        }

        case CACHE_ACCESSES: {
          // TODO - Implement cache_access
          break;
        }

        case COMPUTE_AT: {
          // To find valid compute at pairs, we need to collect the producer
          // consumer pairs. For now, we do not collect all such pairs for
          // simplicity. For now, we collect producer and the immediate parent
          // loop of the consumer. We could collect all the consumer enclosing
          // loops, but then we will have to clean up the ones that are shared
          // with the producer encloser loop. Currently, we only test on the
          // immediate parent loop.
          auto buffers = BufFinder::find(l.root_stmt());
```
- **EN**: This chunk continues `permutation` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `permutation`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 644-664
```cpp
          std::vector<std::pair<StmtPtr, ForPtr>> producer_consumer_pairs;

          for (const auto& buffer : buffers) {
            auto producers = l.getAllWritesToBuf(buffer);
            auto consumers = StmtsReadingBuf::find(l.root_stmt(), buffer);
            if (producers.size() != 1 || consumers.empty()) {
              continue;
            }

            for (const auto& producer : producers) {
              for (const auto& consumer : consumers) {
                auto parent_loop = LoopNest::getParentLoop(consumer);
                auto pc_pair = std::make_pair(producer, parent_loop);
                producer_consumer_pairs.push_back(pc_pair);
              }
            }
          }

          if (producer_consumer_pairs.empty()) {
            break;
          }
```
- **EN**: This chunk continues `permutation` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `permutation`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 666-686
```cpp
          // Choose a random pair
          int pair_n = std::rand() % (int)producer_consumer_pairs.size();
          auto pc_pair = producer_consumer_pairs.at(pair_n);
          auto store = std::get<0>(pc_pair);
          auto for_ptr = std::get<1>(pc_pair);

          // TODO - come up with better message
          message = "computeAt(....);\n";
          randomization_helper::printHistory(n_transform, message);
          l.computeAt(store, for_ptr);
          break;
        }

        case RFACTOR: {
          // TODO - Implement rfactor
          break;
        }

        case VECTORIZE: {
          auto loops = NodeFinder<For>::find(l.root_stmt());
          std::vector<ForPtr> innermost_loops;
```
- **EN**: This chunk continues `permutation` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `permutation`，进一步展开其内部控制流或数据流转。

### Lines 688-707
```cpp
          for (const auto& loop : loops) {
            bool containsSubLoops = false;
            if (BlockPtr body = to<Block>(loop->body())) {
              for (const StmtPtr& stmt : *body) {
                if (ForPtr f2 = to<For>(stmt)) {
                  containsSubLoops = true;
                }
              }
            }

            if (!containsSubLoops) {
              innermost_loops.push_back(loop);
            }
          }

          if (innermost_loops.empty()) {
            break;
          }
          int loop_n = std::rand() % (int)innermost_loops.size();
          auto loop = innermost_loops[loop_n];
```
- **EN**: This chunk continues `permutation` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `permutation`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 709-727
```cpp
          message = "vectorize(loops[" + std::to_string(loop_n) + "]);\n";
          randomization_helper::printHistory(n_transform, message);
          l.vectorize(loop);
          break;
        }

        case VECTORIZE_INNER_LOOPS: {
          message = "vectorizeInnerLoops();\n";
          randomization_helper::printHistory(n_transform, message);
          l.vectorizeInnerLoops();
          break;
        }

        case ELIMINATE_DEAD_STORES: {
          message = "eliminateDeadStores();\n";
          randomization_helper::printHistory(n_transform, message);
          l.eliminateDeadStores();
          break;
        }
```
- **EN**: This chunk continues `permutation` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `permutation`，进一步展开其内部控制流或数据流转。

### Lines 729-744
```cpp
        // TODO: Add remaining transforms
        default:
          break;
      }
    }
  } catch (...) {
    std::cout << "EXCEPTION THROWN!\n";
    std::cout << "SEED: " << seed << '\n';
    throw std::runtime_error("Random test failed");
  }
  message = "End of transformations;\n";
  randomization_helper::printHistory(n_transforms, message);
  return;
}

} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **TransformKind**
  - EN: `TransformKind` is a central symbol declared or implemented in this file.
  - CN: `TransformKind` 是本文件声明或实现的核心符号。
- **max_transformations**
  - EN: `max_transformations` is a central symbol declared or implemented in this file.
  - CN: `max_transformations` 是本文件声明或实现的核心符号。
- **Graph IR**
  - EN: Works on graph, node, and block structures that represent TorchScript programs.
  - CN: 围绕表示 TorchScript 程序的 graph、node 与 block 结构工作。
- **Loop nest scheduling**
  - EN: Reorders, splits, and fuses loops to improve locality and code generation quality.
  - CN: 通过循环重排、切分与融合来提升局部性和代码生成质量。
- **IR simplification**
  - EN: Rewrites expressions and statements into simpler but equivalent forms.
  - CN: 把表达式和语句重写为更简单但等价的形式。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/jit_opt_limit.h`, `torch/csrc/jit/tensorexpr/ir_simplifier.h`, `torch/csrc/jit/tensorexpr/loopnest.h`, `torch/csrc/jit/tensorexpr/loopnest_randomization.h`
- **Standard library / 标准库**: `algorithm`, `iostream`, `random`, `stdexcept`, `typeinfo`, `unordered_map`, `unordered_set`, `vector`
- **Primary symbols in this file / 本文件核心符号**: `TransformKind`, `max_transformations`, `GetAllPerfectlyNestedLoopNests`, `indices`, `find_factor`, `printHistory`, `join`, `indexOf`
- **Note / 说明**: 13 direct includes were detected; only the first few are listed above for readability. / 检测到 13 个直接包含，为便于阅读这里只列出前若干项。
