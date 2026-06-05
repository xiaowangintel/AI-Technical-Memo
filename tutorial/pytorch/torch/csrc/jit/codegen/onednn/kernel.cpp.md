# kernel.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/codegen/onednn/kernel.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Contains code-generation logic or generated declarations that synthesize JIT-related bindings, schemas, or helper code. This specific file centers on `kernel.cpp`. Generation of declarations, registrations, or helper code is part of the responsibility.
- **Purpose (CN)**: 包含代码生成逻辑或生成后的声明，用于产出 JIT 相关绑定、模式或辅助代码。 该文件具体围绕 `kernel.cpp` 展开。 其职责包含声明、注册或辅助代码的生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

```cpp
#include <torch/csrc/jit/codegen/onednn/graph_helper.h>
#include <torch/csrc/jit/codegen/onednn/kernel.h>

#include <torch/csrc/jit/jit_log.h>

namespace torch::jit::fuser::onednn {

using namespace dnnl::graph;
using data_type = dnnl::graph::logical_tensor::data_type;

LlgaKernel::LlgaKernel(const Node* fusionNode)
    : fusionNode_(fusionNode),
      graph_(fusionNode->g(attr::Subgraph)),
      nGraphInputs_(graph_->inputs().size()),
      nOutputs_(graph_->outputs().size()),
      debugName_(genDebugName()) {
  // TODO: This is a workaround to recreate the partitions here.
  // The ideal way is to use the partition serialization API (not available from
  // LLGA now) to carry a serialized string representation from graph rewrite
  // and deserialize it here.
  auto llgaGraphHelper = LlgaGraphHelper(graph_);
  auto partitions = llgaGraphHelper.getPartitions();
  tensorIdToValue_ = llgaGraphHelper.getTensorIdToValue();
  TORCH_CHECK(
      partitions.size() == 1,
      "LLGA subgraph should contain only one partition");
  partition_ = partitions[0];
  nPartitionInputs_ = partition_.get_input_ports().size();
```

- **EN:** It enters or references namespace scopes such as torch::jit::fuser::onednn, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit::fuser::onednn 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Alias declarations such as data_type simplify later API usage.
- **CN:** data_type 等别名声明简化了后续 API 的使用。
- **EN:** Important callable entry points in this range include LlgaKernel, TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 LlgaKernel, TORCH_CHECK。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。

### Lines 29-56 / 第 29-56 行

```cpp
#ifdef GRAPH_DEBUG_ENABLED
  GRAPH_DEBUG("Initialized ", debugName(), '\n', graph_->toString());
#endif
}

bool LlgaKernel::useOpaqueLayout(size_t offset) const {
  return LlgaNodeWrapper(fusionNode_).useOpaqueLayout(offset);
}

void LlgaKernel::initializeConstantInputs() {
  for (auto& lt : partition_.get_input_ports()) {
    auto inputId = lt.get_id();
    if (initializedInputIds_.find(inputId) == initializedInputIds_.end()) {
      TORCH_CHECK(
          tensorIdToValue_.count(inputId) > 0,
          "inputs with inputId ",
          inputId,
          " is missing");
      auto* value = tensorIdToValue_[inputId];

      TORCH_CHECK(
          value->node()->kind() == prim::Constant &&
              value->type()->cast<TensorType>(),
          "inputs with inputId ",
          inputId,
          " should be a Constant tensor");
      constantValues_.emplace_back(value);

```

- **EN:** Important callable entry points in this range include GRAPH_DEBUG, useOpaqueLayout, LlgaNodeWrapper, initializeConstantInputs, TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 GRAPH_DEBUG, useOpaqueLayout, LlgaNodeWrapper, initializeConstantInputs, TORCH_CHECK。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 57-84 / 第 57-84 行

