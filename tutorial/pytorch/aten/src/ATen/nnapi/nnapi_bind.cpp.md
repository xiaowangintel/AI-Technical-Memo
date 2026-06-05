# nnapi_bind.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `aten/src/ATen/nnapi/nnapi_bind.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements the NNAPI bridge used to lower or execute ATen models on Android NNAPI. This file specifically implements the logic associated with `nnapi_bind.cpp`. Quantization-specific scale, zero-point, or kernel-selection concerns are central here. Backend-specific integration details shape the API or implementation choices.
- **Purpose (CN)**: 实现 NNAPI 桥接层，用于在 Android NNAPI 上降级或执行 ATen 模型。 该文件具体实现与 `nnapi_bind.cpp` 相关的逻辑。 量化相关的 scale、zero point 或内核选择问题是这里的核心。 后端专用的集成细节会影响这里的 API 或实现选择。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8 / 第 1-8 行

```cpp
#include <utility>
#include <vector>

#include <ATen/nnapi/nnapi_bind.h>
#include <ATen/nnapi/nnapi_wrapper.h>
#include <ATen/nnapi/nnapi_model_loader.h>
#include <c10/util/irange.h>

```

- **EN:** This range is mainly assembling the headers or imports needed by the declarations and runtime logic that follow.
- **CN:** 这一段主要在组织后续声明与运行时逻辑所需的头文件或导入项。
- **EN:** Backend-specific identifiers appear in this range, so the code is adapting ATen abstractions to a concrete device/runtime interface.
- **CN:** 这一段出现了后端专用标识符，说明代码在把 ATen 抽象适配到具体设备或运行时接口。
- **EN:** Concepts touched here: Backend interop / 后端互操作, Header composition / 头文件组织.
- **CN:** 这里涉及的概念包括：Backend interop / 后端互操作, Header composition / 头文件组织。

### Lines 9-16 / 第 9-16 行

```cpp
namespace torch::nnapi::bind {

// NOLINTNEXTLINE(cppcoreguidelines-avoid-non-const-global-variables)
nnapi_wrapper* nnapi;
// NOLINTNEXTLINE(cppcoreguidelines-avoid-non-const-global-variables)
nnapi_wrapper* check_nnapi;

static void load_platform_library() {
```

- **EN:** It establishes namespace scopes such as torch::nnapi::bind, placing the code inside the relevant ATen/backend subsystem.
- **CN:** 这里建立了 torch::nnapi::bind 等命名空间作用域，把代码放入对应的 ATen/后端子系统中。
- **EN:** Important callable entry points in this range include load_platform_library.
- **CN:** 这一段的重要可调用入口包括 load_platform_library。
- **EN:** Backend-specific identifiers appear in this range, so the code is adapting ATen abstractions to a concrete device/runtime interface.
- **CN:** 这一段出现了后端专用标识符，说明代码在把 ATen 抽象适配到具体设备或运行时接口。
- **EN:** Concepts touched here: Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号, Namespace scoping / 命名空间作用域.
- **CN:** 这里涉及的概念包括：Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号, Namespace scoping / 命名空间作用域。

### Lines 17-27 / 第 17-27 行

```cpp
  static int run_once = [](){
    nnapi_wrapper_load(&nnapi, &check_nnapi);
    CAFFE_ENFORCE(nnapi);
    CAFFE_ENFORCE(nnapi->Model_free);
    CAFFE_ENFORCE(nnapi->Compilation_free);
    CAFFE_ENFORCE(nnapi->Execution_free);
    return 0;
  }();
  (void)run_once;
}

```

- **EN:** Important callable entry points in this range include nnapi_wrapper_load.
- **CN:** 这一段的重要可调用入口包括 nnapi_wrapper_load。
- **EN:** Backend-specific identifiers appear in this range, so the code is adapting ATen abstractions to a concrete device/runtime interface.
- **CN:** 这一段出现了后端专用标识符，说明代码在把 ATen 抽象适配到具体设备或运行时接口。
- **EN:** Return statements finalize the local helper result, state update, or wrapper output for the surrounding API.
- **CN:** 返回语句会为周边 API 给出局部辅助结果、状态更新或封装后的输出。
- **EN:** Concepts touched here: Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号.
- **CN:** 这里涉及的概念包括：Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号。

### Lines 28-43 / 第 28-43 行

```cpp
// NnapiCompilation function definitions:

// Could possibly call load_platform_library in constructor, but error reporting
// can be complicated if the constructor is called during model loading.
// Instead, delay all work until the explicit init call.
void NnapiCompilation::init(
    at::Tensor serialized_model_tensor,
    std::vector<at::Tensor> parameter_buffers
) {
  init2(
    std::move(serialized_model_tensor),
    std::move(parameter_buffers),
    ANEURALNETWORKS_PREFER_SUSTAINED_SPEED,
    false);
}

```

