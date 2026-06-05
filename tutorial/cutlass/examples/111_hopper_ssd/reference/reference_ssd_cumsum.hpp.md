# reference_ssd_cumsum.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/111_hopper_ssd/reference/reference_ssd_cumsum.hpp`  
**Purpose / 用途**: Reference preprocessing kernel for SSD that computes the cumulative DeltaA forms needed by the chunked recurrence. / SSD 的参考预处理内核：计算分块递推所需的 DeltaA 累积形式。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-54 — Header setup and namespace / 头文件设置与命名空间
```cpp
/***************************************************************************************************
 * Copyright (c) 2025 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

#pragma once

#include <algorithm>
#include <random>

#include "cutlass/coord.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/tensor_view.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/host/gemm.h"
#include "cutlass/arch/arch.h"
#include "cutlass/layout/matrix.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/cuda_host_adapter.hpp"

#include "cute/int_tuple.hpp"
#include "cute/atom/mma_traits_sm100.hpp"
#include "cute/util/debug.hpp"
#include "cute/config.hpp"

namespace cutlass::ssd::kernel {

using namespace cute;
```
**EN**: The file is organized like a small CUTLASS kernel definition: it includes architecture, tensor, and adapter headers, then opens `cutlass::ssd::kernel`. Even though many includes are generic, the actual purpose here is narrow and specific—prepare prefix sums for SSD state scaling.
**CN**: 这个文件按一个小型 CUTLASS 内核定义来组织：先引入架构、张量和适配器相关头文件，再进入 `cutlass::ssd::kernel` 命名空间。虽然不少头文件比较通用，但这里的实际任务很专一：为 SSD 的状态缩放准备前缀和。

### Lines 55-77 — Kernel traits and storage contract / 内核 traits 与存储约定
```cpp

template<
  class Element_,
  class ElementD_,
  class TileShape_>
struct CumsumKernel {
  using Element = Element_;
  using ElementD = ElementD_;
  using TileShape = TileShape_; // L,D,N

  // Required by `device_kernel`
  static constexpr int MaxThreadsPerBlock = 128;
  static constexpr int MinBlocksPerMultiprocessor = 1;
  using ArchTag = arch::Sm90;

  static constexpr int AlignmentBytes = 16;

  struct SharedStorage {
    /* empty, no smem needed */
  };

  static constexpr int SharedStorageSize = sizeof(SharedStorage);

```
**EN**: `CumsumKernel` declares the element types, the logical tile shape `(L, D, N)`, and the runtime traits expected by CUTLASS device kernels: architecture tag, thread count, and shared-storage size. Shared memory is empty here because the computation is a straightforward per-slice prefix scan done directly from global memory.
**CN**: `CumsumKernel` 声明了元素类型、逻辑 tile 形状 `(L, D, N)`，以及 CUTLASS 设备内核所需的运行 traits：架构标签、线程数和共享存储大小。这里的共享内存是空的，因为该计算只是针对每个切片执行直接的前缀扫描，不需要显式共享内存缓存。

### Lines 78-123 — Arguments, params, and workspace hooks / 参数、底层参数与工作区接口
```cpp
  struct TransformArguments {
    const Element* ptr_DeltaA;
    ElementD* ptr_Cumsum;
  };

  struct TransformParams {
    const Element* ptr_DeltaA;
    ElementD* ptr_Cumsum;
  };
  
  using ProblemShape = cute::tuple<int, int, int, int>; // b, eh, c, l
  struct Arguments {
    ProblemShape problem_shape{};
    TransformArguments transform{};
    KernelHardwareInfo hw_info{};
  };

  struct Params {
    ProblemShape problem_shape{};
    TransformParams transform{};
    KernelHardwareInfo hw_info{};
  };

  static Params
  to_underlying_arguments(Arguments const& args, void* workspace) {
    return Params{
      ProblemShape{args.problem_shape},
      TransformParams{args.transform.ptr_DeltaA, args.transform.ptr_Cumsum}, 
      KernelHardwareInfo{args.hw_info}};
  }

  static Status
  can_implement(Arguments const& args) {
    return Status::kSuccess;
  }

  static size_t
  get_workspace_size(Arguments const& args) {
    return size_t(0);
  }

  static Status
  initialize_workspace(Arguments const& args, void* workspace = nullptr, cudaStream_t stream = nullptr,
    CudaHostAdapter *cuda_adapter = nullptr) {
    return Status::kSuccess;
  }
```
**EN**: The `Arguments` and `Params` structs carry the problem shape `(B, EH, C, L)`, the input pointer `ptr_DeltaA`, the output pointer `ptr_Cumsum`, and hardware info. `to_underlying_arguments()`, `can_implement()`, `get_workspace_size()`, and `initialize_workspace()` provide the standard CUTLASS adapter hooks; this kernel needs no extra workspace and performs no elaborate validation.
**CN**: `Arguments` 和 `Params` 负责传递问题规模 `(B, EH, C, L)`、输入指针 `ptr_DeltaA`、输出指针 `ptr_Cumsum` 以及硬件信息。`to_underlying_arguments()`、`can_implement()`、`get_workspace_size()` 和 `initialize_workspace()` 则提供标准的 CUTLASS 适配接口；这个内核不需要额外工作区，也没有复杂的合法性检查。

