# LossCTC.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/LossCTC.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `get_target_prime`, `ctc_loss_gpu_template`, `ctc_loss_backward_gpu_template`, `ctc_loss_gpu`.
- 用途（中文）: 实现与 `get_target_prime`, `ctc_loss_gpu_template`, `ctc_loss_backward_gpu_template`, `ctc_loss_gpu` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```cpp
   1: // Copyright (c) 2018 MathInf GmbH, Thomas Viehmann
   2: // Licensed under the BSD-3-Clause license
   3: // This is the GPU implementation of the Connectionist Temporal Loss.
   4: // We mostly follow Graves.
   5: // 1. Graves et al.: http://www.cs.toronto.edu/~graves/icml_2006.pdf
   6: // We use the equations from above link, but note that [1] has 1-based indexing and we (of course) use 0-based.
   7: // Graves et al. call the probabilities y, we use log_probs (also calling them inputs)
   8: // A few optimizations (similar to those here, but also some I didn't take) are described in
   9: // 2. Minmin Sun: http://on-demand.gputechconf.com/gtc/2016/presentation/s6383-minmin-sun-speech-recognition.pdf
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 10-23
```cpp
  10: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
  11: #include <ATen/TensorUtils.h>
  12: #include <c10/util/Exception.h>
  13: #include <c10/macros/Macros.h>
  14: #include <ATen/core/Tensor.h>
  15: #include <ATen/Dispatch.h>
  16: #include <ATen/TensorOperators.h>
  17: #include <ATen/cuda/Atomic.cuh>
  18: #include <ATen/cuda/CUDAContext.h>
  19: 
  20: #ifndef AT_PER_OPERATOR_HEADERS
  21: #include <ATen/Functions.h>
  22: #include <ATen/NativeFunctions.h>
  23: #else
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/TensorUtils.h>`, `<c10/util/Exception.h>`, `<c10/macros/Macros.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/TensorUtils.h>`, `<c10/util/Exception.h>`, `<c10/macros/Macros.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 24-37
```cpp
  24: #include <ATen/ops/_ctc_loss_backward_native.h>
  25: #include <ATen/ops/_ctc_loss_native.h>
  26: #include <ATen/ops/empty.h>
  27: #include <ATen/ops/exp.h>
  28: #include <ATen/ops/full_like.h>
  29: #include <ATen/ops/imag.h>
  30: #include <ATen/ops/logsumexp.h>
  31: #include <ATen/ops/tensor.h>
  32: #include <ATen/ops/where.h>
  33: #include <ATen/ops/zeros.h>
  34: #endif
  35: 
  36: #include <type_traits>
  37: #include <numeric>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/ops/_ctc_loss_backward_native.h>`, `<ATen/ops/_ctc_loss_native.h>`, `<ATen/ops/empty.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/ops/_ctc_loss_backward_native.h>`, `<ATen/ops/_ctc_loss_native.h>`, `<ATen/ops/empty.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 39-60
```cpp
  39: namespace at::native {
  40: 
  41: namespace {
  42: 
  43: // this ad-hoc converts from targets (l in [1]) to augmented targets (l' in [1])
  44: // so if l is l_0 l_1 ... l_(tl-1) then this looks up idx in
  45: // l' = BLANK l_0 BLANK l_1 BLANK ... BLANK l_(tl-1) BLANK
  46: // - note that no bound-checking is done
  47: // - it is important to only call it with idx == 0 if the target length is 0
  48: // - __restrict__ impact to be measured, see
  49: //   https://devblogs.nvidia.com/cuda-pro-tip-optimize-pointer-aliasing/
  50: template <typename target_t>
  51: __device__ static inline int64_t get_target_prime(
  52:     const target_t* __restrict__ target,
  53:     int64_t offset,
  54:     int64_t stride,
  55:     int64_t idx,
  56:     int64_t BLANK) {
  57:   if (idx % 2 == 0) {
  58:     return BLANK;
  59:   } else {
  60:     return target[offset + stride * (idx / 2)];
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `targets`.
- CN: 该代码块定义或继续实现 `targets`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 61-62
```cpp
  61:   }
  62: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 64-72
```cpp
  64: // this kernel is a relatively straightforward implementation of the alpha calculation in the forward backward algorithm (section 4.1).
  65: // A (minor) twist is that we are using log-calculations to enhance numerical stability (log_probs and log_alpha).
  66: // In total it would be more efficient to compute the beta in the same kernel (e.g. cudnn does this). While the beta are not
  67: // needed for the loss itself (just the grad), we can return log_alpha+log_beta (so same space as currently) and the overhead
  68: // is small and the use-case for loss without grad is relatively limited.
  69: // We parallelize by batch and target sequence. Empirically, it is faster to loop over the input (log probs) sequence  and do
  70: // target in parallel, even if it means more frequent __syncthreads.
  71: // In contrast to the cuDNN implementation, we allow large target lengths. For this we need that all previous `s` have been
  72: // computed when we start a new block_s. This is why we have our own for loop here.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 73-94
```cpp
  73: template<typename scalar_t, typename target_t>
  74: __global__ void
  75: #if defined (USE_ROCM)
  76: C10_LAUNCH_BOUNDS_2((std::is_same_v<scalar_t, float> ? 1024 : 896), 1)
  77: #endif
  78: ctc_loss_log_alpha_gpu_kernel(scalar_t* __restrict__ log_alpha_data,
  79:                                     const scalar_t*log_probs_data, const int64_t* __restrict__ input_lengths, int64_t max_input_length,
  80:                                     const target_t* __restrict__ targets_data, const int64_t* __restrict__ target_lengths, int64_t max_target_length,
  81:                                     scalar_t* __restrict__ neg_log_likelihood_data,
  82:                                     int64_t lp_input_stride, int64_t lp_batch_stride, int64_t lp_char_stride,
  83:                                     int64_t la_batch_stride, int64_t la_input_stride, int64_t la_target_stride,
  84:                                     const int64_t* __restrict__ tg_batch_offsets, int64_t tg_target_stride,
  85:                                     int64_t batch_size, int64_t BLANK) {
  86: 
  87:   constexpr scalar_t neginf = -INFINITY;
  88: 
  89:   // bookkeeping
  90:   int64_t b = threadIdx.y + blockIdx.y * blockDim.y;
  91:   int64_t input_length = input_lengths[b];
  92:   int64_t target_length = target_lengths[b];
  93:   int64_t lp_batch_offset = b*lp_batch_stride;
  94:   int64_t la_batch_offset = b*la_batch_stride;
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines GPU kernel entry point(s) `defined`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `defined`，它们会直接在 CUDA 线程上执行。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 95-95
```cpp
  95:   int64_t tg_batch_offset = tg_batch_offsets[b];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 97-98
```cpp
  97:   if (b >= batch_size)
  98:     return;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 100-106
```cpp
 100:   if (input_length == 0) {
 101:     if (threadIdx.x == 0) {
 102:       scalar_t log_likelihood = target_length == 0 ? 0 : neginf;
 103:       neg_log_likelihood_data[b] = -log_likelihood;
 104:     }
 105:     return;
 106:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 108-108
```cpp
 108:   // first row (t=0), the three equations for alpha_1 above eq (6)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 109-130
```cpp
 109:   for (int64_t block_s = 0; block_s < 2*max_target_length+1; block_s += blockDim.x) {
 110:     int64_t s = threadIdx.x + block_s;
 111:     scalar_t la;
 112:     switch (s) {
 113:     case 0:
 114:       la = log_probs_data[lp_batch_offset + lp_char_stride * BLANK];
 115:       break;
 116:     case 1:
 117:       la = target_length == 0 ? neginf
 118:                               : log_probs_data
 119:                                     [lp_batch_offset +
 120:                                      lp_char_stride *
 121:                                          get_target_prime(
 122:                                              targets_data,
 123:                                              tg_batch_offset,
 124:                                              tg_target_stride,
 125:                                              1,
 126:                                              BLANK)];
 127:       break;
 128:     default:
 129:       la = neginf;
 130:     }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 131-133
```cpp
 131:     if (s < 2*max_target_length+1)
 132:       log_alpha_data[la_batch_offset + /* la_input_stride * 0 */ + la_target_stride * s] = la;
 133:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 135-156
```cpp
 135:   for (int64_t block_s = 0; block_s < 2*max_target_length+1; block_s += blockDim.x) {
 136:     int64_t s = threadIdx.x + block_s;
 137: 
 138:     // These two only depend on s, so we can cache them.
 139:     int64_t current_char;       // l_s in eq (6)
 140:     bool have_three;            // flag which of the two cases in eq (6) we have
 141:     if (s < 2 * target_length + 1 && target_length > 0) {
 142:       current_char = get_target_prime(
 143:           targets_data,
 144:           tg_batch_offset,
 145:           tg_target_stride,
 146:           s,
 147:           BLANK);
 148:       have_three =
 149:           ((s > 1) &&
 150:            (get_target_prime(
 151:                 targets_data,
 152:                 tg_batch_offset,
 153:                 tg_target_stride,
 154:                 s - 2,
 155:                 BLANK) != current_char));
 156:     } else {
```
- EN: This block defines or continues the implementation of `eq`.
- CN: 该代码块定义或继续实现 `eq`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 157-178
```cpp
 157:       current_char = BLANK;
 158:       have_three = false;
 159:     }
 160:     for (int64_t t=1; t < max_input_length; t++) {
 161:       __syncthreads(); // on cuda 9 we might use partial synchronization of only the threads within the same batch
 162:       if ((t < input_length) && (s < 2 * target_length + 1)) {
 163:         // only for valid t, s. This is equation (6) and (7), la1, la2, la3 are the three summands,
 164:         // lamax is the maximum for the logsumexp trick.
 165:         scalar_t la1 = log_alpha_data[la_batch_offset + la_input_stride * (t-1) + la_target_stride * s];
 166:         scalar_t lamax = la1;
 167:         scalar_t la2, la3;
 168:         if (s > 0) {
 169:           la2 = log_alpha_data[la_batch_offset + la_input_stride * (t-1) + la_target_stride * (s-1)];
 170:           if (la2 > lamax)
 171:             lamax = la2;
 172:         } else {
 173:           la2 = neginf;
 174:         }
 175:         if (have_three) {
 176:           la3 = log_alpha_data[la_batch_offset + la_input_stride * (t-1) + la_target_stride * (s-2)];
 177:           if (la3 > lamax)
 178:             lamax = la3;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 179-183
```cpp
 179:         } else {
 180:           la3 = neginf;
 181:         }
 182:         if (lamax == neginf) // when all are neginf. (then the whole thing is neginf, but we can pretend)
 183:           lamax = 0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 185-194
```cpp
 185:         log_alpha_data[la_batch_offset + la_input_stride * t + la_target_stride * s] = std::log(std::exp(la1-lamax)+std::exp(la2-lamax)+std::exp(la3-lamax))+lamax
 186:           + log_probs_data[lp_batch_offset + t * lp_input_stride + lp_char_stride * current_char];
 187:       } else {
 188:         // otherwise we just set to neginf
 189:         if (s < 2*max_target_length+1)
 190:           log_alpha_data[la_batch_offset + la_input_stride * t + la_target_stride * s] = neginf;
 191:       }
 192:     }
 193:   }
 194:   __syncthreads(); // on cuda 9 we might use partial synchronization of only the threads within the same batch
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 196-196
```cpp
 196:   // compute the loss (eq (8))
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 197-209
```cpp
 197:   if (threadIdx.x == 0) {
 198:     scalar_t l1 = log_alpha_data[la_batch_offset + la_input_stride * (input_length-1) + la_target_stride * (target_length*2)];
 199:     scalar_t l2 = target_length > 0
 200:         ? log_alpha_data
 201:               [la_batch_offset + la_input_stride * (input_length - 1) +
 202:                la_target_stride * (target_length * 2 - 1)]
 203:         : neginf;
 204:     scalar_t m = ((l1 > l2) ? l1 : l2);
 205:     m = ((m == neginf) ? 0 : m);
 206:     scalar_t log_likelihood = std::log(std::exp(l1-m)+std::exp(l2-m))+m;
 207:     neg_log_likelihood_data[b] = -log_likelihood;
 208:   }
 209: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 211-219
```cpp
 211: // The forward computation. Lot's of admin and a call to the alpha kernel.
 212: // Note: we do not check that the labels are in the valid range. As we use
 213: // them for indexing in the kernels, you'll see memory errors when you
 214: // pass corrupt labels.
 215: // We support both a 2-dimensional tensor as targets (one set of targets in each row) and
 216: // a 1-dimensional tensor where all targets are concatenated (and we use target_lengths
 217: // to figure out where they begin).
 218: // We return log_alpha (currently, might change to (log_alpha+log_beta) to be passed to the
 219: // backward. The dispatch function will only return the loss.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 220-241
```cpp
 220: template<typename scalar_t, ScalarType target_scalar_type>
 221: std::tuple<Tensor, Tensor> ctc_loss_gpu_template(const Tensor& log_probs, const Tensor& targets, IntArrayRef input_lengths, IntArrayRef target_lengths, int64_t BLANK) {
 222:   TORCH_CHECK(log_probs.numel() > 0, "log_probs tensor must not be empty");
 223:   // log_probs: input_len x batch_size x num_labels
 224:   // targets [int64]: batch_size x target_length OR sum(target_lengths)
 225:   CheckedFrom c = "ctc_loss_gpu";
 226:   using target_t = typename std::conditional_t<target_scalar_type == kInt, int, int64_t>;
 227:   auto log_probs_arg = TensorArg(log_probs, "log_probs", 1);
 228:   auto targets_arg = TensorArg(targets, "targets", 2);
 229:   checkAllSameGPU(c, {log_probs_arg, targets_arg});
 230: 
 231:   checkScalarType(c, targets_arg, target_scalar_type);
 232:   checkDim(c, log_probs_arg, 3);
 233:   checkDimRange(c, targets_arg, 1, 3);
 234: 
 235:   int64_t batch_size = log_probs.size(1);
 236:   int64_t num_labels = log_probs.size(2);
 237:   TORCH_CHECK((0 <= BLANK) && (BLANK < num_labels), "blank must be in label range");
 238:   TORCH_CHECK(input_lengths.size() == static_cast<size_t>(batch_size), "input_lengths must be of size batch_size");
 239:   TORCH_CHECK(target_lengths.size() == static_cast<size_t>(batch_size), "target_lengths must be of size batch_size");
 240: 
 241:   int64_t tg_target_stride;
```
- EN: This block defines or continues the implementation of `ctc_loss_gpu_template`.
- CN: 该代码块定义或继续实现 `ctc_loss_gpu_template`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 243-259
```cpp
 243:   int64_t max_target_length = 0;
 244:   auto tg_batch_offsets = at::empty({batch_size}, at::device(at::kCPU).dtype(at::kLong));
 245:   auto tg_batch_offsets_data = tg_batch_offsets.template mutable_data_ptr<int64_t>();
 246:   if (targets.dim() == 1) { // concatenated targets
 247:     int64_t pos = 0;
 248:     for (int64_t i = 0; i < batch_size; i++) {
 249:       TORCH_CHECK(target_lengths[i] >= 0,
 250:                   "Expected target_lengths to have value at least ", 0, ", but got value ", target_lengths[i],
 251:                   " (while checking arguments for ", c, ")");
 252:       tg_batch_offsets_data[i] = pos;
 253:       pos += target_lengths[i];
 254:       if (max_target_length < target_lengths[i])
 255:         max_target_length = target_lengths[i];
 256:     }
 257:     tg_target_stride = targets.stride(0);
 258:     checkSize(c, targets_arg, 0, pos);
 259:   }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 260-276
```cpp
 260:   else { // batch x max_target_length
 261:     // dim is 2
 262:     int64_t tg_batch_stride = targets.stride(0);
 263:     for (int64_t i = 0; i < batch_size; i++) {
 264:       TORCH_CHECK(target_lengths[i] >= 0,
 265:                   "Expected target_lengths to have value at least ", 0, ", but got value ", target_lengths[i],
 266:                   " (while checking arguments for ", c, ")");
 267:       tg_batch_offsets_data[i] = i * tg_batch_stride;
 268:       if (max_target_length < target_lengths[i])
 269:         max_target_length = target_lengths[i];
 270:     }
 271:     tg_target_stride = targets.stride(1);
 272:     checkSize(c, targets_arg, 0, batch_size);
 273:     TORCH_CHECK(targets.size(1) >= max_target_length,
 274:              "Expected tensor to have size at least ", max_target_length, " at dimension 1, but got size ", targets.size(1), " for ", targets_arg,
 275:              " (while checking arguments for ", c, ")");
 276:   }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 277-285
```cpp
 277:   int64_t max_input_length = log_probs.size(0);
 278:   for (int64_t b = 0; b < batch_size; b++) {
 279:     TORCH_CHECK(input_lengths[b] >= 0,
 280:              "Expected input_lengths to have value at least ", 0, ", but got value ", input_lengths[b],
 281:              " (while checking arguments for ", c, ")");
 282:     TORCH_CHECK(input_lengths[b] <= max_input_length,
 283:              "Expected input_lengths to have value at most ", max_input_length, ", but got value ", input_lengths[b],
 284:              " (while checking arguments for ", c, ")");
 285:   }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 287-289
```cpp
 287:   auto target_lengths_t = at::tensor(target_lengths, targets.options().dtype(kLong));
 288:   auto input_lengths_t = at::tensor(input_lengths, targets.options().dtype(kLong));
 289:   tg_batch_offsets = tg_batch_offsets.cuda();
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 291-292
```cpp
 291:   Tensor log_alpha = at::empty({batch_size, log_probs.size(0), 2*max_target_length+1}, log_probs.options());
 292:   Tensor neg_log_likelihood = at::empty({batch_size}, log_probs.options());
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 294-294
```cpp
 294:   // Very likely, we could be more clever here, e.g. learning (or generalizing and reusing) from SoftMax.cu...
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 295-308
```cpp
 295:   constexpr int max_threads_ = std::is_same_v<scalar_t, float> ? 1024 : 768; // we need 72 or so 32 bit registers for double
 296:   int max_threads = max_threads_;
 297:   // Launch bounds for sm120, sm121
 298:   if (at::cuda::getCurrentDeviceProperties()->major >= 12) {
 299:     max_threads = 640; // magic number, tuned to be below resource limit
 300:   }
 301:   int threads_target = max_threads;
 302:   while (threads_target / 2 >= 2*max_target_length+1) {
 303:     threads_target /= 2;
 304:   }
 305:   int threads_batch = std::min(max_threads / threads_target, (int) batch_size);
 306:   dim3 block(threads_target, threads_batch);
 307:   dim3 grid(1, (batch_size+threads_batch-1)/threads_batch);
 308:   cudaStream_t stream = at::cuda::getCurrentCUDAStream();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 310-321
```cpp
 310:   ctc_loss_log_alpha_gpu_kernel<scalar_t, target_t><<<grid, block, 0, stream>>>(
 311:                       log_alpha.mutable_data_ptr<scalar_t>(),
 312:                       log_probs.const_data_ptr<scalar_t>(), input_lengths_t.template const_data_ptr<int64_t>(), log_probs.size(0),
 313:                       targets.const_data_ptr<target_t>(), target_lengths_t.template const_data_ptr<int64_t>(), max_target_length,
 314:                       neg_log_likelihood.mutable_data_ptr<scalar_t>(),
 315:                       log_probs.stride(0), log_probs.stride(1), log_probs.stride(2),
 316:                       log_alpha.stride(0), log_alpha.stride(1), log_alpha.stride(2),
 317:                       tg_batch_offsets.template const_data_ptr<int64_t>(), tg_target_stride,
 318:                       batch_size, BLANK);
 319:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 320:   return std::make_tuple(neg_log_likelihood, log_alpha);
 321: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 323-324
```cpp
 323: // The second (backward) half of the forward backward algorithm, (10) and (11). This is parallel to the
 324: // alpha kernel above. (As mentioned above, it might make sense do the calculation in the alpha kernel.)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 325-346
```cpp
 325: template<typename scalar_t, typename target_t>
 326: __global__ void
 327: C10_LAUNCH_BOUNDS_2((std::is_same_v<scalar_t, float> ? 1024 : 896), 1)
 328: ctc_loss_backward_log_beta_gpu_kernel(scalar_t* __restrict__ log_beta_data,
 329:                                       const scalar_t*log_probs_data, const int64_t* __restrict__ input_lengths, int64_t max_input_length,
 330:                                       const target_t* __restrict__ targets_data, const int64_t* __restrict__ target_lengths, int64_t max_target_length,
 331:                                       int64_t lp_input_stride, int64_t lp_batch_stride, int64_t lp_char_stride,
 332:                                       int64_t lb_batch_stride, int64_t lb_input_stride, int64_t lb_target_stride,
 333:                                       const int64_t* __restrict__ tg_batch_offsets, int64_t tg_target_stride,
 334:                                       int64_t batch_size, int64_t BLANK) {
 335:   constexpr scalar_t neginf = -INFINITY;
 336: 
 337:   int64_t b = threadIdx.y + blockIdx.y * blockDim.y;
 338: 
 339:   if (b >= batch_size)
 340:     return;
 341: 
 342:   int64_t input_length = input_lengths[b];
 343:   int64_t target_length = target_lengths[b];
 344:   int64_t lp_batch_offset = b*lp_batch_stride;
 345:   int64_t lb_batch_offset = b*lb_batch_stride;
 346:   int64_t tg_batch_offset = tg_batch_offsets[b];
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 348-349
```cpp
 348:   if (input_length == 0)
 349:     return;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 351-351
```cpp
 351:   // "first" row, the beta initialization before eq (10) (t=target_length - differs per batch)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 352-371
```cpp
 352:   for (int64_t block_s = 2*max_target_length - (2*max_target_length % blockDim.x); block_s >= 0; block_s -= blockDim.x) {
 353:     int64_t s = threadIdx.x + block_s;
 354:     scalar_t lb;
 355:     if (s == 2*target_length) {
 356:       lb = log_probs_data[lp_batch_offset + (input_length-1) * lp_input_stride + lp_char_stride * BLANK];
 357:     } else if (s == 2 * target_length - 1) { // false for target_length == 0
 358:       int64_t current_target_prime = get_target_prime(
 359:           targets_data,
 360:           tg_batch_offset,
 361:           tg_target_stride,
 362:           s,
 363:           BLANK);
 364:       lb = log_probs_data[lp_batch_offset + (input_length-1) * lp_input_stride + lp_char_stride * current_target_prime];
 365:     } else {
 366:       lb = neginf;
 367:     }
 368:     if (s < 2*max_target_length+1) {
 369:       log_beta_data[lb_batch_offset + (input_length-1) * lb_input_stride + lb_target_stride * s] = lb;
 370:     }
 371:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 373-373
```cpp
 373:   // go backward in s
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 374-395
```cpp
 374:   for (int64_t block_s = 2*max_target_length - (2*max_target_length % blockDim.x); block_s >= 0; block_s -= blockDim.x) {
 375:     int64_t s = threadIdx.x + block_s;
 376:     int64_t current_target_prime;
 377:     bool have_three;
 378:     if (s < 2 * target_length + 1 && target_length > 0) {
 379:       current_target_prime = get_target_prime(
 380:           targets_data,
 381:           tg_batch_offset,
 382:           tg_target_stride,
 383:           s,
 384:           BLANK);
 385:       have_three =
 386:           ((s < 2 * target_length - 1) &&
 387:            (get_target_prime(
 388:                 targets_data,
 389:                 tg_batch_offset,
 390:                 tg_target_stride,
 391:                 s + 2,
 392:                 BLANK) != current_target_prime));
 393:     } else {
 394:       current_target_prime = BLANK;
 395:       have_three = false;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 396-417
```cpp
 396:     }
 397:     // now go backward in t. Note that we need to skip the last timestep that we did above.
 398:     for (int64_t t=max_input_length-2; t>=0; t--) {
 399:       __syncthreads(); // on cuda 9 we might use partial synchronization of only the threads within the same batch item
 400:       if ((t < input_length - 1) && (s < 2 * target_length + 1)) {
 401:         scalar_t lb1 = log_beta_data[lb_batch_offset + lb_input_stride * (t+1) + lb_target_stride * s];
 402:         scalar_t lbmax = lb1;
 403:         scalar_t lb2, lb3;
 404: 
 405:         if (s < 2*target_length) {
 406:           lb2 = log_beta_data[lb_batch_offset + lb_input_stride * (t+1) + lb_target_stride * (s+1)];
 407:           if (lb2 > lbmax)
 408:             lbmax = lb2;
 409:         } else {
 410:           lb2 = neginf;
 411:         }
 412:         if (have_three) {
 413:           lb3 = log_beta_data[lb_batch_offset + lb_input_stride * (t+1) + lb_target_stride * (s+2)];
 414:           if (lb3 > lbmax)
 415:             lbmax = lb3;
 416:         } else {
 417:           lb3 = neginf;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 418-420
```cpp
 418:         }
 419:         if (lbmax == neginf)
 420:           lbmax = 0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 422-423
```cpp
 422:         scalar_t lb = std::log(std::exp(lb1-lbmax)+std::exp(lb2-lbmax)+std::exp(lb3-lbmax))+lbmax
 423:           + log_probs_data[lp_batch_offset + t * lp_input_stride + lp_char_stride * current_target_prime];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 425-436
```cpp
 425:         log_beta_data[lb_batch_offset + lb_input_stride * t + lb_target_stride * s] = lb;
 426:       } else if (
 427:           (s < 2 * max_target_length + 1) &&
 428:           (((target_length == 0) && (s > 0)) || (s >= 2 * target_length + 1) ||
 429:            (t >= input_length))) {
 430:         log_beta_data
 431:             [lb_batch_offset + lb_input_stride * t + lb_target_stride * s] =
 432:                 neginf;
 433:       }
 434:     }
 435:   }
 436: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 438-452
```cpp
 438: // This implements the subtrahend of equation (16) for all *nonblank* characters.
 439: // It assumes you have probs in gradient_data when called
 440: // and it modifies gradient_data to be, the gradient.
 441: // In order to facilitate this inplace update, We don't actually do this in logspace.
 442: // (The other variant implemented uses log_space and the differences seem to be
 443: //  not so problematic at least with unit normal distributed test activations.)
 444: // Internally this uses atomicAdd because different threads may write to the same
 445: // gradient position.
 446: // This is parallelised over b and s again.
 447: // Note that for us, the Z of eqn (16) is actually constant for all t and it is the
 448: // likelihood - this is why we use the negative log likelihood below.
 449: // We also multiply by the input gradient to keep with standard autograd style.
 450: // I took this trick from [2], for moderate alphabet sizes a log-space
 451: // calculation (with an atomic log add) is similarly in performance, but for large
 452: // alphabets the inplace nature is a considerable advantage.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 453-466
```cpp
 453: template<typename scalar_t, typename target_t>
 454: __global__ void
 455: #if defined (USE_ROCM)
 456: C10_LAUNCH_BOUNDS_2((std::is_same_v<scalar_t, float> ? 1024 : 896), 1)
 457: #endif
 458: ctc_loss_backward_collect_nonblank_gpu_kernel(scalar_t* __restrict__ gradient_data,
 459:                                                      const scalar_t* __restrict__ grad_out_data, int64_t grad_out_batch_stride,
 460:                                                      const scalar_t* __restrict__ log_alpha_data, const scalar_t* __restrict__ log_beta_data,
 461:                                                      const scalar_t*log_probs_data, const int64_t* __restrict__ input_lengths,
 462:                                                      const target_t* __restrict__ targets_data, const int64_t* __restrict__ target_lengths,
 463:                                                      const scalar_t* __restrict__ neg_log_likelihood_data,
 464:                                                      int64_t gr_input_stride, int64_t gr_batch_stride, int64_t gr_char_stride,
 465:                                                      int64_t lp_input_stride, int64_t lp_batch_stride, int64_t lp_char_stride,
 466:                                                      int64_t la_batch_stride, int64_t la_input_stride, int64_t la_target_stride,
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 467-488
```cpp
 467:                                                      int64_t lb_batch_stride, int64_t lb_input_stride, int64_t lb_target_stride,
 468:                                                      const int64_t* __restrict__ tg_batch_offsets, int64_t tg_target_stride,
 469:                                               int64_t batch_size, bool zero_infinity) {
 470:   int64_t b = threadIdx.y + blockIdx.y * blockDim.y;
 471:   int64_t s = threadIdx.x + ((int64_t) blockIdx.x) * blockDim.x; // note, this directly indexes into targets, not targets prime!
 472: 
 473:   if (b >= batch_size)
 474:     return;
 475: 
 476:   int64_t input_length = input_lengths[b];
 477:   int64_t target_length = target_lengths[b];
 478:   int64_t gr_batch_offset = b*gr_batch_stride;
 479:   int64_t lp_batch_offset = b*lp_batch_stride;
 480:   int64_t la_batch_offset = b*la_batch_stride;
 481:   int64_t lb_batch_offset = b*lb_batch_stride;
 482:   int64_t tg_batch_offset = tg_batch_offsets[b];
 483: 
 484:   if (s >= target_length)
 485:     return;
 486: 
 487:   int64_t target = targets_data[tg_batch_offset + s * tg_target_stride];
 488:   scalar_t nll = neg_log_likelihood_data[b];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 489-489
```cpp
 489:   scalar_t gr =  grad_out_data[b * grad_out_batch_stride];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 491-492
```cpp
 491:   if (zero_infinity && nll == INFINITY)
 492:     return;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 494-501
```cpp
 494:   for (int64_t t = 0; t < input_length; t++) {
 495:     scalar_t lp = log_probs_data[lp_batch_offset + t * lp_input_stride + lp_char_stride * target];
 496:     gpuAtomicAddNoReturn(&gradient_data[gr_batch_offset + t * gr_input_stride + gr_char_stride * target],
 497:               -std::exp(log_alpha_data[la_batch_offset + la_input_stride * t + la_target_stride * (s*2+1)]
 498:                         + log_beta_data[lb_batch_offset + lb_input_stride * t + lb_target_stride * (s*2+1)]
 499:                         + nll - lp) * gr);
 500:   }
 501: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 503-504
```cpp
 503: // This is the naive implementation of equation (16). It is parallelised in batch and input timestep.
 504: // It appears to be faster than the above method for small batch sizes.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 505-518
```cpp
 505: template<typename scalar_t, typename target_t>
 506: __global__ void
 507: #if defined (USE_ROCM)
 508: C10_LAUNCH_BOUNDS_2((std::is_same_v<scalar_t, float> ? 1024 : 896), 1)
 509: #endif
 510: ctc_loss_backward_collect_gpu_kernel(scalar_t* __restrict__ gradient_data,
 511:                                                      const scalar_t* __restrict__ grad_out_data, int64_t grad_out_batch_stride,
 512:                                                      const scalar_t* __restrict__ log_alpha_data, const scalar_t* __restrict__ log_beta_data,
 513:                                                      const scalar_t*log_probs_data, const int64_t* __restrict__ input_lengths, int64_t max_input_length,
 514:                                                      const target_t* __restrict__ targets_data, const int64_t* __restrict__ target_lengths, int64_t max_target_length,
 515:                                                      const scalar_t* __restrict__ neg_log_likelihood_data,
 516:                                                      int64_t gr_input_stride, int64_t gr_batch_stride, int64_t gr_char_stride,
 517:                                                      int64_t lp_input_stride, int64_t lp_batch_stride, int64_t lp_char_stride,
 518:                                                      int64_t la_batch_stride, int64_t la_input_stride, int64_t la_target_stride,
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 519-540
```cpp
 519:                                                      int64_t lb_batch_stride, int64_t lb_input_stride, int64_t lb_target_stride,
 520:                                                      const int64_t* __restrict__ tg_batch_offsets, int64_t tg_target_stride,
 521:                                      int64_t batch_size, int64_t num_labels, int64_t BLANK, bool zero_infinity) {
 522: 
 523:   constexpr scalar_t neginf = -INFINITY;
 524:   int64_t b = threadIdx.y + blockIdx.y * blockDim.y;
 525:   int64_t t = threadIdx.x + ((int64_t) blockIdx.x) * blockDim.x;
 526: 
 527:   if ((t >= max_input_length) || (b >= batch_size))
 528:     return;
 529: 
 530:   int64_t input_length = input_lengths[b];
 531:   int64_t target_length = target_lengths[b];
 532:   int64_t gr_batch_offset = b*gr_batch_stride;
 533:   int64_t lp_batch_offset = b*lp_batch_stride;
 534:   int64_t la_batch_offset = b*la_batch_stride;
 535:   int64_t lb_batch_offset = b*lb_batch_stride;
 536:   int64_t tg_batch_offset = tg_batch_offsets[b];
 537: 
 538:   // collected[b, t, target'[s]] "log+=" log_alpha[t, s]+log_beta[t, s]
 539:   for (int s = 0; s < 2*max_target_length+1; s++) {
 540:     if (s < 2 * target_length + 1) { // if target_length == 0, s == 0
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 541-555
```cpp
 541:       int64_t current_target_prime = get_target_prime(
 542:           targets_data,
 543:           tg_batch_offset,
 544:           tg_target_stride,
 545:           s,
 546:           BLANK);
 547:       scalar_t log_alpha_beta = (log_alpha_data[la_batch_offset + la_input_stride * t + la_target_stride * s]
 548:                                  + log_beta_data[lb_batch_offset + lb_input_stride * t + lb_target_stride * s]);
 549:       scalar_t& lcab = gradient_data[gr_batch_offset + t * gr_input_stride + gr_char_stride * current_target_prime];
 550:       if (lcab == neginf) {
 551:         lcab = log_alpha_beta;
 552:       } else {
 553:         scalar_t max = ((lcab > log_alpha_beta) ? lcab : log_alpha_beta);
 554:         lcab = std::log(std::exp(lcab-max)+std::exp(log_alpha_beta-max))+max;
 555:       }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 556-557
```cpp
 556:     }
 557:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 559-560
```cpp
 559:   scalar_t nll = neg_log_likelihood_data[b];
 560:   scalar_t gr =  grad_out_data[b * grad_out_batch_stride];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 562-572
```cpp
 562:   for (int64_t c = 0; c < num_labels; c++) {
 563:     scalar_t& res = gradient_data[gr_batch_offset + t * gr_input_stride + gr_char_stride * c];
 564:     if (t < input_length && (! zero_infinity || nll != INFINITY)) {
 565:       scalar_t lp = log_probs_data[lp_batch_offset + t * lp_input_stride + lp_char_stride * c];
 566:       res = (std::exp(lp)-std::exp(res + nll - lp)) * gr;
 567:     }
 568:     else {
 569:       res = 0.;
 570:     }
 571:   }
 572: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 574-576
```cpp
 574: // This is to zero gradients which corresponding to the out-of-sequence position
 575: // Those gradients should not be used in any model update since the input
 576: // elements are padded
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 577-598
```cpp
 577: template<typename scalar_t>
 578: __global__ void
 579: #if defined (USE_ROCM)
 580: C10_LAUNCH_BOUNDS_2((std::is_same_v<scalar_t, float> ? 1024 : 896), 1)
 581: #endif
 582: ctc_loss_zero_padded_gradients(
 583:     scalar_t* __restrict__ gradient_data,   /* (T, B, D) layout */
 584:     const int64_t* __restrict__ input_lengths, /* (B, ) layout */
 585:     int64_t gr_timestep_stride,
 586:     int64_t gr_batch_stride,
 587:     int64_t gr_label_stride,
 588:     int64_t max_input_length, /* T */
 589:     int64_t batch_size, /* B */
 590:     int64_t num_labels  /* D */ ) {
 591:       int64_t b = threadIdx.y + blockIdx.y * blockDim.y;
 592:       int64_t t = threadIdx.x + ((int64_t) blockIdx.x) * blockDim.x;
 593: 
 594:       if (b >= batch_size || t >= max_input_length) {
 595:         return;
 596:       }
 597: 
 598:       scalar_t input_length = input_lengths[b];
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines GPU kernel entry point(s) `defined`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `defined`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 599-605
```cpp
 599:       if (t >= input_length) {
 600:         for (int l = 0; l < num_labels; l++)
 601:           gradient_data[
 602:             t * gr_timestep_stride + b * gr_batch_stride + l * gr_label_stride]
 603:           = 0.0f;
 604:       }
 605:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 608-609
```cpp
 608: // The backward. It essentially computes eq 16 by using the above kernels.
 609: // We don't do a lot of checking as we envision this to be called only when backpropagating through a (well-checked) forward.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 610-631
```cpp
 610: template<typename scalar_t, ScalarType target_scalar_type>
 611: Tensor ctc_loss_backward_gpu_template(const Tensor& grad_out, const Tensor& log_probs, const Tensor& targets, IntArrayRef input_lengths, IntArrayRef target_lengths,
 612:                                       const Tensor& neg_log_likelihood, const Tensor& log_alpha, int64_t BLANK, bool zero_infinity) {
 613:   constexpr scalar_t neginf = -INFINITY;
 614:   using target_t = typename std::conditional_t<target_scalar_type == kInt, int, int64_t>;
 615:   int64_t batch_size = log_probs.size(1);
 616:   int64_t num_labels = log_probs.size(2);
 617:   int64_t tg_target_stride;
 618: 
 619:   int64_t max_target_length;
 620:   auto tg_batch_offsets = at::empty({batch_size}, TensorOptions(at::CPU(kLong)));
 621:   auto tg_batch_offsets_data = tg_batch_offsets.template mutable_data_ptr<int64_t>();
 622:   if (targets.dim() == 1) { // concatenated targets
 623:     int64_t pos = 0;
 624:     max_target_length = 0;
 625:     for (int64_t i = 0; i < batch_size; i++) {
 626:       tg_batch_offsets_data[i] = pos;
 627:       pos += target_lengths[i];
 628:       if (max_target_length < target_lengths[i])
 629:         max_target_length = target_lengths[i];
 630:     }
 631:     tg_target_stride = targets.stride(0);
```
- EN: This block defines or continues the implementation of `ctc_loss_backward_gpu_template`.
- CN: 该代码块定义或继续实现 `ctc_loss_backward_gpu_template`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 632-644
```cpp
 632:   }
 633:   else { // batch x max_target_length
 634:     // dim is 2
 635:     int64_t tg_batch_stride = targets.stride(0);
 636:     for (int64_t i = 0; i < batch_size; i++) {
 637:       tg_batch_offsets_data[i] = i * tg_batch_stride;
 638:     }
 639:     tg_target_stride = targets.stride(1);
 640:     max_target_length = log_alpha.size(2)/2; // targets.size(1) might be larger
 641:   }
 642:   auto target_lengths_t = at::tensor(target_lengths, targets.options().dtype(kLong));
 643:   auto input_lengths_t = at::tensor(input_lengths, targets.options().dtype(kLong));
 644:   tg_batch_offsets = tg_batch_offsets.cuda();
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 646-647
```cpp
 646:   Tensor log_beta = at::empty_like(log_alpha, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
 647:   log_beta.fill_(neginf);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 649-649
```cpp
 649:   Tensor grad = at::full_like(log_probs, neginf, LEGACY_CONTIGUOUS_MEMORY_FORMAT); // initialization for log(sum (alpha beta))
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 651-651
```cpp
 651:   // As above, there may be better configurations to use.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 652-662
```cpp
 652:   constexpr int max_threads_ = std::is_same_v<scalar_t, float> ? 1024 : 896; // we need 72 or so 32 bit registers for double
 653:   int max_threads = max_threads_;
 654:   // Blackwell launch bounds
 655:   if (at::cuda::getCurrentDeviceProperties()->major >= 10) {
 656:     max_threads = 512;
 657:   }
 658:   int threads_target = max_threads;
 659:   while (threads_target / 2 >= 2*max_target_length+1) {
 660:     threads_target /= 2;
 661:   }
 662:   int threads_batch = std::min(max_threads / threads_target, (int) batch_size);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 664-664
```cpp
 664:   cudaStream_t stream = at::cuda::getCurrentCUDAStream();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 666-678
```cpp
 666:   {
 667:     dim3 block(threads_target, threads_batch);
 668:     dim3 grid(1, (batch_size+threads_batch-1)/threads_batch);
 669:     ctc_loss_backward_log_beta_gpu_kernel<scalar_t, target_t><<<grid, block, 0, stream>>>
 670:       (log_beta.mutable_data_ptr<scalar_t>(),
 671:        log_probs.const_data_ptr<scalar_t>(), input_lengths_t.template const_data_ptr<int64_t>(), log_probs.size(0),
 672:        targets.const_data_ptr<target_t>(), target_lengths_t.template const_data_ptr<int64_t>(), max_target_length,
 673:        log_probs.stride(0), log_probs.stride(1), log_probs.stride(2),
 674:        log_beta.stride(0), log_beta.stride(1), log_beta.stride(2),
 675:        tg_batch_offsets.template const_data_ptr<int64_t>(), tg_target_stride,
 676:        batch_size, BLANK);
 677:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 678:   }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 680-683
```cpp
 680:   // Very crude heuristic for what is a small problem., based on linearly regressing problem dimensions on
 681:   // the (capped) difference of timings.
 682:   // Note that for OK problems target length <= input length, so we
 683:   // only consider input length.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 684-705
```cpp
 684:   bool is_large = (2*log_probs.size(0)+(24*batch_size)/10+(2*num_labels)/10) > 450;
 685:   if (is_large) { // large alphabet, large batch
 686:     // this computes the probs, minuend in (16)
 687:     at::exp_out(grad, log_probs);
 688:     // now we compute the subtrahend for the blanks. It is a straightforward reduction because we know that
 689:     // blanks are in every other position.
 690:     // maybe we should kernelize this, too.
 691:     auto grad_blank = grad.narrow(2, BLANK, 1);
 692:     grad_blank -= (at::logsumexp(log_alpha.as_strided({batch_size, log_alpha.size(1), max_target_length+1},
 693:                                                       {log_alpha.stride(0), log_alpha.stride(1), log_alpha.stride(2)*2})
 694:                                  + log_beta.as_strided({batch_size, log_beta.size(1), max_target_length+1},
 695:                                                        {log_beta.stride(0), log_beta.stride(1), log_beta.stride(2)*2}),
 696:                                  2, true)
 697:                    .permute({1, 0, 2})
 698:                    .add_(neg_log_likelihood.view({1, batch_size, 1}))
 699:                    .sub_(log_probs.narrow(2, BLANK, 1))
 700:                    .exp_()
 701:                    );
 702:     // scale by output gradient (blanks and first summand of non-blanks)
 703:     grad *= grad_out.view({1, batch_size, 1});
 704:     if (zero_infinity) {
 705:       grad = at::where(neg_log_likelihood.view({1, batch_size, 1}) == Scalar(INFINITY), at::zeros({}, grad.options()), grad);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 706-706
```cpp
 706:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 708-709
```cpp
 708:     // For the non-blank characters, we use a kernel to compute the subtrahend.
 709:     // Again we might configure block and grid in a better way.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 710-723
```cpp
 710:     int threads_target = max_threads;
 711:     while (threads_target / 2 >= max_target_length && threads_target > 1) {
 712:       threads_target /= 2;
 713:     }
 714:     int threads_batch = std::min(max_threads / threads_target, (int) batch_size);
 715:     dim3 block(threads_target, threads_batch);
 716:     dim3 grid(
 717:         std::max<int>(
 718:             (max_target_length + threads_target - 1) / threads_target, 1),
 719:         (batch_size + threads_batch - 1) / threads_batch,
 720:         1);
 721:     ctc_loss_backward_collect_nonblank_gpu_kernel<scalar_t, target_t><<<grid, block, 0, stream>>>
 722:       (grad.mutable_data_ptr<scalar_t>(),
 723:        grad_out.const_data_ptr<scalar_t>(), grad_out.stride(0),
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 724-737
```cpp
 724:        log_alpha.const_data_ptr<scalar_t>(), log_beta.const_data_ptr<scalar_t>(),
 725:        log_probs.const_data_ptr<scalar_t>(), input_lengths_t.template const_data_ptr<int64_t>(),
 726:        targets.const_data_ptr<target_t>(), target_lengths_t.template const_data_ptr<int64_t>(),
 727:        neg_log_likelihood.const_data_ptr<scalar_t>(),
 728:        grad.stride(0), grad.stride(1), grad.stride(2),
 729:        log_probs.stride(0), log_probs.stride(1), log_probs.stride(2),
 730:        log_alpha.stride(0), log_alpha.stride(1), log_alpha.stride(2),
 731:        log_beta.stride(0), log_beta.stride(1), log_beta.stride(2),
 732:        tg_batch_offsets.template const_data_ptr<int64_t>(), tg_target_stride,
 733:        batch_size, zero_infinity);
 734:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 735:   } else { // small problem, use naive algorithm
 736:     // Still no block/grid configuration guru...
 737:     int threads_input = max_threads;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 738-751
```cpp
 738:     while (threads_input / 2 >= log_probs.size(0) && threads_input > 1) {
 739:       threads_input /= 2;
 740:     }
 741:     threads_batch = std::min(max_threads / threads_input, (int) batch_size);
 742:     dim3 block(threads_input, threads_batch);
 743:     dim3 grid((log_probs.size(0) + threads_input-1)/threads_input, (batch_size+threads_batch-1)/threads_batch);
 744:     ctc_loss_backward_collect_gpu_kernel<scalar_t, target_t><<<grid, block, 0, stream>>>
 745:       (grad.mutable_data_ptr<scalar_t>(),
 746:        grad_out.const_data_ptr<scalar_t>(), grad_out.stride(0),
 747:        log_alpha.const_data_ptr<scalar_t>(), log_beta.const_data_ptr<scalar_t>(),
 748:        log_probs.const_data_ptr<scalar_t>(), input_lengths_t.template const_data_ptr<int64_t>(), log_probs.size(0),
 749:        targets.const_data_ptr<target_t>(), target_lengths_t.template const_data_ptr<int64_t>(), max_target_length,
 750:        neg_log_likelihood.const_data_ptr<scalar_t>(),
 751:        grad.stride(0), grad.stride(1), grad.stride(2),
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 752-758
```cpp
 752:        log_probs.stride(0), log_probs.stride(1), log_probs.stride(2),
 753:        log_alpha.stride(0), log_alpha.stride(1), log_alpha.stride(2),
 754:        log_beta.stride(0), log_beta.stride(1), log_beta.stride(2),
 755:        tg_batch_offsets.template const_data_ptr<int64_t>(), tg_target_stride,
 756:        batch_size, num_labels, BLANK, zero_infinity);
 757:     C10_CUDA_KERNEL_LAUNCH_CHECK(); // catch launch errors
 758:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 760-760
```cpp
 760:   // zero those invalid graident elements due to padding
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 761-782
```cpp
 761:   {
 762:     int threads_input = max_threads;
 763:     while (threads_input / 2 >= log_probs.size(0)) {
 764:       threads_input /= 2;
 765:     }
 766:     threads_batch = std::min(max_threads / threads_input, (int) batch_size);
 767:     dim3 block(threads_input, threads_batch);
 768:     dim3 grid(
 769:       (log_probs.size(0) + threads_input-1)/threads_input,
 770:       (batch_size+threads_batch-1)/threads_batch);
 771:     ctc_loss_zero_padded_gradients<scalar_t><<<grid, block, 0, stream>>>(
 772:       grad.mutable_data_ptr<scalar_t>(),
 773:       input_lengths_t.template const_data_ptr<int64_t>(),
 774:       grad.stride(0),
 775:       grad.stride(1),
 776:       grad.stride(2),
 777:       grad.size(0),
 778:       grad.size(1),
 779:       grad.size(2)
 780:     );
 781:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 782:   }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 784-785
```cpp
 784:   return grad;
 785: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 787-787
```cpp
 787: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 789-798
```cpp
 789: std::tuple<Tensor, Tensor> ctc_loss_gpu(const Tensor& log_probs, const Tensor& targets, IntArrayRef input_lengths, IntArrayRef target_lengths, int64_t BLANK, bool zero_infinity) {
 790:   (void)zero_infinity; // only used for backward
 791:   return AT_DISPATCH_FLOATING_TYPES(log_probs.scalar_type(), "ctc_loss_cuda", [&] {
 792:       if (targets.scalar_type() == kLong) {
 793:         return ctc_loss_gpu_template<scalar_t, kLong>(log_probs, targets, input_lengths, target_lengths, BLANK);
 794:       } else {
 795:         return ctc_loss_gpu_template<scalar_t, kInt>(log_probs, targets, input_lengths, target_lengths, BLANK);
 796:       }
 797:     });
 798: }
```
- EN: This block defines or continues the implementation of `ctc_loss_gpu`.
- CN: 该代码块定义或继续实现 `ctc_loss_gpu`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 800-812
```cpp
 800: Tensor ctc_loss_backward_gpu(const Tensor& grad, const Tensor& log_probs, const Tensor& targets, IntArrayRef input_lengths, IntArrayRef target_lengths,
 801:                              const Tensor& neg_log_likelihood, const Tensor& log_alpha, int64_t BLANK, bool zero_infinity) {
 802:   // See Note [Writing Nondeterministic Operations]
 803:   // Nondeterministic because of atomicAdd usage
 804:   globalContext().alertNotDeterministic("ctc_loss_backward_gpu");
 805:   return AT_DISPATCH_FLOATING_TYPES(log_probs.scalar_type(), "ctc_loss_backward_cuda", [&] {
 806:       if (targets.scalar_type() == kLong) {
 807:         return ctc_loss_backward_gpu_template<scalar_t, kLong>(grad, log_probs, targets, input_lengths, target_lengths, neg_log_likelihood, log_alpha, BLANK, zero_infinity);
 808:       } else {
 809:         return ctc_loss_backward_gpu_template<scalar_t, kInt>(grad, log_probs, targets, input_lengths, target_lengths, neg_log_likelihood, log_alpha, BLANK, zero_infinity);
 810:       }
 811:     });
 812: }
```
- EN: This block defines or continues the implementation of `ctc_loss_backward_gpu`.
- CN: 该代码块定义或继续实现 `ctc_loss_backward_gpu`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 814-814
```cpp
 814: } // at::native
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `gpu_kernel` applies a functor over iterator elements on CUDA. / `gpu_kernel` 在 CUDA 上对迭代器元素应用函数对象。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。
- cuDNN calls delegate work to NVIDIA's tuned deep-learning kernels. / cuDNN 调用把工作委托给 NVIDIA 优化过的深度学习内核。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/TensorUtils.h>`
  - `<c10/util/Exception.h>`
  - `<c10/macros/Macros.h>`
  - `<ATen/core/Tensor.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/TensorOperators.h>`
  - `<ATen/cuda/Atomic.cuh>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/Functions.h>`
  - `<ATen/NativeFunctions.h>`
  - `<ATen/ops/_ctc_loss_backward_native.h>`
  - `<ATen/ops/_ctc_loss_native.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_FLOATING_TYPES`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
