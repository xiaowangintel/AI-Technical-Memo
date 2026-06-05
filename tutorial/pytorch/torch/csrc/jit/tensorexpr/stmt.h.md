# stmt.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/stmt.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines or manipulates expression nodes in the Tensor Expression subsystem.
- **Purpose (CN)**: 定义或操作 Tensor Expression 子系统中的表达式节点。
## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
#pragma once

#include <algorithm>
#include <list>
#include <string>
#include <unordered_set>
#include <utility>
#include <vector>

#include <torch/csrc/jit/tensorexpr/expr.h>

namespace torch::jit::tensorexpr {

// The common base between all statement node.
class TORCH_API Stmt : public std::enable_shared_from_this<Stmt> {
 public:
  Stmt() = default;
  virtual ~Stmt() = default;
  virtual void accept(IRVisitor* visitor) = 0;
  virtual StmtPtr accept_mutator(IRMutator* mutator) = 0;
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/expr.h; standard-library headers such as algorithm, list, string, and 3 more. The preprocessor guard keeps declarations single-instanced when the header is included transitively. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem. It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/expr.h；标准库头文件，如 algorithm、list、string 等共 6 项。 预处理器保护用于避免头文件在传递包含时被重复展开。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 22-41
```cpp
  StmtPtr get_parent() const {
    return parent_ ? parent_->getptr() : nullptr;
  }

  /*
   * Make a deep copy of the given statement.
   *
   * All statements and expressions used in children of the statement are
   * cloned. Note that the variables are not deep-copied since they are
   * immutable.
   */
  static StmtPtr clone(const StmtPtr& s);

 protected:
  static void set_parent(const StmtPtr& s, Stmt* new_parent) {
    s->parent_ = new_parent;
  }
  std::shared_ptr<Stmt> getptr() {
    return shared_from_this();
  }
```
- **EN**: This chunk defines `getptr`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getptr`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 43-64
```cpp
 private:
  Stmt* parent_ = nullptr;
};

template <class Op>
class StmtNode : public Stmt {
 public:
  using StmtNodeBase = StmtNode<Op>;
  void accept(IRVisitor* visitor) override {
    visitor->visit(static_to<Op>(getptr()));
  }
  StmtPtr accept_mutator(IRMutator* mutator) override;
  friend Op;

 private:
  StmtNode() = default;
};

template <class Op>
StmtPtr StmtNode<Op>::accept_mutator(IRMutator* mutator) {
  return mutator->mutate(static_to<Op>(getptr()));
}
```
- **EN**: It introduces or extends StmtNode, which define the primary data structures or interfaces for this portion of the file. This chunk defines `accept_mutator`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 StmtNode，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `accept_mutator`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 66-88
```cpp
// Concrete Stmt classes
class TORCH_API Block : public StmtNode<Block> {
 public:
  static BlockPtr make(const std::vector<StmtPtr>& stmts) {
    std::vector<StmtPtr> valid_stmts;
    for (auto& stmt : stmts) {
      if (!stmt) {
        continue;
      }
      valid_stmts.push_back(stmt);
    }
    if (valid_stmts.empty()) {
      return nullptr;
    }
    return alloc<Block>(valid_stmts);
  }

