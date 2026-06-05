# te_wrapper.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/runtime/static/te_wrapper.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
#include <torch/csrc/jit/runtime/static/te_wrapper.h>

#include <ATen/CPUFunctions.h>
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/runtime/static/impl.h>
#include <torch/csrc/jit/tensorexpr/expr.h>
#include <torch/csrc/jit/tensorexpr/operators/misc.h>
#include <torch/csrc/jit/tensorexpr/operators/operators.h>

#include <utility>

namespace torch::jit {

using namespace torch::jit::tensorexpr;
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/runtime/static/te_wrapper.h, torch/csrc/jit/ir/ir.h, torch/csrc/jit/jit_log.h, and 4 more; ATen/c10 facilities such as ATen/CPUFunctions.h; standard-library headers such as utility. The namespace declarations place the code inside torch::jit, matching the surrounding JIT subsystem.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/runtime/static/te_wrapper.h、torch/csrc/jit/ir/ir.h、torch/csrc/jit/jit_log.h 等共 7 项；ATen/c10 基础设施，如 ATen/CPUFunctions.h；标准库头文件，如 utility。 命名空间声明把代码放入 torch::jit 中，与周边 JIT 子系统保持一致。

### Lines 17-31
```cpp
// Use the width of an AVX-512 vector by default; this happens to work OK for
// AVX2 as well. Some ops benefit from using multiple AVX ports, in which case
// they are vectorized by twice this constant.  An exception is logit, since it
// contains FP divide, which is single-ported.
static constexpr int kVectorWidth = 16;

#ifdef TORCH_ENABLE_LLVM

void TEWrapper::update(std::unique_ptr<LLVMCodeGen>&& cg_) {
  cg = std::move(cg_);
}

void TEWrapper::call(const std::vector<void*>& args) {
  cg->call_raw(args);
}
```
- **EN**: This chunk defines `call`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 这一段定义了 `call`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 33-48
```cpp
static void optimizePointwise(LoopNest* ln, Tensor target, int width) {
  std::vector<ForPtr> loops = ln->getLoopStmtsFor(target);
  ForPtr inner, tail;
  TORCH_CHECK(loops.size() > 0, "No loops created for pointwise op");
  ln->splitWithTail(loops[0], width, &inner, &tail);
  ln->vectorize(inner);
}

static std::shared_ptr<TEWrapper> wrapTECompute(
    std::shared_ptr<TEWrapper> wrap,
    Tensor out,
    std::vector<CodeGen::BufferArg> args,
    int width = kVectorWidth) {
  LoopNest ln({out});
  optimizePointwise(&ln, out, width);
  ln.prepareForCodegen();
```
- **EN**: This chunk defines `wrapTECompute`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `wrapTECompute`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 49-64
```cpp
  StmtPtr s = ln.root_stmt();
  s = IRSimplifier::simplify(s);
  args.insert(args.begin(), out);
  auto cg = std::make_unique<LLVMCodeGen>(s, args);
  cg->cleanup_memory();
  wrap->update(std::move(cg));
  return wrap;
}

static std::shared_ptr<TEWrapper> wrapTECompute(
    std::shared_ptr<TEWrapper> wrap,
    LoopNest* ln,
    std::vector<CodeGen::BufferArg> args) {
  auto cg = std::make_unique<LLVMCodeGen>(ln->root_stmt(), args);
  wrap->update(std::move(cg));
  return wrap;
```
- **EN**: This chunk defines `wrapTECompute`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `wrapTECompute`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 65-79
```cpp
}

#else

void TEWrapper::call(const std::vector<void*>& args) {
  DCHECK(0 && "Invalid call");
}

static std::shared_ptr<TEWrapper> wrapTECompute(
    std::shared_ptr<TEWrapper> wrap,
    const Tensor& out,
    const std::vector<CodeGen::BufferArg>& args,
    int width = kVectorWidth) {
  return wrap;
}
```
- **EN**: This chunk defines `wrapTECompute`, which implements a focused step in the TorchScript execution pipeline. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `wrapTECompute`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 81-95
```cpp
static std::shared_ptr<TEWrapper> wrapTECompute(
    std::shared_ptr<TEWrapper> wrap,
    LoopNest* ln,
    const std::vector<CodeGen::BufferArg>& args) {
  return wrap;
}

#endif

namespace {

std::mutex& getNNCCacheMutex() {
  static std::mutex nncCacheMutex;
  return nncCacheMutex;
}
```
- **EN**: This chunk defines `getNNCCacheMutex`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getNNCCacheMutex`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 97-109
```cpp
c10::FastMap<NodeKind, std::shared_ptr<TEWrapper>>& getNNCCache() {
  static c10::FastMap<NodeKind, std::shared_ptr<TEWrapper>> nncCache;
  return nncCache;
}

std::shared_ptr<TEWrapper> lookupNNCCache(NodeKind kind) {
  std::lock_guard<std::mutex> lock(getNNCCacheMutex());
  auto it = getNNCCache().find(kind);
  if (it != getNNCCache().end()) {
    return it->second;
  }
  return nullptr;
}
```
- **EN**: This chunk defines `lock`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `lock`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 111-123
```cpp
void updateNNCCache(NodeKind kind, std::shared_ptr<TEWrapper> code) {
  std::lock_guard<std::mutex> lock(getNNCCacheMutex());
  getNNCCache()[kind] = std::move(code);
}

} // namespace

