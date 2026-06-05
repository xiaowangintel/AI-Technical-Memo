# mem_dependency_checker.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/mem_dependency_checker.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines or manipulates expression nodes in the Tensor Expression subsystem.
- **Purpose (CN)**: 定义或操作 Tensor Expression 子系统中的表达式节点。
## Line-by-Line Analysis / 逐行分析

### Lines 1-33
```cpp
#include <torch/csrc/jit/tensorexpr/mem_dependency_checker.h>

#include <c10/util/irange.h>

#include <fstream>
#include <iostream>
#include <utility>

namespace torch::jit::tensorexpr::analysis {

const char* AccessToString(AccessType a) {
  switch (a) {
    case AccessType::Input:
      return "Input";
    case AccessType::Output:
      return "Output";
    case AccessType::Load:
      return "Load";
    case AccessType::Store:
      return "Store";
    case AccessType::Call:
      return "Call";
    case AccessType::AtomicAdd:
      return "AtomicAdd";
    case AccessType::Alloc:
      return "Alloc";
    case AccessType::Free:
      return "Free";
    default:
      break;
  }
  return "Unknown";
}
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/mem_dependency_checker.h; ATen/c10 facilities such as c10/util/irange.h; standard-library headers such as fstream, iostream, utility. The namespace declarations place the code inside torch::jit::tensorexpr::analysis, matching the surrounding JIT subsystem. This chunk defines `AccessToString`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/mem_dependency_checker.h；ATen/c10 基础设施，如 c10/util/irange.h；标准库头文件，如 fstream、iostream、utility。 命名空间声明把代码放入 torch::jit::tensorexpr::analysis 中，与周边 JIT 子系统保持一致。 这一段定义了 `AccessToString`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 35-62
```cpp
static void getDependencyChain(
    const std::shared_ptr<AccessInfo>& info,
    DependencySet& dependencies) {
  if (!dependencies.insert(info).second) {
    return;
  }

  for (auto& dep : info->dependencies()) {
    getDependencyChain(dep.second, dependencies);
  }
}

static void getDependentsChain(
    const std::shared_ptr<AccessInfo>& info,
    DependencySet& dependents) {
  if (!dependents.insert(info).second) {
    return;
  }

  for (auto& dep : info->dependents()) {
    getDependencyChain(dep.second, dependents);
  }
}

// AccessInfo

