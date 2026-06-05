# kernel.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/kernel.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-26
```cpp
#include <torch/csrc/jit/tensorexpr/kernel.h>

#include <ATen/ExpandUtils.h>
#include <ATen/Parallel.h>
#include <ATen/TensorGeometry.h>
#include <c10/core/ScalarTypeToTypeMeta.h>
#include <c10/util/irange.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/graph_rewrite_helper.h>
#include <torch/csrc/jit/passes/mkldnn_rewrite.h>
#include <torch/csrc/jit/passes/symbolic_shape_runtime_fusion.h>
#include <torch/csrc/jit/tensorexpr/analysis.h>
#include <torch/csrc/jit/tensorexpr/expr.h>
#include <torch/csrc/jit/tensorexpr/graph_opt.h>
#include <torch/csrc/jit/tensorexpr/ir_printer.h>
#include <torch/csrc/jit/tensorexpr/ir_simplifier.h>
#include <torch/csrc/jit/tensorexpr/loopnest.h>
#include <torch/csrc/jit/tensorexpr/loopnest_randomization.h>
#include <torch/csrc/jit/tensorexpr/operators/operators.h>

#include <utility>

using namespace torch::jit;
using namespace torch::jit::tensorexpr;

namespace torch::jit::tensorexpr {
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/kernel.h, torch/csrc/jit/jit_log.h, torch/csrc/jit/passes/graph_rewrite_helper.h, and 10 more; ATen/c10 facilities such as ATen/ExpandUtils.h, ATen/Parallel.h, ATen/TensorGeometry.h, and 2 more; standard-library headers such as utility. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/kernel.h、torch/csrc/jit/jit_log.h、torch/csrc/jit/passes/graph_rewrite_helper.h 等共 13 项；ATen/c10 基础设施，如 ATen/ExpandUtils.h、ATen/Parallel.h、ATen/TensorGeometry.h 等共 5 项；标准库头文件，如 utility。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。

### Lines 28-54
```cpp
std::string buildErrorMessage(const std::string& s) {
  static const std::string generic_error_message =
      "This error occurred in the fuser. You can turn off the fuser with "
      "torch.jit.enable_fusion(False).";
  if (s.empty()) {
    return generic_error_message;
  }
  if (s.back() == '.') {
    return s + " " + generic_error_message;
  }
  return s + ". " + generic_error_message;
}

static int te_cuda_pointwise_loop_levels = -1;
static int te_cuda_pointwise_block_count = -1;
static int te_cuda_pointwise_block_size = -1;
static bool fallback_allowed = false;
static bool te_generate_block_code = false;
static bool te_must_use_llvm_on_cpu = true;
static bool cat_wo_conditionals = true;
static bool opt_conditionals = false;

bool setFallbackAllowed(bool value) {
  bool old_value = fallback_allowed;
  fallback_allowed = value;
  return old_value;
}
```
- **EN**: This chunk defines `setFallbackAllowed`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `setFallbackAllowed`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 56-90
```cpp
bool fallbackAllowed() {
  static const auto enable_opt =
      c10::utils::get_env("PYTORCH_TENSOREXPR_FALLBACK");
  if (!enable_opt.has_value()) {
    return fallback_allowed;
  }
  if (enable_opt == "0") {
    return false;
  }
  return true;
}

static bool fallbackEnforced() {
  static const auto enable_opt =
      c10::utils::get_env("PYTORCH_TENSOREXPR_FALLBACK");
  if (tensorexpr::getTEGenerateBlockCode()) {
    return false;
  }
  if (!enable_opt.has_value()) {
    return fallback_allowed;
  }
  if (enable_opt == "2") {
    return true;
  }
  return false;
}

static int64_t randomTransformsRequested() {
  const auto enable_opt =
      c10::utils::get_env("PYTORCH_TENSOREXPR_RANDOM_TRANSFORM_SEED");
  if (!enable_opt.has_value()) {
    return 0;
  }
  return std::stoi(enable_opt.value());
}
```
- **EN**: This chunk defines `randomTransformsRequested`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `randomTransformsRequested`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 92-124
```cpp
#ifdef TORCH_ENABLE_LLVM
static bool dontUseLLVMFlag() {
  static const auto enable_opt =
      c10::utils::get_env("PYTORCH_TENSOREXPR_DONT_USE_LLVM");
  if (!enable_opt) {
    return false;
  }
  return enable_opt == "1";
}
#endif

int& getTECudaPointwiseLoopLevels() {
  return te_cuda_pointwise_loop_levels;
}

int& getTECudaPointwiseBlockCount() {
  return te_cuda_pointwise_block_count;
}

int& getTECudaPointwiseBlockSize() {
  return te_cuda_pointwise_block_size;
}

// TODO: Remove this global var
// Ideally Block code gen should be decided
// based on device type in tensor.
bool& getTEGenerateBlockCode() {
  return te_generate_block_code;
}

bool& getTEMustUseLLVMOnCPU() {
  return te_must_use_llvm_on_cpu;
}
```
- **EN**: This chunk defines `getTEMustUseLLVMOnCPU`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getTEMustUseLLVMOnCPU`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 126-161
```cpp
bool& getCatWoConditionals() {
  return cat_wo_conditionals;
}

bool& getOptConditionals() {
  return opt_conditionals;
}

std::optional<at::Device> pickDeviceType(
    const at::ArrayRef<torch::jit::Value*>& inputs) {
  std::optional<at::Device> device = std::nullopt;
  for (auto const& input : inputs) {
    auto tt = input->type()->cast<TensorType>();
    if (tt && tt->device()) {
      if (device && *device != *tt->device()) {
        return std::nullopt;
      }
      device = *tt->device();
    }
  }
  return device;
}

static std::optional<at::Device> pickDeviceType(
    const std::shared_ptr<Graph>& graph) {
  std::optional<at::Device> device = std::nullopt;
  for (auto const& node : graph->nodes()) {
    for (auto const& input : node->inputs()) {
      if (auto tt = input->type()->cast<TensorType>()) {
        if (auto inputDevice = tt->device()) {
          TORCH_INTERNAL_ASSERT(
              !device || *device == *inputDevice,
              buildErrorMessage(
                  "Different devices specified for inputs to the fuser."));
          device = inputDevice;
        }
```
- **EN**: This chunk defines `pickDeviceType`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `pickDeviceType`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 162-188
```cpp
      }
    }
  }
  for (auto const& input : graph->inputs()) {
    if (auto tt = input->type()->cast<TensorType>()) {
      if (auto inputDevice = tt->device()) {
        TORCH_INTERNAL_ASSERT(
            !device || *device == *inputDevice,
            buildErrorMessage(
                "Different devices specified for inputs to the fuser."));
        device = inputDevice;
      }
    }
  }
  if (!device) {
    // By default assume the device is CPU
    device = at::kCPU;
  }
  return device;
}

// If v is a Tensor with concretely-known sizes and dtype, return them, else
// nullopt.
static std::optional<TensorInfo> getTensorInfoJit(torch::jit::Value* v) {
  auto const& it = v->type()->cast<TensorType>();

  c10::ScalarType dtype = c10::ScalarType::Float;
```
- **EN**: This chunk defines `getTensorInfoJit`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getTensorInfoJit`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 190-214
```cpp
  if (!it) {
    return std::nullopt;
  }
  if (!it->isComplete()) {
    return std::nullopt;
  }
  if (it->scalarType()) {
    // TODO: ideally we should be strict here and return nullopt if the dtype is
    // absent in the JIT IR. We're assuming a default Float dtype for now, until
    // dtype propagation is implemented.
    dtype = *it->scalarType();
  }
  auto concrete_sizes = it->sizes().concrete_sizes();
  if (!concrete_sizes) {
    return std::nullopt;
  }
  return TensorInfo{*concrete_sizes, dtype};
}
static std::vector<int64_t> _pair_int(const IValue& v) {
  if (v.isIntList()) {
    return v.toIntVector();
  } else {
    return {v.toInt(), v.toInt()};
  }
}
```
- **EN**: This chunk defines `_pair_int`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `_pair_int`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 216-238
```cpp
bool isContiguous(const torch::jit::Value* v, at::MemoryFormat memory_format) {
  auto const& tt = v->type()->cast<TensorType>();
  if (!tt) {
    return false;
  }
  if (!tt->isComplete()) {
    return false;
  }
  auto const& sizes = tt->sizes().concrete_sizes();
  auto const& strides = tt->strides().concrete_sizes();
  if (!sizes || !strides) {
    return false;
  }

  // Check dimension size first
  auto ndims = (*sizes).size();
  if ((memory_format == at::MemoryFormat::ChannelsLast && ndims != 4) ||
      (memory_format == at::MemoryFormat::ChannelsLast3d && ndims != 5)) {
    return false;
  }

  return *strides == TensorType::contiguousStridesOf(*sizes, memory_format);
}
```
- **EN**: This chunk defines `isContiguous`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `isContiguous`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 240-274
```cpp
static size_t get_conv_groups_index(const torch::jit::Node* node) {
  switch (node->kind()) {
    case aten::conv2d:
      return 6;
    case aten::_convolution:
      return 8;
    default:
      TORCH_CHECK(
          false,
          "mkldnnPrepackedConvIsSupportedJit expects node kind to be conv2d or _convolution but got ",
          node->kind());
  }
}

// The fuser only supports conv2d with very specific properties:
// - Static shapes: 4-d input and filter, 1-d bias.
// - Constant strides/padding/dilation/groups
// - Equal padding and strides, dilation == 1.
// - Depthwise (groups == in_channels == out_channels)
// - 3x3 kernel
bool conv2dIsSupportedJit(const torch::jit::Node* node) {
  auto const& input = getTensorInfoJit(node->input(0));
  auto const& weight = getTensorInfoJit(node->input(1));
  auto const& bias = getTensorInfoJit(node->input(2));
  auto const& stride = toIValue(node->input(3));
  auto const& pad = toIValue(node->input(4));
  auto const& dilation = toIValue(node->input(5));
  size_t groups_index = get_conv_groups_index(node);
  auto const& groups = toIValue(node->input(groups_index));

  // Everything should be statically known.
  if (!input || !weight || !bias || !stride || !pad || !dilation || !groups) {
    GRAPH_DEBUG("some params aren't static");
    return false;
  }
```
- **EN**: This chunk defines `conv2dIsSupportedJit`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `conv2dIsSupportedJit`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 276-308
```cpp
  // All inputs should be contiguous so no transposition is required.
  if (!isContiguous(node->input(0)) || !isContiguous(node->input(1)) ||
      !isContiguous(node->input(2))) {
    GRAPH_DEBUG("conv2dIsSupported: some inputs are not contiguous");
    return false;
  }

  return conv2dIsSupported(
      *input,
      *weight,
      *bias,
      _pair_int(*stride),
      _pair_int(*pad),
      _pair_int(*dilation),
      groups->toInt());
}

