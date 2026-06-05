# scaled_mm_sm90_int8_dispatch.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_sm90_int8_dispatch.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines SM90 int8 GEMM kernel families and shape-based dispatch for scaled matrix multiplication. / 定义 SM90 int8 GEMM 内核家族以及用于 scaled matmul 的基于形状的分发逻辑。

## Line-by-Line Analysis / 逐行分析
### Int8 kernel configs / Int8 内核配置
```cpp
template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm90_int8_config_default {
  // For M > 128 and any N
  static_assert(std::is_same<InType, int8_t>());
  using KernelSchedule =
      typename cutlass::gemm::KernelTmaWarpSpecializedPingpong;
  using EpilogueSchedule = typename cutlass::epilogue::TmaWarpSpecialized;
  using TileShape = Shape<_128, _128, _128>;
  using ClusterShape = Shape<_2, _1, _1>;
  using Cutlass3xGemm =
      cutlass_3x_gemm<InType, OutType, Epilogue, TileShape, ClusterShape,
                      KernelSchedule, EpilogueSchedule>;
// ...
template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm90_int8_config_M64 {
  // For M in (32, 64] and any N
  static_assert(std::is_same<InType, int8_t>());
  using KernelSchedule = typename cutlass::gemm::KernelTmaWarpSpecialized;
  using EpilogueSchedule = typename cutlass::epilogue::TmaWarpSpecialized;
  using TileShape = Shape<_64, _64, _256>;
  using ClusterShape = Shape<_1, _1, _1>;
  using Cutlass3xGemm =
      cutlass_3x_gemm<InType, OutType, Epilogue, TileShape, ClusterShape,
                      KernelSchedule, EpilogueSchedule>;
};

template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm90_int8_config_M32_NBig {
  // For M in [1, 32] and N >= 8192
  static_assert(std::is_same<InType, int8_t>());
  using KernelSchedule = typename cutlass::gemm::KernelTmaWarpSpecialized;
  using EpilogueSchedule = typename cutlass::epilogue::TmaWarpSpecialized;
  using TileShape = Shape<_64, _128, _256>;
  using ClusterShape = Shape<_1, _4, _1>;
  using Cutlass3xGemm =
      cutlass_3x_gemm<InType, OutType, Epilogue, TileShape, ClusterShape,
                      KernelSchedule, EpilogueSchedule>;
};

template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue>
struct sm90_int8_config_M32_NSmall {
  // For M in [1, 32] and N < 8192
  static_assert(std::is_same<InType, int8_t>());
  using KernelSchedule = typename cutlass::gemm::KernelTmaWarpSpecialized;
  using EpilogueSchedule = typename cutlass::epilogue::TmaWarpSpecialized;
  using TileShape = Shape<_64, _64, _256>;
  using ClusterShape = Shape<_1, _8, _1>;
  using Cutlass3xGemm =
      cutlass_3x_gemm<InType, OutType, Epilogue, TileShape, ClusterShape,
                      KernelSchedule, EpilogueSchedule>;
};
```
**EN:** The config structs cover a default large-M kernel plus several smaller-M variants, including separate paths for small and large `N` values.
**CN:** 这些配置结构覆盖了默认的大 M 内核以及多个更小 M 的变体，其中还区分了 `N` 较小和较大的路径。

