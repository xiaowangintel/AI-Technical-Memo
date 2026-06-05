# activation_kernels.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/activation_kernels.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements CUDA kernels for gated activations (SiLU/GELU variants, FatReLU, SwiGLU-OAI) and standalone GELU approximations with vectorized fast paths. / [CN] 实现门控激活（SiLU/GELU 变体、FatReLU、SwiGLU-OAI）以及独立 GELU 近似的 CUDA 内核，并提供向量化快速路径。

## Line-by-Line Analysis / 逐行分析
### Generic gated computation / 通用门控计算
```cpp
template <typename scalar_t, scalar_t (*ACT_FN)(const scalar_t&),
          bool act_first, bool HAS_CLAMP>
__device__ __forceinline__ scalar_t compute(const scalar_t& x,
                                            const scalar_t& y,
                                            const float limit) {
  if constexpr (act_first) {
    scalar_t gate = x;
    scalar_t up = y;
    if constexpr (HAS_CLAMP) {
      gate = (scalar_t)fminf((float)gate, limit);
      up = (scalar_t)fmaxf(fminf((float)up, limit), -limit);
    }
    return ACT_FN(gate) * up;
  } else {
    scalar_t gate = x;
    scalar_t up = y;
    if constexpr (HAS_CLAMP) {
      gate = (scalar_t)fmaxf(fminf((float)gate, limit), -limit);
      up = (scalar_t)fminf((float)up, limit);
    }
    return gate * ACT_FN(up);
  }
}

template <typename packed_t, packed_t (*PACKED_ACT_FN)(const packed_t&),
          bool act_first, bool HAS_CLAMP>
__device__ __forceinline__ packed_t packed_compute(const packed_t& x,
                                                   const packed_t& y,
                                                   const float limit) {
  if constexpr (act_first) {
    packed_t gate = x;
    packed_t up = y;
    if constexpr (HAS_CLAMP) {
      float2 g = cast_to_float2(gate);
      float2 u = cast_to_float2(up);
      g.x = fminf(g.x, limit);
      g.y = fminf(g.y, limit);
      u.x = fmaxf(fminf(u.x, limit), -limit);
      u.y = fmaxf(fminf(u.y, limit), -limit);
      gate = cast_to_packed<packed_t>(g);
      up = cast_to_packed<packed_t>(u);
    }
    return packed_mul(PACKED_ACT_FN(gate), up);
  } else {
    packed_t gate = x;
    packed_t up = y;
    if constexpr (HAS_CLAMP) {
      float2 g = cast_to_float2(gate);
      float2 u = cast_to_float2(up);
      g.x = fmaxf(fminf(g.x, limit), -limit);
      g.y = fmaxf(fminf(g.y, limit), -limit);
      u.x = fminf(u.x, limit);
      u.y = fminf(u.y, limit);
      gate = cast_to_packed<packed_t>(g);
      up = cast_to_packed<packed_t>(u);
    }
    return packed_mul(gate, PACKED_ACT_FN(up));
  }
```
**EN:** `compute` handles scalar inputs while `packed_compute` handles packed pairs such as `half2`/`bfloat162`/`float2`. The `act_first` template parameter flips whether the activation is applied to the gate half or the up half, and `HAS_CLAMP` injects model-specific clamping before the multiply.
**CN:** `compute` 处理标量输入，`packed_compute` 处理 `half2`、`bfloat162`、`float2` 等打包类型。模板参数 `act_first` 控制激活作用在 gate 半边还是 up 半边，`HAS_CLAMP` 则在乘法前插入模型需要的裁剪逻辑。

