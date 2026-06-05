# UpSampleNearest1d.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/UpSampleNearest1d.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `upsample_nearest1d_out_cuda`, `_upsample_nearest_exact1d_out_cuda`, `upsample_nearest1d_backward_out_cuda`, `_upsample_nearest_exact1d_backward_out_cuda`.
- 用途（中文）: 实现与 `upsample_nearest1d_out_cuda`, `_upsample_nearest_exact1d_out_cuda`, `upsample_nearest1d_backward_out_cuda`, `_upsample_nearest_exact1d_backward_out_cuda` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/AccumulateType.h>
   4: #include <ATen/ceil_div.h>
   5: #include <ATen/Dispatch.h>
   6: #include <ATen/TensorUtils.h>
   7: #include <ATen/Utils.h>
   8: #include <ATen/cuda/CUDAContext.h>
   9: #include <ATen/native/cuda/UpSample.cuh>
  10: 
  11: #ifndef AT_PER_OPERATOR_HEADERS
  12: #include <ATen/Functions.h>
  13: #include <ATen/NativeFunctions.h>
  14: #else
  15: #include <ATen/ops/upsample_nearest1d_native.h>
  16: #include <ATen/ops/upsample_nearest1d_backward_native.h>
  17: #include <ATen/ops/_upsample_nearest_exact1d_native.h>
  18: #include <ATen/ops/_upsample_nearest_exact1d_backward_native.h>
  19: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/ceil_div.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/ceil_div.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 21-42