  size_t nstmts() const {
    return stmts_.size();
  }
  bool empty() const {
    return stmts_.empty();
  }
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `empty`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `empty`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 90-110
```cpp
  void prepend_stmt(const StmtPtr& s) {
    if (s->get_parent()) {
      throw malformed_input("Block prepend Stmt with existing parent", s);
    }

    stmts_.push_front(s);
    set_parent(s, this);
  }
  void append_stmt(const StmtPtr& s) {
    if (s->get_parent()) {
      throw malformed_input("Block append Stmt with existing parent", s);
    }

    stmts_.push_back(s);
    set_parent(s, this);
  }

  void insert_stmt_before(const StmtPtr& s, const StmtPtr& before) {
    if (s->get_parent()) {
      throw malformed_input("Block append Stmt with existing parent", s);
    }
```
- **EN**: This chunk defines `insert_stmt_before`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `insert_stmt_before`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 112-133
```cpp
    auto pos = std::find(stmts_.begin(), stmts_.end(), before);
    if (pos == stmts_.end()) {
      throw malformed_input(
          "Inserting after statement that is not in block", s);
    }

    stmts_.insert(pos, s);
    set_parent(s, this);
  }

  void insert_stmt_after(const StmtPtr& s, const StmtPtr& after) {
    if (s->get_parent()) {
      throw malformed_input("Block append Stmt with existing parent", s);
    }

    auto pos = std::find(stmts_.begin(), stmts_.end(), after);
    if (pos == stmts_.end()) {
      throw malformed_input(
          "Inserting after statement that is not in block", s);
    }

    ++pos;
```
- **EN**: This chunk defines `insert_stmt_after`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `insert_stmt_after`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 135-154
```cpp
    stmts_.insert(pos, s);
    set_parent(s, this);
  }

  bool replace_stmt(const StmtPtr& old_stmt, const StmtPtr& new_stmt) {
    if (new_stmt->get_parent()) {
      throw malformed_input(
          "Block replace Stmt with existing parent", new_stmt);
    }

    auto pos = std::find(stmts_.begin(), stmts_.end(), old_stmt);
    if (pos == stmts_.end()) {
      return false;
    }
    stmts_.insert(pos, new_stmt);
    stmts_.erase(pos);
    set_parent(old_stmt, nullptr);
    set_parent(new_stmt, this);
    return true;
  }
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 156-179
```cpp
  // Creates a new block by cloning `this` block and replacing the given
  // statement with a new statement. Note that `old_stmt` refers to a statement
  // in `this` block. If the `old_stmt` is not found, it will return `nullptr`.
  BlockPtr clone_and_replace(const StmtPtr& old_stmt, const StmtPtr& new_stmt) {
    if (new_stmt->get_parent()) {
      throw malformed_input(
          "Block replace Stmt with existing parent", new_stmt);
    }

    std::vector<StmtPtr> stmts(stmts_.begin(), stmts_.end());
    std::vector<StmtPtr> cloned_stmts(stmts.size());
    bool found = false;
    for (int i = 0; i < static_cast<int>(stmts.size()); ++i) {
      if (stmts[i] == old_stmt) {
        found = true;
        cloned_stmts[i] = new_stmt;
      } else {
        cloned_stmts[i] = Stmt::clone(stmts[i]);
      }
    }
    if (!found) {
      return nullptr;
    }
    return alloc<Block>(cloned_stmts);
```
- **EN**: This chunk defines `cloned_stmts`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `cloned_stmts`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 180-202
```cpp
  }

  bool remove_stmt(const StmtPtr& stmt) {
    auto pos = std::find(stmts_.begin(), stmts_.end(), stmt);
    if (pos == stmts_.end()) {
      return false;
    }

    set_parent(stmt, nullptr);
    stmts_.erase(pos);
    return true;
  }

  std::list<StmtPtr> stmts() const {
    return stmts_;
  }

  void clear() {
    for (const auto& s : stmts_) {
      set_parent(s, nullptr);
    }
    stmts_.clear();
  }
```
- **EN**: This chunk defines `clear`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `clear`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 204-226
```cpp
  void set_stmts(const std::vector<StmtPtr>& stmts) {
    clear();
    init(stmts);
  }

  explicit Block(const std::vector<StmtPtr>& stmts) {
    init(stmts);
  }

  typedef std::list<StmtPtr>::iterator iterator;
  typedef std::list<StmtPtr>::const_iterator const_iterator;

  iterator begin() {
    return stmts_.begin();
  }

  const_iterator begin() const {
    return stmts_.begin();
  }

  iterator end() {
    return stmts_.end();
  }
```
- **EN**: This chunk defines `end`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `end`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 228-246
```cpp
  const_iterator end() const {
    return stmts_.end();
  }

  StmtPtr front() {
    return stmts_.front();
  }

  StmtPtr front() const {
    return stmts_.front();
  }

  StmtPtr back() {
    return stmts_.back();
  }

