# registerizer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/registerizer.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines or manipulates expression nodes in the Tensor Expression subsystem.
- **Purpose (CN)**: 定义或操作 Tensor Expression 子系统中的表达式节点。
## Line-by-Line Analysis / 逐行分析

### Lines 1-23
```cpp
#include <torch/csrc/jit/tensorexpr/registerizer.h>
#include <iostream>

namespace torch::jit::tensorexpr {
namespace registerizer {

// AccessInfo

void AccessInfo::addStore(
    const StorePtr& store,
    const std::shared_ptr<Scope>& scope) {
  block_ =
      block_ ? Block::getSharedParent(block_, scope->block()) : scope->block();

  // If there is already a usage and it's this store, that means the same
  // access is present in the RHS.
  firstUsageOverlapped_ |= first_usage_ == store;
  first_usage_ = first_usage_ ? block_->getEnclosedRoot(first_usage_) : store;
  last_usage_ = store;

  store_cost_ =
      IRSimplifier::simplify(alloc<Add>(store_cost_, immLike(store_cost_, 1)));
  stores_.push_back(store);
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/registerizer.h; standard-library headers such as iostream. The namespace declarations place the code inside torch::jit::tensorexpr, registerizer, matching the surrounding JIT subsystem. This chunk defines `addStore`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/registerizer.h；标准库头文件，如 iostream。 命名空间声明把代码放入 torch::jit::tensorexpr、registerizer 中，与周边 JIT 子系统保持一致。 这一段定义了 `addStore`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 25-44
```cpp
  conditionId_ = scope->conditionId();
  hiddenAccess_.reset();
}

void AccessInfo::addLoad(
    const LoadPtr& load,
    const std::shared_ptr<Scope>& scope,
    const StmtPtr& usage) {
  block_ =
      block_ ? Block::getSharedParent(block_, scope->block()) : scope->block();
  first_usage_ = first_usage_ ? block_->getEnclosedRoot(first_usage_) : usage;
  last_usage_ = usage;

  load_cost_ =
      IRSimplifier::simplify(alloc<Add>(load_cost_, immLike(load_cost_, 1)));
  loads_.push_back(load);

  conditionId_ = scope->conditionId();
  hiddenAccess_.reset();
}
```
- **EN**: This chunk defines `addLoad`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段定义了 `addLoad`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 46-67
```cpp
void AccessInfo::merge(const std::shared_ptr<AccessInfo>& other) {
  TORCH_INTERNAL_ASSERT(
      hash_ == other->hash(),
      buildErrorMessage(
          "Expected hashes to match in registerizer in the fuser."));
  TORCH_INTERNAL_ASSERT(
      indices_.size() == other->indices().size(),
      buildErrorMessage(
          "Expected ranks to match in registerizer in the fuser."));

  last_usage_ = other->last_usage();
  for (const auto& s : other->stores()) {
    stores_.push_back(s);
  }
  for (const auto& l : other->loads()) {
    loads_.push_back(l);
  }

  store_cost_ =
      IRSimplifier::simplify(alloc<Add>(store_cost_, other->store_cost()));
  load_cost_ =
      IRSimplifier::simplify(alloc<Add>(load_cost_, other->load_cost()));
```
- **EN**: This chunk defines `merge`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `merge`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 69-90
```cpp
  block_ = Block::getSharedParent(block_, other->block());
  // update first and last usage to be in the parent Block.
  first_usage_ = block_->getEnclosedRoot(first_usage_);
  last_usage_ = block_->getEnclosedRoot(last_usage_);
  hiddenAccess_.reset();
}

bool AccessInfo::overlaps(const std::shared_ptr<AccessInfo>& other) {
  // All accesses to a buf must have the same dimensionality.
  TORCH_INTERNAL_ASSERT(
      indices_.size() == other->indices().size(),
      buildErrorMessage(
          "Expected ranks to match in registerizer in the fuser."));

  auto& other_indices = other->indices();

  // They don't overlap if there is a guaranteed difference in any
  // dimension.
  bool overlap = true;
  for (size_t i = 0; i < indices_.size(); ++i) {
    ExprPtr diff = alloc<Sub>(indices_[i], other_indices[i]);
    diff = IRSimplifier::simplify(diff);
```
- **EN**: This chunk defines `overlaps`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `overlaps`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 92-113
```cpp
    if (diff->isConstant() && !immediateEquals(diff, 0)) {
      overlap = false;
      break;
    }
  }

  return overlap;
}