### Vectorized act-and-mul kernel / 向量化 act-and-mul 内核
```cpp
template <typename scalar_t, typename packed_t,
          scalar_t (*ACT_FN)(const scalar_t&),
          packed_t (*PACKED_ACT_FN)(const packed_t&), bool act_first,
          bool use_vec, bool HAS_CLAMP, bool use_256b = false>
__global__ void act_and_mul_kernel(
    scalar_t* __restrict__ out,          // [..., d]
    const scalar_t* __restrict__ input,  // [..., 2, d]
    const int d, const float limit) {
  const scalar_t* x_ptr = input + blockIdx.x * 2 * d;
  const scalar_t* y_ptr = x_ptr + d;
  scalar_t* out_ptr = out + blockIdx.x * d;

  if constexpr (use_vec) {
    using cuda_t = typename CUDATypeConverter<scalar_t>::Type;
    using pvec_t = PackedVec<cuda_t, use_256b>;

    const pvec_t* x_vec = reinterpret_cast<const pvec_t*>(x_ptr);
    const pvec_t* y_vec = reinterpret_cast<const pvec_t*>(y_ptr);
    pvec_t* out_vec = reinterpret_cast<pvec_t*>(out_ptr);
    const int num_vecs = d / 2 / pvec_t::NUM_ELTS;

    for (int i = threadIdx.x; i < num_vecs; i += blockDim.x) {
      pvec_t x, y;
      if constexpr (use_256b) {
        ld256(x, &x_vec[i]);
        ld256(y, &y_vec[i]);
      } else {
        ld128(x, &x_vec[i]);
        ld128(y, &y_vec[i]);
      }
#pragma unroll
      for (int j = 0; j < pvec_t::NUM_ELTS; j++) {
        x.elts[j] =
            packed_compute<packed_t, PACKED_ACT_FN, act_first, HAS_CLAMP>(
                x.elts[j], y.elts[j], limit);
      }
      if constexpr (use_256b) {
        st256(x, &out_vec[i]);
      } else {
        st128(x, &out_vec[i]);
      }
    }
  } else {
    // Scalar fallback for unaligned data or small d
    for (int64_t idx = threadIdx.x; idx < d; idx += blockDim.x) {
      const scalar_t x = VLLM_LDG(&x_ptr[idx]);
      const scalar_t y = VLLM_LDG(&y_ptr[idx]);
      out_ptr[idx] =
          compute<scalar_t, ACT_FN, act_first, HAS_CLAMP>(x, y, limit);
    }
  }
}
```
**EN:** Each block processes one token row. When alignment and width permit, the kernel reinterprets the row as `PackedVec` chunks and uses 128-bit or 256-bit load/store helpers from `cuda_vec_utils.cuh`; otherwise it falls back to scalar loads through `VLLM_LDG`.
**CN:** 每个 block 处理一行 token 数据。若对齐和宽度满足条件，内核会把该行重解释为 `PackedVec` 块，并调用 `cuda_vec_utils.cuh` 中的 128 位或 256 位读写辅助函数；否则退回到基于 `VLLM_LDG` 的标量路径。