  StmtPtr back() const {
    return stmts_.back();
  }
```
- **EN**: This chunk defines `back`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `back`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 248-265
```cpp
  void splice(Block::iterator it, const BlockPtr& other) {
    for (const StmtPtr& s : *other) {
      set_parent(s, this);
    }

    stmts_.splice(it, other->stmts_);
  }

  static BlockPtr getSharedParent(StmtPtr p1, StmtPtr p2) {
    std::unordered_set<BlockPtr> enclosing;

    StmtPtr p1_p = std::move(p1);
    while (p1_p) {
      if (BlockPtr b = to<Block>(p1_p)) {
        enclosing.insert(b);
      }
      p1_p = p1_p->get_parent();
    }
```
- **EN**: This chunk defines `getSharedParent`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `getSharedParent`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 267-289
```cpp
    StmtPtr p2_p = std::move(p2);
    while (p2_p) {
      if (BlockPtr b = to<Block>(p2_p)) {
        if (enclosing.count(b) != 0) {
          return b;
        }
      }
      p2_p = p2_p->get_parent();
    }

    return nullptr;
  }

  // returns the immediate child containing statement s.
  StmtPtr getEnclosedRoot(StmtPtr s) const {
    while (s && s->get_parent().get() != this) {
      s = s->get_parent();
    }
    return s;
  }

 private:
  std::list<StmtPtr> stmts_;
```
- **EN**: This chunk defines `getEnclosedRoot`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getEnclosedRoot`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 291-305
```cpp
  void init(const std::vector<StmtPtr>& stmts) {
    for (const StmtPtr& s : stmts) {
      if (!s) {
        continue;
      }
      if (!s->get_parent()) {
        // If we get here, it's a bug, but we cannot throw an error from a
        // constructor. But IR verifier would catch this.
        set_parent(s, this);
      }

      stmts_.push_back(s);
    }
  }
};
```
- **EN**: This chunk defines `init`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `init`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 307-328
```cpp
class TORCH_API Store : public StmtNode<Store> {
 public:
  VarPtr base_handle() const {
    return buf_->base_handle();
  }
  std::vector<ExprPtr> indices() const {
    return indices_;
  }
  ExprPtr flat_index() const {
    TORCH_CHECK(indices_.size() == 1, "Indices haven't been flattened.");
    return indices_[0];
  }
  ExprPtr value() const {
    return value_;
  }
  BufPtr buf() const {
    return buf_;
  }

  void set_buf(BufPtr buf) {
    buf_ = std::move(buf);
  }
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `set_buf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `set_buf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 330-349
```cpp
  void set_indices(std::vector<ExprPtr> indices) {
    indices_ = std::move(indices);
  }

  void set_value(ExprPtr value) {
    value_ = std::move(value);
  }

  static StorePtr make(
      const BufHandle& buf,
      const std::vector<ExprHandle>& indices,
      const ExprHandle& value);

  Store(BufPtr buf, std::vector<ExprPtr> indices, ExprPtr value);

 private:
  BufPtr buf_;
  std::vector<ExprPtr> indices_;
  ExprPtr value_;
};
```
- **EN**: This chunk defines `make`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段定义了 `make`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 351-370
```cpp
// Allocate a buffer of given shapes and dtypes and bind it with the given
// buffer var. The life span is at most through the current program, until it is
// explicitly freed. An unfreed memory is likely considered an error.
class TORCH_API Allocate : public StmtNode<Allocate> {
 public:
  static AllocatePtr make(const BufHandle& buf_handle) {
    return alloc<Allocate>(buf_handle.node());
  }

  VarPtr buffer_var() const {
    return buf_->base_handle();
  }

  Dtype dtype() const {
    return buf_->dtype();
  }

