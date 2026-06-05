# leaky_bias.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/44_multi_gemm_ir_and_codegen/leaky_bias.h`
**Purpose / 用途**: Defines CUDA helpers and kernels for fused bias addition with ReLU, Leaky ReLU, or identity activation. / 定义将偏置加法与 ReLU、Leaky ReLU 或恒等激活融合的 CUDA 辅助函数与内核。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1–34 — License Header / 许可证头

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

#pragma once
#include <cuda_fp16.h>
```
**EN**: Standard NVIDIA BSD-3-Clause copyright block.

**CN**: 标准 NVIDIA BSD-3 条款版权声明。

---

### Lines 35–37 — Include Guard and FP16 Header / 头文件保护与 FP16 头文件

```cpp
template <typename T>
__device__
T add(T const & a, T const &b){
```
**EN**: `#pragma once` prevents multiple inclusion. `<cuda_fp16.h>` provides the `half` and `half2` types plus intrinsics such as `__hadd2`, `__hge2`, `__hle2`, `__hmul2`, `__hfma2`, and `__float22half2_rn` that are used throughout the activation functors.

**CN**: `#pragma once` 防止重复包含。`<cuda_fp16.h>` 提供 `half` 和 `half2` 类型，以及整个激活函数仿函数中使用的内建函数，如 `__hadd2`、`__hge2`、`__hle2`、`__hmul2`、`__hfma2` 和 `__float22half2_rn`。

---

### Lines 38–50 — `add` Helper Function Templates / `add` 辅助函数模板

```cpp
    return  (a + b);
}

template <>
__device__
half2 add(half2 const & a, half2 const &b){
    return (__hadd2(a,b));
}

template <typename T>
struct RELU{
    __device__
    T operator()(T const & a){
```
**EN**: A generic `add<T>` uses the natural `+` operator. The `half2` explicit specialisation replaces this with `__hadd2`, the fused dual-lane FP16 addition intrinsic. Using `__hadd2` is essential for correctness and performance: standard `+` is not overloaded for `half2` in all toolchains, and `__hadd2` maps to a single `HADD2` PTX instruction.

**CN**: 泛型 `add<T>` 使用自然的 `+` 运算符。`half2` 的显式特化将其替换为 `__hadd2`，即融合双通道 FP16 加法内建函数。使用 `__hadd2` 对正确性和性能至关重要：并非所有工具链都为 `half2` 重载了标准 `+` 运算符，且 `__hadd2` 直接映射到单条 `HADD2` PTX 指令。

---

### Lines 51–72 — `RELU` Functor / `RELU` 仿函数

```cpp
        return  a > T(0) ? a : T(0);
    }
    __device__
    half2 operator()(half2 const & a){
        float2 a_fp32x2 = __half22float2(a);
        a_fp32x2.x = a_fp32x2.x > 0.f ? a_fp32x2.x : 0.f;
        a_fp32x2.y = a_fp32x2.y > 0.f ? a_fp32x2.y : 0.f;
        if(a_fp32x2.x < 0.f || a_fp32x2.y < 0.f)
        printf(" %f %f\n", a_fp32x2.x ,a_fp32x2.y);
        return __float22half2_rn(a_fp32x2);
    }
};

template <typename T>
struct LEAKY_RELU{
    __device__
    T operator()(T const & a, T const & scale = half(1)){
        return  a > T(0) ? a : scale * a;
    }
    __device__
    half2 operator()(half2 const & a, half const & scale = half(1)){
        half2 zero = __half2half2(half(0));
```
**EN**: `RELU<T>::operator()(T)` clamps `a` to zero via a ternary. The `half2` overload widens to `float2`, applies the clamp per lane, then converts back. The `printf` inside is a debug remnant that fires when the result would be negative — indicating a clamp event — and is safe only because GPU `printf` is seldom triggered in practice. For production use this path would be removed.

**CN**: `RELU<T>::operator()(T)` 通过三目运算符将 `a` 截断为零。`half2` 重载先扩展到 `float2`，对每个通道应用截断，然后转换回去。内部的 `printf` 是调试遗留代码，当结果本应为负时触发（表示发生了截断事件）——由于 GPU `printf` 在实践中很少被触发，所以不会影响性能。生产环境中应将其移除。

---

### Lines 73–94 — `LEAKY_RELU` Functor / `LEAKY_RELU` 仿函数

```cpp
        half2 gt_zero = __hge2(a, zero);
        half2 le_zero = __hle2(a, zero);


        half2 scale_f16x2 = __half2half2(scale);
        half2 mask_scale_f16x2 = __hfma2(le_zero, scale_f16x2, gt_zero);
        return __hmul2(a, mask_scale_f16x2);
    }
};

template <int N, int BLOCKDIM>
__global__ void leaky_and_activation(half* inout, half* bias, half scale, bool mat_bias){

    constexpr bool N_MOD_2 = N & 1 ? false : true;

    using Access_tp = typename std::conditional<N_MOD_2, half2, half>::type;

    constexpr int Access_elements = sizeof(Access_tp) / sizeof(half);

    constexpr int iter = (N + (BLOCKDIM * Access_elements) - 1 ) / (BLOCKDIM * Access_elements);

    LEAKY_RELU<half> Act;
```
**EN**: `LEAKY_RELU<T>::operator()(a, scale)` returns `a` for positive inputs and `scale * a` for non-positive inputs. The `half2` overload uses a branchless mask approach: `gt_zero` is 1.0 per lane where `a >= 0`, `le_zero` is 1.0 where `a <= 0`. The combined mask is `fma(le_zero, scale, gt_zero)`, yielding `scale` for negative lanes and `1.0` for positive ones. Multiplying by this mask replaces the branch entirely with vectorised FP16 arithmetic, which is critical for warp divergence avoidance.

**CN**: `LEAKY_RELU<T>::operator()(a, scale)` 对正输入返回 `a`，对非正输入返回 `scale * a`。`half2` 重载采用无分支掩码方式：`gt_zero` 在 `a >= 0` 的通道为 1.0，`le_zero` 在 `a <= 0` 的通道为 1.0。组合掩码为 `fma(le_zero, scale, gt_zero)`，负数通道得到 `scale`，正数通道得到 `1.0`。将 `a` 乘以该掩码完全用向量化 FP16 算术替代了分支，对于避免 warp 分化至关重要。

---

### Lines 95–127 — `leaky_and_activation` Kernel (with bias) / 带偏置的 `leaky_and_activation` Kernel

```cpp
    Access_tp src_v[iter];
    Access_tp bias_v[iter];

    int batch_id = blockIdx.y;
    int batch_offset = batch_id * gridDim.x * N;

    for(int i = 0; i < iter; i++){
        int idx = (i * BLOCKDIM + threadIdx.x) * Access_elements;
        if (idx < N){
            src_v[i] = *reinterpret_cast<Access_tp*>(inout + blockIdx.x * N + idx + batch_offset);
            if (mat_bias)
                bias_v[i] = *reinterpret_cast<Access_tp*>(bias + blockIdx.x * N + idx + batch_offset);
            else
                bias_v[i] = *reinterpret_cast<Access_tp*>(bias + idx + batch_id * N);
            *reinterpret_cast<Access_tp*>(inout + blockIdx.x * N + idx + batch_offset) = Act(add(src_v[i],bias_v[i]),scale); 
        }
        
    }
}



template <int N, int BLOCKDIM>
__global__ void leaky_and_activation(half* inout, half scale){

    constexpr bool N_MOD_2 = N & 1 ? false : true;

    using Access_tp = typename std::conditional<N_MOD_2, half2, half>::type;

    constexpr int Access_elements = sizeof(Access_tp) / sizeof(half);

    constexpr int iter = (N + (BLOCKDIM * Access_elements) - 1 ) / (BLOCKDIM * Access_elements);
```
**EN**: Template parameters `N` (compile-time column width) and `BLOCKDIM` (thread-block size) drive all bounds. `Access_tp` is `half2` when `N` is even, `half` otherwise — a compile-time branch that doubles effective memory bandwidth. `iter` is the number of loop iterations each thread needs to cover its share of the `N`-wide row. The 2-D grid `(m, b)` maps `blockIdx.x` to matrix rows and `blockIdx.y` to batch index. `mat_bias` selects between per-element bias (offset by row) and per-row bias (offset only by batch).

**CN**: 模板参数 `N`（编译期列宽）和 `BLOCKDIM`（线程块大小）驱动所有边界计算。`Access_tp` 在 `N` 为偶数时为 `half2`，否则为 `half`——这是一个编译期分支，可将有效内存带宽翻倍。`iter` 是每个线程覆盖其在 `N` 宽行中份额所需的循环次数。二维 grid `(m, b)` 将 `blockIdx.x` 映射到矩阵行，将 `blockIdx.y` 映射到批次索引。`mat_bias` 在按元素偏置（按行偏移）和按行偏置（仅按批次偏移）之间选择。

---

### Lines 128–154 — `leaky_and_activation` Kernel (without bias) / 不带偏置的 `leaky_and_activation` Kernel

```cpp
    int batch_id = blockIdx.y;
    int batch_offset = batch_id * gridDim.x * N;

    LEAKY_RELU<half> Act;
    Access_tp src_v[iter];

    for(int i = 0; i < iter; i++){
        int idx = (i * BLOCKDIM + threadIdx.x) * Access_elements;
        if (idx < N){
            src_v[i] = *reinterpret_cast<Access_tp*>(inout + blockIdx.x * N + idx + batch_offset);
            *reinterpret_cast<Access_tp*>(inout + blockIdx.x * N + idx + batch_offset) = Act(src_v[i], scale);
        }
        
    }
}



template <int N, int BLOCKDIM>
void leaky_and_activation(half* inout, half* bias, int m, int b, half scale, bool mat_bias){

    dim3 grid(m, b);
    if (bias == nullptr)
        leaky_and_activation<N, BLOCKDIM><<<grid , BLOCKDIM>>>(inout, scale);
    else
        leaky_and_activation<N, BLOCKDIM><<<grid , BLOCKDIM>>>(inout, bias, scale, mat_bias);
}
```
**EN**: Overload without a bias pointer — just applies Leaky ReLU in-place. The same vectorisation and grid structure as the bias version. Keeping this as a separate kernel avoids a runtime branch per element inside the bias kernel when no bias is needed.

**CN**: 不带偏置指针的重载版本——直接对数据原地应用 Leaky ReLU。与带偏置版本采用相同的向量化和 grid 结构。将其作为单独的 kernel 可避免在不需要偏置时，在带偏置 kernel 内部产生每元素的运行时分支。

---

### Lines 155–163 — `leaky_and_activation` Host Launcher / `leaky_and_activation` 宿主端启动器

```cpp

template <int N, int BLOCKDIM>
__global__ void relu_and_activation(half* inout, half* bias, bool mat_bias){

    constexpr bool N_MOD_2 = N & 1 ? false : true;

    using Access_tp = typename std::conditional<N_MOD_2, half2, half>::type;

    constexpr int Access_elements = sizeof(Access_tp) / sizeof(half);
```
**EN**: Host-side dispatcher: constructs a `(m, b)` grid (rows × batch) and dispatches to the no-bias or bias device function depending on whether `bias == nullptr`. Template parameters `N` and `BLOCKDIM` are always compile-time constants, enabling all the `constexpr` computations inside the kernels.

**CN**: 宿主端分派器：构建 `(m, b)` 的 grid（行数 × 批次数），根据 `bias == nullptr` 分派到无偏置或有偏置的设备函数。模板参数 `N` 和 `BLOCKDIM` 始终是编译期常量，从而支持 kernel 内部所有的 `constexpr` 计算。

---

### Lines 164–201 — `relu_and_activation` Kernels (with and without bias) / 带偏置与不带偏置的 `relu_and_activation` Kernel

```cpp

    constexpr int iter = (N + (BLOCKDIM * Access_elements) - 1 ) / (BLOCKDIM * Access_elements);

    RELU<half> Act;
    Access_tp src_v[iter];
    Access_tp bias_v[iter];
    
    int batch_id = blockIdx.y;
    int batch_offset = batch_id * gridDim.x * N;

    for(int i = 0; i < iter; i++){
        int idx = (i * BLOCKDIM + threadIdx.x) * Access_elements;
        if (idx < N){
            src_v[i] = *reinterpret_cast<Access_tp*>(inout + blockIdx.x * N + idx + batch_offset);
            if (mat_bias)
                bias_v[i] = *reinterpret_cast<Access_tp*>(bias + blockIdx.x * N + idx + batch_offset);
            else
                bias_v[i] = *reinterpret_cast<Access_tp*>(bias + idx + batch_id * N);
            *reinterpret_cast<Access_tp*>(inout + blockIdx.x * N + idx + batch_offset) = Act(add(src_v[i],bias_v[i])); 
        }
        
    }
}



template <int N, int BLOCKDIM>
__global__ void relu_and_activation(half* inout){

    constexpr bool N_MOD_2 = N & 1 ? false : true;

    using Access_tp = typename std::conditional<N_MOD_2, half2, half>::type;

    constexpr int Access_elements = sizeof(Access_tp) / sizeof(half);

    constexpr int iter = (N + (BLOCKDIM * Access_elements) - 1 ) / (BLOCKDIM * Access_elements);

    int batch_id = blockIdx.y;
```
**EN**: Structurally identical to the Leaky ReLU kernels but instantiates `RELU<half>` instead of `LEAKY_RELU<half>`. No `scale` parameter is needed since standard ReLU has no slope for negative values. The same vectorisation strategy (`half2` when `N` is even) and the same `mat_bias` flag apply.

**CN**: 结构上与 Leaky ReLU kernel 完全相同，但实例化的是 `RELU<half>` 而非 `LEAKY_RELU<half>`。由于标准 ReLU 对负值没有斜率，因此不需要 `scale` 参数。相同的向量化策略（`N` 为偶数时使用 `half2`）和相同的 `mat_bias` 标志同样适用。

---

### Lines 202–210 — `relu_and_activation` Host Launcher / `relu_and_activation` 宿主端启动器

```cpp
    int batch_offset = batch_id * gridDim.x * N;

    RELU<half> Act;
    Access_tp src_v[iter];

    for(int i = 0; i < iter; i++){
        int idx = (i * BLOCKDIM + threadIdx.x) * Access_elements;
        if (idx < N){
            src_v[i] = *reinterpret_cast<Access_tp*>(inout + blockIdx.x * N + idx + batch_offset);
```
**EN**: Same host-launcher pattern as Leaky ReLU but without the `scale` argument.

**CN**: 与 Leaky ReLU 相同的宿主端启动器模式，但去掉了 `scale` 参数。

---

### Lines 211–259 — `identity_and_activation` Kernels (with and without bias) / 带偏置与不带偏置的 `identity_and_activation` Kernel

```cpp
            *reinterpret_cast<Access_tp*>(inout + blockIdx.x * N + idx + batch_offset) = Act(src_v[i]);
        }
        
    }
}



template <int N, int BLOCKDIM>
void relu_and_activation(half* inout, half* bias, int m, int b, bool mat_bias){
    dim3 grid(m, b);
    if (bias == nullptr)
        relu_and_activation<N, BLOCKDIM><<<grid , BLOCKDIM>>>(inout);
    else
        relu_and_activation<N, BLOCKDIM><<<grid , BLOCKDIM>>>(inout, bias, mat_bias);
}


template <int N, int BLOCKDIM>
__global__ void identity_and_activation(half* inout, half* bias, bool mat_bias){

    constexpr bool N_MOD_2 = N & 1 ? false : true;

    using Access_tp = typename std::conditional<N_MOD_2, half2, half>::type;

    constexpr int Access_elements = sizeof(Access_tp) / sizeof(half);

    constexpr int iter = (N + (BLOCKDIM * Access_elements) - 1 ) / (BLOCKDIM * Access_elements);

    int batch_id = blockIdx.y;
    int batch_offset = batch_id * gridDim.x * N;

    Access_tp src_v[iter];
    Access_tp bias_v[iter];

    for(int i = 0; i < iter; i++){
        int idx = (i * BLOCKDIM + threadIdx.x) * Access_elements;
        if (idx < N){
            src_v[i] = *reinterpret_cast<Access_tp*>(inout + blockIdx.x * N + idx + batch_offset);
            if (mat_bias)
                bias_v[i] = *reinterpret_cast<Access_tp*>(bias + blockIdx.x * N + idx + batch_offset);
            else
                bias_v[i] = *reinterpret_cast<Access_tp*>(bias + idx + batch_id * N);
            *reinterpret_cast<Access_tp*>(inout + blockIdx.x * N + idx + batch_offset) = (add(src_v[i],bias_v[i])); 
        }
        
    }
}
```
**EN**: The "identity" activation is a pass-through: no activation functor is applied, only the bias addition. The bias-variant simply stores `add(src, bias)`, and the no-bias variant copies the values unchanged. This kernel exists to give the multi-GEMM IR pipeline a uniform API regardless of which activation a given layer uses.

**CN**: "恒等"激活是直通操作：不应用任何激活仿函数，仅进行偏置加法。有偏置版本只存储 `add(src, bias)`，无偏置版本则原样复制数据。该 kernel 的存在是为了让多 GEMM IR 流水线拥有统一的 API，无论某层使用哪种激活函数。

---

### Lines 260–292 — `identity_and_activation` Host Launcher / `identity_and_activation` 宿主端启动器

```cpp
template <int N, int BLOCKDIM>
__global__ void identity_and_activation(half* inout){

    constexpr bool N_MOD_2 = N & 1 ? false : true;

    using Access_tp = typename std::conditional<N_MOD_2, half2, half>::type;

    constexpr int Access_elements = sizeof(Access_tp) / sizeof(half);

    constexpr int iter = (N + (BLOCKDIM * Access_elements) - 1 ) / (BLOCKDIM * Access_elements);

    int batch_id = blockIdx.y;
    int batch_offset = batch_id * gridDim.x * N;
    Access_tp src_v[iter];

    for(int i = 0; i < iter; i++){
        int idx = (i * BLOCKDIM + threadIdx.x) * Access_elements;
        if (idx < N){
            src_v[i] = *reinterpret_cast<Access_tp*>(inout + blockIdx.x * N + idx + batch_offset);
            *reinterpret_cast<Access_tp*>(inout + blockIdx.x * N + idx + batch_offset) = (src_v[i]);
        }
        
    }
}

template <int N, int BLOCKDIM>
void identity_and_activation(half* inout, half* bias, int m, int b, bool mat_bias){
    dim3 grid(m, b);
    if (bias == nullptr)
        identity_and_activation<N, BLOCKDIM><<<grid , BLOCKDIM>>>(inout);
    else
        identity_and_activation<N, BLOCKDIM><<<grid , BLOCKDIM>>>(inout, bias, mat_bias);
}
```
**EN**: Final host launcher following the same pattern. Together, the three families (`leaky_and_activation`, `relu_and_activation`, `identity_and_activation`) form a complete, uniform activation API for the example's multi-GEMM IR codegen pipeline, selectable at compile time via template instantiation.

**CN**: 遵循相同模式的最终宿主端启动器。三个系列（`leaky_and_activation`、`relu_and_activation`、`identity_and_activation`）共同为示例的多 GEMM IR 代码生成流水线构成了完整、统一的激活函数 API，可在编译期通过模板实例化进行选择。

---
## Key Concepts / 关键概念
- Activation functors provide scalar and `half2` paths so fused kernels can keep vectorized execution. / 激活仿函数同时提供标量与 `half2` 路径，使融合 kernel 保持向量化执行。
- Bias can be supplied per row or per element, changing only the accessor logic around the fused computation. / 偏置既可以按行提供，也可以按元素提供，只需更换融合计算附近的访存逻辑。
- Launcher templates hide kernel selection details while keeping activation policy as a template parameter. / 启动器模板隐藏了 kernel 选择细节，同时把激活策略保留为模板参数。
## Dependencies / 依赖项
- `cuda_fp16.h` — `half`/`half2` types and FP16 vector intrinsics / `half`/`half2` 类型及 FP16 向量内建函数
- `cuda_runtime_api.h` — Kernel launch and grid-stride execution rely on CUDA runtime declarations / kernel 启动与网格步进执行依赖 CUDA Runtime 声明