### Activation math primitives / 激活函数数学原语
```cpp
template <typename T>
__device__ __forceinline__ T silu_kernel(const T& x) {
  // x * sigmoid(x)
  return (T)(((float)x) / (1.0f + expf((float)-x)));
}

template <typename packed_t>
__device__ __forceinline__ packed_t packed_silu_kernel(const packed_t& val) {
  // x * sigmoid(x)
  float2 fval = cast_to_float2(val);
  fval.x = fval.x / (1.0f + expf(-fval.x));
  fval.y = fval.y / (1.0f + expf(-fval.y));
  return cast_to_packed<packed_t>(fval);
}

template <typename T>
__device__ __forceinline__ T gelu_kernel(const T& x) {
  // Equivalent to PyTorch GELU with 'none' approximation.
  // Refer to:
  // https://github.com/pytorch/pytorch/blob/8ac9b20d4b090c213799e81acf48a55ea8d437d6/aten/src/ATen/native/cuda/ActivationGeluKernel.cu#L36-L38
  const float f = (float)x;
  constexpr float ALPHA = M_SQRT1_2;
  return (T)(f * 0.5f * (1.0f + ::erf(f * ALPHA)));
}

template <typename packed_t>
__device__ __forceinline__ packed_t packed_gelu_kernel(const packed_t& val) {
  // Equivalent to PyTorch GELU with 'none' approximation.
  // Refer to:
  // https://github.com/pytorch/pytorch/blob/8ac9b20d4b090c213799e81acf48a55ea8d437d6/aten/src/ATen/native/cuda/ActivationGeluKernel.cu#L36-L38
  constexpr float ALPHA = M_SQRT1_2;
  float2 fval = cast_to_float2(val);
  fval.x = fval.x * 0.5f * (1.0f + ::erf(fval.x * ALPHA));
  fval.y = fval.y * 0.5f * (1.0f + ::erf(fval.y * ALPHA));
  return cast_to_packed<packed_t>(fval);
}

template <typename T>
__device__ __forceinline__ T gelu_tanh_kernel(const T& x) {
  // Equivalent to PyTorch GELU with 'tanh' approximation.
  // Refer to:
  // https://github.com/pytorch/pytorch/blob/8ac9b20d4b090c213799e81acf48a55ea8d437d6/aten/src/ATen/native/cuda/ActivationGeluKernel.cu#L25-L30
  const float f = (float)x;
  constexpr float BETA = M_SQRT2 * M_2_SQRTPI * 0.5f;
  constexpr float KAPPA = 0.044715;
  float x_cube = f * f * f;
  float inner = BETA * (f + KAPPA * x_cube);
  return (T)(0.5f * f * (1.0f + ::tanhf(inner)));
}

template <typename packed_t>
__device__ __forceinline__ packed_t
packed_gelu_tanh_kernel(const packed_t& val) {
  // Equivalent to PyTorch GELU with 'tanh' approximation.
  // Refer to:
  // https://github.com/pytorch/pytorch/blob/8ac9b20d4b090c213799e81acf48a55ea8d437d6/aten/src/ATen/native/cuda/ActivationGeluKernel.cu#L25-L30
  float2 fval = cast_to_float2(val);
  constexpr float BETA = M_SQRT2 * M_2_SQRTPI * 0.5f;
  constexpr float KAPPA = 0.044715;

  float x_cube = fval.x * fval.x * fval.x;
  float inner = BETA * (fval.x + KAPPA * x_cube);
  fval.x = 0.5f * fval.x * (1.0f + ::tanhf(inner));

  x_cube = fval.y * fval.y * fval.y;
  inner = BETA * (fval.y + KAPPA * x_cube);
  fval.y = 0.5f * fval.y * (1.0f + ::tanhf(inner));
  return cast_to_packed<packed_t>(fval);
}
```
**EN:** This block defines the device-side formulas for SiLU, exact GELU, and tanh-approximated GELU, plus packed variants that convert packed registers to `float2`, apply math lane-by-lane, and repack the result.
**CN:** 这一段定义了设备侧的 SiLU、精确 GELU、tanh 近似 GELU 公式，以及对应的 packed 版本：先把 packed 寄存器转成 `float2`，逐 lane 执行计算，再重新打包。