  const std::vector<ExprPtr> dims() const {
    return buf_->dims();
  }
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `dims`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `dims`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 372-395
```cpp
  BufPtr buf() const {
    return buf_;
  }

  void set_buf(BufPtr buf) {
    buf_ = std::move(buf);
  }

  explicit Allocate(BufPtr buf) : buf_(std::move(buf)) {}

 private:
  BufPtr buf_;
  // TODO: add memory types.
};

// PlacementAllocate is a variation of the Allocate operator in NNC IR. It does
// not allocate memory but reuse the memory of another buffer for the given
// buffer.
class TORCH_API PlacementAllocate : public StmtNode<PlacementAllocate> {
 public:
  static PlacementAllocatePtr make(
      const BufHandle& buf_handle,
      const BufHandle& buf_handle_to_reuse) {
    return alloc<PlacementAllocate>(
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `make`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `make`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 396-416
```cpp
        buf_handle.node(), buf_handle_to_reuse.node());
  }

  BufPtr buf() const {
    return buf_;
  }

  BufPtr buf_to_reuse() const {
    return buf_to_reuse_;
  }

  void set_buf(BufPtr buf) {
    buf_ = std::move(buf);
  }

  void set_buf_to_reuse(BufPtr buf) {
    buf_to_reuse_ = std::move(buf);
  }

  explicit PlacementAllocate(BufPtr buf, BufPtr buf_to_reuse)
      : buf_(std::move(buf)), buf_to_reuse_(std::move(buf_to_reuse)) {}
```
- **EN**: This chunk defines `PlacementAllocate`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `PlacementAllocate`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 418-440
```cpp
 private:
  BufPtr buf_;
  BufPtr buf_to_reuse_;
};

// Free the specific buffer. It is an error.
class TORCH_API Free : public StmtNode<Free> {
 public:
  static FreePtr make(const BufHandle& buf_handle) {
    return alloc<Free>(buf_handle.node());
  }

  VarPtr buffer_var() const {
    return buf_->base_handle();
  }

  BufPtr buf() const {
    return buf_;
  }

  void set_buf(BufPtr buf) {
    buf_ = std::move(buf);
  }
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `set_buf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `set_buf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 442-464
```cpp
  explicit Free(BufPtr buf) : buf_(std::move(buf)) {}

 private:
  BufPtr buf_;
};

class TORCH_API FreeExt : public StmtNode<FreeExt> {
 public:
  static FreeExtPtr make(const std::vector<BufHandle>& bufs);

  std::vector<BufPtr> bufs() const {
    return bufs_;
  }

  void set_bufs(std::vector<BufPtr> bufs) {
    bufs_ = std::move(bufs);
  }

  explicit FreeExt(std::vector<BufPtr> bufs) : bufs_(std::move(bufs)) {}

 private:
  std::vector<BufPtr> bufs_;
};
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `FreeExt`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `FreeExt`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 466-488
```cpp
class TORCH_API Let : public StmtNode<Let> {
 public:
  static LetPtr make(const VarHandle& var, const ExprHandle& val) {
    return alloc<Let>(var.node(), val.node());
  }

  Let(VarPtr var, ExprPtr val) : var_(std::move(var)), val_(std::move(val)) {}

  VarPtr var() const {
    return var_;
  }

  ExprPtr value() const {
    return val_;
  }

  void set_var(VarPtr var) {
    var_ = std::move(var);
  }

  void set_val(ExprPtr val) {
    val_ = std::move(val);
  }
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `set_val`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `set_val`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 490-510
```cpp
 private:
  VarPtr var_;
  ExprPtr val_;
};

class TORCH_API Cond : public StmtNode<Cond> {
 public:
  static CondPtr make(
      const ExprHandle& condition,
      const StmtPtr& true_stmt,
      const StmtPtr& false_stmt) {
    return alloc<Cond>(condition.node(), true_stmt, false_stmt);
  }

  ExprPtr condition() const {
    return condition_;
  }

  BlockPtr true_stmt() const {
    return true_stmt_;
  }
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `true_stmt`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `true_stmt`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 512-529
```cpp
  BlockPtr false_stmt() const {
    return false_stmt_;
  }

  void set_condition(ExprPtr condition) {
    condition_ = std::move(condition);
  }