bool mkldnnPrepackedConvIsSupportedJit(const torch::jit::Node* node) {
#if AT_MKLDNN_ENABLED()
  auto const& input = getTensorInfoJit(node->input(0));
  auto const& weight = getTensorInfoJit(node->input(1));
  auto const& stride = toIValue(node->input(3));
  auto const& pad = toIValue(node->input(4));
  auto const& dilation = toIValue(node->input(5));
  size_t groups_index = get_conv_groups_index(node);
  auto const& groups = toIValue(node->input(groups_index));

  // Everything should be statically known (bias could be NoneType =
  // prim::Constant()).
  if (!input || !weight || !stride || !pad || !dilation || !groups) {
    GRAPH_DEBUG("some params aren't static");
    return false;
  }
```
- **EN**: This chunk defines `mkldnnPrepackedConvIsSupportedJit`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `mkldnnPrepackedConvIsSupportedJit`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 310-341
```cpp
  // Weights and bias should be Constant when using mkldnn backend
  if (node->input(1)->node()->kind() != prim::Constant ||
      node->input(2)->node()->kind() != prim::Constant) {
    GRAPH_DEBUG(
        "mkldnnPrepackedConvIsSupported: weight or bias is not Constant");
    return false;
  }

  // Input and weight should be NHWC contiguous.
  if (!(isContiguous(node->input(0), at::MemoryFormat::ChannelsLast) &&
        isContiguous(node->input(1), at::MemoryFormat::ChannelsLast))) {
    GRAPH_DEBUG(
        "mkldnnPrepackedConvIsSupported: input or weight is not ChannelsLast contiguous");
    return false;
  }

  return mkldnnPrepackedConvIsSupported(
      *input,
      *weight,
      _pair_int(*stride),
      _pair_int(*pad),
      _pair_int(*dilation),
      groups->toInt());
#else
  return false;
#endif
}

bool isConv2d(const Node* node) {
  if (node->kind() != aten::_convolution) {
    return false;
  }
```
- **EN**: This chunk defines `isConv2d`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `isConv2d`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 343-372
```cpp
  auto const& stride = toIValue(node->input(3));
  auto const& pad = toIValue(node->input(4));
  auto const& dilation = toIValue(node->input(5));
  auto const& transposed = toIValue(node->input(6));
  auto const& output_padding = toIValue(node->input(7));

  if (!stride || !pad || !dilation || !transposed || !output_padding) {
    GRAPH_DEBUG("some params aren't static");
    return false;
  }

  if (stride.value().toIntList().size() != 2 ||
      pad.value().toIntList().size() != 2 ||
      dilation.value().toIntList().size() != 2 ||
      output_padding.value().toIntList().size() != 2) {
    GRAPH_DEBUG("Conv not 2d");
    return false;
  }

  if (transposed.value().toBool()) {
    GRAPH_DEBUG("transposed Conv");
    return false;
  }
  return true;
}

// The fuser currently only supports matmul of 2D x 2D matrices
bool matmulIsSupported(const torch::jit::Node* node) {
  auto const& input0 = getTensorInfoJit(node->input(0));
  auto const& input1 = getTensorInfoJit(node->input(1));
```
- **EN**: This chunk defines `matmulIsSupported`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `matmulIsSupported`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 374-399
```cpp
  // Everything should be statically known.
  if (!input0 || !input1) {
    GRAPH_DEBUG("matmulIsSupported: Input shapes aren't static");
    return false;
  }

  // Proper ndim for tensor inputs.
  if (input0->dims.size() != 2 || input1->dims.size() != 2) {
    GRAPH_DEBUG("matmulIsSupported: Unsupported input sizes");
    return false;
  }

  // Inputs should be contiguous, or the TE will needlessly transpose them.
  if (!isContiguous(node->input(0)) || !isContiguous(node->input(1))) {
    GRAPH_DEBUG("matmulIsSupported: Input shapes are not contiguous");
    return false;
  }

  return true;
}

} // namespace torch::jit::tensorexpr

static at::ScalarType tensorType(const BufPtr& b) {
  return static_cast<at::ScalarType>(b->dtype().scalar_type());
}
```
- **EN**: This chunk defines `tensorType`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `tensorType`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 401-425
```cpp
ExprHandle TensorExprKernel::constant(const torch::jit::Value* v) {
  if (v->node()->kind() == prim::Constant) {
    auto val = toIValue(v).value();
    if (val.isDouble()) {
      return DoubleImm::make(val.toDouble());
    } else if (val.isInt()) {
      return LongImm::make(val.toInt());
    } else if (val.isBool()) {
      return BoolImm::make(val.toBool());
    } else if (val.isNone()) {
      // This is just a placeholder so we don't throw.  None-handling
      // is operator-specific and should be handled properly in
      // the operator-specific lowering code.
      return IntImm::make(0);
    } else {
      throw unsupported_dtype();
    }
  }

  if (!scalars_.count(v)) {
    throw malformed_input("no scalar in Constant");
  }

  return scalars_.at(v);
}
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 427-462
```cpp
ArgValue TensorExprKernel::toArg(const torch::jit::Value* v) const {
  auto vi = scalars_.find(v);
  if (vi != scalars_.end()) {
    return VarHandle(vi->second);
  }
  auto ti = bufs_.find(v);
  if (ti != bufs_.end()) {
    return BufHandle(ti->second);
  }
  if (v->node()->kind() == prim::ListConstruct) {
    std::vector<ArgValue> vec;
    for (auto el : v->node()->inputs()) {
      vec.push_back(toArg(el));
    }
    if (vec.empty()) {
      return BufList(); // Return arbitrarily typed vector
    } else if (std::get_if<BufHandle>(&vec[0])) {
      return convertVecArgValue<BufHandle>(vec);
    } else if (std::get_if<int64_t>(&vec[0])) {
      return convertVecArgValue<int64_t>(vec);
    }
    throw unsupported_dtype();
  }
  if (v->node()->kind() == prim::Constant) {
    auto val = toIValue(v).value();
    if (val.isDouble()) {
      return val.toDouble();
    } else if (val.isInt()) {
      return val.toInt();
    } else if (val.isBool()) {
      return val.toBool();
    } else if (val.isNone()) {
      // This is just a placeholder so we don't throw.  None-handling
      // is operator-specific and should be handled properly in
      // the operator-specific lowering code.
      return ArgNone();
```
- **EN**: This chunk defines `unsupported_dtype`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `unsupported_dtype`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 463-496
```cpp
    } else if (val.isIntList()) {
      return val.toIntVector();
    } else if (val.isDoubleList()) {
      return val.toDoubleVector();
    } else if (val.isString()) {
      return val.toStringRef();
    } else {
      throw unsupported_dtype(val.type()->str());
    }
  }

  if (!scalars_.count(v)) {
    throw malformed_input("no scalar in Constant");
  }
  return scalars_.at(v);
}

ExprHandle TensorExprKernel::getVarForShape(const c10::ShapeSymbol& ss) {
  if (ss.is_static()) {
    return LongImm::make(ss.static_size());
  }
  auto value = ss.value();
  auto it = shapeSymbolToVar_.find(value);
  if (it == shapeSymbolToVar_.end()) {
    VarHandle var("ss" + std::to_string(-value), kLong);
    shapeSymbolToVar_.emplace(value, var);
#if C10_RETURN_MOVE_IF_OLD_COMPILER
    return std::move(var);
#else
    return var;
#endif
  }
  return it->second;
}
```
- **EN**: This chunk defines `var`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `var`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 498-521
```cpp
std::vector<ExprHandle> TensorExprKernel::sizesFromSymbolicShape(
    const c10::SymbolicShape& shape) {
  std::vector<ExprHandle> dims;
  auto maybe_rank = shape.rank();
  TORCH_INTERNAL_ASSERT(maybe_rank);
  auto rank = *maybe_rank;
  for (const auto i : c10::irange(rank)) {
    dims.push_back(getVarForShape(shape[i]));
  }
  return dims;
}

std::vector<ExprHandle> TensorExprKernel::sizesForValue(
    const torch::jit::Value* v) {
  if (known_sizes_.count(v)) {
    return known_sizes_.at(v);
  }

  // If the shape is present in the type info, just extract it from here. No
  // need to infer it.
  if (v->type()->kind() == TypeKind::TensorType) {
    auto tt = v->type()->cast<TensorType>();
    return sizesFromSymbolicShape(tt->symbolic_sizes());
  }
```
- **EN**: This chunk defines `sizesForValue`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `sizesForValue`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 523-553
```cpp
  if (v->type()->isSubtypeOf(*FloatType::get()) ||
      v->type()->isSubtypeOf(*BoolType::get()) ||
      v->type()->isSubtypeOf(*IntType::get())) {
    return {};
  }
  if (v->type()->isSubtypeOf(*NoneType::get())) {
    return {};
  }
  GRAPH_DEBUG("Unknown sizes for the node: ", *v->node());
  GRAPH_DEBUG("Full fusion group graph:\n", *v->node()->owningGraph());
  std::string msg = std::string("Unhandled node kind (in sizesForValue): ") +
      v->node()->kind().toQualString();
  throw malformed_input(msg);
}

static std::optional<ScalarType> findDtypeForValue(const torch::jit::Value* v) {
  if (v->type()->kind() == TypeKind::TensorType) {
    auto tt = v->type()->cast<TensorType>();
    if (tt->scalarType()) {
      return static_cast<ScalarType>(*tt->scalarType());
    }
  }
  return tryScalarTypeFromJitType(*v->type());
}

static bool constZeroDimTensorAsScalarArg(
    const Value* v,
    std::vector<ArgValue>& args) {
  if (v->node()->kind() != prim::Constant || !v->type()->cast<TensorType>()) {
    return false;
  }
```
- **EN**: This chunk defines `constZeroDimTensorAsScalarArg`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `constZeroDimTensorAsScalarArg`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 555-582
```cpp
  const auto t = toIValue(v)->toTensor();
  if (!t.sizes().empty()) {
    return false;
  }

  c10::ScalarType dtype = c10::typeMetaToScalarType(t.dtype());
  switch (dtype) {
    case ScalarType::Float:
      args.emplace_back(t.item().toFloat());
      return true;
    case ScalarType::Long:
      args.emplace_back(t.item().toLong());
      return true;
    default:
      std::stringstream ss;
      ss << "Unsupported tensor dtype:" << dtype
         << " for converting constant 0-dim Tensor to scalar" << '\n';
      throw unsupported_dtype(ss.str());
  }
}

Tensor TensorExprKernel::computeValue(const torch::jit::Value* v) {
  auto inputs = v->node()->inputs();
  auto op = v->node()->kind();

  if (op == aten::rand_like) {
    hasRandom_ = true;
  }
```
- **EN**: This chunk defines `computeValue`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `computeValue`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 584-619
```cpp
  auto outputType = findDtypeForValue(v);
  std::vector<ExprHandle> outputShape = sizesForValue(v);
  std::vector<ExprHandle> outputStrides = {};
  if (memory_layout_policy_ == MemoryLayoutPolicy::kChannelsLastNdContiguous) {
    outputStrides =
        c10::fmap<ExprHandle>(make_channels_last_strides(outputShape));
  } else {
    // Default
    outputStrides = c10::fmap<ExprHandle>(make_contiguous_strides(outputShape));
  }

  std::vector<ArgValue> argInputs;
  if (op == prim::ConstantChunk) {
    auto const& n = v->node();
    argInputs.emplace_back(toArg(inputs[0]));
    argInputs.emplace_back(static_cast<int64_t>(v->offset()));
    argInputs.emplace_back(n->i(attr::dim));
    argInputs.emplace_back(n->i(attr::chunks));
  } else if (op == aten::to) {
    argInputs.emplace_back(toArg(inputs[0]));
  } else if (op == aten::quantize_per_tensor) {
    argInputs.emplace_back(toArg(inputs[0]));
    if (!constZeroDimTensorAsScalarArg(inputs[1], argInputs)) {
      argInputs.emplace_back(toArg(inputs[1]));
    }
    if (!constZeroDimTensorAsScalarArg(inputs[2], argInputs)) {
      argInputs.emplace_back(toArg(inputs[2]));
    }
    argInputs.emplace_back(toArg(inputs[3]));
  } else if (op == aten::conv2d) {
    for (auto inp : inputs) {
      argInputs.emplace_back(toArg(inp));
    }
    // handle optional bias
    if (std::get_if<ArgNone>(&argInputs[2])) {
      Dtype dtype = outputType ? Dtype(*outputType) : kFloat;
```
- **EN**: This chunk continues `computeValue` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `computeValue`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 620-654
```cpp
      std::vector<ExprHandle> biasShape;
      biasShape.push_back(outputShape[1]);
      auto bias_tensor = at::zeros({outputShape[1].AsNode<LongImm>()->value()});
      unpacked_constant_tensors_.push_back(bias_tensor);
      BufPtr buf = alloc<Buf>(
          "conv2d_bias_opt_" + sanitizeName(v->debugName()),
          ExprHandleVectorToExprVector(biasShape),
          dtype);
      constants_.push_back({buf, bias_tensor.data_ptr()});
      argInputs[2] = BufHandle(buf);
    }
  } else {
    for (auto inp : inputs) {
      argInputs.emplace_back(toArg(inp));
    }
  }

  if (NNCLoweringFunction custom_lowering = getCustomLoweringFor(op)) {
    return custom_lowering(
        argInputs, outputShape, outputStrides, outputType, device_);
  }
  if (v->node()->maybeSchema()) {
    if (NNCLoweringFunction lowering =
            getStandardLoweringFor(c10::toString(v->node()->schema()))) {
      return lowering(
          argInputs, outputShape, outputStrides, outputType, device_);
    }
  }
  std::string msg = std::string("Unhandled node kind (in computeValue): ") +
      op.toQualString();
  if (v->node()->maybeSchema()) {
    msg += std::string("\nSchema: ") + c10::toString(v->node()->schema());
  }
  throw malformed_input(msg);
}
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 656-682
```cpp
// True if all the loops in this vector have equal bounds.
static bool loopBoundsAllEqual(const std::vector<ForPtr>& loops) {
  if (loops.size() <= 1) {
    return true;
  }
  const auto& start = loops.front()->start();
  const auto& stop = loops.front()->stop();
  for (size_t i = 1; i < loops.size(); ++i) {
    const auto& curr_start = loops[i]->start();
    const auto& curr_stop = loops[i]->stop();
    if (!exprEquals(start, curr_start) || !exprEquals(stop, curr_stop)) {
      return false;
    }
  }
  return true;
}