std::vector<ExprPtr> AccessInfo::getIndices() const {
  std::vector<ExprPtr> indices;
```
- **EN**: This chunk defines `getIndices`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getIndices`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 64-92
```cpp
  if (expr_) {
    if (auto load = to<Load>(expr_)) {
      indices = load->indices();
    }
  } else {
    if (auto store = to<Store>(stmt_)) {
      indices = store->indices();
    }
  }
  return indices;
}

void AccessInfo::addDependency(const std::shared_ptr<AccessInfo>& write) {
  auto res = dependencies_.emplace(write->id(), write);
  TORCH_INTERNAL_ASSERT(
      res.second,
      buildErrorMessage("Duplicate entry in mem dep checker in the fuser."));
}

void AccessInfo::addDependent(const std::shared_ptr<AccessInfo>& read) {
  auto res = dependents_.emplace(read->id(), read);
  TORCH_INTERNAL_ASSERT(
      res.second,
      buildErrorMessage("Duplicate entry in mem dep checker in the fuser."));
}

bool AccessInfo::hasDependency(const std::shared_ptr<AccessInfo>& info) const {
  return dependencies_.count(info->id()) != 0;
}
```
- **EN**: This chunk defines `hasDependency`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `hasDependency`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 94-124
```cpp
DependencySet AccessInfo::getDirectDependencies() {
  DependencySet res;
  for (auto& depPair : dependencies_) {
    res.insert(depPair.second);
  }
  return res;
}

DependencySet AccessInfo::getIndirectDependencies() {
  DependencySet res;
  for (auto& depPair : dependencies_) {
    getDependencyChain(depPair.second, res);
  }
  return res;
}

DependencySet AccessInfo::getDirectDependents() {
  DependencySet res;
  for (auto& depPair : dependents_) {
    res.insert(depPair.second.lock());
  }
  return res;
}

DependencySet AccessInfo::getIndirectDependents() {
  DependencySet res;
  for (auto& depPair : dependencies_) {
    getDependentsChain(depPair.second, res);
  }
  return res;
}
```
- **EN**: This chunk defines `getIndirectDependents`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getIndirectDependents`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 126-159
```cpp
bool AccessInfo::isRead() const {
  switch (type_) {
    case AccessType::Output:
    case AccessType::Load:
    case AccessType::Call:
    case AccessType::AtomicAdd:
      return true;
    default:
      break;
  }
  return false;
}

bool AccessInfo::isWrite() const {
  switch (type_) {
    case AccessType::Input:
    case AccessType::Store:
    case AccessType::AtomicAdd:
    case AccessType::Alloc:
    case AccessType::Free:
      return true;
    default:
      break;
  }
  return false;
}

void AccessInfo::print() const {
  std::cout << id_ << ". " << AccessToString(type_) << ": " << *var_ << '[';
  if (!bounds_.empty()) {
    for (size_t i = 0; i < bounds_.size() - 1; ++i) {
      bounds_[i].print();
      std::cout << ", ";
    }
```
- **EN**: This chunk defines `print`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `print`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 161-193
```cpp
    size_t i = bounds_.size() - 1;
    bounds_[i].print();
  }
  std::cout << ']';

  if (!dependencies_.empty()) {
    std::cout << " - depends on: ";
    for (auto& pair : dependencies_) {
      std::cout << pair.second->id() << ' ';
    }
  }

  if (!dependents_.empty()) {
    std::cout << " - dependents: ";
    for (auto& pair : dependents_) {
      std::cout << pair.second.lock()->id() << ' ';
    }
  }

  std::cout << '\n';
}

void AccessInfo::dumpDOT(std::ostream& os) const {
  if (type_ == AccessType::Input || type_ == AccessType::Output ||
      type_ == AccessType::Alloc) {
    os << 'n' << id_ << " [\n";
    os << "label = \"" << AccessToString(type_) << "\\n " << *var_ << '[';
    if (!bounds_.empty()) {
      for (size_t i = 0; i < bounds_.size() - 1; ++i) {
        os << *IRSimplifier::simplify(
                  alloc<Add>(bounds_[i].end, immLike(bounds_[i].end, 1)))
           << ", ";
      }
```
- **EN**: This chunk defines `dumpDOT`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `dumpDOT`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 195-230
```cpp
      size_t i = bounds_.size() - 1;
      os << *IRSimplifier::simplify(
          alloc<Add>(bounds_[i].end, immLike(bounds_[i].end, 1)));
      os << "]\"\n ";
    }
    if (isWrite()) {
      os << "\tshape = \"invhouse\"\n";
    } else {
      os << "\tshape = \"house\"\n";
    }
  } else {
    os << 'n' << id_ << " [\n";
    os << "label = \"" << AccessToString(type_) << " (#" << id_ << ")\\n";
    os << "buf : " << *var_ << "\\n";
    os << "bounds : [";
    if (!bounds_.empty()) {
      for (size_t i = 0; i < bounds_.size() - 1; ++i) {
        os << '(' << *bounds_[i].start << ", " << *bounds_[i].end << "), ";
      }

      size_t i = bounds_.size() - 1;
      os << '(' << *bounds_[i].start << ", " << *bounds_[i].end << ")]";
    }
    os << "\"\n";
    os << "\tshape = \"box\"\n";
  }
  os << "\tstyle=\"filled\"\n";
  os << "\tcolor=\"" << AccessTypeColour() << "\"\n";
  std::string edgeColour;
  if (isWrite()) {
    edgeColour = "cornflowerblue";
  } else {
    edgeColour = "goldenrod";
  }
  os << "]\n";
  for (auto& pair : dependencies_) {
```
- **EN**: This chunk continues `dumpDOT` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `dumpDOT`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 231-260
```cpp
    os << 'n' << pair.second->id() << " -> " << 'n' << id_ << " [color=\""
       << edgeColour << "\"]\n";
  }
}

const char* AccessInfo::AccessTypeColour() const {
  switch (type_) {
    case AccessType::Input:
    case AccessType::Output:
      return "palegreen";
    case AccessType::Load:
      return "peachpuff";
    case AccessType::Store:
      return "dodgerblue";
    case AccessType::Call:
      return "violet";
    case AccessType::Alloc:
    case AccessType::Free:
      return "sandybrown";
    default:
      break;
  }
  return "white";
}

// MemDependencyChecker
//
MemDependencyChecker::MemDependencyChecker() {
  currentScope_ = std::make_shared<Scope>(nullptr, nullptr);
}
```
- **EN**: This chunk defines `AccessTypeColour`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `AccessTypeColour`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 262-296
```cpp
MemDependencyChecker::MemDependencyChecker(
    const std::unordered_set<BufPtr>& inputs,
    const std::unordered_set<BufPtr>& outputs) {
  for (const auto& s : inputs) {
    inputs_[s] = nullptr;
  }
  for (const auto& s : outputs) {
    outputs_[s] = nullptr;
  }

  currentScope_ = std::make_shared<Scope>(nullptr, nullptr);
}

MemDependencyChecker::MemDependencyChecker(
    const std::vector<BufHandle>& inputs,
    const std::vector<BufHandle>& outputs) {
  for (auto& s : inputs) {
    inputs_[s.node()] = nullptr;
  }
  for (auto& s : outputs) {
    outputs_[s.node()] = nullptr;
  }

  currentScope_ = std::make_shared<Scope>(nullptr, nullptr);
}

bool MemDependencyChecker::allowLoopExecutionOrderAnalysis(bool allow) {
  std::swap(allowExecutionOrderAnalysis_, allow);
  return allow;
}

const std::vector<std::shared_ptr<AccessInfo>>& MemDependencyChecker::
    getHistory() const {
  return currentScope_->accesses_;
}
```
- **EN**: This chunk defines `allowLoopExecutionOrderAnalysis`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `allowLoopExecutionOrderAnalysis`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 298-330
```cpp
void MemDependencyChecker::dumpDAG(const std::string& filename) const {
  std::ofstream dotfile(filename);

  dotfile << "digraph {\n";
  for (auto& wi : getHistory()) {
    wi->dumpDOT(dotfile);
  }
  dotfile << "}\n";
  dotfile.close();
}

// dependsDirectly, dependsIndirectly and friends:

DependencySet MemDependencyChecker::getAllWriteDependencies(
    const DependencySet& products) {
  DependencySet writes;

  for (auto& info : products) {
    DependencySet dependencies;
    getDependencyChain(info, dependencies);
    for (auto& other : dependencies) {
      if (other->isWrite()) {
        writes.insert(other);
      }
    }
  }

  return writes;
}

bool MemDependencyChecker::dependsDirectly(const ExprPtr& A, const StmtPtr& B) {
  return dependsDirectlyHelper(A, B);
}
```
- **EN**: This chunk defines `dependsDirectly`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `dependsDirectly`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 332-364
```cpp
bool MemDependencyChecker::dependsDirectly(const StmtPtr& A, const StmtPtr& B) {
  return dependsDirectlyHelper(A, B);
}

bool MemDependencyChecker::dependsDirectly(const BufPtr& O, const StmtPtr& B) {
  auto outputAccess = output(O);
  auto bWrites = getAllWritesWithin(B);

  for (auto& depPair : outputAccess->dependencies()) {
    if (bWrites.count(depPair.second) != 0) {
      return true;
    }
  }

  return false;
}

bool MemDependencyChecker::dependsDirectly(const StmtPtr& A, const BufPtr& I) {
  auto aReads = getAllReadsWithin(A);
  auto inputAccess = input(I);

  for (auto& depPair : inputAccess->dependents()) {
    if (aReads.count(depPair.second) != 0) {
      return true;
    }
  }

  return false;
}

bool MemDependencyChecker::dependsDirectly(const ExprPtr& A, const BufPtr& I) {
  auto aReads = getAllReadsWithin(A);
  auto inputAccess = input(I);
```
- **EN**: This chunk defines `dependsDirectly`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `dependsDirectly`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 366-399
```cpp
  for (auto& depPair : inputAccess->dependents()) {
    if (aReads.count(depPair.second) != 0) {
      return true;
    }
  }

  return false;
}

bool MemDependencyChecker::dependsDirectly(
    const std::shared_ptr<AccessInfo>& A,
    const std::shared_ptr<AccessInfo>& B) {
  return A->hasDependency(B) && B->isWrite();
}

bool MemDependencyChecker::dependsIndirectly(
    const ExprPtr& A,
    const StmtPtr& B) {
  return dependsIndirectlyHelper(A, B);
}

bool MemDependencyChecker::dependsIndirectly(
    const StmtPtr& A,
    const StmtPtr& B) {
  return dependsIndirectlyHelper(A, B);
}

bool MemDependencyChecker::dependsIndirectly(
    const BufPtr& O,
    const StmtPtr& B) {
  auto outputAccess = output(O);

  DependencySet dependencies;
  getDependencyChain(outputAccess, dependencies);
```
- **EN**: This chunk defines `dependsIndirectly`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `dependsIndirectly`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 401-435
```cpp
  auto bWrites = getAllWritesWithin(B);
  for (auto& dep : dependencies) {
    if (bWrites.count(dep) != 0) {
      return true;
    }
  }

  return false;
}

bool MemDependencyChecker::dependsIndirectly(
    const StmtPtr& A,
    const BufPtr& I) {
  auto aReads = getAllReadsWithin(A);
  auto inputAccess = input(I);

  auto aDeps = getAllWriteDependencies(aReads);

  return aDeps.count(inputAccess) != 0;
}

bool MemDependencyChecker::dependsIndirectly(
    const ExprPtr& A,
    const BufPtr& I) {
  auto aReads = getAllReadsWithin(A);
  auto inputAccess = input(I);

  auto aDeps = getAllWriteDependencies(aReads);

  return aDeps.count(inputAccess) != 0;
}

bool MemDependencyChecker::dependsIndirectly(const BufPtr& O, const BufPtr& I) {
  auto outputAccess = output(O);
  auto inputAccess = input(I);
```
- **EN**: This chunk defines `dependsIndirectly`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `dependsIndirectly`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 437-465
```cpp
  return dependsIndirectly(outputAccess, inputAccess);
}

bool MemDependencyChecker::dependsIndirectly(
    const std::shared_ptr<AccessInfo>& A,
    const std::shared_ptr<AccessInfo>& B) {
  if (!B->isWrite()) {
    return false;
  }

  DependencySet dependencies;
  getDependencyChain(A, dependencies);
  if (dependencies.count(B) == 0) {
    return false;
  }

  return true;
}

std::shared_ptr<AccessInfo> MemDependencyChecker::accessFor(
    const StmtPtr& A) const {
  auto bound = stmtToAccess_.equal_range(A);
  for (auto it = bound.first; it != bound.second; ++it) {
    if (it->second->expr() == nullptr) {
      return it->second;
    }
  }
  return nullptr;
}
```
- **EN**: This chunk defines `accessFor`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `accessFor`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 467-498
```cpp
std::shared_ptr<AccessInfo> MemDependencyChecker::accessFor(
    const ExprPtr& A) const {
  // TODO exprs can have multiple accesses... we're returning the first but that
  // isn't great. Can't do much here.
  auto bound = exprToAccess_.equal_range(A);
  if (bound.first != exprToAccess_.end()) {
    return bound.first->second;
  }

  return nullptr;
}

std::unordered_set<std::shared_ptr<AccessInfo>> MemDependencyChecker::
    accessesWithin(const StmtPtr& A) const {
  auto it = scopeToAccesses_.find(A);
  if (it != scopeToAccesses_.end()) {
    return std::unordered_set<std::shared_ptr<AccessInfo>>(
        it->second.begin(), it->second.end());
  }

  std::unordered_set<std::shared_ptr<AccessInfo>> ret;
  auto bound = stmtToAccess_.equal_range(A);
  for (auto it = bound.first; it != bound.second; ++it) {
    ret.insert(it->second);
  }
  return ret;
}

std::unordered_set<std::shared_ptr<AccessInfo>> MemDependencyChecker::
    accessesWithin(const ExprPtr& A) const {
  return {accessFor(A)};
}
```
- **EN**: This chunk defines `accessFor`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `accessFor`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 500-532
```cpp
std::shared_ptr<AccessInfo> MemDependencyChecker::input(const BufPtr& b) const {
  auto it = inputs_.find(b);
  if (it == inputs_.end()) {
    return nullptr;
  }
  return it->second;
}

std::shared_ptr<AccessInfo> MemDependencyChecker::output(
    const BufPtr& b) const {
  auto it = outputs_.find(b);
  if (it == outputs_.end()) {
    return nullptr;
  }
  return it->second;
}

// Node visitors:

void MemDependencyChecker::visit(const StorePtr& v) {
  StmtPtr last = lastStmt_;
  lastStmt_ = v;
  v->value()->accept(this);

  for (const ExprPtr& ind : v->indices()) {
    ind->accept(this);
  }
  lastStmt_ = last;

  // Create a new AccessInfo for the store.
  VarPtr var = v->buf()->base_handle();
  auto info = std::make_shared<AccessInfo>(
      nextAccess_++, AccessType::Store, v, var, getIndicesBounds(v->indices()));
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 534-560
```cpp
  // Add a dependency to any accesses that are within the scope of this store
  // (ie. the RHS).
  auto bound = stmtToAccess_.equal_range(v);
  for (auto it = bound.first; it != bound.second; ++it) {
    info->addDependency(it->second);
    it->second->addDependent(info);
  }

  stmtToAccess_.emplace(v, info);

  // This write is open, and will close any open writes that it totally
  // overlaps.
  auto& history = currentScope_->openWrites_[var];
  updateWriteHistory(history, info, info->id());
  currentScope_->accesses_.push_back(info);
}

void MemDependencyChecker::visit(const LoadPtr& v) {
  // Create a temporary scope to hold any loads that occur within the indices of
  // this load.
  auto indicesScope =
      std::make_shared<Scope>(currentScope_->block, currentScope_);
  currentScope_ = indicesScope;

  for (const ExprPtr& ind : v->indices()) {
    ind->accept(this);
  }
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 562-594
```cpp
  // Create a new AccessInfo for the load.
  VarPtr var = v->buf()->base_handle();
  auto load = std::make_shared<AccessInfo>(
      nextAccess_++,
      AccessType::Load,
      v,
      lastStmt_,
      var,
      getIndicesBounds(v->indices()));

  // If there were loads in the indices, this load depends on them, and merge
  // them in.
  if (!indicesScope->accesses_.empty()) {
    for (auto& access : indicesScope->accesses_) {
      load->addDependency(access);
      access->addDependent(load);
    }
    mergeScope(indicesScope, indicesScope->parent, false);
  }

  currentScope_ = indicesScope->parent;

  stmtToAccess_.emplace(lastStmt_, load);
  exprToAccess_.emplace(v, load);

  // This is a read, and does not close any accesses - but we need to establish
  // dependencies on accesses in the same scope.
  // Intentionally using operator[], we want it to be created if it does not
  // exist.
  auto& writeHistory = currentScope_->openWrites_[var];
  updateWriteHistory(writeHistory, load, load->id());
  currentScope_->accesses_.push_back(load);
}
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 596-629
```cpp
// This check determines if two accesses within a loop are "safe" from loop-self
// dependence. This function does not consider overlap in bound range, but
// rather the stride of the bound relative to the loop variable. This is the
// section of the code which considers iteration order, if allowed.
static bool executionSafetyCheck(
    const std::shared_ptr<AccessInfo>& info,
    const std::shared_ptr<AccessInfo>& other,
    const std::vector<ExprPtr>& aStrides,
    const std::vector<ExprPtr>& oStrides,
    bool parallelized) {
  if (aStrides.empty() || oStrides.empty()) {
    return false;
  }
  TORCH_INTERNAL_ASSERT(
      info->bounds().size() == other->bounds().size(),
      buildErrorMessage(
          "Dimension mismatch for two accesses in mem dep checker in the fuser."));
  for (size_t b = 0; b < info->bounds().size(); ++b) {
    ExprPtr aIndexStride = aStrides[b];
    ExprPtr oIndexStride = oStrides[b];
    // can't be safe on this index if we can't determine stride.
    if (!aIndexStride->isConstant() || !oIndexStride->isConstant()) {
      continue;
    }

    ExprPtr minStride =
        IRSimplifier::simplify(alloc<Min>(aIndexStride, oIndexStride, true));
    ExprPtr maxStride =
        IRSimplifier::simplify(alloc<Max>(aIndexStride, oIndexStride, true));

    // If the first access has no stride don't apply safety).
    if (immediateEquals(minStride, 0)) {
      continue;
    }
```
- **EN**: This chunk defines `executionSafetyCheck`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `executionSafetyCheck`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 631-665
```cpp
    ExprPtr modCheck = IRSimplifier::simplify(alloc<Mod>(maxStride, minStride));

    // if the strides can't have easily inferable distinct offsets, they're not
    // safe.
    if (!immediateEquals(modCheck, 0)) {
      continue;
    }

    // If the loop has a defined execution order (ie. sequential for) then
    // the order of execution can provide safety from overlaps.
    // Specifically if the difference in first access position for any
    // axis is the same sign as the common stride, then they will not
    // overlap.

    ExprPtr startDiff = IRSimplifier::simplify(
        alloc<Sub>(info->bounds()[b].start, other->bounds()[b].start));

    bool diffNegative = immediateIsNegative(startDiff);
    bool strideNegative = immediateIsNegative(minStride);

    // Invert the startDiff so mod works.
    if (diffNegative != strideNegative) {
      startDiff =
          IRSimplifier::simplify(alloc<Sub>(immLike(startDiff, 0), startDiff));
    }

    // If both accesses have the same stride, and the difference in start
    // element is smaller than this stride then the entire range is distinct.
    if (exprEquals(minStride, maxStride)) {
      ExprPtr check1 = IRSimplifier::simplify(
          alloc<CompareSelect>(startDiff, minStride, kLT));
      if (check1->isConstant() && immediateEquals(check1, 1)) {
        return true;
      }
    }
```
- **EN**: This chunk continues `executionSafetyCheck` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `executionSafetyCheck`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 667-698
```cpp
    startDiff = IRSimplifier::simplify(alloc<Mod>(startDiff, minStride));

    CompareSelectOperation op = strideNegative ? kLT : kGT;

    ExprPtr check = IRSimplifier::simplify(
        alloc<CompareSelect>(startDiff, immLike(startDiff, 0), op));

    // If the start difference modulo the minimum stride is offset from that
    // stride, then the ranges have distinct strides.
    if (check->isConstant() && immediateEquals<int>(check, 1)) {
      return true;
    }

    // If we can consider execution order and the difference in offset is
    // opposite signed to the stride then the read occurs in the past and we can
    // infer safety.
    if (!parallelized && diffNegative == strideNegative &&
        immediateEquals(startDiff, 0)) {
      return true;
    }
  }

  return false;
}

void MemDependencyChecker::visit(const ForPtr& v) {
  VarPtr var = v->var();

  StmtPtr last = lastStmt_;
  lastStmt_ = v;

  v->var()->accept(this);
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 700-732
```cpp
  // Loads inside the For's start and stop expression are special.
  // They exist in the enclosing scope, but accesses within the loop body may
  // depend on them via usage of the loop variable.
  // The way we handle this is to create a new scope so we have an easily
  // accessible list of the accesses within the extents.
  auto extentsScope =
      std::make_shared<Scope>(currentScope_->block, currentScope_);
  currentScope_ = extentsScope;

  v->start()->accept(this);
  v->stop()->accept(this);

  currentScope_ = currentScope_->parent;

  auto newScope = std::make_shared<Scope>(v->body(), currentScope_);
  currentScope_ = newScope;

  v->body()->accept(this);

  lastStmt_ = last;

  // Ok now we need to determine whether accesses in the loop depend on
  // other loop iterations.
  //
  // This is the real challenge here, it depends on both the fully expanded
  // bounds and the symbolic bounds.

  // The indices must change monotonically to avoid intersection. This is
  // hard to determine, so here's our heuristic I hope it's conservative
  // enough.

  // the size of at least one dependent index must be >= the size of the
  // loop.
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。

### Lines 734-762
```cpp
  // First step is to infer the stride relative to each dimension of each
  // access, which we do via substituting the loop var with (var+1) into the
  // indices expr.

  std::vector<std::vector<ExprPtr>> loopStrides;
  loopStrides.resize(currentScope_->accesses_.size());

  for (size_t a = 0; a < currentScope_->accesses_.size(); ++a) {
    auto& info = currentScope_->accesses_[a];

    std::vector<ExprPtr> indices = info->getIndices();

    std::vector<ExprPtr>& loopIndicesStride = loopStrides[a];
    loopIndicesStride.resize(indices.size());

    // index expr must depend on the loop var in some way to have a stride.
    for (const auto i : c10::irange(indices.size())) {
      VarFinder vf;
      if (vf.find(indices[i]).count(var) == 0) {
        loopIndicesStride[i] = immLike(indices[i], 0);
      } else {
        // If we've previously swapped the start and end of this bound, we
        // should apply the substitution to the reverse of the bounds.
        if (info->bounds()[i].swapped) {
          info->bounds()[i].end = IRSimplifier::simplify(
              SubstituteInClone(info->bounds()[i].end, {{var, v->start()}}));
          info->bounds()[i].start = IRSimplifier::simplify(SubstituteInClone(
              info->bounds()[i].start,
              {{var, alloc<Sub>(v->stop(), immLike(v->stop(), 1))}}));
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 764-793
```cpp
        } else {
          info->bounds()[i].start = IRSimplifier::simplify(
              SubstituteInClone(info->bounds()[i].start, {{var, v->start()}}));
          info->bounds()[i].end = IRSimplifier::simplify(SubstituteInClone(
              info->bounds()[i].end,
              {{var, alloc<Sub>(v->stop(), immLike(v->stop(), 1))}}));
        }

        ExprPtr zeroStep = indices[i];
        ExprPtr oneStep = SubstituteInClone(
            indices[i], {{var, alloc<Add>(var, immLike(var, 1))}});
        loopIndicesStride[i] =
            IRSimplifier::simplify(alloc<Sub>(oneStep, zeroStep));

        // If the start < end then swap the order of the bound.
        ExprPtr diff = IRSimplifier::simplify(
            alloc<Sub>(info->bounds()[i].end, info->bounds()[i].start));
        if (diff->isConstant() && immediateIsNegative(diff)) {
          info->bounds()[i].swap();
        }

        // If this access uses the loop var, it depends on loads used to compute
        // the loop var.
        for (auto& extentLoad : extentsScope->accesses_) {
          info->addDependency(extentLoad);
          extentLoad->addDependent(info);
        }
      }
    }
  }
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 795-824
```cpp
  // Now we need to update the bounds in openWrites since that is what we use to
  // merge.
  for (auto& openWritePair : currentScope_->openWrites_) {
    for (auto& pair : openWritePair.second) {
      IndexBounds& bounds = pair.first;

      // The bounds may not contain the loop var, but in that case Substitute
      // does nothing.
      for (auto& bound : bounds) {
        bound.start = IRSimplifier::simplify(
            SubstituteInClone(bound.start, {{var, v->start()}}));
        bound.end = IRSimplifier::simplify(SubstituteInClone(
            bound.end, {{var, alloc<Sub>(v->stop(), immLike(v->stop(), 1))}}));

        // If the start < end then swap the order of the bound.
        ExprPtr diff =
            IRSimplifier::simplify(alloc<Sub>(bound.end, bound.start));
        if (diff->isConstant() && immediateIsNegative(diff)) {
          bound.swap();
        }
      }
    }
  }

  // TODO this isn't a scalable way to determine parallelism.
  bool parallelized = v->loop_options().is_gpu_block_index() ||
      v->loop_options().is_gpu_thread_index();

  // Store buffers allocated at this scope.
  std::unordered_set<VarPtr> local_intermediates;
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 826-858
```cpp
  // Scanning from the top of the loop, we look for accesses which may depend
  // on a previous or parallel loop iteration.
  for (size_t a = 0; a < currentScope_->accesses_.size(); ++a) {
    auto& info = currentScope_->accesses_[a];
    if (info->type() == AccessType::Alloc) {
      local_intermediates.insert(info->var());
      continue;
    }

    if (!info->isRead()) {
      continue;
    }

    // Vars that don't carry outside this scope can't have loop self dependence.
    if (local_intermediates.count(info->var())) {
      continue;
    }

    // Copy the bounds so we can keep track of open bounds internally without
    // affecting the merge into the enclosing scope. The open portion of the
    // bounds may be cut into multiple independent slices.
    std::vector<IndexBounds> openBounds({info->bounds()});

    // Scan from the bottom of the loop.
    for (size_t j = currentScope_->accesses_.size() - 1; j > a; --j) {
      std::shared_ptr<AccessInfo> other = currentScope_->accesses_[j];
      if (!other->isWrite()) {
        continue;
      }

      if (info->var() != other->var()) {
        continue;
      }
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 860-892
```cpp
      if (info->hasDependency(other)) {
        continue;
      }

      // Whether or not the accesses within the loop are dependent on other
      // iterations depends whether the loop could be parallelized, the
      // difference in their strides and their start offset.
      bool iterationsDistinct = executionSafetyCheck(
          info,
          other,
          loopStrides[a],
          loopStrides[j],
          !allowExecutionOrderAnalysis_ || parallelized);

      if (iterationsDistinct) {
        continue;
      }

      std::vector<IndexBounds> newBoundSlices;
      for (auto& b : openBounds) {
        OverlapKind overlap = overlaps(b, other->bounds());
        if (overlap == OverlapKind::NoOverlap) {
          newBoundSlices.push_back(b);
          continue;
        }

        // It's dependent, link it to other.
        info->addDependency(other);
        other->addDependent(info);

        if (overlap == OverlapKind::Contains) {
          continue;
        }
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 894-925
```cpp
        // Otherwise update openBounds.
        auto slices = subtractIndicesBounds(b, other->bounds(), overlap);
        std::move(
            slices.begin(), slices.end(), std::back_inserter(newBoundSlices));
      }

      if (newBoundSlices.empty()) {
        break;
      }
      openBounds.swap(newBoundSlices);
    }
  }

  std::vector<std::shared_ptr<AccessInfo>> mergedAccesses;
  mergedAccesses.reserve(
      extentsScope->accesses_.size() + currentScope_->accesses_.size());
  std::copy(
      extentsScope->accesses_.begin(),
      extentsScope->accesses_.end(),
      std::back_inserter(mergedAccesses));
  std::copy(
      currentScope_->accesses_.begin(),
      currentScope_->accesses_.end(),
      std::back_inserter(mergedAccesses));
  scopeToAccesses_.emplace(v, mergedAccesses);

  // it's a little faster to merge without closing, and since no writes can
  // occur within the start and stop exprs we'll do that.
  mergeScope(extentsScope, extentsScope->parent, false);
  mergeScope(currentScope_, currentScope_->parent, true);
  currentScope_ = currentScope_->parent;
}
```
- **EN**: This chunk continues `visit` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `visit`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 927-959
```cpp
void MemDependencyChecker::visit(const CondPtr& v) {
  StmtPtr last = lastStmt_;
  lastStmt_ = v;

  auto enclosingScope =
      std::make_shared<Scope>(currentScope_->block, currentScope_);

  // condition is in enclosing scope.
  v->condition()->accept(this);

  BlockPtr true_stmt = v->true_stmt();
  BlockPtr false_stmt = v->false_stmt();

  // Create scopes so the Block visitor doesn't create and merge a new scope.
  auto trueScope = std::make_shared<Scope>(true_stmt, enclosingScope);
  auto falseScope = std::make_shared<Scope>(false_stmt, enclosingScope);

  if (true_stmt) {
    currentScope_ = trueScope;
    true_stmt->accept(this);
  }

  if (false_stmt) {
    currentScope_ = falseScope;
    false_stmt->accept(this);
  }

  // TODO(nickg): this logic isn't quite correct, if a write's Bound range is
  // present in both the true and false branches then we can close overlapping
  // accesses in the enclosing scope. Without that analysis future accesses
  // may be dependent on a write of a common range in all three of the
  // enclosing, true and false scope. This is a false positive so not too bad
  // in the short term, I think.
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 961-993
```cpp
  // Merge both true and false branches into the parent, but don't close any
  // accesses.
  mergeScope(trueScope, enclosingScope, false);
  mergeScope(falseScope, enclosingScope, false);

  // Merge the enclosing scope into it's parent.
  mergeScope(enclosingScope, enclosingScope->parent, false);

  currentScope_ = enclosingScope;
  scopeToAccesses_.emplace(v, enclosingScope->accesses_);

  currentScope_ = enclosingScope->parent;
  lastStmt_ = last;
}

void MemDependencyChecker::visit(const IfThenElsePtr& v) {
  // condition is in enclosing scope.
  v->condition()->accept(this);

  ExprPtr true_value = v->true_value();
  ExprPtr false_value = v->false_value();

  auto enclosingScope = currentScope_;

  // Create scopes to hold downstream Loads. It's safe to put nullptr for the
  // Scope's Block as it is only used by Stmts, not Exprs.
  auto trueScope = std::make_shared<Scope>(nullptr, enclosingScope);
  auto falseScope = std::make_shared<Scope>(nullptr, enclosingScope);

  if (true_value) {
    currentScope_ = trueScope;
    true_value->accept(this);
  }
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 995-1025
```cpp
  if (false_value) {
    currentScope_ = falseScope;
    false_value->accept(this);
  }

  // This doesn't have the same issue as Cond where there could be false
  // positives from the enclosing scope since there are no Exprs which are
  // writes.

  // Merge both true and false branches into the parent, but don't close any
  // accesses.
  mergeScope(trueScope, enclosingScope, false);
  mergeScope(falseScope, enclosingScope, false);

  currentScope_ = enclosingScope;
}