  void set_true_stmt(StmtPtr true_stmt) {
    if (true_stmt) {
      BlockPtr b = to<Block>(true_stmt);
      if (!b) {
        b = alloc<Block>(std::vector<StmtPtr>({std::move(true_stmt)}));
      }
      true_stmt_ = b;
      set_parent(true_stmt_, this);
    }
  }
```
- **EN**: This chunk defines `set_true_stmt`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `set_true_stmt`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 531-552
```cpp
  void set_false_stmt(StmtPtr false_stmt) {
    if (false_stmt) {
      BlockPtr b = to<Block>(false_stmt);
      if (!b) {
        b = alloc<Block>(std::vector<StmtPtr>({std::move(false_stmt)}));
      }
      false_stmt_ = b;
      set_parent(false_stmt_, this);
    }
  }

  Cond(ExprPtr condition, StmtPtr true_stmt, StmtPtr false_stmt)
      : condition_(std::move(condition)) {
    set_true_stmt(std::move(true_stmt));
    set_false_stmt(std::move(false_stmt));
  }

  CondPtr cloneWithNewBodies(
      const StmtPtr& true_stmt,
      const StmtPtr& false_stmt) {
    return alloc<Cond>(condition_, true_stmt, false_stmt);
  }
```
- **EN**: This chunk defines `cloneWithNewBodies`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `cloneWithNewBodies`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 554-577
```cpp
  CondPtr cloneWithNewBody(const StmtPtr& true_stmt) {
    return alloc<Cond>(condition_, true_stmt, nullptr);
  }

 private:
  ExprPtr condition_;
  BlockPtr true_stmt_ = nullptr;
  BlockPtr false_stmt_ = nullptr;
};

class TORCH_API LoopOptions {
 public:
  enum {
    IDX_UNSET = -1,
    IDX_X = 0,
    IDX_Y = 1,
    IDX_Z = 2,
    IDX_W = 3,
    IDX_MAX = IDX_W,
  };
  // GPU Block Index
  bool is_gpu_block_index() const {
    return gpu_block_index_ != IDX_UNSET;
  }
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `is_gpu_block_index`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `is_gpu_block_index`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 579-601
```cpp
  int gpu_block_index() const {
    return gpu_block_index_;
  }

  std::string gpu_block_index_str() const {
    if (!is_gpu_block_index()) {
      throw malformed_input("Has no GPU block index");
    }

    // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
    static constexpr const char* kBlockIndexNames[] = {
        "blockIdx.x",
        "blockIdx.y",
        "blockIdx.z",
        "blockIdx.w",
    };

    if (gpu_block_index_ < IDX_X || gpu_block_index_ > IDX_MAX) {
      throw malformed_input("invalid GPU block index");
    }

    return kBlockIndexNames[gpu_block_index_];
  }
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 603-624
```cpp
  void set_gpu_block_index(int index) {
    if (index == IDX_UNSET) {
      gpu_block_index_ = IDX_UNSET;
    }

    if (is_gpu_thread_index()) {
      throw std::runtime_error("Cannot set both gpu block and thread index");
    }
    if (is_gpu_block_index() && gpu_block_index() != index) {
      throw std::runtime_error("Cannot set a previously set block index");
    }
    gpu_block_index_ = index;
  }

  // GPU Thread Index
  bool is_gpu_thread_index() const {
    return gpu_thread_index() != IDX_UNSET;
  }

  int gpu_thread_index() const {
    return gpu_thread_index_;
  }
```
- **EN**: This chunk defines `gpu_thread_index`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `gpu_thread_index`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 626-645
```cpp
  std::string gpu_thread_index_str() const {
    if (!is_gpu_thread_index()) {
      throw malformed_input("has no GPU thread index");
    }

    // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
    static constexpr const char* kThreadIndexNames[] = {
        "threadIdx.x", "threadIdx.y", "threadIdx.z", "threadIdx.w"};

    if (gpu_thread_index_ < IDX_X || gpu_thread_index_ > IDX_MAX) {
      throw malformed_input("invalid GPU thread index");
    }

    return kThreadIndexNames[gpu_thread_index_];
  }