// Recursively fuse all the loops with matching bounds in `st`.  Stops fusing
// at any level containing non-loops or non-matching bounds.  The restriction
// on matching bounds exists to avoid inserting conditionals on the loop
// indices where none would be needed, which would significantly complicate
// vectorization.
static void fuseAllLoops(const StmtPtr& st) {
  auto block = to<tensorexpr::Block>(st);
  if (block == nullptr) {
    return;
  }
```
- **EN**: This chunk defines `fuseAllLoops`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `fuseAllLoops`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 684-714
```cpp
  std::vector<std::vector<ForPtr>> all_outer_loops;
  std::vector<ForPtr> outer_loops;
  for (const auto& stmt : *block) {
    auto loop = to<For>(stmt);
    auto hasReduction = !NodeFinder<ReduceOp>::find(stmt).empty();
    if (!loop || hasReduction) {
      all_outer_loops.push_back(outer_loops);
      outer_loops.clear();
    } else {
      outer_loops.push_back(loop);
    }
  }
  all_outer_loops.push_back(outer_loops);

  for (const auto& outer_loops : all_outer_loops) {
    if (outer_loops.empty()) {
      continue;
    }

    if (!loopBoundsAllEqual(outer_loops)) {
      continue;
    }

    ForPtr fusedLoop;
    if (!LoopNest::fuseLoops(outer_loops, &fusedLoop)) {
      continue;
    }

    fuseAllLoops(fusedLoop->body());
  }
}
```
- **EN**: This chunk continues `fuseAllLoops` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `fuseAllLoops`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 716-740
```cpp
// Compute the trip count of a loop if it is a constant.
static std::optional<int64_t> tripCount(const ForPtr& loop) {
  auto tc = IRSimplifier::simplify(
      cast<int64_t>(ExprHandle(loop->stop()) - ExprHandle(loop->start())));
  if (auto val = to<LongImm>(tc.node())) {
    return val->value();
  }
  return std::nullopt;
}

// Prune innermost loops until iterations satisfies a minimum grain size.
static void pruneByGrainSize(std::vector<ForPtr>& loops) {
  constexpr int64_t minGrainSize = 32768;
  int64_t grainSize = 1;
  for (int64_t i = loops.size(); i > 0; i--) {
    auto tc = tripCount(loops[i - 1]);
    if (!tc) {
      break;
    }
    grainSize *= *tc;
    if (grainSize < minGrainSize) {
      loops.pop_back();
    }
  }
}
```
- **EN**: This chunk defines `pruneByGrainSize`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `pruneByGrainSize`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 742-768
```cpp
// Retain enough outermost loops to fill the number of threads.
static void pruneByThreadCount(std::vector<ForPtr>& loops) {
  int64_t trips = 1;
  auto threads = at::get_num_threads();
  auto it = loops.begin();
  for (; it != loops.end(); it++) {
    if (trips >= threads) {
      break;
    }
    auto tc = tripCount(*it);
    if (!tc) {
      break;
    }
    trips *= *tc;
  }
  loops.erase(it, loops.end());
}

// Flatten and parallelize outer loops, subject to a minimum number of elements
// in the inner loop, and a maximum level of thread-level parallelism in the
// outer loops.
template <typename Bufs>
static void parallelizeOuterLoops(LoopNest& l, const Bufs& bufs) {
  for (auto const& buf : bufs) {
    auto loops = l.getLoopStmtsFor(buf);
    pruneByGrainSize(loops);
    pruneByThreadCount(loops);
```
- **EN**: This chunk defines `parallelizeOuterLoops`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `parallelizeOuterLoops`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 770-794
```cpp
    // There are no loops to parallelize; give up.
    if (loops.size() == 0) {
      continue;
    }
    // The loop nest contains a reduction; give up.
    auto reductions = NodeFinder<ReduceOp>::find(loops[0]);
    if (reductions.size() > 0) {
      continue;
    }
    // The loop nest has loop carried dependences; give up.
    if (LoopNest::hasLoopCarriedDependence(loops[0])) {
      continue;
    }
    // Try to flatten the outer loops and parallelize them if successful.
    ForPtr flattened = nullptr;
    if (loops.size() == 1) {
      flattened = loops[0];
    } else {
      LoopNest::flatten(loops, &flattened);
    }
    if (flattened) {
      flattened->set_parallel();
    }
  }
}
```
- **EN**: This chunk continues `parallelizeOuterLoops` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `parallelizeOuterLoops`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 796-821
```cpp
StmtPtr TensorExprKernel::transformLoops(BackendType backendType, StmtPtr st) {
  torch::jit::tensorexpr::LoopNest l(std::move(st), bufOutputs_);
  LoopNest::sanitizeNames(l.root_stmt());
  GRAPH_DEBUG("Original Stmt:\n", std::to_string(l.root_stmt()), '\n');
  int64_t random_tr_seed = randomTransformsRequested();
  if (random_tr_seed) {
    if (random_tr_seed == -1)
      random_tr_seed = std::time(nullptr);
    loopnestRandomization(random_tr_seed, l);
    GRAPH_DEBUG(
        "After random transform:\n", std::to_string(l.root_stmt()), "\n");
  }

  bool hasReduction = !NodeFinder<ReduceOp>::find(l.root_stmt()).empty();

  // For Block codegen we create a map of tensor dims before
  // inlining. Like GPU codegen we need to inline. But the order
  // where this analysis is run matters.
  auto block_analysis = std::make_unique<CreateBufferMap>();
  if (backendType == kBlockCodeGen) {
    // Run Block analysis to get multi dim buffer info
    auto root_stmt = l.root_stmt();
    root_stmt->accept(block_analysis.get());
  }
  l.simplify();
  GRAPH_DEBUG("after simplify", *l.root_stmt());
```
- **EN**: This chunk defines `l`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `l`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 823-850
```cpp
  // Inlining output & intermediate buffers can duplicate computation.
  // Duplicating work can slow down the program if it's not ameliorated in some
  // way, but we've empirically found that:
  // - On CPU, LLVM's CSE does a good job as long as you horizontally fuse
  //   output loops.
  // - On GPU, there's enough compute to hide the extra work, and inlining
  //   avoids synchronizing between kernels.
  l.inlineIntermediateBufs(/*allow_duplicated_work=*/true);
  GRAPH_DEBUG("after inline", *l.root_stmt());

  // Optimizing conditionals needs to be performed after inlining because
  // inlining wouldn't work once the loops are split. Also, it has to be
  // performed before loop fusion because loop fusion introduces cases where
  // multiple conditionals are in the same loop and this optimization does not
  // handle such cases yet.
  if (getOptConditionals()) {
    l.optimizeConditionals();
    GRAPH_DEBUG("after optimizing conditionals: ", *l.root_stmt());
  }

  // Fuse loops "horizontally".  This pass allows us to combine loops that
  // write to different output buffers, as long as they have the same bounds.
  if (backendType == kLLVMCodeGen) {
    fuseAllLoops(l.root_stmt());
    GRAPH_DEBUG("after fuse", *l.root_stmt());
    parallelizeOuterLoops(l, bufsToBeParallelized_);
    GRAPH_DEBUG("after parallelize", *l.root_stmt());
  }
```
- **EN**: This chunk continues `l` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `l`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 852-887
```cpp
  if (backendType == kCudaCodeGen) {
    for (const auto& buf : bufOutputs_) {
      std::vector<ForPtr> loops = l.getLoopStmtsFor(buf);
      if (loops.empty()) {
        // This happens when Buf is 0-dim
        continue;
      }
      ForPtr flattened = nullptr;
      LoopNest::flatten(loops, &flattened);
      assert(flattened);

      int loopLevels = getTECudaPointwiseLoopLevels();
      const int kDefaultLoopLevels = 2;
      loopLevels = (loopLevels > 0) ? loopLevels : kDefaultLoopLevels;
      int blockCount = getTECudaPointwiseBlockCount();
      int blockSize = getTECudaPointwiseBlockSize();

      if (loopLevels == 2) {
        ForPtr inner;
        const int kDefaultBlockSize = 512;
        if (blockSize < 0) {
          blockSize = kDefaultBlockSize;
        }
        LoopNest::splitWithMask(flattened, blockSize, &inner);
        flattened->set_gpu_block_index(0);
        inner->set_gpu_thread_index(0);
      } else if (loopLevels == 3) {
        ForPtr inner;
        ForPtr inner1;
        // TODO: change the number of microprocessors
        const int kDefaultBlockCount = 1280;
        const int kDefaultBlockSize = 256;
        blockCount = (blockCount > 0) ? blockCount : kDefaultBlockCount;
        blockSize = (blockSize > 0) ? blockSize : kDefaultBlockSize;
        LoopNest::splitWithMask(flattened, blockCount * blockSize, &inner);
        LoopNest::splitWithMask(inner, blockSize, &inner1);
```
- **EN**: This chunk continues `l` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `l`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 888-922
```cpp
        inner->set_gpu_block_index(0);
        inner1->set_gpu_thread_index(0);
      } else {
        throw std::runtime_error(
            "Invalid loop-level: " + std::to_string(loopLevels));
      }
    }
  }

  if (backendType == kBlockCodeGen) {
    for (const auto& buf : bufOutputs_) {
      const int default_fp16_blocksize = 16;
      const int default_uint8_blocksize = 32;
      int blockSize = default_fp16_blocksize;
      // We only handle looplevels == 2 for now
      if (buf->dtype().scalar_type() == ScalarType::Byte) {
        blockSize = default_uint8_blocksize;
      }
      std::vector<ForPtr> loops = l.getLoopStmtsFor(buf);
      TORCH_INTERNAL_ASSERT(
          !loops.empty(),
          buildErrorMessage(
              "No loops found for the buffer " + buf->name_hint() +
              " in the fuser."));
      ForPtr flattened = nullptr;
      LoopNest::flatten(loops, &flattened);
      assert(flattened);

      ForPtr inner = nullptr;
      LoopNest::splitWithMask(flattened, blockSize, &inner);
      flattened->set_gpu_block_index(0);
      inner->set_gpu_thread_index(0);
      flattened->set_buffer_map(block_analysis->getBufferMap());
    }
  }
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 924-959
```cpp
  if (pre_alloc_) {
    auto interm_bufs = l.getIntermediateBufs();
    preAllocIntermediateBufs(interm_bufs);
  }

  l.prepareForCodegen();

  GRAPH_DEBUG("after prepareForCodegen", *l.root_stmt());
  l.simplify();
  GRAPH_DEBUG("after simplification", *l.root_stmt());

  if (backendType == kLLVMCodeGen && !hasReduction) {
    l.vectorizeInnerLoops();
    GRAPH_DEBUG("after vectorization", *l.root_stmt());
  }

  StmtPtr stmt = l.root_stmt();
  // Arithmetic Simplification.
  stmt = IRSimplifier::simplify(stmt);
  GRAPH_DEBUG("Final Stmt:\n", std::to_string(stmt), '\n');
  return stmt;
}

std::string TensorExprKernel::getCodeGenName(BackendType backendType) {
  switch (backendType) {
    case kCudaCodeGen:
      return "cuda_codegen";
    case kLLVMCodeGen:
      return "llvm_codegen";
    case kSimpleIREval:
      return "simple_ir_eval";
    case kBlockCodeGen:
      return "block_codegen";
    default:
      throw std::runtime_error(
          "invalid backend type: " +
```
- **EN**: This chunk defines `getCodeGenName`, which lowers higher-level IR into a backend-specific executable form. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getCodeGenName`，其作用是把高层 IR 降级为后端特定的可执行形式。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 960-989
```cpp
          std::to_string(static_cast<int>(backendType)));
  }
}

