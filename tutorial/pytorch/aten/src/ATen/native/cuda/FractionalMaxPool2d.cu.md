# FractionalMaxPool2d.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/FractionalMaxPool2d.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `fractional_max_pool2d_out_cuda`, `fractional_max_pool2d_backward_cuda`, `get_interval`, `fractional_max_pool2d_out_cuda_frame`.
- 用途（中文）: 实现与 `fractional_max_pool2d_out_cuda`, `fractional_max_pool2d_backward_cuda`, `get_interval`, `fractional_max_pool2d_out_cuda_frame` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/AccumulateType.h>
   4: #include <ATen/Dispatch.h>
   5: #include <ATen/cuda/Atomic.cuh>
   6: #include <ATen/cuda/CUDAContext.h>
   7: #include <ATen/cuda/NumericLimits.cuh>
   8: #include <ATen/cuda/detail/IndexUtils.cuh>
   9: #include <ATen/cuda/detail/KernelUtils.h>
  10: #include <ATen/NumericUtils.h>
  11: #include <ATen/TensorUtils.h>
  12: #include <ATen/Utils.h>
  13: #include <ATen/native/FractionalMaxPooling.h>
  14: #include <c10/macros/Macros.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/Dispatch.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/Dispatch.h>`。

### Lines 15-25
```cpp
  15: #include <c10/util/Exception.h>
  16: #ifndef AT_PER_OPERATOR_HEADERS
  17: #include <ATen/NativeFunctions.h>
  18: #else
  19: #include <ATen/ops/fractional_max_pool2d_backward_native.h>
  20: #include <ATen/ops/fractional_max_pool2d_native.h>
  21: #endif
  22: 
  23: #include <algorithm>
  24: #include <cfloat>
  25: #include <cmath>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<c10/util/Exception.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/fractional_max_pool2d_backward_native.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<c10/util/Exception.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/fractional_max_pool2d_backward_native.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 27-48
```cpp
  27: namespace at::native {
  28: 
  29: using namespace at::cuda::detail;
  30: 
  31: namespace {
  32: 
  33: template <typename scalar_t, typename accscalar_t>
  34: __device__ inline int get_interval(accscalar_t sample,
  35:   int index, int inputSize, int outputSize, int poolSize) {
  36:   accscalar_t alpha = static_cast<accscalar_t>(inputSize - poolSize) /
  37:     static_cast<accscalar_t>(outputSize - 1);
  38:   if (index == outputSize - 1) {
  39:     return inputSize - poolSize;
  40:   } else {
  41:     return static_cast<int>((index + sample) * alpha) -
  42:       static_cast<int>(sample * alpha);
  43:   }
  44: }
  45: 
  46: template <typename scalar_t>
  47: __global__ void fractional_max_pool2d_out_cuda_frame(
  48:   PackedTensorAccessor<scalar_t, 4> output,
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `fractional_max_pool2d_out_cuda_frame`, `get_interval`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `fractional_max_pool2d_out_cuda_frame`, `get_interval`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 49-70
```cpp
  49:   PackedTensorAccessor<int64_t, 4> indices,
  50:   PackedTensorAccessor<const scalar_t, 4> input,
  51:   PackedTensorAccessor<const scalar_t, 3> samples,
  52:   int poolSizeH, int poolSizeW) {
  53: 
  54:   using accscalar_t = at::acc_type<scalar_t, /*is_cuda=*/true>;
  55: 
  56:   int ourOutputPoint = threadIdx.x + blockIdx.x * blockDim.x;
  57:   int plane = blockIdx.y;
  58:   int batch = blockIdx.z;
  59: 
  60:   // Each thread generates a specific output point
  61:   if (ourOutputPoint < output.size(2) * output.size(3)) {
  62:     int outputW = ourOutputPoint % output.size(3);
  63:     int outputH = ourOutputPoint / output.size(3);
  64: 
  65:     int poolW = get_interval<scalar_t, accscalar_t>(
  66:       static_cast<accscalar_t>(samples[batch][plane][0]),
  67:         outputW, input.size(3), output.size(3), poolSizeW);
  68:     int poolH = get_interval<scalar_t, accscalar_t>(
  69:       static_cast<accscalar_t>(samples[batch][plane][1]),
  70:         outputH, input.size(2), output.size(2), poolSizeH);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 72-73
```cpp
  72:     scalar_t maxVal = at::numeric_limits<scalar_t>::lower_bound();
  73:     int maxIndex = poolH * input.size(3) + poolW;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 75-96
```cpp
  75:     for (int h = poolH; h < poolH + poolSizeH; ++h) {
  76:       if (poolSizeW < 2 || poolSizeW > 7) {
  77:         for (int w = poolW; w < poolW + poolSizeW; ++w) {
  78:           scalar_t val = input[batch][plane][h][w];
  79:           // for consistency with THNN, favor the first max
  80:           if (val > maxVal || at::_isnan(val)) {
  81:             maxIndex = h * input.size(3) + w;
  82:             maxVal = val;
  83:           }
  84:         }
  85:       } else {
  86:         for (int i = 0; i < poolSizeW; ++i) {
  87:           int w = i + poolW;
  88:           scalar_t val = input[batch][plane][h][w];
  89:           // for consistency with THNN, favor the first max
  90:           if (val > maxVal || at::_isnan(val)) {
  91:             maxIndex = h * input.size(3) + w;
  92:             maxVal = val;
  93:           }
  94:         }
  95:       }
  96:     }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 98-101
```cpp
  98:     indices[batch][plane][outputH][outputW] = maxIndex;
  99:     output[batch][plane][outputH][outputW] = maxVal;
 100:   }
 101: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 103-123
```cpp
 103: template <typename scalar_t>
 104: __global__ void fractional_max_pool2d_backward_out_cuda_frame(
 105:   PackedTensorAccessor<scalar_t, 4> gradInput,
 106:   PackedTensorAccessor<const scalar_t, 4> gradOutput,
 107:   PackedTensorAccessor<const int64_t, 4> indices) {
 108:   // Output (h, w) point that this thread is responsible for
 109:   int ourOutputPoint = threadIdx.x + blockIdx.x * blockDim.x;
 110:   int plane = blockIdx.y;
 111:   int batch = blockIdx.z;
 112: 
 113:   // Each thread generates a specific output point
 114:   if (ourOutputPoint < gradOutput.size(2) *
 115:     gradOutput.size(3)) {
 116:     int outputW = ourOutputPoint % gradOutput.size(3);
 117:     int outputH = ourOutputPoint / gradOutput.size(3);
 118: 
 119:     int index = indices[batch][plane][outputH][outputW];
 120:     CUDA_KERNEL_ASSERT(index >= 0);
 121:     int inputW = index % gradInput.size(3);
 122:     int inputH = index / gradInput.size(3);
 123:     CUDA_KERNEL_ASSERT(inputH < gradInput.size(2));
```
- EN: This block defines GPU kernel entry point(s) `fractional_max_pool2d_backward_out_cuda_frame`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `fractional_max_pool2d_backward_out_cuda_frame`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 125-130
```cpp
 125:     gpuAtomicAddNoReturn(
 126:       &gradInput[batch][plane][inputH][inputW],
 127:       gradOutput[batch][plane][outputH][outputW]
 128:     );
 129:   }
 130: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 132-132
```cpp
 132: } // anonymous namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 134-155
```cpp
 134: TORCH_IMPL_FUNC(fractional_max_pool2d_out_cuda) (
 135:   const Tensor& input,
 136:   IntArrayRef pool_size,
 137:   IntArrayRef output_size,
 138:   const Tensor& randomSamples,
 139:   const Tensor& output,
 140:   const Tensor& indices
 141: ) {
 142:   fractional_max_pool_check_shape</*ndim*/ 2>(input, randomSamples);
 143: 
 144:   int planeDim = 0;
 145: 
 146:   int ndims = input.ndimension();
 147: 
 148:   if (ndims == 4) {
 149:     planeDim++;
 150:   }
 151: 
 152:   /* sizes */
 153:   int numPlanes = input.size(planeDim);
 154: 
 155:   int outputH = output_size[0];
```
- EN: This block defines or continues the implementation of `fractional_max_pool2d_out_cuda`.
- CN: 该代码块定义或继续实现 `fractional_max_pool2d_out_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 156-158
```cpp
 156:   int outputW = output_size[1];
 157:   int poolSizeH = pool_size[0];
 158:   int poolSizeW = pool_size[1];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 160-162
```cpp
 160:   auto output_ = output;
 161:   auto input_ = input;
 162:   auto indices_ = indices;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 164-168
```cpp
 164:   if(ndims == 3) {
 165:     output_ = output_.reshape({1, numPlanes, outputH, outputW});
 166:     indices_ = indices_.reshape({1, numPlanes, outputH, outputW});
 167:     input_ = input_.reshape({1, input.size(0), input.size(1), input.size(2)});
 168:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 170-172
```cpp
 170:   if (output_.numel() == 0) {
 171:     return;
 172:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 174-175
```cpp
 174:   // block is limited to 4 warps
 175:   // grid handles overflow per each plane
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 176-181
```cpp
 176:   int outputPlaneSize = output_.size(2) *
 177:     output_.size(3);
 178:   dim3 grid((outputPlaneSize + 127) / 128, // ceil(outputPlaneSize / 128)
 179:             input_.size(1),
 180:             input_.size(0));
 181:   dim3 block(outputPlaneSize > 128 ? 128 : outputPlaneSize);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 183-198
```cpp
 183:   AT_DISPATCH_FLOATING_TYPES_AND2(
 184:     at::ScalarType::Half,
 185:     at::ScalarType::BFloat16,
 186:     input.scalar_type(),
 187:     "fractional_max_pool2d_out_cuda_frame",
 188:     [&] {
 189:       auto devInput = input_.packed_accessor64<const scalar_t, 4>();
 190:       auto devOutput = output_.packed_accessor64<scalar_t, 4>();
 191:       auto devIndices = indices_.packed_accessor64<int64_t, 4>();
 192:       auto devSamples = randomSamples.packed_accessor64<const scalar_t, 3>();
 193:       fractional_max_pool2d_out_cuda_frame<scalar_t>
 194:         <<<grid, block, 0, at::cuda::getCurrentCUDAStream()>>>(
 195:           devOutput, devIndices, devInput, devSamples,
 196:           poolSizeH, poolSizeW);
 197:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 198:      }
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 199-200
```cpp
 199:    );
 200: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 202-222
```cpp
 202: TORCH_IMPL_FUNC(fractional_max_pool2d_backward_cuda)(
 203:   const Tensor& gradOutput,
 204:   const Tensor& input,
 205:   IntArrayRef pool_size /* unused */,
 206:   IntArrayRef output_size,
 207:   const Tensor& indices,
 208:   const Tensor& gradInput)
 209: {
 210: 
 211:   // See Note [Writing Nondeterministic Operations]
 212:   // Nondeterministic because of atomicAdd usage
 213:   globalContext().alertNotDeterministic("fractional_max_pool2d_backward_cuda");
 214: 
 215:   int dimh = 1;
 216:   int dimw = 2;
 217: 
 218:   int ndims = input.ndimension();
 219:   if (ndims == 4) {
 220:     dimh++;
 221:     dimw++;
 222:   }
```
- EN: This block defines or continues the implementation of `fractional_max_pool2d_backward_cuda`.
- CN: 该代码块定义或继续实现 `fractional_max_pool2d_backward_cuda`。

### Lines 224-224
```cpp
 224:   /* sizes */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 225-226
```cpp
 225:   int inputH = input.size(dimh);
 226:   int inputW = input.size(dimw);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 228-229
```cpp
 228:   int outputH = output_size[0];
 229:   int outputW = output_size[1];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 231-233
```cpp
 231:   if (gradInput.numel() == 0) {
 232:     return;
 233:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 235-235
```cpp
 235:   gradInput.zero_();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 237-239
```cpp
 237:   auto gradInput_ = gradInput;
 238:   auto gradOutput_ = gradOutput;
 239:   auto indices_ = indices;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 241-245
```cpp
 241:   if(ndims == 3) {
 242:     gradInput_ = gradInput_.reshape({1, input.size(0), inputH, inputW});
 243:     gradOutput_ = gradOutput_.reshape({1, gradOutput.size(0), outputH, outputW});
 244:     indices_ = indices_.reshape({1, indices_.size(0), outputH, outputW});
 245:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 247-249
```cpp
 247:   /* backprop */
 248:   // block is limited to 4 warps
 249:   // grid handles overflow per each plane
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 250-255
```cpp
 250:   int outputPlaneSize = gradOutput_.size(2) *
 251:     gradOutput_.size(3);
 252:   dim3 grid((outputPlaneSize + 127) / 128, // ceil(outputPlaneSize / 128)
 253:             gradInput_.size(1),
 254:             gradInput_.size(0));
 255:   dim3 block(outputPlaneSize > 128 ? 128 : outputPlaneSize);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 257-270
```cpp
 257:   auto devIndices = indices_.packed_accessor64<const int64_t, 4>();
 258:   AT_DISPATCH_FLOATING_TYPES_AND2(
 259:     at::ScalarType::Half,
 260:     at::ScalarType::BFloat16,
 261:     gradOutput.scalar_type(),
 262:     "fractional_max_pool2d_backward_out_cuda_frame",
 263:     [&] {
 264:       auto devGradInput = gradInput_.packed_accessor64<scalar_t, 4>();
 265:       auto devGradOutput = gradOutput_.packed_accessor64<const scalar_t, 4>();
 266:       fractional_max_pool2d_backward_out_cuda_frame<scalar_t>
 267:         <<<grid, block, 0, at::cuda::getCurrentCUDAStream()>>>(
 268:         devGradInput, devGradOutput, devIndices);
 269:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 270:     }
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 271-272
```cpp
 271:   );
 272: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 274-274
```cpp
 274: }// at::native
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_IMPL_FUNC` provides an out/inplace-style structured kernel implementation. / `TORCH_IMPL_FUNC` 提供结构化的 out/inplace 风格内核实现。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/AccumulateType.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/cuda/Atomic.cuh>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/NumericLimits.cuh>`
  - `<ATen/cuda/detail/IndexUtils.cuh>`
  - `<ATen/cuda/detail/KernelUtils.h>`
  - `<ATen/NumericUtils.h>`
  - `<ATen/TensorUtils.h>`
  - `<ATen/Utils.h>`
  - `<ATen/native/FractionalMaxPooling.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `TORCH_IMPL_FUNC`
  - `at::cuda::detail`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