  void set_gpu_thread_index(int index) {
    if (index == IDX_UNSET) {
      gpu_thread_index_ = IDX_UNSET;
    }
```
- **EN**: This chunk defines `set_gpu_thread_index`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `set_gpu_thread_index`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 647-662
```cpp
    if (is_gpu_block_index()) {
      throw std::runtime_error("Cannot set both gpu thread and block index");
    }
    if (is_gpu_thread_index() && gpu_thread_index() != index) {
      throw std::runtime_error("Cannot set a previously set thread index");
    }
    gpu_thread_index_ = index;
  }

  void set_parallel() {
    is_parallel_ = true;
  }

  bool is_parallel() const {
    return is_parallel_;
  }
```
- **EN**: This chunk defines `is_parallel`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `is_parallel`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 664-686
```cpp
  std::string ToString() const {
    if (is_gpu_block_index()) {
      return gpu_block_index_str();
    } else if (is_gpu_thread_index()) {
      return gpu_thread_index_str();
    } else if (is_parallel()) {
      return "parallel";
    }
    return "";
  }

  bool isDefault() const {
    return gpu_block_index_ == IDX_UNSET && gpu_thread_index_ == IDX_UNSET &&
        !is_parallel_;
  }

  void set_buffer_mapping(const std::unordered_map<std::string, BufPtr>& map) {
    map_input_to_tensor_bufs_ = map;
  }

  std::unordered_map<std::string, BufPtr> get_buffer_mapping() const {
    return map_input_to_tensor_bufs_;
  }
```
- **EN**: This chunk defines `set_buffer_mapping`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `set_buffer_mapping`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 688-711
```cpp
 private:
  int gpu_block_index_{IDX_UNSET};
  int gpu_thread_index_{IDX_UNSET};
  bool is_parallel_{false};
  std::unordered_map<std::string, BufPtr> map_input_to_tensor_bufs_;
};

class TORCH_API For : public StmtNode<For> {
 public:
  VarPtr var() const {
    return var_;
  }
  ExprPtr start() const {
    return start_;
  }
  ExprPtr stop() const {
    return stop_;
  }
  BlockPtr body() const {
    return body_;
  }
  static ForPtr make(
      const VarHandle& var,
      const ExprHandle& start,
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `body`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `body`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 712-733
```cpp
      const ExprHandle& stop,
      const StmtPtr& body) {
    if (!body) {
      return nullptr;
    }
    return alloc<For>(var.node(), start.node(), stop.node(), body);
  }
  static ForPtr make(
      const VarHandle& var,
      const ExprHandle& start,
      const ExprHandle& stop,
      const StmtPtr& body,
      const LoopOptions& loop_options) {
    if (!body) {
      return nullptr;
    }
    return alloc<For>(
        var.node(), start.node(), stop.node(), body, loop_options);
  }
  const LoopOptions loop_options() const {
    return loop_options_;
  }
```
- **EN**: This chunk defines `loop_options`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `loop_options`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 735-758
```cpp
  For(VarPtr var, ExprPtr start, ExprPtr stop, StmtPtr body)
      : var_(std::move(var)), start_(std::move(start)), stop_(std::move(stop)) {
    BlockPtr b = to<Block>(body);
    if (!b) {
      b = alloc<Block>(std::vector<StmtPtr>({std::move(body)}));
    }
    body_ = b;
    set_parent(body_, this);
  }

  For(VarPtr var,
      ExprPtr start,
      ExprPtr stop,
      StmtPtr body,
      LoopOptions loop_options)
      : var_(std::move(var)),
        start_(std::move(start)),
        stop_(std::move(stop)),
        loop_options_(std::move(loop_options)) {
    if (!var_) {
      throw malformed_input("invalid Var in For loop");
    } else if (!start_) {
      throw malformed_input("invalid Start in For loop");
    } else if (!stop_) {
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 759-778
```cpp
      throw malformed_input("invalid Stop in For loop");
    } else if (!body || body->get_parent()) {
      throw malformed_input("invalid Body in For loop");
    }

    BlockPtr b = to<Block>(body);
    if (!b) {
      b = alloc<Block>(std::vector<StmtPtr>({std::move(body)}));
    }
    body_ = b;
    set_parent(body_, this);
  }