template <typename T>
static bool isValidPrimProperty(const std::optional<T>& a, T b) {
  return !a.has_value() || *a == b;
}

TensorExprKernel::BackendType TensorExprKernel::inferBackendTypeFromDevice(
    at::Device device) {
  BackendType backendType = BackendType::kUninitialized;
  if (device.type() == at::kCUDA) {
    backendType = kCudaCodeGen;
  } else if (device.type() == at::kCPU && getTEGenerateBlockCode()) {
    backendType = kBlockCodeGen;
  } else if (device.type() == at::kCPU) {
#ifdef TORCH_ENABLE_LLVM
    backendType = dontUseLLVMFlag() ? kSimpleIREval : kLLVMCodeGen;
#else
    backendType = kSimpleIREval;
#endif
    if (getTEMustUseLLVMOnCPU() && backendType == kSimpleIREval) {
      throw std::runtime_error("LLVM Backend not found");
    }
  } else {
    throw std::runtime_error("Invalid device type");
  }
  return backendType;
}
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 991-1016
```cpp
// we use the debug names in printing cuda code, they need to be removed
// of characters that can't be used in a variable identifier
void TensorExprKernel::genInputDebugNames() {
  std::unordered_set<std::string> name_set;
  std::unordered_map<const torch::jit::Value*, std::string> value_to_name;
  for (const torch::jit::Value* input : graph_->inputs()) {
    std::string sanitized_name = sanitizeName(input->debugName());
    // we could get fancier here, but name conflict is extremely unlikely
    while (name_set.count(sanitized_name)) {
      sanitized_name.append("_");
    }
    value_to_name[input] = sanitized_name;
    name_set.insert(sanitized_name);
  }
  input_name_map_ = std::move(value_to_name);
}

template <typename T>
static std::vector<ExprHandle> toExprHandles(const std::vector<T>& sizes) {
  std::vector<ExprHandle> dims;
  dims.reserve(sizes.size());
  for (auto const& size : sizes) {
    dims.emplace_back(size);
  }
  return dims;
}
```
- **EN**: This chunk defines `toExprHandles`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `toExprHandles`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1018-1043
```cpp
ExprHandle TensorExprKernel::getStrideArg(
    size_t tensor_input_index,
    size_t stride_index) {
  auto it = strideArgToVar_.find(
      std::pair<size_t, size_t>(tensor_input_index, stride_index));
  if (it == strideArgToVar_.end()) {
    VarHandle var(
        "stride_arg" + std::to_string(tensor_input_index) + "_" +
            std::to_string(stride_index),
        kLong);
    strideArgToVar_[std::pair<size_t, size_t>(
        tensor_input_index, stride_index)] = var;
#if C10_RETURN_MOVE_IF_OLD_COMPILER
    return std::move(var);
#else
    return var;
#endif
  }
  return it->second;
}

std::vector<torch::jit::StrideInput>& TensorExprKernel::getSymbolicStrideDesc(
    const torch::jit::Value* value) {
  TORCH_INTERNAL_ASSERT(symbolic_strides_.count(value));
  return symbolic_strides_[value];
}
```
- **EN**: This chunk defines `getSymbolicStrideDesc`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getSymbolicStrideDesc`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1045-1069
```cpp
std::vector<ExprHandle> TensorExprKernel::getInputStrides(
    const torch::jit::Value* input,
    const std::vector<ExprHandle>& inputTensorDims) {
  std::vector<ExprHandle> inputTensorStrides;
  if (input->isCompleteTensor()) {
    auto const strides =
        input->type()->expect<TensorType>()->strides().concrete_sizes();
    std::vector<ExprHandle> inputTensorStrides;
    for (size_t stride : *strides) {
      inputTensorStrides.push_back(LongImm::make(stride));
    }
    return inputTensorStrides;
  }

  size_t rank = inputTensorDims.size();
  std::vector<StrideInput>& stride_input = getSymbolicStrideDesc(input);
  if (stride_input.size() == 1 &&
      (stride_input[0] == StrideInput::TENSOR_CONT_CHANNELS_LAST ||
       stride_input[0] == StrideInput::TENSOR_CONT)) {
    auto strides = stride_input[0] == StrideInput::TENSOR_CONT
        ? make_contiguous_strides(inputTensorDims)
        : make_channels_last_strides(inputTensorDims);
    return fmap(
        strides, [&](ExprPtr stride) { return ExprHandle(std::move(stride)); });
  }
```
- **EN**: This chunk defines `getInputStrides`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getInputStrides`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1071-1093
```cpp
  inputTensorStrides.resize(rank);
  std::vector<bool> stride_set(rank, false);
  // first, generate non-dependent values
  size_t generated_strides = 0;
  for (const auto i : c10::irange(rank)) {
    if (stride_input[i] == torch::jit::StrideInput::S_ONE) {
      inputTensorStrides[i] = LongImm::make(1);
      stride_set[i] = true;
      generated_strides++;
    } else if (stride_input[i] == torch::jit::StrideInput::S_AS_ARG) {
      size_t input_index = input->offset();
      inputTensorStrides[i] = getStrideArg(input_index, i);
      stride_set[i] = true;
      generated_strides++;
    }
  }
  // Contiguous and Transposed Contiguous depend on adjacent values
  while (generated_strides != rank) {
    for (int i = static_cast<int>(rank) - 1; i >= 0; i--) {
      if (stride_input[i] == torch::jit::StrideInput::S_CONT &&
          stride_set[i + 1]) {
        inputTensorStrides[i] =
            inputTensorStrides[i + 1] * inputTensorDims[i + 1];
```
- **EN**: This chunk defines `stride_set`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `stride_set`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1095-1127
```cpp
        stride_set[i] = true;
        generated_strides++;
      }
    }
    for (int i = 0; i < static_cast<int>(rank); i++) {
      if (stride_input[i] == torch::jit::StrideInput::S_TRAN_CONT &&
          stride_set[i - 1]) {
        inputTensorStrides[i] =
            inputTensorStrides[i - 1] * inputTensorDims[i - 1];
        stride_set[i] = true;
        generated_strides++;
      }
    }
  }
  return inputTensorStrides;
}

Tensor TensorExprKernel::bindInput(const torch::jit::Value* input) {
  auto const& t = input->type();
  auto const& outputs = input->owningGraph()->outputs();
  std::unordered_set<const Value*> outputs_set(outputs.begin(), outputs.end());

  auto is_concrete_cont = [](const torch::jit::Value* input,
                             const MemoryLayoutPolicy& mem_layout_policy) {
    if (input->isCompleteTensor()) {
      auto mem_layout = (mem_layout_policy == MemoryLayoutPolicy::kContiguous)
          ? at::MemoryFormat::Contiguous
          : at::MemoryFormat::ChannelsLast;
      return isContiguous(input, mem_layout);
    } else {
      return false;
    }
  };
```
- **EN**: This chunk defines `outputs_set`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `outputs_set`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1129-1156
```cpp
  auto is_symbolic_cont = [](std::vector<torch::jit::StrideInput> desc,
                             const MemoryLayoutPolicy& mem_layout_policy) {
    if (desc.size() == 1) {
      auto mem_layout = (mem_layout_policy == MemoryLayoutPolicy::kContiguous)
          ? torch::jit::StrideInput::TENSOR_CONT
          : torch::jit::StrideInput::TENSOR_CONT_CHANNELS_LAST;
      return desc[0] == mem_layout;
    } else {
      return false;
    }
  };

  Tensor result(nullptr, nullptr);
  switch (t->kind()) {
    case TypeKind::TensorType: {
      auto tt = input->type()->cast<TensorType>();
      bool contiguous_concrete_tensor =
          is_concrete_cont(input, memory_layout_policy_);
      bool contiguous_symbolic_tensor = false;
      if (has_symbolic_shapes_) {
        auto desc = getSymbolicStrideDesc(input);
        contiguous_symbolic_tensor =
            is_symbolic_cont(desc, memory_layout_policy_);
      }

      // Get input size and strides
      auto size_handles = sizesFromSymbolicShape(tt->symbolic_sizes());
      auto inputTensorStrides = getInputStrides(input, size_handles);
```
- **EN**: This chunk defines `result`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `result`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1158-1193
```cpp
      // We don't need to copy the input if:
      //  1) it is not an output AND
      //  2) it is contiguous
      bool contiguous =
          contiguous_concrete_tensor || contiguous_symbolic_tensor;
      if (!outputs_set.count(input) && contiguous) {
        BufHandle inBuffer(
            "t" + input_name_map_[input],
            sizesFromSymbolicShape(tt->symbolic_sizes()),
            inputTensorStrides,
            ToDtype(static_cast<ScalarType>(*tt->scalarType())));
        TORCH_INTERNAL_ASSERT_DEBUG_ONLY(
            inBuffer.node()->is_contiguous() ||
            inBuffer.node()->is_channels_last_1d_contiguous() ||
            inBuffer.node()->is_contiguous(at::MemoryFormat::ChannelsLast) ||
            inBuffer.node()->is_contiguous(at::MemoryFormat::ChannelsLast3d));
        bufs_.emplace(input, inBuffer.node());
        bufferArgs_.emplace_back(inBuffer);
        break;
      }

      // if the input isn't contiguous or is an output,
      // write strided input into  contiguous buffer that is
      // then used in all further compute
      ExprHandle flat_size = 1;
      for (size_t i = 0; i < size_handles.size(); ++i) {
        auto size = size_handles[i];
        if (size.AsNode<LongImm>() && immediateAs<int64_t>(size.node()) == 0) {
          flat_size = 0;
          break;
        }
        flat_size = flat_size + (size - 1) * inputTensorStrides[i];
      }
      flat_size = IRSimplifier::simplify(flat_size);
      BufHandle inBuffer(
          "t" + input_name_map_[input],
```
- **EN**: This chunk defines `inBuffer`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `inBuffer`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1194-1229
```cpp
          {flat_size},
          ToDtype(static_cast<ScalarType>(*tt->scalarType())));

      result = Compute(
          "input" + std::to_string(bufs_.size() + 1),
          size_handles,
          [&](const std::vector<VarHandle>& axes) {
            ExprHandle idx = 0;
            for (size_t i = 0; i < axes.size(); i++) {
              idx = idx + axes[i] * inputTensorStrides[i];
            }
            return inBuffer.load(idx);
          });
      bufs_.emplace(input, result.buf());
      bufferArgs_.emplace_back(inBuffer);
      break;
    }
    case TypeKind::FloatType: {
      VarHandle v("v" + input_name_map_[input], kDouble);
      bufferArgs_.emplace_back(v);
      scalars_.emplace(input, v);
      break;
    }
    case TypeKind::BoolType: {
      VarHandle v("v" + input_name_map_[input], kBool);
      bufferArgs_.emplace_back(v);
      scalars_.emplace(input, v);
      break;
    }
    case TypeKind::IntType: {
      VarHandle v("v" + input_name_map_[input], kLong);
      bufferArgs_.emplace_back(v);
      scalars_.emplace(input, v);
      break;
    }
    default: {
```
- **EN**: This chunk defines `v`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `v`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1230-1260
```cpp
      throw unsupported_dtype(t->repr_str());
      break;
    }
  }
  return result;
}

NNCLoweringFunction TensorExprKernel::getCustomLoweringFor(
    c10::Symbol op) const {
  if (custom_lowerings_.count(op))
    return custom_lowerings_.at(op);
  return nullptr;
}

template <typename T>
static std::vector<size_t> reverse_sort_indices(const std::vector<T>& v) {
  // initialize original index locations
  std::vector<size_t> idx(v.size());
  iota(idx.begin(), idx.end(), 0);

  std::sort(idx.begin(), idx.end(), [&v](size_t i1, size_t i2) {
    return v[i1] > v[i2];
  });
  return idx;
}

static bool denseAndNonOverlapping(
    at::ArrayRef<int64_t> sizes,
    at::ArrayRef<int64_t> strides) {
  return (strides == at::infer_dense_strides(sizes, strides));
}
```
- **EN**: This chunk defines `denseAndNonOverlapping`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `denseAndNonOverlapping`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1262-1297
```cpp
Tensor TensorExprKernel::convertSymbolicOutputToCorrectStrides(
    const std::vector<ExprHandle>& sizes,
    const std::vector<size_t>& sorted_stride_indices_descending,
    const std::vector<ExprPtr>& strides,
    BufPtr& buf) {
  // We need to convert the output tensor so that its values are laid
  // so that when viewed from the output strides the values are correct.
  // A contiguous Tensor of size(2, 3) with values 0-5 is laid out as:
  // [0] [1] [2] [3] [4] [5]
  // The same valued tensor with strides (1, 2) would be laid out like
  // [0] [3] [1] [4] [2] [5]
  // When we are doing the re-ordering of values into the output tensor,
  // we are iterating per-element of the input, and we are fixed
  // in indexing in to the output tensor at [i, j] = val
  // `val` we want here is equal to the indices for the output
  // tensor that would have given the same position as the output
  // The position is equal to the sum of stride[i] * index[i],
  // and we can can calculate the equivalent indices in the
  // output tensor strides by iteratively computing the index of
  // the biggest stride:
  // absolute = ...
  // for stride in strides_from_largest_to_smallest:
  //     cur_idx = absolute // stride
  //     absolute = absolute % stride
  std::vector<ExprPtr> default_strides = make_contiguous_strides(sizes);
  auto zero = LongImm::make(0);
  return Compute(
      "output_1", sizes, [&](const std::vector<VarHandle>& axes_input) {
        std::vector<ExprHandle> axes(axes_input.begin(), axes_input.end());
        auto absolute_position = ExprHandle(immLike(axes[0], 0));
        for (size_t i = 0; i < axes.size(); ++i) {
          ExprHandle stride(default_strides[i]);
          absolute_position = absolute_position + (stride * axes[i]);
        }
        std::vector<ExprHandle> new_axes(
            sorted_stride_indices_descending.size());
```
- **EN**: This chunk defines `new_axes`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `new_axes`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1298-1332
```cpp
        for (size_t stride_index : sorted_stride_indices_descending) {
          const auto& stride = strides[stride_index];
          auto index = absolute_position / ExprHandle(stride);
          // XXX, in symbolic output ordering, we do not the arbitrary
          // ordering of strides as in usual output ordering, just
          // channels last, so even in the presence of size == 1
          // we produce correct output here
          absolute_position = absolute_position % ExprHandle(stride);
          new_axes[stride_index] = index;
        }
        return BufHandle(buf).load(new_axes);
      });
}

Tensor TensorExprKernel::convertSymbolicOutputToCorrectStrides(
    torch::jit::Value* v) {
  const TensorTypePtr& tt = v->type()->expect<TensorType>();
  TORCH_INTERNAL_ASSERT(
      bufs_.count(v),
      buildErrorMessage(
          "Output tensor has no corresponding bufs in the fuser."));
  BufPtr buf = bufs_.at(v);
  TORCH_INTERNAL_ASSERT(buf != nullptr);
  TORCH_INTERNAL_ASSERT(tt != nullptr);
  TORCH_INTERNAL_ASSERT(tt->symbolic_sizes().rank() != std::nullopt);

  auto stride_desc = getSymbolicStrideDesc(v);
  TORCH_INTERNAL_ASSERT(stride_desc.size() == 1);
  auto memory_format = (stride_desc[0] == torch::jit::StrideInput::TENSOR_CONT)
      ? at::MemoryFormat::Contiguous
      : at::MemoryFormat::ChannelsLast;
  // output is contiguous with specified memory format, no work to do
  if (buf->is_contiguous(memory_format)) {
    return Tensor(buf, nullptr);
  }
```
- **EN**: This chunk defines `convertSymbolicOutputToCorrectStrides`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `convertSymbolicOutputToCorrectStrides`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1334-1363
```cpp
  TORCH_INTERNAL_ASSERT(
      stride_desc[0] == torch::jit::StrideInput::TENSOR_CONT_CHANNELS_LAST);
  auto sizes = sizesFromSymbolicShape(tt->symbolic_sizes());
  auto strides = make_channels_last_strides(sizes);
  // For a tensor with dimensions N C H W, channels last
  // format will is in format N H W C,
  // so the order largest to smallest will be N, H, W, C
  std::vector<size_t> sorted_stride_indices = {0, 2, 3, 1};
  auto zero = LongImm::make(0);
  std::vector<ExprPtr> default_strides = make_contiguous_strides(sizes);
  // See explanation in convertOutputToCorrectStrides
  return convertSymbolicOutputToCorrectStrides(
      sizes, sorted_stride_indices, strides, buf);
}

Tensor TensorExprKernel::convertStaticShapeOutputToCorrectStrides(
    torch::jit::Value* v) {
  const TensorTypePtr& tt = v->type()->expect<TensorType>();
  TORCH_INTERNAL_ASSERT(
      bufs_.count(v),
      buildErrorMessage(
          "Output tensor has no corresponding bufs in the fuser."));
  BufPtr buf = bufs_.at(v);

  // No shape info is present in the graph
  if (!tt->sizes().concrete_sizes()) {
    std::string msg =
        std::string("Shapes for output '%") + v->debugName() + "' are unknown";
    throw malformed_input(msg);
  }
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1365-1395
```cpp
  TORCH_INTERNAL_ASSERT(
      tt->sizes().concrete_sizes(),
      buildErrorMessage("Output shapes are unknown."));
  auto sizes = *tt->sizes().concrete_sizes();
  at::MemoryFormat memory_format =
      (memory_layout_policy_ == MemoryLayoutPolicy::kContiguous)
      ? c10::MemoryFormat::Contiguous
      : c10::MemoryFormat::ChannelsLast;
  std::vector<int64_t> default_strides =
      TensorType::contiguousStridesOf(sizes, memory_format);
  if (!tt->strides().concrete_sizes()) {
    return Tensor(buf, nullptr);
  }
  TORCH_INTERNAL_ASSERT(
      tt->strides().concrete_sizes(),
      buildErrorMessage("Output strides are unknown."));
  const std::vector<int64_t> strides = *tt->strides().concrete_sizes();
  // All Tensors in NNC are laid out in default, contiguous layout.
  // If the output is also default contiguous we don't need to do anything
  if (strides == default_strides) {
    return Tensor(buf, nullptr);
  }
  // If the tensor is not dense or overlaps, we have
  // no way of matching the profiled striding
  if (!denseAndNonOverlapping(sizes, strides)) {
    return Tensor(buf, nullptr);
  }

  auto dims = sizesForValue(v);
  auto zero = LongImm::make(0);
  std::vector<size_t> sorted_stride_indices = reverse_sort_indices(strides);
```
- **EN**: This chunk continues `malformed_input` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `malformed_input`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1397-1424
```cpp
  // TODO: call into `convertOutputToCorrectStrides`. Currently this causes a
  // bug in IRSimplifier to occur. See explanation in
  // `convertOutputToCorrectStrides`
  return Compute(
      "output_1", dims, [&](const std::vector<VarHandle>& axes_input) {
        std::vector<ExprHandle> axes(axes_input.begin(), axes_input.end());
        auto absolute_position = ExprHandle(immLike(axes[0], 0));
        for (size_t i = 0; i < axes.size(); ++i) {
          absolute_position = absolute_position +
              (ExprHandle(immLike(axes[i], default_strides[i])) * axes[i]);
        }

        std::vector<ExprHandle> new_axes(sorted_stride_indices.size());
        for (size_t stride_index : sorted_stride_indices) {
          auto size = sizes[stride_index];
          auto index = zero;
          if (size != 1) {
            auto stride = strides[stride_index];
            index = absolute_position /
                ExprHandle(immLike(absolute_position, stride));
            absolute_position = absolute_position %
                ExprHandle(immLike(absolute_position, stride));
          }
          new_axes[stride_index] = index;
        }
        return BufHandle(buf).load(new_axes);
      });
}
```
- **EN**: This chunk defines `new_axes`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `new_axes`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1426-1460
```cpp
void TensorExprKernel::bindConstant(const torch::jit::Value* v) {
  auto val = toIValue(v).value();
  if (torch::isCustomClass(val)) {
    auto name_hint = "const_" + sanitizeName(v->debugName());
    auto dtype = Dtype(ScalarType::Float);
    std::vector<ExprPtr> dims;
    BufPtr buf = alloc<Buf>(name_hint, dims, dtype);
    auto dataPtr = val.toObjectRef().getSlot(0).toCapsule().get();
    // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
    constants_.push_back({buf, dataPtr, const_cast<Node*>(v->node())});
    bufs_[v] = buf;
    return;
  }
  if (!v->type()->cast<TensorType>()) {
    // Only Tensor constants need to be bound, scalar constants will be turned
    // into immediates in TE IR
    return;
  }
  auto const_tensor = toIValue(v)->toTensor();
  auto scalar_type = c10::typeMetaToScalarType(const_tensor.options().dtype());
  auto sizes = const_tensor.sizes();
  std::vector<ExprHandle> te_sizes;
  te_sizes.reserve(sizes.size());
  for (auto s : sizes) {
    te_sizes.emplace_back(s);
  }
  BufPtr buf = alloc<Buf>(
      "const_" + sanitizeName(v->debugName()),
      ExprHandleVectorToExprVector(te_sizes),
      ToDtype(scalar_type));

  if (!const_tensor.is_contiguous()) {
    const_tensor = const_tensor.clone(at::MemoryFormat::Contiguous);
    unpacked_constant_tensors_.push_back(const_tensor);
  }
```
- **EN**: This chunk defines `bindConstant`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `bindConstant`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1462-1494
```cpp
  constants_.push_back({buf, const_tensor.data_ptr()});
  bufs_[v] = buf;
}