void MemDependencyChecker::visit(const CompareSelectPtr& v) {
  // condition is in enclosing scope.
  v->lhs()->accept(this);
  v->rhs()->accept(this);

  ExprPtr true_value = v->ret_val1();
  ExprPtr false_value = v->ret_val2();

  auto enclosingScope = currentScope_;

  // Create scopes to hold downstream Loads. It's safe to put nullptr for the
  // Scope's Block as it is only used by Stmts, not Exprs.
  auto trueScope = std::make_shared<Scope>(nullptr, enclosingScope);
  auto falseScope = std::make_shared<Scope>(nullptr, enclosingScope);
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1027-1062
```cpp
  if (true_value) {
    currentScope_ = trueScope;
    true_value->accept(this);
  }

  if (false_value) {
    currentScope_ = falseScope;
    false_value->accept(this);
  }

  // This doesn't have the same issue as Cond where there could be false
  // positives from the enclosing scope since there are no Exprs which are
  // writes.

  // Merge both true and false branches into the parent, but don't close any
  // accesses.
  mergeScope(trueScope, enclosingScope, false);
  mergeScope(falseScope, enclosingScope, false);

  currentScope_ = enclosingScope;
}

// Inserts accesses for a map of buffers (ie. for inputs and outputs).
void MemDependencyChecker::insertBuffers(
    std::unordered_map<BufPtr, std::shared_ptr<AccessInfo>>& bufs,
    AccessType type) {
  for (auto& pair : bufs) {
    const BufPtr& b = pair.first;
    VarPtr var = b->base_handle();
    IndexBounds bounds;
    for (const auto& d : b->dims()) {
      bounds.emplace_back(
          immLike(d, 0), IRSimplifier::simplify(alloc<Sub>(d, immLike(d, 1))));
    }
    auto info =
        std::make_shared<AccessInfo>(nextAccess_++, type, nullptr, var, bounds);
```
- **EN**: This chunk defines `insertBuffers`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `insertBuffers`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1064-1095
```cpp
    bufs[b] = info;

    auto& history = currentScope_->openWrites_[var];
    updateWriteHistory(history, info, info->id());
    currentScope_->accesses_.push_back(info);
  }
}

void MemDependencyChecker::visit(const BlockPtr& v) {
  auto prev_scope = currentScope_;

  // handle kernel inputs.
  if (prev_scope->block == nullptr) {
    insertBuffers(inputs_, AccessType::Input);
  }

  if (currentScope_->block != v) {
    currentScope_ = std::make_shared<Scope>((BlockPtr)v, prev_scope);
  }

  for (const auto& s : *v) {
    s->accept(this);
  }

  for (const auto& v : currentScope_->localVars) {
    knownVarBounds_.erase(v);
  }
  for (auto& pair : currentScope_->shadowedVarBounds) {
    knownVarBounds_[pair.first] = pair.second;
  }

  scopeToAccesses_.emplace(v, currentScope_->accesses_);
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1097-1129
```cpp
  if (currentScope_ != prev_scope) {
    mergeScope(currentScope_, prev_scope, true);
    currentScope_ = prev_scope;
  }

  // handle kernel outputs.
  if (prev_scope->block == nullptr) {
    insertBuffers(outputs_, AccessType::Output);
  }
}