### Launch macro and public wrappers / 启动宏与公开包装函数
```cpp
#define LAUNCH_ACTIVATION_GATE_KERNEL(KERNEL, PACKED_KERNEL, ACT_FIRST,        \
                                      HAS_CLAMP, LIMIT)                        \
  auto dtype = input.scalar_type();                                            \
  int d = input.size(-1) / 2;                                                  \
  int64_t num_tokens = input.numel() / input.size(-1);                         \
  if (num_tokens == 0) {                                                       \
    return;                                                                    \
  }                                                                            \
  dim3 grid(num_tokens);                                                       \
  int cc_major = at::cuda::getCurrentDeviceProperties()->major;                \
  int support_vec =                                                            \
      (CUDA_VERSION >= 12090 && cc_major >= 10 && num_tokens > 128)            \
          ? vllm::VecTraits<true>::ARCH_MAX_VEC_SIZE                           \
          : vllm::VecTraits<false>::ARCH_MAX_VEC_SIZE;                         \
  int vec_size = support_vec / at::elementSize(dtype);                         \
  const bool use_vec = (d % vec_size == 0);                                    \
  const at::cuda::OptionalCUDAGuard device_guard(device_of(input));            \
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();                \
  if (use_vec) {                                                               \
    dim3 block(std::min(d / vec_size, 1024));                                  \
    if (CUDA_VERSION >= 12090 && cc_major >= 10 && num_tokens > 128) {         \
      VLLM_DISPATCH_FLOATING_TYPES(dtype, "act_and_mul_kernel", [&] {          \
        vllm::act_and_mul_kernel<                                              \
            scalar_t, typename vllm::PackedTypeConverter<scalar_t>::Type,      \
            KERNEL<scalar_t>,                                                  \
            PACKED_KERNEL<typename vllm::PackedTypeConverter<scalar_t>::Type>, \
            ACT_FIRST, true, HAS_CLAMP, true><<<grid, block, 0, stream>>>(     \
            out.data_ptr<scalar_t>(), input.data_ptr<scalar_t>(), d, LIMIT);   \
      });                                                                      \
    } else {                                                                   \
      VLLM_DISPATCH_FLOATING_TYPES(dtype, "act_and_mul_kernel", [&] {          \
        vllm::act_and_mul_kernel<                                              \
            scalar_t, typename vllm::PackedTypeConverter<scalar_t>::Type,      \
            KERNEL<scalar_t>,                                                  \
            PACKED_KERNEL<typename vllm::PackedTypeConverter<scalar_t>::Type>, \
            ACT_FIRST, true, HAS_CLAMP, false><<<grid, block, 0, stream>>>(    \
            out.data_ptr<scalar_t>(), input.data_ptr<scalar_t>(), d, LIMIT);   \
      });                                                                      \
    }                                                                          \
  } else {                                                                     \
    dim3 block(std::min(d, 1024));                                             \
    VLLM_DISPATCH_FLOATING_TYPES(dtype, "act_and_mul_kernel", [&] {            \
      vllm::act_and_mul_kernel<                                                \
          scalar_t, typename vllm::PackedTypeConverter<scalar_t>::Type,        \
          KERNEL<scalar_t>,                                                    \
          PACKED_KERNEL<typename vllm::PackedTypeConverter<scalar_t>::Type>,   \
          ACT_FIRST, false, HAS_CLAMP><<<grid, block, 0, stream>>>(            \
          out.data_ptr<scalar_t>(), input.data_ptr<scalar_t>(), d, LIMIT);     \
    });                                                                        \
  }

void silu_and_mul(torch::Tensor& out,    // [..., d]
                  torch::Tensor& input)  // [..., 2 * d]
{
  LAUNCH_ACTIVATION_GATE_KERNEL(vllm::silu_kernel, vllm::packed_silu_kernel,
                                true, false, 0.0f);
}

void silu_and_mul_clamp(torch::Tensor& out,    // [..., d]
                        torch::Tensor& input,  // [..., 2 * d]
                        double limit) {
  LAUNCH_ACTIVATION_GATE_KERNEL(vllm::silu_kernel, vllm::packed_silu_kernel,
                                true, true, (float)limit);
}

void mul_and_silu(torch::Tensor& out,    // [..., d]
                  torch::Tensor& input)  // [..., 2 * d]
{
  // The difference between mul_and_silu and silu_and_mul is that mul_and_silu
  // applies the silu to the latter half of the input.
  LAUNCH_ACTIVATION_GATE_KERNEL(vllm::silu_kernel, vllm::packed_silu_kernel,
                                false, false, 0.0f);
}

void gelu_and_mul(torch::Tensor& out,    // [..., d]
                  torch::Tensor& input)  // [..., 2 * d]
{
  LAUNCH_ACTIVATION_GATE_KERNEL(vllm::gelu_kernel, vllm::packed_gelu_kernel,
                                true, false, 0.0f);
}

void gelu_tanh_and_mul(torch::Tensor& out,    // [..., d]
                       torch::Tensor& input)  // [..., 2 * d]
{
  LAUNCH_ACTIVATION_GATE_KERNEL(
      vllm::gelu_tanh_kernel, vllm::packed_gelu_tanh_kernel, true, false, 0.0f);
```
**EN:** The launch macro centralizes shape decoding, vector-width selection, CUDA stream/device guarding, and dtype dispatch. The exported functions such as `silu_and_mul` and `gelu_tanh_and_mul` differ only in which device function template they pass into this macro.
**CN:** 启动宏统一处理形状解析、向量宽度选择、CUDA stream/device 保护以及 dtype 分发。`silu_and_mul`、`gelu_tanh_and_mul` 等导出函数的差别仅在于传入了哪一个设备侧激活模板。

