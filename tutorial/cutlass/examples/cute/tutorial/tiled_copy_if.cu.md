# tiled_copy_if.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/cute/tutorial/tiled_copy_if.cu`

**Purpose / 用途**: Extends `tiled_copy.cu` with *predicated* copies using `cute::copy_if()`. This handles tensors whose shape is **not** divisible by the block tile size — the general case required for production GEMM epilogues and arbitrary-size tensor operations. Demonstrates construction of coordinate-based identity tensors and lazy predicate tensors. / 扩展 `tiled_copy.cu`，使用 `cute::copy_if()` 实现*谓词*复制，处理形状**不能**被块瓦片大小整除的张量——这是生产级 GEMM 尾声和任意大小张量操作所需的通用情况。演示基于坐标的恒等张量和惰性谓词张量的构建。

---

## Progression from tiled_copy to tiled_copy_if / 从 tiled_copy 到 tiled_copy_if 的演进

| Aspect / 方面 | `tiled_copy.cu` | `tiled_copy_if.cu` |
|---|---|---|
| Tensor shape | 256×512 (divisible by 128×64) | 528×300 (NOT divisible by 128×64) |
| Grid launch | Exact: 2×8 CTAs | Over-provisioned: ceil(528/128)×ceil(300/64) = 5×5 = 25 CTAs |
| Boundary handling | None needed | `copy_if` with predicate tensor |
| Copy primitive | `copy(src, dst)` | `copy_if(pred, src, dst)` |
| Predicate construction | N/A | `make_identity_tensor` + `lazy::transform` |
| Kernels | `copy_kernel`, `copy_kernel_vectorized` | `copy_if_kernel`, `copy_if_kernel_vectorized` |
| Tile extraction | `tiled_divide` (main only) | `local_tile` (inside kernel) |

---

## Line-by-Line Analysis / 逐行分析

### License, Includes, and File Comment / 许可证、头文件与文件注释

```cuda
/***************************************************************************************************
 * Copyright (c) 2023 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 **************************************************************************************************/
#include <thrust/host_vector.h>
#include <thrust/device_vector.h>
#include <cute/tensor.hpp>
#include "cutlass/util/print_error.hpp"
#include "cutlass/util/GPU_Clock.hpp"
#include "cutlass/util/helper_cuda.hpp"
```

**EN**: Same include set as `tiled_copy.cu`. The `tiled_copy_if.cu` file adds predication capability without needing any new headers — `copy_if`, `make_identity_tensor`, and `lazy::transform` are all part of `<cute/tensor.hpp>`.  
**CN**: 与 `tiled_copy.cu` 相同的头文件集。`tiled_copy_if.cu` 无需任何新头文件即可添加谓词功能——`copy_if`、`make_identity_tensor` 和 `lazy::transform` 都是 `<cute/tensor.hpp>` 的一部分。

---

### File-level Comment on Predication / 文件级谓词说明注释

```cuda
// This example extends `tiled_copy` using predicate tensors to guard memory accesses performed
// by `cute::copy_if()`. This enables tensors to have shapes that are not integer multiples of
// block sizes.
//
// This is accomplished by instantiating a tensor of coordinates which correspond to tensor elements
// to be accessed and then computing a predicate tensor which masks accesses. The example demonstrates
// how constructing of an identity tensor containing coordinates and a predicate tensor containing
// mask bits can be implemented using the same CuTe operations used to tile the tensors in
// Global Memory.
```

**EN**: The key insight described here: predication is implemented using the *same* tiling operations (partitioning, local_tile) applied to a *coordinate tensor* and *predicate tensor* as applied to the data tensor. This uniformity is a core CuTe design principle — the predicate mask follows the exact same partitioning as the data, guaranteeing correctness without separate index arithmetic.  
**CN**: 此处描述的关键洞见：谓词通过对*坐标张量*和*谓词张量*应用与数据张量*相同*的瓦片操作（分区、local_tile）来实现。这种统一性是 CuTe 的核心设计原则——谓词掩码遵循与数据完全相同的分区方式，无需单独的索引计算即可保证正确性。

