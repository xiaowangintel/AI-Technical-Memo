# TensorFactories.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/TensorFactories.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `eye_out_cuda`, `empty_cuda`, `_efficientzerotensor_cuda`, `empty_strided_cuda`.
- 用途（中文）: 实现与 `eye_out_cuda`, `empty_cuda`, `_efficientzerotensor_cuda`, `empty_strided_cuda` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/Dispatch.h>
   4: #include <ATen/cuda/CUDAApplyUtils.cuh>
   5: #include <ATen/cuda/CUDAContext.h>
   6: #include <ATen/cuda/EmptyTensor.h>
   7: #include <ATen/InitialTensorOptions.h>
   8: #include <ATen/native/cuda/Resize.h>
   9: #include <ATen/native/TensorFactories.h>
  10: #include <c10/util/accumulate.h>
  11: #include <c10/util/Exception.h>
  12: #include <ATen/native/cuda/Loops.cuh>
  13: 
  14: #ifndef AT_PER_OPERATOR_HEADERS
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/cuda/CUDAApplyUtils.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/cuda/CUDAApplyUtils.cuh>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 15-28
```cpp
  15: #include <ATen/Functions.h>
  16: #include <ATen/NativeFunctions.h>
  17: #else
  18: #include <ATen/ops/_efficientzerotensor_native.h>
  19: #include <ATen/ops/empty_native.h>
  20: #include <ATen/ops/empty_strided_native.h>
  21: #include <ATen/ops/eye_native.h>
  22: #include <ATen/ops/tril_indices_native.h>
  23: #include <ATen/ops/tril_native.h>
  24: #include <ATen/ops/triu_indices_native.h>
  25: #include <ATen/ops/triu_native.h>
  26: #endif
  27: 
  28: #include <algorithm>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/_efficientzerotensor_native.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/_efficientzerotensor_native.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 29-30
```cpp
  29: #include <cmath>
  30: #include <cstddef>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<cmath>`, `<cstddef>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<cmath>`, `<cstddef>`。

### Lines 32-52
```cpp
  32: namespace at::native {
  33: 
  34: Tensor& eye_out_cuda(int64_t n, Tensor& result) {
  35:   // the default value of `m` equals to `n`
  36:   return at::native::eye_out_cuda(n, n, result);
  37: }
  38: 
  39: Tensor& eye_out_cuda(int64_t n, int64_t m, Tensor& result) {
  40:   TORCH_CHECK(n >= 0, "n must be greater or equal to 0, got ", n);
  41:   TORCH_CHECK(m >= 0, "m must be greater or equal to 0, got ", m);
  42: 
  43:   result.resize_({n, m});
  44:   result.zero_();
  45: 
  46:   int64_t sz = std::min<int64_t>(n, m);
  47:   int64_t stride = result.stride(0) + result.stride(1);
  48: 
  49:   Tensor diag = result.as_strided({sz}, {stride});
  50:   diag.fill_(1);
  51:   return result;
  52: }
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `eye_out_cuda`.
- CN: 该代码块定义或继续实现 `eye_out_cuda`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 54-61
```cpp
  54: Tensor empty_cuda(IntArrayRef size, std::optional<ScalarType> dtype_opt, std::optional<Layout> layout_opt, std::optional<Device> device_opt, std::optional<bool> pin_memory_opt, std::optional<c10::MemoryFormat> memory_format_opt) {
  55:   Tensor result = at::detail::empty_cuda(size, dtype_opt, layout_opt, device_opt, pin_memory_opt, memory_format_opt);
  56:   // See Note [Enabling Deterministic Operations]
  57:   if (C10_UNLIKELY(at::globalContext().deterministicAlgorithms() && at::globalContext().deterministicFillUninitializedMemory())) {
  58:     fill_empty_deterministic_(result);
  59:   }
  60:   return result;
  61: }
```
- EN: This block defines or continues the implementation of `empty_cuda`.
- CN: 该代码块定义或继续实现 `empty_cuda`。

### Lines 63-77
```cpp
  63: Tensor _efficientzerotensor_cuda(IntArrayRef size,
  64:     std::optional<ScalarType> dtype,
  65:     std::optional<Layout> layout,
  66:     std::optional<Device> device,
  67:     std::optional<bool> pin_memory) {
  68:     auto device_ = device_or_default(device);
  69:     if (!device_.has_index()) {
  70:       device_.set_index(at::cuda::current_device());
  71:     }
  72:     auto allocator = at::native::ZeroTensorAllocator(device_);
  73:     auto dtype_ = dtype_or_default(dtype);
  74:     auto zero_ks = at::DispatchKeySet(c10::DispatchKey::CUDA) | at::DispatchKeySet(c10::DispatchKey::ZeroTensor);
  75:     auto out = at::detail::empty_generic(size, &allocator, zero_ks, dtype_, std::nullopt);
  76:     return out;
  77: }
