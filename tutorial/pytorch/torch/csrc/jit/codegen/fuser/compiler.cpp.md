# compiler.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/codegen/fuser/compiler.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Contains code-generation logic or generated declarations that synthesize JIT-related bindings, schemas, or helper code. This specific file centers on `compiler.cpp`. Generation of declarations, registrations, or helper code is part of the responsibility.
- **Purpose (CN)**: 包含代码生成逻辑或生成后的声明，用于产出 JIT 相关绑定、模式或辅助代码。 该文件具体围绕 `compiler.cpp` 展开。 其职责包含声明、注册或辅助代码的生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

```cpp
#include <torch/csrc/jit/codegen/fuser/compiler.h>

#include <ATen/core/jit_type.h>
#include <c10/util/Exception.h>
#include <c10/util/irange.h>
#include <torch/csrc/jit/codegen/fuser/codegen.h>
#include <torch/csrc/jit/codegen/fuser/kernel_cache.h>
#include <torch/csrc/jit/codegen/fuser/tensor_desc.h>
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/passes/shape_analysis.h>
#include <torch/csrc/jit/runtime/operator.h>

#include <atomic>
#include <memory>
#include <string>
#include <unordered_set>
#include <utility>

namespace {
std::mutex& fusionBackendLock() {
  static std::mutex fusion_backends_lock_{};
  return fusion_backends_lock_;
}
} // namespace

namespace torch::jit::fuser {

static std::unordered_map<at::Device::Type, FusedKernelConstructor>&
```

- **EN:** This range collects the headers or imports needed by the following declarations and implementation blocks.
- **CN:** 这一段集中引入后续声明与实现所需的头文件或模块。
- **EN:** It enters or references namespace scopes such as torch::jit::fuser, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit::fuser 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Important callable entry points in this range include fusionBackendLock.
- **CN:** 这一段的重要可调用入口包括 fusionBackendLock。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。

### Lines 29-56 / 第 29-56 行

```cpp
getFusionBackends() {
  static std::unordered_map<at::Device::Type, FusedKernelConstructor>
      fusion_backends;
  return fusion_backends;
}

void registerFusionBackend(
    at::Device::Type backend_type,
    FusedKernelConstructor ctor) {
  std::lock_guard<std::mutex> guard(fusionBackendLock());
  getFusionBackends()[backend_type] = std::move(ctor);
}

bool hasFusionBackend(at::Device::Type backend_type) {
  std::lock_guard<std::mutex> guard(fusionBackendLock());
  return getFusionBackends().count(backend_type);
}

static const FusedKernelConstructor& getConstructor(
    at::Device::Type backend_type) {
  std::lock_guard<std::mutex> guard(fusionBackendLock());
  return getFusionBackends().at(backend_type);
}

// Counter for number of kernels compiled, used for debugging and
// creating arbitrary kernel names.
static std::atomic<size_t> next_kernel_id{0};
static int debug_fusion{-1};
```

- **EN:** Important callable entry points in this range include getFusionBackends, registerFusionBackend, guard, hasFusionBackend, getConstructor.
- **CN:** 这一段的重要可调用入口包括 getFusionBackends, registerFusionBackend, guard, hasFusionBackend, getConstructor。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Backend integration / 后端集成, Optimization pass / 优化 pass, Registration / 注册机制.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Backend integration / 后端集成, Optimization pass / 优化 pass, Registration / 注册机制。

### Lines 57-84 / 第 57-84 行

```cpp

size_t nCompiledKernels() {
  return next_kernel_id.load();
}

int debugFuser() {
  if (debug_fusion < 0) {
    const auto debug_env = c10::utils::get_env("PYTORCH_FUSION_DEBUG");
    debug_fusion = debug_env ? atoi(debug_env->c_str()) : 0;
  }
  return debug_fusion;
}

// If the given node is used once by a chunk node, returns that node.
// Returns nullptr otherwise.
static const Node* usedInFusedChunk(const Value* input) {
  const auto& uses = input->uses();
  if (uses.size() == 1) {
    const Node* user = uses[0].user;
    if (user->kind() == prim::ConstantChunk) {
      return user;
    }
  }
  return nullptr;
}

static void setInputChunkDescriptors(KernelSpec& spec) {
  // We only have as many chunk descriptors as tensor inputs,
```

- **EN:** Important callable entry points in this range include nCompiledKernels, debugFuser, usedInFusedChunk, setInputChunkDescriptors.
- **CN:** 这一段的重要可调用入口包括 nCompiledKernels, debugFuser, usedInFusedChunk, setInputChunkDescriptors。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 85-112 / 第 85-112 行