void MemDependencyChecker::visit(const LetPtr& v) {
  StmtPtr last = lastStmt_;
  lastStmt_ = v;

  IRVisitor::visit(v);

  lastStmt_ = last;

  VarPtr var = v->var();
  if (knownVarBounds_.count(var) != 0) {
    currentScope_->shadowedVarBounds[var] = knownVarBounds_[var];
  }

  currentScope_->localVars.insert(var);
  knownVarBounds_[var] = {v->value(), v->value()};
}

// Don't support AtomicAdd yet, it's a bit more complex since it's both a read
// and a write. It's only inserted during Cuda codegen so this should be okay.
void MemDependencyChecker::visit(const AtomicAddPtr& v) {
  throw std::runtime_error("MemDependencyChecker AtomicAdd unimplemented");
}
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1131-1165
```cpp
void MemDependencyChecker::visit(const AllocatePtr& v) {
  StmtPtr last = lastStmt_;
  lastStmt_ = v;

  IRVisitor::visit(v);

  VarPtr var = v->buffer_var();
  IndexBounds bounds;
  // TODO: remove the "buf_flat_size" process below and extend the buf bound
  // check to support N-d indices access and 1-d index access.
  // "Allocate" stmt is based on "Buf" which supports N-d indices access and 1-d
  // index access. Currently the write bound check in memory analysis cannot
  // identify 1-d index access for N-d bufs. Thus we flatten N-d bufs here to
  // avoid failing the bound check. But this is not the correct approach and
  // should be fixed.
  ExprPtr flat_size = buf_flat_size(v->buf());
  flat_size =
      IRSimplifier::simplify(alloc<Sub>(flat_size, immLike(flat_size, 1)));
  bounds.emplace_back(immLike(flat_size, 0), flat_size);

  auto info = std::make_shared<AccessInfo>(
      nextAccess_++, AccessType::Alloc, nullptr, var, bounds);

  intermediates_[var] = info;

  auto& history = currentScope_->openWrites_[var];
  history.emplace_back(std::make_pair(info->bounds(), info));
  currentScope_->accesses_.push_back(info);

  lastStmt_ = last;
}

void MemDependencyChecker::visit(const FreePtr& v) {
  StmtPtr last = lastStmt_;
  lastStmt_ = v;
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。

### Lines 1167-1194
```cpp
  IRVisitor::visit(v);

  VarPtr var = v->buffer_var();
  auto it = intermediates_.find(var);
  TORCH_INTERNAL_ASSERT(
      it != intermediates_.end(),
      buildErrorMessage(
          "Expected to find '" + var->name_hint() +
          "' in intermediate vars in mem dep checker in the fuser."));

  IndexBounds bounds = it->second->bounds();
  auto info = std::make_shared<AccessInfo>(
      nextAccess_++, AccessType::Free, nullptr, var, bounds);

  auto& history = currentScope_->openWrites_[var];
  updateWriteHistory(history, info, info->id());
  currentScope_->accesses_.push_back(info);

  lastStmt_ = last;
}

