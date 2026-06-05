# NLLLoss2d.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/NLLLoss2d.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `optional_contiguous`, `optional_data`, `check_inputs_nll_loss2d`, `nll_loss2d_forward_out_cuda_template`.
- 用途（中文）: 实现与 `optional_contiguous`, `optional_data`, `check_inputs_nll_loss2d`, `nll_loss2d_forward_out_cuda_template` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/AccumulateType.h>
   4: #include <ATen/Dispatch.h>
   5: #include <ATen/TensorUtils.h>
   6: #include <ATen/cuda/Atomic.cuh>
   7: #include <ATen/cuda/CUDAContext.h>
   8: #include <ATen/core/TensorAccessor.h>
   9: #include <ATen/cuda/detail/KernelUtils.h>
  10: #include <c10/cuda/CUDAException.h>
  11: #include <c10/macros/Macros.h>
  12: #include <ATen/native/IndexingUtils.h>
  13: #include <ATen/native/Resize.h>
  14: #include <ATen/native/cuda/block_reduce.cuh>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/Dispatch.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/Dispatch.h>`。

### Lines 15-24
```cpp
  15: 
  16: #ifndef AT_PER_OPERATOR_HEADERS
  17: #include <ATen/Functions.h>
  18: #include <ATen/NativeFunctions.h>
  19: #else
  20: #include <ATen/ops/empty.h>
  21: #include <ATen/ops/empty_like.h>
  22: #include <ATen/ops/nll_loss2d_forward_native.h>
  23: #include <ATen/ops/nll_loss2d_backward_native.h>
  24: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/empty.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/empty.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 26-47
```cpp
  26: namespace at::native {
  27: 
  28: namespace {
  29: 
  30: // Returns a contiguous tensor if the source tensor
  31: // is defined. Otherwise returns the undefined
  32: // source tensor unmodified.
  33: inline Tensor optional_contiguous(const Tensor& source) {
  34:   return source.defined() ? source.contiguous() : source;
  35: }
  36: 
  37: // Returns the address of the first element of a tensor
  38: // or nullptr if the tensor is undefined.
  39: template <typename scalar_t>
  40: inline const scalar_t* optional_data(const Tensor& source) {
  41:   return source.defined() ? source.const_data_ptr<scalar_t>() : nullptr;
  42: }
  43: 
  44: using at::cuda::detail::CUDA_NUM_THREADS;
  45: using at::cuda::detail::GET_BLOCKS;
  46: 
  47: // TODO(crcrpar): Think about introducing `canUse32BitIndexMath` and choose int or int64_t for `target`.
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `optional_contiguous`, `optional_data`.
- CN: 该代码块定义或继续实现 `optional_contiguous`, `optional_data`。

### Lines 48-69
```cpp
  48: template <typename scalar_t>
  49: C10_LAUNCH_BOUNDS_1(CUDA_NUM_THREADS)
  50: __global__ void nll_loss2d_forward_no_reduce_kernel(
  51:   int64_t n_threads,
  52:   PackedTensorAccessor64<scalar_t, 4> input,
  53:   PackedTensorAccessor64<int64_t, 3> target,
  54:   PackedTensorAccessor64<scalar_t, 3> output,
  55:   const scalar_t* weight,
  56:   int64_t ignore_index
  57: ) {
  58:   int64_t batch_size = input.size(0);
  59:   int64_t n_classes = input.size(1);
  60:   int64_t H = input.size(2);
  61:   int64_t W = input.size(3);
  62: 
  63:   CUDA_KERNEL_LOOP(index, n_threads) {
  64:     const int64_t b = index % batch_size;
  65:     const int64_t h = (index / batch_size) % H;
  66:     const int64_t w = (index / (batch_size * H)) % W;
  67: 
  68:     int64_t cur_target = target[b][h][w];
  69:     if (cur_target == ignore_index) {
```
- EN: This block defines GPU kernel entry point(s) `nll_loss2d_forward_no_reduce_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `nll_loss2d_forward_no_reduce_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 70-78
```cpp
  70:       output[b][h][w] = static_cast<scalar_t>(0);
  71:       continue;
  72:     }
  73:     CUDA_KERNEL_ASSERT(cur_target >= 0 && cur_target < n_classes);
  74:     scalar_t value = input[b][cur_target][h][w];
  75:     scalar_t cur_weight = weight != nullptr ? weight[cur_target] : static_cast<scalar_t>(1);
  76:     output[b][h][w] = -value * cur_weight;
  77:   }
  78: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 80-101
```cpp
  80: template <typename scalar_t, typename accscalar_t, typename index_t>
  81: C10_LAUNCH_BOUNDS_1(CUDA_NUM_THREADS)
  82: __global__ void nll_loss2d_forward_kernel(
  83:   scalar_t* output,
  84:   scalar_t* total_weight,
  85:   const scalar_t* input,
  86:   const int64_t* target,
  87:   const scalar_t* weight,
  88:   int n_classes,
  89:   int map_nelem,
  90:   int blocks_per_sample,
  91:   int64_t ignore_index) {
  92: 
  93:   scalar_t cur_weight;
  94:   accscalar_t input_sum = 0;
  95:   accscalar_t acc_weight = 0;
  96: 
  97:   index_t sample = blockIdx.x / blocks_per_sample;
  98:   index_t toffset = sample * map_nelem;
  99:   index_t ioffset = sample * map_nelem * n_classes;
 100:   int step = blockDim.x * blocks_per_sample;
 101:   for (int i = (blockIdx.x % blocks_per_sample) * blockDim.x + threadIdx.x;
```
- EN: This block defines GPU kernel entry point(s) `nll_loss2d_forward_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `nll_loss2d_forward_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 102-113
```cpp
 102:        i < map_nelem;
 103:        i += step) {
 104:     index_t t = target[toffset + i];
 105:     if (t != ignore_index) {
 106:       CUDA_KERNEL_ASSERT(t >= 0 && t < n_classes);
 107:       cur_weight = weight != nullptr ? weight[t] : static_cast<scalar_t>(1);
 108:       const auto input_index = ioffset + i + map_nelem * t;
 109:       CUDA_KERNEL_ASSERT(input_index >= 0);
 110:       input_sum -= input[input_index] * cur_weight;
 111:       acc_weight += cur_weight;
 112:     }
 113:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 115-116
```cpp
 115:   __shared__ accscalar_t acc_weight_smem[CUDA_NUM_THREADS];
 116:   __shared__ accscalar_t input_sum_smem[CUDA_NUM_THREADS];
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 118-119
```cpp
 118:   auto acc_weight_ = cuda_utils::BlockReduceSum(acc_weight, acc_weight_smem);
 119:   auto input_sum_ = cuda_utils::BlockReduceSum(input_sum, input_sum_smem);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 121-125
```cpp
 121:   if (threadIdx.x == 0) {
 122:     gpuAtomicAdd(total_weight, static_cast<scalar_t>(acc_weight_));
 123:     gpuAtomicAdd(output, static_cast<scalar_t>(input_sum_));
 124:   }
 125: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 127-134
```cpp
 127: template <typename scalar_t>
 128: C10_LAUNCH_BOUNDS_1(CUDA_NUM_THREADS)
 129: __global__ void nll_loss2d_forward_size_average_kernel(
 130:   scalar_t* output,
 131:   const scalar_t* total_weight
 132: ) {
 133:   *output /= *total_weight;
 134: }
```
- EN: This block defines GPU kernel entry point(s) `nll_loss2d_forward_size_average_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `nll_loss2d_forward_size_average_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 136-157
```cpp
 136: template <typename scalar_t>
 137: C10_LAUNCH_BOUNDS_1(CUDA_NUM_THREADS)
 138: __global__ void nll_loss2d_backward_no_reduce_kernel(
 139:   int64_t n_threads,
 140:   PackedTensorAccessor64<int64_t, 3> target,
 141:   PackedTensorAccessor64<scalar_t, 3> grad_output,
 142:   PackedTensorAccessor64<scalar_t, 4> grad_input,
 143:   const scalar_t* weight,
 144:   int64_t ignore_index
 145: ) {
 146:   int64_t batch_size = target.size(0);
 147:   int64_t H = target.size(1);
 148:   int64_t W = target.size(2);
 149:   int64_t n_classes = grad_input.size(1);
 150: 
 151:   CUDA_KERNEL_LOOP(index, n_threads) {
 152:     const int64_t b = index % batch_size;
 153:     const int64_t h = (index / batch_size) % H;
 154:     const int64_t w = (index / (batch_size * H)) % W;
 155: 
 156:     int64_t cur_target = target[b][h][w];
 157:     if (cur_target == ignore_index) {
```
- EN: This block defines GPU kernel entry point(s) `nll_loss2d_backward_no_reduce_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `nll_loss2d_backward_no_reduce_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 158-164
```cpp
 158:       continue;
 159:     }
 160:     CUDA_KERNEL_ASSERT(cur_target >= 0 && cur_target < n_classes);
 161:     scalar_t value = -(weight != nullptr ? weight[cur_target] : static_cast<scalar_t>(1));
 162:     grad_input[b][cur_target][h][w] = value * grad_output[b][h][w];
 163:   }
 164: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 166-187
```cpp
 166: template <typename scalar_t>
 167: C10_LAUNCH_BOUNDS_1(CUDA_NUM_THREADS)
 168: __global__ void nll_loss2d_backward_kernel(
 169:   scalar_t* grad_input,
 170:   const scalar_t* grad_output,
 171:   const int64_t* target,
 172:   const scalar_t* weights,
 173:   const scalar_t* total_weight,
 174:   bool size_average,
 175:   int n_classes,
 176:   int map_nelem,
 177:   int blocks_per_sample,
 178:   int64_t ignore_index
 179: ) {
 180:   const auto grad = -(size_average ? *grad_output / *total_weight
 181:                                    : *grad_output);
 182: 
 183:   const int sample = blockIdx.x / blocks_per_sample;
 184:   const int step = blockDim.x * blocks_per_sample;
 185: 
 186:   const int toffset = sample * map_nelem;
 187:   const auto* const target_thread = target + toffset;
```
- EN: This block defines GPU kernel entry point(s) `nll_loss2d_backward_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `nll_loss2d_backward_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 189-190
```cpp
 189:   const int ioffset = sample * map_nelem * n_classes;
 190:   auto* const grad_input_thread = grad_input + ioffset;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 192-204
```cpp
 192:   for (int i = (blockIdx.x % blocks_per_sample) * blockDim.x + threadIdx.x;
 193:        i < map_nelem;
 194:        i += step) {
 195:     const int64_t t = target_thread[i];
 196:     if (t != ignore_index) {
 197:       CUDA_KERNEL_ASSERT(t >= 0 && t < n_classes);
 198:       const auto grad_input_index = i + map_nelem * t;
 199:       CUDA_KERNEL_ASSERT(grad_input_index >= 0);
 200:       grad_input_thread[i + map_nelem * t] = weights != nullptr ? weights[t] * grad
 201:                                                                 : grad;
 202:     }
 203:   }
 204: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 206-227
```cpp
 206: void check_inputs_nll_loss2d(
 207:     const Tensor& input,
 208:     const Tensor& target,
 209:     const Tensor& weight) {
 210:   TORCH_CHECK(
 211:       target.dim() == 3,
 212:       "only batches of spatial targets supported (3D tensors)"
 213:       " but got targets of size: : ",
 214:       target.sizes());
 215:   TORCH_CHECK(
 216:       input.dim() == 4,
 217:       "only batches of spatial inputs supported (4D tensors), "
 218:       "but got input of size: ",
 219:       input.sizes());
 220:   TORCH_CHECK(
 221:       !weight.defined() || weight.numel() == input.size(1),
 222:       "weight tensor should be defined either for all or no classes");
 223: 
 224:   TORCH_CHECK(
 225:       input.size(0) == target.size(0) && input.size(2) == target.size(1) &&
 226:           input.size(3) == target.size(2),
 227:       "input and target batch or spatial sizes don't match: target ",
```
- EN: This block defines or continues the implementation of `check_inputs_nll_loss2d`.
- CN: 该代码块定义或继续实现 `check_inputs_nll_loss2d`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 228-231
```cpp
 228:       target.sizes(),
 229:       ", input ",
 230:       input.sizes());
 231: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 233-253
```cpp
 233: void nll_loss2d_forward_out_cuda_template(
 234:     Tensor& output,
 235:     Tensor& total_weight,
 236:     const Tensor& input,
 237:     const Tensor& target,
 238:     const std::optional<Tensor>& weight_opt,
 239:     int64_t reduction,
 240:     int64_t ignore_index) {
 241:   // See Note [Writing Nondeterministic Operations]
 242:   // Nondeterministic because of atomicAdd usage in 'sum' or 'mean' reductions.
 243:   if (reduction != at::Reduction::None) {
 244:     at::globalContext().alertNotDeterministic("nll_loss2d_forward_out_cuda_template");
 245:   }
 246: 
 247:   // See [Note: hacky wrapper removal for optional tensor]
 248:   c10::MaybeOwned<Tensor> weight_maybe_owned =
 249:       at::borrow_from_optional_tensor(weight_opt);
 250:   const Tensor& weight = *weight_maybe_owned;
 251: 
 252:   check_inputs_nll_loss2d(input, target, weight);
 253:   total_weight.resize_({});
```
- EN: This block defines or continues the implementation of `nll_loss2d_forward_out_cuda_template`.
- CN: 该代码块定义或继续实现 `nll_loss2d_forward_out_cuda_template`。

### Lines 255-276
```cpp
 255:   if (reduction == at::Reduction::None) {
 256:     int64_t batch_size = input.size(0);
 257:     int64_t H = input.size(2);
 258:     int64_t W = input.size(3);
 259:     int64_t count = batch_size * H * W;
 260: 
 261:     at::native::resize_output(output, {batch_size, H, W});
 262:     if (count == 0) {
 263:       // This guards from unnecessary operations and launching CUDA kernel with
 264:       // 0 blocks.
 265:       return;
 266:     }
 267:     auto weight_ = optional_contiguous(weight);
 268:     AT_DISPATCH_FLOATING_TYPES_AND2(
 269:         at::ScalarType::Half,
 270:         at::ScalarType::BFloat16,
 271:         input.scalar_type(),
 272:         "nll_loss2d_forward_no_reduce_kernel",
 273:         [&] {
 274:           nll_loss2d_forward_no_reduce_kernel<scalar_t>
 275:               <<<GET_BLOCKS(count),
 276:                  CUDA_NUM_THREADS,
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 277-288
```cpp
 277:                  0,
 278:                  at::cuda::getCurrentCUDAStream()>>>(
 279:                   count,
 280:                   input.packed_accessor64<scalar_t, 4>(),
 281:                   target.packed_accessor64<int64_t, 3>(),
 282:                   output.packed_accessor64<scalar_t, 3>(),
 283:                   optional_data<scalar_t>(weight_),
 284:                   ignore_index);
 285:           C10_CUDA_KERNEL_LAUNCH_CHECK();
 286:         });
 287:     return;
 288:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 290-290
```cpp
 290:   // produce scalar outputs for the reduction case
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 291-291
```cpp
 291:   at::native::resize_output(output, {});
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 293-304
```cpp
 293:   if (target.numel() == 0) {
 294:     // Here target (and input) have zero elements
 295:     // Mean reduction on empty tensors produces NaN. See the discussion in
 296:     // https://github.com/pytorch/pytorch/pull/64572#issuecomment-926504162
 297:     if (reduction == Reduction::Mean) {
 298:       output.fill_(std::numeric_limits<double>::quiet_NaN());
 299:     } else {
 300:       output.zero_();
 301:     }
 302:     total_weight.zero_();
 303:     return;
 304:   }
```
- EN: This block defines or continues the implementation of `target`.
- CN: 该代码块定义或继续实现 `target`。

### Lines 306-308
```cpp
 306:   auto input_ = input.contiguous();
 307:   auto weight_ = optional_contiguous(weight);
 308:   auto target_ = target.contiguous();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 310-311
```cpp
 310:   output.zero_();
 311:   total_weight.zero_();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 313-317
```cpp
 313:   auto batch_size = target.size(0);
 314:   int64_t map_nelem = target.numel() / batch_size;
 315:   int blocks_per_sample = GET_BLOCKS(map_nelem) / 128;
 316:   blocks_per_sample = (blocks_per_sample == 0) ? 1 : blocks_per_sample;
 317:   int total_blocks = blocks_per_sample * batch_size;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 319-340
```cpp
 319:   AT_DISPATCH_FLOATING_TYPES_AND2(
 320:       at::ScalarType::Half,
 321:       at::ScalarType::BFloat16,
 322:       input.scalar_type(),
 323:       "nll_loss2d_forward_kernel",
 324:       [&] {
 325:         using accscalar_t = acc_type<scalar_t, true>;
 326:     AT_DISPATCH_INDEX_TYPES(
 327:         at::native::canUse32BitIndexMath(input_, INT_MAX) ? ScalarType::Int : ScalarType::Long,
 328:         "nll_loss2d_forward_launcher", [&] {
 329:             nll_loss2d_forward_kernel<scalar_t, accscalar_t, index_t>
 330:                 <<<total_blocks,
 331:                   CUDA_NUM_THREADS,
 332:                   0,
 333:                   at::cuda::getCurrentCUDAStream()>>>(
 334:                     output.mutable_data_ptr<scalar_t>(),
 335:                     total_weight.mutable_data_ptr<scalar_t>(),
 336:                     input_.const_data_ptr<scalar_t>(),
 337:                     target_.const_data_ptr<int64_t>(),
 338:                     optional_data<scalar_t>(weight_),
 339:                     input_.size(1),
 340:                     input_.size(2) * input_.size(3),
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 341-354
```cpp
 341:                     blocks_per_sample,
 342:                     ignore_index);
 343:             C10_CUDA_KERNEL_LAUNCH_CHECK();
 344:             // Divide by total_weight
 345:             if (reduction == at::Reduction::Mean) {
 346:               nll_loss2d_forward_size_average_kernel<scalar_t>
 347:                   <<<1, 1, 0, at::cuda::getCurrentCUDAStream()>>>(
 348:                       output.mutable_data_ptr<scalar_t>(),
 349:                       total_weight.const_data_ptr<scalar_t>());
 350:               C10_CUDA_KERNEL_LAUNCH_CHECK();
 351:             }
 352:     });
 353:       });
 354: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 356-377
```cpp
 356: void nll_loss2d_backward_out_cuda_template(
 357:     Tensor& grad_input,
 358:     const Tensor& grad_output,
 359:     const Tensor& input,
 360:     const Tensor& target,
 361:     const std::optional<Tensor>& weight_opt,
 362:     int64_t reduction,
 363:     int64_t ignore_index,
 364:     const Tensor& total_weight) {
 365:   // See [Note: hacky wrapper removal for optional tensor]
 366:   c10::MaybeOwned<Tensor> weight_maybe_owned =
 367:       at::borrow_from_optional_tensor(weight_opt);
 368:   const Tensor& weight = *weight_maybe_owned;
 369: 
 370:   check_inputs_nll_loss2d(input, target, weight);
 371:   grad_input.resize_as_(input);
 372:   grad_input.zero_();
 373:   TORCH_CHECK(grad_input.is_contiguous(), "grad_input must be contiguous");
 374:   TORCH_CHECK(
 375:       total_weight.numel() == 1,
 376:       "expected total_weight to be a single element tensor, got: ",
 377:       total_weight.sizes(),
```
- EN: This block defines or continues the implementation of `nll_loss2d_backward_out_cuda_template`.
- CN: 该代码块定义或继续实现 `nll_loss2d_backward_out_cuda_template`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 378-380
```cpp
 378:       " (",
 379:       total_weight.numel(),
 380:       " elements)");
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 383-404
```cpp
 383:   if (reduction == at::Reduction::None) {
 384:     TORCH_CHECK(
 385:         grad_output.dim() == 3,
 386:         "grad_output must have same dimension as target (3) but got dimension: ",
 387:         grad_output.sizes());
 388:     TORCH_CHECK(
 389:         grad_output.size(0) == target.size(0) &&
 390:             grad_output.size(1) == target.size(1) &&
 391:             grad_output.size(2) == target.size(2),
 392:         "grad_output sizes don't match target sizes: target ",
 393:         target.sizes(),
 394:         ", grad_output ",
 395:         grad_output.sizes())
 396:     int64_t batch_size = input.size(0);
 397:     int64_t H = input.size(2);
 398:     int64_t W = input.size(3);
 399:     int64_t count = batch_size * H * W;
 400: 
 401:     if (count == 0) {
 402:       // This guards from unnecessary operations and launching CUDA kernel with
 403:       // 0 blocks.
 404:       return;
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 405-418
```cpp
 405:     }
 406:     auto weight_ = optional_contiguous(weight);
 407:     AT_DISPATCH_FLOATING_TYPES_AND2(
 408:         at::ScalarType::Half,
 409:         at::ScalarType::BFloat16,
 410:         input.scalar_type(),
 411:         "nll_loss2d_backward_no_reduce_kernel",
 412:         [&] {
 413:           nll_loss2d_backward_no_reduce_kernel<scalar_t>
 414:               <<<GET_BLOCKS(count),
 415:                  CUDA_NUM_THREADS,
 416:                  0,
 417:                  at::cuda::getCurrentCUDAStream()>>>(
 418:                   count,
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 419-427
```cpp
 419:                   target.packed_accessor64<int64_t, 3>(),
 420:                   grad_output.packed_accessor64<scalar_t, 3>(),
 421:                   grad_input.packed_accessor64<scalar_t, 4>(),
 422:                   optional_data<scalar_t>(weight_),
 423:                   ignore_index);
 424:           C10_CUDA_KERNEL_LAUNCH_CHECK();
 425:         });
 426:     return;
 427:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 429-450
```cpp
 429:   int64_t batch_size = target.size(0);
 430:   auto target_numel = target.numel();
 431:   if (batch_size != 0 && target_numel != 0) {
 432:     // This guards from unnecessary operations and launching CUDA kernel with 1
 433:     // blocks.
 434:     auto target_ = target.contiguous();
 435:     auto weight_ = optional_contiguous(weight);
 436: 
 437:     int64_t map_nelem = target_numel / batch_size;
 438:     int blocks_per_sample = GET_BLOCKS(map_nelem) / 128;
 439:     blocks_per_sample = (blocks_per_sample == 0) ? 1 : blocks_per_sample;
 440:     int total_blocks = blocks_per_sample * batch_size;
 441: 
 442:     AT_DISPATCH_FLOATING_TYPES_AND2(
 443:         at::ScalarType::Half,
 444:         at::ScalarType::BFloat16,
 445:         input.scalar_type(),
 446:         "nll_loss2d_backward_kernel",
 447:         [&] {
 448:           nll_loss2d_backward_kernel<scalar_t>
 449:               <<<total_blocks,
 450:                 CUDA_NUM_THREADS,
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 451-464
```cpp
 451:                 0,
 452:                 at::cuda::getCurrentCUDAStream()>>>(
 453:                   grad_input.mutable_data_ptr<scalar_t>(),
 454:                   grad_output.const_data_ptr<scalar_t>(),
 455:                   target_.const_data_ptr<int64_t>(),
 456:                   optional_data<scalar_t>(weight_),
 457:                   total_weight.const_data_ptr<scalar_t>(),
 458:                   reduction == at::Reduction::Mean,
 459:                   input.size(1),
 460:                   map_nelem,
 461:                   blocks_per_sample,
 462:                   ignore_index);
 463:           C10_CUDA_KERNEL_LAUNCH_CHECK();
 464:         });
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 465-467
```cpp
 465:   }
 466: }
 467: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 469-480
```cpp
 469: std::tuple<Tensor&, Tensor&> nll_loss2d_forward_out_cuda(
 470:     const Tensor& self,
 471:     const Tensor& target,
 472:     const std::optional<Tensor>& weight_opt,
 473:     int64_t reduction,
 474:     int64_t ignore_index,
 475:     Tensor& output,
 476:     Tensor& total_weight) {
 477:   nll_loss2d_forward_out_cuda_template(
 478:       output, total_weight, self, target, weight_opt, reduction, ignore_index);
 479:   return std::tuple<Tensor&, Tensor&>(output, total_weight);
 480: }
```
- EN: This block defines or continues the implementation of `nll_loss2d_forward_out_cuda`.
- CN: 该代码块定义或继续实现 `nll_loss2d_forward_out_cuda`。

### Lines 482-493
```cpp
 482: std::tuple<Tensor, Tensor> nll_loss2d_forward_cuda(
 483:     const Tensor& self,
 484:     const Tensor& target,
 485:     const std::optional<Tensor>& weight_opt,
 486:     int64_t reduction,
 487:     int64_t ignore_index) {
 488:   auto output = at::empty({0}, self.options());
 489:   auto total_weight = at::empty({0}, self.options());
 490:   nll_loss2d_forward_out_cuda_template(
 491:       output, total_weight, self, target, weight_opt, reduction, ignore_index);
 492:   return std::make_tuple(output, total_weight);
 493: }
```
- EN: This block defines or continues the implementation of `nll_loss2d_forward_cuda`.
- CN: 该代码块定义或继续实现 `nll_loss2d_forward_cuda`。

### Lines 495-514
```cpp
 495: Tensor& nll_loss2d_backward_out_cuda(
 496:     const Tensor& grad_output,
 497:     const Tensor& self,
 498:     const Tensor& target,
 499:     const std::optional<Tensor>& weight_opt,
 500:     int64_t reduction,
 501:     int64_t ignore_index,
 502:     const Tensor& total_weight,
 503:     Tensor& grad_input) {
 504:   nll_loss2d_backward_out_cuda_template(
 505:       grad_input,
 506:       grad_output,
 507:       self,
 508:       target,
 509:       weight_opt,
 510:       reduction,
 511:       ignore_index,
 512:       total_weight);
 513:   return grad_input;
 514: }
```
- EN: This block defines or continues the implementation of `nll_loss2d_backward_out_cuda`.
- CN: 该代码块定义或继续实现 `nll_loss2d_backward_out_cuda`。

### Lines 516-535
```cpp
 516: Tensor nll_loss2d_backward_cuda(
 517:     const Tensor& grad_output,
 518:     const Tensor& self,
 519:     const Tensor& target,
 520:     const std::optional<Tensor>& weight_opt,
 521:     int64_t reduction,
 522:     int64_t ignore_index,
 523:     const Tensor& total_weight) {
 524:   auto grad_input = at::empty_like(self);
 525:   nll_loss2d_backward_out_cuda_template(
 526:       grad_input,
 527:       grad_output,
 528:       self,
 529:       target,
 530:       weight_opt,
 531:       reduction,
 532:       ignore_index,
 533:       total_weight);
 534:   return grad_input;
 535: }
```
- EN: This block defines or continues the implementation of `nll_loss2d_backward_cuda`.
- CN: 该代码块定义或继续实现 `nll_loss2d_backward_cuda`。

### Lines 537-537
```cpp
 537: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `CUDA_KERNEL_LOOP` expands index-space iteration on the GPU. / `CUDA_KERNEL_LOOP` 展开 GPU 上的索引空间循环。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/AccumulateType.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/TensorUtils.h>`
  - `<ATen/cuda/Atomic.cuh>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/core/TensorAccessor.h>`
  - `<ATen/cuda/detail/KernelUtils.h>`
  - `<c10/cuda/CUDAException.h>`
  - `<c10/macros/Macros.h>`
  - `<ATen/native/IndexingUtils.h>`
  - `<ATen/native/Resize.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `AT_DISPATCH_INDEX_TYPES`
  - `at::cuda::detail::CUDA_NUM_THREADS`
  - `at::cuda::detail::GET_BLOCKS`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