bool AccessInfo::dependsOnVar(const VarPtr& v) {
  VarFinder vf;
  for (const auto& i : indices_) {
    i->accept(&vf);
  }

  return vf.vars().count(v);
}

std::shared_ptr<AccessInfo> AccessInfo::cloneWithHiddenInfo(
    const std::shared_ptr<AccessInfo>& orig) {
  std::shared_ptr<AccessInfo> newInfo = std::make_shared<AccessInfo>(
      orig->hash(), orig->buf(), orig->indices(), orig->accessOrder());
```
- **EN**: This chunk defines `cloneWithHiddenInfo`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `cloneWithHiddenInfo`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 115-131
```cpp
  newInfo->block_ = orig->block_;
  newInfo->first_usage_ = orig->first_usage_;
  newInfo->last_usage_ = orig->last_usage_;
  newInfo->firstUsageOverlapped_ = orig->firstUsageOverlapped_;
  newInfo->store_cost_ = orig->store_cost_;
  newInfo->load_cost_ = orig->load_cost_;
  for (const auto& s : orig->stores_) {
    newInfo->stores_.push_back(s);
  }
  for (const auto& s : orig->loads_) {
    newInfo->loads_.push_back(s);
  }

  newInfo->conditionId_ = orig->conditionId_;
  newInfo->hiddenAccess_ = orig;
  return newInfo;
}
```
- **EN**: This chunk continues `cloneWithHiddenInfo` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `cloneWithHiddenInfo`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 133-151
```cpp
void AccessInfo::print() const {
  std::cout << "Access: " << *buf_ << '{';
  for (const auto& i : indices_) {
    std::cout << *i << ' ';
  }
  std::cout << "} stores: " << stores_.size() << " (" << *store_cost_ << ") -";
  std::cout << " loads: " << loads_.size() << " (" << *load_cost_ << ')';
  if (conditionId_) {
    std::cout << " cond: " << conditionId_;
  }

  std::cout << '\n';
}

// Scope

void Scope::closeAccess(const std::shared_ptr<AccessInfo>& info) {
  closedAccesses_.push_back(info);
}
```
- **EN**: This chunk defines `closeAccess`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `closeAccess`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 153-172
```cpp
AccessHashMap& Scope::getAccessMapByBuf(const BufPtr& b) {
  auto it = openAccesses_.find(b);
  if (it == openAccesses_.end()) {
    // create and return
    return openAccesses_[b];
  }

  return it->second;
}

void Scope::filterClosed() {
  std::erase_if(closedAccesses_, [](auto info) {
    return info->store_cost()->isConstant() &&
        immediateAs<int>(info->store_cost()) <= 1 &&
        info->load_cost()->isConstant() &&
        immediateAs<int>(info->load_cost()) <= 1;
  });
}

// RegisterizerAnalysis
```
- **EN**: This chunk defines `filterClosed`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `filterClosed`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 174-196
```cpp
void RegisterizerAnalysis::closeAccessIntoScope(
    const std::shared_ptr<AccessInfo>& info,
    const std::shared_ptr<Scope>& scope) {
  if (exprConditionals_.count(info->conditionId()) != 0) {
    return;
  }

  if (info->hiddenAccess()) {
    closeAccessIntoScope(info->hiddenAccess(), scope);
    return;
  }
  scope->closeAccess(info);
}

