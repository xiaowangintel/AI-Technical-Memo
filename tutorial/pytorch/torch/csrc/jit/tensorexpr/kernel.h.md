# kernel.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/kernel.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements the TorchScript interpreter, bytecode execution, frame management, and graph preprocessing.
- **Purpose (CN)**: 实现 TorchScript 解释器、字节码执行、栈帧管理以及图预处理。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
#pragma once

#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/passes/symbolic_shape_runtime_fusion.h>
#include <torch/csrc/jit/passes/utils/subgraph_utils.h>
#include <torch/csrc/jit/runtime/interpreter.h>
#include <torch/csrc/jit/tensorexpr/analysis.h>
#include <torch/csrc/jit/tensorexpr/codegen.h>
#include <torch/csrc/jit/tensorexpr/lowerings.h>
#include <torch/csrc/jit/tensorexpr/tensor.h>

namespace torch::jit::tensorexpr {
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/ir/ir.h, torch/csrc/jit/passes/symbolic_shape_runtime_fusion.h, torch/csrc/jit/passes/utils/subgraph_utils.h, and 5 more. The preprocessor guard keeps declarations single-instanced when the header is included transitively. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/ir/ir.h、torch/csrc/jit/passes/symbolic_shape_runtime_fusion.h、torch/csrc/jit/passes/utils/subgraph_utils.h 等共 8 项。 预处理器保护用于避免头文件在传递包含时被重复展开。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。

### Lines 14-29
```cpp
struct SmallSizeTPairHash {
 public:
  std::size_t operator()(const std::pair<size_t, size_t>& x) const {
    // hashing input index and then dim index
    return x.first * 128 + x.second;
  }
};

// Returns true if the TE fuser supports this conv2d.
bool conv2dIsSupportedJit(const Node* node);
// Returns true if the TE fuser supports this conv2d with mkldnn prepacked conv.
bool mkldnnPrepackedConvIsSupportedJit(const Node* node);
// Returns true if the TE _convolution node is Conv2d.
bool isConv2d(const Node* node);
// Returns true if the TE fuser supports this matmul.
bool matmulIsSupported(const Node* node);
```
- **EN**: It introduces or extends SmallSizeTPairHash, which define the primary data structures or interfaces for this portion of the file. This chunk defines `matmulIsSupported`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Registration-style entries bind symbolic schemas or library fragments to concrete runtime behavior. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 SmallSizeTPairHash，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `matmulIsSupported`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 这类注册条目把符号化 schema 或库片段绑定到具体的运行时行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 30-40
```cpp
template <typename T>
inline std::vector<int64_t> bufferSizes(const T& t) {
  std::vector<int64_t> sizes;
  for (size_t i = 0; i < t->ndim(); i++) {
    sizes.push_back(*intValue(t->dim(i)));
  }
  return sizes;
}

// Get the dimensions of a value.
std::vector<ExprHandle> valueShape(const ArgValue& v);
```
- **EN**: This chunk defines `valueShape`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `valueShape`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 42-56
```cpp
// If v is a tensor, broadcast it to match the shape of axes, or return
// directly if v is a constant.
ExprHandle tensorOrConstant(
    const ArgValue& v,
    const std::vector<ExprHandle>& axes);

int64_t normalizeAndCheckIndex(int64_t idx, int64_t list_size);

ExprHandle broadcast(const BufHandle& b, const std::vector<ExprHandle>& axes);

ExprHandle constant(const ArgValue& v);

std::vector<ExprHandle> computeIndicesToBroadcast(
    const std::vector<ExprHandle>& outputAxes,
    const std::vector<ExprHandle>& inputSizes);
```
- **EN**: This chunk declares `computeIndicesToBroadcast`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段声明了 `computeIndicesToBroadcast`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 58-73
```cpp
inline std::string getArgValueName(const ArgValue& a) {
  if (std::holds_alternative<tensorexpr::BufHandle>(a)) {
    return "BufHandle";
  } else if (std::holds_alternative<tensorexpr::VarHandle>(a)) {
    return "VarHandle";
  } else if (std::holds_alternative<double>(a)) {
    return "double";
  } else if (std::holds_alternative<int64_t>(a)) {
    return "int64_t";
  } else if (std::holds_alternative<bool>(a)) {
    return "bool";
  } else if (std::holds_alternative<BufList>(a)) {
    return "BufList";
  } else if (std::holds_alternative<DoubleList>(a)) {
    return "DoubleList";
  } else if (std::holds_alternative<IntList>(a)) {
```
- **EN**: This chunk defines `getArgValueName`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getArgValueName`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 74-89
```cpp
    return "IntList";
  } else if (std::holds_alternative<ArgNone>(a)) {
    return "None";
  } else {
    throw std::runtime_error("ArgValue type not handled in string conversion");
  }
}