### Parameterized activations and interleaved SwiGLU / 带参数激活与交错式 SwiGLU
```cpp
template <typename T>
__device__ __forceinline__ T fatrelu_kernel(const T& x, const float threshold) {
  const float f = (float)x;
  return (T)(f > threshold ? f : 0.0f);
}

template <typename packed_t>
__device__ __forceinline__ packed_t
packed_fatrelu_kernel(const packed_t& val, const float threshold) {
  float2 fval = cast_to_float2(val);
  fval.x = fval.x > threshold ? fval.x : 0.0f;
  fval.y = fval.y > threshold ? fval.y : 0.0f;
  return cast_to_packed<packed_t>(fval);
}

template <typename scalar_t, typename packed_t,
          scalar_t (*ACT_FN)(const scalar_t&, const float),
          packed_t (*PACKED_ACT_FN)(const packed_t&, const float), bool use_vec,
          bool use_256b = false>
__global__ void act_and_mul_kernel_with_param(
    scalar_t* __restrict__ out, const scalar_t* __restrict__ input, const int d,
    const float param) {
  const scalar_t* x_ptr = input + blockIdx.x * 2 * d;
  const scalar_t* y_ptr = x_ptr + d;
  scalar_t* out_ptr = out + blockIdx.x * d;

  if constexpr (use_vec) {
    using cuda_t = typename CUDATypeConverter<scalar_t>::Type;
    using pvec_t = PackedVec<cuda_t, use_256b>;

    const pvec_t* x_vec = reinterpret_cast<const pvec_t*>(x_ptr);
    const pvec_t* y_vec = reinterpret_cast<const pvec_t*>(y_ptr);
    pvec_t* out_vec = reinterpret_cast<pvec_t*>(out_ptr);
    const int num_vecs = d / 2 / pvec_t::NUM_ELTS;

    for (int i = threadIdx.x; i < num_vecs; i += blockDim.x) {
      pvec_t x, y;
      if constexpr (use_256b) {
        ld256(x, &x_vec[i]);
        ld256(y, &y_vec[i]);
      } else {
        ld128(x, &x_vec[i]);
        ld128(y, &y_vec[i]);
      }
#pragma unroll
      for (int j = 0; j < pvec_t::NUM_ELTS; j++) {
        x.elts[j] = packed_mul(PACKED_ACT_FN(x.elts[j], param), y.elts[j]);
      }
      if constexpr (use_256b) {
        st256(x, &out_vec[i]);
      } else {
        st128(x, &out_vec[i]);
      }
    }
  } else {
    // Scalar fallback for unaligned data or small d
    for (int64_t idx = threadIdx.x; idx < d; idx += blockDim.x) {
      const scalar_t x = VLLM_LDG(&x_ptr[idx]);
      const scalar_t y = VLLM_LDG(&y_ptr[idx]);
      out_ptr[idx] = ACT_FN(x, param) * y;
    }
  }
}

template <typename T>
__device__ __forceinline__ T swigluoai_and_mul(const T& gate, const T& up,
                                               float alpha, float limit) {
  // Clamp gate to (-inf, limit] and up to [-limit, limit]
  const float g = fminf((float)gate, limit);
  const float u = fmaxf(fminf((float)up, limit), -limit);
  // glu = gate * sigmoid(gate * alpha), then return (up + 1) * glu
  return (T)((u + 1.0f) * g / (1.0f + expf(-g * alpha)));
}

// Interleaved gate/up: input has [gate0, up0, gate1, up1, ...].
template <typename scalar_t,
          scalar_t (*ACT_FN)(const scalar_t&, const scalar_t&, const float,
                             const float)>
__global__ void swigluoai_and_mul_kernel(
    scalar_t* __restrict__ out,          // [..., d]
    const scalar_t* __restrict__ input,  // [..., 2 * d] (interleaved)
    const int d, const float alpha, const float limit) {
  // For interleaved data: input has 2*d elements per token (gate/up pairs)
  // output has d elements per token
  constexpr int VEC_SIZE = 16 / sizeof(scalar_t);
  constexpr int PAIRS = VEC_SIZE / 2;  // Number of gate/up pairs per int4 load
  const int64_t token_idx = blockIdx.x;
  const scalar_t* in_ptr = input + token_idx * 2 * d;
  scalar_t* out_ptr = out + token_idx * d;

  // Check alignment for 128-bit vectorized access on input.
  // For output we use int2 (64-bit) which has 8-byte alignment requirement.
  const bool in_aligned = is_16byte_aligned(in_ptr);
  const bool out_aligned =
      (reinterpret_cast<uintptr_t>(out_ptr) & 7) == 0;  // 8-byte for int2

  if (in_aligned && out_aligned && d >= PAIRS) {
    // Fast path: vectorized loop
    // Each int4 load gives VEC_SIZE elements = PAIRS gate/up pairs
    // Each int2 store writes PAIRS output elements
    const int4* in_vec = reinterpret_cast<const int4*>(in_ptr);
    int2* out_vec = reinterpret_cast<int2*>(out_ptr);
    const int num_vecs = d / PAIRS;
    const int vec_end = num_vecs * PAIRS;

    for (int i = threadIdx.x; i < num_vecs; i += blockDim.x) {
      int4 v = VLLM_LDG(&in_vec[i]);
      int2 r;
      auto* vp = reinterpret_cast<scalar_t*>(&v);
      auto* rp = reinterpret_cast<scalar_t*>(&r);
#pragma unroll
      for (int j = 0; j < PAIRS; j++) {
        rp[j] = ACT_FN(vp[2 * j], vp[2 * j + 1], alpha, limit);
      }
      out_vec[i] = r;
    }
    // Scalar cleanup for remaining elements
    for (int i = vec_end + threadIdx.x; i < d; i += blockDim.x) {
      out_ptr[i] = ACT_FN(VLLM_LDG(&in_ptr[2 * i]),
                          VLLM_LDG(&in_ptr[2 * i + 1]), alpha, limit);
    }
  } else {
    // Scalar fallback for unaligned data or small d
    for (int64_t idx = threadIdx.x; idx < d; idx += blockDim.x) {
      // gate = x[..., ::2]  (even indices)
      const scalar_t gate = VLLM_LDG(&in_ptr[2 * idx]);
      // up = x[..., 1::2]   (odd indices)
      const scalar_t up = VLLM_LDG(&in_ptr[2 * idx + 1]);
      out_ptr[idx] = ACT_FN(gate, up, alpha, limit);
    }
  }
}
```
**EN:** `fatrelu_kernel` introduces a threshold parameter, while `swigluoai_and_mul_kernel` handles an interleaved layout `[gate0, up0, gate1, up1, ...]`. The latter uses `int4` loads and `int2` stores on aligned data so each vector transaction covers several gate/up pairs.
**CN:** `fatrelu_kernel` 引入阈值参数；`swigluoai_and_mul_kernel` 则处理 `[gate0, up0, gate1, up1, ...]` 这种交错布局。后者在对齐条件满足时使用 `int4` 读取和 `int2` 写回，使一次向量事务覆盖多个 gate/up 对。