```cpp
      auto const_tensor = toIValue(value)->toTensor();
      constantInputs_.emplace_back(const_tensor);
    }
  }
}

std::map<size_t, int64_t> LlgaKernel::initializeTensorIdToOccurrence() const {
  std::map<size_t, int64_t> tensorIdToOccurrence;
  for (auto& lt : partition_.get_input_ports()) {
    auto inputId = lt.get_id();
    std::map<size_t, int64_t>::iterator it(tensorIdToOccurrence.find(inputId));
    if (it != tensorIdToOccurrence.end()) {
      it->second++;
    } else {
      tensorIdToOccurrence[inputId] = 1;
    }
  }
  return tensorIdToOccurrence;
}

ArgSpecs LlgaKernel::initializeInputSpecs(const TensorArgs& inputs) {
  ArgSpecs inputSpecs;
  inputSpecs.reserve(nPartitionInputs_);
  GRAPH_DEBUG("Initializing graph input logical tensors");
  std::map<size_t, int64_t> tensorIdToOccurrence =
      initializeTensorIdToOccurrence();
  for (const auto i : c10::irange(nGraphInputs_)) {
    auto spec = ArgSpec(graph_->inputs()[i]).supplementTensorInfo(inputs[i]);
```

- **EN:** Important callable entry points in this range include initializeTensorIdToOccurrence, it, initializeInputSpecs, GRAPH_DEBUG.
- **CN:** 这一段的重要可调用入口包括 initializeTensorIdToOccurrence, it, initializeInputSpecs, GRAPH_DEBUG。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 85-112 / 第 85-112 行

```cpp
    initializedInputIds_.insert(spec.tid());
    int64_t occurrence = tensorIdToOccurrence[spec.tid()];
    inputSpecs.insert(inputSpecs.end(), occurrence, spec);
    runArgsIdx_.insert(runArgsIdx_.end(), occurrence, i);
  }
  GRAPH_DEBUG("Initializing constant input tensors");
  initializeConstantInputs();

  TORCH_CHECK(
      inputSpecs.size() + constantValues_.size() ==
          static_cast<size_t>(nPartitionInputs_),
      "Partition inputs are missing");
  GRAPH_DEBUG(
      "Concatenating constant input logical tensors to graph input "
      "logical tensors");
  for (Value* constant_value : constantValues_) {
    ArgSpec constantInputSpec(constant_value);
    inputSpecs.emplace_back(constantInputSpec);
    constantLogicalTensors_.emplace_back(constantInputSpec.logical_tensor());
  }
  return inputSpecs;
}

ArgSpecs LlgaKernel::initializeOutputSpecs() const {
  ArgSpecs outputSpecs;
  outputSpecs.reserve(nOutputs_);
  for (const auto i : c10::irange(nOutputs_)) {
    auto spec = ArgSpec(graph_->outputs()[i]);
```

- **EN:** Important callable entry points in this range include GRAPH_DEBUG, initializeConstantInputs, TORCH_CHECK, constantInputSpec, initializeOutputSpecs.
- **CN:** 这一段的重要可调用入口包括 GRAPH_DEBUG, initializeConstantInputs, TORCH_CHECK, constantInputSpec, initializeOutputSpecs。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 113-140 / 第 113-140 行

```cpp
    if (useOpaqueLayout(i)) {
      spec = spec.any();
    }
    outputSpecs.emplace_back(spec);
  }
  return outputSpecs;
}

std::tuple<RunArgs, RunArgs> LlgaKernel::prepareRunArgs(
    const TensorArgs& inputs,
    TensorArgs& outputs) const {
  RunArgs runInputs, runOutputs;
  auto numInputs = runArgsIdx_.size();
  for (const auto i : c10::irange(numInputs)) {
    auto spec = inputSpecs_[i];
    const auto& input = inputs[runArgsIdx_[i]];
    runInputs.emplace_back(
        spec.logical_tensor(), Engine::getEngine(), input.data_ptr());
  }
  auto numConstantInputs = constantInputs_.size();
  for (size_t i = 0; i < numConstantInputs; i++) {
    // constantInputSpecs are placed after graphInputSpecs
    auto constantInputSpecIdx = nGraphInputs_ + i;
    auto constantInputSpec = inputSpecs_[constantInputSpecIdx];
    runInputs.emplace_back(
        constantLogicalTensors_[i],
        Engine::getEngine(),
        constantInputs_[i].data_ptr());
```

- **EN:** Important callable entry points in this range include prepareRunArgs, getEngine.
- **CN:** 这一段的重要可调用入口包括 prepareRunArgs, getEngine。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 141-168 / 第 141-168 行