void RegisterizerAnalysis::visit(const ForPtr& v) {
  if (v->loop_options().is_gpu_block_index() ||
      v->loop_options().is_gpu_thread_index()) {
    throw malformed_input(
        "Registerization must occur after parallelism flattening");
  }

  auto parent = currentScope_;
  currentScope_ = std::make_shared<Scope>(v->body(), parent);
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 198-217
```cpp
  currentScope_->addLocalVar(v->var());

  stmtStack_.push_front(v);
  v->body()->accept(this);
  stmtStack_.pop_front();

  ExprPtr loopExtent =
      IRSimplifier::simplify(alloc<Sub>(v->stop(), v->start()));

  // now we need to see which accesses we can hoist out of the for loop, their
  // costs should be multiplied by the loop extent.
  for (auto& pair : currentScope_->openAccesses()) {
    if (pair.second.empty()) {
      continue;
    }

    auto& childAccesses = pair.second;

    for (auto it = childAccesses.begin(); it != childAccesses.end();) {
      std::shared_ptr<AccessInfo>& candidate = it->second;
```
- **EN**: This chunk continues `malformed_input` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `malformed_input`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 219-242
```cpp
      // If the access is open, but conditional, then we have a problem. It's
      // possible that an access at a higher scope could "unhide" the
      // conditional access, in which case we need to hoist. If there is no
      // access to this element at a higher scope then we cannot safely hoist.
      // We cannot know at this level whether that will or won't occur.
      //
      // The solution we take here is to split the space-time continuum, and
      // keep both versions of the access handy. If the hoisted access is not
      // used above, we'll fall back to using the hidden, conditional
      // AccessInfo - if it is, we'll delete the copy.
      if (candidate->conditionId() != 0) {
        candidate = AccessInfo::cloneWithHiddenInfo(candidate);
      }

      bool closed = false;
      // If this access depends on a locally scoped variable, it cannot be
      // hosted out of the loop.
      for (const auto& v : currentScope_->localVars()) {
        if (candidate->dependsOnVar(v)) {
          closeAccessIntoScope(candidate, currentScope_);
          closed = true;
          break;
        }
      }
```
- **EN**: This chunk continues `malformed_input` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `malformed_input`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 243-260
```cpp
      if (closed) {
        it = childAccesses.erase(it);
        continue;
      }

      // hoist!
      // By hoisting we pull the reads and writes out of the loop, and so the
      // benefit of registerizing this access is multiplied by the loop extent.
      candidate->setEnclosingBlock(parent->block());
      candidate->hoistCosts(loopExtent);

      // in the parent block, this loop Stmt is the insertion point for the
      // initializer and finalizer.
      candidate->setUsageMarks(v, v);

      ++it;
    }
  }
```
- **EN**: This chunk continues `malformed_input` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `malformed_input`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 262-280
```cpp
  // If an access is closed within a loop then it cannot be merged into an
  // existing open access, but will still close that existing access. This is
  // somewhat different from the regular merge so we need to handle closed
  // accesses first.
  mergeHiddenScope(true);

  // having hoisted, now we can merge normally.
  mergeCurrentScopeIntoParent();
}

void RegisterizerAnalysis::visit(const CondPtr& v) {
  ExprPtr condition = v->condition();
  BlockPtr true_stmt = v->true_stmt();
  BlockPtr false_stmt = v->false_stmt();

  stmtStack_.push_front(v);

  // condition is in the enclosing scope.
  condition->accept(this);
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。

### Lines 282-304
```cpp
  auto prev_scope = currentScope_;
  auto true_scope =
      std::make_shared<Scope>(true_stmt, prev_scope, ++conditionId_);
  auto false_scope =
      std::make_shared<Scope>(false_stmt, prev_scope, ++conditionId_);

  if (true_stmt) {
    currentScope_ = true_scope;
    true_stmt->accept(this);
    mergeHiddenScope(true);
    mergeCurrentScopeIntoParent();
  }
  if (false_stmt) {
    currentScope_ = false_scope;
    false_stmt->accept(this);
    mergeHiddenScope(true);
    mergeCurrentScopeIntoParent();
  }

  // TODO: even though both scopes are conditional, we can merge accesses if
  // they totally overlap in both branches, since we can guarantee one
  // definition will be hit. We might need a 3-way merge? Not as simple as
  // merging the true and false scopes together first.
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 306-324
```cpp
  stmtStack_.pop_front();
}

// IfThenElses are just like Conds except they are not Stmts, which means no
// registerization can occur internally. However, the first reference to an
// access can occur within one if its visible outside the condition.
void RegisterizerAnalysis::visit(const IfThenElsePtr& v) {
  ExprPtr condition = v->condition();
  ExprPtr true_value = v->true_value();
  ExprPtr false_value = v->false_value();

  // condition is in enclosing scope.
  condition->accept(this);

  auto prev_scope = currentScope_;
  auto true_scope =
      std::make_shared<Scope>(prev_scope->block(), prev_scope, ++conditionId_);
  auto false_scope =
      std::make_shared<Scope>(prev_scope->block(), prev_scope, ++conditionId_);
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。

### Lines 326-348
```cpp
  // We store IfThenElse scopes in a global map, which we use to prevent closing
  // any access that would require inserting statements in the values, which
  // cannot enclose Stmts.
  exprConditionals_.insert(true_scope->conditionId());
  exprConditionals_.insert(false_scope->conditionId());

  if (true_value) {
    currentScope_ = true_scope;
    true_value->accept(this);
    mergeHiddenScope(false);
    mergeCurrentScopeIntoParent();
  }

  if (false_value) {
    currentScope_ = false_scope;
    false_value->accept(this);
    mergeHiddenScope(false);
    mergeCurrentScopeIntoParent();
  }
}