  void set_gpu_block_index(int block_index) {
    loop_options_.set_gpu_block_index(block_index);
  }

  void set_gpu_thread_index(int thread_index) {
    loop_options_.set_gpu_thread_index(thread_index);
  }
```
- **EN**: This chunk defines `set_gpu_thread_index`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `set_gpu_thread_index`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 780-801
```cpp
  void set_parallel() {
    loop_options_.set_parallel();
  }

  bool is_parallel() const {
    return loop_options_.is_parallel();
  }

  void set_buffer_map(const std::unordered_map<std::string, BufPtr>& map) {
    loop_options_.set_buffer_mapping(map);
  }

  ForPtr cloneWithNewBody(const StmtPtr& body) const {
    return alloc<For>(var_, start_, stop_, body, loop_options_);
  }

  BlockPtr removeBody() {
    auto res = body_;
    set_parent(res, nullptr);
    body_ = nullptr;
    return res;
  }
```
- **EN**: This chunk defines `removeBody`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `removeBody`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 803-822
```cpp
  void set_body(StmtPtr body) {
    BlockPtr b = to<Block>(body);
    if (!b) {
      b = alloc<Block>(std::vector<StmtPtr>({std::move(body)}));
    }
    body_ = b;
    set_parent(body_, this);
  }

  void set_start(ExprPtr start) {
    start_ = std::move(start);
  }

  void set_stop(ExprPtr stop) {
    stop_ = std::move(stop);
  }

  void set_var(VarPtr var) {
    var_ = std::move(var);
  }
```
- **EN**: This chunk defines `set_var`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `set_var`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 824-845
```cpp
 private:
  VarPtr var_;
  ExprPtr start_;
  ExprPtr stop_;
  BlockPtr body_;
  LoopOptions loop_options_;
};

// A backend specific IR Node that implements atomic-add.
// This node could only shows up as an internal with GPU backends.
// TODO: move to this an internal IR.
// TODO: make IR nodes extensible.
class TORCH_API AtomicAdd : public StmtNode<AtomicAdd> {
 public:
  AtomicAdd(BufPtr buf, std::vector<ExprPtr> indices, ExprPtr value)
      : buf_(std::move(buf)),
        indices_(std::move(indices)),
        value_(std::move(value)) {}

