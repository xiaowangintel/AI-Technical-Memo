# context.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/mobile/nnc/context.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements loading, execution, optimization, or serialization support for the mobile / lite interpreter runtime. This specific file centers on `context.cpp`. It is tailored for lite interpreter or mobile deployment constraints.
- **Purpose (CN)**: 实现移动端 / 轻量解释器运行时的加载、执行、优化或序列化支持。 该文件具体围绕 `context.cpp` 展开。 其设计面向轻量解释器或移动端部署约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

```cpp
#include <torch/csrc/jit/mobile/nnc/context.h>

#include <ATen/Functions.h>
#include <ATen/core/functional.h>
#include <c10/core/CPUAllocator.h>
#include <c10/util/irange.h>

#include <torch/csrc/jit/mobile/nnc/registry.h>

namespace torch::jit::mobile::nnc {

constexpr int64_t kProducedNNCFileFormatVersion = 0x1L;

namespace {

c10::IValue Tup(std::initializer_list<c10::IValue> ivalues) {
  return c10::ivalue::Tuple::create(ivalues);
}

c10::IValue Tup(std::vector<c10::IValue>&& ivalues) {
  return c10::ivalue::Tuple::create(ivalues);
}

} // namespace

InputSpec::InputSpec(const c10::IValue& value) {
  auto dict = value.toGenericDict();
  sizes_ = dict.at("sizes").toIntVector();
```

- **EN:** It enters or references namespace scopes such as torch::jit::mobile::nnc, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit::mobile::nnc 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Important callable entry points in this range include Tup, create, InputSpec.
- **CN:** 这一段的重要可调用入口包括 Tup, create, InputSpec。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。

### Lines 29-56 / 第 29-56 行

```cpp
  dtype_ = dict.at("dtype").toScalarType();
}

c10::IValue InputSpec::serialize() const {
  c10::Dict<c10::IValue, c10::IValue> dict(
      at::StringType::get(), at::AnyType::get());
  dict.insert("sizes", sizes_);
  dict.insert("dtype", dtype_);
  return dict;
}

bool InputSpec::validate(const at::Tensor& input) const {
  if (sizes_.size() != input.sizes().size() || input.scalar_type() != dtype_) {
    return false;
  }
  auto spec_sizes = sizes_;
  for (const auto i : c10::irange(spec_sizes.size())) {
    // InputSpec size 0 means that the dimension is dynamic
    if (spec_sizes[i] != 0 && spec_sizes[i] != input.sizes()[i]) {
      return false;
    }
  }
  return true;
}

OutputSpec::OutputSpec(const c10::IValue& value) {
  auto dict = value.toGenericDict();
  sizes_ = dict.at("sizes").toIntVector();
```

- **EN:** Important callable entry points in this range include serialize, dict, validate, OutputSpec.
- **CN:** 这一段的重要可调用入口包括 serialize, dict, validate, OutputSpec。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 57-84 / 第 57-84 行

```cpp
  dtype_ = dict.at("dtype").toScalarType();
  if (dict.contains("qscale")) {
    qscale_ = dict.at("qscale").toDouble();
  }
  if (dict.contains("qzero")) {
    qzero_ = dict.at("qzero").toInt();
  }
}

c10::IValue OutputSpec::serialize() const {
  c10::Dict<c10::IValue, c10::IValue> dict(
      at::StringType::get(), at::AnyType::get());
  dict.insert("sizes", sizes_);
  dict.insert("dtype", dtype_);
  if (qscale_) {
    dict.insert("qscale", *qscale_);
  }
  if (qzero_) {
    dict.insert("qzero", *qzero_);
  }
  return dict;
}

at::Tensor OutputSpec::allocate() const {
  if (isQIntType(dtype_)) {
    TORCH_CHECK(
        qscale_ && qzero_,
        "Quantized output tensor must have qscale_ and qzero_");
```

- **EN:** Important callable entry points in this range include serialize, dict, allocate, TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 serialize, dict, allocate, TORCH_CHECK。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 85-112 / 第 85-112 行

