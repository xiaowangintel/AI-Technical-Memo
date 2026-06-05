# NNPACK.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/NNPACK.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on nnpack; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 nnpack；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/core/Tensor.h>
#include <ATen/Config.h>

#include <c10/util/error.h>

#include <thread>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/_nnpack_available_native.h>
#include <ATen/ops/_nnpack_spatial_convolution_native.h>
#include <ATen/ops/empty.h>
#include <ATen/ops/zeros.h>
#endif

#if !AT_NNPACK_ENABLED()

namespace at::native {

at::Tensor _nnpack_spatial_convolution(
    const Tensor& input,
    const Tensor& weight, const std::optional<Tensor>& bias_opt,
    const IntArrayRef padding,
    const IntArrayRef stride) {
  TORCH_CHECK(false, "nnpack_spatial_convolution: ATen not compiled with NNPACK support");
}

bool _nnpack_available() {
  return false;
}

} // namespace at::native

#else

#include <nnpack.h>

#include <caffe2/utils/threadpool/pthreadpool-cpp.h>
#include <ATen/native/ConvUtils.h>
#include <ATen/Parallel.h>
#include <c10/util/irange.h>

namespace at::native {

static bool init_nnpack() {
  const static nnp_status nnpack_status = nnp_initialize();
  auto nnpack_successfully_initialized_ = (nnp_status_success == nnpack_status);

  if (nnpack_status != nnp_status_success) {
    if (nnpack_status == nnp_status_out_of_memory) {
      LOG(WARNING) << "Could not initialize NNPACK! Reason: Out of memory.";
    } else if (nnpack_status == nnp_status_unsupported_hardware) {
      LOG(WARNING) << "Could not initialize NNPACK! Reason: Unsupported hardware.";
    } else {
      LOG(WARNING) << "Could not initialize NNPACK! Reason: Unknown error!";
    }
  }
```
- EN: Lines 1-60 pull in 15 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are AT_NNPACK_ENABLED, _nnpack_available, init_nnpack, concentrating a specific part of the operator behavior.
- CN: 第 1-60 行引入了 15 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 AT_NNPACK_ENABLED, _nnpack_available, init_nnpack，它们承载了某一部分算子行为的核心逻辑。

### Lines 61-120
```cpp
  return nnpack_successfully_initialized_;
}

static pthreadpool_t nnpack_threadpool() {
#ifdef C10_MOBILE
  return caffe2::pthreadpool_();
#else
  static pthreadpool_t nnpack_threadpool_ = nullptr;
  static bool called_nnpack_threadpool_ = false;

  if (!called_nnpack_threadpool_) {
    called_nnpack_threadpool_ = true;

#ifdef INTRA_OP_PARALLEL
    const uint32_t threads = at::get_num_threads();
#else
    const uint32_t threads = std::thread::hardware_concurrency();
#endif

    nnpack_threadpool_ = pthreadpool_create(threads);
    if (!nnpack_threadpool_) {
      LOG(WARNING) << "Failed to initialize pthreadpool! Running NNPACK in single-threaded mode.";
    }
  }

  return nnpack_threadpool_;
#endif
}

bool _nnpack_available() {
  return init_nnpack();
}

namespace {
struct Workspace {
  void* buffer = nullptr;
  size_t size = 0;

  void deallocate() {
    if (buffer) {
      // NOLINTNEXTLINE(cppcoreguidelines-no-malloc)
      std::free(buffer);
      buffer = nullptr;
    }
  }

