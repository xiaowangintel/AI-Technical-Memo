# cuda_codegen.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/cuda_codegen.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-27
```cpp
#include <torch/csrc/jit/tensorexpr/cuda_codegen.h>
#include <torch/csrc/jit/tensorexpr/half_support.h>

#include <ATen/cuda/CUDAGeneratorImpl.h>
#include <ATen/native/cuda/jit_utils.h>
#include <c10/cuda/CUDAFunctions.h>
#include <c10/util/irange.h>
#include <torch/csrc/jit/codegen/fuser/cuda/fused_kernel.h>
#include <torch/csrc/jit/codegen/fuser/cuda/resource_strings.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/tensorexpr/analysis.h>
#include <torch/csrc/jit/tensorexpr/cuda_random.h>
#include <torch/csrc/jit/tensorexpr/eval.h>
#include <torch/csrc/jit/tensorexpr/exceptions.h>
#include <torch/csrc/jit/tensorexpr/ir_simplifier.h>
#include <torch/csrc/jit/tensorexpr/registerizer.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/empty_strided_native.h>
#endif

#include <unordered_map>
#include <utility>

namespace torch::jit::tensorexpr {
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/cuda_codegen.h, torch/csrc/jit/tensorexpr/half_support.h, torch/csrc/jit/codegen/fuser/cuda/fused_kernel.h, and 8 more; ATen/c10 facilities such as ATen/cuda/CUDAGeneratorImpl.h, ATen/native/cuda/jit_utils.h, c10/cuda/CUDAFunctions.h, and 3 more; standard-library headers such as unordered_map, utility. The preprocessor guard keeps declarations single-instanced when the header is included transitively. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/cuda_codegen.h、torch/csrc/jit/tensorexpr/half_support.h、torch/csrc/jit/codegen/fuser/cuda/fused_kernel.h 等共 11 项；ATen/c10 基础设施，如 ATen/cuda/CUDAGeneratorImpl.h、ATen/native/cuda/jit_utils.h、c10/cuda/CUDAFunctions.h 等共 6 项；标准库头文件，如 unordered_map、utility。 预处理器保护用于避免头文件在传递包含时被重复展开。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。

### Lines 29-63
```cpp
// A RAII wrapper to manage a variable and name pair in the look-up table.
// TODO: move this to a more shared place.
class ScopedVarName {
 public:
  ScopedVarName(VarNameMap* mapping, const VarPtr& var, const std::string& name)
      : mapping_(mapping), var_(var) {
    auto iter = mapping->find(var);
    if (iter != mapping->end()) {
      throw std::runtime_error("Duplicate var entry: " + var->name_hint());
    }
    mapping->insert(std::make_pair(var, name));
  }

  ScopedVarName(
      UniqueNameManager* manager,
      const VarPtr& var,
      const std::string& name)
      : ScopedVarName(&manager->unique_name_mapping_, var, name) {}

  ScopedVarName(const ScopedVarName&) = delete;
  ScopedVarName& operator=(const ScopedVarName&) = delete;

  ~ScopedVarName() noexcept(false) {
    mapping_->erase(var_);
  }

 private:
  VarNameMap* mapping_ = nullptr;
  VarPtr var_ = nullptr;
};

static bool is_zero(const ExprPtr& expr) {
  auto v = intValue(expr);
  return v && *v == 0;
}
```
- **EN**: It introduces or extends ScopedVarName, which define the primary data structures or interfaces for this portion of the file. This chunk defines `is_zero`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 ScopedVarName，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `is_zero`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 65-95
```cpp
static const at::cuda::NVRTC& nvrtc() {
  return at::globalContext().getNVRTC();
}

std::string CudaPrinter::dtypeToCppString(const Dtype& dtype) {
  switch (dtype.scalar_type()) {
    case ScalarType::Bool:
      return "bool";
    case ScalarType::Half:
      return "half";
    case ScalarType::BFloat16:
      return fuser::cuda::bfloat16_type_string;
    case ScalarType::Char:
      return "char";
    case ScalarType::Byte:
      return "unsigned char";
    case ScalarType::Short:
      return "short";
    case ScalarType::Long:
      return "long long";
    default:
      return dtype.ToCppString();
  }
}

