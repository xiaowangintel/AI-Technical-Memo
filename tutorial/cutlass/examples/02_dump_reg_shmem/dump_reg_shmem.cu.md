# dump_reg_shmem.cu — Code Analysis / 代码分析
**Source / 源文件**: `examples/02_dump_reg_shmem/dump_reg_shmem.cu`
**Purpose / 用途**: Shows how to dump per-thread fragments and shared memory from CUTLASS iterators / 展示如何从 CUTLASS 迭代器转储每线程 fragment 和共享内存
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-35 / 第 1-35 行
```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/

/*! \file
  \brief Demonstrate CUTLASS debugging tool for dumping fragments and shared
  memory
 */
```
**EN**: License and file comment identify a debugging example for fragment and shared-memory dumps.
**CN**: 许可证和文件说明表明这是用于 fragment 与共享内存转储的调试示例。

### Lines 37-69 / 第 37-69 行
```cpp
///////////////////////////////////////////////////////////////////////////////////////////////////

// Standard Library includes

#include <iostream>

//
// CUTLASS includes
//

#include "cutlass/aligned_buffer.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/layout/matrix.h"
#include "cutlass/matrix_shape.h"
#include "cutlass/numeric_types.h"

#include "cutlass/core_io.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/tensor_view_io.h"

#include "cutlass/util/reference/host/gemm.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_fill.h"

#include "cutlass/transform/pitch_linear_thread_map.h"
#include "cutlass/transform/threadblock/predicated_tile_iterator.h"
#include "cutlass/transform/threadblock/regular_tile_iterator_tensor_op.h"

#include "cutlass/util/debug.h"
#include "cutlass/util/device_dump.h"

#define EXAMPLE_MATRIX_ROW 64
#define EXAMPLE_MATRIX_COL 32
```
**EN**: Includes bring in matrix shapes, half type, host tensors, host fill/I/O, thread maps, global/shared tile iterators, and debug dump utilities. The macros fix the example tile at 64 rows by 32 columns.
**CN**: 包含矩阵形状、half 类型、HostTensor、主机填充/I/O、线程映射、全局/共享 tile 迭代器和调试转储工具。宏把示例 tile 固定为 64 行 x 32 列。

### Lines 71-89 / 第 71-89 行
```cpp
///////////////////////////////////////////////////////////////////////////////////////////////////

template <typename Element, typename GmemIterator, typename SmemIterator>
__global__ void kernel_dump(typename GmemIterator::Params params,
                            typename GmemIterator::TensorRef ref) {
  extern __shared__ Element shared_storage[];

  // Construct the global iterator and load the data to the fragments.
  int tb_thread_id = threadIdx.y * blockDim.x + threadIdx.x;

  GmemIterator gmem_iterator(params, ref.data(),
                             {EXAMPLE_MATRIX_ROW, EXAMPLE_MATRIX_COL},
                             tb_thread_id);

  typename GmemIterator::Fragment frag;

  frag.clear();
  gmem_iterator.load(frag);

```
**EN**: `kernel_dump` is templated on element type plus global and shared iterators. It allocates dynamic shared memory, computes a linear thread ID, constructs a predicated global-memory iterator over the 64x32 extent, clears a fragment, and loads data into that register fragment.
**CN**: `kernel_dump` 以元素类型、全局迭代器和共享迭代器为模板参数。它声明动态共享内存，计算线性线程 ID，构造覆盖 64x32 范围的 predicated 全局内存迭代器，清空 fragment，并把数据加载到寄存器 fragment。