```cpp
  }

  for (const auto i : c10::irange(nOutputs_)) {
    auto spec = outputSpecs_[i];
    auto opt = c10::TensorOptions(spec.aten_scalar_type()).device(device_);

    if (spec.reuses_input_tensor()) {
#ifdef GRAPH_DEBUG_ENABLED
      GRAPH_DEBUG("inplace computation - input tensor would be reused");
#endif
      auto inputTensor = inputs[spec.get_input_tensor_index()];
      if (inputTensor.is_mkldnn()) {
        auto dataType = spec.dtype();
        if (C10_UNLIKELY(!useOpaqueLayout(i))) {
          // If the input tensor was between two partitions, it would've been
          // wrapped with LlgaTensorImpl. But if it's being reused as the output
          // tensor, which is not between two partitions, then we'd have to
          // re-wrap it with a sub-class of TensorImpl, as it'd be fed into a
          // PyTorch op.
#ifdef GRAPH_DEBUG_ENABLED
          GRAPH_DEBUG("rewrap tensors");
#endif
          auto llgaImpl =
              static_cast<LlgaTensorImpl*>(inputTensor.unsafeGetTensorImpl());
          switch (dataType) {
            case data_type::f32:
            case data_type::bf16:
              inputTensor = LlgaTensorImpl::llga_to_aten_tensor(llgaImpl);
```

- **EN:** The block declares or refines core types including of.
- **CN:** 该代码块声明或细化了 of 等核心类型。
- **EN:** Important callable entry points in this range include GRAPH_DEBUG.
- **CN:** 这一段的重要可调用入口包括 GRAPH_DEBUG。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 169-196 / 第 169-196 行

```cpp
              break;
            case data_type::s32:
            default:
              TORCH_CHECK(
                  false, "Invalid data type ", static_cast<size_t>(dataType));
          }
        }
        outputs.push_back(inputTensor);
        runOutputs.emplace_back(
            spec.logical_tensor(), Engine::getEngine(), inputTensor.data_ptr());
        return std::make_tuple(runInputs, runOutputs);
      }
    }
    if (useOpaqueLayout(i)) {
      // Wrap tensors between partitions with LlgaTensorImpl wrapper, so that we
      // can bypass guard-check, as strides would be different than those
      // expected.
#ifdef GRAPH_DEBUG_ENABLED
      GRAPH_DEBUG("Between two oneDNN Graph partitions");
#endif
      auto tensor = empty_llga(spec, opt);
      outputs.push_back(tensor);
      runOutputs.push_back(llga_from_aten_tensor(tensor));
    } else {
#ifdef GRAPH_DEBUG_ENABLED
      GRAPH_DEBUG("Neither opaque to PyTorch nor inplace-computation");
#endif
      auto tensor = at::empty_strided(spec.sizes(), spec.strides(), opt);
```

- **EN:** Important callable entry points in this range include TORCH_CHECK, make_tuple, GRAPH_DEBUG.
- **CN:** 这一段的重要可调用入口包括 TORCH_CHECK, make_tuple, GRAPH_DEBUG。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 197-224 / 第 197-224 行

```cpp
      outputs.push_back(tensor);
      runOutputs.emplace_back(
          spec.logical_tensor(), Engine::getEngine(), tensor.data_ptr());
    }
  }

  return std::make_tuple(runInputs, runOutputs);
}

compiled_partition LlgaKernel::compile(const partition& partition) {
  auto inputs = fmap(inputSpecs_, toLogicalTensor);
  auto outputs = fmap(outputSpecs_, toLogicalTensor);
  auto compilation = partition.compile(inputs, outputs, Engine::getEngine());

  // Since layouts of opaque outputs would be known after compilation,
  // we need to query them out from compilation and update outputSpecs
  for (const auto i : c10::irange(nOutputs_)) {
    auto tid = outputSpecs_[i].tid();
    outputSpecs_[i] = compilation.query_logical_tensor(tid);
  }

  // Build static mapping from output id to input offset
  // in accordance with available inplace options
  for (auto&& option : compilation.get_inplace_ports()) {
    size_t inputId = option.first;
    size_t outputId = option.second;
    auto inputSpecIter =
        std::find_if(inputSpecs_.begin(), inputSpecs_.end(), [&](auto& spec) {
```

- **EN:** Important callable entry points in this range include make_tuple, compile, find_if.
- **CN:** 这一段的重要可调用入口包括 make_tuple, compile, find_if。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 225-252 / 第 225-252 行