```cpp
    return at::_empty_affine_quantized(
        sizes_,
        at::TensorOptions()
            .dtype(dtype_)
            .layout(at::kStrided)
            .device(at::kCPU)
            .requires_grad(false),
        *qscale_,
        *qzero_);
  }
  return at::empty(
      sizes_,
      at::TensorOptions()
          .dtype(dtype_)
          .layout(at::kStrided)
          .device(at::kCPU)
          .requires_grad(false));
}

MemoryPlan::MemoryPlan(const c10::IValue& value) {
  auto dict = value.toGenericDict();
  buffer_sizes_ = dict.at("buffer_sizes").toIntVector();
}

c10::IValue MemoryPlan::serialize() const {
  c10::Dict<c10::IValue, c10::IValue> dict(
      at::StringType::get(), at::AnyType::get());
  dict.insert("buffer_sizes", buffer_sizes_);
```

- **EN:** Important callable entry points in this range include _empty_affine_quantized, empty, MemoryPlan, serialize, dict.
- **CN:** 这一段的重要可调用入口包括 _empty_affine_quantized, empty, MemoryPlan, serialize, dict。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Serialization / 序列化, Alias analysis / 别名分析, Declared symbols / 声明的符号.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Serialization / 序列化, Alias analysis / 别名分析, Declared symbols / 声明的符号。

### Lines 113-140 / 第 113-140 行

```cpp
  return dict;
}

void MemoryPlan::allocate(ExecutionState* state) const {
  auto& allocations = state->preallocations_;
  allocations.clear();
  allocations.reserve(buffer_sizes_.size());
  for (int64_t buffer_size : buffer_sizes_) {
    at::DataPtr buffer = c10::GetCPUAllocator()->allocate(buffer_size);
    allocations.emplace_back(std::move(buffer));
  }
}

Function::Function(const c10::IValue& value) {
  auto dict = value.toGenericDict();
  name_ = c10::QualifiedName(dict.at("name").toStringRef());
  nnc_kernel_id_ = dict.at("nnc_kernel_id").toStringRef();
  parameters_ = dict.at("parameters").toList();

  // input_specs_
  for (const auto& input_value :
       dict.at("input_specs").toTupleRef().elements()) {
    input_specs_.emplace_back(input_value);
  }

  // output_specs_
  for (const auto& output_value :
       dict.at("output_specs").toTupleRef().elements()) {
```

- **EN:** Important callable entry points in this range include allocate, Function.
- **CN:** 这一段的重要可调用入口包括 allocate, Function。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 141-168 / 第 141-168 行

```cpp
    output_specs_.emplace_back(output_value);
  }

  // memory_plan_
  memory_plan_ = MemoryPlan(dict.at("memory_plan"));

  // symbolic shape positions
  for (const auto& sym_shape_pos :
       dict.at("sym_shape_pos").toTupleRef().elements()) {
    auto sym_shape_elements = sym_shape_pos.toTupleRef().elements();
    sym_shape_positions_.emplace_back(
        sym_shape_elements[0].toInt(), sym_shape_elements[1].toInt());
  }
}

c10::IValue Function::serialize() const {
  c10::Dict<c10::IValue, c10::IValue> dict(
      at::StringType::get(), at::AnyType::get());

  dict.insert("name", name_.qualifiedName());
  dict.insert("nnc_kernel_id", nnc_kernel_id_);
  // TODO: should serialize parameters with Module instead of with each Method.
  // And ideally the parameters should be shared between the compiled model
  // and the original model if we can serialize both in the same model file.
  dict.insert("parameters", parameters_);

  // input_specs_
  std::vector<c10::IValue> input_specs;
```

- **EN:** Important callable entry points in this range include serialize, dict.
- **CN:** 这一段的重要可调用入口包括 serialize, dict。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Module API / 模块 API, Serialization / 序列化, Backend integration / 后端集成, Alias analysis / 别名分析.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Module API / 模块 API, Serialization / 序列化, Backend integration / 后端集成, Alias analysis / 别名分析。

### Lines 169-196 / 第 169-196 行