void RegisterizerAnalysis::visit(const LetPtr& v) {
  currentScope_->addLocalVar(v->var());
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 350-371
```cpp
  stmtStack_.push_front(v);
  v->value()->accept(this);
  stmtStack_.pop_front();
}

void RegisterizerAnalysis::visit(const BlockPtr& v) {
  auto prev_scope = currentScope_;
  if (currentScope_->block() != v) {
    currentScope_ = std::make_shared<Scope>(v, prev_scope);
  }

  stmtStack_.push_front(v);

  for (const auto& s : *v) {
    s->accept(this);
    if (currentScope_->block() != v) {
      // merge the inner block's accesses into this Block's accesses.
      mergeCurrentScopeIntoParent();
    }
  }

  stmtStack_.pop_front();
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 373-391
```cpp
  if (prev_scope->block() == nullptr) {
    // close any open candidates.
    for (auto& p1 : currentScope_->openAccesses()) {
      for (auto& p2 : p1.second) {
        closeAccessIntoScope(p2.second, currentScope_);
      }
    }
  }
}

void RegisterizerAnalysis::visit(const StorePtr& v) {
  stmtStack_.push_front(v);
  v->value()->accept(this);
  stmtStack_.pop_front();

  if (v->indices().empty()) {
    // already a scalar.
    return;
  }
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 393-411
```cpp
  // hash the Store:
  SimplifierHashType accessHash = hasher_.hash(v->buf());
  for (const auto& i : v->indices()) {
    accessHash = hasher_.hash_combine(accessHash, i);
  }

  auto& bufAccesses = currentScope_->getAccessMapByBuf(v->buf());
  auto candidateIt = bufAccesses.find(accessHash);

  // If an identical access already exists, add this Store to it.
  if (candidateIt != bufAccesses.end()) {
    candidateIt->second->addStore(v, currentScope_);
    return;
  }

  // Otherwise make a new AccessInfo and add this store.
  auto info = std::make_shared<AccessInfo>(
      accessHash, v->buf(), v->indices(), accessOrder_++);
  info->addStore(v, currentScope_);
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 413-435
```cpp
  // This new access may overlap an existing open access, in which case we need
  // to close the older of the two.
  bool alreadyOverlapped = false;
  for (auto it = bufAccesses.begin(); it != bufAccesses.end();) {
    auto other = it->second;
    if (info->overlaps(other)) {
      if (other->last_usage() == v) {
        // we are already overlapped by an access in the RHS.
        alreadyOverlapped = true;
      }
      closeAccessIntoScope(other, currentScope_);
      it = bufAccesses.erase(it);
    } else {
      ++it;
    }
  }

  if (alreadyOverlapped) {
    closeAccessIntoScope(info, currentScope_);
  } else {
    bufAccesses.emplace(accessHash, info);
  }
}
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 437-458
```cpp
void RegisterizerAnalysis::visit(const LoadPtr& v) {
  if (v->indices().empty()) {
    // already a scalar.
    return;
  }
  // hash the Load:
  SimplifierHashType accessHash = hasher_.hash(v->buf());
  for (const auto& i : v->indices()) {
    accessHash = hasher_.hash_combine(accessHash, i);
  }

  auto& bufAccesses = currentScope_->getAccessMapByBuf(v->buf());
  auto candidateIt = bufAccesses.find(accessHash);
  if (candidateIt != bufAccesses.end()) {
    // found the right access, can just insert.
    candidateIt->second->addLoad(v, currentScope_, stmtStack_.front());
    return;
  }

  std::shared_ptr<AccessInfo> info = std::make_shared<AccessInfo>(
      accessHash, v->buf(), v->indices(), accessOrder_++);
  info->addLoad(v, currentScope_, stmtStack_.front());
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 460-476
```cpp
  bool alreadyOverlapped = false;
  // This new access may overlap an existing open access, in which case we need
  // to finalize the older of the two.
  for (auto it = bufAccesses.begin(); it != bufAccesses.end();) {
    auto other = it->second;
    if (info->overlaps(other)) {
      if (info->last_usage() == other->last_usage()) {
        // if these two accesses are from the same Stmt, they already overlap
        // each other.
        alreadyOverlapped = true;
      }
      closeAccessIntoScope(other, currentScope_);
      it = bufAccesses.erase(it);
    } else {
      ++it;
    }
  }
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 478-500
```cpp
  if (alreadyOverlapped) {
    closeAccessIntoScope(info, currentScope_);
  } else {
    bufAccesses.emplace(accessHash, info);
  }
}

// Loop and Conditional scopes are different in that it may or may not be
// possible to hoist the initializer of a scalar variable outside the block
// depending on if we can tell that the Buffer access is valid outside. This is
// tricky because the access that demonstrates this may be later in the tree and
// we haven't encountered it yet.
// The allowClosed flag indicates whether we want to keep the closed accesses
// (For and Cond), or not (IfThenElse).
void RegisterizerAnalysis::mergeHiddenScope(bool allowClosed) {
  // The rule is that if any access is closed within the conditional block, any
  // accesses which overlap it must also be closed - since their initializer
  // cannot be hoisted out of the block.
  std::list<std::shared_ptr<AccessInfo>> newClosed;
  for (auto& info : currentScope_->closedAccesses()) {
    auto& candidates = currentScope_->getAccessMapByBuf(info->buf());
    for (auto it = candidates.begin(); it != candidates.end();) {
      std::shared_ptr<AccessInfo> candidate = it->second;
```
- **EN**: This chunk defines `mergeHiddenScope`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `mergeHiddenScope`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 502-522
```cpp
      if (info->hash() == candidate->hash() || info->overlaps(candidate)) {
        newClosed.push_back(candidate);
        it = candidates.erase(it);
      } else {
        ++it;
      }
    }
  }

  if (allowClosed) {
    for (auto& info : newClosed) {
      closeAccessIntoScope(info, currentScope_);
    }
  } else {
    currentScope_->closedAccesses().clear();
  }
}

// Merge currentScope_ into it's parent, and make parent the new currentScope_.
void RegisterizerAnalysis::mergeCurrentScopeIntoParent() {
  auto parent = currentScope_->parent();
```
- **EN**: This chunk defines `mergeCurrentScopeIntoParent`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `mergeCurrentScopeIntoParent`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 524-543
```cpp
  // copy across current closed accesses, merging / closing as necessary
  for (auto& candidate : currentScope_->closedAccesses()) {
    auto& parentAccesses = parent->getAccessMapByBuf(candidate->buf());

    auto parentIt = parentAccesses.find(candidate->hash());
    if (parentIt != parentAccesses.end()) {
      std::shared_ptr<AccessInfo> pCandidate = parentIt->second;

      // if the access is closed inside a condition, it can only be merged if
      // the parent is in the same condition.
      if (candidate->conditionId() &&
          pCandidate->conditionId() != candidate->conditionId()) {
        // the parent's access must be closed.
        closeAccessIntoScope(pCandidate, parent);
        parentAccesses.erase(parentIt);

        // the children access inserted into the parent scope.
        closeAccessIntoScope(candidate, parent);
        continue;
      }
```
- **EN**: This chunk continues `mergeCurrentScopeIntoParent` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `mergeCurrentScopeIntoParent`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 545-567
```cpp
      // merge totally overlapping accesses.
      parentIt->second->merge(candidate);
      closeAccessIntoScope(parentIt->second, parent);
      parentAccesses.erase(parentIt);
      continue;
    }

    // we didn't find a perfect match, but we need to check all open accesses of
    // this buf for partial overlap.
    for (auto it = parentAccesses.begin(); it != parentAccesses.end();) {
      std::shared_ptr<AccessInfo> pCandidate = it->second;
      // Partial overlap of parent access: close parent access.
      if (candidate->overlaps(pCandidate)) {
        closeAccessIntoScope(pCandidate, parent);
        it = parentAccesses.erase(it);
        continue;
      }
      ++it;
    }

    // Insert the children closed access into the parent scope.
    closeAccessIntoScope(candidate, parent);
  }
```
- **EN**: This chunk continues `mergeCurrentScopeIntoParent` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `mergeCurrentScopeIntoParent`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 569-585
```cpp
  // copy across current open accesses, merging as necessary.
  // for each Buf with an open access:
  for (auto& pair : currentScope_->openAccesses()) {
    BufPtr buf = pair.first;
    if (pair.second.empty()) {
      continue;
    }

    auto& parentAccesses = parent->getAccessMapByBuf(buf);

    // for each open access in the child scope for this Buf:
    for (auto& hpair : pair.second) {
      bool handled{false};
      std::shared_ptr<AccessInfo> candidate = hpair.second;

      for (auto it = parentAccesses.begin(); it != parentAccesses.end();) {
        std::shared_ptr<AccessInfo> pCandidate = it->second;
```
- **EN**: This chunk continues `mergeCurrentScopeIntoParent` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `mergeCurrentScopeIntoParent`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 587-609
```cpp
        // If it completely overlaps then merge.
        if (candidate->hash() == pCandidate->hash()) {
          // if both accesses are found in conditional blocks, they cannot be
          // merged, but the earlier must be closed.
          if (pCandidate->conditionId() != parent->conditionId() &&
              pCandidate->conditionId() != candidate->conditionId()) {
            closeAccessIntoScope(pCandidate, parent);
            it = parentAccesses.erase(it);
            continue;
          }
          pCandidate->merge(candidate);
          handled = true;
          ++it;
          continue;
        }

        // It can overlap an access in the parent: close the parent access.
        // The child access may still be open.
        if (candidate->overlaps(pCandidate)) {
          closeAccessIntoScope(pCandidate, parent);
          it = parentAccesses.erase(it);
          continue;
        }
```
- **EN**: This chunk continues `mergeCurrentScopeIntoParent` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `mergeCurrentScopeIntoParent`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 611-633
```cpp
        ++it;
      }

      // If this access depends on a locally scoped variable, it cannot be
      // lifted out of the loop.
      for (const auto& v : currentScope_->localVars()) {
        if (candidate->dependsOnVar(v)) {
          closeAccessIntoScope(candidate, parent);
          handled = true;
          break;
        }
      }

      if (!handled) {
        // If the inner scope was not conditional, but the outer scope is: all
        // current accesses are now conditional in the parent scope.
        if (candidate->conditionId() == 0) {
          candidate->setConditionId(parent->conditionId());
        }
        parentAccesses[candidate->hash()] = candidate;
      }
    }
  }
```
- **EN**: This chunk continues `mergeCurrentScopeIntoParent` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `mergeCurrentScopeIntoParent`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 635-656
```cpp
  currentScope_ = parent;
}

std::vector<std::shared_ptr<AccessInfo>> RegisterizerAnalysis::getCandidates() {
  currentScope_->filterClosed();
  std::sort(
      currentScope_->closedAccesses().begin(),
      currentScope_->closedAccesses().end(),
      [](auto i1, auto i2) { return i1->accessOrder() < i2->accessOrder(); });
  return currentScope_->closedAccesses();
}

// RegisterizerReplacer

ExprPtr RegisterizerReplacer::mutate(const LoadPtr& v) {
  auto it = loadToAccess_.find(v);
  if (it == loadToAccess_.end()) {
    // This access cannot be registerized.
    return v;
  }

  auto& info = it->second;
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 658-675
```cpp
  return info->replacement().var;
}