---

### Kernel 1: `copy_if_kernel` — Signature / 核函数 1：`copy_if_kernel` 签名

```cuda
/// Simple copy kernel.
//
// Uses local_partition() to partition a tile among threads arranged as (THR_M, THR_N).
template <class TensorS, class TensorD, class BlockShape, class ThreadLayout>
__global__ void copy_if_kernel(TensorS S, TensorD D, BlockShape block_shape, ThreadLayout)
```

**EN**: Four template parameters vs. three in `tiled_copy`'s `copy_kernel`:
- `BlockShape` is now explicit (vs. implicit in the tiled tensor shape). This is because `tiled_copy_if` uses `local_tile` inside the kernel, requiring the block shape to be passed explicitly.
- `TensorS` and `TensorD` are the *full* (non-tiled) global tensors, not pre-tiled. This is another key difference from `tiled_copy.cu`: the tiling happens inside the kernel using `local_tile`, not in main using `tiled_divide`.

**CN**: 与 `tiled_copy` 的 `copy_kernel` 相比多一个模板参数：
- `BlockShape` 现在是显式的（而非隐含在瓦片化张量形状中）。这是因为 `tiled_copy_if` 在核函数内部使用 `local_tile`，需要显式传递块形状。
- `TensorS` 和 `TensorD` 是*完整*（未瓦片化）的全局张量，不是预先瓦片化的。这是与 `tiled_copy.cu` 的另一个关键区别：瓦片化在核函数内部使用 `local_tile` 完成，而非在 main 中使用 `tiled_divide`。

---

### Kernel 1: Identity Tensor and Lazy Predicate / 核函数 1：恒等张量与惰性谓词

```cuda
  using namespace cute;

  // Construct a coordinate tensor whose elements are the coordinates used to access tensors S and D.
  auto shape_S = shape(S);
  Tensor C = make_identity_tensor(shape_S);
  // Construct a predicate tensor which compares the coordinates with the original shape
  Tensor P = cute::lazy::transform(C, [&](auto c) { return elem_less(c, shape_S); });
```

**EN**: This is the core new concept of `tiled_copy_if`:

1. **`shape(S)`**: Retrieves the full shape of tensor S (e.g., `(528, 300)`).

2. **`make_identity_tensor(shape)`**: Creates a "coordinate tensor" where element at position `(i,j)` has value `(i,j)` — i.e., `C(i,j) = make_coord(i,j)`. This is the identity mapping: the "value" at each position is the position itself. No storage is allocated — this is a zero-overhead logical construct.

3. **`cute::lazy::transform(C, lambda)`**: Creates a lazy (not immediately evaluated) transformed tensor where each element is the result of applying the lambda to the corresponding element of C. `elem_less(c, shape_S)` returns `true` if coordinate `c` is element-wise less than `shape_S`, i.e., if the element is within bounds. The result `P` is a lazy boolean tensor where `P(i,j) = (i < 528) && (j < 300)`.