```cpp
  21: namespace at::native {
  22: namespace {
  23: 
  24: #define MAX_THREADS 512
  25: 
  26: // Define a typedef to dispatch to nearest_neighbor_compute_source_index or
  27: // nearest_neighbor_exact_compute_source_index
  28: typedef int (*nn_compute_source_index_fn_t)(const float, int, int);
  29: 
  30: // Define a typedef to dispatch to nearest_neighbor_bw_compute_source_index or
  31: // nearest_neighbor_exact_bw_compute_source_index
  32: typedef int (*nn_bw_compute_source_index_fn_t)(const float, int, int);
  33: 
  34: 
  35: // see NOTE [ Nearest neighbor upsampling kernel implementation ]
  36: template <typename scalar_t, nn_compute_source_index_fn_t nn_compute_source_index_fn>
  37: C10_LAUNCH_BOUNDS_1(1024)
  38: __global__ void upsample_nearest1d_out_frame(
  39:     const scalar_t* input,
  40:     size_t dim_b,
  41:     size_t dim_c,
  42:     size_t src_dim_w,
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `upsample_nearest1d_out_frame`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `upsample_nearest1d_out_frame`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 43-64
```cpp
  43:     size_t dst_dim_w,
  44:     scalar_t* output,
  45:     float scale_factor) {
  46:   int dst_idx = blockIdx.x * blockDim.x + threadIdx.x;
  47:   if (dst_idx >= dim_c * dst_dim_w)
  48:     return;
  49: 
  50:   int c = (dst_idx / dst_dim_w) % dim_c;
  51: 
  52:   int dst_x = dst_idx % dst_dim_w;
  53:   int src_x = nn_compute_source_index_fn(scale_factor, dst_x, src_dim_w);
  54: 
  55:   int src_idx = c * src_dim_w + src_x;
  56:   int src_stride = dim_c * src_dim_w;
  57:   int dst_stride = dim_c * dst_dim_w;
  58: 
  59:   for (int b = 0; b < dim_b; b++) {
  60:     output[dst_idx] = input[src_idx];
  61:     src_idx += src_stride;
  62:     dst_idx += dst_stride;
  63:   }
  64: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 66-67
```cpp
  66: // see NOTE [ Nearest neighbor upsampling kernel implementation ]
  67: // Backward operation
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 68-89
```cpp
  68: template <typename scalar_t, typename accscalar_t, nn_bw_compute_source_index_fn_t nn_bw_compute_source_index_fn>
  69: C10_LAUNCH_BOUNDS_1(1024)
  70: __global__ void upsample_nearest1d_backward_out_frame(
  71:     const scalar_t* grad_o,
  72:     size_t dim_b,
  73:     size_t dim_c,
  74:     size_t src_dim_w,
  75:     size_t dst_dim_w,
  76:     scalar_t* grad_i,
  77:     float scale_factor) {
  78: 
  79:   int dst_idx = blockIdx.x * blockDim.x + threadIdx.x;
  80:   if (dst_idx >= dim_c * dst_dim_w)
  81:     return;
  82: 
  83:   int c = (dst_idx / (dst_dim_w)) % dim_c;
  84: 
  85:   int dst_x = dst_idx % dst_dim_w;
  86:   // note that we do not want to clamp src_x to src_dim_w, since we might
  87:   // intentionally want to skip in case of scale_factor < 1.0
  88:   int src_x = nn_bw_compute_source_index_fn(scale_factor, dst_x, src_dim_w);
  89:   int src_x_up = nn_bw_compute_source_index_fn(scale_factor, dst_x+1, src_dim_w);
```
- EN: This block defines GPU kernel entry point(s) `upsample_nearest1d_backward_out_frame`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `upsample_nearest1d_backward_out_frame`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 91-100
```cpp
  91:   for (int b = 0; b < dim_b; b++) {
  92:     accscalar_t grad = 0;
  93:     int src_idx = b * dim_c * src_dim_w + c * src_dim_w + src_x;
  94:     for (int x = src_x; x < src_x_up; x++) {
  95:       grad += grad_o[src_idx++];
  96:     }
  97:     grad_i[dst_idx] = grad;
  98:     dst_idx += dim_c * dst_dim_w;
  99:   }
 100: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 102-123
```cpp
 102: template<nn_compute_source_index_fn_t nn_compute_source_index_fn>
 103: static void upsample_nearest1d_out_cuda_template(
 104:     const Tensor& output,
 105:     const Tensor& input_,
 106:     IntArrayRef output_size,
 107:     std::optional<double> scales) {
 108:   TensorArg input_arg{input_, "input_", 1}, output_arg{output, "output", 2};
 109:   checkAllSameGPU("upsample_nearest1d_out_cuda", {input_arg, output_arg});
 110: 
 111:   int output_width = output_size[0];
 112: 
 113:   int nbatch = input_.size(0);
 114:   int channels = input_.size(1);
 115:   int input_width = input_.size(2);
 116: 
 117:   Tensor input = input_.contiguous();
 118: 
 119:   if (input.numel() == 0) {
 120:     return;
 121:   }
 122: 
 123:   // upsample_nearest1d meta call makes sure `nbatch != 0`
```
- EN: This block defines or continues the implementation of `upsample_nearest1d_out_cuda_template`.
- CN: 该代码块定义或继续实现 `upsample_nearest1d_out_cuda_template`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 124-129
```cpp
 124:   unsigned int n = output.numel() / nbatch;
 125:   dim3 bdim{std::min<unsigned int>(
 126:       at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock, MAX_THREADS)};
 127:   dim3 gdim{ceil_div(n, bdim.x)};
 128:   // safe check for int32 indexing; implicitly restrict launch config for kernel
 129:   TORCH_CHECK(output.numel() <= std::numeric_limits<int32_t>::max());
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 131-144
```cpp
 131:   cudaStream_t stream = at::cuda::getCurrentCUDAStream();
 132:   AT_DISPATCH_FLOATING_TYPES_AND3(ScalarType::Half, ScalarType::BFloat16, ScalarType::Byte, input.scalar_type(), "upsample_nearest1d_out_frame", [&] {
 133:         using accscalar_t = at::acc_type<scalar_t, true>;
 134: 
 135:         auto idata = input.const_data_ptr<scalar_t>();
 136:         auto odata = output.mutable_data_ptr<scalar_t>();
 137: 
 138:         const float scale_factor = compute_scales_value<float>(scales, input_width, output_width);
 139: 
 140:         upsample_nearest1d_out_frame<scalar_t, nn_compute_source_index_fn><<<gdim, bdim, 0, stream>>>(
 141:             idata, nbatch, channels, input_width, output_width, odata, scale_factor);
 142:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 143:       });
 144: }
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 146-167
```cpp
 146: template<nn_compute_source_index_fn_t nn_bw_compute_source_index_fn>
 147: static void upsample_nearest1d_backward_out_cuda_template(
 148:     const Tensor& grad_input,
 149:     const Tensor& grad_output_,
 150:     IntArrayRef output_size,
 151:     IntArrayRef input_size,
 152:     std::optional<double> scales) {
 153:   TensorArg grad_input_arg{grad_input, "grad_input", 1},
 154:       grad_output_arg{grad_output_, "grad_output_", 2};
 155:   checkAllSameGPU(
 156:       "upsample_nearest1d_backward_out_cuda_template",
 157:       {grad_output_arg, grad_input_arg});
 158: 
 159:   int output_width = output_size[0];
 160: 
 161:   int nbatch = input_size[0];
 162:   int channels = input_size[1];
 163:   int input_width = input_size[2];
 164: 
 165:   Tensor grad_output = grad_output_.contiguous();
 166: 
 167:   if (grad_input.numel() == 0) {
```
- EN: This block defines or continues the implementation of `upsample_nearest1d_backward_out_cuda_template`.
- CN: 该代码块定义或继续实现 `upsample_nearest1d_backward_out_cuda_template`。

### Lines 168-169
```cpp
 168:     return;
 169:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 171-171
```cpp
 171:   // upsample_nearest1d meta call makes sure `nbatch != 0`
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 172-180
```cpp
 172:   unsigned int n = grad_input.numel() / nbatch;
 173:   dim3 bdim{std::min<unsigned int>(
 174:       at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock, MAX_THREADS)};
 175:   dim3 gdim{ceil_div(n, bdim.x)};
 176:   // safe check for int32 indexing; implicitly restrict launch config for kernel
 177:   TORCH_CHECK(grad_input.numel() <= std::numeric_limits<int32_t>::max(),
 178:     "upsample_nearest1d_backward only supports input tensors with less than INT_MAX elements, but got ", grad_input.sizes());
 179:   TORCH_CHECK(grad_output.numel() <= std::numeric_limits<int32_t>::max(),
 180:         "upsample_nearest1d_backward only supports output tensors with less than INT_MAX elements, but got ", grad_output.sizes());
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 182-195
```cpp
 182:   cudaStream_t stream = at::cuda::getCurrentCUDAStream();
 183:   AT_DISPATCH_FLOATING_TYPES_AND3(ScalarType::Half, ScalarType::BFloat16, ScalarType::Byte, grad_output.scalar_type(), "upsample_nearest1d_backward_out_frame", [&] {
 184:         using accscalar_t = at::acc_type<scalar_t, true>;
 185: 
 186:         auto idata = grad_input.mutable_data_ptr<scalar_t>();
 187:         auto odata = grad_output.const_data_ptr<scalar_t>();
 188: 
 189:         const float scale_factor = compute_scales_value_backwards<float>(scales, output_width, input_width);
 190: 
 191:         upsample_nearest1d_backward_out_frame<scalar_t, accscalar_t, nn_bw_compute_source_index_fn>
 192:             <<<gdim, bdim, 0, stream>>>(
 193:                 odata, nbatch, channels, output_width, input_width, idata, scale_factor);
 194:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 195:       });
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 196-196
```cpp
 196: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 198-198
```cpp
 198: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 200-208
```cpp
 200: TORCH_IMPL_FUNC(upsample_nearest1d_out_cuda) (
 201:     const Tensor& input,
 202:     IntArrayRef output_size,
 203:     std::optional<double> scales,
 204:     const Tensor& output
 205: ) {
 206:   upsample_nearest1d_out_cuda_template<nearest_neighbor_compute_source_index>(
 207:       output, input, output_size, scales);
 208: }
```
- EN: This block defines or continues the implementation of `upsample_nearest1d_out_cuda`.
- CN: 该代码块定义或继续实现 `upsample_nearest1d_out_cuda`。

### Lines 210-217
```cpp
 210: TORCH_IMPL_FUNC(_upsample_nearest_exact1d_out_cuda) (
 211:     const Tensor& input,
 212:     IntArrayRef output_size,
 213:     std::optional<double> scales,
 214:     const Tensor& output
 215: ) {
 216:   upsample_nearest1d_out_cuda_template<nearest_neighbor_exact_compute_source_index>(output, input, output_size, scales);
 217: }
```
- EN: This block defines or continues the implementation of `_upsample_nearest_exact1d_out_cuda`.
- CN: 该代码块定义或继续实现 `_upsample_nearest_exact1d_out_cuda`。

### Lines 219-228
```cpp
 219: TORCH_IMPL_FUNC(upsample_nearest1d_backward_out_cuda) (
 220:     const Tensor& grad_output,
 221:     IntArrayRef output_size,
 222:     IntArrayRef input_size,
 223:     std::optional<double> scales,
 224:     const Tensor& grad_input
 225: ) {
 226:   upsample_nearest1d_backward_out_cuda_template<nearest_neighbor_bw_compute_source_index>(
 227:       grad_input, grad_output, output_size, input_size, scales);
 228: }
```
- EN: This block defines or continues the implementation of `upsample_nearest1d_backward_out_cuda`.
- CN: 该代码块定义或继续实现 `upsample_nearest1d_backward_out_cuda`。

### Lines 230-239
```cpp
 230: TORCH_IMPL_FUNC(_upsample_nearest_exact1d_backward_out_cuda) (
 231:     const Tensor& grad_output,
 232:     IntArrayRef output_size,
 233:     IntArrayRef input_size,
 234:     std::optional<double> scales,
 235:     const Tensor& grad_input
 236: ) {
 237:   upsample_nearest1d_backward_out_cuda_template<nearest_neighbor_exact_bw_compute_source_index>(
 238:       grad_input, grad_output, output_size, input_size, scales);
 239: }
```
- EN: This block defines or continues the implementation of `_upsample_nearest_exact1d_backward_out_cuda`.
- CN: 该代码块定义或继续实现 `_upsample_nearest_exact1d_backward_out_cuda`。

### Lines 241-241
```cpp
 241: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_IMPL_FUNC` provides an out/inplace-style structured kernel implementation. / `TORCH_IMPL_FUNC` 提供结构化的 out/inplace 风格内核实现。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/AccumulateType.h>`
  - `<ATen/ceil_div.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/TensorUtils.h>`
  - `<ATen/Utils.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/native/cuda/UpSample.cuh>`
  - `<ATen/Functions.h>`
  - `<ATen/NativeFunctions.h>`
  - `<ATen/ops/upsample_nearest1d_native.h>`
  - `<ATen/ops/upsample_nearest1d_backward_native.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_FLOATING_TYPES_AND3`
  - `TORCH_IMPL_FUNC`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