std::vector<BufPtr> TensorExprKernel::preAllocIntermediateBufs(
    const std::vector<BufPtr>& interm_bufs) {
  std::vector<BufPtr> remaining_interm_bufs;
  for (const auto& buf : interm_bufs) {
    // Check if buf shape is static and compute its size if static.
    bool is_static = true;
    size_t size =
        elementSize(buf->dtype().scalar_type()) * buf->dtype().lanes();
    for (auto& d : buf->dims()) {
      if (!d->isConstant()) {
        is_static = false;
        break;
      }
      size = size * (*intValue(d));
    }
    // Only allocate memory for static bufs.
    if (!is_static) {
      remaining_interm_bufs.push_back(buf);
      continue;
    }
    auto bp = malloc(size);
    if (!bp) {
      remaining_interm_bufs.push_back(buf);
      continue;
    }
    constants_.push_back({buf, bp});
  }
  return remaining_interm_bufs;
}
```
- **EN**: This chunk defines `preAllocIntermediateBufs`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `preAllocIntermediateBufs`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1496-1531
```cpp
BlockPtr TensorExprKernel::bindAllInputs() {
  std::vector<CodeGen::BufferArg> symbolic_shape_args;
  std::vector<CodeGen::BufferArg> symbolic_stride_args;

  auto symbolic_shape_inputs_start_pos =
      nInputs_ - symbolic_shape_inputs_.size();
  if (has_symbolic_shapes_) {
    // The graph is supposed to have input params that represent the symbolic
    // dims at the end of the list of inputs. The number of such symbolic input
    // params is defined by the size of the `symbolic_shape_inputs_` vector.
    //
    // TODO: Check if the tensors with symbolic shapes are contiguous.
    TORCH_CHECK(
        nInputs_ > static_cast<int64_t>(symbolic_shape_inputs_.size()),
        "Symbolic dims not provided as inputs to the graph");

    // First, process the symbolic input params and create a new variable for
    // each of them.
    // NOTE: This has to be done before processing the tensor inputs, because
    // their symbolic sizes needs to be associated with these variables we
    // create for the symbolic input params.
    symbolic_shape_args.reserve(symbolic_shape_inputs_.size());

    for (size_t i = symbolic_shape_inputs_start_pos;
         i < static_cast<size_t>(nInputs_);
         ++i) {
      auto input = graph_->inputs()[i];
      if (input->type()->kind() != TypeKind::IntType) {
        throw std::runtime_error(
            "Expected integer type input to graph for symbolic dims.");
      }
      VarHandle v("v" + input_name_map_[input], kLong);
      symbolic_shape_args.emplace_back(v);
      scalars_.emplace(input, v);
      shapeSymbolInputPos_[scalars_[input].node()] = i;
    }
```
- **EN**: This chunk defines `v`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `v`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1532-1558
```cpp
    // For every shape symbol, store a map to the corresponding var.
    for (size_t i = 0; i < symbolic_shape_inputs_.size(); ++i) {
      shapeSymbolToVar_[symbolic_shape_inputs_[i]] =
          scalars_[graph_->inputs()[symbolic_shape_inputs_start_pos + i]];
    }

    // Next, process symbolic input params and create an argument for symbolic
    for (size_t i = 0; i < symbolic_shape_inputs_start_pos; ++i) {
      auto input = graph_->inputs()[i];
      auto tt = input->type()->cast<TensorType>();
      if (!tt) {
        continue;
      }
      auto symbolic_stride = getSymbolicStrideDesc(input);
      for (size_t j = 0; j < symbolic_stride.size(); ++j) {
        if (symbolic_stride[j] == torch::jit::StrideInput::S_AS_ARG) {
          VarHandle v("v" + input_name_map_[input], kLong);
          symbolic_stride_args.emplace_back(v);
          strideArgToVar_[{i, j}] = v;
          input_stride_args_.emplace_back(i, j);
        }
      }
    }
  }

  // Block to collect the Stmts corresponding to all tensors.
  auto block = alloc<Block>(std::vector<StmtPtr>({}));
```
- **EN**: This chunk defines `v`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `v`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1560-1595
```cpp
  // Process the inputs before the symbolic input params.
  for (const auto i : c10::irange(symbolic_shape_inputs_start_pos)) {
    auto input = graph_->inputs()[i];
    Tensor t = bindInput(input);
    if (t.stmt()) {
      block->append_stmt(t.stmt());
    }
  }
  // Now, add all the variables corresponding to the symbolic input params.
  bufferArgs_.insert(
      bufferArgs_.end(),
      symbolic_shape_args.begin(),
      symbolic_shape_args.end());

  // Now, add all the variables corresponding to symbolic stride inputs
  bufferArgs_.insert(
      bufferArgs_.end(),
      symbolic_stride_args.begin(),
      symbolic_stride_args.end());

  return block;
}