```cpp
          return spec.tid() == inputId;
        });
    TORCH_CHECK(inputSpecIter != inputSpecs_.end(), "In-place input not found");
    auto inputOffset = inputSpecIter - inputSpecs_.begin();
    auto outputSpecIter =
        std::find_if(outputSpecs_.begin(), outputSpecs_.end(), [&](auto& spec) {
          return spec.tid() == outputId;
        });
    auto outputOffset = outputSpecIter - outputSpecs_.begin();
    outputSpecs_[outputOffset].set_compute_inplace();
    outputSpecs_[outputOffset].set_input_tensor_index(inputOffset);
  }

  return compilation;
}

void LlgaKernel::run(Stack& stack) {
#ifdef GRAPH_DEBUG_ENABLED
  GRAPH_DEBUG("In ", debugName(), '\n');
#endif

  // Grab input values from stack
  auto stackInputs = last(stack, nGraphInputs_);
  auto inputs = fmap(stackInputs, [&](const IValue& v) {
    TORCH_CHECK(
        v.isTensor(), "Stack values for LLGA partition must be Tensor type");
    return v.toTensor();
  });
```

- **EN:** Important callable entry points in this range include TORCH_CHECK, find_if, run, GRAPH_DEBUG.
- **CN:** 这一段的重要可调用入口包括 TORCH_CHECK, find_if, run, GRAPH_DEBUG。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 253-280 / 第 253-280 行

```cpp

  // Even in case of concurrent threads, the kernel would be initialized once.
  // TODO: Try not using an atomic lock
  c10::call_once(
      initialized_flag,
      [&](const TensorArgs& inputs) {
        GRAPH_DEBUG("Initializing input logical tensors");
        inputSpecs_ = initializeInputSpecs(inputs);
        GRAPH_DEBUG("Initializing output logical tensors");
        outputSpecs_ = initializeOutputSpecs();
        GRAPH_DEBUG("Compiling partition");
        compilation_ = compile(partition_);
        is_initialized_ = true;
      },
      inputs);
#ifdef GRAPH_DEBUG_ENABLED
  GRAPH_DEBUG("Preparing runtime tensors");
#endif
  TensorArgs outputs;
  auto [runInputs, runOutputs] = prepareRunArgs(inputs, outputs);
#ifdef GRAPH_DEBUG_ENABLED
  GRAPH_DEBUG("Executing partition");
#endif
  compilation_.execute(Stream::getStream(), runInputs, runOutputs);
#ifdef GRAPH_DEBUG_ENABLED
  GRAPH_DEBUG("Partition executed");
#endif

```

- **EN:** Important callable entry points in this range include call_once, GRAPH_DEBUG.
- **CN:** 这一段的重要可调用入口包括 call_once, GRAPH_DEBUG。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Backend integration / 后端集成, Declared symbols / 声明的符号, Macro control flow / 宏控制流.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Backend integration / 后端集成, Declared symbols / 声明的符号, Macro control flow / 宏控制流。

### Lines 281-290 / 第 281-290 行

```cpp
  // Update the stack.
  drop(stack, nGraphInputs_);
  for (auto& o : outputs)
    push_one(stack, std::move(o));
#ifdef GRAPH_DEBUG_ENABLED
  GRAPH_DEBUG("Stack updated");
#endif
}

} // namespace torch::jit::fuser::onednn
```

- **EN:** Important callable entry points in this range include drop, GRAPH_DEBUG.
- **CN:** 这一段的重要可调用入口包括 drop, GRAPH_DEBUG。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Nodes and values / 节点与值, Declared symbols / 声明的符号, Macro control flow / 宏控制流, Namespace scoping / 命名空间作用域, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Nodes and values / 节点与值, Declared symbols / 声明的符号, Macro control flow / 宏控制流, Namespace scoping / 命名空间作用域, Iteration / 迭代处理。

## Key Concepts / 关键概念

- **Generated code pipeline** — 代码生成流水线
- **Graph IR** — 图中间表示
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Serialization** — 序列化
- **Backend integration** — 后端集成
- **Code generation** — 代码生成
- **Optimization pass** — 优化 pass
- **Alias analysis** — 别名分析

## Dependencies / 依赖关系

- `torch/csrc/jit/codegen/onednn/graph_helper.h`
- `torch/csrc/jit/codegen/onednn/kernel.h`
- `torch/csrc/jit/jit_log.h`
