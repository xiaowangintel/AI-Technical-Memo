# peephole_list_idioms.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/peephole_list_idioms.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for peephole list idioms, including graph analysis and rewrites.
- 用途 (CN): 实现与 peephole list idioms 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10
```cpp
#include <ATen/core/jit_type.h>
#include <torch/csrc/jit/ir/alias_analysis.h>
#include <torch/csrc/jit/ir/ir_views.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/peephole_list_idioms.h>
#include <torch/csrc/jit/passes/value_refinement_utils.h>
#include <torch/csrc/jit/runtime/slice_indices_adjust.h>
#include <limits>
#include <utility>

```
- EN: Pulls in the headers needed by the peephole list idioms logic. Internal dependencies: `ATen/core/jit_type.h`, `torch/csrc/jit/ir/alias_analysis.h`, `torch/csrc/jit/ir/ir_views.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/peephole_list_idioms.h`, `...`; external dependencies: `limits`, `utility`.
- CN: 为 peephole list idioms 相关逻辑引入所需头文件。内部依赖：`ATen/core/jit_type.h`, `torch/csrc/jit/ir/alias_analysis.h`, `torch/csrc/jit/ir/ir_views.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/peephole_list_idioms.h`, `...`；外部依赖：`limits`, `utility`。

### Lines 11-17
```cpp
namespace torch::jit {

static std::optional<size_t> normalizeIndex(int64_t index, size_t len) {
  if (index < 0) {
    index = index + len;
  }
  if (index >= 0 && index < static_cast<int64_t>(len)) {
```
- EN: This block handles conditional branches. Key symbols: `normalizeIndex`.
- CN: 该代码块处理条件分支。关键符号：`normalizeIndex`。

### Lines 18-25
```cpp
    return index;
  } else {
    return std::nullopt;
  }
}

// see [value refinement algorithm]

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 26-32
```cpp
struct ListLenRefiner {
  ListLenRefiner(
      std::shared_ptr<Graph> graph,
      std::unordered_set<Value*>& mutated_lists)
      : graph_(std::move(graph)), mutated_lists_(mutated_lists) {}

  bool run() {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `ListLenRefiner`, `graph_`, `move`, `mutated_lists_`, `run`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`ListLenRefiner`, `graph_`, `move`, `mutated_lists_`, `run`。

### Lines 33-40
```cpp
    std::unordered_set<Value*> li_with_len_use;
    collectListsToRefine(graph_->block(), li_with_len_use);
    if (lists_to_refine_.empty()) {
      return false;
    }
    ListRefinement refinements;
    RefineListLens(graph_->block(), std::move(refinements));
    return changed_;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `collectListsToRefine`, `block`, `empty`, `RefineListLens`, `move`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`collectListsToRefine`, `block`, `empty`, `RefineListLens`, `move`。

### Lines 41-47
```cpp
  }

  // we only need to analyze lists that have multiple uses of len(), and we can
  // only analyze lists that are not mutated
  void collectListsToRefine(
      Block* b,
      std::unordered_set<Value*>& li_with_len_use) {
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `collectListsToRefine`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`collectListsToRefine`。

### Lines 48-56
```cpp
    for (Node* n : b->nodes()) {
      for (Block* block : n->blocks()) {
        collectListsToRefine(block, li_with_len_use);
      }

      if (n->kind() != aten::len) {
        continue;
      }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `nodes`, `blocks`, `collectListsToRefine`, `kind`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`nodes`, `blocks`, `collectListsToRefine`, `kind`。

### Lines 57-68
```cpp
      auto first_input = n->input(0);
      if (first_input->type()->castRaw<ListType>() &&
          !mutated_lists_.count(first_input)) {
        if (!li_with_len_use.count(first_input)) {
          li_with_len_use.insert(first_input);
        } else {
          lists_to_refine_.insert(first_input);
        }
      }
    }
  }