### Standalone elementwise activations / 独立逐元素激活
```cpp
// Element-wise activation kernel template.
template <typename scalar_t, scalar_t (*ACT_FN)(const scalar_t&), bool use_vec,
          bool use_256b = false>
__global__ void activation_kernel(
    scalar_t* __restrict__ out,          // [..., d]
    const scalar_t* __restrict__ input,  // [..., d]
    const int d) {
  const scalar_t* in_ptr = input + blockIdx.x * d;
  scalar_t* out_ptr = out + blockIdx.x * d;

  if constexpr (use_vec) {
    // Fast path: 128-bit/256-bit vectorized loop
    using vec_t = typename VecTraits<use_256b>::vec_t;
    constexpr int ARCH_MAX_VEC_SIZE = VecTraits<use_256b>::ARCH_MAX_VEC_SIZE;
    constexpr int VEC_SIZE = ARCH_MAX_VEC_SIZE / sizeof(scalar_t);
    const vec_t* in_vec = reinterpret_cast<const vec_t*>(in_ptr);
    vec_t* out_vec = reinterpret_cast<vec_t*>(out_ptr);
    const int num_vecs = d / VEC_SIZE;

    for (int i = threadIdx.x; i < num_vecs; i += blockDim.x) {
      vec_t v;
      if constexpr (use_256b) {
        ld256(v, &in_vec[i]);
      } else {
        v = VLLM_LDG(&in_vec[i]);
      }
      auto* vp = reinterpret_cast<scalar_t*>(&v);
#pragma unroll
      for (int j = 0; j < VEC_SIZE; j++) {
        vp[j] = ACT_FN(vp[j]);
      }
      if constexpr (use_256b) {
        st256(v, &out_vec[i]);
      } else {
        out_vec[i] = v;
      }
    }
  } else {
    // Scalar fallback for unaligned data or small d
    for (int64_t idx = threadIdx.x; idx < d; idx += blockDim.x) {
      const scalar_t x = VLLM_LDG(&in_ptr[idx]);
      out_ptr[idx] = ACT_FN(x);
    }
  }
}

}  // namespace vllm

// Launch element-wise activation kernel.
#define LAUNCH_ACTIVATION_KERNEL(KERNEL)                                 \
  auto dtype = input.scalar_type();                                      \
  int d = input.size(-1);                                                \
  int64_t num_tokens = input.numel() / input.size(-1);                   \
  if (num_tokens == 0) {                                                 \
    return;                                                              \
  }                                                                      \
  dim3 grid(num_tokens);                                                 \
  int cc_major = at::cuda::getCurrentDeviceProperties()->major;          \
  int support_vec =                                                      \
      (CUDA_VERSION >= 12090 && cc_major >= 10 && num_tokens > 128)      \
          ? vllm::VecTraits<true>::ARCH_MAX_VEC_SIZE                     \
          : vllm::VecTraits<false>::ARCH_MAX_VEC_SIZE;                   \
  int vec_size = support_vec / at::elementSize(dtype);                   \
  const bool use_vec = (d % vec_size == 0);                              \
  const at::cuda::OptionalCUDAGuard device_guard(device_of(input));      \
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();          \
  if (use_vec) {                                                         \
    dim3 block(std::min(d / vec_size, 1024));                            \
    if (CUDA_VERSION >= 12090 && cc_major >= 10 && num_tokens > 128) {   \
      VLLM_DISPATCH_FLOATING_TYPES(dtype, "activation_kernel", [&] {     \
        vllm::activation_kernel<scalar_t, KERNEL<scalar_t>, true, true>  \
            <<<grid, block, 0, stream>>>(out.data_ptr<scalar_t>(),       \
                                         input.data_ptr<scalar_t>(), d); \
      });                                                                \
    } else {                                                             \
      VLLM_DISPATCH_FLOATING_TYPES(dtype, "activation_kernel", [&] {     \
        vllm::activation_kernel<scalar_t, KERNEL<scalar_t>, true, false> \
            <<<grid, block, 0, stream>>>(out.data_ptr<scalar_t>(),       \
                                         input.data_ptr<scalar_t>(), d); \
      });                                                                \
    }                                                                    \
  } else {                                                               \
    dim3 block(std::min(d, 1024));                                       \
    VLLM_DISPATCH_FLOATING_TYPES(dtype, "activation_kernel", [&] {       \
      vllm::activation_kernel<scalar_t, KERNEL<scalar_t>, false>         \
          <<<grid, block, 0, stream>>>(out.data_ptr<scalar_t>(),         \
                                       input.data_ptr<scalar_t>(), d);   \
    });                                                                  \
  }

namespace vllm {

template <typename T>
__device__ __forceinline__ T gelu_new_kernel(const T& x) {
  const float x3 = (float)(x * x * x);
  const T t = (T)tanhf((T)(0.79788456f * (float)(x + (T)(0.044715f * x3))));
  return ((T)0.5) * x * (((T)1.0) + t);
}

template <typename T>
__device__ __forceinline__ T gelu_fast_kernel(const T& x) {
  const float f = (float)x;
  const T t =
      (T)tanhf(((T)(f * 0.79788456f)) * (((T)1.0) + (T)(0.044715f * f) * x));
  return ((T)0.5) * x * (((T)1.0) + t);
}

template <typename T>
__device__ __forceinline__ T gelu_quick_kernel(const T& x) {
  // x * sigmoid(1.702 * x)
  return (T)(((float)x) / (1.0f + expf(-1.702f * (float)x)));
}

}  // namespace vllm

void gelu_new(torch::Tensor& out,    // [..., d]
              torch::Tensor& input)  // [..., d]
{
  LAUNCH_ACTIVATION_KERNEL(vllm::gelu_new_kernel);
}

void gelu_fast(torch::Tensor& out,    // [..., d]
               torch::Tensor& input)  // [..., d]
{
  LAUNCH_ACTIVATION_KERNEL(vllm::gelu_fast_kernel);
}

void gelu_quick(torch::Tensor& out,    // [..., d]
                torch::Tensor& input)  // [..., d]
{
  LAUNCH_ACTIVATION_KERNEL(vllm::gelu_quick_kernel);
}
```
**EN:** The file ends with a second kernel family for pure elementwise activations. `activation_kernel` reuses the same vector/scalar split, and the exported wrappers provide three GELU approximations (`gelu_new`, `gelu_fast`, `gelu_quick`) used by different model implementations.
**CN:** 文件末尾定义了另一套纯逐元素激活内核。`activation_kernel` 复用了同样的向量/标量分流策略，而导出包装函数提供了三种 GELU 近似（`gelu_new`、`gelu_fast`、`gelu_quick`），供不同模型实现选择。

