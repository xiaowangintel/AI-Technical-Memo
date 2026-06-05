# batch_mm.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/batch_mm.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for batch mm, including graph analysis and rewrites.
- 用途 (CN): 实现与 batch mm 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13
```cpp
#include <torch/csrc/jit/passes/batch_mm.h>

#include <ATen/core/functional.h>
#include <ATen/core/symbol.h>
#include <c10/util/Exception.h>
#include <c10/util/irange.h>
#include <torch/csrc/jit/ir/alias_analysis.h>
#include <torch/csrc/jit/ir/constants.h>
#include <torch/csrc/jit/passes/dead_code_elimination.h>
#include <torch/csrc/jit/passes/peephole.h>
#include <torch/csrc/jit/runtime/custom_operator.h>
#include <torch/csrc/jit/runtime/graph_iterator.h>

```
- EN: Pulls in the headers needed by the batch mm logic. Internal dependencies: `torch/csrc/jit/passes/batch_mm.h`, `ATen/core/functional.h`, `ATen/core/symbol.h`, `c10/util/Exception.h`, `c10/util/irange.h`, `...`; external dependencies: none.
- CN: 为 batch mm 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/passes/batch_mm.h`, `ATen/core/functional.h`, `ATen/core/symbol.h`, `c10/util/Exception.h`, `c10/util/irange.h`, `...`；外部依赖：无。

### Lines 14-20
```cpp
#include <ATen/ATen.h>
#include <algorithm>
#include <unordered_map>
#include <utility>