  VarPtr base_handle() const {
    return buf_->base_handle();
  }
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `base_handle`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `base_handle`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 847-866
```cpp
  BufPtr buf() const {
    return buf_;
  }

  ExprPtr flat_index() const {
    TORCH_CHECK(indices_.size() == 1, "Indices haven't been flattened.");
    return indices_[0];
  }

  ExprPtr value() const {
    return value_;
  }

  const std::vector<ExprPtr>& indices() const {
    return indices_;
  }

  void set_buf(BufPtr buf) {
    buf_ = std::move(buf);
  }
```
- **EN**: This chunk defines `set_buf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `set_buf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 868-885
```cpp
  void set_indices(std::vector<ExprPtr> indices) {
    indices_ = std::move(indices);
  }

  void set_value(ExprPtr value) {
    value_ = std::move(value);
  }

 private:
  BufPtr buf_;
  std::vector<ExprPtr> indices_;
  ExprPtr value_;
};

class TORCH_API SyncThreads : public StmtNode<SyncThreads> {
 public:
  SyncThreads() = default;
};
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `set_value`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `set_value`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 887-910
```cpp
/*
 * ExternalCall statement represents a call to an external function that would
 * compute the contents of the output buffer. An ExternalCall statement consists
 * of:
 *   1) output buffer - the buffer that'll be initialized by the call
 *   2) external function name - a key from the NNC function registry to lookup
 *      the actual function to call
 *   3) buffer arguments - the input buffers used by the function
 *   4) non-buffer arguments - scalar arguments to pass to the function
 *
 * An example:
 *   A = nnc_conv2d(buf_args={Input, Weight, Bias}, args={1})
 * Here 'A' is the output buffer, "nnc_conv2d" is the function name, the buffer
 * arguments are 'Input', 'Weight', and 'Bias', and there is a single non-buffer
 * argument - 1.
 *
 * The semantics of the scalar arguments is defined solely by the implementation
 * of the external function.
 */
class TORCH_API ExternalCall : public StmtNode<ExternalCall> {
 public:
  static ExternalCallPtr make(
      BufHandle buf,
      const std::string& func_name,
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 911-932
```cpp
      const std::vector<BufHandle>& buf_args,
      const std::vector<ExprHandle>& args);

  BufPtr buf() const {
    return buf_;
  }

  std::string func_name() const {
    return func_name_;
  }

  std::vector<BufPtr> buf_args() const {
    return buf_args_;
  }

  std::vector<ExprPtr> args() const {
    return args_;
  }

  void set_buf(BufPtr buf) {
    buf_ = std::move(buf);
  }
```
- **EN**: This chunk defines `set_buf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `set_buf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 934-950
```cpp
  void set_buf_args(std::vector<BufPtr> buf_args) {
    buf_args_ = std::move(buf_args);
  }

  void set_args(std::vector<ExprPtr> args) {
    args_ = std::move(args);
  }

  ExternalCall(
      BufPtr buf,
      std::string func_name,
      std::vector<BufPtr> buf_args,
      std::vector<ExprPtr> args)
      : buf_(std::move(buf)),
        func_name_(std::move(func_name)),
        buf_args_(std::move(buf_args)),
        args_(std::move(args)) {}
```
- **EN**: This chunk defines `set_args`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段定义了 `set_args`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 952-973
```cpp
 private:
  BufPtr buf_;
  std::string func_name_;
  std::vector<BufPtr> buf_args_;
  std::vector<ExprPtr> args_;
};

class TORCH_API ExternalCallWithAlloc : public StmtNode<ExternalCallWithAlloc> {
 public:
  static ExternalCallWithAllocPtr make(
      const std::string& func_name,
      const std::vector<BufHandle>& buf_out_args,
      const std::vector<BufHandle>& buf_args,
      const std::vector<ExprHandle>& args);

  std::vector<BufPtr> buf_out_args() const {
    return buf_out_args_;
  }

  std::string func_name() const {
    return func_name_;
  }
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `func_name`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `func_name`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 975-993
```cpp
  std::vector<BufPtr> buf_args() const {
    return buf_args_;
  }

  std::vector<ExprPtr> args() const {
    return args_;
  }

  void set_buf_out_args(std::vector<BufPtr> buf_out_args) {
    buf_out_args_ = std::move(buf_out_args);
  }

  void set_buf_args(std::vector<BufPtr> buf_args) {
    buf_args_ = std::move(buf_args);
  }

  void set_args(std::vector<ExprPtr> args) {
    args_ = std::move(args);
  }
```
- **EN**: This chunk defines `set_args`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `set_args`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 995-1012
```cpp
  ExternalCallWithAlloc(
      std::string func_name,
      std::vector<BufPtr> buf_out_args,
      std::vector<BufPtr> buf_args,
      std::vector<ExprPtr> args)
      : func_name_(std::move(func_name)),
        buf_out_args_(std::move(buf_out_args)),
        buf_args_(std::move(buf_args)),
        args_(std::move(args)) {}

 private:
  std::string func_name_;
  std::vector<BufPtr> buf_out_args_;
  std::vector<BufPtr> buf_args_;
  std::vector<ExprPtr> args_;
};

} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk continues `set_args` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `set_args`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **TORCH_API**
  - EN: `TORCH_API` is a central symbol declared or implemented in this file.
  - CN: `TORCH_API` 是本文件声明或实现的核心符号。
- **StmtNode**
  - EN: `StmtNode` is a central symbol declared or implemented in this file.
  - CN: `StmtNode` 是本文件声明或实现的核心符号。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/expr.h`
- **Standard library / 标准库**: `algorithm`, `list`, `string`, `unordered_set`, `utility`, `vector`
- **Primary symbols in this file / 本文件核心符号**: `TORCH_API`, `StmtNode`, `get_parent`, `clone`, `set_parent`, `getptr`, `accept`, `accept_mutator`