StmtPtr RegisterizerReplacer::mutate(const StorePtr& v) {
  if (eliminatedIntializers_.count(v) != 0) {
    // This store is the initializer for a scalar var that is already inserted.
    return nullptr;
  }

  auto it = storeToAccess_.find(v);
  if (it == storeToAccess_.end()) {
    // This access cannot be registerized.
    return IRMutator::mutate(v);
  }

  auto& info = it->second;

  ExprPtr new_val = v->value()->accept_mutator(this);
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 677-699
```cpp
  v->set_value(new_val);
  v->set_buf(info->replacement().var_wrapper);
  v->set_indices({});
  return v;
}

StmtPtr RegisterizerReplacer::mutate(const BlockPtr& v) {
  auto& scope = parentToAccesses_[v];

  std::vector<StmtPtr> stmts;
  for (const StmtPtr& stmt : v->stmts()) {
    {
      // Insert the initializer for any Scalars scoped to this block.
      auto it = scope.initializerPoints_.find(stmt);
      if (it != scope.initializerPoints_.end()) {
        for (auto& info : it->second) {
          StmtPtr initializer =
              info->replacement().initializer->accept_mutator(this);
          stmts.push_back(initializer);
        }
        scope.initializerPoints_.erase(it);
      }
    }
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 701-721
```cpp
    StmtPtr stmt_new = stmt->accept_mutator(this);
    if (stmt_new) {
      if (stmt_new->get_parent()) {
        stmt_new = Stmt::clone(stmt_new);
      }
      stmts.push_back(stmt_new);
    }

    {
      // Insert the finalizer for any Scalars scoped to this block.
      auto it = scope.finalizePoints_.find(stmt);
      if (it != scope.finalizePoints_.end()) {
        for (auto& info : it->second) {
          StorePtr finalizer = alloc<Store>(
              info->buf(), info->indices(), info->replacement().var);
          stmts.push_back(finalizer);
        }
        scope.finalizePoints_.erase(it);
      }
    }
  }