std::shared_ptr<TEWrapper> createDiv() {
  auto wrap = lookupNNCCache(aten::div);
  if (wrap) {
    return wrap;
  }
  wrap = std::make_shared<TEWrapper>();
```
- **EN**: This chunk defines `createDiv`, which constructs derived state from the current inputs and invariants. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `createDiv`，其作用是根据当前输入和约束构建派生状态。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 125-136
```cpp
  auto dim = VarHandle("dim", kInt);
  auto mode = VarHandle("mode", kInt);
  BufHandle A("A", {dim}, kFloat);
  BufHandle B("B", {dim}, kFloat);

  using axis = const VarHandle&;
  Tensor C = Compute("C", {dim}, [&](axis x) {
    auto true_div_result = A.load(x) / B.load(x);

    auto mode_default = IntImm::make(0);
    auto mode_trunc = IntImm::make(1);
    auto mode_floor = IntImm::make(2);
```
- **EN**: This chunk continues `createDiv` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `createDiv`，进一步展开其内部控制流或数据流转。

### Lines 138-152
```cpp
    // this is a glorified ternary choice operator train
    return CompareSelect::make(
        mode,
        mode_default,
        true_div_result,
        CompareSelect::make(
            mode,
            mode_trunc,
            trunc(true_div_result),
            floor(true_div_result),
            kEQ),
        kEQ);
  });

  wrap = wrapTECompute(wrap, C, {A, B, mode, dim});
```
- **EN**: This chunk continues `createDiv` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `createDiv`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 154-169
```cpp
  updateNNCCache(aten::div, wrap);
  return wrap;
}

std::shared_ptr<TEWrapper> createLogit() {
  auto wrap = lookupNNCCache(aten::logit);
  if (wrap) {
    return wrap;
  }
  wrap = std::make_shared<TEWrapper>();
  auto N = VarHandle("N", kInt);
  auto C = VarHandle("C", kFloat);
  BufHandle A("A", {N}, kFloat);
  Tensor B = Compute("B", {N}, [&](const VarHandle& i) {
    auto A_elem = [&]() {
      auto elem = A.load(i);
```
- **EN**: This chunk defines `createLogit`, which constructs derived state from the current inputs and invariants. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `createLogit`，其作用是根据当前输入和约束构建派生状态。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 170-181
```cpp
      auto one = FloatImm::make(1.0f);
      const auto& min = C;
      auto max = one - C;
      elem = CompareSelect::make(elem, min, min, elem, kLT);
      return CompareSelect::make(elem, max, max, elem, kGT);
    }();
    return log_vml(A_elem / (FloatImm::make(1.0f) - A_elem));
  });
  wrap = wrapTECompute(wrap, B, {A, N, C});
  updateNNCCache(aten::logit, wrap);
  return wrap;
}
```
- **EN**: This chunk continues `createLogit` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `createLogit`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 183-198
```cpp
std::shared_ptr<TEWrapper> createRelu() {
  auto wrap = lookupNNCCache(aten::relu);
  if (wrap) {
    return wrap;
  }
  wrap = std::make_shared<TEWrapper>();
  auto N = VarHandle("N", kInt);
  BufHandle A("A", {N}, kFloat);
  Tensor B = Compute("B", {N}, [&](const VarHandle& i) {
    auto zero = FloatImm::make(0.f);
    auto a = A.load(i);
    return CompareSelect::make(a, zero, zero, a, kLT);
  });
  wrap = wrapTECompute(wrap, B, {A, N});
  updateNNCCache(aten::relu, wrap);
  return wrap;
```
- **EN**: This chunk defines `createRelu`, which constructs derived state from the current inputs and invariants. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `createRelu`，其作用是根据当前输入和约束构建派生状态。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 199-214
```cpp
}