  void allocate() {
    deallocate();

    // NNPack has alignment requirements
    constexpr size_t nnpack_memory_alignment_boundary = 64;

    // Won't work on Windows, but NNPACK doesn't support Windows either
    auto res = posix_memalign(&buffer, nnpack_memory_alignment_boundary, size);
    if (res != 0) {
      TORCH_CHECK(false, "posix_memalign failed:", c10::utils::str_error(errno), " (", errno, ")");
    }
    return;
  }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are nnpack_threadpool, _nnpack_available, deallocate, NOLINTNEXTLINE, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 nnpack_threadpool, _nnpack_available, deallocate, NOLINTNEXTLINE，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-180
```cpp
  ~Workspace() {
    deallocate();
  }
};
} // namespace

// Make thread_local for safety in cases where we have multiple threads running
// Convs at once
static thread_local Workspace workspace;

Tensor _nnpack_spatial_convolution(
    const Tensor& input,
    const Tensor& weight, const std::optional<Tensor>& bias_opt,
    const IntArrayRef padding,
    const IntArrayRef stride) {
  // See [Note: hacky wrapper removal for optional tensor]
  c10::MaybeOwned<Tensor> bias_maybe_owned = at::borrow_from_optional_tensor(bias_opt);
  const Tensor& bias = *bias_maybe_owned;

  at::Tensor output = at::empty(
      conv_output_size(input.sizes(), weight.sizes(), padding, stride),
      input.options());

  // Our input Tensor must be in the form N,C,H,W
  TORCH_CHECK(
      input.ndimension() == 4,
      "NNPack convolutionOutput expects 4D input Tensor N,C,H,W");

  // Our weight Tensor must be in the form oC,iC,kH,kW
  TORCH_CHECK(
      weight.ndimension() == 4,
      "NNPack convolutionOutput expects 4D weight Tensor oC,iC,kH,kW");

  // Our output Tensor must be in the form N,oC,oH,oW
  TORCH_CHECK(
      output.ndimension() == 4,
      "NNPack convolutionOutput expects 4D output Tensor N,oC,oH,oW");

  // Some basic shape checking, not comprehensive
  TORCH_CHECK(
      input.size(1) == weight.size(1),
      "Mismatch between number of input channels in input Tensor (",
      input.size(1),
      ") and weight Tensor (",
      weight.size(1),
      ") in NNPack convolutionOutput");

  TORCH_CHECK(
      weight.size(0) == output.size(1),
      "Mismatch between number of output channels in weight Tensor (",
      weight.size(0),
      ") and output Tensor (",
      output.size(1),
      ") in NNPack convolutionOutput");

  TORCH_CHECK(
      input.size(0) == output.size(0),
      "Mismatch between batch size in input Tensor (",
      input.size(0),
      ") and output Tensor (",
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are Workspace, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 Workspace，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 181-240
```cpp
      output.size(0),
      ") in NNPack convolutionOutput");

  // All Tensors must be float Tensors
  if (input.device().type() != kCPU || input.scalar_type() != kFloat ||
      weight.device().type() != kCPU || weight.scalar_type() != kFloat ||
      output.device().type() != kCPU || output.scalar_type() != kFloat ||
      (bias.defined() && (bias.device().type() != kCPU || bias.scalar_type() != kFloat))) {
    TORCH_CHECK(false, "Mismatched Tensor types in NNPack convolutionOutput");
  }

  const auto algorithm = nnp_convolution_algorithm_auto;
  const size_t input_channels = input.size(1);
  const size_t output_channels = weight.size(0);
  const struct nnp_size input_size = {
      .width = static_cast<size_t>(input.size(3)),
      .height = static_cast<size_t>(input.size(2)),
  };
  const struct nnp_padding input_padding = {
      .top = static_cast<size_t>(padding[0]),
      .right = static_cast<size_t>(padding[1]),
      .bottom = static_cast<size_t>(padding[0]),
      .left = static_cast<size_t>(padding[1]),
  };
  const struct nnp_size kernel_size = {
      .width = static_cast<size_t>(weight.size(3)),
      .height = static_cast<size_t>(weight.size(2)),
  };
  const struct nnp_size output_size = {
      .width = static_cast<size_t>(output.size(3)),
      .height = static_cast<size_t>(output.size(2)),
  };
  const nnp_size output_subsample = {
      .width = static_cast<std::size_t>(stride[1]),
      .height = static_cast<std::size_t>(stride[0]),
  };

  const auto input_ = input.contiguous();
  const auto weight_ = weight.contiguous();
  // If we don't have a defined bias Tensor, we need to create one filled with zeroes
  const auto bias_ = bias.defined() ? bias.contiguous() : at::zeros({weight.size(0)}, input.options());