void TensorExprKernel::deduceMemoryLayoutPolicy() {
  // If the tensor is channels-last contiguous, the preferred memory layout
  // propagation policy is to use channels-last. Otherwise, the preferred policy
  // is to use contiguous.
  auto _prefer_symbolic_mem =
      [](const torch::jit::Value* val,
         const std::vector<torch::jit::StrideInput>& stride_desc_vec) {
        TORCH_INTERNAL_ASSERT(!stride_desc_vec.empty());
        // Has symbolic stride information
        auto cur_stride_desc = stride_desc_vec[0];
        return (cur_stride_desc ==
                torch::jit::StrideInput::TENSOR_CONT_CHANNELS_LAST)
            ? MemoryLayoutPolicy::kChannelsLastNdContiguous
```
- **EN**: This chunk defines `deduceMemoryLayoutPolicy`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `deduceMemoryLayoutPolicy`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1596-1631
```cpp
            : MemoryLayoutPolicy::kContiguous;
      };

  auto _prefer_static_mem = [](const torch::jit::Value* val) {
    // No shape info is present in the graph
    TORCH_INTERNAL_ASSERT(
        val->isCompleteTensor(),
        buildErrorMessage(val->debugName() + " is not a complete tensor."));
    const auto& tt = val->type()->expect<TensorType>();
    const auto sizes = *tt->sizes().concrete_sizes();
    const auto strides = *tt->strides().concrete_sizes();
    return (c10::is_channels_last_strides_2d(sizes, strides))
        ? MemoryLayoutPolicy::kChannelsLastNdContiguous
        : MemoryLayoutPolicy::kContiguous;
  };

  // Filter out the tensor from the graph inputs and outputs to
  // deduce the memory layout propagation policy
  auto _is_tensor = [](const jit::Value* el) {
    return el->type()->kind() == TypeKind::TensorType;
  };
  std::vector<torch::jit::Value*> graph_io_tensors;
  std::copy_if(
      graph_->inputs().begin(),
      graph_->inputs().end(),
      std::back_inserter(graph_io_tensors),
      _is_tensor);
  std::copy_if(
      graph_->outputs().begin(),
      graph_->outputs().end(),
      std::back_inserter(graph_io_tensors),
      _is_tensor);
  // std::all_of returns true if the range is empty. But we prefer to keep
  // the original memory layout propagation policy for this case. So we
  // check whether the range is empty.
  auto prefer_channels_last = (!graph_io_tensors.empty());
```
- **EN**: This chunk continues `deduceMemoryLayoutPolicy` and expands its internal control flow or data movement. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `deduceMemoryLayoutPolicy`，进一步展开其内部控制流或数据流转。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1632-1664
```cpp
  for (auto el : graph_io_tensors) {
    auto is_complete = el->isCompleteTensor();
    auto is_symbolic = symbolic_strides_.count(el);

    auto preferred_mem_layout = is_complete
        ? _prefer_static_mem(el)
        : (is_symbolic ? _prefer_symbolic_mem(el, symbolic_strides_[el])
                       : MemoryLayoutPolicy::kContiguous);
    if (preferred_mem_layout != MemoryLayoutPolicy::kChannelsLastNdContiguous) {
      prefer_channels_last = false;
      break;
    }
  }

  // If the memory layout of all the input and outputs is channels-last
  // contiguous, the propagated memory layout should be channels-last.
  // Otherwise, the propagated memory layout is contiguous which is as
  // same as current situation.
  memory_layout_policy_ = prefer_channels_last
      ? MemoryLayoutPolicy::kChannelsLastNdContiguous
      : MemoryLayoutPolicy::kContiguous;
}