void CudaAnalysis::visit(const FreePtr& v) {
  if (thread_local_bufs_.count(v->buffer_var()) == 0 &&
      cross_block_bufs_.count(v->buffer_var()) == 0) {
    throw std::runtime_error("Global free not supported yet");
  }
}
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 97-123
```cpp
void CudaAnalysis::visit(const AllocatePtr& v) {
  StmtPtr p = v->get_parent();
  while (p) {
    ForPtr for_v = to<For>(p);
    if (for_v) {
      if (for_v->loop_options().is_gpu_block_index()) {
        // TODO: This isn't right if there's a thread index at a higher level
        // than this.
        cross_block_bufs_.insert(v->buffer_var());
        return;
      } else if (for_v->loop_options().is_gpu_thread_index()) {
        thread_local_bufs_.insert(v->buffer_var());
        return;
      }
    }
    p = p->get_parent();
  }
  throw std::runtime_error("Global alloc not supported yet");
}

void CudaAnalysis::visit(const PlacementAllocatePtr& v) {
  throw std::runtime_error("Memory reuse not supported yet");
}

void CudaAnalysis::visit(const ForPtr& v) {
  // Recurse first.
  v->body()->accept(this);
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 125-160
```cpp
  const LoopOptions& loop_options = v->loop_options();
  if (loop_options.is_gpu_block_index()) {
    int gpu_block_index = loop_options.gpu_block_index();
    if (gpu_block_index >= 3) {
      throw std::runtime_error("support only 3D gpu_block_index");
    }
    ExprPtr prev = nullptr;
    if (gpu_block_extents_.size() <= static_cast<size_t>(gpu_block_index)) {
      gpu_block_extents_.resize(gpu_block_index + 1);
    } else {
      prev = gpu_block_extents_[gpu_block_index];
    }
    if (!is_zero(v->start())) {
      throw std::runtime_error(
          "start must be zero for gpu_block_index: " +
          std::to_string(v->start()));
    }

    // NOLINTNEXTLINE(bugprone-branch-clone)
    if (prev == nullptr) {
      gpu_block_extents_[gpu_block_index] = v->stop();
    } else if (prev->isConstant() && immediateEquals(prev, 1)) {
      // extents must be positive so if the current extent is 1 then even if the
      // stop is symbolic it's the max.
      gpu_block_extents_[gpu_block_index] = v->stop();
    } else {
      gpu_block_extents_[gpu_block_index] =
          IRSimplifier::simplify(alloc<Max>(prev, v->stop(), true));
    }
  } else if (loop_options.is_gpu_thread_index()) {
    int gpu_thread_index = loop_options.gpu_thread_index();
    if (gpu_thread_index >= 3) {
      throw std::runtime_error("support only 3D gpu_thread_index");
    }
    ExprPtr prev = nullptr;
    if (gpu_thread_extents_.size() <= static_cast<size_t>(gpu_thread_index)) {
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 161-183
```cpp
      gpu_thread_extents_.resize(gpu_thread_index + 1);
    } else {
      prev = gpu_thread_extents_[gpu_thread_index];
    }
    if (!is_zero(v->start())) {
      throw std::runtime_error(
          "start must be zero for gpu_thread_index: " +
          std::to_string(v->start()));
    }

    // NOLINTNEXTLINE(bugprone-branch-clone)
    if (prev == nullptr) {
      gpu_thread_extents_[gpu_thread_index] = v->stop();
    } else if (prev->isConstant() && immediateEquals(prev, 1)) {
      // extents must be positive so if the current extent is 1 then even if the
      // stop is symbolic it's the max.
      gpu_thread_extents_[gpu_thread_index] = v->stop();
    } else {
      gpu_thread_extents_[gpu_thread_index] =
          IRSimplifier::simplify(alloc<Max>(prev, v->stop(), true));
    }
  }
}
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 185-217
```cpp
void CudaPrinter::print_flat_alloc(const AllocatePtr& alloc) {
  std::vector<ExprPtr> dims = alloc->dims();
  // TODO: this should be merged with the storage flattener.
  int64_t flat_size = 1;
  for (const auto& dim : dims) {
    auto dim_i = intValue(dim);
    if (dim_i) {
      flat_size *= *dim_i;
    } else {
      throw std::runtime_error("Only integer dimensions are supported for now");
    }
  }
  os() << dtypeToCppString(alloc->dtype()) << ' ' << (*alloc->buffer_var())
       << '[' << flat_size << "];" << '\n';
}

void CudaPrinter::visit(const AllocatePtr& v) {
  // TODO: handle dynamic shapes here.
  if (cuda_analysis_->cross_block_bufs().count(v->buffer_var()) != 0) {
    emitIndent();
    os() << "__shared__ ";
    print_flat_alloc(v);
    return;
  }

  if (cuda_analysis_->thread_local_bufs().count(v->buffer_var()) != 0) {
    emitIndent();
    print_flat_alloc(v);
    return;
  }

  throw std::runtime_error("Encountered Alloc not local to block or thread");
}
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 219-253
```cpp
void CudaPrinter::visit(const FreePtr& v) {
  // do nothing
}

void CudaPrinter::visit(const ForPtr& v) {
  IRPrinter::visit(v);
}

void CudaPrinter::visit(const CastPtr& v) {
  std::string castFn = v->dtype().scalar_type() == ScalarType::Half
      ? "__float2half"
      : v->dtype().scalar_type() == ScalarType::BFloat16 ? "__float2bfloat16"
      : v->src_value()->dtype().scalar_type() == ScalarType::Half
      ? "__half2float"
      : v->src_value()->dtype().scalar_type() == ScalarType::BFloat16
      ? "__bfloat162float"
      : ("(" + dtypeToCppString(v->dtype()) + ")");
  os() << castFn << '(';
  v->src_value()->accept(this);
  os() << ')';
}

void CudaPrinter::visit(const IntrinsicsPtr& v) {
  if (v->op_type() == IntrinsicsOp::kRand) {
    os() << "Uint32ToFloat(" << *rand_func_ << "())";
    return;
  }

  std::string func_name = v->func_name();

  // get type of resulting expression.
  ScalarType returnType = v->param(0)->dtype().scalar_type();
  for (size_t i = 1; i < v->nparams(); ++i) {
    returnType = promoteTypes(returnType, v->param(i)->dtype().scalar_type());
  }
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 255-279
```cpp
  if (returnType == ScalarType::Half || returnType == ScalarType::Float) {
    func_name = func_name + "f";
  }
  if (v->op_type() == IntrinsicsOp::kAbs &&
      !c10::isIntegralType(returnType, true)) {
    // since kAbs's func_name is `abs`, prefix `f` for floating point
    func_name = "f" + func_name;
  }
  if (v->op_type() == IntrinsicsOp::kIsNan) {
    func_name = "isnan";
  }

  os() << func_name << '(';
  for (const auto i : c10::irange(v->nparams())) {
    if (i > 0) {
      os() << ", ";
    }
    os() << *v->param(i);
  }
  os() << ')';
}

void CudaPrinter::visit(const ExternalCallPtr& v) {
  throw unimplemented_lowering(v);
}
```
- **EN**: This chunk defines `unimplemented_lowering`, which lowers higher-level IR into a backend-specific executable form. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `unimplemented_lowering`，其作用是把高层 IR 降级为后端特定的可执行形式。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 281-304
```cpp
void CudaPrinter::visit(const LoadPtr& v) {
  // TODO: find a better metric in using ldg or not. Support different dtypes.
  // Detects whether the load target is also a store target.
  // TODO: this is currently too wide. It detects whether a store-target
  // exists within the program. In fact, this check is only necessary within a
  // kernel.
  if (v->indices().empty()) {
    os() << *v->base_handle();
    return;
  }
  if (v->dtype().scalar_type() == ScalarType::Bool ||
      v->dtype().scalar_type() == ScalarType::Half ||
      v->dtype().scalar_type() == ScalarType::BFloat16) {
    // There's no __ldg overload for bool or half.
    os() << *v->base_handle() << '[' << *v->flat_index() << ']';
    return;
  }
  if (cuda_analysis_->is_buf_store_target(v->buf())) {
    // Cuda __ldg can only be applied on read-only buffers.
    os() << *v->base_handle() << '[' << *v->flat_index() << ']';
    return;
  }
  os() << "__ldg(" << *v->base_handle() << " + " << *v->flat_index() << ')';
}
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 306-335
```cpp
// TODO: maybe this should be a more shared location?
// TODO: investigate how "ExprPtr" can be implicitly converted to "ExprHandle"
// as a bool.
static bool CheckEqual(const ExprPtr& lhs, const ExprPtr& rhs) {
  // The fast path. Checks if the pointers are the same.
  if (lhs == rhs) {
    return true;
  }
  ExprHandle diff = Sub::make(ExprHandle(lhs), ExprHandle(rhs));
  ExprHandle diff_s = IRSimplifier::simplify(diff);
  return immediateEquals(diff_s.node(), 0);
}

class AtomicAddFuser : public IRMutator {
 public:
  AtomicAddFuser(
      const std::unordered_set<VarPtr>& thread_local_bufs,
      const GPUMetaVarRewriter& metavars)
      : thread_local_bufs_(thread_local_bufs) {
    const std::vector<ExprPtr>& block_extents = metavars.gpu_block_extents();
    const std::vector<VarPtr>& block_vars = metavars.gpu_block_vars();
    for (size_t i = 0; i < block_extents.size(); ++i) {
      MetaVarExtent extent{block_extents[i], false};
      if (extent.expr->isConstant() && immediateEquals(extent.expr, 1)) {
        extent.trivial = true;
      } else {
        nontrivial_metavars_.insert(block_vars[i]);
      }
      metavars_[block_vars[i]] = extent;
    }
```
- **EN**: It introduces or extends AtomicAddFuser, which define the primary data structures or interfaces for this portion of the file. This chunk defines `CheckEqual`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 AtomicAddFuser，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `CheckEqual`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 337-372
```cpp
    const std::vector<ExprPtr>& thread_extents = metavars.gpu_thread_extents();
    const std::vector<VarPtr>& thread_vars = metavars.gpu_thread_vars();
    for (size_t i = 0; i < thread_extents.size(); ++i) {
      MetaVarExtent extent{thread_extents[i], false};
      if (extent.expr->isConstant() && immediateEquals(extent.expr, 1)) {
        extent.trivial = true;
      } else {
        nontrivial_metavars_.insert(thread_vars[i]);
      }
      metavars_[thread_vars[i]] = extent;
    }
  }

  StmtPtr mutate(const StorePtr& v) override {
    BufPtr buf = v->buf();

    // Thread locals never need to be atomic.
    if (thread_local_bufs_.count(buf->base_handle()) != 0) {
      return v;
    }

    ScalarType dtype = v->value()->dtype().scalar_type();
    if (dtype != ScalarType::Float && dtype != ScalarType::Double) {
      return v;
    }
    AddPtr add_v = to<Add>(v->value());
    if (!add_v) {
      return v;
    }
    LoadPtr load_v = to<Load>(add_v->lhs());
    if (!load_v) {
      return v;
    }
    if (v->base_handle() != load_v->base_handle()) {
      return v;
    }
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 373-407
```cpp
    if (v->indices().empty() && load_v->indices().empty()) {
      return v;
    }
    bool index_equal = CheckEqual(v->flat_index(), load_v->flat_index());
    if (!index_equal) {
      return v;
    }

    // TODO: this checks that the metavars occur directly as an index, but this
    // is pessimistic, blockIdx.x + 1 is fine too if there is no overlapping.
    std::unordered_set<VarPtr> vars_to_find = nontrivial_metavars_;
    for (const ExprPtr& e : v->indices()) {
      if (VarPtr v = to<Var>(e)) {
        vars_to_find.erase(v);
      }
    }

    if (vars_to_find.empty()) {
      // All metavars accounted for.
      return v;
    }

    return alloc<AtomicAdd>(buf, v->indices(), add_v->rhs());
  }

 private:
  // NOLINTNEXTLINE(cppcoreguidelines-avoid-const-or-ref-data-members)
  const std::unordered_set<VarPtr>& thread_local_bufs_;
  struct MetaVarExtent {
    ExprPtr expr{nullptr};
    bool trivial{false};
  };
  std::unordered_map<VarPtr, MetaVarExtent> metavars_;
  std::unordered_set<VarPtr> nontrivial_metavars_;
};
```
- **EN**: It introduces or extends MetaVarExtent, which define the primary data structures or interfaces for this portion of the file. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 MetaVarExtent，这些类型定义了本段涉及的主要数据结构或接口。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 409-443
```cpp
void CudaPrinter::visit(const StorePtr& v) {
  emitIndent();
  if (v->indices().empty()) {
    os() << *v->base_handle() << " = ";
  } else {
    os() << *v->base_handle() << '[' << *v->flat_index() << "] = ";
  }
  os() << *v->value() << ';';
  os() << '\n';
}