```
- EN: This block defines or continues the implementation of `_efficientzerotensor_cuda`.
- CN: 该代码块定义或继续实现 `_efficientzerotensor_cuda`。

### Lines 80-87
```cpp
  80: Tensor empty_strided_cuda(IntArrayRef size, IntArrayRef stride, std::optional<ScalarType> dtype_opt, std::optional<Layout> layout_opt, std::optional<Device> device_opt, std::optional<bool> pin_memory_opt) {
  81:   Tensor result = at::detail::empty_strided_cuda(size, stride, dtype_opt, layout_opt, device_opt, pin_memory_opt);
  82:   // See Note [Enabling Deterministic Operations]
  83:   if (C10_UNLIKELY(at::globalContext().deterministicAlgorithms() && at::globalContext().deterministicFillUninitializedMemory())) {
  84:     fill_empty_deterministic_(result);
  85:   }
  86:   return result;
  87: }
```
- EN: This block defines or continues the implementation of `empty_strided_cuda`.
- CN: 该代码块定义或继续实现 `empty_strided_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 89-89
```cpp
  89: // ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ triangle ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 91-112
```cpp
  91: namespace {
  92: // To find the max integer that does not exceed the root of an int64_t variable,
  93: // we could use a loop to test one bit at a time, which takes up to 31
  94: // iterations. This would give the accurate result, but is relatively slow and
  95: // is an overkill for most cases where double's precision suffice.
  96: //
  97: // If we directly use sqrt to calculate the root, the conversion from int64_t
  98: // to double would lose 11 bits precision.
  99: //
 100: // The following solution uses sqrt directly for most cases, and would only
 101: // special handle it if there is indeed precision loss.
 102: __device__
 103: inline int64_t resolve_root_int(
 104:     int64_t b, int64_t cX4, int64_t x, int32_t sign) {
 105:   int64_t bXb_cX4 = b*b - cX4;
 106:   // potential precision loss could occur here when casting int64_t (63 bits
 107:   // precision) to double (52 bits precision)
 108:   double sr = ::sqrt((double)bXb_cX4);
 109:   int64_t res = ::__double2ll_rd((-b + sign * sr)/2);
 110: 
 111:   // have to cast double to int64_t, otherwise it would only compare up to the
 112:   // precision of a double variable, ignoring the precision loss
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `resolve_root_int`.
- CN: 该代码块定义或继续实现 `resolve_root_int`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 113-134
```cpp
 113:   if (bXb_cX4 != (int64_t) (sr * sr)) {
 114:     // handle precision loss by using binary search
 115:     int64_t llsr = ::__double2ll_rd(sr);
 116:     // Use the following math to reduce search space.
 117:     // Suppose z is the accurate result of sqrt(bXb_cX4) without precision loss
 118:     // let d = abs(bXb_cX4 - llsr * llsr), then we have:
 119:     // z = sqrt(bXb_cX4) <= sqrt(llsr * llsr + d) <= llsr + sqrt(d)
 120:     // z = sqrt(bXb_cX4) >= sqrt(llsr * llsr - d) >= llsr - sqrt(d)
 121:     // Hence, it is sufficient to search range [llsr - sqrt(d), llsr + sqrt(d)).
 122:     // And the true value of row would also be with in range,
 123:     //            [res - sqrt(d), res + sqrt(d) + 1)
 124:     // as the denominator would only reduce the precision penalty.
 125:     int64_t diff =
 126:       ::__double2ll_ru(::sqrt(::fabs((double)(bXb_cX4 - llsr * llsr))));
 127:     // l never exceeds (could equal to) the target row index
 128:     auto l = res > diff ? res - diff : 0;
 129:     // r is always larger than the target row index
 130:     auto r = res + diff + 1;
 131: 
 132:     // binary search for the correct answer
 133:     x <<= 1; // the loop always compares with 2x, so do it once here
 134:     while (l + 1 < r) {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 135-147
```cpp
 135:       auto m = (l + r) >> 1;
 136:       // for tril:
 137:       //    b = 2f - 1, sign = 1, hence (2f + m - 1) * m / 2
 138:       // for triu:
 139:       //    b = -2f - 1, sign = -1, hence (2f - m + 1) * m / 2
 140:       if (sign * (b + m) * m > x) {
 141:         r = m;
 142:       } else {
 143:         l = m;
 144:       }
 145:     }
 146:     res = l;
 147:   }