### Shape-based dispatch / 基于形状的分发
```cpp
template <typename InType, typename OutType,
          template <typename, typename, typename> typename Epilogue,
          typename... EpilogueArgs>
inline void cutlass_gemm_sm90_int8_dispatch(torch::stable::Tensor& out,
                                            torch::stable::Tensor const& a,
                                            torch::stable::Tensor const& b,
                                            EpilogueArgs&&... args) {
  static_assert(std::is_same<InType, int8_t>());
  STD_TORCH_CHECK(a.scalar_type() == torch::headeronly::ScalarType::Char);
  STD_TORCH_CHECK(b.scalar_type() == torch::headeronly::ScalarType::Char);

  using Cutlass3xGemmDefault =
      typename sm90_int8_config_default<InType, OutType,
                                        Epilogue>::Cutlass3xGemm;
  using Cutlass3xGemmM128 =
      typename sm90_int8_config_M128<InType, OutType, Epilogue>::Cutlass3xGemm;
  using Cutlass3xGemmM64 =
      typename sm90_int8_config_M64<InType, OutType, Epilogue>::Cutlass3xGemm;
  using Cutlass3xGemmM32NBig =
      typename sm90_int8_config_M32_NBig<InType, OutType,
                                         Epilogue>::Cutlass3xGemm;
  using Cutlass3xGemmM32NSmall =
      typename sm90_int8_config_M32_NSmall<InType, OutType,
                                           Epilogue>::Cutlass3xGemm;

  uint32_t const n = out.size(1);
  bool const is_small_n = n < 8192;

  uint32_t const m = a.size(0);
  uint32_t const mp2 =
      std::max(static_cast<uint32_t>(32), next_pow_2(m));  // next power of 2

  if (mp2 <= 32) {
    // m in [1, 32]
    if (is_small_n) {
      return cutlass_gemm_caller<Cutlass3xGemmM32NSmall>(
          out, a, b, std::forward<EpilogueArgs>(args)...);
    } else {
      return cutlass_gemm_caller<Cutlass3xGemmM32NBig>(
          out, a, b, std::forward<EpilogueArgs>(args)...);
    }
  } else if (mp2 <= 64) {
    // m in (32, 64]
    return cutlass_gemm_caller<Cutlass3xGemmM64>(
        out, a, b, std::forward<EpilogueArgs>(args)...);
  } else if (mp2 <= 128) {
    // m in (64, 128]
    return cutlass_gemm_caller<Cutlass3xGemmM128>(
        out, a, b, std::forward<EpilogueArgs>(args)...);
  } else {
    // m in (128, inf)
    return cutlass_gemm_caller<Cutlass3xGemmDefault>(
        out, a, b, std::forward<EpilogueArgs>(args)...);
  }
}
```
**EN:** The dispatcher checks input dtypes, rounds `M` up to the next power of two, and uses that plus the `N < 8192` split to select the most appropriate kernel family.
**CN:** 分发器会检查输入数据类型，把 `M` 向上取到最近的 2 的幂，再结合 `N < 8192` 的分界选择最合适的内核家族。

### Output type wrapper / 输出类型包装器
```cpp
template <template <typename, typename, typename> typename Epilogue,
          typename... EpilogueArgs>
void cutlass_scaled_mm_sm90_int8_epilogue(torch::stable::Tensor& out,
                                          torch::stable::Tensor const& a,
                                          torch::stable::Tensor const& b,
                                          EpilogueArgs&&... epilogue_args) {
  STD_TORCH_CHECK(a.scalar_type() == torch::headeronly::ScalarType::Char);
  STD_TORCH_CHECK(b.scalar_type() == torch::headeronly::ScalarType::Char);

  if (out.scalar_type() == torch::headeronly::ScalarType::BFloat16) {
    return cutlass_gemm_sm90_int8_dispatch<int8_t, cutlass::bfloat16_t,
                                           Epilogue>(
        out, a, b, std::forward<EpilogueArgs>(epilogue_args)...);
  } else {
    STD_TORCH_CHECK(out.scalar_type() == torch::headeronly::ScalarType::Half);
    return cutlass_gemm_sm90_int8_dispatch<int8_t, cutlass::half_t, Epilogue>(
        out, a, b, std::forward<EpilogueArgs>(epilogue_args)...);
  }
```
**EN:** The final wrapper maps the runtime output tensor type to either BF16 or FP16 instantiations of the int8 dispatcher.
**CN:** 最后的包装器把运行时输出张量类型映射到 int8 分发器的 BF16 或 FP16 实例化。

## Key Concepts / 关键概念
- **Power-of-two bucketing / 按 2 的幂分桶**: Rounding `M` up simplifies the runtime heuristic and aligns neighboring problem sizes to the same tuned kernel family. / 把 `M` 向上取整到 2 的幂，可简化运行时启发式，并让相邻规模共享同一组调优内核。

## Dependencies / 依赖关系
- **cutlass_gemm_caller.cuh / cutlass_gemm_caller.cuh**: Relies on the shared launcher helper to allocate workspace and invoke the prepared CUTLASS kernel. / 依赖共享启动辅助函数来分配工作区并调用已准备好的 CUTLASS 内核。
- **scaled_mm.cuh / scaled_mm.cuh**: Reuses the shared SM90 GEMM builder template for int8 kernels. / 复用 `scaled_mm.cuh` 中共享的 SM90 GEMM 构建模板来生成 int8 内核。