```cpp
  // furthermore we know that the tensor inputs are in the
  // beginning of the fusion group's inputs.
  spec.inputChunks().reserve(spec.nTensorInputs());
  for (const auto i : c10::irange(spec.nTensorInputs())) {
    const Value* input = spec.graph()->inputs()[i];
    if (const Node* chunk = usedInFusedChunk(input)) {
      spec.inputChunks().emplace_back(
          chunk->i(attr::chunks), chunk->i(attr::dim));
    } else {
      spec.inputChunks().emplace_back(1, 0);
    }
  }
}

// Run a DFS traversal to find all inputs that affect a given output value
static std::vector<int64_t> getInputDependencies(const Value* output) {
  std::vector<const Value*> queue{output};
  std::unordered_set<const Value*> inputs;
  std::unordered_set<const Value*> seen;
  while (!queue.empty()) {
    const Value* val = queue.back();
    queue.pop_back();
    const Node* producer = val->node();
    // Here we assume that only tensor inputs are used in
    // the computation of the outputs.
    // This is currently true, as the only inputs will be
    // sizes (for _grad_sum_to_size as the derivative
    // of broadcasts), which will only be used after
```

- **EN:** Important callable entry points in this range include getInputDependencies.
- **CN:** 这一段的重要可调用入口包括 getInputDependencies。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 113-140 / 第 113-140 行

```cpp
    // the fusion kernel, and Tensors.
    // This needs to be revisited when you start allowing
    // other things e.g. nonconstant scalars.
    if (producer->kind() == prim::Param &&
        val->type()->isSubtypeOf(*TensorType::get())) {
      inputs.insert(val);
      continue;
    }
    for (const Value* input : producer->inputs()) {
      if (/*bool inserted = */ seen.insert(input).second) {
        queue.push_back(input);
      }
    }
  }

  // Convert Value* into offsets into the graph's input list
  std::vector<int64_t> offsets;
  offsets.reserve(inputs.size());
  for (const Value* input : inputs) {
    offsets.push_back(input->offset());
  }

  std::sort(offsets.begin(), offsets.end());
  return offsets;
}

static void setInputBroadcastGroups(KernelSpec& spec) {
  std::unordered_set<std::vector<int64_t>, c10::hash<std::vector<int64_t>>>
```

- **EN:** Important callable entry points in this range include sort, setInputBroadcastGroups.
- **CN:** 这一段的重要可调用入口包括 sort, setInputBroadcastGroups。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 141-168 / 第 141-168 行

```cpp
      broadcast_groups;
  for (const Value* output : (spec.graph())->outputs()) {
    if (output->node()->kind() == prim::FusedConcat) {
      for (const Value* concat_input : output->node()->inputs()) {
        broadcast_groups.insert(getInputDependencies(concat_input));
      }
    } else {
      broadcast_groups.insert(getInputDependencies(output));
    }
  }
  std::copy(
      broadcast_groups.begin(),
      broadcast_groups.end(),
      std::back_inserter(spec.inputBroadcastGroups()));
}

// Performs "upfront" compilation where storage is known but shapes are not.
// Currently identifies how to expand all tensors so that all intermediate
// tensors are the same shape, simplifying code generation.
// Broadcast groups and chunks are identified without shape information
// using logical properties of how each works. In particular, tensors
// are always expandable to the outputs of pointwise operations they
// or their descendants are involved in, which means that in a DAG of
// pointwise operations all tensors are expandable to the (single) output.
// Note: The logic is slightly complicated by concatenation and chunking.
static void upfrontCompilation(KernelSpec& spec) {
  setInputBroadcastGroups(spec);
  setInputChunkDescriptors(spec);
```

- **EN:** Important callable entry points in this range include copy, upfrontCompilation, setInputBroadcastGroups, setInputChunkDescriptors.
- **CN:** 这一段的重要可调用入口包括 copy, upfrontCompilation, setInputBroadcastGroups, setInputChunkDescriptors。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 169-196 / 第 169-196 行

```cpp
}

int64_t registerFusion(const Node* fusion_group) {
  auto graph = normalizeGraphForCache(fusion_group->g(attr::Subgraph));

  // Don't re-register the fusion if we can use a pre-existing one
  const auto maybe_spec = lookupGraph(graph);
  if (maybe_spec) {
    return (*maybe_spec)->key();
  }

  // Unconditionally create and register the fusion
  // This is necessary to support our global disable fusions flag: if someone
  // runs some code under no-fusions mode and then runs some code with fusions
  // enabled, the second time around the returned spec from the cache should
  // be a valid spec (must have had upfrontCompilation run on it).
  const auto key = store(graph);
  const auto maybe_retrieved_spec = retrieve(key);
  AT_ASSERT(maybe_retrieved_spec);
  upfrontCompilation(**maybe_retrieved_spec);

  return key;
}

std::shared_ptr<FusedKernel> compileKernel(
    const KernelSpec& spec,
    const ArgSpec& arg_spec,
    const std::vector<int64_t>& map_size,
```

- **EN:** Important callable entry points in this range include registerFusion, AT_ASSERT, upfrontCompilation.
- **CN:** 这一段的重要可调用入口包括 registerFusion, AT_ASSERT, upfrontCompilation。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 197-224 / 第 197-224 行

