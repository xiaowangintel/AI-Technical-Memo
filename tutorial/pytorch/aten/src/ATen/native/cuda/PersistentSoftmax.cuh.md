# PersistentSoftmax.cuh — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/PersistentSoftmax.cuh`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Declares or defines CUDA helpers/templates associated with `log2_ceil`, `warp_reduce`, `softmax_warp_forward`, `softmax_warp_backward`.
- 用途（中文）: 声明或定义与 `log2_ceil`, `warp_reduce`, `softmax_warp_forward`, `softmax_warp_backward` 相关的 CUDA 辅助函数/模板。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```cpp
   1: #pragma once
   2: 
   3: #include <cfloat>
   4: #include <limits>
   5: #include <stdint.h>
   6: #include <cuda_fp16.h>
   7: #include <c10/macros/Macros.h>
   8: 
   9: #include <ATen/cuda/DeviceUtils.cuh>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<cfloat>`, `<limits>`, `<stdint.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<cfloat>`, `<limits>`, `<stdint.h>`。

### Lines 11-31
```cpp
  11: namespace {
  12: 
  13: int log2_ceil(int value) {
  14:     int log2_value = 0;
  15:     while ((1 << log2_value) < value) ++log2_value;
  16:     return log2_value;
  17: }
  18: 
  19: template<typename T>
  20: struct Add {
  21:   __device__ __forceinline__ T operator()(T a, T b) const {
  22:     return a + b;
  23:   }
  24: };
  25: 
  26: template<typename T>
  27: struct Max {
  28:   __device__ __forceinline__ T operator()(T a, T b) const {
  29:     return a < b ? b : a;
  30:   }
  31: };
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `log2_ceil`.
- CN: 该代码块定义或继续实现 `log2_ceil`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 33-44
```cpp
  33: template <typename acc_t, int WARP_BATCH, int WARP_SIZE, template<typename> class ReduceOp>
  34: __device__ __forceinline__ void warp_reduce(acc_t* sum) {
  35:     ReduceOp<acc_t> r;
  36:     #pragma unroll
  37:     for (int offset = WARP_SIZE / 2; offset > 0; offset /= 2) {
  38:         #pragma unroll
  39:         for (int i = 0;  i < WARP_BATCH;  ++i) {
  40:             acc_t b = WARP_SHFL_XOR(sum[i], offset, WARP_SIZE);
  41:             sum[i] = r(sum[i], b);
  42:         }
  43:     }
  44: }
```
- EN: This block defines or continues the implementation of `warp_reduce`.
- CN: 该代码块定义或继续实现 `warp_reduce`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 46-65
```cpp
  46: // The softmax_warp_* methods perform softmax forward and backward propagation on samples spanning the fast dimension.
  47: // Each sample contains element_count scalar elements. element_count can be any integer value <= 1024.
  48: // The template arguments have the following meaning:
  49: // One "WARP" works on one "BATCH". One "BATCH" contains "WARP_BATCH" samples.
  50: // WARP_BATCH is equal to 1 when element_count is large, and > 1 when element_count is small.
  51: // A "WARP" contains "C10_WARPS_SIZE" threads, these treads are guaranteed to belong to the same warp.
  52: // This is important because it means only __shfl_ instructions are required for reductions.
  53: // Note that this means WARP_SIZE must be a power of two and <= architecture warp size.
  54: // CUDA warp size is 32 for all existing GPU architectures, but there is no guarantee this will not change for future arch.
  55: // ROCm warp size is 64 for all currently ROCm-supported GPU architectures, but this may change for future archs.
  56: // is_log_softmax is a flag indicating whether SoftMax or LogSoftMax should be computed.
  57: // is_masked is a flag indicating whether SoftMax or MaskedSoftMax should be computed.
  58: // The template can be instantiated with any floating point type for the type arguments input_t, output_t and acc_t.
  59: // This allows SoftMax to be fused with a cast immediately following the SoftMax.
  60: // The mask should have the same shape as input, with a boolean indicate if the value is masked.
  61: // The head_chunk_size is only used for transformer mask softmax, equals to H * D * D.
  62: // For instance:
  63: // input_t=half,  acc_t=float, output_t=half  => read half tensor, float accumulators, write half tensor.
  64: // input_t=half,  acc_t=float, output_t=float => read half tensor, float accumulators, write float tensor.
  65: // input_t_float, acc_t=float, output_t=half  => read float tensor, float accumulators, write half tensor.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 67-88