template <class T>
std::vector<T> convertVecArgValue(const std::vector<ArgValue>& v) {
  std::vector<T> res;
  for (auto& x : v) {
    auto val = std::get_if<T>(&x);
    if (val) {
      res.push_back(*val);
    } else {
```
- **EN**: This chunk defines `convertVecArgValue`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `convertVecArgValue`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 90-105
```cpp
      throw std::runtime_error(
          "vector type not homogeneous - found " + getArgValueName(x) +
          ", expected " + getArgValueName(v[0]));
    }
  }
  return res;
}

class TORCH_API TensorExprKernel {
  struct ConstantDescr {
    BufPtr buf;
    // Only one of ptr and node is used at a time
    // 1) ptr for the constant tensors
    // 2) node for the constant custom class objects
    void* ptr = nullptr;
    Node* node = nullptr;
```
- **EN**: It introduces or extends TORCH_API, ConstantDescr, which define the primary data structures or interfaces for this portion of the file. This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 TORCH_API、ConstantDescr，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 106-121
```cpp
  };

 public:
  // Constructor Params:
  //  * subgraph
  //      - the graph that needs to be compiled.
  //  * kernel_func_name
  //      - the name that should be used for the generated kernel.
  //  * custom_lowerings
  //      - map that represents custom lowering definitions for a set of ops.
  //  * symbolic_shape_inputs
  //      - a list of symbolic graph inputs that represent the symbolic dims of
  //        the input tensors.
  //  * pre_alloc
  //      - a flag to control pre-allocation of buffers.
  explicit TensorExprKernel(
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。

### Lines 122-137
```cpp
      const std::shared_ptr<Graph>& subgraph,
      std::string kernel_func_name,
      std::unordered_map<c10::Symbol, NNCLoweringFunction> custom_lowerings =
          {},
      std::vector<int64_t> symbolic_shape_inputs = {},
      bool pre_alloc = false,
      std::unordered_map<
          const torch::jit::Value*,
          std::vector<torch::jit::StrideInput>> symbolic_strides = {});

  explicit TensorExprKernel(
      const std::shared_ptr<Graph>& subgraph,
      std::unordered_map<c10::Symbol, NNCLoweringFunction> custom_lowerings =
          {},
      std::vector<int64_t> symbolic_shape_inputs = {},
      bool pre_alloc = false,
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。

### Lines 138-147
```cpp
      std::unordered_map<
          const torch::jit::Value*,
          std::vector<torch::jit::StrideInput>> symbolic_strides = {})
      : TensorExprKernel(
            subgraph,
            SubgraphUtils::generateNameForGraph(subgraph),
            std::move(custom_lowerings),
            std::move(symbolic_shape_inputs),
            pre_alloc,
            std::move(symbolic_strides)) {}
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。

### Lines 149-163
```cpp
  void run(Stack& stack) const;
  void runFast(
      const std::vector<void*>& inputs,
      const std::vector<void*>& outputs) const;
  // Expected format of stack:
  //  ... <outputs> <inputs>
  // i.e., output IValues must be below the input IValues in the stack.
  void runWithAllocatedOutputs(Stack& stack) const;

  void fallback(Stack& stack) const {
    InterpreterState(code_).run(stack);
  }
  void recompile();

  StmtPtr getCodeGenStmt();
```
- **EN**: This chunk defines `getCodeGenStmt`, which lowers higher-level IR into a backend-specific executable form.
- **CN**: 这一段定义了 `getCodeGenStmt`，其作用是把高层 IR 降级为后端特定的可执行形式。

### Lines 165-179
```cpp
  std::string getCodeText(const std::string& attr = "") {
    return codegen_->getCodeText(attr);
  }

  const std::shared_ptr<Graph> graph() {
    return graph_;
  }

  const std::vector<ConstantDescr>& getConstantDescriptors() const {
    return constants_;
  }

  const std::vector<CodeGen::BufferArg>& getBufferArgs() const {
    return bufferArgs_;
  }
```
- **EN**: This chunk defines `getBufferArgs`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getBufferArgs`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 181-196
```cpp
  const std::string& getKernelName() const {
    return (codegen_ ? codegen_->kernel_func_name() : kernel_func_name_);
  }

