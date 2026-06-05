# ScanUtils.cuh — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/ScanUtils.cuh`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Declares or defines CUDA helpers/templates associated with `ceil_div`, `get_log_num_threads_x_inner_scan`, `binary_op_update`, `tensor_kernel_scan_innermost_dim_with_indices`.
- 用途（中文）: 声明或定义与 `ceil_div`, `get_log_num_threads_x_inner_scan`, `binary_op_update`, `tensor_kernel_scan_innermost_dim_with_indices` 相关的 CUDA 辅助函数/模板。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```cpp
   1: #pragma once
   2: #include <ATen/NumericUtils.h>
   3: #include <ATen/core/TensorBase.h>
   4: #include <ATen/cuda/cub.cuh>
   5: #include <ATen/cuda/CUDAContext.h>
   6: 
   7: #include <c10/util/Load.h>
   8: #include <limits>
   9: #include <cmath>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/NumericUtils.h>`, `<ATen/core/TensorBase.h>`, `<ATen/cuda/cub.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/NumericUtils.h>`, `<ATen/core/TensorBase.h>`, `<ATen/cuda/cub.cuh>`。

### Lines 11-32
```cpp
  11: namespace at::native {
  12: 
  13: template <typename integer>
  14: constexpr inline integer ceil_div(integer n, integer m) {
  15:   return (n + m - 1) / m;
  16: }
  17: 
  18: template <typename integer>
  19: constexpr inline integer get_log_num_threads_x_inner_scan(integer num_rows, integer row_size) {
  20:   integer log_num_threads_x = 0;
  21:   integer log_num_threads_y = 0;
  22:   while (((integer)1 << log_num_threads_x) < row_size) {
  23:     ++log_num_threads_x;
  24:   }
  25:   while (((integer)1 << log_num_threads_y) < num_rows) {
  26:     ++log_num_threads_y;
  27:   }
  28:   // we want to keep the ratio between the x-threads and y-threads about the same as
  29:   // the ratio between the row_size and num_rows, but the total number of threads in
  30:   // a block should be about 512
  31:   integer diff = log_num_threads_x - log_num_threads_y;
  32:   // 9 is from log2(512)
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `ceil_div`, `get_log_num_threads_x_inner_scan`.
- CN: 该代码块定义或继续实现 `ceil_div`, `get_log_num_threads_x_inner_scan`。

### Lines 33-40
```cpp
  33:   log_num_threads_x = ((integer)9 + diff) / (integer)2;
  34:   // I found that in having larger log_num_threads_x can give significant speed up in some cases,
  35:   // but detrimental in another case, so just keep the lower bound to be log2(16) == 4 to make it
  36:   // similar to the previous implementation
  37:   // Keeping the upper bound to be log2(512) == 9 as the maximum number of threads in a block.
  38:   log_num_threads_x = std::min(std::max((integer)4, log_num_threads_x), (integer)9);
  39:   return log_num_threads_x;
  40: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 42-55