std::shared_ptr<TEWrapper> createTanh() {
  auto wrap = lookupNNCCache(aten::tanh);
  if (wrap) {
    return wrap;
  }
  wrap = std::make_shared<TEWrapper>();
  auto N = VarHandle("N", kInt);
  BufHandle A("A", {N}, kFloat);
  Tensor B = Compute("B", {N}, [&](const VarHandle& i) {
    auto a = A.load(i);
    return fast_tanh(a);
  });
  wrap = wrapTECompute(wrap, B, {A, N});
  updateNNCCache(aten::tanh, wrap);
```
- **EN**: This chunk defines `createTanh`, which constructs derived state from the current inputs and invariants. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `createTanh`，其作用是根据当前输入和约束构建派生状态。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 215-230
```cpp
  return wrap;
}

std::shared_ptr<TEWrapper> createSigmoid() {
  auto wrap = lookupNNCCache(aten::sigmoid);
  if (wrap) {
    return wrap;
  }
  wrap = std::make_shared<TEWrapper>();
  auto N = VarHandle("N", kInt);
  BufHandle A("A", {N}, kFloat);
  Tensor B = Compute(
      "B", {N}, [&](const VarHandle& i) { return fast_sigmoid(A.load(i)); });
  wrap = wrapTECompute(wrap, B, {A, N});
  updateNNCCache(aten::sigmoid, wrap);
  return wrap;
```
- **EN**: This chunk defines `createSigmoid`, which constructs derived state from the current inputs and invariants. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `createSigmoid`，其作用是根据当前输入和约束构建派生状态。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 231-242
```cpp
}

std::shared_ptr<TEWrapper> createClamp() {
  static auto clamp_symbol = c10::Symbol::fromQualString("aten::clamp");
  auto wrap = lookupNNCCache(clamp_symbol);
  if (wrap) {
    return wrap;
  }
  wrap = std::make_shared<TEWrapper>();
  auto N = VarHandle("N", kInt);
  auto min_handle = VarHandle("min", kFloat);
  auto max_handle = VarHandle("max", kFloat);
```
- **EN**: This chunk defines `createClamp`, which constructs derived state from the current inputs and invariants. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `createClamp`，其作用是根据当前输入和约束构建派生状态。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 244-259
```cpp
  BufHandle A("A", {N}, kFloat);
  Tensor result = Compute("aten_clamp", {N}, [&](const VarHandle& i) {
    auto a = A.load(i);
    return tensorexpr::clamp(min_handle, max_handle, a);
  });
  wrap = wrapTECompute(wrap, result, {A, min_handle, max_handle, N});
  updateNNCCache(clamp_symbol, wrap);
  return wrap;
}

std::shared_ptr<TEWrapper> createClampNanToNum() {
  static auto symbol =
      c10::Symbol::fromQualString("static_runtime::clamp_nan_to_num");
  auto wrap = lookupNNCCache(symbol);
  if (wrap) {
    return wrap;
```
- **EN**: This chunk defines `createClampNanToNum`, which constructs derived state from the current inputs and invariants. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `createClampNanToNum`，其作用是根据当前输入和约束构建派生状态。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 260-275
```cpp
  }
  wrap = std::make_shared<TEWrapper>();
  auto N = VarHandle("N", kInt);
  auto min_handle = VarHandle("min", kFloat);
  auto max_handle = VarHandle("max", kFloat);
  auto nan_replace_val = VarHandle("nan_replace_val", kFloat);

  BufHandle A("A", {N}, kFloat);
  Tensor result = Compute("aten_clamp", {N}, [&](const VarHandle& i) {
    auto a = A.load(i);
    auto clamp = tensorexpr::clamp(min_handle, max_handle, a);
    auto is_nan = tensorexpr::isnan(clamp);
    auto nans_replaced =
        tensorexpr::CompareSelect::make(is_nan, 1, nan_replace_val, clamp, kEQ);
    return nans_replaced;
  });
```
- **EN**: This chunk continues `createClampNanToNum` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `createClampNanToNum`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 276-291
```cpp
  wrap = wrapTECompute(
      wrap, result, {A, min_handle, max_handle, nan_replace_val, N});
  updateNNCCache(symbol, wrap);
  return wrap;
}