  const std::vector<int64_t>& getSymbolicShapeInputs() const {
    return symbolic_shape_inputs_;
  }

 private:
  enum BackendType {
    kUninitialized,
    kSimpleIREval,
    kLLVMCodeGen,
    kCudaCodeGen,
    kBlockCodeGen,
  };
```
- **EN**: It introduces or extends BackendType, which define the primary data structures or interfaces for this portion of the file. This chunk defines `getSymbolicShapeInputs`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 BackendType，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `getSymbolicShapeInputs`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 198-207
```cpp
  enum MemoryLayoutPolicy {
    kContiguous,
    kChannelsLastNdContiguous,
  };

  void compile();
  void genInputDebugNames();
  void runKernel(Stack& stack) const;

  std::vector<ExprHandle> sizesForValue(const torch::jit::Value* v);
```
- **EN**: It introduces or extends MemoryLayoutPolicy, which define the primary data structures or interfaces for this portion of the file. This chunk defines `sizesForValue`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 它引入或扩展了 MemoryLayoutPolicy，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `sizesForValue`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 209-223
```cpp
  // These functions broadcast shape and also store a `hasBroadcast_` variable.
  std::vector<ExprHandle> broadcastShapesMut(
      const std::vector<ExprHandle>& a,
      const std::vector<ExprHandle>& b);
  std::vector<ExprHandle> broadcastShapesMut(
      std::vector<std::vector<ExprHandle>> shapes);

  ArgValue toArg(const torch::jit::Value* v) const;
  ExprHandle constant(const torch::jit::Value* v);

  Tensor computeValue(const torch::jit::Value* v);

  void bindConstant(const torch::jit::Value* v);

  StmtPtr transformLoops(BackendType backendType, StmtPtr st);
```
- **EN**: This chunk declares `transformLoops`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段声明了 `transformLoops`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 225-238
```cpp
  std::string getCodeGenName(BackendType backendType);

  void getStaticOutputSizesAndStrides(
      const at::ArrayRef<IValue>& inputs,
      std::vector<std::vector<int64_t>>* static_sizes,
      std::vector<std::vector<int64_t>>* static_strides) const;

  std::vector<CodeGen::CallArg> prepareRunArgs(
      const at::ArrayRef<IValue>& inputs,
      std::vector<at::Tensor>& outputs) const;
  BackendType inferBackendTypeFromDevice(at::Device device);

  Tensor bindInput(const torch::jit::Value* input);
  BlockPtr bindAllInputs();
```
- **EN**: This chunk declares `bindAllInputs`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段声明了 `bindAllInputs`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 240-251
```cpp
  // Deduce the memory layout policy to be propagated within
  // NNC fusion group. The memory layout policy could be `kContiguous`
  // or `kChannelsLastNdContiguous`.
  //    `kContiguous`: Always convert the non-contiguous input tensors and
  //        internal buffers to contiguous.
  //    `kChannelsLastNdContiguous`: Always convert the input tensors and
  //        internal buffers to channels-last contiguous.
  // Currently, the rule is simple.
  //    If all the input and out tensors of NNC fusion group are channels-last
  //    contiguous, the policy is `kChannelsLastNdContiguous`. Otherwise, it
  //    is always `kContiguous`.
  void deduceMemoryLayoutPolicy();
```
- **EN**: This chunk declares `deduceMemoryLayoutPolicy`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段声明了 `deduceMemoryLayoutPolicy`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 253-265
```cpp
  Tensor convertSymbolicOutputToCorrectStrides(torch::jit::Value* v);
  Tensor convertStaticShapeOutputToCorrectStrides(torch::jit::Value* v);
  Tensor convertSymbolicOutputToCorrectStrides(
      const std::vector<ExprHandle>& sizes,
      const std::vector<size_t>& sorted_stride_indices_descending,
      const std::vector<ExprPtr>& strides,
      BufPtr& buf);