## Key Concepts / 关键概念
- **EN:** Template parameters encode activation choice, vector width, clamp behavior, and memory path so the runtime launcher stays small.
  **CN:** 模板参数把激活种类、向量宽度、裁剪行为和访存路径都编码进编译期，运行时 launcher 因而保持简洁。
- **EN:** `PackedTypeConverter`, `PackedVec`, and `VecTraits` let the same kernel body handle FP16, BF16, and FP32 efficiently.
  **CN:** `PackedTypeConverter`、`PackedVec` 与 `VecTraits` 让同一套内核主体可以高效覆盖 FP16、BF16 与 FP32。
- **EN:** The code prefers 256-bit PTX on Blackwell-class GPUs with CUDA 12.9+, then falls back to 128-bit or scalar code for portability.
  **CN:** 代码优先在 Blackwell 级 GPU 且 CUDA 12.9+ 上使用 256 位 PTX，再为可移植性退回 128 位或标量实现。
- **EN:** Gated kernels assume the last dimension packs two streams (`gate` and `up`) and write only the fused result.
  **CN:** 门控内核假定最后一维中打包了两路数据（`gate` 与 `up`），最终只写回融合后的结果。

## Dependencies / 依赖关系
- **EN:** `cuda_vec_utils.cuh` supplies packed-vector types, conversions, and 128/256-bit load/store primitives.
  **CN:** `cuda_vec_utils.cuh` 提供 packed 向量类型、类型转换以及 128/256 位读写原语。
- **EN:** `cuda_compat.h` provides backend-neutral helpers such as `VLLM_LDG` and shuffle wrappers.
  **CN:** `cuda_compat.h` 提供 `VLLM_LDG`、shuffle 封装等后端无关辅助接口。
- **EN:** `dispatch_utils.h` expands dtype dispatch macros used by all exported entry points.
  **CN:** `dispatch_utils.h` 展开所有导出入口使用的 dtype 分发宏。
- **EN:** `ATen/cuda` and `c10/cuda` provide stream access and device guards for PyTorch extension launches.
  **CN:** `ATen/cuda` 与 `c10/cuda` 为 PyTorch 扩展启动提供 stream 获取与 device guard。