### Lines 124-135 — Grid and block geometry / 网格与线程块配置
```cpp

  static dim3
  get_grid_shape(Params const& params) {
    auto [B, EH, C, L] = params.problem_shape;
    return dim3(B*EH, 1, 1);
  }

  static dim3
  get_block_shape() {
    return dim3(MaxThreadsPerBlock, 1, 1);
  }

```
**EN**: `get_grid_shape()` launches one block for each `(B, EH)` pair, and `get_block_shape()` fixes the block size at 128 threads. That launch geometry matches the intended decomposition: different heads and batches are independent, while the per-slice `C × L` prefix work is handled cooperatively inside one block.
**CN**: `get_grid_shape()` 为每个 `(B, EH)` 对启动一个线程块，`get_block_shape()` 则把块大小固定为 128 线程。这种网格划分正对应本问题的独立性结构：不同 batch 与 head 之间互不依赖，而单个切片上的 `C × L` 前缀和工作则在一个线程块内协同完成。

### Lines 136-157 — Operator setup, tensor views, and packed types / 核函数主体的设置、张量视图与打包类型
```cpp
  CUTE_HOST_DEVICE
  void
  operator()(Params params, [[maybe_unused]] char* smem_buf = nullptr) {
    auto [B, EH, C, L] = params.problem_shape;

    auto layout = make_layout(make_shape(L, C, EH*B));

    auto mD_bcl = make_tensor(make_gmem_ptr(params.transform.ptr_DeltaA), make_layout(reverse(layout.shape()), reverse(layout.stride())));
    auto mC_bcl = make_tensor(make_gmem_ptr(params.transform.ptr_Cumsum), make_layout(reverse(layout.shape()), reverse(layout.stride())));
    auto cD_bcl = make_identity_tensor(shape(mD_bcl));

    int blk_idx = blockIdx.x;
    int thread_idx = threadIdx.x;

    auto tD = logical_divide(mD_bcl(blk_idx,_,_), make_shape(Int<128>{},_))(make_coord(thread_idx,_),_);
    auto tC = logical_divide(mC_bcl(blk_idx,_,_), make_shape(Int<128>{},_))(make_coord(thread_idx,_),_);
    auto cD = logical_divide(cD_bcl(blk_idx,_,_), make_shape(Int<128>{},_))(make_coord(thread_idx,_),_);

    static constexpr int NumPacked = AlignmentBytes / sizeof(ElementD);
    using PackedTypeDeltaA = uint_bit_t<sizeof_bits_v<Element> * NumPacked>;
    using PackedTypeCumsum = uint_bit_t<sizeof_bits_v<ElementD> * NumPacked>;

```
**EN**: Inside `operator()`, the kernel builds a logical `(EH*B, C, L)` tensor view for both the input and output, then extracts the slice selected by `blockIdx.x`. `logical_divide()` maps that slice onto thread-local views, and the packed integer aliases (`PackedTypeDeltaA`, `PackedTypeCumsum`) let the code move aligned 16-byte chunks at a time.
**CN**: 在 `operator()` 内部，内核先为输入和输出构造逻辑上的 `(EH*B, C, L)` 张量视图，再根据 `blockIdx.x` 取出当前切片。`logical_divide()` 把这个切片映射成线程局部视图，而 `PackedTypeDeltaA` 与 `PackedTypeCumsum` 这样的打包整数别名则允许代码按对齐的 16 字节块进行读写。