std::shared_ptr<TEWrapper> createSignedLog1p() {
  static auto signed_log1p_symbol =
      c10::Symbol::fromQualString("static_runtime::signed_log1p");
  auto wrap = lookupNNCCache(signed_log1p_symbol);
  if (wrap) {
    return wrap;
  }
  wrap = std::make_shared<TEWrapper>();
  auto N = VarHandle("N", kInt);
  BufHandle A("A", {N}, kFloat);
```
- **EN**: This chunk defines `createSignedLog1p`, which constructs derived state from the current inputs and invariants. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `createSignedLog1p`，其作用是根据当前输入和约束构建派生状态。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 292-307
```cpp
  Tensor abs_result = Compute("aten_abs", {N}, [&](const VarHandle& i) {
    return tensorexpr::abs(A.load(i));
  });
  Tensor log1p_result = Compute("aten_log1p", {N}, [&](const VarHandle& i) {
    return log1p(abs_result.load(i));
  });
  Tensor sign = computeSign({A}, {N});
  Tensor output = Compute("aten_mul", {N}, [&](const VarHandle& i) {
    return sign.load(i) * log1p_result.load(i);
  });
  LoopNest ln({output}, {abs_result, log1p_result, sign, output});
  GRAPH_DEBUG("Original stmt: ", *ln.root_stmt());
  ln.inlineIntermediateBufs(true);
  ln.prepareForCodegen();
  ln.simplify();
  ln.vectorizeInnerLoops();
```
- **EN**: This chunk continues `createSignedLog1p` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `createSignedLog1p`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 308-315
```cpp
  ln.simplify();
  GRAPH_DEBUG("Final stmt: ", *ln.root_stmt());
  wrap = wrapTECompute(wrap, &ln, {output, A, N});
  updateNNCCache(signed_log1p_symbol, wrap);
  return wrap;
}

} // namespace torch::jit
```
- **EN**: This chunk continues `createSignedLog1p` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `createSignedLog1p`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **TorchScript runtime**
  - EN: Implements TorchScript runtime execution, specialization, operator dispatch, and static-runtime support.
  - CN: 实现 TorchScript 运行时执行、特化、算子分发以及静态运行时支持。
- **update**
  - EN: `update` is a central symbol declared or implemented in this file.
  - CN: `update` 是本文件声明或实现的核心符号。
- **call**
  - EN: `call` is a central symbol declared or implemented in this file.
  - CN: `call` 是本文件声明或实现的核心符号。
- **Static runtime**
  - EN: Optimizes execution around precomputed layouts, generated operators, and lightweight runtime state.
  - CN: 围绕预计算布局、生成算子以及轻量运行时状态来优化执行。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
- **Tensor Expression IR**
  - EN: Represents lowered tensor computations in an optimization-friendly intermediate form.
  - CN: 以便于优化的中间表示来表达降级后的张量计算。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/runtime/static/te_wrapper.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/runtime/static/impl.h`, `torch/csrc/jit/tensorexpr/expr.h`, `torch/csrc/jit/tensorexpr/operators/misc.h`, `torch/csrc/jit/tensorexpr/operators/operators.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `ATen/CPUFunctions.h`
- **Standard library / 标准库**: `utility`
- **Primary symbols in this file / 本文件核心符号**: `update`, `call`, `optimizePointwise`, `wrapTECompute`, `getNNCCacheMutex`, `lookupNNCCache`, `lock`, `updateNNCCache`