void TensorExprKernel::optimizeOwningGraph() {
  GRAPH_DUMP("TensorExprKernel graph (Before graph optimization):", graph_);

  // We may manipulate output pointers in graph manipulation. So we store the
  // original outputs for symbolic strides information synchronization
  auto _orignal_graph_outputs = graph_->outputs().vec();

  // Get the graph device information first. The graph optimization
  // might be device specific.
  device_ = *pickDeviceType(graph_);
```
- **EN**: This chunk defines `optimizeOwningGraph`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `optimizeOwningGraph`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1666-1700
```cpp
  // Determine the propagated memory layout
  deduceMemoryLayoutPolicy();

  // Fuse Conv with Eltwise Op
  graph_rewrite_helper::replaceConvolutionWithAtenConv(graph_);
  FuseConvWithEltwise(graph_);

  // Optimize the concatenation
  OptimizeCat(graph_);

  // Synchronize the symbolic strides information
  auto graph_outputs = graph_->outputs();
  TORCH_INTERNAL_ASSERT(graph_outputs.size() == _orignal_graph_outputs.size());
  for (auto i : c10::irange(graph_outputs.size())) {
    auto el_orig = _orignal_graph_outputs.at(i);
    auto el_new = graph_outputs.at(i);
    if (symbolic_strides_.count(el_orig) && (el_orig != el_new)) {
      symbolic_strides_[el_new] = symbolic_strides_[el_orig];
      symbolic_strides_.erase(el_orig);
    }
  }

  GRAPH_DUMP("TensorExprKernel graph (After graph optimization):", graph_);
}