void CudaPrinter::visit(const AtomicAddPtr& v) {
  emitIndent();
  if (cuda_analysis_->thread_local_bufs().count(v->base_handle()) > 0) {
    // atomicAdd only works on global and shared memory
    os() << *v->base_handle() << '[' << *v->flat_index()
         << "] += " << *v->value() << ';';
  } else {
    os() << "atomicAdd(&" << *v->base_handle() << '[' << *v->flat_index() << ']'
         << ", " << *v->value() << ");";
  }
  os() << '\n';
}

void CudaPrinter::visit(const MaxPtr& v) {
  if (v->dtype().is_integral()) {
    os() << "max(";
  } else {
    os() << "maximum(";
  }
  v->lhs()->accept(this);
  os() << ',';
  v->rhs()->accept(this);
  os() << ')';
}
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 445-478
```cpp
void CudaPrinter::visit(const MinPtr& v) {
  if (v->dtype().is_integral()) {
    os() << "min(";
  } else {
    os() << "minimum(";
  }
  v->lhs()->accept(this);
  os() << ',';
  v->rhs()->accept(this);
  os() << ')';
}

void CudaPrinter::visit(const IfThenElsePtr& v) {
  os() << "((";
  v->condition()->accept(this);
  os() << ") ? ";
  v->true_value()->accept(this);
  os() << " : ";
  v->false_value()->accept(this);
  os() << ')';
}

void CudaPrinter::visit(const BlockPtr& v) {
  os() << '{' << '\n';
  indent_++;

  for (const StmtPtr& s : v->stmts()) {
    s->accept(this);
  }

  indent_--;
  emitIndent();
  os() << '}';
}
```
- **EN**: This chunk defines `visit`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `visit`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 480-515
```cpp
void CudaPrinter::visit(const LetPtr& v) {
  emitIndent();
  os() << dtypeToCppString(v->var()->dtype());
  os() << ' ' << *v->var() << " = ";
  v->value()->accept(this);
  os() << ';' << '\n';
}

class PrioritizeLoad : public IRMutator {
 public:
  ExprPtr mutate(const LoadPtr& v) override {
    // Look at the declaration of this variable for more details.
    if (nested_if_then_else_ > 0) {
      return IRMutator::mutate(v);
    }
    if (nested_let_) {
      return IRMutator::mutate(v);
    }
    if (thread_local_bufs_.count(v->base_handle()) > 0) {
      return IRMutator::mutate(v);
    }
    if (v->indices().empty()) {
      return IRMutator::mutate(v);
    }
    if (nested_store_) {
      if (v->base_handle() == nested_store_->buf()->base_handle() &&
          v->indices().size() == nested_store_->indices().size()) {
        // also check indices
        bool same = true;
        for (const auto i : c10::irange(v->indices().size())) {
          if (!exprEquals(v->indices()[i], nested_store_->indices()[i])) {
            same = false;
            break;
          }
        }
        if (same) {
```
- **EN**: It introduces or extends PrioritizeLoad, which define the primary data structures or interfaces for this portion of the file. This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 PrioritizeLoad，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 516-550
```cpp
          return IRMutator::mutate(v);
        }
      } else if (nested_store_->indices().empty()) {
        return IRMutator::mutate(v);
      }
    }

    MemLoadList& load_list = load_stack_.back();
    VarPtr load_new_var = alloc<Var>("v", v->dtype());
    ExprPtr new_value = IRMutator::mutate(v);
    load_list.emplace_back(load_new_var, new_value);

    return load_new_var;
  }

  ExprPtr mutate(const CastPtr& v) override {
    LoadPtr src_load = to<Load>(v->src_value());
    ExprPtr new_src = v->src_value()->accept_mutator(this);
    VarPtr new_var = to<Var>(new_src);
    if (!src_load || !new_var) {
      return alloc<Cast>(v->dtype(), new_src);
    }

    // We just did the prioritize load, let's fold in the Cast.
    MemLoadList& load_list = load_stack_.back();
    assert(!load_list.empty());
    auto pair = load_list.back();
    assert(pair.first == new_var);
    load_list.pop_back();

    new_var = alloc<Var>("v", v->dtype());
    ExprPtr new_value = alloc<Cast>(v->dtype(), pair.second);
    load_list.emplace_back(new_var, new_value);
    return new_var;
  }
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 552-582
```cpp
  StmtPtr mutate(const StorePtr& v) override {
    StorePtr last = nested_store_;
    nested_store_ = v;
    StmtPtr s = IRMutator::mutate(v);
    nested_store_ = last;
    return s;
  }

  StmtPtr mutate(const LetPtr& v) override {
    nested_let_ = true;
    StmtPtr s = IRMutator::mutate(v);
    nested_let_ = false;
    return s;
  }

  StmtPtr mutate(const BlockPtr& v) override {
    std::list<StmtPtr> stmts = v->stmts();
    for (const StmtPtr& stmt : stmts) {
      PushList();
      StmtPtr stmt_new = stmt->accept_mutator(this);

      AddMemLoadsFromList(v, stmt);
      PopList();

      if (stmt_new == stmt) {
        continue;
      }
      v->replace_stmt(stmt, stmt_new);
    }
    return v;
  }
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 584-614
```cpp
  ExprPtr mutate(const IfThenElsePtr& v) override {
    nested_if_then_else_++;
    ExprPtr new_v = IRMutator::mutate(v);
    nested_if_then_else_--;
    return new_v;
  }

 private:
  using MemLoadEntry = std::pair<VarPtr, ExprPtr>;
  using MemLoadList = std::vector<MemLoadEntry>;
  using MemoryLoadStack = std::vector<MemLoadList>;

  void PushList() {
    load_stack_.emplace_back();
  }

  void PopList() {
    load_stack_.pop_back();
  }

  void AddMemLoadsFromList(const BlockPtr& block, const StmtPtr& last) {
    MemLoadList& load_list = load_stack_.back();
    if (load_list.empty()) {
      return;
    }

    for (auto& pair : load_list) {
      StmtPtr news = alloc<Let>(pair.first, pair.second);
      block->insert_stmt_before(news, last);
    }
  }
```
- **EN**: This chunk defines `AddMemLoadsFromList`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `AddMemLoadsFromList`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 616-643
```cpp
  MemoryLoadStack load_stack_;
  // TODO: For now, we are not moving the loads with the IfThenElse.
  // Eventually, we should switch to a more generic structure like:
  // int v2 = IfThenElse(cond, true_v, false_v) + 2 ->
  //
  // int v;
  // if (cond) {
  //   v = true_v;
  // } else {
  //   v = false_v;
  // }
  // int v2 = v + 2;
  int nested_if_then_else_{0};
  StorePtr nested_store_{nullptr};
  bool nested_let_{false};
  std::unordered_set<VarPtr> thread_local_bufs_;
};

std::string CudaCodeGen::GetUniqueFuncName(const std::string& func_prefix) {
  int64_t counter = 0;
  std::string name = func_prefix;
  while (taken_func_names.count(name)) {
    name = func_prefix + "_" + std::to_string(counter++);
  }

  taken_func_names.insert(name);
  return name;
}
```
- **EN**: This chunk defines `GetUniqueFuncName`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `GetUniqueFuncName`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 645-676
```cpp
bool GPUMetaVarRewriter::isFullExtent() {
  {
    auto& extents = cuda_analysis_->gpu_block_extents();
    for (int i = 0; i < 3; ++i) {
      if (!exprEquals(current_block_reach_[i], extents[i])) {
        return false;
      }
    }
  }

  {
    auto& extents = cuda_analysis_->gpu_thread_extents();
    for (int i = 0; i < 3; ++i) {
      if (!exprEquals(current_thread_reach_[i], extents[i])) {
        return false;
      }
    }
  }

  return true;
}

StmtPtr GPUMetaVarRewriter::mutate(const ForPtr& v) {
  StmtPtr body = v->body();
  ExprPtr old_reach = nullptr;
  const LoopOptions& loop_options = v->loop_options();
  if (loop_options.is_gpu_block_index()) {
    int gpu_block_index = loop_options.gpu_block_index();
    if (gpu_block_index >= 3) {
      throw std::runtime_error("support only 3D gpu_block_index");
    }
    old_reach = current_block_reach_[gpu_block_index];
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 678-708
```cpp
    // Extents must be positive, assume >= 1.
    if (old_reach->isConstant() && immediateEquals(old_reach, 1)) {
      current_block_reach_[gpu_block_index] = v->stop();
    } else {
      current_block_reach_[gpu_block_index] =
          IRSimplifier::simplify(alloc<Max>(old_reach, v->stop(), true));
    }

    VarPtr metaVar = gpu_block_vars_[gpu_block_index];
    body = Substitute(Stmt::clone(body), {{v->var(), metaVar}});
  } else if (loop_options.is_gpu_thread_index()) {
    int gpu_thread_index = loop_options.gpu_thread_index();
    if (gpu_thread_index >= 3) {
      throw std::runtime_error("support only 3D gpu_thread_index");
    }
    old_reach = current_thread_reach_[gpu_thread_index];

    // Extents must be positive, assume >= 1.
    if (old_reach->isConstant() && immediateEquals(old_reach, 1)) {
      current_thread_reach_[gpu_thread_index] = v->stop();
    } else {
      current_thread_reach_[gpu_thread_index] =
          IRSimplifier::simplify(alloc<Max>(old_reach, v->stop(), true));
    }

    VarPtr metaVar = gpu_thread_vars_[gpu_thread_index];
    body = Substitute(Stmt::clone(body), {{v->var(), metaVar}});
  }

  // Recurse into body block.
  body = Stmt::clone(body->accept_mutator(this));
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 710-741
```cpp
  // pop the internal reach off the stack.
  if (loop_options.is_gpu_block_index()) {
    current_block_reach_[loop_options.gpu_block_index()] = old_reach;
    return body;
  } else if (loop_options.is_gpu_thread_index()) {
    current_thread_reach_[loop_options.gpu_thread_index()] = old_reach;
    return body;
  }

  return v->cloneWithNewBody(body);
}

StmtPtr GPUMetaVarRewriter::mutate(const BlockPtr& v) {
  std::vector<Segment> innerSegments;
  Segment current;

  auto pushAndReset = [&](bool mask) {
    if (!current.empty()) {
      innerSegments.push_back(current);
    }
    current.reset(mask);
  };

  // Here's we're slicing the Block's contents into segments that should have
  // the same launch reach. Segments are comprised of all statements that aren't
  // loops - which are their own segments. Some operations, such as threading
  // and memory ops should never be masked and so also get their own segment.
  for (const StmtPtr& stmt : *v) {
    StmtPtr stmt_new = stmt->accept_mutator(this);
    if (stmt == stmt_new) {
      stmt_new = Stmt::clone(stmt_new);
    }
```
- **EN**: This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 743-776
```cpp
    // Likewise, Allocate and Free should never be masked.
    if (to<Allocate>(stmt) || to<Free>(stmt)) {
      pushAndReset(false);
    }

    // If the current stmt *was* a loop, it's a segment boundary.
    if (ForPtr f = to<For>(stmt)) {
      pushAndReset(false);
    }

    current.stmts().push_back(stmt_new);
    // if the current segment should not be masked, it's a segment boundary on
    // the far side as well.
    if (!current.mask()) {
      pushAndReset(true);
    }
  }

  if (!current.empty()) {
    innerSegments.push_back(current);
  }

  // We are max extent in all dimensions, so need no masks at this level.
  if (isFullExtent()) {
    // flatten inner segments.
    std::vector<StmtPtr> stmts;
    for (auto& v : innerSegments) {
      for (const auto& s : v.stmts()) {
        stmts.push_back(s);
      }
    }

    return alloc<Block>(stmts);
  }
```
- **EN**: This chunk continues `mutate` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `mutate`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 778-813
```cpp
  std::vector<StmtPtr> stmts;
  for (auto& segment : innerSegments) {
    bool need_sync = false;
    // We never mask loops, they'll mask their contents.
    if (!segment.mask()) {
      TORCH_INTERNAL_ASSERT(segment.stmts().size() == 1, buildErrorMessage());
      stmts.push_back(segment.stmts()[0]);
      continue;
    }

    // If we get here, we must mask since we're not full reach and our direct
    // child isn't a For.
    StmtPtr inner = alloc<Block>(segment.stmts());
    // threads inside blocks.
    auto& thread_extents = cuda_analysis_->gpu_thread_extents();
    for (size_t i = 0; i < gpu_thread_vars_.size(); ++i) {
      if (!exprEquals(current_thread_reach_[i], thread_extents[i])) {
        need_sync = true;
        // Mask it against the current dimensions.
        inner = alloc<Cond>(
            alloc<CompareSelect>(
                gpu_thread_vars_[i],
                current_thread_reach_[i],
                CompareSelectOperation::kLT),
            inner,
            nullptr);
      }
    }
    auto& block_extents = cuda_analysis_->gpu_block_extents();
    for (size_t i = 0; i < gpu_block_vars_.size(); ++i) {
      if (!exprEquals(current_block_reach_[i], block_extents[i])) {
        // Mask it against the current dimensions.
        inner = alloc<Cond>(
            alloc<CompareSelect>(
                gpu_block_vars_[i],
                current_block_reach_[i],
```
- **EN**: This chunk continues `mutate` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `mutate`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 814-848
```cpp
                CompareSelectOperation::kLT),
            inner,
            nullptr);
      }
    }

    if (need_sync) {
      stmts.push_back(alloc<SyncThreads>());
    }
    stmts.push_back(inner);
    if (need_sync) {
      stmts.push_back(alloc<SyncThreads>());
    }
  }

  return alloc<Block>(stmts);
}

static std::ostream& operator<<(
    std::ostream& out,
    const std::vector<ExprPtr>& exprs) {
  size_t i = 0;
  for (const auto& expr : exprs) {
    if (i++ > 0) {
      out << ", ";
    }
    out << *expr;
  }
  return out;
}

static constexpr const char* device_resource_string = R"(
#define NAN __int_as_float(0x7fffffff)
#define POS_INFINITY __int_as_float(0x7f800000)
#define NEG_INFINITY __int_as_float(0xff800000)
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk continues `mutate` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段延续了 `mutate`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 850-879
```cpp
)";

