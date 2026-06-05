# reference_ssd_cumsum.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/112_blackwell_ssd/reference/reference_ssd_cumsum.hpp`  
**Purpose / 用途**: Reference preprocessing kernel for Blackwell SSD that computes cumulative DeltaA terms used by the chunked recurrence. / Blackwell SSD 的参考预处理内核：计算分块递推所需的 DeltaA 累积项。

---

## Line-by-Line Analysis / 逐行分析

### 1. Lines 1-32 | Header boilerplate | 头文件样板

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
```

- **EN:** This range is the standard CUTLASS copyright block plus `#pragma once`. It is not algorithmic, but it clearly marks the file as a standalone reusable header.

- **CN:** 这一段是标准 CUTLASS 版权声明和 `#pragma once`。虽然不涉及算法，但它表明该文件是一个可复用的独立头文件。

### 2. Lines 33-60 | Includes, namespace, and kernel template declaration | 头文件依赖、命名空间与 kernel 模板声明

```cpp

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

template<
  class Element_,
  class ElementD_,
  class TileShape_>
struct CumsumKernel {
```

- **EN:** The code brings in host/device tensor helpers and opens `cutlass::ssd::kernel`, then starts the `CumsumKernel` template parameterized by input type, output type, and tile shape.

- **CN:** 代码引入张量辅助头文件并打开 `cutlass::ssd::kernel` 命名空间，然后声明 `CumsumKernel` 模板，模板参数包括输入类型、输出类型和 tile 形状。

### 3. Lines 61-99 | Kernel metadata and argument carriers | kernel 元数据与参数载体

```cpp
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
```

- **EN:** This part defines launch traits, empty shared storage, and the `Arguments`/`Params` structs that carry pointers, problem shape `(b, eh, c, l)`, and hardware info. It is the structural glue that lets the kernel plug into CUTLASS launch machinery.

- **CN:** 这一部分定义了启动特征、空共享存储以及 `Arguments`/`Params` 结构体，用于携带指针、问题形状 `(b, eh, c, l)` 和硬件信息。它是该 kernel 接入 CUTLASS 启动机制的结构性粘合层。

### 4. Lines 100-135 | Adapter-facing helper methods | 面向适配器的辅助方法

```cpp

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

- **EN:** These static methods translate arguments, declare zero workspace requirements, and define the grid/block shapes. The grid is `B*EH` blocks wide so one block processes one `(batch, head)` slice.

- **CN:** 这些静态方法负责转换参数、声明工作区大小为零，并定义 grid/block 形状。grid 的宽度是 `B*EH`，因此一个线程块处理一个 `(batch, head)` 切片。

### 5. Lines 136-188 | Operator body: tensor views, tiling, recasting, and running sum | 算子主体：张量视图、切分、重解释与运行和

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

#if 0
    if (thread_idx % 128 == 0 && blk_idx == 0) {
      print("tD  : ");print(tD);print("\n");
      print("tC  : ");print(tC);print("\n");
      print("cD  : ");print(cD);print("\n");
    }
#endif

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
```

- **EN:** The operator builds tensor views for input/output, carves a thread-local view with `logical_divide`, then iterates over packed vectors while maintaining `last_element` as the running prefix sum. Each packed input fragment is unpacked, accumulated in fp32, and written back to the output packet type.

- **CN:** 算子主体先为输入/输出构建张量视图，再通过 `logical_divide` 切出线程私有视图，然后在打包向量上循环，用 `last_element` 保存运行中的前缀和。每个打包输入片段都会被解包、以 fp32 方式累计，再写回输出打包类型。

### 6. Lines 189-194 | Class/namespace closure | 类与命名空间收尾

```cpp

private:

};

} // End namespace cutlass
```

- **EN:** The file ends with the private section placeholder and namespace close. There is no additional state because this helper kernel is intentionally minimal.

- **CN:** 文件以私有区占位和命名空间结束收尾。由于该辅助 kernel 被刻意设计得很轻量，因此没有额外状态。

## Takeaways | 总结

- **EN:** `reference_ssd_cumsum.hpp` is the SSD pipeline’s cheap preprocessing step: convert `DeltaA` increments into cumulative form once, then let the main kernel reuse that result.

- **CN:** `reference_ssd_cumsum.hpp` 是 SSD 流程中的廉价预处理步骤：先把 `DeltaA` 增量一次性转换为累计形式，再让主 kernel 直接复用。

---

## Key Concepts / 关键概念

- **Preprocessing scan / 预处理扫描**

  - **EN:** The kernel materializes prefix sums of `DeltaA`, giving the main SSD kernel chunk-local accumulated `A` values instead of raw increments.

  - **CN:** 该 kernel 负责把 `DeltaA` 变成前缀和，让主 SSD kernel 直接得到 chunk 内累积后的 `A` 值，而不是原始增量。

- **Packed vector memory access / 打包向量化访存**

  - **EN:** The code recasts thread tiles into 16-byte packet types so each loop iteration handles several scalar elements at once.

  - **CN:** 代码把线程分到的 tile 重解释为 16 字节打包类型，因此每次循环都能一次处理多个标量元素。

- **Per-`B*EH` block decomposition / 按 `B*EH` 分块**

  - **EN:** Each thread block handles one combined `(batch, expanded-head)` slice, while the inner loops walk the local `C × L` region.

  - **CN:** 每个线程块负责一个合并后的 `(batch, expanded-head)` 切片，内部循环再遍历本地的 `C × L` 区域。

- **CUTLASS transform adapter contract / CUTLASS 变换适配器契约**

  - **EN:** The kernel exposes `Arguments`, `Params`, launch shape helpers, and `to_underlying_arguments` so it can be wrapped by `TransformUniversalAdapter`.

  - **CN:** 该 kernel 提供 `Arguments`、`Params`、启动形状辅助函数和 `to_underlying_arguments`，从而可以被 `TransformUniversalAdapter` 封装调用。

- **Simple ALU kernel reused in Blackwell flow / 在 Blackwell 流程中复用的简单 ALU kernel**

  - **EN:** `ArchTag = arch::Sm90` signals that this helper is not using Blackwell-specific tensor core instructions; it is a lightweight utility inserted into the Blackwell SSD pipeline.

  - **CN:** `ArchTag = arch::Sm90` 说明这个辅助 kernel 并不依赖 Blackwell 专有张量核指令；它是插入到 Blackwell SSD 流程中的一个轻量级工具步骤。

## Dependencies / 依赖项

- **`cute/tensor.hpp`**

  - **EN:** Provides the tensor/view utilities, layout transforms, `logical_divide`, and recast helpers used throughout the operator body.

  - **CN:** 提供张量/视图工具、布局变换、`logical_divide` 和 recast 辅助函数，是算子主体的基础。

- **CUTLASS kernel-launch conventions**

  - **EN:** The metadata fields (`MaxThreadsPerBlock`, `ArchTag`, `SharedStorage`, `Arguments`, `Params`) match CUTLASS device-kernel expectations.

  - **CN:** 元数据字段（`MaxThreadsPerBlock`、`ArchTag`、`SharedStorage`、`Arguments`、`Params`）遵循 CUTLASS 设备 kernel 的启动约定。

- **`KernelHardwareInfo` and transform adapter**

  - **EN:** The `Arguments` structure carries hardware information and is consumed by the transform adapter launched from the host example.

  - **CN:** `Arguments` 结构携带硬件信息，并被主机示例中的 transform adapter 消费。