```cpp
  input_specs.reserve(input_specs_.size());
  for (const auto& input_spec : input_specs_) {
    input_specs.emplace_back(input_spec.serialize());
  }
  dict.insert("input_specs", Tup(std::move(input_specs)));

  // output_specs_
  std::vector<c10::IValue> output_specs;
  output_specs.reserve(output_specs_.size());
  for (const auto& output_spec : output_specs_) {
    output_specs.emplace_back(output_spec.serialize());
  }
  dict.insert("output_specs", Tup(std::move(output_specs)));

  // memory_plan_
  dict.insert("memory_plan", memory_plan_.serialize());

  // sym_shape_positions_
  std::vector<c10::IValue> sym_shape_pos_vec;
  sym_shape_pos_vec.reserve(sym_shape_positions_.size());
  for (const auto& sym_shape_pos : sym_shape_positions_) {
    sym_shape_pos_vec.emplace_back(
        Tup({sym_shape_pos.input_idx_, sym_shape_pos.dim_idx_}));
  }
  dict.insert("sym_shape_pos", Tup(std::move(sym_shape_pos_vec)));

  return dict;
}
```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 197-224 / 第 197-224 行

```cpp

void Function::init_execution_state() const {
  if (execution_state_ != nullptr) {
    return;
  }

  ExecutionState state;
  memory_plan_.allocate(&state);

  // The arguments vector consists of 5 sections: inputs, symbolic shapes,
  // outputs, parameters and buffers.
  auto input_args = input_specs_.size();
  auto sym_shape_args = sym_shape_positions_.size();
  auto output_args = output_specs_.size();
  auto param_args = parameters_.size();
  auto buffer_args = state.preallocations_.size();

  auto& arguments = state.arguments_;
  arguments.reserve(
      input_args + sym_shape_args + output_args + param_args + buffer_args);

  // Keep empty slots to fill in inputs/outputs pointers at execution time.
  arguments.resize(input_args + sym_shape_args + output_args);

  // Fill in parameters as untyped raw pointers.
  // The underlying storage of the parameters should be owned by `parameters_`,
  // which should be alive when `execution_state_` is being used.
  for (const auto& param : parameters_) {
```

- **EN:** Important callable entry points in this range include init_execution_state.
- **CN:** 这一段的重要可调用入口包括 init_execution_state。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 225-252 / 第 225-252 行

```cpp
    const c10::IValue& ivalue = (c10::IValue)param;
    if (ivalue.isTensor()) {
      arguments.emplace_back(ivalue.toTensor().data_ptr());
    } else if (torch::isCustomClass(ivalue)) {
      arguments.emplace_back(ivalue.toObjectRef().getSlot(0).toCapsule().get());
    } else {
      TORCH_CHECK(false, "Invalid parameter: ", ivalue);
    }
  }

  // Fill in preallocated buffer pointers.
  for (const auto& preallocation : state.preallocations_) {
    arguments.emplace_back(preallocation.get());
  }

  execution_state_ = std::make_unique<ExecutionState>(std::move(state));
}

c10::impl::GenericList Function::run(
    const c10::impl::GenericList& inputs) const {
  TORCH_CHECK(
      registry::has_nnc_kernel(nnc_kernel_id_),
      "Cannot find NNC kernel: ",
      nnc_kernel_id_);

  init_execution_state();

  std::vector<void*>& args = execution_state_->arguments_;
```

- **EN:** Important callable entry points in this range include TORCH_CHECK, run, init_execution_state.
- **CN:** 这一段的重要可调用入口包括 TORCH_CHECK, run, init_execution_state。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 253-280 / 第 253-280 行

```cpp

  // Fill in input tensors.
  TORCH_CHECK(
      input_specs_.size() == inputs.size(),
      "Input size doesn't match the spec, expect: ",
      input_specs_.size(),
      " actual: ",
      inputs.size());
  std::vector<int64_t> scalar_values;
  int offset = 0;
  for (const auto i : c10::irange(inputs.size())) {
    const c10::IValue& input = inputs[i];
    const auto& spec = input_specs_[i];
    const auto& input_tensor = input.toTensor();
    TORCH_CHECK(spec.validate(input_tensor), "Invalid input at pos: ", i);
    args[i] = input_tensor.data_ptr();
  }
  offset += inputs.size();

  scalar_values.reserve(sym_shape_positions_.size());
  for (const auto i : c10::irange(sym_shape_positions_.size())) {
    const auto& sym_shape_pos = sym_shape_positions_[i];
    const c10::IValue& input = inputs[sym_shape_pos.input_idx_];
    auto dim = input.toTensor().size(sym_shape_pos.dim_idx_);
    scalar_values.push_back(dim);
    args[i + offset] = &scalar_values[scalar_values.size() - 1];
  }
  offset += sym_shape_positions_.size();
```