```cpp
    const at::Device& device) {
  const std::vector<TensorDesc>& input_desc = arg_spec.descs();

  auto graph = spec.graph()->copy();

  for (const auto i : c10::irange(input_desc.size())) {
    const auto& desc = input_desc[i];

    // TODO: can't get rid of this use of TensorType
    // until we switch to ProfilingGraphExecutor, so we don't have to
    // run PropagateInputShapes below
    graph->inputs()[i]->setType(TensorType::create(
        desc.scalar_type,
        device,
        {desc.nDim()},
        false)); // TODO: nDim is bad, as it is collapsed
  }

  PropagateInputShapes(graph);

  // Creates chunk and flattened input descriptions
  std::vector<PartitionDesc> chunk_desc;
  std::vector<std::pair<const Value*, const std::optional<TensorDesc>>>
      flat_inputs;
  {
    size_t input_index = 0;
    for (const auto& p : graph->inputs()) {
      if (p->type()->isSubtypeOf(*FloatType::get())) {
```

- **EN:** Important callable entry points in this range include PropagateInputShapes.
- **CN:** 这一段的重要可调用入口包括 PropagateInputShapes。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 225-252 / 第 225-252 行

```cpp
        flat_inputs.emplace_back(p, std::nullopt);
      }
      if (!p->type()->isSubtypeOf(*TensorType::get())) {
        continue;
      }
      if (const Node* chunk = usedInFusedChunk(p)) {
        int64_t dim = chunk->i(attr::dim);
        int64_t chunks = chunk->i(attr::chunks);
        chunk_desc.emplace_back(input_desc[input_index++], chunks, dim);
        for (const auto* o : chunk->outputs()) {
          flat_inputs.emplace_back(o, *chunk_desc.back().subTensorDesc());
        }
      } else {
        chunk_desc.emplace_back();
        flat_inputs.emplace_back(p, input_desc[input_index++]);
      }
    }
  }

  // Creates output, concat, and flattened output descriptions
  std::vector<TensorDesc> output_desc;
  std::vector<PartitionDesc> concat_desc;
  std::vector<std::pair<const Value*, const TensorDesc>> flat_outputs;
  for (const Value* o : graph->outputs()) {
    // Creates output description
    std::vector<int64_t> sizes = map_size;
    if (o->node()->kind() == prim::FusedConcat) {
      sizes.at(o->node()->i(attr::dim)) *= o->node()->inputs().size();
```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 253-280 / 第 253-280 行

```cpp
    }

    auto scalar_type = o->type()->expectRef<TensorType>().scalarType();
    TORCH_INTERNAL_ASSERT(scalar_type);
    auto type = TensorType::createContiguous(*scalar_type, device, sizes);
    output_desc.emplace_back(type);
    const auto& desc = output_desc.back();

    // Creates concat and flattened output descriptions (relies on output desc)
    if (o->node()->kind() != prim::FusedConcat) {
      concat_desc.emplace_back();
      flat_outputs.emplace_back(o, desc);
    } else {
      const auto cat = o->node();
      concat_desc.emplace_back(desc, cat->inputs().size(), cat->i(attr::dim));
      for (const auto& c : cat->inputs()) {
        flat_outputs.emplace_back(c, *concat_desc.back().subTensorDesc());
      }
    }
  }

  const bool use_cuda = device.is_cuda();
  const std::string name = "kernel_" + std::to_string(next_kernel_id++);
  std::string code =
      generateKernel(name, *graph, flat_inputs, flat_outputs, use_cuda);
  const FusedKernelConstructor& kernel_ctor =
      getConstructor(use_cuda ? DeviceType::CUDA : DeviceType::CPU);
  return kernel_ctor(
```

- **EN:** Important callable entry points in this range include TORCH_INTERNAL_ASSERT, generateKernel, getConstructor.
- **CN:** 这一段的重要可调用入口包括 TORCH_INTERNAL_ASSERT, generateKernel, getConstructor。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 281-291 / 第 281-291 行

```cpp
      device.index(),
      name,
      code,
      input_desc,
      output_desc,
      chunk_desc,
      concat_desc,
      spec.hasRandom());
}

} // namespace torch::jit::fuser
```

- **EN:** Concepts touched here: Nodes and values / 节点与值, Namespace scoping / 命名空间作用域.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Namespace scoping / 命名空间作用域。

## Key Concepts / 关键概念

- **Generated code pipeline** — 代码生成流水线
- **Graph IR** — 图中间表示
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Backend integration** — 后端集成
- **Code generation** — 代码生成
- **Optimization pass** — 优化 pass
- **Registration** — 注册机制
- **Alias analysis** — 别名分析

## Dependencies / 依赖关系

- `torch/csrc/jit/codegen/fuser/compiler.h`
- `ATen/core/jit_type.h`
- `c10/util/Exception.h`
- `c10/util/irange.h`
- `torch/csrc/jit/codegen/fuser/codegen.h`
- `torch/csrc/jit/codegen/fuser/kernel_cache.h`
- `torch/csrc/jit/codegen/fuser/tensor_desc.h`
- `torch/csrc/jit/ir/ir.h`
- `torch/csrc/jit/passes/shape_analysis.h`
- `torch/csrc/jit/runtime/operator.h`