```
- EN: This block defines or continues the implementation of `hence`.
- CN: 该代码块定义或继续实现 `hence`。

### Lines 149-150
```cpp
 149:   return res;
 150: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 152-165
```cpp
 152: // f: the number of elements in the first row of the trapezoid.
 153: // x: the index of the target coordinates ordered by row and then column.
 154: //
 155: // View the tril as a top trapezoid stacked on a bottom rectangle. Assume x
 156: // corresponds to the coordinate (row, col) in the trapezoid, where the row and
 157: // the col both start from 0, then we have:
 158: //
 159: //                   (f + f + row - 1) * row / 2 <= x                       [1]
 160: //                 (f + f + row) * (row + 1) / 2  > x                       [2]
 161: //
 162: // Therefore, row is the maximum integer satisfying the following inequality:
 163: //
 164: //                       (row + 2f - 1)row <= 2x
 165: //                  row^2 + (2f-1)row - 2x <= 0.                            [3]
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 166-179
```cpp
 166: //
 167: // Based on inequality [3], we have the following coefficients for formula of
 168: // root:
 169: //                               a = 1
 170: //                               b = 2f - 1
 171: //                               c = -2x
 172: // There are two roots, and we should use the largest integer that does not
 173: // exceed the root on the right. Intuitively, it is because:
 174: //  i)  the valid solution range of row is between two roots, as it is <= 0;
 175: //  ii) as we count in more rows, the total # of elements should always
 176: //      increase, hence so does the left-hand side row^2 + (2f-1)row - 2x.
 177: //      Therefore, the valid range of row lies in between the nadir point and
 178: //      the larger root on the right.
 179: // Full proof can be derived from inequality [2]. So, we calculate the result
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 180-183
```cpp
 180: // coordinate as:
 181: //
 182: //                   row = floor((-b + sqrt(b^2 - 4c)) / 2)
 183: //                   col = x - (f + f + row - 1) * row / 2
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 184-192
```cpp
 184: __device__
 185: inline void get_coordinate_in_tril_trapezoid(
 186:     int64_t f, int64_t x, int64_t & row, int64_t & col) {
 187:   f <<= 1; // all statements use 2f, so only calculate it once here.
 188:   auto b = f - 1;
 189:   auto cX4 = - (x << 3); // 4 * c = 4 * (-2x) = -8x;
 190:   row = resolve_root_int(b, cX4, x, 1);
 191:   col = x - ((f + row - 1) * row >> 1);
 192: }
