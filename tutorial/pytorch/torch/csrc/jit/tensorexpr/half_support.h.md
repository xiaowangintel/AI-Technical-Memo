# half_support.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/half_support.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Lowers Tensor Expression IR into backend-specific code generation paths.
- **Purpose (CN)**: 把 Tensor Expression IR 降级到后端特定的代码生成路径。
## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
#pragma once

#include <torch/csrc/jit/tensorexpr/codegen.h>
#include <torch/csrc/jit/tensorexpr/ir.h>
#include <torch/csrc/jit/tensorexpr/ir_visitor.h>
#include <torch/csrc/jit/tensorexpr/tensor.h>

namespace torch::jit::tensorexpr {

// Walk the Statement looking for Half size loads/stores.
class HalfChecker : public IRVisitor {
 public:
  HalfChecker(const std::vector<CodeGen::BufferArg>& args) {
    for (const auto& BA : args) {
      hasHalf_ |= BA.dtype().scalar_type() == ScalarType::Half;
    }
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/codegen.h, torch/csrc/jit/tensorexpr/ir.h, torch/csrc/jit/tensorexpr/ir_visitor.h, and 1 more. The preprocessor guard keeps declarations single-instanced when the header is included transitively. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem. It introduces or extends HalfChecker, which define the primary data structures or interfaces for this portion of the file. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/codegen.h、torch/csrc/jit/tensorexpr/ir.h、torch/csrc/jit/tensorexpr/ir_visitor.h 等共 4 项。 预处理器保护用于避免头文件在传递包含时被重复展开。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。 它引入或扩展了 HalfChecker，这些类型定义了本段涉及的主要数据结构或接口。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 17-31
```cpp
  }

  bool hasHalf() const {
    return hasHalf_;
  }

  bool hasBFloat16() const {
    return hasBFloat16_;
  }

  void visit(const LoadPtr& v) override {
    hasHalf_ |= v->dtype().scalar_type() == ScalarType::Half;
    hasBFloat16_ |= v->dtype().scalar_type() == ScalarType::BFloat16;
    IRVisitor::visit(v);
  }
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 33-45
```cpp
  void visit(const StorePtr& v) override {
    hasHalf_ |= v->buf()->dtype().scalar_type() == ScalarType::Half;
    hasBFloat16_ |= v->buf()->dtype().scalar_type() == ScalarType::BFloat16;
    IRVisitor::visit(v);
  }

  void visit(const HalfImmPtr& v) override {
    hasHalf_ = true;
  }

  void visit(const BFloat16ImmPtr& v) override {
    hasBFloat16_ = true;
  }
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。

### Lines 47-56
```cpp
  void visit(const CastPtr& v) override {
    hasHalf_ |= v->dtype().scalar_type() == ScalarType::Half;
    hasBFloat16_ |= v->dtype().scalar_type() == ScalarType::BFloat16;
    IRVisitor::visit(v);
  }

 private:
  bool hasHalf_{false};
  bool hasBFloat16_{false};
};
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。

### Lines 58-70
```cpp
class HalfRewriter : public IRMutator {
  ExprPtr mutate(const LoadPtr& v) override {
    ExprPtr child = IRMutator::mutate(v);
    if (!isHalf(child)) {
      return child;
    }

    ExprPtr ret = alloc<Cast>(
        child->dtype().cloneWithScalarType(ScalarType::Float), child);

    inserted_half_casts_.insert(ret);
    return ret;
  }
```
- **EN**: It introduces or extends HalfRewriter, which define the primary data structures or interfaces for this portion of the file. This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 HalfRewriter，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 72-82
```cpp
  StmtPtr mutate(const StorePtr& v) override {
    // Since mutation changes the `value()` expression in-place, we need to
    // get the dtype of the `value()` before that is mutated.
    auto newType = v->value()->dtype();
    ExprPtr new_val = v->value()->accept_mutator(this);
    auto bufType = v->buf()->dtype();

    if (isHalf(newType.scalar_type())) {
      new_val = alloc<Cast>(newType, new_val);
      inserted_half_casts_.insert(new_val);
    }
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 84-97
```cpp
    // The scalar_type of value is not Half while the buf is Half
    if (!isHalf(newType.scalar_type()) && isHalf(bufType.scalar_type())) {
      new_val = alloc<Cast>(
          newType.cloneWithScalarType(bufType.scalar_type()), new_val);
      inserted_half_casts_.insert(new_val);
    }

    v->set_value(new_val);
    return v;
  }

  ExprPtr mutate(const HalfImmPtr& v) override {
    return alloc<Cast>(kFloat, v);
  }
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 99-113
```cpp
  ExprPtr mutate(const BFloat16ImmPtr& v) override {
    return alloc<Cast>(kFloat, v);
  }

  ExprPtr mutate(const CastPtr& v) override {
    ExprPtr child = v->src_value()->accept_mutator(this);

    // just don't allow half casts we didn't insert.
    if (isHalf(v)) {
      if (inserted_half_casts_.count(v) < 1) {
        v->set_src_value(child);
        v->set_dtype(v->dtype().cloneWithScalarType(c10::kFloat));
        return v;
      }
    }
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 115-127
```cpp
    // Remove Half(Float()) and friends.
    CastPtr cast_child = to<Cast>(child);
    if (cast_child) {
      auto cast_to_double = v->dtype().scalar_type() == ScalarType::Double;
      auto from_half = isHalf(cast_child->src_value());
      // Cannot simplify the double(float(half)) to double(half) as NNC does
      // not support cast BF16 to double directly.
      auto not_cast_half_to_doulbe = !(cast_to_double && from_half);
      if (v->dtype().is_floating_point() &&
          cast_child->dtype().is_floating_point() && not_cast_half_to_doulbe) {
        return alloc<Cast>(v->dtype(), cast_child->src_value());
      }
    }
```
- **EN**: This chunk continues `mutate` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `mutate`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 129-142
```cpp
    if (child == v->src_value()) {
      return v;
    }

    return alloc<Cast>(v->dtype(), child);
  }

  StmtPtr mutate(const LetPtr& v) override {
    if (isHalf(v->var()->dtype().scalar_type())) {
      VarPtr load_new_var = alloc<Var>(v->var()->name_hint(), kFloat);
      ExprPtr new_value = alloc<Cast>(
          v->var()->dtype().cloneWithScalarType(ScalarType::Float),
          v->value()->accept_mutator(this));
      var_map[v->var()] = load_new_var;
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 144-157
```cpp
      return alloc<Let>(load_new_var, new_value);
    }

    return IRMutator::mutate(v);
  }

  ExprPtr mutate(const VarPtr& v) override {
    auto it = var_map.find(v);
    if (it != var_map.end()) {
      return it->second;
    }

    return v;
  }
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 159-174
```cpp
  template <typename T>
  ExprPtr mutateArithmetic(T v) {
    IRMutator::mutate(v);
    if (isHalf(v)) {
      v->set_dtype(v->dtype().cloneWithScalarType(c10::kFloat));
    }
    return v;
  }

  ExprPtr mutate(const AddPtr& v) override {
    return mutateArithmetic(v);
  }
  ExprPtr mutate(const SubPtr& v) override {
    return mutateArithmetic(v);
  }
  ExprPtr mutate(const MulPtr& v) override {
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 175-190
```cpp
    return mutateArithmetic(v);
  }
  ExprPtr mutate(const DivPtr& v) override {
    return mutateArithmetic(v);
  }
  ExprPtr mutate(const MaxPtr& v) override {
    return mutateArithmetic(v);
  }
  ExprPtr mutate(const MinPtr& v) override {
    return mutateArithmetic(v);
  }
  ExprPtr mutate(const CompareSelectPtr& v) override {
    return mutateArithmetic(v);
  }
  ExprPtr mutate(const BroadcastPtr& v) override {
    return mutateArithmetic(v);
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 191-202
```cpp
  }
  ExprPtr mutate(const IfThenElsePtr& v) override {
    return mutateArithmetic(v);
  }
  ExprPtr mutate(const IntrinsicsPtr& v) override {
    return mutateArithmetic(v);
  }

 private:
  static bool isHalf(ScalarType st) {
    return st == ScalarType::Half || st == ScalarType::BFloat16;
  }
```
- **EN**: This chunk defines `isHalf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `isHalf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 204-212
```cpp
  static bool isHalf(const ExprPtr& v) {
    return isHalf(v->dtype().scalar_type());
  }

  std::unordered_set<ExprPtr> inserted_half_casts_;
  std::unordered_map<VarPtr, VarPtr> var_map;
};

} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk defines `isHalf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `isHalf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **HalfChecker**
  - EN: `HalfChecker` is a central symbol declared or implemented in this file.
  - CN: `HalfChecker` 是本文件声明或实现的核心符号。
- **HalfRewriter**
  - EN: `HalfRewriter` is a central symbol declared or implemented in this file.
  - CN: `HalfRewriter` 是本文件声明或实现的核心符号。
- **IR simplification**
  - EN: Rewrites expressions and statements into simpler but equivalent forms.
  - CN: 把表达式和语句重写为更简单但等价的形式。
- **Backend code generation**
  - EN: Translates IR into backend-specific source, kernels, or machine-level artifacts.
  - CN: 把 IR 翻译为后端特定的源码、内核或机器级产物。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/codegen.h`, `torch/csrc/jit/tensorexpr/ir.h`, `torch/csrc/jit/tensorexpr/ir_visitor.h`, `torch/csrc/jit/tensorexpr/tensor.h`
- **Primary symbols in this file / 本文件核心符号**: `HalfChecker`, `HalfRewriter`, `hasHalf`, `hasBFloat16`, `visit`, `mutate`, `mutateArithmetic`, `isHalf`