- **EN:** Important callable entry points in this range include init, init2.
- **CN:** 这一段的重要可调用入口包括 init, init2。
- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Backend interop / 后端互操作, Declared symbols / 声明符号.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Backend interop / 后端互操作, Declared symbols / 声明符号。

### Lines 44-51 / 第 44-51 行

```cpp
void NnapiCompilation::init2(
    at::Tensor serialized_model_tensor,
    const std::vector<at::Tensor>& parameter_buffers,
    int64_t compilation_preference,
    bool relax_f32_to_f16
  ) {
  TORCH_CHECK(!model_, "Attempted to re-initialize NnapiCompilation.");

```

- **EN:** Important callable entry points in this range include init2.
- **CN:** 这一段的重要可调用入口包括 init2。
- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Backend interop / 后端互操作, Declared symbols / 声明符号.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Backend interop / 后端互操作, Declared symbols / 声明符号。

### Lines 52-59 / 第 52-59 行

```cpp
  load_platform_library();

  std::vector<const void*> buffers;
  buffers.reserve(parameter_buffers.size());
  std::vector<int32_t> buffer_sizes;
  buffer_sizes.reserve(parameter_buffers.size());
  for (auto& t : parameter_buffers) {
    TORCH_CHECK(t.is_contiguous());
```

- **EN:** Important callable entry points in this range include load_platform_library.
- **CN:** 这一段的重要可调用入口包括 load_platform_library。
- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Loop constructs iterate over dimensions, parameters, test sweeps, or kernel inputs in this portion.
- **CN:** 这一部分中的循环用于遍历维度、参数组合、测试扫面或内核输入。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Memory layout / 内存布局, Declared symbols / 声明符号, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Memory layout / 内存布局, Declared symbols / 声明符号, Iteration / 迭代处理。

### Lines 60-74 / 第 60-74 行

```cpp
    buffers.push_back(t.data_ptr());
    buffer_sizes.push_back(t.nbytes());
  }

  TORCH_CHECK(serialized_model_tensor.is_contiguous());
  // This is currently always int32_t, but support uint8_t for old models
  // and possible future changes to the generator.
  uint8_t* ser_model_ptr =
    serialized_model_tensor.scalar_type() == at::ScalarType::Byte
      ? serialized_model_tensor.data_ptr<uint8_t>()
      : reinterpret_cast<uint8_t*>(serialized_model_tensor.data_ptr<int32_t>());
  c10::ArrayRef<uint8_t> ser_model = {
    ser_model_ptr,
    serialized_model_tensor.nbytes()
  };
```

- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Memory layout / 内存布局, Random generator state / 随机数生成器状态.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Memory layout / 内存布局, Random generator state / 随机数生成器状态。

### Lines 75-90 / 第 75-90 行

```cpp
  TORCH_CHECK(!ser_model.empty());

  ANeuralNetworksModel* model{};
  check_nnapi->Model_create(&model);
  CAFFE_ENFORCE(model);
  model_.reset(model);

  int load_result = ::caffe2::nnapi::load_nnapi_model(
      nnapi,
      model_.get(),
      ser_model.data(),
      ser_model.size(),
      buffers.size(),
      buffers.data(),
      buffer_sizes.data(),
      0,
```

- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Backend interop / 后端互操作, Testing harness / 测试框架.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Backend interop / 后端互操作, Testing harness / 测试框架。

### Lines 91-98 / 第 91-98 行

```cpp
      nullptr,
      nullptr,
      &num_inputs_,
      &num_outputs_,
      nullptr);
  CAFFE_ENFORCE(load_result == 0);

  if (relax_f32_to_f16) {
```

- **EN:** Conditional branches separate feature checks, special cases, or fast-path decisions.
- **CN:** 条件分支用于区分特性检查、特殊情况或快路径决策。
- **EN:** Concepts touched here: Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Branching logic / 分支逻辑。

### Lines 99-110 / 第 99-110 行

```cpp
    check_nnapi->Model_relaxComputationFloat32toFloat16(model_.get(), true);
  }
  check_nnapi->Model_finish(model_.get());

  ANeuralNetworksCompilation* compilation{};
  check_nnapi->Compilation_create(model_.get(), &compilation);
  // TODO: Make this configurable.
  check_nnapi->Compilation_setPreference(compilation, static_cast<int32_t>(compilation_preference));
  check_nnapi->Compilation_finish(compilation);
  compilation_.reset(compilation);
}

```