```
- **EN**: This chunk continues `mutate` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `mutate`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 723-739
```cpp
  return alloc<Block>(stmts);
}

void RegisterizerReplacer::buildReplacements() {
  // Traverse the list of replacements, creating vars and updating our local
  // maps.
  for (auto& info : infoSet_) {
    VarPtr v = alloc<Var>(
        info->buf()->name_hint() + "_" +
            std::to_string(getBufferAccessCount(info->buf())),
        info->buf()->dtype());

    info->replacement().var = v;

    // we need to wrap the Var in a Buf so we can Load or Store it.
    info->replacement().var_wrapper =
        alloc<Buf>(v, std::vector<ExprPtr>({}), info->buf()->dtype());
```
- **EN**: This chunk defines `buildReplacements`, which constructs derived state from the current inputs and invariants. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `buildReplacements`，其作用是根据当前输入和约束构建派生状态。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 741-758
```cpp
    bool first = true;
    for (const auto& s : info->stores()) {
      if (first && info->first_usage() == s && !info->firstUsageOverlapped()) {
        info->replacement().initializer = alloc<Let>(v, s->value());
        eliminatedIntializers_.insert(s);
      } else {
        storeToAccess_[s] = info;
      }

      first = false;
    }

    for (const auto& s : info->loads()) {
      loadToAccess_[s] = info;
    }

    auto& scope = parentToAccesses_[info->block()];
    scope.initializerPoints_[info->first_usage()].push_back(info);
```
- **EN**: This chunk continues `buildReplacements` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `buildReplacements`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 760-778
```cpp
    // Only finalize if the scalar is written.
    if (!info->stores().empty()) {
      // push front to finalize in reverse order of encounter.
      scope.finalizePoints_[info->last_usage()].push_front(info);
    }

    // create a default initializer by reading the access.
    if (info->replacement().initializer == nullptr) {
      info->replacement().initializer = alloc<Let>(
          v, alloc<Load>(info->buf()->dtype(), info->buf(), info->indices()));
    }
  }
}

} // namespace registerizer