namespace torch::jit {

```
- EN: This block implements local helper logic for batch mm. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 batch mm 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 21-34
```cpp
namespace {
c10::AliasAnalysisKind aliasAnalysisIsSpecialCase() {
  return AliasAnalysisKind::INTERNAL_SPECIAL_CASE;
}
} // namespace

// This pass looks for trees in the graph, where leaves are mm ops, and the
// inner vertices are add nodes. Once we have such a tree they can be reduced to
// two concats and a single mm (basically into a single multiply of a wide
// matrix, with a tall matrix). Such patterns show up mostly in backward of
// RNNs, since the derivative of many uses of matrix multiplies with same
// weights forms exactly such a tree (note that it's usually also highly
// imbalanced i.e. has O(n) depth).
//
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `aliasAnalysisIsSpecialCase`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`aliasAnalysisIsSpecialCase`。

### Lines 35-48
```cpp
// This (or any tree of adds of MMs):
//
// +------+ +------+   +------+ +------+   +------+
// |      | |      |   |      | |      |   |      |
// |  L1  | |  R1  | + |  L2  | |  R2  | = |  O   |
// |      | |      |   |      | |      |   |      |
// +------+ +------+   +------+ +------+   +------+
//
// can be basically transformed into a single MM which looks like this
// (we concat all lhs operands, concat rhs operands, do mm):
//
//                 +------+
//                 |      |
//                 |  R1  |
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 49-60
```cpp
//                 |      |
//                 +------+
//                 |      |
//                 |  R2  |
//                 |      |
//                 +------+
// +------+------+ +------+
// |      |      | |      |
// |  L1  |  L2  | |  O   |
// |      |      | |      |
// +------+------+ +------+

```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 61-74
```cpp
// Note [Further optimizations]
// It would be straightforward to extend the TreeToken class to also detect if
// all MMs had the same lhs/rhs. In such case it's more efficient to expand the
// lhs and use bmm + sum instead of repeating it in memory via concat.

// Note [Overlapping trees]
// Additionally it wouldn't be too hard to add support for partially overlapping
// trees. Right now the it's forbidden in the algorithm (only a single tree will
// be allowed), so theoretically we might miss some optimization options,
// especially that the rejected tree could be much larger. I didn't implement
// that because it's not necessary for the simple RNN cases I saw, so I decided
// to keep stuff simple. If we ever get around implementing this, the right
// solution is probably to fuse MMs for the common part, and assume it's an
// input leaf for the outer two parts (I don't think it's beneficial to
```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 75-81
```cpp
// recompute, unless the subtree is super small, but let's not get into such
// details).

// The algorithm we're using is simple. We're iterating through the graph in the
// topological order and labeling nodes with TreeTokens. Then, we look for roots
// of the trees we formed and fuse them.

```
- EN: Documents the intent, constraints, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、限制条件或兼容性要求。

### Lines 82-89
```cpp
// Tunable parameter. Set to something larger if it turns out to be better.
static constexpr size_t min_fusion_size = 4;

static bool have_same_shape(at::TensorList inputs) {
  auto expected_sizes = inputs[0].sizes();
  return (std::all_of(
      inputs.begin(), inputs.end(), [expected_sizes](const at::Tensor& t) {
        return t.sizes() == expected_sizes;
```
- EN: This block produces a result or forwards a computed value. Key symbols: `have_same_shape`, `sizes`, `all_of`, `begin`, `end`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`have_same_shape`, `sizes`, `all_of`, `begin`, `end`。

### Lines 90-98
```cpp
      }));
}

static bool should_be_transposed(at::TensorList inputs) {
  return (std::all_of(inputs.begin(), inputs.end(), [](const at::Tensor& t) {
    return t.stride(0) == 1 && t.stride(1) == t.size(0);
  }));
}

```
- EN: This block produces a result or forwards a computed value. Key symbols: `should_be_transposed`, `all_of`, `begin`, `end`, `stride`, `size`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`should_be_transposed`, `all_of`, `begin`, `end`, `stride`, `size`。

### Lines 99-110
```cpp
static std::vector<at::Tensor> transpose_inputs(at::TensorList inputs) {
  return fmap(inputs, [](const at::Tensor& i) { return i.t(); });
}

static bool shape_is_fast_for_reduce(
    const at::Tensor& lhs,
    const at::Tensor& rhs) {
  size_t l = lhs.size(0);
  size_t m = lhs.size(1);
  size_t r = rhs.size(1);
  // Numbers obtained by some simple benchmarks of fp32 gemms on a TITAN V
  return m < 512 || ((l < 256 && r < 256) || (l > 256 && r > 256));
```
- EN: This block produces a result or forwards a computed value. Key symbols: `transpose_inputs`, `fmap`, `t`, `shape_is_fast_for_reduce`, `size`.
- CN: 该代码块返回结果或转发已计算的值。关键符号：`transpose_inputs`, `fmap`, `t`, `shape_is_fast_for_reduce`, `size`。

### Lines 111-117
```cpp
}

static RegisterOperators mm_tree_reduction_reg({Operator(
    "prim::MMTreeReduce(...) -> Tensor",
    [](Stack& stack) {
      auto num_inputs = pop(stack).toInt();
      std::vector<at::Tensor> inputs;
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover batch mm behavior. Symbols: `mm_tree_reduction_reg`, `Operator`, `MMTreeReduce`, `pop`, `toInt`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 batch mm 的行为。符号：`mm_tree_reduction_reg`, `Operator`, `MMTreeReduce`, `pop`, `toInt`。

### Lines 118-127
```cpp
      inputs.reserve(num_inputs);
      for (auto it = stack.end() - num_inputs; it != stack.end(); ++it) {
        inputs.push_back(std::move(*it).toTensor());
      }
      drop(stack, num_inputs);

      AT_ASSERT(!inputs.empty());
      AT_ASSERT(inputs.size() % 2 == 0);
      size_t side_num_elems = inputs.size() / 2;
      auto lhs_inputs = at::TensorList(inputs).slice(0, side_num_elems);
```
- EN: This block iterates over collections or graph structures. Key symbols: `reserve`, `end`, `push_back`, `move`, `toTensor`, `drop`, `...`.
- CN: 该代码块遍历集合或图结构。关键符号：`reserve`, `end`, `push_back`, `move`, `toTensor`, `drop`, `...`。

### Lines 128-136
```cpp
      auto rhs_inputs = at::TensorList(inputs).slice(side_num_elems);
      // TODO: checking this is not free, so we should stop if this keeps
      // failing
      if (have_same_shape(lhs_inputs) && have_same_shape(rhs_inputs) &&
          shape_is_fast_for_reduce(lhs_inputs[0], rhs_inputs[0])) {
        // sometimes lhs_inputs or rhs_inputs are not contiguous, and that
        // causes at::cat to go through slow path view them as contiguous if
        // possible by transposing
        bool lhs_input_transposed = should_be_transposed(lhs_inputs);
```
- EN: This block handles conditional branches. Key symbols: `TensorList`, `slice`, `have_same_shape`, `shape_is_fast_for_reduce`, `should_be_transposed`.
- CN: 该代码块处理条件分支。关键符号：`TensorList`, `slice`, `have_same_shape`, `shape_is_fast_for_reduce`, `should_be_transposed`。

### Lines 137-147
```cpp
        bool rhs_input_transposed = should_be_transposed(rhs_inputs);
        at::Tensor lhs, rhs;
        if (lhs_input_transposed) {
          std::vector<at::Tensor> lhs_contig_inputs =
              transpose_inputs(lhs_inputs);
          lhs = at::cat(lhs_contig_inputs, /*dim*/ 0);
          lhs = lhs.t();
        } else {
          lhs = at::cat(lhs_inputs, /*dim=*/1);
        }
        if (rhs_input_transposed) {
```
- EN: This block handles conditional branches. Key symbols: `should_be_transposed`, `transpose_inputs`, `cat`, `t`.
- CN: 该代码块处理条件分支。关键符号：`should_be_transposed`, `transpose_inputs`, `cat`, `t`。

### Lines 148-157
```cpp
          std::vector<at::Tensor> rhs_contig_inputs =
              transpose_inputs(rhs_inputs);
          rhs = at::cat(rhs_contig_inputs, /*dim*/ 1);
          rhs = rhs.t();
        } else {
          rhs = at::cat(rhs_inputs, /*dim=*/0);
        }
        push(stack, at::mm(lhs, rhs));
      } else {
        auto acc = at::mm(inputs[0], inputs[side_num_elems]);
```
- EN: This block implements local helper logic for batch mm. Key symbols: `transpose_inputs`, `cat`, `t`, `push`, `mm`.
- CN: 该代码块实现与 batch mm 相关的局部辅助逻辑。关键符号：`transpose_inputs`, `cat`, `t`, `push`, `mm`。

### Lines 158-165
```cpp
        for (const auto i : c10::irange(1, side_num_elems)) {
          acc.add_(at::mm(inputs[i], inputs[side_num_elems + i]));
        }
        push(stack, std::move(acc));
      }
    },
    aliasAnalysisIsSpecialCase())});

```
- EN: This block iterates over collections or graph structures. Key symbols: `irange`, `add_`, `mm`, `push`, `move`, `aliasAnalysisIsSpecialCase`.
- CN: 该代码块遍历集合或图结构。关键符号：`irange`, `add_`, `mm`, `push`, `move`, `aliasAnalysisIsSpecialCase`。

### Lines 166-173
```cpp
// TreeTokens will be used to label nodes of the graph, if the nodes will fit
// our mm/add tree pattern. Basically we do dynamic programming on DAGs, where
// when we reach node N with inputs A and B, then A and B have already been
// processed, and we can try to unify their TreeTokens (if they have them)
// and build a larger tree.
struct TreeToken {
  uint64_t tree_size = 0; // NOTE: measured in number of leaves i.e. mm ops
  Node* node = nullptr;
```
- EN: Declares core types or data containers for this file. Prominent symbols: `TreeToken`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`TreeToken`。

### Lines 174-181
```cpp
  bool is_root = false;

  static TreeToken mm(Node* mm) {
    TreeToken token;
    token.tree_size = 1;
    token.node = mm;
    token.is_root = true;
    return token;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `mm`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`mm`。

### Lines 182-188
```cpp
  }

  // NB: the returned token might be invalid, so make sure to check its boolean
  // value!
  static TreeToken transpose(Node* t, TreeToken& inp_token) {
    TreeToken token;
    if (!inp_token.node->matches(
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `transpose`, `matches`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`transpose`, `matches`。

### Lines 189-196
```cpp
            "aten::mm(Tensor self, Tensor mat2) -> Tensor")) {
      return token;
    }
    token.tree_size = 1;
    token.node = t;
    token.is_root = true;
    inp_token.is_root = false;
    return token;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `mm`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`mm`。

### Lines 197-204
```cpp
  }

  // NB: the returned token might be invalid, so make sure to check its boolean
  // value!
  static TreeToken add(Node* add, TreeToken& l, TreeToken& r) {
    TreeToken token;
    // See Note [Overlapping trees]
    if (&l == &r || !l.is_root || !r.is_root)
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `add`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`add`。

### Lines 205-211
```cpp
      return token;
    token.tree_size = l.tree_size + r.tree_size;
    token.node = add;
    token.is_root = true;
    l.is_root = r.is_root =
        false; // Reserve the subtrees, so they can't be used again.
    return token;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：无明显局部符号。

### Lines 212-218
```cpp
  }

  explicit operator bool() {
    return is_root;
  }

  std::vector<Node*> removeTransposesAndGatherMatmuls() {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `bool`, `removeTransposesAndGatherMatmuls`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`bool`, `removeTransposesAndGatherMatmuls`。

### Lines 219-225
```cpp
    std::vector<Node*> matmuls;
    std::vector<Node*> queue{node};
    Graph* graph = node->owningGraph();
    while (!queue.empty()) {
      auto n = queue.back();
      queue.pop_back();
      if (n->matches("aten::mm(Tensor self, Tensor mat2) -> Tensor")) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `owningGraph`, `empty`, `back`, `pop_back`, `matches`, `mm`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`owningGraph`, `empty`, `back`, `pop_back`, `matches`, `mm`。

### Lines 226-233
```cpp
        matmuls.push_back(n);
      } else if (n->matches("aten::t(Tensor self) -> Tensor")) {
        Node* input_node = n->input()->node();
        AT_ASSERT(input_node->matches(
            "aten::mm(Tensor self, Tensor mat2) -> Tensor"));
        // (AB)^T == B^TA^T
        WithInsertPoint insert_guard{input_node};
        Value* A = input_node->inputs()[0];
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `push_back`, `matches`, `t`, `input`, `node`, `mm`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`push_back`, `matches`, `t`, `input`, `node`, `mm`, `...`。

### Lines 234-247
```cpp
        Value* B = input_node->inputs()[1];
        Value* AT = graph->insert(aten::t, {A});
        Value* BT = graph->insert(aten::t, {B});
        Value* BTAT = graph->insert(aten::mm, {BT, AT});
        n->output()->replaceAllUsesWith(BTAT);
        matmuls.push_back(BTAT->node());
      } else if (
          n->matches(
              "aten::add(Tensor self, Tensor other, *, Scalar alpha) -> Tensor")) {
        queue.push_back(n->inputs()[0]->node());
        queue.push_back(n->inputs()[1]->node());
      } else {
        TORCH_INTERNAL_ASSERT(
            false, "Unsupported node found in a BatchMM tree!");
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `inputs`, `insert`, `output`, `replaceAllUsesWith`, `push_back`, `node`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`inputs`, `insert`, `output`, `replaceAllUsesWith`, `push_back`, `node`, `...`。

### Lines 248-254
```cpp
      }
    }
    return matmuls;
  }
};

enum class Side { LHS, RHS };
```
- EN: Declares core types or data containers for this file. Prominent symbols: `class`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`class`。

### Lines 255-261
```cpp

static void BatchMMTreeReduce(Block* block, AliasDb& alias_db) {
  auto graph = block->owningGraph();

  // Look for trees in the block
  std::unordered_map<Node*, TreeToken> tokens;
  for (auto node : block->nodes()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; advances frontend parsing or lowering state; protects shared state or ordering assumptions. Key symbols: `BatchMMTreeReduce`, `owningGraph`, `nodes`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；推进前端解析或降级状态；保护共享状态或执行顺序假设。关键符号：`BatchMMTreeReduce`, `owningGraph`, `nodes`。

### Lines 262-268
```cpp
    if (node->matches("aten::mm(Tensor self, Tensor mat2) -> Tensor") &&
        !alias_db.hasWriters(node)) {
      tokens[node] = TreeToken::mm(node);
    } else if (
        node->matches("aten::t(Tensor self) -> Tensor") &&
        !alias_db.hasWriters(node)) {
      auto input_it = tokens.find(node->input()->node());
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `matches`, `mm`, `hasWriters`, `t`, `find`, `input`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`matches`, `mm`, `hasWriters`, `t`, `find`, `input`, `...`。

### Lines 269-276
```cpp
      if (input_it != tokens.end()) {
        tokens[node] = TreeToken::transpose(node, input_it->second);
      }
    } else if (
        node->matches(
            "aten::add(Tensor self, Tensor other, *, Scalar alpha) -> Tensor") &&
        !alias_db.hasWriters(node)) {
      Node* lhs = node->inputs()[0]->node();
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `end`, `transpose`, `matches`, `add`, `hasWriters`, `inputs`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`end`, `transpose`, `matches`, `add`, `hasWriters`, `inputs`, `...`。

### Lines 277-287
```cpp
      Node* rhs = node->inputs()[1]->node();
      auto lhs_it = tokens.find(lhs);
      auto rhs_it = tokens.find(rhs);
      // See Note [Overlapping trees] (regarding the uses().size() == 1 check)
      // We could treat a subtree with multiple uses as if it was overlapping.
      // XXX: uses().size() == 1 is also something that guarantees that this
      // transform is valid, because we know for sure that the none of these
      // operands depend on the result of the other. If we were to remove this,
      // we need to compute a transitive closure and actually check the
      // dependencies.
      if (lhs_it != tokens.end() && rhs_it != tokens.end() &&
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; advances frontend parsing or lowering state. Key symbols: `inputs`, `node`, `find`, `end`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；推进前端解析或降级状态。关键符号：`inputs`, `node`, `find`, `end`。

### Lines 288-295
```cpp
          lhs->output()->uses().size() == 1 &&
          rhs->output()->uses().size() == 1) {
        if (auto token = TreeToken::add(node, lhs_it->second, rhs_it->second)) {
          tokens[node] = token;
        }
      }
    } else {
      for (auto block : node->blocks()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; advances frontend parsing or lowering state; protects shared state or ordering assumptions. Key symbols: `output`, `uses`, `size`, `add`, `blocks`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；推进前端解析或降级状态；保护共享状态或执行顺序假设。关键符号：`output`, `uses`, `size`, `add`, `blocks`。

### Lines 296-302
```cpp
        BatchMMTreeReduce(block, alias_db);
      }
    }
  }

  // Merge trees we've found
  for (auto& item : tokens) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; advances frontend parsing or lowering state; protects shared state or ordering assumptions. Key symbols: `BatchMMTreeReduce`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；推进前端解析或降级状态；保护共享状态或执行顺序假设。关键符号：`BatchMMTreeReduce`。

### Lines 303-310
```cpp
    auto& root = item.second;
    if (!root || root.tree_size < min_fusion_size)
      continue;
    auto matmuls = root.removeTransposesAndGatherMatmuls();
    WithInsertPoint insert_guard{root.node};
    Node* tree_reduce =
        graph->insertNode(graph->create(Symbol::prim("MMTreeReduce")));
    for (Node* matmul : matmuls) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `removeTransposesAndGatherMatmuls`, `insertNode`, `create`, `prim`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`removeTransposesAndGatherMatmuls`, `insertNode`, `create`, `prim`。

### Lines 311-320
```cpp
      tree_reduce->addInput(matmul->inputs().at(0));
    }
    for (Node* matmul : matmuls) {
      tree_reduce->addInput(matmul->inputs().at(1));
    }
    root.node->output()->replaceAllUsesWith(tree_reduce->output());
    // NB: don't bother with cleaning up after yourself. We'll use DCE for that.
  }
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `addInput`, `inputs`, `output`, `replaceAllUsesWith`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`addInput`, `inputs`, `output`, `replaceAllUsesWith`。

### Lines 321-331
```cpp
static bool shape_is_fast_for_side(const at::Tensor& other_side_input) {
  // Cutoff chose by benchmarking on a TITAN V
  return other_side_input.numel() <= 1024 * 2048;
}

static RegisterOperators mm_batch_side_reg({Operator(
    prim::MMBatchSide,
    [](const Node* node) -> Operation {
      size_t num_other_side_inputs = node->inputs().size() - 1;
      Side single_side = static_cast<Side>(node->i(Symbol::attr("side")));
      return [num_other_side_inputs, single_side](Stack& stack) {
```
- EN: Registers callable entities, pass hooks, or exported interfaces so runtime code can discover batch mm behavior. Symbols: `shape_is_fast_for_side`, `numel`, `mm_batch_side_reg`, `Operator`, `inputs`, `size`, `...`.
- CN: 注册可调用实体、pass 钩子或导出接口，使运行时能够发现 batch mm 的行为。符号：`shape_is_fast_for_side`, `numel`, `mm_batch_side_reg`, `Operator`, `inputs`, `size`, `...`。

### Lines 332-341
```cpp
        at::Tensor side_input;
        std::vector<at::Tensor> other_side_inputs;
        other_side_inputs.reserve(num_other_side_inputs);
        for (auto it = stack.end() - num_other_side_inputs; it != stack.end();
             ++it) {
          other_side_inputs.push_back(std::move(*it).toTensor());
        }
        drop(stack, num_other_side_inputs);
        pop(stack, side_input);

```
- EN: This block iterates over collections or graph structures. Key symbols: `reserve`, `end`, `push_back`, `move`, `toTensor`, `drop`, `...`.
- CN: 该代码块遍历集合或图结构。关键符号：`reserve`, `end`, `push_back`, `move`, `toTensor`, `drop`, `...`。

### Lines 342-350
```cpp
        auto any_other_input = other_side_inputs[0];
        if (have_same_shape(other_side_inputs) &&
            shape_is_fast_for_side(other_side_inputs[0])) {
          auto other_side_input =
              at::cat(other_side_inputs, single_side == Side::LHS ? 1 : 0);
          auto mm_out = single_side == Side::LHS
              ? side_input.mm(other_side_input)
              : other_side_input.mm(side_input);
          auto outputs = at::chunk(
```
- EN: This block handles conditional branches. Key symbols: `have_same_shape`, `shape_is_fast_for_side`, `cat`, `mm`, `chunk`.
- CN: 该代码块处理条件分支。关键符号：`have_same_shape`, `shape_is_fast_for_side`, `cat`, `mm`, `chunk`。

### Lines 351-357
```cpp
              mm_out,
              num_other_side_inputs,
              /*dim=*/single_side == Side::LHS ? 1 : 0);
          stack.insert(
              stack.end(),
              std::make_move_iterator(outputs.begin()),
              std::make_move_iterator(outputs.end()));
```
- EN: This block implements local helper logic for batch mm. Key symbols: `insert`, `end`, `make_move_iterator`, `begin`.
- CN: 该代码块实现与 batch mm 相关的局部辅助逻辑。关键符号：`insert`, `end`, `make_move_iterator`, `begin`。

### Lines 358-364
```cpp
        } else {
          if (single_side == Side::LHS) {
            for (at::Tensor& other : other_side_inputs) {
              stack.emplace_back(side_input.mm(other));
            }
          } else {
            for (at::Tensor& other : other_side_inputs) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures. Key symbols: `emplace_back`, `mm`.
- CN: 该代码块处理条件分支；遍历集合或图结构。关键符号：`emplace_back`, `mm`。

### Lines 365-372
```cpp
              stack.emplace_back(other.mm(side_input));
            }
          }
        }
      };
    },
    aliasAnalysisIsSpecialCase())});

```
- EN: This block implements local helper logic for batch mm. Key symbols: `emplace_back`, `mm`, `aliasAnalysisIsSpecialCase`.
- CN: 该代码块实现与 batch mm 相关的局部辅助逻辑。关键符号：`emplace_back`, `mm`, `aliasAnalysisIsSpecialCase`。

### Lines 373-380
```cpp
static std::pair<std::vector<Node*>, std::vector<Node*>> gatherIndependentMMUses(
    Value* value,
    AliasDb& alias_db) {
  const auto postprocess = [&](std::vector<Node*> mms) {
    if (mms.empty()) {
      return mms;
    }
    std::sort(mms.begin(), mms.end(), [](Node* n, Node* m) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `gatherIndependentMMUses`, `empty`, `sort`, `begin`, `end`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`gatherIndependentMMUses`, `empty`, `sort`, `begin`, `end`。

### Lines 381-387
```cpp
      return n->isBefore(m);
    });
    // Filter out dependent MMs. This algorithm might do very badly if e.g. you
    // have a lot of independent MMs, that depend on the first one, but I doubt
    // this will be a common scenario.
    for (const auto i : c10::irange(mms.size())) {
      if (mms[i] == nullptr)
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value. Key symbols: `isBefore`, `irange`, `size`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值。关键符号：`isBefore`, `irange`, `size`。

### Lines 388-397
```cpp
        continue;
      for (size_t j = i + 1; j < mms.size(); ++j) {
        if (mms[j] == nullptr)
          continue;
        if (!alias_db.couldMoveBeforeTopologically(mms[j], mms[i])) {
          mms[j] = nullptr;
        }
      }
    }
    return c10::filter(mms, [](Node* n) { return n != nullptr; });
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `size`, `couldMoveBeforeTopologically`, `filter`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`size`, `couldMoveBeforeTopologically`, `filter`。

### Lines 398-404
```cpp
  };

  Block* block = value->node()->owningBlock();
  std::vector<Node*> lhses; // Will contain nodes where value is used as an lhs
  std::vector<Node*> rhses; // Like above, but rhs
  for (Use u : value->uses()) {
    if (u.user->owningBlock() == block &&
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `node`, `owningBlock`, `uses`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`node`, `owningBlock`, `uses`。

### Lines 405-414
```cpp
        u.user->matches("aten::mm(Tensor self, Tensor mat2) -> Tensor") &&
        !alias_db.hasWriters(u.user)) {
      if (u.offset == 0 && u.user->inputs()[1] != value) {
        lhses.push_back(u.user);
      } else if (u.offset == 1 && u.user->inputs()[0] != value) {
        rhses.push_back(u.user);
      }
    }
  }
  return std::make_pair(
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `matches`, `mm`, `hasWriters`, `inputs`, `push_back`, `make_pair`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`matches`, `mm`, `hasWriters`, `inputs`, `push_back`, `make_pair`。

### Lines 415-423
```cpp
      postprocess(std::move(lhses)), postprocess(std::move(rhses)));
}

static void BatchMMSide(Block* block, AliasDb& alias_db) {
  // NB: 8 is the current loop unrolling factor
  static constexpr size_t how_many_is_many = 8;
  const auto batch_side = [&](std::vector<Node*>& mms, Side side) {
    AT_ASSERT(!mms.empty());
    for (int64_t i = static_cast<int64_t>(mms.size()) - 2; i >= 0; --i) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `postprocess`, `move`, `BatchMMSide`, `empty`, `size`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`postprocess`, `move`, `BatchMMSide`, `empty`, `size`。

### Lines 424-435
```cpp
      bool move_ok = alias_db.moveBeforeTopologicallyValid(mms[i], mms[i + 1]);
      AT_ASSERT(move_ok);
    }
    WithInsertPoint insert_guard{mms[0]};
    Graph* graph = mms[0]->owningGraph();
    Node* batch_mm = graph->create(
        prim::MMBatchSide,
        /*inputs=*/{},
        /*num_outputs=*/mms.size());
    graph->insertNode(batch_mm);
    batch_mm->i_(Symbol::attr("side"), static_cast<int>(side));
    Value* const_side = mms[0]->inputs().at(side == Side::LHS ? 0 : 1);
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `moveBeforeTopologicallyValid`, `owningGraph`, `create`, `size`, `insertNode`, `i_`, `...`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`moveBeforeTopologicallyValid`, `owningGraph`, `create`, `size`, `insertNode`, `i_`, `...`。

### Lines 436-442
```cpp
    batch_mm->addInput(const_side);
    for (const auto i : c10::irange(mms.size())) {
      batch_mm->addInput(mms[i]->inputs().at(side == Side::LHS ? 1 : 0));
      mms[i]->output()->replaceAllUsesWith(batch_mm->outputs().at(i));
    }
  };

```
- EN: This block iterates over collections or graph structures. Key symbols: `addInput`, `irange`, `size`, `inputs`, `output`, `replaceAllUsesWith`, `...`.
- CN: 该代码块遍历集合或图结构。关键符号：`addInput`, `irange`, `size`, `inputs`, `output`, `replaceAllUsesWith`, `...`。

### Lines 443-451
```cpp
  std::unordered_set<Value*> considered_values;
  for (Node* node : block->nodes()) {
    if (node->matches("aten::mm(Tensor self, Tensor mat2) -> Tensor") &&
        !alias_db.hasWriters(node)) {
      for (Value* input : node->inputs()) {
        if (/*bool not_inserted = */ !considered_values.emplace(input).second) {
          continue;
        }
        auto uses_with_many = gatherIndependentMMUses(input, alias_db);
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `matches`, `mm`, `hasWriters`, `inputs`, `emplace`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `matches`, `mm`, `hasWriters`, `inputs`, `emplace`, `...`。

### Lines 452-460
```cpp
        if (uses_with_many.first.size() >= how_many_is_many) {
          batch_side(uses_with_many.first, Side::LHS);
        }
        if (uses_with_many.second.size() >= how_many_is_many) {
          batch_side(uses_with_many.second, Side::RHS);
        }
      }
    } else {
      for (Block* subblock : node->blocks()) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `size`, `batch_side`, `blocks`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`size`, `batch_side`, `blocks`。

### Lines 461-467
```cpp
        BatchMMSide(subblock, alias_db);
      }
    }
  }
}

static bool hasMMOperators(std::shared_ptr<Graph>& graph) {
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `BatchMMSide`, `hasMMOperators`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`BatchMMSide`, `hasMMOperators`。

### Lines 468-475
```cpp
  DepthFirstGraphNodeIterator it(graph);
  Node* n = nullptr;
  while ((n = it.next()) != nullptr) {
    if (n->matches("aten::mm(Tensor self, Tensor mat2) -> Tensor")) {
      return true;
    }
  }
  return false;
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `it`, `next`, `matches`, `mm`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`it`, `next`, `matches`, `mm`。

### Lines 476-488
```cpp
}

void BatchMM(std::shared_ptr<Graph>& graph) {
  if (!hasMMOperators(graph)) {
    return;
  }
  AliasDb alias_db(graph);
  BatchMMTreeReduce(graph->block(), alias_db);
  BatchMMSide(graph->block(), alias_db);
  EliminateDeadCode(graph);
  // It's possible that transpose rearrangements have created sequences of
  // consecutive transposes that didn't exist before.

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `BatchMM`, `hasMMOperators`, `alias_db`, `BatchMMTreeReduce`, `block`, `BatchMMSide`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`BatchMM`, `hasMMOperators`, `alias_db`, `BatchMMTreeReduce`, `block`, `BatchMMSide`, `...`。

### Lines 489-493
```cpp
  // tensor type properties are not guaranteed to be correct
  PeepholeOptimize(graph, /*disable_shape_peepholes*/ true);
}

} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Alias and mutation reasoning — the code tracks side effects and write/read interactions.
  CN: 别名与变更分析——代码会跟踪副作用以及读写关系。
- EN: Parsing pipeline — the logic translates source text into TorchScript syntax structures.
  CN: 解析流水线——相关逻辑将源码文本转换为 TorchScript 语法结构。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Fusion support — the file prepares groups of operations for fused execution.
  CN: 融合支持——该文件会为融合执行准备算子分组。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/passes/batch_mm.h`, `ATen/core/functional.h`, `ATen/core/symbol.h`, `c10/util/Exception.h`, `c10/util/irange.h`, `torch/csrc/jit/ir/alias_analysis.h`, `torch/csrc/jit/ir/constants.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `torch/csrc/jit/passes/peephole.h`, `torch/csrc/jit/runtime/custom_operator.h`, `...`
- External includes / 外部头文件: `algorithm`, `unordered_map`, `utility`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `aliasAnalysisIsSpecialCase`, `have_same_shape`, `sizes`, `all_of`, `begin`, `end`, `should_be_transposed`, `stride`, `size`, `transpose_inputs`, `...`