- **EN:** Backend-specific identifiers appear in this range, so the code is adapting ATen abstractions to a concrete device/runtime interface.
- **CN:** 这一段出现了后端专用标识符，说明代码在把 ATen 抽象适配到具体设备或运行时接口。
- **EN:** Concepts touched here: Backend interop / 后端互操作, Testing harness / 测试框架.
- **CN:** 这里涉及的概念包括：Backend interop / 后端互操作, Testing harness / 测试框架。

### Lines 111-118 / 第 111-118 行

```cpp
void NnapiCompilation::run(
    std::vector<at::Tensor> inputs,
    std::vector<at::Tensor> outputs) {
  ANeuralNetworksExecution* execution = nullptr;
  check_nnapi->Execution_create(compilation_.get(), &execution);
  ExecutionPtr execution_unique_ptr(execution);

  TORCH_CHECK((int32_t)inputs.size() == num_inputs_);
```

- **EN:** Important callable entry points in this range include run, execution_unique_ptr.
- **CN:** 这一段的重要可调用入口包括 run, execution_unique_ptr。
- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号。

### Lines 119-134 / 第 119-134 行

```cpp
  TORCH_CHECK((int32_t)outputs.size() == num_outputs_);

  for (const auto i : c10::irange(inputs.size())) {
    auto& t = inputs[i];
    // TODO: Check contiguous and dtype.
    ANeuralNetworksOperandType op_type;
    std::vector<uint32_t> dim;
    get_operand_type(t, &op_type, &dim);
    check_nnapi->Execution_setInput(
        execution,
        i,
        &op_type,
        t.data_ptr(),
        t.nbytes());
  }

```

- **EN:** Important callable entry points in this range include get_operand_type.
- **CN:** 这一段的重要可调用入口包括 get_operand_type。
- **EN:** Backend-specific identifiers appear in this range, so the code is adapting ATen abstractions to a concrete device/runtime interface.
- **CN:** 这一段出现了后端专用标识符，说明代码在把 ATen 抽象适配到具体设备或运行时接口。
- **EN:** Loop constructs iterate over dimensions, parameters, test sweeps, or kernel inputs in this portion.
- **CN:** 这一部分中的循环用于遍历维度、参数组合、测试扫面或内核输入。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Memory layout / 内存布局, Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Memory layout / 内存布局, Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号, Iteration / 迭代处理。

### Lines 135-145 / 第 135-145 行

```cpp
  for (const auto i : c10::irange(static_cast<int32_t>(outputs.size()))) {
    auto& t = outputs[i];
    // TODO: Check contiguous and dtype.
    check_nnapi->Execution_setOutput(
        execution,
        i,
        nullptr,
        t.data_ptr(),
        t.nbytes());
  }

```

- **EN:** Backend-specific identifiers appear in this range, so the code is adapting ATen abstractions to a concrete device/runtime interface.
- **CN:** 这一段出现了后端专用标识符，说明代码在把 ATen 抽象适配到具体设备或运行时接口。
- **EN:** Loop constructs iterate over dimensions, parameters, test sweeps, or kernel inputs in this portion.
- **CN:** 这一部分中的循环用于遍历维度、参数组合、测试扫面或内核输入。
- **EN:** Concepts touched here: Memory layout / 内存布局, Backend interop / 后端互操作, Testing harness / 测试框架, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Memory layout / 内存布局, Backend interop / 后端互操作, Testing harness / 测试框架, Iteration / 迭代处理。

### Lines 146-160 / 第 146-160 行

```cpp
  check_nnapi->Execution_compute(execution);

  // TODO: Maybe skip this for fixed-size outputs?
  for (const auto i : c10::irange(static_cast<int32_t>(outputs.size()))) {
    auto& t = outputs[i];
    uint32_t rank = 0;
    check_nnapi->Execution_getOutputOperandRank(execution, i, &rank);
    std::vector<uint32_t> dims(rank);
    check_nnapi->Execution_getOutputOperandDimensions(execution, i, dims.data());
    std::vector<int64_t> long_dims(dims.begin(), dims.end());
    // TODO: Maybe check that only the batch dimension is changed?
    t.resize_(long_dims);
  }
}

```

- **EN:** Important callable entry points in this range include dims, long_dims.
- **CN:** 这一段的重要可调用入口包括 dims, long_dims。
- **EN:** Backend-specific identifiers appear in this range, so the code is adapting ATen abstractions to a concrete device/runtime interface.
- **CN:** 这一段出现了后端专用标识符，说明代码在把 ATen 抽象适配到具体设备或运行时接口。
- **EN:** Loop constructs iterate over dimensions, parameters, test sweeps, or kernel inputs in this portion.
- **CN:** 这一部分中的循环用于遍历维度、参数组合、测试扫面或内核输入。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号, Iteration / 迭代处理。