```
- EN: This block handles conditional branches. Key symbols: `input`, `type`, `count`, `insert`.
- CN: 该代码块处理条件分支。关键符号：`input`, `type`, `count`, `insert`。

### Lines 69-75
```cpp
  ListRefinement RefineListLens(Block* b, ListRefinement block_refinements) {
    active_refinements_.push_back(&block_refinements);
    for (Node* n : b->nodes()) {
      if (n->matches("aten::eq(int a, int b) -> bool") ||
          n->matches("aten::ne(int a, int b) -> bool")) {
        // check for one input constant and the other coming from len(li)
        for (size_t const_index : {0, 1}) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `RefineListLens`, `push_back`, `nodes`, `matches`, `eq`, `ne`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`RefineListLens`, `push_back`, `nodes`, `matches`, `eq`, `ne`。

### Lines 76-89
```cpp
          auto ival = constant_as<int64_t>(n->input(const_index));
          if (!ival) {
            continue;
          }
          auto li_len = n->input(1 - const_index);
          if (!li_len->node()->matches("aten::len.t(t[] a) -> int") ||
              !lists_to_refine_.count(li_len->node()->input())) {
            continue;
          }
          ListRefinement refine;
          refine[li_len->node()->input()] = *ival;
          boolean_value_refinements_[n->output()] = n->kind() == aten::eq
              ? BooleanRefinementMapping::TrueRefinements(std::move(refine))
              : BooleanRefinementMapping::FalseRefinements(std::move(refine));
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `input`, `node`, `matches`, `t`, `count`, `output`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`input`, `node`, `matches`, `t`, `count`, `output`, `...`。

### Lines 90-100
```cpp
        }
      } else if (n->kind() == aten::len) {
        if (auto maybe_len = tryFindRefinement(n->input(0))) {
          changed_ = true;
          WithInsertPoint guard(n);
          n->output()->replaceAllUsesWith(
              graph_->insertConstant(static_cast<int64_t>(*maybe_len)));
        }
      } else if (n->kind() == prim::If) {
        IfView if_n(n);
        bool has_cond_ref = boolean_value_refinements_.count(if_n.cond()) != 0;
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `kind`, `tryFindRefinement`, `input`, `guard`, `output`, `replaceAllUsesWith`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`kind`, `tryFindRefinement`, `input`, `guard`, `output`, `replaceAllUsesWith`, `...`。

### Lines 101-111
```cpp
        ListRefinement empty;
        auto true_block_refinements = RefineListLens(
            if_n.thenBlock(),
            has_cond_ref ? boolean_value_refinements_[if_n.cond()].true_refine()
                         : empty);
        auto false_block_refinements = RefineListLens(
            if_n.elseBlock(),
            has_cond_ref
                ? boolean_value_refinements_[if_n.cond()].false_refine()
                : empty);

```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `RefineListLens`, `thenBlock`, `cond`, `true_refine`, `elseBlock`, `false_refine`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`RefineListLens`, `thenBlock`, `cond`, `true_refine`, `elseBlock`, `false_refine`。

### Lines 112-125
```cpp
        joinIfRefinements(
            n,
            throwing_blocks_,
            block_refinements,
            true_block_refinements,
            false_block_refinements,
            boolean_value_refinements_);
      } else {
        handleCommonRefinentOperators(
            n, throwing_blocks_, boolean_value_refinements_);
      }
    }
    active_refinements_.pop_back();
    return block_refinements;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `joinIfRefinements`, `handleCommonRefinentOperators`, `pop_back`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`joinIfRefinements`, `handleCommonRefinentOperators`, `pop_back`。

### Lines 126-132
```cpp
  }

  std::optional<int64_t> tryFindRefinement(Value* v) {
    for (const auto& ref : active_refinements_) {
      auto maybe_refinement = ref->find(v);
      if (maybe_refinement != ref->end()) {
        return maybe_refinement->second;
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `tryFindRefinement`, `find`, `end`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`tryFindRefinement`, `find`, `end`。

### Lines 133-139
```cpp
      }
    }
    return std::nullopt;
  }

  std::shared_ptr<Graph> graph_;
  std::unordered_set<Value*> mutated_lists_;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 140-147
```cpp
  // candidate lists for optimizations
  std::unordered_set<Value*> lists_to_refine_;
  // A stack of active refinements, one for each block
  std::vector<ListRefinement*> active_refinements_;
  // A map from Boolean Value * -> associated refinements
  std::unordered_map<Value*, BooleanRefinementMapping>
      boolean_value_refinements_;
  std::unordered_set<Block*> throwing_blocks_;
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：无明显局部符号。

### Lines 148-154
```cpp
  bool changed_ = false;
};

// This pass only does optimizations on lists which aren't mutated,
// so we first use the Alias Db to collect the set of list values
// which we shouldn't optimize.
struct PeepholeOptimizeListIdiomsImpl {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `PeepholeOptimizeListIdiomsImpl`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`PeepholeOptimizeListIdiomsImpl`。

### Lines 155-161
```cpp
  PeepholeOptimizeListIdiomsImpl(
      std::shared_ptr<Graph> graph,
      bool refine_list_len)
      : graph_(std::move(graph)),
        aliasDb_(std::make_unique<AliasDb>(graph_)),
        refine_list_len_(refine_list_len) {}

```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `PeepholeOptimizeListIdiomsImpl`, `graph_`, `move`, `aliasDb_`, `refine_list_len_`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`PeepholeOptimizeListIdiomsImpl`, `graph_`, `move`, `aliasDb_`, `refine_list_len_`。

### Lines 162-168
```cpp
  bool run() {
    collectMutatedLists(graph_->block());
    bool changed = runBlock(graph_->block());
    if (refine_list_len_) {
      changed |= ListLenRefiner(graph_, mutated_lists_).run();
    }
    return changed;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `run`, `collectMutatedLists`, `block`, `runBlock`, `ListLenRefiner`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`run`, `collectMutatedLists`, `block`, `runBlock`, `ListLenRefiner`。

### Lines 169-177
```cpp
  }

 private:
  void checkForMutatedList(Value* v) {
    if (v->type()->castRaw<ListType>() && aliasDb_->hasWriters(v)) {
      mutated_lists_.insert(v);
    }
  }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `checkForMutatedList`, `type`, `hasWriters`, `insert`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`checkForMutatedList`, `type`, `hasWriters`, `insert`。

### Lines 178-186
```cpp
  void collectMutatedLists(Block* b) {
    for (Value* v : b->inputs()) {
      checkForMutatedList(v);
    }
    for (Node* n : b->nodes()) {
      for (Value* v : n->outputs()) {
        checkForMutatedList(v);
      }
      for (Block* block : n->blocks()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `collectMutatedLists`, `inputs`, `checkForMutatedList`, `nodes`, `outputs`, `blocks`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`collectMutatedLists`, `inputs`, `checkForMutatedList`, `nodes`, `outputs`, `blocks`。

### Lines 187-193
```cpp
        collectMutatedLists(block);
      }
    }
  }

  bool optimizeSlice(Node* slice_node, Node* list_construct_node) {
    auto start_val = toIValue(slice_node->input(1));
```
- EN: This block manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `collectMutatedLists`, `optimizeSlice`, `toIValue`, `input`.
- CN: 该代码块操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`collectMutatedLists`, `optimizeSlice`, `toIValue`, `input`。

### Lines 194-200
```cpp
    auto end_val = toIValue(slice_node->input(2));
    auto step_val = toIValue(slice_node->input(3));

    // All args must be constant to apply this optimization.
    if (start_val == std::nullopt || end_val == std::nullopt ||
        step_val == std::nullopt) {
      return false;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `toIValue`, `input`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`toIValue`, `input`。

### Lines 201-208
```cpp
    }

    int64_t start = start_val->isInt() ? start_val->to<int64_t>()
                                       : std::numeric_limits<int64_t>::max();
    int64_t end = end_val->isInt() ? end_val->to<int64_t>()
                                   : std::numeric_limits<int64_t>::max();
    int64_t step = step_val->isInt() ? step_val->to<int64_t>() : 1;

```
- EN: This block implements local helper logic for peephole list idioms. Key symbols: `isInt`, `max`.
- CN: 该代码块实现与 peephole list idioms 相关的局部辅助逻辑。关键符号：`isInt`, `max`。

### Lines 209-216
```cpp
    size_t list_size = list_construct_node->inputs().size();
    size_t num_values = slice_indices_adjust(list_size, &start, &end, step);

    WithInsertPoint guard(slice_node);
    auto slice_list_construct =
        graph_->insertNode(graph_->create(prim::ListConstruct));
    slice_list_construct->output()->setType(slice_node->output()->type());
    for (size_t i = start, j = 0; j < num_values; ++j) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `inputs`, `size`, `slice_indices_adjust`, `guard`, `insertNode`, `create`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`inputs`, `size`, `slice_indices_adjust`, `guard`, `insertNode`, `create`, `...`。

### Lines 217-225
```cpp
      slice_list_construct->addInput(list_construct_node->input(i));
      i += step;
    }

    slice_node->output()->replaceAllUsesWith(slice_list_construct->output());
    if (mutated_lists_.count(slice_node->output())) {
      mutated_lists_.insert(slice_list_construct->output());
    }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `addInput`, `input`, `output`, `replaceAllUsesWith`, `count`, `insert`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`addInput`, `input`, `output`, `replaceAllUsesWith`, `count`, `insert`。

### Lines 226-232
```cpp
    return true;
  }

  bool runBlock(Block* block) {
    bool changed = false;
    for (Node* node : block->nodes()) {
      for (Block* b : node->blocks()) {
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `runBlock`, `nodes`, `blocks`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`runBlock`, `nodes`, `blocks`。

### Lines 233-241
```cpp
        changed |= runBlock(b);
      }

      // only optimizing list ops
      if (node->inputs().empty() ||
          !node->input(0)->type()->castRaw<ListType>()) {
        continue;
      }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `runBlock`, `inputs`, `empty`, `input`, `type`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`runBlock`, `inputs`, `empty`, `input`, `type`。

### Lines 242-248
```cpp
      auto first_input = node->input(0);

      // only optimizing ops with unmutated lists
      if (mutated_lists_.count(first_input)) {
        continue;
      }

```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `input`, `count`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`input`, `count`。

### Lines 249-260
```cpp
      auto list_creation_node = first_input->node();
      if (list_creation_node->kind() != prim::ListConstruct) {
        continue;
      }

      if (node->kind() == aten::len) {
        WithInsertPoint guard(node);
        node->output()->replaceAllUsesWith(graph_->insertConstant(
            static_cast<int64_t>(first_input->node()->inputs().size())));
        changed = true;
      } else if (node->kind() == aten::__getitem__) {
        if (auto index = toIValue(node->input(1))) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `node`, `kind`, `guard`, `output`, `replaceAllUsesWith`, `insertConstant`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`node`, `kind`, `guard`, `output`, `replaceAllUsesWith`, `insertConstant`, `...`。

### Lines 261-270
```cpp
          size_t list_size = list_creation_node->inputs().size();
          if (auto norm_index = normalizeIndex(index->toInt(), list_size)) {
            node->output()->replaceAllUsesWith(
                list_creation_node->input(*norm_index));
            changed = true;
          }
        }
      } else if (node->kind() == prim::ListUnpack) {
        // if sizes are unequal it's a runtime error
        if (list_creation_node->inputs().size() != node->outputs().size()) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `inputs`, `size`, `normalizeIndex`, `toInt`, `output`, `replaceAllUsesWith`, `...`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`inputs`, `size`, `normalizeIndex`, `toInt`, `output`, `replaceAllUsesWith`, `...`。

### Lines 271-278
```cpp
          continue;
        }
        for (size_t i = 0; i < node->outputs().size(); ++i) {
          node->output(i)->replaceAllUsesWith(list_creation_node->input(i));
          changed = true;
        }
      } else if (node->kind() == aten::add) {
        if (node->inputs().size() != 2) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `outputs`, `size`, `output`, `replaceAllUsesWith`, `input`, `kind`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`outputs`, `size`, `output`, `replaceAllUsesWith`, `input`, `kind`, `...`。

### Lines 279-286
```cpp
          continue;
        }
        auto second_input = node->input(1);
        // already checked first, need to check second
        if (mutated_lists_.count(second_input)) {
          continue;
        }
        if (second_input->node()->kind() != prim::ListConstruct) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `input`, `count`, `node`, `kind`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`input`, `count`, `node`, `kind`。

### Lines 287-293
```cpp
          continue;
        }
        WithInsertPoint guard(node);
        auto list_construct =
            graph_->insertNode(graph_->create(prim::ListConstruct));
        list_construct->output()->setType(node->output()->type());
        for (Value* v : first_input->node()->inputs()) {
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; protects shared state or ordering assumptions. Key symbols: `guard`, `insertNode`, `create`, `output`, `setType`, `type`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；保护共享状态或执行顺序假设。关键符号：`guard`, `insertNode`, `create`, `output`, `setType`, `type`, `...`。

### Lines 294-300
```cpp
          list_construct->addInput(v);
        }
        for (Value* v : second_input->node()->inputs()) {
          list_construct->addInput(v);
        }
        node->output()->replaceAllUsesWith(list_construct->output());
        if (mutated_lists_.count(node->output())) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `addInput`, `node`, `inputs`, `output`, `replaceAllUsesWith`, `count`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`addInput`, `node`, `inputs`, `output`, `replaceAllUsesWith`, `count`。

### Lines 301-308
```cpp
          mutated_lists_.insert(list_construct->output());
        }
        changed = true;
      } else if (node->kind() == aten::slice) {
        changed |= optimizeSlice(node, first_input->node());
      }
    }
    return changed;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `insert`, `output`, `kind`, `optimizeSlice`, `node`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`insert`, `output`, `kind`, `optimizeSlice`, `node`。

### Lines 309-315
```cpp
  }

  std::unordered_set<Value*> mutated_lists_;
  std::shared_ptr<Graph> graph_;
  std::unique_ptr<AliasDb> aliasDb_;
  bool refine_list_len_;
};
```
- EN: This block manipulates TorchScript IR objects. Key symbols: no dominant local symbol names.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：无明显局部符号。

### Lines 316-323
```cpp

bool PeepholeOptimizeListIdioms(
    const std::shared_ptr<Graph>& graph,
    bool refine_list_len) {
  PeepholeOptimizeListIdiomsImpl opt(graph, refine_list_len);
  return opt.run();
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `PeepholeOptimizeListIdioms`, `opt`, `run`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`PeepholeOptimizeListIdioms`, `opt`, `run`。

### Lines 324-324
```cpp
} // namespace torch::jit
```
- EN: Opens or closes namespace scopes so the remaining declarations live in the expected JIT/frontend module.
- CN: 打开或关闭命名空间作用域，使后续声明归属于预期的 JIT/前端模块。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Alias and mutation reasoning — the code tracks side effects and write/read interactions.
  CN: 别名与变更分析——代码会跟踪副作用以及读写关系。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `ATen/core/jit_type.h`, `torch/csrc/jit/ir/alias_analysis.h`, `torch/csrc/jit/ir/ir_views.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/peephole_list_idioms.h`, `torch/csrc/jit/passes/value_refinement_utils.h`, `torch/csrc/jit/runtime/slice_indices_adjust.h`
- External includes / 外部头文件: `limits`, `utility`
- Namespaces / 命名空间: `torch::jit`
- Representative symbols / 代表性符号: `normalizeIndex`, `ListLenRefiner`, `graph_`, `move`, `mutated_lists_`, `run`, `collectListsToRefine`, `block`, `empty`, `RefineListLens`, `...`