  NNCLoweringFunction getCustomLoweringFor(c10::Symbol op) const;
  std::unordered_map<c10::Symbol, NNCLoweringFunction> getCustomLowerings()
      const {
    return custom_lowerings_;
  }
```
- **EN**: This chunk defines `getCustomLoweringFor`, which lowers higher-level IR into a backend-specific executable form. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getCustomLoweringFor`，其作用是把高层 IR 降级为后端特定的可执行形式。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 267-278
```cpp
  // Allocate memory for intermediate buffers at compile time.
  // Specifically, we pre-allocate memory for intermediate buffers with static
  // size and manage these buffers in the way we manage JIT constant tensors:
  // push the buf args into the stack so NNC IR can access them at runtime.
  std::vector<BufPtr> preAllocIntermediateBufs(
      const std::vector<BufPtr>& interm_bufs);

  struct UnpackedTensorOptions {
    std::optional<c10::ScalarType> dtype;
    std::optional<c10::Layout> layout;
    std::optional<c10::Device> device;
    std::optional<bool> pinned_memory;
```
- **EN**: It introduces or extends UnpackedTensorOptions, which define the primary data structures or interfaces for this portion of the file. This chunk defines `preAllocIntermediateBufs`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 它引入或扩展了 UnpackedTensorOptions，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `preAllocIntermediateBufs`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 280-295
```cpp
    UnpackedTensorOptions(const c10::TensorOptions& opts)
        : dtype(c10::optTypeMetaToScalarType(opts.dtype_opt())),
          layout(opts.layout_opt()),
          device(opts.device_opt()),
          pinned_memory(opts.pinned_memory_opt()) {}
  };

  ExprHandle getVarForShape(const c10::ShapeSymbol& ss);
  std::vector<ExprHandle> computeInputTensorDims(
      const torch::jit::Value* input);
  ExprHandle getStrideArg(size_t tensor_input, size_t stride_index);
  std::vector<ExprHandle> sizesFromSymbolicShape(
      const c10::SymbolicShape& shape);
  std::vector<ExprHandle> getInputStrides(
      const torch::jit::Value* input,
      const std::vector<ExprHandle>& inputTensorDims);
```
- **EN**: This chunk defines `getInputStrides`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段定义了 `getInputStrides`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 296-311
```cpp
  std::vector<torch::jit::StrideInput>& getSymbolicStrideDesc(
      const torch::jit::Value* value);

  // Apply the optimizations to the graph owned by the current fusion group,
  // like concatenation optimization, post-op fusion, and some other graph-level
  // optimizations.
  void optimizeOwningGraph();

  int64_t nInputs_ = 0;
  int64_t nOutputs_ = 0;
  std::vector<CodeGen::BufferArg> bufferArgs_;
  std::vector<std::vector<int64_t>> tensorOutputSizes_;
  std::vector<std::vector<int64_t>> tensorOutputStrides_;
  std::vector<torch::jit::StrideInput> tensorOutputStrideDesc_;
  std::vector<bool> isOutputScalar_;
  std::vector<UnpackedTensorOptions> tensorOutputTensorOptions_;
```
- **EN**: This chunk declares `optimizeOwningGraph`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段声明了 `optimizeOwningGraph`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 312-326
```cpp
  std::unordered_set<BufPtr> bufOutputs_;
  std::unordered_set<BufPtr> bufsToBeParallelized_;
  std::unordered_map<const torch::jit::Value*, BufPtr> bufs_;
  std::unordered_map<const torch::jit::Value*, VarHandle> scalars_;
  std::unordered_map<const torch::jit::Value*, std::string> input_name_map_;
  std::unique_ptr<CodeGen> codegen_;
  at::Device device_ = at::kCPU;
  std::shared_ptr<Graph> graph_;
  Code code_;
  bool allow_fallback_{false};
  bool use_fallback_{false};
  bool hasRandom_{false};
  bool hasBroadcast_{false};
  std::unordered_map<const torch::jit::Value*, std::vector<ExprHandle>>
      known_sizes_;
```
- **EN**: This chunk continues `optimizeOwningGraph` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `optimizeOwningGraph`，进一步展开其内部控制流或数据流转。

### Lines 328-339
```cpp
  std::vector<std::vector<ExprHandle>> tensorOutputSymbolicSizes_;
  // A map from ShapeSymbol.value() to the corresponding Var.
  std::unordered_map<int64_t, VarHandle> shapeSymbolToVar_;
  std::unordered_map<ExprPtr, size_t> shapeSymbolInputPos_;
  // List of values corresponding to the ShapeSymbols that are inputs to
  // kernel being compiled. The order of these values correspond to the order
  // of the symbolic inputs at the end of the list of inputs to the kernel.
  std::vector<int64_t> symbolic_shape_inputs_;
  bool has_symbolic_shapes_{false};

  std::vector<at::Tensor> unpacked_constant_tensors_;
  std::vector<ConstantDescr> constants_;
```
- **EN**: This chunk continues `optimizeOwningGraph` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `optimizeOwningGraph`，进一步展开其内部控制流或数据流转。

### Lines 341-355
```cpp
  std::unordered_map<c10::Symbol, NNCLoweringFunction> custom_lowerings_;
  StmtPtr stmt_ = nullptr;
  bool pre_alloc_{false};
  std::string kernel_func_name_;

  // index of stack, stride index of tensor that will be appended as a codegen
  // arg
  std::vector<std::pair<size_t, size_t>> input_stride_args_;
  // map from <input index, tensor dimension> to stride as arg VarHandle
  std::unordered_map<std::pair<size_t, size_t>, VarHandle, SmallSizeTPairHash>
      strideArgToVar_;
  std::unordered_map<
      const torch::jit::Value*,
      std::vector<torch::jit::StrideInput>>
      symbolic_strides_;
```
- **EN**: This chunk continues `optimizeOwningGraph` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `optimizeOwningGraph`，进一步展开其内部控制流或数据流转。

### Lines 357-369
```cpp
  // Memory layout to be propagated with fusion group
  MemoryLayoutPolicy memory_layout_policy_ = MemoryLayoutPolicy::kContiguous;
};

TORCH_API int& getTECudaPointwiseLoopLevels();
TORCH_API int& getTECudaPointwiseBlockCount();
TORCH_API int& getTECudaPointwiseBlockSize();
TORCH_API bool& getTEGenerateBlockCode();
TORCH_API bool& getTEMustUseLLVMOnCPU();
TORCH_API bool fallbackAllowed();
TORCH_API bool setFallbackAllowed(bool value);
TORCH_API bool& getCatWoConditionals();
TORCH_API bool& getOptConditionals();
```
- **EN**: This chunk declares `getOptConditionals`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段声明了 `getOptConditionals`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 371-378
```cpp
TORCH_API std::optional<at::Device> pickDeviceType(
    const at::ArrayRef<torch::jit::Value*>& inputs);

bool isContiguous(
    const torch::jit::Value* v,
    at::MemoryFormat memory_format = at::MemoryFormat::Contiguous);

} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk declares `isContiguous`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段声明了 `isContiguous`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **SmallSizeTPairHash**
  - EN: `SmallSizeTPairHash` is a central symbol declared or implemented in this file.
  - CN: `SmallSizeTPairHash` 是本文件声明或实现的核心符号。
- **TORCH_API**
  - EN: `TORCH_API` is a central symbol declared or implemented in this file.
  - CN: `TORCH_API` 是本文件声明或实现的核心符号。
- **TorchScript runtime**
  - EN: Coordinates execution plans, stacks, and operator dispatch for scripted programs.
  - CN: 协调脚本程序的执行计划、栈以及算子分发。
- **Graph IR**
  - EN: Works on graph, node, and block structures that represent TorchScript programs.
  - CN: 围绕表示 TorchScript 程序的 graph、node 与 block 结构工作。
- **IValue data model**
  - EN: Moves values through generic boxed runtime containers used by TorchScript.
  - CN: 通过 TorchScript 使用的通用装箱运行时容器传递数值。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/passes/symbolic_shape_runtime_fusion.h`, `torch/csrc/jit/passes/utils/subgraph_utils.h`, `torch/csrc/jit/runtime/interpreter.h`, `torch/csrc/jit/tensorexpr/analysis.h`, `torch/csrc/jit/tensorexpr/codegen.h`, `torch/csrc/jit/tensorexpr/lowerings.h`, `torch/csrc/jit/tensorexpr/tensor.h`
- **Primary symbols in this file / 本文件核心符号**: `SmallSizeTPairHash`, `TORCH_API`, `ConstantDescr`, `BackendType`, `MemoryLayoutPolicy`, `conv2dIsSupportedJit`, `mkldnnPrepackedConvIsSupportedJit`, `isConv2d`