void TensorExprKernel::compile() {
  GRAPH_DUMP("TensorExprKernel graph:", graph_);

  has_symbolic_shapes_ = !symbolic_shape_inputs_.empty();
  nInputs_ = graph_->inputs().size();
  nOutputs_ = graph_->outputs().size();
  genInputDebugNames();

  // Bind inputs to buffers.
  auto block = bindAllInputs();
```
- **EN**: This chunk defines `compile`, which lowers higher-level IR into a backend-specific executable form. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `compile`，其作用是把高层 IR 降级为后端特定的可执行形式。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1702-1726
```cpp
  // Bind nodes to tensor compute expressions.
  for (auto const& n : graph_->nodes()) {
    if (n->kind() == prim::ListConstruct) {
      continue;
    } else if (n->kind() == prim::Constant) {
      bindConstant(n->output());
      continue;
    } else {
      for (auto const& output : n->outputs()) {
        if (output->hasUses()) {
          Tensor t = computeValue(output);

          // If there are for-loops before ExternalCall as follows,
          //   stmt1: for:
          //   stmt2    for:
          //   stmt3: ExternalCall
          // the for-loops would not be parallelized. So we mark the
          // buf args of ExternalCall as to be parallelized to make sure
          // its previous loop still could be parallelized.
          if (to<ExternalCall>(t.stmt())) {
            auto _external_call = to<ExternalCall>(t.stmt());
            for (const auto& _buf : _external_call->buf_args()) {
              bufsToBeParallelized_.insert(_buf);
            }
          }
```
- **EN**: This chunk continues `compile` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `compile`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1728-1763
```cpp
          if (output->type()->cast<TensorType>()) {
            // Value is tensor
            if (t.buf()) {
              bufs_.emplace(output, t.buf());
            }
            block->append_stmt(t.stmt());
          } else {
            // Value is scalar
            //
            // We represent scalar computations in TE with a pair of statements:
            //   Let val = <compute_expression>
            //   Store(buf_for_scalar[0], val)
            //
            // Subsequent computations will use val when they refer to the
            // given value, and the buffer will be used if we need to return
            // the computed value as an output of the kernel. If this is not an
            // output, the store will be removed later by DCE.
            //
            // NB: NNC's lowering functions return Tensor, which is a pair
            // <Buf, Stmt>, but here we also need Var. How can we obtain all of
            // Var, Buf, and Stmt?
            // We use the following trick: the lowering function creates the
            // Let-stmt and a "fake" buffer, whose only purpose is to hold the
            // Var. Then outside the lowering function (namely, right here) we
            // generate the store and the actual buffer.
            VarPtr v = t.buf()->base_handle();
            scalars_[output] = VarHandle(v);
            block->append_stmt(t.stmt());
            BufHandle buf(
                "scalar_" + sanitizeName(output->debugName()), {}, v->dtype());
            StmtPtr store = Store::make(buf, {}, ExprHandle(v));
            block->append_stmt(store);
            bufs_.emplace(output, buf.node());
          }
        }
      }
```
- **EN**: This chunk continues `compile` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `compile`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1764-1789
```cpp
    }
    if (hasRandom_ && hasBroadcast_) {
      throw std::runtime_error(
          "Cannot support broadcast and random within one kernel");
    }
  }

  // Move output operands from `bufs_` to `bufOutputs_`
  for (auto i : c10::irange(graph_->outputs().size())) {
    auto& output = graph_->outputs().at(i);
    if (!bufs_.count(output)) {
      throw malformed_input("cannot find output Tensor");
    }
    if (!output->type()->cast<TensorType>()) {
      // Scalar outputs are represented as 0-dim buffers.
      bufOutputs_.insert(bufs_.at(output));
      bufsToBeParallelized_.insert(bufs_.at(output));
      bufferArgs_.emplace_back(BufHandle(bufs_.at(output)));
      tensorOutputTensorOptions_.emplace_back(
          c10::TensorOptions(tensorType(bufs_.at(output))).device(device_));
      tensorOutputSizes_.emplace_back();
      tensorOutputStrides_.emplace_back();
      isOutputScalar_.push_back(true);
      bufs_.erase(output);
      continue;
    }
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1791-1818
```cpp
    const auto& tt = output->type()->expect<TensorType>();
    if (has_symbolic_shapes_) {
      auto sizes = sizesFromSymbolicShape(tt->symbolic_sizes());
      tensorOutputSymbolicSizes_.push_back(sizes);
      TORCH_INTERNAL_ASSERT(symbolic_strides_.count(output));
      auto stride_desc_vec = symbolic_strides_[output];
      TORCH_INTERNAL_ASSERT(stride_desc_vec.size() == 1);
      auto stride_desc = stride_desc_vec[0];
      tensorOutputStrideDesc_.push_back(stride_desc);
      Tensor properly_strided_output =
          convertSymbolicOutputToCorrectStrides(output);
      if (properly_strided_output.stmt()) {
        block->append_stmt(properly_strided_output.stmt());
      }
      bufs_[output] = properly_strided_output.buf();
    } else {
      // The "strided" tensor will be incorrect if used in NNC,
      // since NNC views it as contiguous. Only convert it to the right
      // strides at the end of the kernel (if already contiguous it's a no-op)
      Tensor properly_strided_output =
          convertStaticShapeOutputToCorrectStrides(output);
      if (properly_strided_output.stmt()) {
        block->append_stmt(properly_strided_output.stmt());
      }
      bufs_[output] = properly_strided_output.buf();
      auto sizes = *tt->sizes().concrete_sizes();
      tensorOutputSizes_.push_back(sizes);
      auto strides = tt->strides().concrete_sizes();
```
- **EN**: This chunk continues `malformed_input` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `malformed_input`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1820-1848
```cpp
      // If the tensor is not dense or overlaps, we have
      // no way of matching the profiled striding
      if (strides && denseAndNonOverlapping(sizes, *strides)) {
        tensorOutputStrides_.push_back(*strides);
      } else {
        tensorOutputStrides_.push_back(TensorType::contiguousStridesOf(sizes));
      }
    }

    bufOutputs_.insert(bufs_.at(output));
    bufsToBeParallelized_.insert(bufs_.at(output));
    bufferArgs_.emplace_back(BufHandle(bufs_.at(output)));
    tensorOutputTensorOptions_.emplace_back(
        c10::TensorOptions(tensorType(bufs_.at(output))).device(device_));
    isOutputScalar_.push_back(false);
    bufs_.erase(output);
  }

  BackendType backendType = inferBackendTypeFromDevice(device_);
  stmt_ = transformLoops(backendType, block);

  for (const auto& c : constants_) {
    bufferArgs_.emplace_back(BufHandle(c.buf));
  }

  if (has_symbolic_shapes_) {
    tensorOutputSizes_.resize(bufOutputs_.size());
    tensorOutputStrides_.resize(bufOutputs_.size());
  }
```
- **EN**: This chunk continues `malformed_input` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `malformed_input`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1850-1882
```cpp
  // Generate code.
  codegen_ = CreateCodeGen(
      getCodeGenName(backendType),
      stmt_,
      bufferArgs_,
      device_,
      kernel_func_name_);
}

void TensorExprKernel::recompile() {
  codegen_ = CreateCodeGen(
      "llvm_codegen", stmt_, bufferArgs_, device_, kernel_func_name_);
}

TensorExprKernel::TensorExprKernel(
    const std::shared_ptr<Graph>& subgraph,
    std::string kernel_func_name,
    std::unordered_map<c10::Symbol, NNCLoweringFunction> custom_lowerings,
    std::vector<int64_t> symbolic_shape_inputs,
    bool pre_alloc /*= false*/,
    std::unordered_map<
        const torch::jit::Value*,
        std::vector<torch::jit::StrideInput>> symbolic_strides)
    : graph_(subgraph),
      code_(subgraph, ""),
      symbolic_shape_inputs_(std::move(symbolic_shape_inputs)),
      custom_lowerings_(std::move(custom_lowerings)),
      pre_alloc_(pre_alloc),
      kernel_func_name_(std::move(kernel_func_name)),
      symbolic_strides_(std::move(symbolic_strides)) {
  optimizeOwningGraph();
  // NOLINTNEXTLINE(cppcoreguidelines-prefer-member-initializer)
  allow_fallback_ = fallbackAllowed();
```
- **EN**: This chunk defines `recompile`, which lowers higher-level IR into a backend-specific executable form.
- **CN**: 这一段定义了 `recompile`，其作用是把高层 IR 降级为后端特定的可执行形式。

### Lines 1884-1913
```cpp
  if (!allow_fallback_) {
    compile();
    return;
  }

  use_fallback_ = fallbackEnforced();
  if (use_fallback_) {
    return;
  }

  try {
    compile();
  } catch (...) {
    use_fallback_ = true;
  }
}

void TensorExprKernel::run(Stack& stack) const {
  if (!use_fallback_ && !allow_fallback_) {
    runKernel(stack);
  } else if (!use_fallback_ && allow_fallback_) {
    try {
      runKernel(stack);
    } catch (...) {
      fallback(stack);
    }
  } else {
    fallback(stack);
  }
}
```
- **EN**: This chunk defines `run`, which executes prepared logic against runtime values or IR state. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `run`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1915-1946
```cpp
void TensorExprKernel::getStaticOutputSizesAndStrides(
    const at::ArrayRef<IValue>& inputs,
    std::vector<std::vector<int64_t>>* sizes,
    std::vector<std::vector<int64_t>>* strides) const {
  TORCH_INTERNAL_ASSERT(has_symbolic_shapes_);
  // If there are symbolic shapes, then the output tensor size wouldn't have
  // been computed at compile time. That has to be done here by using the
  // symbolic shape input params passed in to this call.
  TORCH_INTERNAL_ASSERT(
      tensorOutputSymbolicSizes_.size() == bufOutputs_.size());

  TORCH_INTERNAL_ASSERT(sizes);
  TORCH_INTERNAL_ASSERT(strides);
  *sizes = tensorOutputSizes_;
  *strides = tensorOutputStrides_;
  auto& static_sizes = *sizes;
  auto& static_strides = *strides;
  for (size_t i = 0, e = bufOutputs_.size(); i < e; ++i) {
    static_sizes[i].clear();
    for (auto t : tensorOutputSymbolicSizes_[i]) {
      if (t.AsNode<LongImm>()) {
        static_sizes[i].emplace_back(immediateAs<int64_t>(t.node()));
      } else {
        auto input_pos = shapeSymbolInputPos_.at(t.node());
        TORCH_INTERNAL_ASSERT(input_pos < inputs.size());
        TORCH_INTERNAL_ASSERT(inputs[input_pos].isInt());
        static_sizes[i].emplace_back(inputs[input_pos].toInt());
      }
    }

    if (tensorOutputStrideDesc_[i] == torch::jit::StrideInput::TENSOR_CONT) {
      static_strides[i] = TensorType::contiguousStridesOf(static_sizes[i]);
```
- **EN**: This chunk defines `getStaticOutputSizesAndStrides`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `getStaticOutputSizesAndStrides`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1948-1980
```cpp
    } else if (
        tensorOutputStrideDesc_[i] ==
        torch::jit::StrideInput::TENSOR_CONT_CHANNELS_LAST) {
      static_strides[i] = at::get_channels_last_strides_2d(static_sizes[i]);

    } else {
      std::string output_desc = toString(tensorOutputStrideDesc_[i]);
      TORCH_INTERNAL_ASSERT(
          false, "Expected contiguous or channels last, got ", output_desc);
    }
  }
}

std::vector<CodeGen::CallArg> TensorExprKernel::prepareRunArgs(
    const at::ArrayRef<IValue>& inputs,
    std::vector<at::Tensor>& outputs) const {
  // TODO: preallocate `runArgs` during compilation and fill in values where
  // possible (e.g. for constant tensors)
  std::vector<CodeGen::CallArg> runArgs;
  runArgs.reserve(
      inputs.size() + input_stride_args_.size() + bufOutputs_.size());

  for (auto& input : inputs) {
    if (input.isInt()) {
      runArgs.emplace_back(input.toInt());
    } else if (input.isBool()) {
      runArgs.emplace_back(input.toBool());
    } else if (input.isDouble()) {
      runArgs.emplace_back(input.toDouble());
    } else if (input.isTensor()) {
      runArgs.emplace_back(input.toTensor().data_ptr());
    }
  }
```
- **EN**: This chunk defines `prepareRunArgs`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `prepareRunArgs`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1982-2017
```cpp
  if (has_symbolic_shapes_) {
    std::vector<std::vector<int64_t>> static_sizes;
    std::vector<std::vector<int64_t>> static_strides;
    getStaticOutputSizesAndStrides(inputs, &static_sizes, &static_strides);

    // add stride args
    for (const auto& input_stride_arg : input_stride_args_) {
      runArgs.emplace_back(
          inputs[input_stride_arg.first].toTensor().strides().at(
              input_stride_arg.second));
    }

    for (size_t i = 0, e = bufOutputs_.size(); i < e; ++i) {
      auto const& opts = tensorOutputTensorOptions_[i];
      outputs.emplace_back(codegen_->empty_strided(
          static_sizes[i],
          static_strides[i],
          opts.dtype,
          opts.layout,
          opts.device,
          opts.pinned_memory));
      runArgs.emplace_back(outputs.back().data_ptr());
    }
  } else {
    for (size_t i = 0, e = bufOutputs_.size(); i < e; ++i) {
      auto const& opts = tensorOutputTensorOptions_[i];
      outputs.emplace_back(codegen_->empty_strided(
          tensorOutputSizes_[i],
          tensorOutputStrides_[i],
          opts.dtype,
          opts.layout,
          opts.device,
          opts.pinned_memory));
      runArgs.emplace_back(outputs.back().data_ptr());
    }
  }
```
- **EN**: This chunk continues `prepareRunArgs` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `prepareRunArgs`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 2019-2053
```cpp
  for (const auto& c : constants_) {
    runArgs.emplace_back(c.ptr);
  }

  return runArgs;
}

StmtPtr TensorExprKernel::getCodeGenStmt() {
  return codegen_->stmt();
}

void TensorExprKernel::runKernel(Stack& stack) const {
  // Set up arguments (inputs, then outputs) for kernel call.
  auto inputs = last(stack, nInputs_);
  std::vector<at::Tensor> outputs;

  std::vector<CodeGen::CallArg> runArgs = prepareRunArgs(inputs, outputs);

  // Call the kernel.
  codegen_->call(runArgs);

  // Update the stack.
  drop(stack, nInputs_);

  int64_t idx = 0;
  for (auto& o : outputs) {
    if (isOutputScalar_[idx++]) {
      // Scalar outputs are returned as 0-dim tensors, we need to extract the
      // scalar value from them
      push_one(stack, o.item());
    } else {
      push_one(stack, std::move(o));
    }
  }
}
```
- **EN**: This chunk defines `runKernel`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `runKernel`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 2055-2081
```cpp
void TensorExprKernel::runFast(
    const std::vector<void*>& inputs,
    const std::vector<void*>& outputs) const {
  std::vector<void*> args(inputs);
  args.reserve(inputs.size() + outputs.size() + constants_.size());
  args.insert(args.end(), outputs.begin(), outputs.end());

  // TODO: we can consider preallocating and pre-filling the args vector.
  for (const auto& c : constants_) {
    args.push_back(c.ptr);
  }

  // Call the kernel.
  codegen_->call_raw(args);
}

void TensorExprKernel::runWithAllocatedOutputs(Stack& stack) const {
  TORCH_INTERNAL_ASSERT(
      device_ == at::kCPU,
      "Pre-allocated output tensors are supported only on CPUs.");
  std::vector<void*> args;
  args.reserve(nInputs_ + nOutputs_ + constants_.size());

  // stack has inputs on the top and outputs right below them.
  auto stack_ivals = last(stack, nOutputs_ + nInputs_);
  auto stack_outputs = stack_ivals.slice(0, nOutputs_);
  auto stack_inputs = stack_ivals.slice(nOutputs_);
```
- **EN**: This chunk defines `runWithAllocatedOutputs`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `runWithAllocatedOutputs`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 2083-2111
```cpp
  std::vector<int64_t> int_inputs(nInputs_);
  for (auto i : c10::irange(nInputs_)) {
    const auto& inp = stack_inputs[i];
    if (inp.isInt()) {
      int_inputs[i] = inp.toInt();
      args.emplace_back(&int_inputs[i]);
    } else if (inp.isTensor()) {
      args.emplace_back(inp.toTensor().data_ptr());
    } else {
      TORCH_INTERNAL_ASSERT(
          false, "Unhandled input type while calling TensorExprKernel");
    }
  }

  std::vector<int64_t> stride_values(input_stride_args_.size());
  if (has_symbolic_shapes_) {
    std::vector<std::vector<int64_t>> static_sizes;
    std::vector<std::vector<int64_t>> static_strides;
    getStaticOutputSizesAndStrides(
        stack_inputs, &static_sizes, &static_strides);

    // add stride args
    for (auto idx : c10::irange(input_stride_args_.size())) {
      const auto& input_stride_arg = input_stride_args_[idx];
      stride_values[idx] =
          stack_inputs[input_stride_arg.first].toTensor().strides().at(
              input_stride_arg.second);
      args.emplace_back(&stride_values[idx]);
    }
```
- **EN**: This chunk defines `stride_values`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `stride_values`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 2113-2138
```cpp
    TORCH_INTERNAL_ASSERT(
        nOutputs_ == static_cast<int64_t>(bufOutputs_.size()));
    for (size_t i = 0, e = bufOutputs_.size(); i < e; ++i) {
      auto& out = stack_outputs[i].toTensor();
      // This has only been tested on CPUs.
      // TODO: Test on GPUs.
      out.resize_(static_sizes[i]);
      args.emplace_back(out.data_ptr());
    }
  } else {
    for (auto i : c10::irange(nOutputs_)) {
      args.emplace_back(stack_outputs[i].toTensor().data_ptr());
    }
  }

  for (const auto& c : constants_) {
    args.emplace_back(c.ptr);
  }

  // Call the kernel.
  codegen_->call_raw(args);

  // Remove the inputs from the stack. The outputs are already below the inputs
  // in the stack.
  drop(stack, nInputs_);
}
```
- **EN**: This chunk continues `stride_values` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `stride_values`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **buildErrorMessage**
  - EN: `buildErrorMessage` is a central symbol declared or implemented in this file.
  - CN: `buildErrorMessage` 是本文件声明或实现的核心符号。
- **setFallbackAllowed**
  - EN: `setFallbackAllowed` is a central symbol declared or implemented in this file.
  - CN: `setFallbackAllowed` 是本文件声明或实现的核心符号。
- **TorchScript runtime**
  - EN: Coordinates execution plans, stacks, and operator dispatch for scripted programs.
  - CN: 协调脚本程序的执行计划、栈以及算子分发。
- **Graph IR**
  - EN: Works on graph, node, and block structures that represent TorchScript programs.
  - CN: 围绕表示 TorchScript 程序的 graph、node 与 block 结构工作。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/kernel.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/graph_rewrite_helper.h`, `torch/csrc/jit/passes/mkldnn_rewrite.h`, `torch/csrc/jit/passes/symbolic_shape_runtime_fusion.h`, `torch/csrc/jit/tensorexpr/analysis.h`, `torch/csrc/jit/tensorexpr/expr.h`, `torch/csrc/jit/tensorexpr/graph_opt.h`, `torch/csrc/jit/tensorexpr/ir_printer.h`, `torch/csrc/jit/tensorexpr/ir_simplifier.h`, `torch/csrc/jit/tensorexpr/loopnest.h`, `torch/csrc/jit/tensorexpr/loopnest_randomization.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `ATen/ExpandUtils.h`, `ATen/Parallel.h`, `ATen/TensorGeometry.h`, `c10/core/ScalarTypeToTypeMeta.h`, `c10/util/irange.h`
- **Standard library / 标准库**: `utility`
- **Primary symbols in this file / 本文件核心符号**: `buildErrorMessage`, `setFallbackAllowed`, `fallbackAllowed`, `fallbackEnforced`, `randomTransformsRequested`, `dontUseLLVMFlag`, `getTECudaPointwiseLoopLevels`, `getTECudaPointwiseBlockCount`
- **Note / 说明**: 19 direct includes were detected; only the first few are listed above for readability. / 检测到 19 个直接包含，为便于阅读这里只列出前若干项。