### Lines 90-105 / 第 90-105 行
```cpp
  // Call dump_fragment() with different parameters.
  if (threadIdx.x == 0 && blockIdx.x == 0)
    printf("\nAll threads dump all the elements:\n");
  cutlass::debug::dump_fragment(frag);

  if (threadIdx.x == 0 && blockIdx.x == 0)
    printf("\nFirst thread dumps all the elements:\n");
  cutlass::debug::dump_fragment(frag, /*N = */ 1);

  if (threadIdx.x == 0 && blockIdx.x == 0)
    printf("\nFirst thread dumps first 16 elements:\n");
  cutlass::debug::dump_fragment(frag, /*N = */ 1, /*M = */ 16);

  if (threadIdx.x == 0 && blockIdx.x == 0)
    printf("\nFirst thread dumps first 16 elements with a stride of 8:\n");
  cutlass::debug::dump_fragment(frag, /*N = */ 1, /*M = */ 16, /*S = */ 8);
```
**EN**: `dump_fragment` is called with progressively narrower scopes: all threads/all elements, first thread only, first thread first 16 elements, and the same with stride 8. This demonstrates the optional N/M/S controls.
**CN**: `dump_fragment` 以逐渐收窄的范围调用：所有线程/所有元素、仅第一个线程、首线程前 16 个元素、以及步长 8 的前 16 个元素，展示可选 N/M/S 控制。

### Lines 107-125 / 第 107-125 行
```cpp
  // Construct the shared iterator and store the data to the shared memory.
  SmemIterator smem_iterator(
      typename SmemIterator::TensorRef(
          {shared_storage, SmemIterator::Layout::packed(
                               {EXAMPLE_MATRIX_ROW, EXAMPLE_MATRIX_COL})}),
      tb_thread_id);

  smem_iterator.store(frag);

  // Call dump_shmem() with different parameters.
  if (threadIdx.x == 0 && blockIdx.x == 0) printf("\nDump all the elements:\n");
  cutlass::debug::dump_shmem(shared_storage,
                             EXAMPLE_MATRIX_ROW * EXAMPLE_MATRIX_COL);

  if (threadIdx.x == 0 && blockIdx.x == 0)
    printf("\nDump all the elements with a stride of 8:\n");
  cutlass::debug::dump_shmem(
      shared_storage, EXAMPLE_MATRIX_ROW * EXAMPLE_MATRIX_COL, /*S = */ 8);
}
```
**EN**: A `RegularTileIterator` stores the fragment to shared memory using a packed tensor-op multiplicand layout. `dump_shmem` then prints the whole shared-memory buffer and a stride-8 view.
**CN**: `RegularTileIterator` 使用 packed 的 tensor-op multiplicand 布局把 fragment 存入共享内存。随后 `dump_shmem` 打印整个共享内存缓冲区及步长 8 视图。

### Lines 127-149 / 第 127-149 行
```cpp
///////////////////////////////////////////////////////////////////////////////////////////////////

/// Entry point for dump_reg_shmem example.
//
// usage:
//
//   02_dump_reg_shmem
//
int main() {
  // Initialize a 64x32 column major matrix with sequential data (1,2,3...).
  using Element = cutlass::half_t;
  using Layout = cutlass::layout::ColumnMajor;

  cutlass::HostTensor<Element, Layout> matrix(
      {EXAMPLE_MATRIX_ROW, EXAMPLE_MATRIX_COL});
  cutlass::reference::host::BlockFillSequential(matrix.host_data(),
                                                matrix.capacity());

  // Dump the matrix.
  std::cout << "Matrix:\n" << matrix.host_view() << "\n";

  // Copy the matrix to the device.
  matrix.sync_device();
```
**EN**: `main` selects `cutlass::half_t` with column-major layout, allocates a 64x32 `HostTensor`, fills it sequentially, prints the host matrix, and synchronizes it to device memory.
**CN**: `main` 选择 `cutlass::half_t` 和列主序布局，分配 64x32 `HostTensor`，顺序填充，打印主机矩阵，并同步到设备内存。