```cpp
  42: template<typename scalar_t, typename idx_t, typename BinaryOperation>
  43: __device__ void binary_op_update(const scalar_t lhs, scalar_t& rhs, const idx_t lhs_idx, idx_t& rhs_idx, BinaryOperation binary_op) {
  44:   if(!at::_isnan(rhs) && (at::_isnan(lhs) || !binary_op(rhs, lhs))) {
  45:     rhs = lhs;
  46:     rhs_idx = lhs_idx;
  47:   }
  48: }
  49: /* Perform an inclusive scan along the innermost dimension of a tensor.
  50:  *
  51:  * - num_rows is the size of the flattened outer dimensions;
  52:  * - row_size is the size of the innermost dimension;
  53:  *
  54:  * The outer dimensions of the tensor are considered as a single dimension, i.e. the tensor is
  55:  * considered as having 'num_rows' rows of size 'row_size'.
```
- EN: This block defines or continues the implementation of `binary_op_update`.
- CN: 该代码块定义或继续实现 `binary_op_update`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 56-58
```cpp
  56:  * Each thread block processes one or more sets of contiguous rows (processing multiple rows
  57:  * per thread block is quicker than processing a single row, especially for short rows).
  58:  */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 59-80
```cpp
  59: template<typename scalar_t, class BinaryFunction>
  60: __global__ void tensor_kernel_scan_innermost_dim_with_indices(const scalar_t *self_, scalar_t *values_, int64_t *indices_,
  61:                                                 int num_rows, int row_size,
  62:                                                 const uint32_t num_threads, const uint32_t log_num_threads_x,
  63:                                                 scalar_t init, BinaryFunction binary_op) {
  64:   // dynamic memory allocation for vbuf and ibuf
  65:   alignas(sizeof(double)) extern __shared__ char buf[];
  66:   scalar_t* vbuf = reinterpret_cast<scalar_t*>(buf); // the size is num_threads * 2
  67:   int64_t* ibuf = reinterpret_cast<int64_t*>(vbuf + num_threads * 2);
  68:   const uint32_t num_threads_x = 1 << log_num_threads_x;
  69:   scalar_t* row_buf = vbuf + 2 * num_threads_x * threadIdx.y;
  70:   int64_t* row_idx_buf = ibuf + 2 * num_threads_x * threadIdx.y;
  71: 
  72:   for (int block_row = blockIdx.x * blockDim.y;
  73:        block_row < num_rows;
  74:        block_row += blockDim.y * gridDim.x) {
  75:     int row = block_row + threadIdx.y;
  76:     const scalar_t *row_self = self_ + row * row_size;
  77:     scalar_t *row_values = values_ + row * row_size;
  78:     int64_t *row_indices = indices_ + row * row_size;
  79:     scalar_t block_total = init;
  80:     int64_t block_idx_final = 0;
```
- EN: This block defines GPU kernel entry point(s) `tensor_kernel_scan_innermost_dim_with_indices`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `tensor_kernel_scan_innermost_dim_with_indices`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 81-102
```cpp
  81:     const bool row_exists = row < num_rows;
  82:     // Perform scan on one block at a time, keeping track of the total value of
  83:     // all blocks processed so far.
  84:     for (int block_col = 0; block_col < row_size; block_col += 2 * num_threads_x) {
  85:       // Load data into shared memory (two values per thread).
  86:       int col1 = block_col + threadIdx.x;
  87:       int col2 = block_col + num_threads_x + threadIdx.x;
  88:       if (row_exists) {
  89:         if (col1 < row_size) {
  90:           row_buf[threadIdx.x] = c10::load(&row_self[col1]);
  91:           row_idx_buf[threadIdx.x] = col1;
  92:         } else {
  93:           row_buf[threadIdx.x] = init;
  94:           // No need to set the index here as the value in init will never be selected
  95:         }
  96: 
  97:         if (col2 < row_size) {
  98:           row_buf[num_threads_x + threadIdx.x] = c10::load(&row_self[col2]);
  99:           row_idx_buf[num_threads_x + threadIdx.x] = col2;
 100:         } else {
 101:           row_buf[num_threads_x + threadIdx.x] = init;
 102:           // No need to set the index here as the value in init will never be selected
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 103-103
```cpp
 103:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 105-105
```cpp
 105:         // Add the total value of all previous blocks to the first value of this block.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 106-110
```cpp
 106:         if (threadIdx.x == 0) {
 107:           binary_op_update(block_total, row_buf[0], block_idx_final, row_idx_buf[0], binary_op);
 108:         }
 109:       }
 110:       __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 112-113
```cpp
 112:       // Parallel reduction with Sklansky method. The diagram can be seen on this paper:
 113:       // https://research.nvidia.com/publication/single-pass-parallel-prefix-scan-decoupled-look-back
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 114-122
```cpp
 114:       for (uint32_t s = 1; s <= num_threads_x; s <<= 1) {
 115:         if (row_exists) {
 116:           uint32_t a = (threadIdx.x / s) * (2 * s) + s;
 117:           uint32_t ti = a + (threadIdx.x % s);
 118:           uint32_t si = a - 1;
 119:           binary_op_update(row_buf[si], row_buf[ti], row_idx_buf[si], row_idx_buf[ti], binary_op);
 120:         }
 121:         __syncthreads();
 122:       }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 124-124
```cpp
 124:       // Write back to output.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 125-138
```cpp
 125:       if (row_exists) {
 126:         if (col1 < row_size){
 127:           row_values[col1] = row_buf[threadIdx.x];
 128:           row_indices[col1] = row_idx_buf[threadIdx.x];
 129:         }
 130:         if (col2 < row_size) {
 131:           row_values[col2] = row_buf[num_threads_x + threadIdx.x];
 132:           row_indices[col2] = row_idx_buf[num_threads_x + threadIdx.x];
 133:         }
 134:       }
 135:       block_total = row_buf[2 * num_threads_x - 1];
 136:       block_idx_final = row_idx_buf[2 * num_threads_x - 1];
 137:       __syncthreads();
 138:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 139-140
```cpp
 139:   }
 140: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 142-152
```cpp
 142: /* Perform an inclusive scan along an outer dimension of a tensor.
 143:  *
 144:  * - num_orows is the size of the flattened outer dimensions;
 145:  * - num_irows is the size of the flattened inner dimensions;
 146:  * - row_size is the size of the dimension along which to compute the variance;
 147:  *
 148:  * The dimensions to the outside and inside of the specified dimension are considered as flattened.
 149:  * Thread blocks with the same blockIdx.y process an "outer row" (i.e. an element of the flattened
 150:  * outer dimensions, which contains several "inner rows").
 151:  * Each thread processes a single inner row at a time.
 152:  */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 153-174
```cpp
 153: template<typename scalar_t, class BinaryFunction>
 154: __global__ void tensor_kernel_scan_outer_dim_with_indices(const scalar_t *self_, scalar_t *values_, int64_t *indices_,
 155:                   const uint32_t num_orows, const uint32_t num_irows, const uint32_t row_size, scalar_t init, BinaryFunction binary_op) {
 156:   for (uint32_t orow = blockIdx.x; orow < num_orows; orow += gridDim.x) {
 157:     for (uint32_t irow = blockIdx.y * blockDim.x + threadIdx.x; irow < num_irows; irow += gridDim.y * blockDim.x) {
 158:       const scalar_t *self = self_ + orow * row_size * num_irows + irow;
 159:       scalar_t *values = values_ + orow * row_size * num_irows + irow;
 160:       int64_t *indices = indices_ + orow * row_size * num_irows + irow;
 161:       scalar_t out = init;
 162:       int64_t out_idx = 0;
 163: 
 164:       for (auto col = decltype(row_size){0}; col < row_size; ++col) {
 165:         const auto val = c10::load(self);
 166:         if(at::_isnan(val) || (!at::_isnan(out) && binary_op(val, out))) {
 167:           out = val;
 168:           out_idx = col;
 169:         }
 170:         *values = out;
 171:         *indices = out_idx;
 172:         self += num_irows;
 173:         values += num_irows;
 174:         indices += num_irows;
```
- EN: This block defines GPU kernel entry point(s) `tensor_kernel_scan_outer_dim_with_indices`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `tensor_kernel_scan_outer_dim_with_indices`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 175-178
```cpp
 175:       }
 176:     }
 177:   }
 178: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 180-184
```cpp
 180: inline void check_fits_in_unsigned(int64_t val, const char* name) {
 181:   constexpr auto umax = std::numeric_limits<uint32_t>::max();
 182:   TORCH_CHECK(
 183:       val >= 0 && val <= umax, name, " must fit in a 32-bit uint32_t value");
 184: }
```
- EN: This block defines or continues the implementation of `check_fits_in_unsigned`.
- CN: 该代码块定义或继续实现 `check_fits_in_unsigned`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 187-208
```cpp
 187: template<typename scalar_t, class BinaryFunction>
 188: __host__ void scan_outer_dim_with_indices(
 189:     const TensorBase& self, const TensorBase& values, const TensorBase& indices,
 190:     int dim, scalar_t init, BinaryFunction binary_op) {
 191:   int64_t row_size = self.size(dim);
 192:   auto sizes = self.sizes();
 193: 
 194:   // Treat all outer dimensions (i.e. dim_ < dim) as one.
 195:   const int64_t num_orows = c10::multiply_integers(sizes.begin(), sizes.begin() + dim);
 196: 
 197:   // Treat all inner dimensions (i.e. dim > dimension) as one.
 198:   const int64_t num_irows = c10::multiply_integers(sizes.begin() + dim + 1, sizes.end());
 199:   //for performance reasons, cuda kernels use uint32_t for loops over irows, orows and row,
 200:   //make sure that input is not bigger than supported by uint32_t
 201:   check_fits_in_unsigned(num_irows, "num_irows");
 202:   check_fits_in_unsigned(num_orows, "num_orows");
 203:   check_fits_in_unsigned(row_size, "row_size");
 204: 
 205: 
 206:   dim3 threads(std::min(512, int(num_irows)));
 207:   int64_t maxGridDim = at::cuda::getCurrentDeviceProperties()->maxGridSize[1];
 208:   dim3 grid(std::min(maxGridDim, num_orows), std::min(maxGridDim, ceil_div(num_irows, int64_t{threads.x})));
```
- EN: This block defines or continues the implementation of `scan_outer_dim_with_indices`.
- CN: 该代码块定义或继续实现 `scan_outer_dim_with_indices`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 209-213
```cpp
 209:   tensor_kernel_scan_outer_dim_with_indices<scalar_t><<<grid, threads, 0, at::cuda::getCurrentCUDAStream()>>>(
 210:     self.const_data_ptr<scalar_t>(), values.mutable_data_ptr<scalar_t>(), indices.mutable_data_ptr<int64_t>(),
 211:     num_orows, num_irows, row_size, init, binary_op);
 212:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 213: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 215-236
```cpp
 215: template <typename scalar_t, class BinaryFunction>
 216: __host__ void scan_innermost_dim_with_indices(
 217:     const TensorBase& self, const TensorBase& values, const TensorBase& indices,
 218:     scalar_t init, BinaryFunction binary_op) {
 219:   int ndim = self.dim();
 220:   // Treat all outer dimensions as a single dimension.
 221:   int row_size = self.size(ndim - 1);
 222:   int num_rows = self.numel() / row_size;
 223: 
 224:   // assuming max_num_threads per block is 512
 225:   const uint32_t num_threads = 512;
 226:   const uint32_t log_num_threads_x = get_log_num_threads_x_inner_scan<uint32_t>(num_rows, row_size);
 227:   const uint32_t num_threads_x = (1 << log_num_threads_x);
 228:   const uint32_t num_threads_y = num_threads / num_threads_x;
 229:   dim3 threads(num_threads_x, num_threads_y);
 230:   dim3 grid(std::min(at::cuda::getCurrentDeviceProperties()->maxGridSize[0], ceil_div(num_rows, int(threads.y))));
 231: 
 232:   const uint32_t mem_size = 2 * num_threads * (sizeof(scalar_t) + sizeof(int64_t));
 233:   tensor_kernel_scan_innermost_dim_with_indices<scalar_t><<<grid, threads, mem_size,
 234:                                                             at::cuda::getCurrentCUDAStream()>>>(
 235:     self.const_data_ptr<scalar_t>(), values.mutable_data_ptr<scalar_t>(), indices.mutable_data_ptr<int64_t>(),
 236:     num_rows, row_size, num_threads, log_num_threads_x, init, binary_op);
```
- EN: This block defines or continues the implementation of `scan_innermost_dim_with_indices`.
- CN: 该代码块定义或继续实现 `scan_innermost_dim_with_indices`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 237-238
```cpp
 237:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 238: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 240-261
```cpp
 240: template<typename scalar_t, typename BinaryFunction>
 241: void scan_dim_with_indices(const TensorBase& self, const TensorBase& values, const TensorBase& indices, //int64_t dim) {
 242:      int64_t dim, scalar_t init, BinaryFunction binary_op) {
 243:   int ndim = self.dim();
 244:   auto self_ = self.expect_contiguous();
 245:   TORCH_INTERNAL_ASSERT(values.is_contiguous() && indices.is_contiguous());
 246:   if (dim == ndim - 1) {
 247:     scan_innermost_dim_with_indices<scalar_t>(*self_, values, indices, init, binary_op);
 248:   } else {
 249:     scan_outer_dim_with_indices<scalar_t>(*self_, values, indices, dim, init, binary_op);
 250:   }
 251: }
 252: 
 253: // TODO: The implementation of `tensor_kernel_scan_outer_dim` and
 254: // `tensor_kernel_scan_innermost_dim` is similar to
 255: // `tensor_kernel_scan_outer_dim_with_indices`
 256: // `tensor_kernel_scan_outer_dim_with_indices` and should be refactored to
 257: // remove the duplication.
 258: 
 259: /* Perform an inclusive scan along an outer dimension of a tensor.
 260:  *
 261:  * - num_orows is the size of the flattened outer dimensions;
```
- EN: This block defines or continues the implementation of `scan_dim_with_indices`.
- CN: 该代码块定义或继续实现 `scan_dim_with_indices`。

### Lines 262-269
```cpp
 262:  * - num_irows is the size of the flattened inner dimensions;
 263:  * - row_size is the size of the dimension along which to scan;
 264:  *
 265:  * The dimensions to the outside and inside of the specified dimension are considered as flattened.
 266:  * Thread blocks with the same blockIdx.y process an "outer row" (i.e. an element of the flattened
 267:  * outer dimensions, which contains several "inner rows").
 268:  * Each thread processes a single inner row at a time.
 269:  */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 270-290
```cpp
 270: template<typename scalar_t, typename index_t, class BinaryOp>
 271: __global__ void tensor_kernel_scan_outer_dim(scalar_t *tgt_, const scalar_t *src_,
 272:                                               const uint32_t num_orows, const uint32_t num_irows, const uint32_t row_size,
 273:                                               const scalar_t init, BinaryOp binary_op)
 274: {
 275:   for (uint32_t orow = blockIdx.x; orow < num_orows; orow += gridDim.x) {
 276:     for (uint32_t irow = blockIdx.y * blockDim.x + threadIdx.x; irow < num_irows; irow += gridDim.y * blockDim.x) {
 277:       const scalar_t *src = src_ + static_cast<index_t>(orow) * row_size * num_irows + irow;
 278:       scalar_t *tgt = tgt_ + (index_t) orow * row_size * num_irows + irow;
 279:       scalar_t acc = init;
 280: 
 281:       for (uint32_t col = 0; col < row_size; ++col) {
 282:         acc = binary_op(acc, c10::load(src));
 283:         *tgt = acc;
 284: 
 285:         src += num_irows;
 286:         tgt += num_irows;
 287:       }
 288:     }
 289:   }
 290: }
```
- EN: This block defines GPU kernel entry point(s) `tensor_kernel_scan_outer_dim`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `tensor_kernel_scan_outer_dim`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 292-301
```cpp
 292: /* Perform an inclusive scan along the innermost dimension of a tensor.
 293:  *
 294:  * - num_rows is the size of the flattened outer dimensions;
 295:  * - row_size is the size of the innermost dimension;
 296:  *
 297:  * The outer dimensions of the tensor are considered as a single dimension, i.e. the tensor is
 298:  * considered as having 'num_rows' rows of size 'row_size'.
 299:  * Each thread block processes one or more sets of contiguous rows (processing multiple rows
 300:  * per thread block is quicker than processing a single row, especially for short rows).
 301:  */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 302-323
```cpp
 302: template<typename T, typename index_t, class BinaryFunction>
 303: __device__ void tensor_kernel_scan_innermost_dim_impl(T* row_buf, T *tgt_, const T *src_,
 304:                                       const uint32_t num_rows, const uint32_t row_size,
 305:                                       const uint32_t log_num_threads_x,
 306:                                       T init, BinaryFunction binary_op){
 307:   const index_t num_threads_x = 1 << log_num_threads_x;
 308:   for (index_t block_row = blockIdx.x * (index_t) blockDim.y;
 309:        block_row < num_rows;
 310:        block_row += blockDim.y * gridDim.x) {
 311:     index_t row = block_row + (index_t) threadIdx.y;
 312:     T block_total = init;
 313: 
 314:     const T *row_src = src_ + row * row_size;
 315:     T *row_tgt = tgt_ + row * row_size;
 316:     const bool row_exists = row < num_rows;
 317: 
 318:     // Perform scan on one block at a time, keeping track of the total value of
 319:     // all blocks processed so far.
 320:     for (index_t block_col = 0; block_col < row_size; block_col += 2 * num_threads_x) {
 321:       // Load data into shared memory (two values per thread).
 322:       index_t col1 = block_col + (index_t) threadIdx.x;
 323:       index_t col2 = block_col + num_threads_x + (index_t) threadIdx.x;
```
- EN: This block defines or continues the implementation of `tensor_kernel_scan_innermost_dim_impl`.
- CN: 该代码块定义或继续实现 `tensor_kernel_scan_innermost_dim_impl`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 324-341
```cpp
 324:       if (row_exists) {
 325:         if (col1 < row_size) {
 326:           row_buf[threadIdx.x] = row_src[col1];
 327:         } else {
 328:           row_buf[threadIdx.x] = init;
 329:         }
 330: 
 331:         if (col2 < row_size) {
 332:           row_buf[num_threads_x + threadIdx.x] = row_src[col2];
 333:         } else {
 334:           row_buf[num_threads_x + threadIdx.x] = init;
 335:         }
 336: 
 337:         // Add the total value of all previous blocks to the first value of this block.
 338:         if (threadIdx.x == 0) {
 339:           row_buf[0] = binary_op(row_buf[0], block_total);
 340:         }
 341:       }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 342-342
```cpp
 342:       __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 344-345
```cpp
 344:       // Parallel reduction with Sklansky method. The diagram can be seen on this paper:
 345:       // https://research.nvidia.com/publication/single-pass-parallel-prefix-scan-decoupled-look-back
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 346-355
```cpp
 346:       for (int m = 0; m <= log_num_threads_x; ++m) {
 347:         if (row_exists) {
 348:           index_t s = 1 << m; // s = 2 ^ m
 349:           auto a = static_cast<index_t>((threadIdx.x >> m) << (m + 1)) | s; // a = (threadIdx.x / s) * (2 * s) + s
 350:           index_t ti = a + (threadIdx.x % s);
 351:           index_t si = a - 1;
 352:           row_buf[ti] = binary_op(row_buf[ti], row_buf[si]);
 353:         }
 354:         __syncthreads();
 355:       }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 357-357
```cpp
 357:       // Write back to output.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 358-366
```cpp
 358:       if (row_exists) {
 359:         if (col1 < row_size) row_tgt[col1] = row_buf[threadIdx.x];
 360:         if (col2 < row_size) row_tgt[col2] = row_buf[num_threads_x + threadIdx.x];
 361:       }
 362:       block_total = row_buf[2 * num_threads_x - 1];
 363:       __syncthreads();
 364:     }
 365:   }
 366: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 368-389
```cpp
 368: template <
 369:     typename T,
 370:     class BinaryFunction>
 371: __global__ void tensor_kernel_scan_innermost_dim(
 372:     T* tgt_,
 373:     const T* src_,
 374:     const uint32_t num_rows,
 375:     const uint32_t row_size,
 376:     const uint32_t log_num_threads_x,
 377:     T init,
 378:     BinaryFunction binary_op) {
 379:   alignas(sizeof(double)) extern __shared__ char sbuf[];
 380:   T* sbuf2 = reinterpret_cast<T*>(sbuf);
 381:   const uint32_t num_threads_x = 1 << log_num_threads_x;
 382:   T* row_buf = reinterpret_cast<T*>(sbuf2 + num_threads_x * 2 * threadIdx.y);
 383:   if (num_rows * (size_t) row_size <= UINT_MAX) {
 384:       tensor_kernel_scan_innermost_dim_impl<T, uint32_t>(
 385:           row_buf, tgt_, src_, num_rows, row_size, log_num_threads_x, init, binary_op);
 386:   } else {
 387:       tensor_kernel_scan_innermost_dim_impl<T, size_t>(
 388:           row_buf, tgt_, src_, num_rows, row_size, log_num_threads_x, init, binary_op);
 389:   }
```
- EN: This block defines GPU kernel entry point(s) `tensor_kernel_scan_innermost_dim`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `tensor_kernel_scan_innermost_dim`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 390-390
```cpp
 390: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 393-414
```cpp
 393: template<typename scalar_t, class BinaryFunction>
 394: __host__ void scan_outer_dim(const TensorBase& self, const TensorBase& result,
 395:                              int dim, scalar_t init, BinaryFunction binary_op) {
 396:   const int64_t row_size = self.size(dim);
 397:   auto sizes = self.sizes();
 398: 
 399:   // Treat all outer dimensions (i.e. dim_ < dim) as one.
 400:   const int64_t num_orows = c10::multiply_integers(sizes.begin(), sizes.begin() + dim);
 401: 
 402:   // Treat all inner dimensions (i.e. dim > dimension) as one.
 403:   const int64_t num_irows = c10::multiply_integers(sizes.begin() + dim + 1, sizes.end());
 404: 
 405:   dim3 threads(std::min(512, int(num_irows)));
 406:   int64_t maxGridDim = at::cuda::getCurrentDeviceProperties()->maxGridSize[1];
 407:   dim3 grid(std::min(maxGridDim, num_orows), std::min(maxGridDim, ceil_div(num_irows, int64_t{threads.x})));
 408: 
 409:   check_fits_in_unsigned(num_irows, "num_irows");
 410:   check_fits_in_unsigned(num_orows, "num_orows");
 411:   check_fits_in_unsigned(row_size, "row_size");
 412:   if (static_cast<size_t>(num_irows) * num_orows * row_size <= UINT_MAX) {
 413:   tensor_kernel_scan_outer_dim<scalar_t, uint32_t><<<grid, threads, 0, at::cuda::getCurrentCUDAStream()>>>(
 414:     result.mutable_data_ptr<scalar_t>(), self.const_data_ptr<scalar_t>(),
```
- EN: This block defines or continues the implementation of `scan_outer_dim`.
- CN: 该代码块定义或继续实现 `scan_outer_dim`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 415-422
```cpp
 415:     num_orows, num_irows, row_size, init, binary_op);
 416:   } else  {
 417:   tensor_kernel_scan_outer_dim<scalar_t, size_t><<<grid, threads, 0, at::cuda::getCurrentCUDAStream()>>>(
 418:     result.mutable_data_ptr<scalar_t>(), self.const_data_ptr<scalar_t>(),
 419:     num_orows, num_irows, row_size, init, binary_op);
 420:   }
 421:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 422: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 424-445
```cpp
 424: template <typename scalar_t, class BinaryFunction>
 425: void scan_innermost_dim(const TensorBase& self, const TensorBase& result,
 426:                         scalar_t init, BinaryFunction binary_op) {
 427:   int64_t ndim = self.dim();
 428:   // Treat all outer dimensions as a single dimension.
 429:   int64_t row_size = self.size(ndim - 1);
 430:   int64_t num_rows = self.numel() / row_size;
 431: 
 432:   // assuming max_num_threads per block is 512
 433:   const uint32_t num_threads = 512;
 434:   const uint32_t log_num_threads_x = get_log_num_threads_x_inner_scan<uint32_t>(num_rows, row_size);
 435:   const uint32_t num_threads_x = (1 << log_num_threads_x);
 436:   const uint32_t num_threads_y = num_threads / num_threads_x;
 437:   dim3 threads(num_threads_x, num_threads_y);
 438:   int64_t maxGridDim = at::cuda::getCurrentDeviceProperties()->maxGridSize[0];
 439:   dim3 grid(std::min(maxGridDim, ceil_div(num_rows, int64_t{threads.y})));
 440: 
 441:   check_fits_in_unsigned(num_rows, "Number of rows (self.numel()/self.size(self.dim()-1))");
 442:   check_fits_in_unsigned(row_size, "row_size");
 443: 
 444:   tensor_kernel_scan_innermost_dim<scalar_t><<<grid, threads, num_threads * 2 * sizeof(scalar_t),
 445:                                                at::cuda::getCurrentCUDAStream()>>>(
```
- EN: This block defines or continues the implementation of `scan_innermost_dim`.
- CN: 该代码块定义或继续实现 `scan_innermost_dim`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 446-449
```cpp
 446:     result.mutable_data_ptr<scalar_t>(), self.const_data_ptr<scalar_t>(),
 447:     num_rows, row_size, log_num_threads_x, init, binary_op);
 448:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 449: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 451-472
```cpp
 451: template<typename scalar_t, typename BinaryFunction>
 452: void scan_dim(const TensorBase& self, const TensorBase& result,
 453:      int64_t dim, scalar_t init, BinaryFunction binary_op) {
 454:   int ndim = self.dim();
 455:   auto self_ = self.expect_contiguous();
 456:   TORCH_INTERNAL_ASSERT(result.is_contiguous());
 457: 
 458:   if (self.numel() == self.size(dim)) {
 459:     if constexpr (std::is_same_v<BinaryFunction, std::plus<scalar_t>>) {
 460:       if (C10_UNLIKELY(at::globalContext().deterministicAlgorithms()) && (self.is_floating_point() || self.is_complex())) {
 461:         cuda::cub::inclusive_deterministic_scan(self_->const_data_ptr<scalar_t>(), result.mutable_data_ptr<scalar_t>(), binary_op, self.numel());
 462:       } else {
 463:         cuda::cub::inclusive_scan(self_->const_data_ptr<scalar_t>(), result.mutable_data_ptr<scalar_t>(), binary_op, self.numel());
 464:       }
 465:     } else {
 466:       cuda::cub::inclusive_scan(self_->const_data_ptr<scalar_t>(), result.mutable_data_ptr<scalar_t>(), binary_op, self.numel());
 467:     }
 468:   } else if (dim == ndim - 1) {
 469:     scan_innermost_dim<scalar_t>(*self_, result, init, binary_op);
 470:   } else {
 471:     scan_outer_dim<scalar_t>(*self_, result, dim, init, binary_op);
 472:   }
```
- EN: This block defines or continues the implementation of `scan_dim`, `constexpr`.
- CN: 该代码块定义或继续实现 `scan_dim`, `constexpr`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 473-473
```cpp
 473: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 475-475
```cpp
 475: }  // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- CUB primitives provide parallel CUDA building blocks such as reductions or scans. / CUB 原语提供归约、扫描等并行 CUDA 基础组件。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/NumericUtils.h>`
  - `<ATen/core/TensorBase.h>`
  - `<ATen/cuda/cub.cuh>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<c10/util/Load.h>`
  - `<limits>`
  - `<cmath>`
- Runtime symbols / 运行时符号:
  - `cub::inclusive_deterministic_scan`
  - `cub::inclusive_scan`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