4. **Laziness**: Neither C nor P allocates memory or computes values until an element is actually accessed. This is critical for performance — only the accessed elements (determined by the thread's partition) are evaluated.

**CN**: 这是 `tiled_copy_if` 的核心新概念：

1. **`shape(S)`**：获取张量 S 的完整形状（例如 `(528, 300)`）。

2. **`make_identity_tensor(shape)`**：创建"坐标张量"，位置 `(i,j)` 处的元素值为 `(i,j)`，即 `C(i,j) = make_coord(i,j)`。这是恒等映射："值"就是位置本身。不分配任何存储——这是零开销的逻辑构造。

3. **`cute::lazy::transform(C, lambda)`**：创建惰性（不立即求值）变换张量，每个元素是对 C 对应元素应用 lambda 的结果。`elem_less(c, shape_S)` 在坐标 `c` 逐元素小于 `shape_S` 时返回 `true`，即元素在边界内。结果 `P` 是惰性布尔张量，`P(i,j) = (i < 528) && (j < 300)`。

4. **惰性求值**：C 和 P 均不分配内存或计算值，直到实际访问某个元素为止。这对性能至关重要——只有被访问的元素（由线程分区决定）才会被求值。

---

### Kernel 1: `local_tile` Inside the Kernel / 核函数 1：核函数内部的 `local_tile`

```cuda
  // Tile the input tensor into blocks
  auto block_coord = make_coord(blockIdx.x, blockIdx.y);
  Tensor tile_S = local_tile(S, block_shape, block_coord);   // (BlockShape_M, BlockShape_N)
  Tensor tile_D = local_tile(D, block_shape, block_coord);   // (BlockShape_M, BlockShape_N)
  Tensor tile_P = local_tile(P, block_shape, block_coord);   // (BlockShape_M, BlockShape_N)
```

**EN**: Unlike `tiled_copy.cu` where tiling was done in `main` with `tiled_divide`, here `local_tile` is called inside the kernel:
- `local_tile(S, block_shape, block_coord)`: Returns this block's tile of S. For boundary blocks (e.g., blockIdx.x=4 with block_shape_M=128, giving elements [512,640) vs tensor size 528), `local_tile` may address elements beyond the tensor bounds.
- Critically, `tile_P` is tiled the same way: `local_tile(P, block_shape, block_coord)`. The predicate tile will be `false` for out-of-bounds elements, preventing writes/reads to invalid memory.
- `block_coord = make_coord(blockIdx.x, blockIdx.y)`: 2D coordinate without the wildcard `_`, because we want a single tile, not the full K-axis.

**CN**: 与 `tiled_copy.cu` 中在 `main` 里用 `tiled_divide` 完成瓦片化不同，此处在核函数内部调用 `local_tile`：
- `local_tile(S, block_shape, block_coord)`：返回 S 当前块的瓦片。对于边界块（例如 blockIdx.x=4，block_shape_M=128，对应元素 [512,640)，而张量大小为 528），`local_tile` 可能寻址超出张量边界的元素。
- 关键在于，`tile_P` 以相同方式瓦片化：`local_tile(P, block_shape, block_coord)`。对于越界元素，谓词瓦片将为 `false`，防止对无效内存进行读写。
- `block_coord = make_coord(blockIdx.x, blockIdx.y)`：无通配符 `_` 的二维坐标，因为我们需要单个瓦片，而非完整 K 轴。

---

### Kernel 1: Thread Partitioning with Predicate / 核函数 1：带谓词的线程分区

```cuda
  // Construct a partitioning of the tile among threads with the given thread arrangement.

  // Concept:                         Tensor  ThrLayout       ThrIndex
  Tensor thr_tile_S = local_partition(tile_S, ThreadLayout{}, threadIdx.x);
  Tensor thr_tile_D = local_partition(tile_D, ThreadLayout{}, threadIdx.x);
  Tensor thr_tile_P = local_partition(tile_P, ThreadLayout{}, threadIdx.x);

  // Copy from GMEM to GMEM using `thr_tile_P` to guard accesses.
  copy_if(thr_tile_P, thr_tile_S, thr_tile_D);
```

**EN**: The predicate tensor `P` is partitioned *identically* to the data tensors using the same `local_partition` call. This ensures that `thr_tile_P(i,j)` corresponds exactly to whether `thr_tile_S(i,j)` is a valid memory access.

`copy_if(pred, src, dst)`: A CuTe primitive that copies `src[i]` to `dst[i]` only when `pred[i]` is `true`. For boundary elements, the predicate is `false`, so those elements are silently skipped — no out-of-bounds memory access occurs.

Note: Unlike `copy_kernel` in `tiled_copy.cu`, this version copies directly from GMEM to GMEM (no intermediate RMEM staging). For the vectorized version, RMEM staging is added back.

**CN**: 谓词张量 `P` 使用完全相同的 `local_partition` 调用与数据张量*相同地*分区。这确保 `thr_tile_P(i,j)` 精确对应 `thr_tile_S(i,j)` 是否是有效内存访问。

`copy_if(pred, src, dst)`：CuTe 原语，仅在 `pred[i]` 为 `true` 时将 `src[i]` 复制到 `dst[i]`。对于边界元素，谓词为 `false`，因此这些元素被静默跳过——不会发生越界内存访问。

注意：与 `tiled_copy.cu` 中的 `copy_kernel` 不同，此版本直接从 GMEM 复制到 GMEM（无中间 RMEM 暂存）。向量化版本会重新加入 RMEM 暂存。

---

### Kernel 2: `copy_if_kernel_vectorized` — Predicate Construction / 核函数 2：`copy_if_kernel_vectorized` 谓词构建

```cuda
template <class TensorS, class TensorD, class BlockShape, class Tiled_Copy>
__global__ void copy_if_kernel_vectorized(TensorS S, TensorD D, BlockShape block_shape, Tiled_Copy tiled_copy)
{
  using namespace cute;

  auto shape_S = shape(S);
  Tensor C = make_identity_tensor(shape_S);
  Tensor P = cute::lazy::transform(C, [&](auto c) { return elem_less(c, shape_S); });
```

**EN**: Identical predicate construction to `copy_if_kernel`. The identity tensor and lazy predicate are created the same way regardless of whether scalar or vectorized copies are used. This is a key design virtue: the predication logic is independent of the copy strategy.  
**CN**: 与 `copy_if_kernel` 相同的谓词构建。无论使用标量还是向量化复制，恒等张量和惰性谓词的创建方式完全相同。这是关键的设计优点：谓词逻辑与复制策略无关。

---

### Kernel 2: `local_tile` and `ThrCopy` Partitioning / 核函数 2：`local_tile` 与 `ThrCopy` 分区

```cuda
  auto block_coord = make_coord(blockIdx.x, blockIdx.y);
  Tensor tile_S = local_tile(S, block_shape, block_coord);       // (BlockShape_M, BlockShape_N)
  Tensor tile_D = local_tile(D, block_shape, block_coord);       // (BlockShape_M, BlockShape_N)
  Tensor tile_P = local_tile(P, block_shape, block_coord);       // (BlockShape_M, BlockShape_N)

  ThrCopy thr_copy = tiled_copy.get_thread_slice(threadIdx.x);
  Tensor thr_tile_S = thr_copy.partition_S(tile_S);              // (CPY, CPY_M, CPY_N)
  Tensor thr_tile_D = thr_copy.partition_D(tile_D);              // (CPY, CPY_M, CPY_N)
  Tensor thr_tile_P = thr_copy.partition_S(tile_P);              // (CPY, CPY_M, CPY_N)
```

**EN**: Critical detail: `thr_tile_P` is obtained with `partition_S` (source partition), NOT `partition_D`. This is because the predicate tensor acts as a *guard on the source* — it determines which source elements are valid to read. The predicate follows the source addressing, not the destination.

The `TiledCopy` partitioning (`partition_S/D`) is applied uniformly to data tiles and the predicate tile. This ensures the predicate aligns with the vectorized access pattern: if a vector of 4 elements is partially out of bounds, the predicate is checked per-vector (the entire vector is skipped if any element is OOB — this requires the tensor shape to be aligned with the vector width for correctness).

**CN**: 关键细节：`thr_tile_P` 通过 `partition_S`（源分区）而非 `partition_D` 获取。这是因为谓词张量充当*源端的守卫*——它决定哪些源元素可以安全读取。谓词遵循源端寻址，而非目标端。

`TiledCopy` 分区（`partition_S/D`）统一应用于数据瓦片和谓词瓦片。这确保谓词与向量化访问模式对齐：如果一组 4 个元素的向量部分越界，谓词按向量检查（如果任何元素越界则跳过整个向量——这要求张量形状与向量宽度对齐以保证正确性）。

---

### Kernel 2: Disabled Direct GMEM→GMEM Path / 核函数 2：禁用的直接 GMEM→GMEM 路径

```cuda
#if 0
  // Copy from GMEM to GMEM
  copy_if(tiled_copy, thr_tile_P, thr_tile_S, thr_tile_D);
#else
```

**EN**: A disabled code path showing the direct GMEM→GMEM copy variant: `copy_if(tiled_copy, pred, src, dst)` — the 4-argument vectorized form. This is simpler but may have correctness issues in some architectures where direct global-to-global copies with vector instructions aren't available or require special handling. The preferred path uses RMEM staging.  
**CN**: 禁用的代码路径，展示直接 GMEM→GMEM 复制变体：`copy_if(tiled_copy, pred, src, dst)` — 4 参数向量化形式。这更简单，但在某些架构上可能存在正确性问题，这些架构不支持带向量指令的直接全局→全局复制，或需要特殊处理。首选路径使用 RMEM 暂存。

---

### Kernel 2: Enabled RMEM-Staged Path / 核函数 2：启用的 RMEM 暂存路径

```cuda
  // make_fragment_like() constructs a tensor in RMEM with the same shape as thr_tile_S.
  Tensor frag = make_fragment_like(thr_tile_S);

  // Copy from GMEM to RMEM and from RMEM to GMEM
  copy_if(tiled_copy, thr_tile_P, thr_tile_S, frag);
  copy_if(tiled_copy, thr_tile_P, frag,       thr_tile_D);
#endif
```

**EN**: The preferred two-step approach:
1. `copy_if(tiled_copy, thr_tile_P, thr_tile_S, frag)`: Load valid elements from GMEM to RMEM; OOB elements are skipped (frag retains garbage for OOB positions, but those are never written to D).
2. `copy_if(tiled_copy, thr_tile_P, frag, thr_tile_D)`: Store valid elements from RMEM to GMEM D; OOB elements are skipped.

The predicate is applied identically to both copies. Even though `frag` contains garbage at OOB positions, the second `copy_if` with the same predicate ensures those positions are not written to `thr_tile_D`. This two-step approach is more robust and portable.

**CN**: 首选的两步方法：
1. `copy_if(tiled_copy, thr_tile_P, thr_tile_S, frag)`：从 GMEM 加载有效元素到 RMEM；越界元素被跳过（越界位置的 frag 保留垃圾值，但这些位置不会被写入 D）。
2. `copy_if(tiled_copy, thr_tile_P, frag, thr_tile_D)`：从 RMEM 存储有效元素到 GMEM D；越界元素被跳过。

谓词对两次复制的应用方式完全相同。即使 `frag` 在越界位置包含垃圾值，第二个使用相同谓词的 `copy_if` 也确保这些位置不会被写入 `thr_tile_D`。这种两步方法更加健壮和可移植。

---

### `main` — Non-Divisible Tensor Shape / `main` 函数：不可整除的张量形状

```cuda
  auto tensor_shape = make_shape(528, 300);
```

**EN**: The critical difference from `tiled_copy.cu` (which used 256×512). Here:
- 528 / 128 = 4.125 → need 5 blocks of 128, covering elements [0,640) but tensor only has [0,528).
- 300 / 64 = 4.6875 → need 5 blocks of 64, covering elements [0,320) but tensor only has [0,300).
- Grid: `ceil(528/128) × ceil(300/64)` = 5 × 5 = 25 CTAs.
- OOB elements: last M-block covers [512, 640) but only [512, 528) is valid; last N-block covers [256, 320) but only [256, 300) is valid.

**CN**: 与 `tiled_copy.cu`（使用 256×512）的关键区别：
- 528 / 128 = 4.125 → 需要 5 个 128 块，覆盖 [0,640)，但张量仅有 [0,528)。
- 300 / 64 = 4.6875 → 需要 5 个 64 块，覆盖 [0,320)，但张量仅有 [0,300)。
- 网格：`ceil(528/128) × ceil(300/64)` = 5 × 5 = 25 个 CTA。
- 越界元素：最后一个 M 块覆盖 [512, 640)，但仅 [512, 528) 有效；最后一个 N 块覆盖 [256, 320)，但仅 [256, 300) 有效。

---

### `main` — Data Initialization / `main` 函数：数据初始化

```cuda
  thrust::host_vector<Element> h_S(size(tensor_shape));
  thrust::host_vector<Element> h_D(size(tensor_shape));

  for (size_t i = 0; i < h_S.size(); ++i) {
    h_S[i] = static_cast<Element>(i);
    h_D[i] = Element{};
  }

  thrust::device_vector<Element> d_S = h_S;
  thrust::device_vector<Element> d_D = h_D;
  thrust::device_vector<Element> d_Zero = h_D;
```

**EN**: Same sequential integer initialization as `tiled_copy.cu`. Importantly, `d_Zero` is also created as a copy of the zeroed `h_D` — this will be used to reset `d_D` to zero between the two kernel launches (scalar and vectorized) to ensure independent verification.  
**CN**: 与 `tiled_copy.cu` 相同的连续整数初始化。重要的是，`d_Zero` 也作为清零 `h_D` 的副本创建——用于在两次核函数启动（标量和向量化）之间将 `d_D` 重置为零，确保独立验证。

---

### `main` — Tensor Construction (Full, Non-Tiled) / `main` 函数：张量构建（完整，未瓦片化）

```cuda
  Tensor tensor_S = make_tensor(make_gmem_ptr(d_S.data().get()), make_layout(tensor_shape));
  Tensor tensor_D = make_tensor(make_gmem_ptr(d_D.data().get()), make_layout(tensor_shape));
```

**EN**: Creates full (non-tiled) tensors over global memory. Unlike `tiled_copy.cu`, these are passed directly to the kernel — tiling happens inside the kernel with `local_tile`. The `.data().get()` idiom extracts the raw pointer from a Thrust `device_vector`.  
**CN**: 在全局内存上创建完整（未瓦片化）的张量。与 `tiled_copy.cu` 不同，这些张量直接传递给核函数——瓦片化在核函数内部使用 `local_tile` 完成。`.data().get()` 惯用法从 Thrust `device_vector` 中提取原始指针。

---

### `main` — Block Shape and Tiled Dimension for Grid / `main` 函数：块形状和用于网格计算的瓦片维度

```cuda
  auto block_shape = make_shape(Int<128>{}, Int<64>{});

  Tensor tiled_tensor_D = tiled_divide(tensor_D, block_shape);  // ((M, N), m', n')

  Layout thr_layout = make_layout(make_shape(Int<32>{}, Int< 8>{}));

  dim3 gridDim (size<1>(tiled_tensor_D), size<2>(tiled_tensor_D));
  dim3 blockDim(size(thr_layout));
```

**EN**: `tiled_divide` is called here *not* to pass to the kernel (the kernel uses `local_tile` internally), but purely to compute the grid dimensions `m'` and `n'`. 
- `size<1>(tiled_tensor_D)` = `ceil(528/128)` = 5 (m-direction tiles).
- `size<2>(tiled_tensor_D)` = `ceil(300/64)` = 5 (n-direction tiles, since `tiled_divide` uses ceiling division).

The 5×5 = 25 CTAs over-cover the 528×300 tensor; the predicate inside the kernel clips boundary tiles.

**CN**: 此处调用 `tiled_divide` *不是*为了传递给核函数（核函数内部使用 `local_tile`），而纯粹是为了计算网格维度 `m'` 和 `n'`：
- `size<1>(tiled_tensor_D)` = `ceil(528/128)` = 5（m 方向瓦片数）。
- `size<2>(tiled_tensor_D)` = `ceil(300/64)` = 5（n 方向瓦片数，因为 `tiled_divide` 使用上取整除法）。

5×5 = 25 个 CTA 超过覆盖 528×300 张量；核函数内的谓词裁剪边界瓦片。

---

### `main` — Scalar Kernel Launch and Verification / `main` 函数：标量核函数启动与验证

```cuda
  // copy_if()
  copy_if_kernel<<< gridDim, blockDim >>>(
    tensor_S,
    tensor_D,
    block_shape,
    thr_layout);

  cudaError result = cudaDeviceSynchronize();
  // ... error check ...
  h_D = d_D;

  auto verify = [](thrust::host_vector<Element> const &S, thrust::host_vector<Element> const &D){
    // ...
    for (size_t i = 0; i < D.size(); ++i) {
      if (S[i] != D[i]) { /* error */ }
    }
    return errors;
  };

  if (verify(h_D, h_S)) { return -1; }
  else { std::cout << "Success." << std::endl; }
```

**EN**: Launches `copy_if_kernel` (scalar, non-vectorized). The `verify` lambda performs element-wise comparison of all 528×300 = 158,400 elements. The lambda is defined inline and reused for both kernel verifications — a clean C++14 pattern.  
**CN**: 启动 `copy_if_kernel`（标量，非向量化）。`verify` lambda 对所有 528×300 = 158,400 个元素执行逐元素比较。lambda 内联定义并在两次核函数验证中复用——简洁的 C++14 模式。

---

### `main` — Reset and Vectorized Kernel Launch / `main` 函数：重置并启动向量化核函数

```cuda
  thrust::copy(d_Zero.begin(), d_Zero.end(), d_D.begin());

  // Value arrangement per thread
  Layout val_layout = make_layout(make_shape(Int<4>{}, Int<1>{}));   // (4,1) -> val_idx

  using CopyOp = UniversalCopy<uint_byte_t<sizeof(Element) * size(val_layout)>>;
  using Atom = Copy_Atom<CopyOp, Element>;

  TiledCopy tiled_copy = make_tiled_copy(Atom{}, thr_layout, val_layout);

  copy_if_kernel_vectorized<<< gridDim, blockDim >>>(
    tensor_S,
    tensor_D,
    block_shape,
    tiled_copy);

  // ... sync, error check, verify ...
  if (verify(h_D, h_S)) { return -1; }
  else { std::cout << "Success." << std::endl; }
  return 0;
```

**EN**: After verifying the scalar kernel:
1. `thrust::copy(d_Zero, d_D)`: Resets D to all zeros for a clean verification of the vectorized kernel.
2. Constructs `TiledCopy` identical to `tiled_copy.cu` — same 128-bit vectorized copy with 32×8 thread layout and 4×1 value layout.
3. Launches `copy_if_kernel_vectorized` with the same grid/block dimensions.
4. Verifies again with the same `verify` lambda.

**Important note**: For vectorized predication to be correct, the tensor shape must be aligned with the vector width in the K-dim. Here, 528 % 4 = 0 (valid alignment for 4-float vectors along M), and the value layout `(4,1)` means vectors are along M, so boundary tiles in the N-direction are handled element-by-element within the predicate, not per-vector.

**CN**: 验证标量核函数后：
1. `thrust::copy(d_Zero, d_D)`：将 D 重置为全零，为向量化核函数提供干净的验证环境。
2. 构建与 `tiled_copy.cu` 相同的 `TiledCopy`——相同的 128 位向量化复制，32×8 线程布局，4×1 值布局。
3. 以相同的网格/块维度启动 `copy_if_kernel_vectorized`。
4. 使用相同的 `verify` lambda 再次验证。

**重要说明**：向量化谓词要正确工作，张量形状必须与 K 维度的向量宽度对齐。此处 528 % 4 = 0（对 M 方向 4 浮点向量有效对齐），且值布局 `(4,1)` 意味着向量沿 M 方向，因此 N 方向的边界瓦片在谓词内逐元素处理，而非按向量处理。

---

## Key Concepts / 关键概念

- **`make_identity_tensor(shape)`** / **恒等张量**: Creates a coordinate tensor where element `(i,j,...)` has value `(i,j,...)`. Zero overhead — a logical construct. Used as the basis for predicate computation. / 创建坐标张量，元素 `(i,j,...)` 的值为 `(i,j,...)`。零开销——逻辑构造，用作谓词计算的基础。

- **`cute::lazy::transform(tensor, fn)`** / **惰性变换**: Creates a lazily-evaluated tensor where each element is `fn(original_element)`. No computation or allocation until element access. Enables predicate tensors to be tiled and partitioned with zero overhead. / 创建惰性求值张量，每个元素为 `fn(原始元素)`。元素访问前无计算或分配。使谓词张量能以零开销进行瓦片化和分区。

- **`elem_less(coord, shape)`** / **元素比较**: Returns `true` iff all components of `coord` are strictly less than corresponding components of `shape`. The building block for predication: `P(i,j) = (i < M) && (j < N)`. / 当且仅当 `coord` 的所有分量严格小于 `shape` 对应分量时返回 `true`。谓词的基本构建块：`P(i,j) = (i < M) && (j < N)`。

- **`copy_if(pred, src, dst)`** / **带谓词复制**: Copies `src[i]` to `dst[i]` only when `pred[i]` is true. The predicate tensor is partitioned identically to the data tensors, ensuring element-to-predicate correspondence. / 仅当 `pred[i]` 为真时将 `src[i]` 复制到 `dst[i]`。谓词张量与数据张量以相同方式分区，确保元素与谓词的对应关系。

- **Over-provisioned grid** / **过度配置的网格**: When tensor shapes aren't divisible by block shapes, launch `ceil(dim/block)` blocks in each direction. Some blocks will be entirely OOB (predicate always false) or partially OOB. `copy_if` handles both cases correctly. / 当张量形状不能被块形状整除时，在每个方向启动 `ceil(dim/block)` 个块。某些块可能完全越界（谓词始终为假）或部分越界。`copy_if` 正确处理两种情况。

- **`local_tile` inside kernel** / **核函数内部的 `local_tile`**: `tiled_copy_if` uses `local_tile` inside the kernel (vs. `tiled_divide` in main in `tiled_copy.cu`). Advantage: the full original tensor can be passed, enabling the kernel to compute predicates against the original shape. / `tiled_copy_if` 在核函数内部使用 `local_tile`（而非 `tiled_copy.cu` 中在 main 里使用 `tiled_divide`）。优势：可传递完整原始张量，使核函数能根据原始形状计算谓词。

- **Alignment requirement for vectorized `copy_if`** / **向量化 `copy_if` 的对齐要求**: The tensor's extents along vectorized dimensions must be divisible by the vector width (here: 528 % 4 = 0 for M, value layout `(4,1)`). Misalignment would cause incorrect predication at vector boundaries. / 张量沿向量化维度的范围必须能被向量宽度整除（此处：M 方向 528 % 4 = 0，值布局 `(4,1)`）。未对齐会导致向量边界处谓词不正确。

- **Uniform predicate-data partitioning** / **统一的谓词-数据分区**: The same `local_partition`/`ThrCopy.partition_S` is applied to both data and predicate tensors. This is CuTe's elegant solution: any partitioning valid for data is automatically valid for the corresponding predicate. / 对数据和谓词张量应用相同的 `local_partition`/`ThrCopy.partition_S`。这是 CuTe 的优雅解决方案：对数据有效的任何分区自动对对应谓词有效。

---

## Dependencies / 依赖项

- `<thrust/host_vector.h>`, `<thrust/device_vector.h>` — Memory containers and `thrust::copy` for device reset / 内存容器和用于设备重置的 `thrust::copy`
- `<cute/tensor.hpp>` — `make_identity_tensor`, `lazy::transform`, `elem_less`, `copy_if`, `local_tile`, `TiledCopy` / 谓词复制全套 API
- `"cutlass/util/print_error.hpp"` — Error formatting / 错误格式化
- `"cutlass/util/GPU_Clock.hpp"` — Timing (included, unused in main logic) / 计时（已包含，主逻辑中未使用）
- `"cutlass/util/helper_cuda.hpp"` — CUDA device utilities / CUDA 设备工具