- **EN:** Important callable entry points in this range include TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 TORCH_CHECK。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Shape/resource guard / 形状或资源保护, Declared symbols / 声明的符号, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Shape/resource guard / 形状或资源保护, Declared symbols / 声明的符号, Iteration / 迭代处理。

### Lines 281-308 / 第 281-308 行

```cpp

  // Preallocate and fill in output tensors.
  c10::List<at::Tensor> outputs;
  outputs.reserve(output_specs_.size());
  for (const auto i : c10::irange(output_specs_.size())) {
    at::Tensor output = output_specs_[i].allocate();
    outputs.emplace_back(output);
    args[i + offset] = output.data_ptr();
  }

  // TODO: check consistency, e.g.: code version, input shape and compiled
  // shape, etc.
  auto kernel = registry::get_nnc_kernel(nnc_kernel_id_);
  kernel->execute(args.data());

  return c10::impl::toList(outputs);
}

CompilationUnit::CompilationUnit(const c10::IValue& value) {
  const auto& root = value.toTupleRef().elements();
  const auto& functions = root[1].toTupleRef().elements();
  for (const auto& function : functions) {
    register_function(std::make_unique<Function>(function));
  }
}

c10::IValue CompilationUnit::serialize() const {
  auto functions =
```

- **EN:** Important callable entry points in this range include toList, CompilationUnit, register_function, serialize.
- **CN:** 这一段的重要可调用入口包括 toList, CompilationUnit, register_function, serialize。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 309-336 / 第 309-336 行

```cpp
      c10::fmap(functions_, [](decltype(functions_)::const_reference func) {
        return func.second->serialize();
      });
  return Tup({kProducedNNCFileFormatVersion, Tup(std::move(functions))});
}

c10::impl::GenericList CompilationUnit::run(
    const c10::QualifiedName& name,
    const c10::impl::GenericList& inputs) const {
  Function* func = find_function(name);
  TORCH_CHECK(
      func != nullptr, "Function '", name.qualifiedName(), "' is not defined.");
  return func->run(inputs);
}

void CompilationUnit::register_function(std::unique_ptr<Function> fn) {
  TORCH_CHECK(
      0 == functions_.count(fn->name()),
      "method '",
      fn->name().qualifiedName(),
      "' already defined.");
  const auto& name = fn->name();
  functions_.emplace(name, std::move(fn));
}

Function* CompilationUnit::find_function(const c10::QualifiedName& name) const {
  auto it = functions_.find(name);
  if (it == functions_.end()) {
```

- **EN:** Important callable entry points in this range include fmap, run, TORCH_CHECK, register_function, find_function.
- **CN:** 这一段的重要可调用入口包括 fmap, run, TORCH_CHECK, register_function, find_function。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 337-342 / 第 337-342 行

```cpp
    return nullptr;
  }
  return it->second.get();
}

} // namespace torch::jit::mobile::nnc
```

- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Operator schema / 算子模式, Mobile runtime / 移动端运行时, Namespace scoping / 命名空间作用域, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Operator schema / 算子模式, Mobile runtime / 移动端运行时, Namespace scoping / 命名空间作用域, Result propagation / 结果传递。

## Key Concepts / 关键概念

- **Lite interpreter runtime** — 轻量解释器运行时
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Module API** — 模块 API
- **Serialization** — 序列化
- **Mobile runtime** — 移动端运行时
- **Backend integration** — 后端集成
- **Registration** — 注册机制
- **Alias analysis** — 别名分析

## Dependencies / 依赖关系

- `torch/csrc/jit/mobile/nnc/context.h`
- `ATen/Functions.h`
- `ATen/core/functional.h`
- `c10/core/CPUAllocator.h`
- `c10/util/irange.h`
- `torch/csrc/jit/mobile/nnc/registry.h`