### Lines 158-164 — Optional debug instrumentation / 可选调试输出
```cpp
#if 0
    if (thread_idx % 128 == 0 && blk_idx == 0) {
      print("tD  : ");print(tD);print("\n");
      print("tC  : ");print(tC);print("\n");
      print("cD  : ");print(cD);print("\n");
    }
#endif
```
**EN**: The `#if 0` region contains debugging prints for the per-thread views. It is disabled in normal builds, but it shows the author was checking how CuTe partitioned the `(C, L)` work across the block.
**CN**: `#if 0` 代码块里是针对线程局部视图的调试打印。正常编译时它被关闭，但能看出作者曾用它来检查 CuTe 如何把 `(C, L)` 上的工作划分到一个线程块中的不同线程。

### Lines 165-194 — Packed prefix-scan loop / 打包前缀扫描循环
```cpp

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < shape<0>(tD); ++i) {
      float last_element = 0.f;
      auto crd = cD(i,_0{});
      auto tD_recast = recast<PackedTypeDeltaA>(tD);
      auto tC_recast = recast<PackedTypeCumsum>(tC);
      if (elem_less(crd, shape(mD_bcl))) {
        for (int j = 0; j < shape<1>(tD_recast); ++j) {
          auto tD_slice = make_tensor<Element>(make_shape(Int<NumPacked>{}));
          auto tC_slice = make_tensor<ElementD>(make_shape(Int<NumPacked>{}));
          auto tD_slice_recast = recast<PackedTypeDeltaA>(tD_slice);
          auto tC_slice_recast = recast<PackedTypeCumsum>(tC_slice);

          tD_slice_recast(_0{}) = tD_recast(i,j);
          for (int k = 0; k < NumPacked; ++ k) {
            last_element += static_cast<float>(tD_slice(k));
            tC_slice(k) = static_cast<ElementD>(last_element);
          }
          tC_recast(i,j) = tC_slice_recast(_0{});
        }
      }
    }
  }

private:

};

} // End namespace cutlass
```
**EN**: The outer loop walks the independent rows, while `last_element` carries the running prefix total along the sequence direction. Each packed load is unpacked to scalar elements, accumulated as `last_element += DeltaA`, converted to `ElementD`, and packed back into the output tensor.

The result is a raw cumsum, not an exponentiated quantity. Later SSD code turns these prefix sums into factors like `exp(prefix_i - prefix_j)`, which is exactly the semiseparable trick used to express chunk-local and inter-chunk state transitions efficiently.
**CN**: 外层循环遍历彼此独立的行，而 `last_element` 则沿着序列方向保存当前的前缀累计值。每次打包读取后，代码先把元素拆成标量，再按 `last_element += DeltaA` 累加，转换成 `ElementD`，最后重新打包写回输出张量。

这里得到的是“原始前缀和”，而不是已经取指数的量。后续 SSD 代码会把这些前缀和值转成 `exp(prefix_i - prefix_j)` 这类因子，这正是半可分表示中高效表达 chunk 内和 chunk 间状态转移的关键技巧。

---

## Key Concepts / 关键概念

- Per-slice prefix scan over `DeltaA` / 针对 `DeltaA` 切片的前缀扫描
- One CUDA block per `(batch, expanded-head)` pair / 每个 `(batch, expanded-head)` 对应一个 CUDA 线程块
- 16-byte packed memory movement / 16 字节打包访存
- Using prefix differences to recover exponential decay / 用前缀差恢复指数衰减因子

## Dependencies / 依赖项

- `cute/int_tuple.hpp` — integer tuple helpers used by CuTe layouts / CuTe 布局使用的整型元组工具
- `cutlass/cuda_host_adapter.hpp` — adapter hook type for CUTLASS initialization / CUTLASS 初始化用的适配器类型
- `cutlass/arch/arch.h` — declares `arch::Sm90` and architecture tags / 声明 `arch::Sm90` 及相关架构标签
- `cute/util/debug.hpp` — optional debug printing for tensor partitions / 张量分块的可选调试输出