static constexpr const char* shared_resource_string = R"(
template<typename T>
__device__ T maximum(T a, T b) {
  return isnan(a) ? a : (a > b ? a : b);
}

template<typename T>
__device__ T minimum(T a, T b) {
  return isnan(a) ? a : (a < b ? a : b);
}

)";

void CudaCodeGen::Initialize() {
  // TODO: handle multiple kernels.
  // TODO: handle dynamic dimension.
  // TODO: call nvrtc.
  // TODO: merge HasRand with CudaAnalysis.
  GenericIntrinsicsExpander intrinsics_expander;
  apply_mutator(&intrinsics_expander);

  HasRand has_rand_func(stmt());
  has_random_ = has_rand_func.has_rand();
  cuda_analysis_ = std::make_unique<CudaAnalysis>();
  printer_ =
      std::make_unique<CudaPrinter>(&oss_, cuda_analysis_.get(), has_random_);
  metavar_rewriter_ =
      std::make_unique<GPUMetaVarRewriter>(cuda_analysis_.get());
```
- **EN**: This chunk defines `has_rand_func`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `has_rand_func`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 881-916
```cpp
  // Check whether the statement uses the Half type, if so add the
  // half_support_literal.
  StmtPtr stmt_v = stmt();
  HalfChecker halfChecker(buffer_args());
  stmt_v->accept(&halfChecker);

  os() << device_resource_string << shared_resource_string;

  if (has_random_) {
    os() << philox_random_string << '\n';
  }

  if (halfChecker.hasHalf()) {
    os() << fuser::cuda::half_support_literal << '\n';
  }
  if (halfChecker.hasBFloat16()) {
    os() << fuser::cuda::bfloat16_support_literal << '\n';
  }

  std::string func_name = GetUniqueFuncName(kernel_func_name());
  os() << "extern \"C\" __global__" << '\n';
#if defined(USE_ROCM)
  // CUDA has a default limit of threads per block (=flat work group size)
  // of 1024, but ROCm uses 256 by default. At the time of writing
  // (#45506), I am unaware of a stricter limit that TensorExpr imposes
  // (maybe for perf),so I use 1024 as maximum flat work group size.
  // We put a minimum value of 1, this is also used by hip (ROCm 3.8) in
  // the __launch_bound__ implementation. The arguments for the attribute
  // are (min, max), for details see the documentation at
  // https://clang.llvm.org/docs/AttributeReference.html#amdgpu-flat-work-group-size
  os() << "__attribute__((amdgpu_flat_work_group_size(1, 1024)))" << std::endl;
#endif
  os() << "void " << func_name << '(';
  const std::vector<BufferArg> buffer_args = this->buffer_args();
  for (size_t i = 0; i < buffer_args.size(); i++) {
    if (i > 0) {
```
- **EN**: This chunk defines `halfChecker`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `halfChecker`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 917-951
```cpp
      os() << ", ";
    }
    const BufferArg& buffer_arg = buffer_args[i];
    VarPtr var = buffer_arg.var();
    Dtype dtype = buffer_arg.dtype();

    os() << printer_->dtypeToCppString(dtype)
         << (buffer_arg.isVar() ? " " : "* ")
         << name_manager()->get_unique_name(var);
  }
  VarPtr rand_seed;
  VarPtr rand_offset;
  if (has_random_) {
    // TODO: switch to kUint64 when it is available.
    rand_seed = alloc<Var>("rand_seed", kInt);
    rand_offset = alloc<Var>("rand_offset", kInt);
    std::string uint64_str = "unsigned long long";
    os() << ", " << uint64_str << ' ' << *rand_seed << ", " << uint64_str << ' '
         << *rand_offset;
  }
  os() << ") {";
  os() << '\n';

  if (has_random_) {
    VarPtr idx = alloc<Var>("idx", kInt);
    os() << "int " << *idx << " = blockIdx.x*blockDim.x + threadIdx.x;" << '\n';
    VarPtr rand_func = printer_->rand_func();
    os() << "Philox " << *rand_func << '(' << *rand_seed << ", " << *idx << ", "
         << *rand_offset << ");" << '\n';
    os() << '\n';
  }

  stmt_v->accept(cuda_analysis_.get());

  stmt_v = stmt_v->accept_mutator(metavar_rewriter_.get());
```
- **EN**: This chunk continues `halfChecker` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `halfChecker`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 953-980
```cpp
  AtomicAddFuser atomic_add_fuser(
      cuda_analysis_->thread_local_bufs(), *metavar_rewriter_);
  stmt_v = stmt_v->accept_mutator(&atomic_add_fuser);

  stmt_v = registerize(stmt_v);

  PrioritizeLoad prioritize_load;
  stmt_v = stmt_v->accept_mutator(&prioritize_load);

  // The registerizer might insert half-type scalars, we don't want this.
  HalfRewriter hsFix;
  stmt_v = stmt_v->accept_mutator(&hsFix);

  stmt_v = IRSimplifier::simplify(stmt_v);
  set_stmt(stmt_v);

  stmt_v->accept(printer_.get());
  os() << '\n';
  os() << '}';

  // Check that all block extents had been set.
  const std::vector<ExprPtr>& gpu_block_extents =
      metavar_rewriter_->gpu_block_extents();
  for (size_t i = 0; i < gpu_block_extents.size(); i++) {
    if (!gpu_block_extents[i]) {
      throw std::runtime_error("Missing gpu_block_index: " + std::to_string(i));
    }
  }
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 982-1007
```cpp
  // Precompute block and thread extents for call_with_numel().  If
  // precomputation can't be done (block/thread extents aren't
  // constant), then disallow call_with_numel.
  auto block_extents = metavar_rewriter_->gpu_block_extents();
  auto thread_extents = metavar_rewriter_->gpu_thread_extents();
  bool canCallWithNumel =
      !has_random_ && !block_extents.empty() && !thread_extents.empty();
  for (size_t i = 1; i < block_extents.size() && canCallWithNumel; i++) {
    canCallWithNumel = canCallWithNumel && block_extents[i]->isConstant() &&
        immediateAs<int>(block_extents[i]) == 1;
  }
  for (size_t i = 1; i < thread_extents.size() && canCallWithNumel; i++) {
    canCallWithNumel = canCallWithNumel && thread_extents[i]->isConstant() &&
        immediateAs<int>(thread_extents[i]) == 1;
  }
  if (canCallWithNumel && thread_extents[0]->isConstant()) {
    // We assume block_extents[0] is output.numel()/thread_block_size_.
    thread_block_size_ = immediateAs<int>(thread_extents[0]);
  } else {
    // Disable call_with_numel.
    thread_block_size_ = -1;
  }

  // Build an LLVM based eval expression for the extents
  block_extents_eval_.reserve(block_extents.size());
  std::vector<BufferArg> extents_buffer_args;
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1009-1044
```cpp
  // We need to extract the args that are used in the thread and block extents
  // from bufferArgs and only use those for the `ExprEval` below. Without this,
  // bufferArgs might contain arbitrary types that are not handled by LLVM and
  // hence would result in an error.
  std::unordered_set<VarPtr> vars_in_extents;
  for (const auto& be : block_extents) {
    auto v = VarFinder::find(be);
    vars_in_extents.insert(v.begin(), v.end());
  }
  for (const auto& te : thread_extents) {
    auto v = VarFinder::find(te);
    vars_in_extents.insert(v.begin(), v.end());
  }
  for (const size_t i : c10::irange(buffer_args.size())) {
    if (vars_in_extents.count(buffer_args[i].var())) {
      extents_buffer_args.push_back(buffer_args[i]);
      arg_pos_in_extents_.push_back(true);
    } else {
      arg_pos_in_extents_.push_back(false);
    }
  }
  for (const auto& be : block_extents) {
#ifdef TORCH_ENABLE_LLVM
    block_extents_eval_.emplace_back(
        ExprEval<LLVMCodeGen>(ExprHandle(be), extents_buffer_args));
#else
    block_extents_eval_.emplace_back(ExprHandle(be), extents_buffer_args);
#endif
  }
  thread_extents_eval_.reserve(thread_extents.size());
  for (const auto& te : thread_extents) {
#ifdef TORCH_ENABLE_LLVM
    thread_extents_eval_.emplace_back(
        ExprEval<LLVMCodeGen>(ExprHandle(te), extents_buffer_args));
#else
    thread_extents_eval_.emplace_back(ExprHandle(te), extents_buffer_args);
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1045-1075
```cpp
#endif
  }

  GRAPH_DEBUG(
      "Fused TE CUDA kernel:\n",
      oss_.str(),
      "\n",
      "gpu_block_extents: (",
      metavar_rewriter_->gpu_block_extents(),
      ")\n",
      "gpu_thread_extents: (",
      metavar_rewriter_->gpu_thread_extents(),
      ")");

  CompileToNVRTC(oss_.str(), func_name);
}

void CudaCodeGen::call_with_numel(void** args, int64_t numel) {
  if (C10_UNLIKELY(numel == 0)) {
    return;
  }
  if (C10_UNLIKELY(thread_block_size_ <= 0)) {
    TORCH_INTERNAL_ASSERT(
        thread_block_size_ >= 0,
        "call_with_numel() requires a precomputed thread block size");
  }

  auto const& buffer_args = this->buffer_args();
  size_t gpu_block_extents =
      (numel + thread_block_size_ - 1) / thread_block_size_;
  size_t gpu_thread_extents = thread_block_size_;
```
- **EN**: This chunk defines `call_with_numel`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `call_with_numel`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1077-1111
```cpp
  // In CUDA we need to pass pointers to pointers for buffers, thus we need to
  // go over args and add an extra indirection for such non-scalar
  // arguments.
  // Why? See some details here:
  // https://stackoverflow.com/questions/34388712/cannot-understand-how-jcuda-culaunchkernel-work
  std::vector<void*> ptr_to_args(buffer_args.size());
  for (size_t i = 0; i < buffer_args.size(); i++) {
    ptr_to_args[i] = buffer_args[i].isVar() ? args[i] : (&args[i]);
  }

  const auto device = this->device().index();
  const auto prior_device = at::cuda::current_device();
  if (prior_device != device) {
    at::cuda::set_device(device);
  }

  auto stream = at::cuda::getCurrentCUDAStream();
  at::cuda::jit::initializeCudaContext();
  AT_CUDA_DRIVER_CHECK(nvrtc().cuLaunchKernel(
      function_,
      gpu_block_extents,
      1,
      1,
      gpu_thread_extents,
      1,
      1,
      0,
      stream,
      ptr_to_args.data(),
      nullptr));

  if (prior_device != device) {
    at::cuda::set_device(prior_device);
  }
}
```
- **EN**: This chunk defines `ptr_to_args`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `ptr_to_args`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1113-1148
```cpp
void CudaCodeGen::call_raw(const std::vector<void*>& raw_args) {
  auto const& buffer_args = this->buffer_args();

  // TODO: move as much of this into the constructors.
  const std::vector<ExprPtr>& gpu_block_extents =
      metavar_rewriter_->gpu_block_extents();
  const std::vector<ExprPtr>& gpu_thread_extents =
      metavar_rewriter_->gpu_thread_extents();
  if (gpu_block_extents.size() > 3 || gpu_thread_extents.size() > 3) {
    throw malformed_input(
        "cuda_codegen: block or thread extent greater than 3D");
  }

  std::vector<int64_t> gpu_block_extents_v(3, 1);
  std::vector<int64_t> gpu_thread_extents_v(3, 1);

  // evaluate all the block/thread extents into values
  // TODO: eventually, codegen these calculations and make them part of the
  // module.
  std::vector<void*> extent_args;
  size_t raw_args_size = raw_args.size();
  extent_args.reserve(raw_args_size);
  for (size_t i = 0; i < raw_args_size; ++i) {
    if (arg_pos_in_extents_[i]) {
      extent_args.push_back(raw_args[i]);
    }
  }
  for (size_t i = 0; i < gpu_block_extents.size(); i++) {
    if (gpu_block_extents[i]->isConstant()) {
      gpu_block_extents_v[i] = immediateAs<int64_t>(gpu_block_extents[i]);
      continue;
    }
    {
      // invocation of block_extents_eval_ isn't thread safe and this function
      // may be invoked by multiple threads
      std::lock_guard<std::mutex> guard(eval_lock_);
```
- **EN**: This chunk defines `guard`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `guard`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1149-1178
```cpp
      gpu_block_extents_v[i] =
          block_extents_eval_[i].value<int64_t>(extent_args);
    }
  }
  for (size_t i = 0; i < gpu_thread_extents.size(); i++) {
    if (gpu_thread_extents[i]->isConstant()) {
      gpu_thread_extents_v[i] = immediateAs<int64_t>(gpu_thread_extents[i]);
      continue;
    }
    {
      std::lock_guard<std::mutex> guard(eval_lock_);
      gpu_thread_extents_v[i] =
          thread_extents_eval_[i].value<int64_t>(extent_args);
    }
  }

  // Skip launching the kernel if there are no elements to process.
  for (auto extent : gpu_block_extents_v) {
    if (extent == 0) {
      return;
    }
  }

  auto ptr_count = buffer_args.size();
  // If the kernel has a rand call in it, add two extra arguments for random
  // seed and offset.
  if (has_random_) {
    ptr_count += 2;
  }
  std::vector<void*> ptr_to_args(ptr_count);
```
- **EN**: This chunk defines `ptr_to_args`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `ptr_to_args`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1180-1206
```cpp
  // In CUDA we need to pass pointers to pointers for buffers, thus we need to
  // go over raw_args and add an extra indirection for such non-scalar
  // arguments.
  // Why? See some details here:
  // https://stackoverflow.com/questions/34388712/cannot-understand-how-jcuda-culaunchkernel-work
  for (size_t i = 0; i < buffer_args.size(); i++) {
    ptr_to_args[i] =
        buffer_args[i].isVar() ? raw_args[i] : const_cast<void**>(&raw_args[i]);
  }

  if (has_random_) {
    uint64_t rand_seed = uint64_t(-1);
    uint64_t rand_offset = uint64_t(-1);
    auto gen = at::cuda::detail::getDefaultCUDAGenerator();
    // TODO: total hack. Switch to numel when it is available.
    int64_t total_elements_per_thread = (1LL << 28);
    {
      std::lock_guard<std::mutex> lock(gen.mutex());
      auto philox_engine_inputs =
          at::check_generator<at::CUDAGeneratorImpl>(gen)->philox_engine_inputs(
              total_elements_per_thread);
      rand_seed = philox_engine_inputs.first;
      rand_offset = philox_engine_inputs.second;
    }
    ptr_to_args[buffer_args.size()] = &rand_seed;
    ptr_to_args[buffer_args.size() + 1] = &rand_offset;
  }
```
- **EN**: This chunk defines `lock`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `lock`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1208-1236
```cpp
  auto prior_device = at::cuda::current_device();
  if (prior_device != this->device().index()) {
    at::cuda::set_device(this->device().index());
  }
  // Launch the kernels
  auto stream = at::cuda::getCurrentCUDAStream();
  at::cuda::jit::initializeCudaContext();
  AT_CUDA_DRIVER_CHECK(nvrtc().cuLaunchKernel(
      function_,
      gpu_block_extents_v[0],
      gpu_block_extents_v[1],
      gpu_block_extents_v[2],
      gpu_thread_extents_v[0],
      gpu_thread_extents_v[1],
      gpu_thread_extents_v[2],
      0,
      stream,
      ptr_to_args.data(),
      nullptr));

  if (prior_device != this->device().index()) {
    at::cuda::set_device(prior_device);
  }
}

void CudaCodeGen::call(const std::vector<CallArg>& args) {
  if (args.size() != buffer_args().size()) {
    throw malformed_input("cuda_codegen: wrong number of args in call");
  }
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1238-1273
```cpp
  auto const& buffer_args = this->buffer_args();
  std::vector<void*> raw_args(buffer_args.size());
  for (size_t i = 0; i < buffer_args.size(); i++) {
    auto const& bufferArg = buffer_args[i];
    auto const& callArg = args[i];
    raw_args[i] = argToPtr(bufferArg, callArg);
  }
  call_raw(raw_args);
}

at::Tensor CudaCodeGen::empty_strided(
    c10::IntArrayRef size,
    c10::IntArrayRef stride,
    std::optional<c10::ScalarType> dtype_opt,
    std::optional<c10::Layout> layout_opt,
    std::optional<c10::Device> device_opt,
    std::optional<bool> pin_memory_opt) {
  c10::DeviceGuard device_guard(device_opt.value());
  return at::native::empty_strided_cuda(
      size, stride, dtype_opt, layout_opt, device_opt, pin_memory_opt);
}

void CudaCodeGen::CompileToNVRTC(
    const std::string& code,
    const std::string& func_name) {
  at::cuda::jit::initializeCudaContext();
  // Note: hacked at::DeviceGuard since at::DeviceGuard was failing to work
  // properly in some scenarios
  auto prior_device = at::cuda::current_device();
  if (prior_device != this->device().index()) {
    at::cuda::set_device(this->device().index());
  }
  // Acquires device and NVRTC properties (for compile arch and occupancy
  // calculations)
  cudaDeviceProp* prop = at::cuda::getCurrentDeviceProperties();
  int major = 0, minor = 0;
```
- **EN**: This chunk defines `CompileToNVRTC`, which lowers higher-level IR into a backend-specific executable form. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `CompileToNVRTC`，其作用是把高层 IR 降级为后端特定的可执行形式。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1274-1302
```cpp
  bool compile_to_sass = false;
  fuser::cuda::codegenOutputQuery(prop, major, minor, compile_to_sass);

  // Creates the NVRTC program
  nvrtcProgram program{nullptr};
  AT_CUDA_NVRTC_CHECK(nvrtc().nvrtcCreateProgram(
      &program, code.c_str(), nullptr, 0, nullptr, nullptr));

#if defined(USE_ROCM)
  std::vector<const char*> args = {"--std=c++20"};
  args.push_back("-hip-pch");
#else
  const std::string compute = std::string("--gpu-architecture=") +
#if !defined(USE_ROCM)
      // CUDA 11.1 allows going directly to SASS (sm_) instead of PTX (compute_)
      // which gives better backwards compatibility to work on older driver,
      // (since older driver doesn't necessarily recognize PTX emitted by new
      // toolkit);
      // Meanwhile, for forward compatibility (future device with
      // `compile_to_sass==false`), since SASS are not necessarily compatible,
      // we fallback to PTX instead.
      (compile_to_sass ? "sm_" : "compute_") +
#else
      "compute_" +
#endif
      std::to_string(major) + std::to_string(minor);
  const std::vector<const char*> args = {
      "--std=c++20", compute.c_str(), "-default-device"};
#endif
```
- **EN**: This chunk continues `CompileToNVRTC` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `CompileToNVRTC`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1304-1336
```cpp
  auto result = nvrtc().nvrtcCompileProgram(
      program, static_cast<int>(args.size()), args.data());
  if (result != NVRTC_SUCCESS) {
    size_t logsize = 0;
    AT_CUDA_NVRTC_CHECK(nvrtc().nvrtcGetProgramLogSize(program, &logsize));
    std::vector<char> log(logsize);
    AT_CUDA_NVRTC_CHECK(nvrtc().nvrtcGetProgramLog(program, log.data()));
    std::stringstream cu;
    cu << log.data() << '\n';
    cu << "nvrtc compilation failed: " << '\n';
    cu << code << '\n';
    throw std::runtime_error(cu.str());
  }
  ResourceGuard holdProgram(
      [&] { AT_CUDA_NVRTC_CHECK(nvrtc().nvrtcDestroyProgram(&program)); });
  AT_CUDA_NVRTC_CHECK(result);
  size_t ptx_size = 0;
  std::vector<char> ptx;
#if !defined(USE_ROCM)
  // compile_to_sass determines whether we are generating SASS or PTX, hence
  // the different API.
  auto getSize = compile_to_sass
      ? at::globalContext().getNVRTC().nvrtcGetCUBINSize
      : at::globalContext().getNVRTC().nvrtcGetPTXSize;
  auto getFunc = compile_to_sass ? at::globalContext().getNVRTC().nvrtcGetCUBIN
                                 : at::globalContext().getNVRTC().nvrtcGetPTX;
#else
  auto getSize = at::globalContext().getNVRTC().nvrtcGetPTXSize;
  auto getFunc = at::globalContext().getNVRTC().nvrtcGetPTX;
#endif
  AT_CUDA_NVRTC_CHECK(getSize(program, &ptx_size));
  ptx.resize(ptx_size);
  AT_CUDA_NVRTC_CHECK(getFunc(program, ptx.data()));
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1338-1352
```cpp
  CUmodule module{nullptr};
  AT_CUDA_DRIVER_CHECK(nvrtc().cuModuleLoadData(&module, ptx.data()));
  AT_CUDA_DRIVER_CHECK(
      nvrtc().cuModuleGetFunction(&function_, module, func_name.c_str()));

  if (prior_device != this->device().index()) {
    at::cuda::set_device(prior_device);
  }
}

CudaCodeGen::~CudaCodeGen() = default;

RegisterCodeGen<CudaCodeGen> cuda_codegen_reg("cuda_codegen");

} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk defines `cuda_codegen_reg`, which lowers higher-level IR into a backend-specific executable form. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `cuda_codegen_reg`，其作用是把高层 IR 降级为后端特定的可执行形式。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **ScopedVarName**
  - EN: `ScopedVarName` is a central symbol declared or implemented in this file.
  - CN: `ScopedVarName` 是本文件声明或实现的核心符号。
- **runtime_error**
  - EN: `runtime_error` is a central symbol declared or implemented in this file.
  - CN: `runtime_error` 是本文件声明或实现的核心符号。
- **TorchScript runtime**
  - EN: Coordinates execution plans, stacks, and operator dispatch for scripted programs.
  - CN: 协调脚本程序的执行计划、栈以及算子分发。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
- **IR simplification**
  - EN: Rewrites expressions and statements into simpler but equivalent forms.
  - CN: 把表达式和语句重写为更简单但等价的形式。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/cuda_codegen.h`, `torch/csrc/jit/tensorexpr/half_support.h`, `torch/csrc/jit/codegen/fuser/cuda/fused_kernel.h`, `torch/csrc/jit/codegen/fuser/cuda/resource_strings.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/tensorexpr/analysis.h`, `torch/csrc/jit/tensorexpr/cuda_random.h`, `torch/csrc/jit/tensorexpr/eval.h`, `torch/csrc/jit/tensorexpr/exceptions.h`, `torch/csrc/jit/tensorexpr/ir_simplifier.h`, `torch/csrc/jit/tensorexpr/registerizer.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `ATen/cuda/CUDAGeneratorImpl.h`, `ATen/native/cuda/jit_utils.h`, `c10/cuda/CUDAFunctions.h`, `c10/util/irange.h`, `ATen/NativeFunctions.h`, `ATen/ops/empty_strided_native.h`
- **Standard library / 标准库**: `unordered_map`, `utility`
- **Primary symbols in this file / 本文件核心符号**: `ScopedVarName`, `runtime_error`, `is_zero`, `nvrtc`, `dtypeToCppString`, `visit`, `print_flat_alloc`
- **Note / 说明**: 19 direct includes were detected; only the first few are listed above for readability. / 检测到 19 个直接包含，为便于阅读这里只列出前若干项。