### Lines 161-168 / 第 161-168 行

```cpp
void NnapiCompilation::get_operand_type(const at::Tensor& t, ANeuralNetworksOperandType* operand, std::vector<uint32_t>* dims) {
  operand->dimensionCount = t.dim();
  TORCH_CHECK(operand->dimensionCount == t.dim()); // Check for overflow.
  dims->resize(t.dim());
  operand->dimensions = dims->data();
  for (const auto i : c10::irange(dims->size())) {
    (*dims)[i] = t.sizes()[i];
    TORCH_CHECK((*dims)[i] == t.sizes()[i]); // Check for overflow.
```

- **EN:** Important callable entry points in this range include get_operand_type.
- **CN:** 这一段的重要可调用入口包括 get_operand_type。
- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Loop constructs iterate over dimensions, parameters, test sweeps, or kernel inputs in this portion.
- **CN:** 这一部分中的循环用于遍历维度、参数组合、测试扫面或内核输入。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Backend interop / 后端互操作, Declared symbols / 声明符号, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Backend interop / 后端互操作, Declared symbols / 声明符号, Iteration / 迭代处理。

### Lines 169-176 / 第 169-176 行

```cpp
  }
  if (t.scalar_type() == c10::kFloat) {
    operand->type = ANEURALNETWORKS_TENSOR_FLOAT32;
    operand->scale = 0;
    operand->zeroPoint = 0;
    return;
  }
  if (t.scalar_type() == c10::kQUInt8) {
```

- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Conditional branches separate feature checks, special cases, or fast-path decisions.
- **CN:** 条件分支用于区分特性检查、特殊情况或快路径决策。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Quantization / 量化, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Quantization / 量化, Branching logic / 分支逻辑。

### Lines 177-189 / 第 177-189 行

```cpp
    TORCH_CHECK(t.is_quantized());
    operand->type = ANEURALNETWORKS_TENSOR_QUANT8_ASYMM;
    operand->scale = static_cast<float>(t.q_scale());
    operand->zeroPoint = static_cast<int32_t>(t.q_zero_point());
    return;
  }
  if (t.scalar_type() == c10::kInt) {
    operand->type = ANEURALNETWORKS_TENSOR_INT32;
    operand->scale = 0;
    operand->zeroPoint = 0;
    return;
  }
  if (t.scalar_type() == c10::kShort) {
```

- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Conditional branches separate feature checks, special cases, or fast-path decisions.
- **CN:** 条件分支用于区分特性检查、特殊情况或快路径决策。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Quantization / 量化, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Quantization / 量化, Branching logic / 分支逻辑。

### Lines 190-199 / 第 190-199 行

```cpp
    TORCH_WARN(
      "NNAPI qint16 inputs to model are only supported for ",
      "testing with fixed scale, zero_point. Please change your ",
      "inputs if you see this in production");
    operand->type = ANEURALNETWORKS_TENSOR_QUANT16_ASYMM;
    operand->scale = 0.125;
    operand->zeroPoint = 0;
    return;
  }

```

- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Quantization / 量化, Backend interop / 后端互操作.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Quantization / 量化, Backend interop / 后端互操作。

### Lines 200-204 / 第 200-204 行

```cpp
  // TODO: Support more dtypes.
  CAFFE_THROW("Bad dtype: " + std::to_string(static_cast<int8_t>(t.scalar_type())));
}

} // namespace torch
```

- **EN:** Concepts touched here: Namespace scoping / 命名空间作用域.
- **CN:** 这里涉及的概念包括：Namespace scoping / 命名空间作用域。

## Key Concepts / 关键概念

- **NNAPI bridge** — NNAPI 桥接层
- **Tensor metadata** — 张量元数据
- **Memory layout** — 内存布局
- **Quantization** — 量化
- **Random generator state** — 随机数生成器状态
- **Backend interop** — 后端互操作
- **Testing harness** — 测试框架
- **Core symbols: load_platform_library, nnapi_wrapper_load, init, init2, run, execution_unique_ptr, get_operand_type, dims** — 核心符号：load_platform_library、nnapi_wrapper_load、init、init2、run、execution_unique_ptr、get_operand_type、dims

## Dependencies / 依赖关系

- `utility`
- `vector`
- `ATen/nnapi/nnapi_bind.h`
- `ATen/nnapi/nnapi_wrapper.h`
- `ATen/nnapi/nnapi_model_loader.h`
- `c10/util/irange.h`