### Lines 151-166 / 第 151-166 行
```cpp
  // Define a global iterator, a shared iterator and their thread map.
  using ThreadMap = cutlass::transform::PitchLinearWarpRakedThreadMap<
      cutlass::layout::PitchLinearShape<EXAMPLE_MATRIX_ROW, EXAMPLE_MATRIX_COL>,
      32, cutlass::layout::PitchLinearShape<8, 4>, 8>;

  using GmemIterator =
      cutlass::transform::threadblock::PredicatedTileIterator<
          cutlass::MatrixShape<EXAMPLE_MATRIX_ROW, EXAMPLE_MATRIX_COL>, Element,
          Layout, 1, ThreadMap>;

  typename GmemIterator::Params params(matrix.layout());

  using SmemIterator = cutlass::transform::threadblock::RegularTileIterator<
      cutlass::MatrixShape<EXAMPLE_MATRIX_ROW, EXAMPLE_MATRIX_COL>, Element,
      cutlass::layout::ColumnMajorTensorOpMultiplicandCongruous<16, 64>, 1,
      ThreadMap>;
```
**EN**: `PitchLinearWarpRakedThreadMap<Shape,32,Delta,8>` maps 32 threads across the tile in a warp-raked pattern. `PredicatedTileIterator<MatrixShape,Element,Layout,1,ThreadMap>` performs guarded global loads. `RegularTileIterator<MatrixShape,Element,ColumnMajorTensorOpMultiplicandCongruous<16,64>,1,ThreadMap>` writes to a tensor-core-friendly shared layout; `16` is element bit width and `64` is the crosswise/congruous tile parameter.
**CN**: `PitchLinearWarpRakedThreadMap<Shape,32,Delta,8>` 以 warp-raked 模式把 32 个线程映射到 tile。`PredicatedTileIterator<MatrixShape,Element,Layout,1,ThreadMap>` 做带边界保护的全局加载。`RegularTileIterator<MatrixShape,Element,ColumnMajorTensorOpMultiplicandCongruous<16,64>,1,ThreadMap>` 写入适合 Tensor Core 的共享内存布局；`16` 为元素位宽，`64` 为 crosswise/congruous tile 参数。

### Lines 168-186 / 第 168-186 行
```cpp
  dim3 grid(1, 1);
  dim3 block(32, 1, 1);

  int smem_size =
      int(sizeof(Element) * EXAMPLE_MATRIX_ROW * EXAMPLE_MATRIX_COL);

  kernel_dump<Element, GmemIterator, SmemIterator>
      <<<grid, block, smem_size, 0>>>(params, matrix.device_ref());

  cudaError_t result = cudaDeviceSynchronize();

  if (result != cudaSuccess) {
    std::cout << "Failed" << std::endl;
  }

  return (result == cudaSuccess ? 0 : -1);
}

///////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The kernel launches one block of 32 threads with enough dynamic shared memory for all 64x32 half elements, synchronizes the device, and reports success or failure.
**CN**: 内核以一个 32 线程块启动，并分配容纳全部 64x32 half 元素的动态共享内存，随后同步设备并报告成败。

---
## Key Concepts / 关键概念
- CUTLASS fragments are per-thread register arrays owned by iterators and MMA pipelines. / CUTLASS fragment 是每线程拥有的寄存器数组，由迭代器和 MMA 流水线使用。
- Predicated iterators avoid out-of-bounds accesses by guarding logical tile coordinates. / Predicated 迭代器通过保护逻辑 tile 坐标避免越界访问。
- TensorOp shared layouts permute addresses to match `mma` access patterns and reduce bank conflicts. / TensorOp 共享内存布局会排列地址以匹配 `mma` 访问模式并降低 bank 冲突。

## Dependencies / 依赖项
- `cutlass/transform/threadblock/predicated_tile_iterator.h` — global-memory tile iterator with masks
- `cutlass/transform/threadblock/regular_tile_iterator_tensor_op.h` — shared-memory tensor-op tile iterator
- `cutlass/transform/pitch_linear_thread_map.h` — thread-to-tile mapping policies
- `cutlass/util/debug.h` — `dump_fragment`
- `cutlass/util/device_dump.h` — `dump_shmem`
- `cutlass/util/host_tensor.h` — host/device matrix storage
- `cutlass/layout/matrix.h` — matrix layouts including column-major
- `cutlass/numeric_types.h` — `half_t`