```cpp
  67: template <typename input_t, typename output_t, typename acc_t, int log2_elements, bool is_log_softmax, bool is_masked, int WARP_SIZE_PARAM>
  68: __global__ void softmax_warp_forward(output_t *dst, const input_t *src, int batch_size, int stride, int element_count, const bool *mask = nullptr, const int head_chunk_size = -1, bool is_transformer_mask = false)
  69: {
  70:     // WARP_SIZE and WARP_BATCH must match the return values batches_per_warp and warp_size of method warp_softmax_forward_kernel.
  71:     constexpr int next_power_of_two = 1 << log2_elements;
  72:     constexpr int WARP_SIZE = (next_power_of_two < WARP_SIZE_PARAM) ? next_power_of_two : WARP_SIZE_PARAM;
  73:     constexpr int WARP_ITERATIONS = next_power_of_two / WARP_SIZE;
  74:     constexpr int WARP_BATCH = (next_power_of_two <= 128) ? 2 : 1;
  75: 
  76:     int first_batch = (blockDim.y * blockIdx.x + threadIdx.y) * WARP_BATCH;
  77: 
  78:     // batch_size might not be a multiple of WARP_BATCH. Check how
  79:     // many batches have to computed within this WARP.
  80:     int local_batches = batch_size - first_batch;
  81:     if (local_batches > WARP_BATCH)
  82:         local_batches = WARP_BATCH;
  83: 
  84:     // there might be multiple batches per warp. compute the index within the batch
  85:     int local_idx = threadIdx.x;
  86:     int idx_offset = first_batch * stride + local_idx;
  87: 
  88:     src += idx_offset;
```
- EN: This block defines GPU kernel entry point(s) `softmax_warp_forward`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `softmax_warp_forward`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 89-89
```cpp
  89:     dst += idx_offset;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 91-99
```cpp
  91:     if (is_transformer_mask) {
  92:         mask += ((first_batch * stride) / head_chunk_size) * stride + local_idx;
  93:     } else {
  94:         mask += idx_offset;
  95:     }
  96:     // The nested loops over WARP_BATCH and then WARP_ITERATIONS can be simplified to one loop,
  97:     // but I think doing so would obfuscate the logic of the algorithm, thus I chose to keep
  98:     // the nested loops.
  99:     // This should have no impact on performance because the loops are unrolled anyway.
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 101-101
```cpp
 101:     // load data from global memory
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 102-113
```cpp
 102:     acc_t elements[WARP_BATCH][WARP_ITERATIONS];
 103:     for (int i = 0;  i < WARP_BATCH;  ++i) {
 104:         int batch_element_count = (i >= local_batches) ? 0 : element_count;
 105:         for (int it = 0;  it < WARP_ITERATIONS;  ++it) {
 106:             int element_index = local_idx + it * WARP_SIZE;
 107:             if (element_index < batch_element_count) {
 108:                 elements[i][it] = src[i*element_count+it*WARP_SIZE];
 109:             } else {
 110:                 elements[i][it] = -std::numeric_limits<acc_t>::infinity();
 111:             }
 112:         }
 113:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 115-115
```cpp
 115:     // compute max_value
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 116-137
```cpp
 116:     acc_t max_value[WARP_BATCH];
 117:     #pragma unroll
 118:     for (int i = 0;  i < WARP_BATCH;  ++i) {
 119:         int batch_element_count = (i >= local_batches) ? 0 : element_count;
 120:         bool is_meaningful_max = false;
 121:         max_value[i] = elements[i][0];
 122:         #pragma unroll
 123:         for (int it = 0;  it < WARP_ITERATIONS;  ++it) {
 124:             if (is_masked) {
 125:                 int idx = it*WARP_SIZE;
 126:                 if ((idx + local_idx) < batch_element_count) {
 127:                     if (!is_transformer_mask) {
 128:                         idx += i*element_count;
 129:                     }
 130:                     if (!mask[idx]) {
 131:                         max_value[i] = (is_meaningful_max && max_value[i] > elements[i][it]) ? max_value[i] : elements[i][it];
 132:                         is_meaningful_max = true;
 133:                     }
 134:                 }
 135:             } else {
 136:                 max_value[i] = max_value[i] > elements[i][it] ? max_value[i] : elements[i][it];
 137:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 138-145
```cpp
 138:         }
 139:         if (is_masked) {
 140:             if (!is_meaningful_max) {
 141:                 max_value[i] = -std::numeric_limits<acc_t>::infinity();
 142:             }
 143:         }
 144:     }
 145:     warp_reduce<acc_t, WARP_BATCH, WARP_SIZE, Max>(max_value);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 147-168
```cpp
 147:     acc_t sum[WARP_BATCH] { 0.0f };
 148:     #pragma unroll
 149:     for (int i = 0;  i < WARP_BATCH;  ++i) {
 150:         int batch_element_count = (i >= local_batches) ? 0 : element_count;
 151:         #pragma unroll
 152:         for (int it = 0;  it < WARP_ITERATIONS;  ++it) {
 153:             if (!is_masked) {
 154:                 if (is_log_softmax) {
 155:                     sum[i] += std::exp(elements[i][it] - max_value[i]);
 156:                 } else {
 157:                     elements[i][it] = std::exp(elements[i][it] - max_value[i]);
 158:                     sum[i] += elements[i][it];
 159:                 }
 160:             } else {
 161:                 int idx = it*WARP_SIZE;
 162:                 bool valid = (idx + local_idx) < batch_element_count;
 163:                 if (!is_transformer_mask) {
 164:                     idx += i*element_count;
 165:                 }
 166:                 if (valid) {
 167:                     if (!mask[idx]) {
 168:                         if (is_log_softmax) {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 169-182
```cpp
 169:                             sum[i] += std::exp(elements[i][it] - max_value[i]);
 170:                         } else {
 171:                             elements[i][it] = std::exp(elements[i][it] - max_value[i]);
 172:                             sum[i] += elements[i][it];
 173:                         }
 174:                     } else {
 175:                         if (!is_log_softmax) {
 176:                             // Masked values are treated as -infinity, and std::exp(-infinity) is 0.
 177:                             elements[i][it] = 0;
 178:                         }
 179:                     }
 180:                 } else {
 181:                     if (!is_log_softmax) {
 182:                         elements[i][it] = 0.;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 183-188
```cpp
 183:                     }
 184:                 }
 185:             }
 186:         }
 187:     }
 188:     warp_reduce<acc_t, WARP_BATCH, WARP_SIZE, Add>(sum);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 190-190
```cpp
 190:     // store result
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 191-191
```cpp
 191:     #pragma unroll
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 192-211
```cpp
 192:     for (int i = 0;  i < WARP_BATCH;  ++i) {
 193:         if (i >= local_batches)
 194:             break;
 195:         if (is_log_softmax) sum[i] = std::log(sum[i]);
 196:         #pragma unroll
 197:         for (int it = 0;  it < WARP_ITERATIONS;  ++it) {
 198:             int element_index = local_idx + it * WARP_SIZE;
 199:             if (element_index < element_count) {
 200:                 if (is_log_softmax) {
 201:                     dst[i*element_count+it*WARP_SIZE] = elements[i][it] - max_value[i] - sum[i];
 202:                 } else if (sum[i] == 0) {
 203:                     dst[i*element_count+it*WARP_SIZE] = std::numeric_limits<acc_t>::quiet_NaN();
 204:                 } else {
 205:                     dst[i*element_count+it*WARP_SIZE] = elements[i][it] / sum[i];
 206:                 }
 207:             } else {
 208:                 break;
 209:             }
 210:         }
 211:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 212-212
```cpp
 212: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 214-235
```cpp
 214: template <typename input_t, typename output_t, typename acc_t, int log2_elements, bool is_log_softmax, bool is_masked, int WARP_SIZE_PARAM>
 215: __global__ void softmax_warp_backward(output_t *gradInput, const input_t *grad, const input_t *output, int batch_size, int stride, int element_count, const bool *mask = nullptr)
 216: {
 217:     // WARP_SIZE and WARP_BATCH must match the return values batches_per_warp and warp_size of method warp_softmax_backward_kernel.
 218:     constexpr int next_power_of_two = 1 << log2_elements;
 219:     constexpr int WARP_SIZE = (next_power_of_two < WARP_SIZE_PARAM) ? next_power_of_two : WARP_SIZE_PARAM;
 220:     constexpr int WARP_ITERATIONS = next_power_of_two / WARP_SIZE;
 221:     constexpr int WARP_BATCH = (next_power_of_two <= 128) ? 2 : 1;
 222: 
 223:     int first_batch = (blockDim.y * blockIdx.x + threadIdx.y) * WARP_BATCH;
 224: 
 225:     // batch_size might not be a multiple of WARP_BATCH. Check how
 226:     // many batches have to computed within this WARP.
 227:     int local_batches = batch_size - first_batch;
 228:     if (local_batches > WARP_BATCH)
 229:         local_batches = WARP_BATCH;
 230: 
 231:     // there might be multiple batches per warp. compute the index within the batch
 232:     int local_idx = threadIdx.x % WARP_SIZE;
 233: 
 234:     // the first element to process by the current thread
 235:     int thread_offset = first_batch * stride + local_idx;
```
- EN: This block defines GPU kernel entry point(s) `softmax_warp_backward`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `softmax_warp_backward`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 236-241
```cpp
 236:     grad += thread_offset;
 237:     output += thread_offset;
 238:     gradInput += thread_offset;
 239:     if (is_masked) {
 240:         mask += thread_offset;
 241:     }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 243-248
```cpp
 243:     // The nested loops over WARP_BATCH and then WARP_ITERATIONS can be simplified to one loop,
 244:     // but I think doing so would obfuscate the logic of the algorithm, thus I chose to keep
 245:     // the nested loops.
 246:     // This should have no impact on performance because the loops are unrolled anyway.
 247: 
 248:     // load data from global memory
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 249-263
```cpp
 249:     acc_t grad_reg[WARP_BATCH][WARP_ITERATIONS];
 250:     acc_t output_reg[WARP_BATCH][WARP_ITERATIONS];
 251:     for (int i = 0;  i < WARP_BATCH;  ++i) {
 252:         int batch_element_count = (i >= local_batches) ? 0 : element_count;
 253:         for (int it = 0;  it < WARP_ITERATIONS;  ++it) {
 254:             int element_index = local_idx + it * WARP_SIZE;
 255:             if (element_index < batch_element_count) {
 256:                 grad_reg[i][it] = grad[i*element_count+it*WARP_SIZE];
 257:                 output_reg[i][it] = output[i*element_count+it*WARP_SIZE];
 258:             } else {
 259:                 grad_reg[i][it] = acc_t(0);
 260:                 output_reg[i][it] = acc_t(0);
 261:             }
 262:         }
 263:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 265-275
```cpp
 265:     acc_t sum[WARP_BATCH] { 0.0f };
 266:     #pragma unroll
 267:     for (int i = 0;  i < WARP_BATCH;  ++i) {
 268:         #pragma unroll
 269:         for (int it = 0;  it < WARP_ITERATIONS;  ++it) {
 270:             if (!is_masked || !mask[i*element_count+it*WARP_SIZE]) {
 271:                 sum[i] += grad_reg[i][it];
 272:             }
 273:         }
 274:     }
 275:     warp_reduce<acc_t, WARP_BATCH, WARP_SIZE, Add>(sum);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 277-277
```cpp
 277:     // store result
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 278-278
```cpp
 278:     #pragma unroll
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 279-297
```cpp
 279:     for (int i = 0;  i < WARP_BATCH;  ++i) {
 280:         if (i >= local_batches)
 281:             break;
 282:         #pragma unroll
 283:         for (int it = 0;  it < WARP_ITERATIONS;  ++it) {
 284:             int element_index = local_idx + it * WARP_SIZE;
 285:             if (element_index < element_count) {
 286:                 if (is_masked && mask[i*element_count+it*WARP_SIZE]) {
 287:                     gradInput[i*element_count+it*WARP_SIZE] = 0;
 288:                 }
 289:                 // compute gradients
 290:                 else if (is_log_softmax) {
 291:                     gradInput[i*element_count+it*WARP_SIZE] = (grad_reg[i][it] - std::exp(output_reg[i][it]) * sum[i]);
 292:                 } else {
 293:                     gradInput[i*element_count+it*WARP_SIZE] = (grad_reg[i][it] - output_reg[i][it] * sum[i]);
 294:                 }
 295:             }
 296:         }
 297:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 298-298
```cpp
 298: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 300-300
```cpp
 300: } // end of anonymous namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 302-323
```cpp
 302: template<typename input_t, typename output_t, typename acc_t, bool is_log_softmax, bool is_masked>
 303: void dispatch_softmax_forward(output_t *dst, const input_t *src, int softmax_elements, int softmax_elements_stride, int batch_count, const bool *mask = nullptr, int chunk_size = -1, bool is_transformer_mask = false)
 304: {
 305:     TORCH_INTERNAL_ASSERT( softmax_elements >= 0 && softmax_elements <= 2048 );
 306:     if (softmax_elements == 0) {
 307:         return;
 308:     } else {
 309:         int log2_elements = log2_ceil(softmax_elements);
 310:         const int next_power_of_two = 1 << log2_elements;
 311: 
 312:         // This value must match the WARP_SIZE constexpr value computed inside softmax_warp_forward.
 313:         int warp_size = at::cuda::warp_size();
 314:         warp_size = (next_power_of_two < warp_size) ? next_power_of_two : warp_size;
 315: 
 316:         // This value must match the WARP_BATCH constexpr value computed inside softmax_warp_forward.
 317:         int batches_per_warp = (next_power_of_two <= 128) ? 2 : 1;
 318: 
 319:         // use 128 threads per block to maximize gpu utilization
 320:         constexpr int threads_per_block = 128;
 321: 
 322:         int warps_per_block = (threads_per_block / warp_size);
 323:         int batches_per_block = warps_per_block * batches_per_warp;
```
- EN: This block defines or continues the implementation of `dispatch_softmax_forward`.
- CN: 该代码块定义或继续实现 `dispatch_softmax_forward`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 324-345
```cpp
 324:         int blocks = (batch_count + batches_per_block - 1) / batches_per_block;
 325:         dim3 threads(warp_size, warps_per_block, 1);
 326:         // Launch code would be more elegant if C++ supported FOR CONSTEXPR
 327:         switch (log2_elements) {
 328: #ifdef USE_ROCM
 329:             // To support ROCm amdgcnspirv target, we must compile both a 32 and 64 warpSize version of each kernel
 330:             #define LAUNCH_SOFTMAX_WARP_FORWARD(L2E) case L2E:                    \
 331:             if (warp_size == 64) { \
 332:               softmax_warp_forward<input_t, output_t, acc_t, L2E, is_log_softmax, is_masked, 64>   \
 333:                   <<<blocks, threads, 0, at::cuda::getCurrentCUDAStream()>>>(dst,   \
 334:                       src, batch_count, softmax_elements_stride, softmax_elements, mask, chunk_size, is_transformer_mask); \
 335:             } \
 336:             else { \
 337:               softmax_warp_forward<input_t, output_t, acc_t, L2E, is_log_softmax, is_masked, 32>   \
 338:                   <<<blocks, threads, 0, at::cuda::getCurrentCUDAStream()>>>(dst,   \
 339:                       src, batch_count, softmax_elements_stride, softmax_elements, mask, chunk_size, is_transformer_mask); \
 340:             } \
 341:             C10_CUDA_KERNEL_LAUNCH_CHECK();                                       \
 342:             break;
 343: #else
 344:             #define LAUNCH_SOFTMAX_WARP_FORWARD(L2E) case L2E:                    \
 345:             softmax_warp_forward<input_t, output_t, acc_t, L2E, is_log_softmax, is_masked, 32>   \
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 346-350
```cpp
 346:                 <<<blocks, threads, 0, at::cuda::getCurrentCUDAStream()>>>(dst,   \
 347:                     src, batch_count, softmax_elements_stride, softmax_elements, mask, chunk_size, is_transformer_mask); \
 348:             C10_CUDA_KERNEL_LAUNCH_CHECK();                                       \
 349:             break;
 350: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 352-365
```cpp
 352:             LAUNCH_SOFTMAX_WARP_FORWARD(0);  // 1
 353:             LAUNCH_SOFTMAX_WARP_FORWARD(1);  // 2
 354:             LAUNCH_SOFTMAX_WARP_FORWARD(2);  // 4
 355:             LAUNCH_SOFTMAX_WARP_FORWARD(3);  // 8
 356:             LAUNCH_SOFTMAX_WARP_FORWARD(4);  // 16
 357:             LAUNCH_SOFTMAX_WARP_FORWARD(5);  // 32
 358:             LAUNCH_SOFTMAX_WARP_FORWARD(6);  // 64
 359:             LAUNCH_SOFTMAX_WARP_FORWARD(7);  // 128
 360:             LAUNCH_SOFTMAX_WARP_FORWARD(8);  // 256
 361:             LAUNCH_SOFTMAX_WARP_FORWARD(9);  // 512
 362:             LAUNCH_SOFTMAX_WARP_FORWARD(10); // 1024
 363:             LAUNCH_SOFTMAX_WARP_FORWARD(11); // 2048
 364:             default:
 365:                 break;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 366-368
```cpp
 366:         }
 367:     }
 368: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 370-391
```cpp
 370: template<typename input_t, typename output_t, typename acc_t, bool is_log_softmax, bool is_masked>
 371: void dispatch_softmax_backward(output_t *grad_input, const input_t *grad, const input_t *output, int softmax_elements, int softmax_elements_stride, int batch_count, const bool *mask = nullptr)
 372: {
 373:     TORCH_INTERNAL_ASSERT( softmax_elements >= 0 && softmax_elements <= 1024 );
 374:     if (softmax_elements == 0) {
 375:        return;
 376:     } else {
 377:         int log2_elements = log2_ceil(softmax_elements);
 378:         const int next_power_of_two = 1 << log2_elements;
 379: 
 380:         // This value must match the WARP_SIZE constexpr value computed inside softmax_warp_backward.
 381:         int warp_size = at::cuda::warp_size();
 382:         warp_size = (next_power_of_two < warp_size) ? next_power_of_two : warp_size;
 383: 
 384:         // This value must match the WARP_BATCH constexpr value computed inside softmax_warp_backward.
 385:         int batches_per_warp = (next_power_of_two <= 128) ? 2 : 1;
 386: 
 387:         // use 128 threads per block to maximize gpu utilization
 388:         constexpr int threads_per_block = 128;
 389: 
 390:         int warps_per_block = (threads_per_block / warp_size);
 391:         int batches_per_block = warps_per_block * batches_per_warp;
```
- EN: This block defines or continues the implementation of `dispatch_softmax_backward`.
- CN: 该代码块定义或继续实现 `dispatch_softmax_backward`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 392-413
```cpp
 392:         int blocks = (batch_count + batches_per_block - 1) / batches_per_block;
 393:         dim3 threads(warp_size, warps_per_block, 1);
 394:         // Launch code would be more elegant if C++ supported FOR CONSTEXPR
 395:         switch (log2_elements) {
 396: #ifdef USE_ROCM
 397:             // To support ROCm amdgcnspirv target, we must compile both a 32 and 64 warpSize version of each kernel
 398:             #define LAUNCH_SOFTMAX_WARP_BACKWARD(L2E) case L2E:                      \
 399:             if (warp_size == 64) { \
 400:               softmax_warp_backward<input_t, output_t, acc_t, L2E, is_log_softmax, is_masked, 64> \
 401:                   <<<blocks, threads, 0, at::cuda::getCurrentCUDAStream()>>>       \
 402:                   (grad_input, grad, output, batch_count, softmax_elements_stride, \
 403:                   softmax_elements, mask);                                              \
 404:             } \
 405:             else { \
 406:               softmax_warp_backward<input_t, output_t, acc_t, L2E, is_log_softmax, is_masked, 32> \
 407:                   <<<blocks, threads, 0, at::cuda::getCurrentCUDAStream()>>>       \
 408:                   (grad_input, grad, output, batch_count, softmax_elements_stride, \
 409:                   softmax_elements, mask);                                              \
 410:             } \
 411:             C10_CUDA_KERNEL_LAUNCH_CHECK();                                      \
 412:             break;
 413: #else
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 414-414
```cpp
 414:             #define LAUNCH_SOFTMAX_WARP_BACKWARD(L2E) case L2E:                      \
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 415-421
```cpp
 415:             softmax_warp_backward<input_t, output_t, acc_t, L2E, is_log_softmax, is_masked, 32> \
 416:                 <<<blocks, threads, 0, at::cuda::getCurrentCUDAStream()>>>       \
 417:                 (grad_input, grad, output, batch_count, softmax_elements_stride, \
 418:                 softmax_elements, mask);                                              \
 419:             C10_CUDA_KERNEL_LAUNCH_CHECK();                                      \
 420:             break;
 421: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 423-436
```cpp
 423:             LAUNCH_SOFTMAX_WARP_BACKWARD(0); // 1
 424:             LAUNCH_SOFTMAX_WARP_BACKWARD(1); // 2
 425:             LAUNCH_SOFTMAX_WARP_BACKWARD(2); // 4
 426:             LAUNCH_SOFTMAX_WARP_BACKWARD(3); // 8
 427:             LAUNCH_SOFTMAX_WARP_BACKWARD(4); // 16
 428:             LAUNCH_SOFTMAX_WARP_BACKWARD(5); // 32
 429:             LAUNCH_SOFTMAX_WARP_BACKWARD(6); // 64
 430:             LAUNCH_SOFTMAX_WARP_BACKWARD(7); // 128
 431:             LAUNCH_SOFTMAX_WARP_BACKWARD(8); // 256
 432:             LAUNCH_SOFTMAX_WARP_BACKWARD(9); // 512
 433:             LAUNCH_SOFTMAX_WARP_BACKWARD(10); // 1024
 434:             default:
 435:                 break;
 436:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 437-438
```cpp
 437:     }
 438: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

## Key Concepts / 关键概念

- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<cfloat>`
  - `<limits>`
  - `<stdint.h>`
  - `<cuda_fp16.h>`
  - `<c10/macros/Macros.h>`
  - `<ATen/cuda/DeviceUtils.cuh>`
- Runtime symbols / 运行时符号:
  - `at::cuda::warp_size`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