void MemDependencyChecker::updateWriteHistory(
    std::list<BoundRelationship>& writeHistory,
    const std::shared_ptr<AccessInfo>& info,
    size_t latestAccessToClose,
    bool closeOverlapped,
    bool insert) {
  bool isWrite = info->isWrite();
```
- **EN**: This chunk defines `updateWriteHistory`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `updateWriteHistory`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1196-1230
```cpp
  for (auto it = writeHistory.begin(); it != writeHistory.end();) {
    auto& indexBounds = it->first;
    std::shared_ptr<AccessInfo> other = it->second;
    if (info->hasDependency(other)) {
      ++it;
      continue;
    }

    OverlapKind overlap = overlaps(indexBounds, info->bounds());

    if (overlap == OverlapKind::NoOverlap) {
      ++it;
      continue;
    }

    // Only writes can close open accesses.
    if (!isWrite) {
      info->addDependency(other);
      other->addDependent(info);
      ++it;
      continue;
    }

    // If we're not closing accesses we can stop here.
    if (!closeOverlapped || other->id() > latestAccessToClose) {
      ++it;
      continue;
    }

    if (overlap == OverlapKind::ContainedOrEqual) {
      // Total overlap is easy - the new access totally replaces the old.
      it = writeHistory.erase(it);
    } else {
      // The new write partially overlaps a previous write. We want to keep
      // both, but only track the uncovered part of the earlier write.
```
- **EN**: This chunk continues `updateWriteHistory` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `updateWriteHistory`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1232-1265
```cpp
      // Determine the slices of the earlier bound not covered by info.
      auto newBounds =
          subtractIndicesBounds(indexBounds, info->bounds(), overlap);

      // Erase the old slice.
      it = writeHistory.erase(it);

      // Add all new slices.
      for (auto& b : newBounds) {
        writeHistory.insert(it, std::make_pair(b, other));
      }
      // No need to increment the iterator since it has been updated after
      // `erase` above.
    }
  }

  if (insert && isWrite) {
    writeHistory.emplace_back(info->bounds(), info);
  }
}