// Apply scalar replacement to all accesses in s.
StmtPtr registerize(StmtPtr s) {
  s = IRSimplifier::simplify(s);
```
- **EN**: This chunk defines `registerize`, which registers schemas, operators, or passes with the surrounding runtime. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `registerize`，其作用是向周边运行时注册 schema、算子或 pass。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 780-794
```cpp
  // The outermost node must be a Block so we have somewhere to put outer scope
  // scalars.
  if (!to<Block>(s)) {
    s = alloc<Block>(std::vector<StmtPtr>({s}));
  }
  registerizer::RegisterizerAnalysis analysis;
  s->accept(&analysis);
  auto candidates = analysis.getCandidates();

  registerizer::RegisterizerReplacer replacer(candidates);
  s = s->accept_mutator(&replacer);
  return s;
}

} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk defines `replacer`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `replacer`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **addStore**
  - EN: `addStore` is a central symbol declared or implemented in this file.
  - CN: `addStore` 是本文件声明或实现的核心符号。
- **addLoad**
  - EN: `addLoad` is a central symbol declared or implemented in this file.
  - CN: `addLoad` 是本文件声明或实现的核心符号。
- **IR simplification**
  - EN: Rewrites expressions and statements into simpler but equivalent forms.
  - CN: 把表达式和语句重写为更简单但等价的形式。
- **Static analysis**
  - EN: Infers structural properties that later optimizations depend on.
  - CN: 推断后续优化所依赖的结构属性。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/registerizer.h`
- **Standard library / 标准库**: `iostream`
- **Primary symbols in this file / 本文件核心符号**: `addStore`, `addLoad`, `merge`, `overlaps`, `dependsOnVar`, `cloneWithHiddenInfo`, `print`, `closeAccess`