```
- EN: This block defines or continues the implementation of `get_coordinate_in_tril_trapezoid`.
- CN: 该代码块定义或继续实现 `get_coordinate_in_tril_trapezoid`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 194-207
```cpp
 194: // f: the number of elements in the first row of the bottom trapezoid.
 195: // x: the index of the target coordinates ordered by row and then column.
 196: //
 197: // View the triu as a top rectangle stacked on a bottom trapezoid, where the
 198: // trapezoid is upside down. Assume x corresponds to the coordinate (row, col)
 199: // in the bottom trapezoid, where the row and the col start from 0, then we
 200: // have:
 201: //
 202: //                   (f + f - row + 1) * row / 2 <= x                       [1]
 203: //                 (f + f - row) * (row + 1) / 2  > x                       [2]
 204: //
 205: // Therefore, row is the maximum integer satisfying the following inequality:
 206: //
 207: //                       (-row + 2f + 1)row <= 2x
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 208-221
```cpp
 208: //                   row^2 - (2f+1)row + 2x >= 0.                           [3]
 209: //
 210: // Based on inequality [3], we have the following coefficients for formula of
 211: // root:
 212: //                               a = 1
 213: //                               b = -1 - 2f
 214: //                               c = 2x
 215: // There are two roots, and we should use the largest integer that does not
 216: // exceed the root on the left. Intuitively, it is because:
 217: //  i)  the valid solution range of row is outside of the two roots, as it is <
 218: //      > 0;
 219: //  ii) as we count in more rows, the total # of elements should always
 220: //      increase, hence so does the left-hand side row^2 - (2f+1)row + 2x.
 221: //      Therefore, the valid range of row lies to the left of the smaller root
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 222-227
```cpp
 222: //      on the left.
 223: // Full proof can be derived from inequality [2]. So, we calculate the result
 224: // coordinate as:
 225: //
 226: //                   row = floor((-b - sqrt(b^2 - 4c)) / 2)
 227: //                   col = x - (f + f - row + 1) * row / 2
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 228-236
```cpp
 228: __device__
 229: inline void get_coordinate_in_triu_trapezoid(
 230:     int64_t f, int64_t x, int64_t & row, int64_t & col) {
 231:   f <<= 1; // all statements use 2f, so only calculate it once here.
 232:   auto b = -1 - f;
 233:   auto cX4 = x << 3; // 4 * c = 4 * (2x) = 8x;
 234:   row = resolve_root_int(b, cX4, x, -1);
 235:   col = x - ((f - row + 1) * row >> 1) + row;
 236: }
```
- EN: This block defines or continues the implementation of `get_coordinate_in_triu_trapezoid`.
- CN: 该代码块定义或继续实现 `get_coordinate_in_triu_trapezoid`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 238-238
```cpp
 238: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 240-261
```cpp
 240: template <typename scalar_t>
 241: __global__
 242: #if defined(USE_ROCM)
 243: C10_LAUNCH_BOUNDS_1(512)
 244: #endif
 245: void tril_indices_kernel(scalar_t * tensor,
 246:                          int64_t row_offset,
 247:                          int64_t m_first_row,
 248:                          int64_t col,
 249:                          int64_t trapezoid_size,
 250:                          int64_t tril_size) {
 251:   int64_t linear_index = ((int64_t) blockIdx.x) * blockDim.x + threadIdx.x;
 252: 
 253:   if (linear_index < tril_size) {
 254:     int64_t r, c;
 255:     if (linear_index < trapezoid_size) {
 256:       // the coordinate is within the top trapezoid
 257:       get_coordinate_in_tril_trapezoid(m_first_row, linear_index, r, c);
 258:     } else {
 259:       // the coordinate falls in the bottom rectangle
 260:       auto surplus = linear_index - trapezoid_size;
 261:       // add the height of trapezoid: m_last_row (col) - m_first_row + 1
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines GPU kernel entry point(s) `defined`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `defined`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 262-265
```cpp
 262:       r = surplus / col + col - m_first_row + 1;
 263:       c = surplus % col;
 264:     }
 265:     r += row_offset;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 267-270
```cpp
 267:     tensor[linear_index] = r;
 268:     tensor[linear_index + tril_size] = c;
 269:   }
 270: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 272-275
```cpp
 272: // Some Large test cases for the fallback binary search path is disabled by
 273: // default to speed up CI tests and to avoid OOM error. When modifying the
 274: // implementation, please enable them in test/test_cuda.py and make sure they
 275: // pass on your local server.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 276-297
```cpp
 276: Tensor tril_indices_cuda(
 277:     int64_t row, int64_t col, int64_t offset, std::optional<ScalarType> dtype_opt,
 278:     std::optional<Layout> layout_opt, std::optional<Device> device_opt, std::optional<bool> pin_memory_opt) {
 279:   check_args(row, col, layout_opt);
 280: 
 281:   auto tril_size = get_tril_size(row, col, offset);
 282:   auto tensor = empty_cuda({2, tril_size}, dtype_opt, layout_opt, device_opt, pin_memory_opt);
 283: 
 284:   if (tril_size > 0) {
 285:     auto m_first_row = offset > 0 ?
 286:       std::min<int64_t>(col, 1 + offset) : // upper bounded by col
 287:       row + offset > 0; // either 0 or 1
 288:     auto trapezoid_row_offset = std::max<int64_t>(0, -offset);
 289:     auto rectangle_row_offset = trapezoid_row_offset + col - m_first_row + 1;
 290:     int64_t rectangle_size = 0;
 291:     if (rectangle_row_offset < row) {
 292:       rectangle_size = (row - rectangle_row_offset) * col;
 293:     }
 294: 
 295:     dim3 dim_block = cuda::getApplyBlock();
 296:     dim3 dim_grid;
 297:     // using tril_size instead of tensor.numel(), as each thread takes care of
```
- EN: This block defines or continues the implementation of `tril_indices_cuda`.
- CN: 该代码块定义或继续实现 `tril_indices_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 298-298
```cpp
 298:     // two elements in the tensor.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 299-301
```cpp
 299:     TORCH_CHECK(
 300:       cuda::getApplyGrid(tril_size, dim_grid, tensor.get_device()),
 301:       "unable to get dim grid");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 303-314
```cpp
 303:     AT_DISPATCH_INDEX_TYPES(tensor.scalar_type(), "tril_indices_cuda", [&] {
 304:       tril_indices_kernel<<<
 305:           dim_grid, dim_block, 0, at::cuda::getCurrentCUDAStream()>>>(
 306:         tensor.mutable_data_ptr<index_t>(),
 307:         trapezoid_row_offset,
 308:         m_first_row,
 309:         col,
 310:         tril_size - rectangle_size,
 311:         tril_size);
 312:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 313:     });
 314:   }
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 316-317
```cpp
 316:   return tensor;
 317: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 319-340
```cpp
 319: template <typename scalar_t>
 320: __global__
 321: void triu_indices_kernel(scalar_t * tensor,
 322:                          int64_t col_offset,
 323:                          int64_t m_first_row,
 324:                          int64_t col,
 325:                          int64_t rectangle_size,
 326:                          int64_t triu_size) {
 327:   int64_t linear_index = ((int64_t) blockIdx.x) * blockDim.x + threadIdx.x;
 328: 
 329:   if (linear_index < triu_size) {
 330:     int64_t r, c;
 331:     if (linear_index < rectangle_size) {
 332:       // the coordinate is within the top rectangle
 333:       r = linear_index / col;
 334:       c = linear_index % col;
 335:     } else {
 336:       // the coordinate falls in the bottom trapezoid
 337:       get_coordinate_in_triu_trapezoid(
 338:         m_first_row, linear_index - rectangle_size, r, c);
 339:       r += rectangle_size / col;
 340:     }
```
- EN: This block defines GPU kernel entry point(s) `triu_indices_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `triu_indices_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 342-346
```cpp
 342:     c += col_offset;
 343:     tensor[linear_index] = r;
 344:     tensor[linear_index + triu_size] = c;
 345:   }
 346: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 348-351
```cpp
 348: // Some Large test cases for the fallback binary search path is disabled by
 349: // default to speed up CI tests and to avoid OOM error. When modifying the
 350: // implementation, please enable them in test/test_cuda.py and make sure they
 351: // pass on your local server.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 352-373
```cpp
 352: Tensor triu_indices_cuda(
 353:     int64_t row, int64_t col, int64_t offset, std::optional<ScalarType> dtype_opt,
 354:     std::optional<Layout> layout_opt, std::optional<Device> device_opt, std::optional<bool> pin_memory_opt) {
 355:   check_args(row, col, layout_opt);
 356: 
 357:   auto triu_size = row * col - get_tril_size(row, col, offset - 1);
 358:   auto tensor = empty_cuda({2, triu_size}, dtype_opt, layout_opt, device_opt, pin_memory_opt);
 359: 
 360:   if (triu_size > 0) {
 361:     // # of triu elements in the first row
 362:     auto m_first_row = offset > 0 ?
 363:       std::max<int64_t>(col - offset, 0) : // upper bounded by col
 364:       col;
 365: 
 366:     // size of the top rectangle
 367:     int64_t rectangle_size = 0;
 368:     if (offset < 0) {
 369:       rectangle_size = std::min<int64_t>(row, -offset) * col;
 370:     }
 371: 
 372:     dim3 dim_block = cuda::getApplyBlock();
 373:     dim3 dim_grid;
```
- EN: This block defines or continues the implementation of `triu_indices_cuda`.
- CN: 该代码块定义或继续实现 `triu_indices_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 375-376
```cpp
 375:     // using triu_size instead of tensor.numel(), as each thread takes care of
 376:     // two elements in the tensor.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 377-379
```cpp
 377:     TORCH_CHECK(
 378:       cuda::getApplyGrid(triu_size, dim_grid, tensor.get_device()),
 379:       "unable to get dim grid");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 381-392
```cpp
 381:     AT_DISPATCH_INDEX_TYPES(tensor.scalar_type(), "triu_indices_cuda", [&] {
 382:       triu_indices_kernel<<<
 383:           dim_grid, dim_block, 0, at::cuda::getCurrentCUDAStream()>>>(
 384:         tensor.mutable_data_ptr<index_t>(),
 385:         std::max<int64_t>(0, offset),
 386:         m_first_row,
 387:         col,
 388:         rectangle_size,
 389:         triu_size);
 390:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 391:     });
 392:   }
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 394-395
```cpp
 394:   return tensor;
 395: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 397-397
```cpp
 397: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/cuda/CUDAApplyUtils.cuh>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/EmptyTensor.h>`
  - `<ATen/InitialTensorOptions.h>`
  - `<ATen/native/cuda/Resize.h>`
  - `<ATen/native/TensorFactories.h>`
  - `<c10/util/accumulate.h>`
  - `<c10/util/Exception.h>`
  - `<ATen/native/cuda/Loops.cuh>`
  - `<ATen/Functions.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_INDEX_TYPES`
  - `at::cuda::current_device`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