  const auto compute = [&](const size_t batch_size) -> nnp_status {
    if ((batch_size == 1) || (output_subsample.width != 1) || (output_subsample.height != 1)) {
      const size_t input_size_per_batch = input_channels * input_size.width * input_size.height;
      const size_t output_size_per_batch = output_channels * output_size.width * output_size.height;

      for (const auto batch : c10::irange(0u, batch_size)) {
        const nnp_status status = nnp_convolution_inference(
            algorithm,
            nnp_convolution_transform_strategy_compute,
            input_channels,
            output_channels,
            input_size,
            input_padding,
            kernel_size,
            output_subsample,
            input_.const_data_ptr<float>() + batch * input_size_per_batch,
            weight_.const_data_ptr<float>(),
            bias_.const_data_ptr<float>(),
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are defined, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 defined，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 241-300
```cpp
            output.data_ptr<float>() + batch * output_size_per_batch,
            workspace.buffer,
            &workspace.size,
            nnp_activation_identity,
            nullptr,
            nnpack_threadpool(),
            nullptr );

        if (nnp_status_success != status) {
          return status;
        }
      }

      return nnp_status_success;
    }
    else {
      return nnp_convolution_output(
        algorithm,
        batch_size,
        input_channels,
        output_channels,
        input_size,
        input_padding,
        kernel_size,
        input_.const_data_ptr<float>(),
        weight_.const_data_ptr<float>(),
        bias_.const_data_ptr<float>(),
        output.data_ptr<float>(),
        workspace.buffer,
        &workspace.size,
        nnp_activation_identity,
        nullptr,
        nnpack_threadpool(),
        nullptr );
    }
  };

  const size_t batch_size = input.size(0);

  auto size_and_allocate_ws = [&]() {
    // Run a single pass to get the size of memory workspace buffer
    const auto status = compute(batch_size);
    TORCH_CHECK(
        status == nnp_status_success,
        "NNPACK SpatialConvolution_updateOutput failed");
    workspace.allocate();
  };

  // If no workspace created yet, allocate it
  if (workspace.buffer == nullptr) {
    size_and_allocate_ws();
  }

  // Try to run with the newly created, or existing workspace
  auto status = compute(batch_size);

  if (status == nnp_status_insufficient_buffer) {
    // Need to reallocate the workspace
    workspace.deallocate();
    size_and_allocate_ws();
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 301-315
```cpp

    // Try one more time
    status = compute(batch_size);
  }

  TORCH_CHECK(
      status == nnp_status_success,
      "NNPACK SpatialConvolution_updateOutput failed");

  return output;
}

} // namespace at::native

#endif // AT_NNPACK_ENABLED
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

## Key Concepts / 关键概念

- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: _nnpack_spatial_convolution, _nnpack_available, init_nnpack, nnpack_threadpool, deallocate, allocate, Workspace.
- CN: 重要符号：_nnpack_spatial_convolution, _nnpack_available, init_nnpack, nnpack_threadpool, deallocate, allocate, Workspace。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/Tensor.h, ATen/Config.h, c10/util/error.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_nnpack_available_native.h, ATen/ops/_nnpack_spatial_convolution_native.h, ATen/ops/empty.h, ATen/ops/zeros.h, ATen/native/ConvUtils.h, ATen/Parallel.h, c10/util/irange.h`.
- CN: 主要内部头文件：`ATen/core/Tensor.h, ATen/Config.h, c10/util/error.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_nnpack_available_native.h, ATen/ops/_nnpack_spatial_convolution_native.h, ATen/ops/empty.h, ATen/ops/zeros.h, ATen/native/ConvUtils.h, ATen/Parallel.h, c10/util/irange.h`。
- EN: External/system headers: `thread, nnpack.h, caffe2/utils/threadpool/pthreadpool-cpp.h`.
- CN: 外部/系统头文件：`thread, nnpack.h, caffe2/utils/threadpool/pthreadpool-cpp.h`。
- EN: The implementation revolves around symbols such as `_nnpack_spatial_convolution, _nnpack_available, init_nnpack, nnpack_threadpool, deallocate, allocate, Workspace`.
- CN: 实现围绕 `_nnpack_spatial_convolution, _nnpack_available, init_nnpack, nnpack_threadpool, deallocate, allocate, Workspace` 等符号展开。