void MemDependencyChecker::mergeScope(
    const std::shared_ptr<Scope>& child,
    const std::shared_ptr<Scope>& parent,
    bool closeOverlapped) {
  if (child->accesses_.empty()) {
    return;
  }

  // Update dependencies, but don't add new open writes yet.
  for (auto& info : child->accesses_) {
    // Intentionally using operator[], we want it to be created if it does not
    // exist.
    auto& writeHistory = parent->openWrites_[info->var()];
```
- **EN**: This chunk defines `mergeScope`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mergeScope`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1267-1295
```cpp
    size_t latestAccessToClose = child->accesses_.front()->id();
    updateWriteHistory(
        writeHistory, info, latestAccessToClose, closeOverlapped, false);
  }

  // Copy open writes up.
  for (auto& pair : child->openWrites_) {
    VarPtr var = pair.first;

    // Intentionally using operator[], we want it to be created if it does not
    // exist.
    auto& writeHistory = parent->openWrites_[var];

    for (auto& rel : pair.second) {
      writeHistory.push_back(rel);
    }
  }

  // the parent scope is responsible for holding all accesses now.
  parent->accesses_.insert(
      parent->accesses_.end(),
      std::make_move_iterator(child->accesses_.begin()),
      std::make_move_iterator(child->accesses_.end()));
}

// A visitor which applies known Bounds to symbolic expressions.
class VarBoundBinder : public IRVisitor {
 public:
  VarBoundBinder(const VarBoundMap& vars) : vars_(vars) {}
```
- **EN**: It introduces or extends VarBoundBinder, which define the primary data structures or interfaces for this portion of the file. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 它引入或扩展了 VarBoundBinder，这些类型定义了本段涉及的主要数据结构或接口。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 1297-1331
```cpp
  Bound getBounds(const ExprPtr& e) {
    min_ = e;
    max_ = e;
    e->accept(this);
    min_ = IRSimplifier::simplify(min_);
    max_ = IRSimplifier::simplify(max_);
    return {min_, max_};
  }

 private:
  void visit(const VarPtr& v) override {
    auto it = vars_.find(v);
    if (it == vars_.end()) {
      return;
    }

    min_ = SubstituteInClone(min_, {{v, it->second.start}});
    max_ = SubstituteInClone(max_, {{v, it->second.end}});
  }

  ExprPtr min_{nullptr};
  ExprPtr max_{nullptr};
  const VarBoundMap& vars_;
};

std::vector<Bound> MemDependencyChecker::getIndicesBounds(
    const std::vector<ExprPtr>& indices) {
  std::vector<Bound> bounds;
  bounds.reserve(indices.size());
  VarBoundBinder binder(knownVarBounds_);
  for (const auto& s : indices) {
    bounds.push_back(binder.getBounds(s));
  }
  return bounds;
}
```
- **EN**: This chunk defines `binder`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `binder`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1333-1333
```cpp
} // namespace torch::jit::tensorexpr::analysis
```
- **EN**: This chunk continues `binder` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `binder`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **AccessToString**
  - EN: `AccessToString` is a central symbol declared or implemented in this file.
  - CN: `AccessToString` 是本文件声明或实现的核心符号。
- **getDependencyChain**
  - EN: `getDependencyChain` is a central symbol declared or implemented in this file.
  - CN: `getDependencyChain` 是本文件声明或实现的核心符号。
- **IR simplification**
  - EN: Rewrites expressions and statements into simpler but equivalent forms.
  - CN: 把表达式和语句重写为更简单但等价的形式。
- **Static analysis**
  - EN: Infers structural properties that later optimizations depend on.
  - CN: 推断后续优化所依赖的结构属性。
- **Bounds inference**
  - EN: Tracks valid index ranges so transformations remain safe.
  - CN: 跟踪合法索引范围，以保证变换安全。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/mem_dependency_checker.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `c10/util/irange.h`
- **Standard library / 标准库**: `fstream`, `iostream`, `utility`
- **Primary symbols in this file / 本文件核心符号**: `AccessToString`, `getDependencyChain`, `getDependentsChain`, `getIndices`, `addDependency`, `addDependent`, `hasDependency`, `getDirectDependencies`
