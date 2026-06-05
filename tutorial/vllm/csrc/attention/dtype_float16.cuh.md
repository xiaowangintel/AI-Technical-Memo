# dtype_float16.cuh — Code Analysis / 代码分析

        ## Source / 来源
        - **File**: `./csrc/attention/dtype_float16.cuh`
        - **Repository**: `vllm-project/vllm`
        - **Purpose**: **EN:** Specializes the attention math primitives for packed FP16 data using low-level CUDA/ROCm instructions and FP32 accumulator forms. **CN:** 该文件使用底层 CUDA/ROCm 指令以及 FP32 累计形式，为打包 FP16 数据特化注意力数学原语。

        ## Line-by-Line Analysis / 逐行分析
        ### [packed FP16 type mapping / 打包 FP16 类型映射]
        ```cpp
        template <>
        struct Vec<uint16_t, 1> {
          using Type = uint16_t;
        };
        template <>
        struct Vec<uint16_t, 2> {
          using Type = uint32_t;
        };
        template <>
        struct Vec<uint16_t, 4> {
          using Type = uint2;
        };
        template <>
        struct Vec<uint16_t, 8> {
          using Type = uint4;
        };
        ```
        **EN:** FP16 data is represented as raw integer containers rather than C++ half objects. That lets the kernel use packed arithmetic instructions (`f16x2`) efficiently and reinterpret contiguous bytes without extra wrapper overhead.

        **CN:** FP16 数据在这里不是用 C++ 的 half 对象表示，而是直接用整数容器承载原始位模式。这样做可以更高效地使用 `f16x2` 一类打包指令，也便于直接按字节重解释连续内存。

        ### [FP32 accumulator mapping / FP32 累计映射]
        ```cpp
        template <>
        struct FloatVec<uint16_t> {
          using Type = float;
        };
        template <>
        struct FloatVec<uint32_t> {
          using Type = float2;
        };
        template <>
        struct FloatVec<uint2> {
          using Type = Float4_;
        };
        template <>
        struct FloatVec<uint4> {
          using Type = Float8_;
        };
        ```
        **EN:** These mappings tell higher-level generic code how to widen packed FP16 vectors into FP32-friendly accumulator shapes. For example, an 8-half pack (`uint4`) accumulates as `Float8_`, which stores four `float2` lanes.

        **CN:** 这些映射告诉上层泛型代码：打包 FP16 向量应该扩宽成什么样的 FP32 累计形状。例如，8 个 half 的打包形式 `uint4` 会对应为 `Float8_`，也就是由四个 `float2` 组成的累计容器。

        ### [conversion helpers and inline assembly / 转换辅助与内联汇编]
        ```cpp
        inline __device__ uint32_t h0_h0(uint16_t a) {
        #ifndef USE_ROCM
          uint32_t b;
          asm volatile("mov.b32 %0, {%1, %1};" : "=r"(b) : "h"(a));
          return b;
        #else
          union {
            uint32_t u32;
            uint16_t u16[2];
          } tmp;
          tmp.u16[0] = a;
          tmp.u16[1] = a;
          return tmp.u32;
        #endif
        }

        inline __device__ float half_to_float(uint16_t h) {
          float f;
        #ifndef USE_ROCM
          asm volatile("cvt.f32.f16 %0, %1;
" : "=f"(f) : "h"(h));
        #else
          asm volatile("v_cvt_f32_f16 %0, %1;" : "=v"(f) : "v"(h));
        #endif
          return f;
        }
        ```
        **EN:** `h0_h0` duplicates a single half into both lanes of a 32-bit packed register, which is useful for scalar-times-packed-vector operations. The conversion helpers use inline assembly on both CUDA and ROCm paths to get direct access to hardware conversion instructions instead of relying on heavier abstractions.

        **CN:** `h0_h0` 会把一个 half 复制到 32 位打包寄存器的两个 lane 中，这对“标量 × 打包向量”运算很有用。相关转换函数在 CUDA 与 ROCm 路径上都使用了内联汇编，以便直接调用硬件转换指令，而不是依赖更重的抽象层。

        ### [packed add and multiply / 打包加法与乘法]
        ```cpp
        inline __device__ uint32_t add(uint32_t a, uint32_t b) {
          uint32_t c;
        #ifndef USE_ROCM
          asm volatile("add.f16x2 %0, %1, %2;
" : "=r"(c) : "r"(a), "r"(b));
        #else
          asm volatile("v_pk_add_f16 %0, %1, %2;
" : "=v"(c) : "v"(a), "v"(b));
        #endif
          return c;
        }

        template <>
        inline __device__ uint32_t mul(uint16_t a, uint32_t b) {
          return mul<uint32_t, uint32_t, uint32_t>(h0_h0(a), b);
        }

        template <>
        inline __device__ uint4 mul(uint4 a, uint4 b) {
          uint4 c;
          c.x = mul<uint32_t, uint32_t, uint32_t>(a.x, b.x);
          c.y = mul<uint32_t, uint32_t, uint32_t>(a.y, b.y);
          c.z = mul<uint32_t, uint32_t, uint32_t>(a.z, b.z);
          c.w = mul<uint32_t, uint32_t, uint32_t>(a.w, b.w);
          return c;
        }
        ```
        **EN:** Packed arithmetic is defined recursively: basic 2-half operations run through native packed instructions, and larger `uint2`/`uint4` vectors are handled by applying those same operations lane by lane. The scalar-times-packed overload uses `h0_h0` to broadcast one FP16 value across both lanes.

        **CN:** 打包算术是递归构建的：最基础的 2-half 操作直接落到原生打包指令上，而更大的 `uint2`/`uint4` 则按子 lane 逐个复用相同逻辑。标量乘打包向量的重载借助 `h0_h0` 先把单个 FP16 广播到两个 lane。

        ### [widened multiply-add paths / 扩宽的乘加路径]
        ```cpp
        template <>
        inline __device__ Float8_ mul(uint4 a, uint4 b) {
          Float8_ fc;
          fc.x = mul<float2, uint32_t, uint32_t>(a.x, b.x);
          fc.y = mul<float2, uint32_t, uint32_t>(a.y, b.y);
          fc.z = mul<float2, uint32_t, uint32_t>(a.z, b.z);
          fc.w = mul<float2, uint32_t, uint32_t>(a.w, b.w);
          return fc;
        }

        inline __device__ Float8_ fma(uint4 a, uint4 b, Float8_ fc) {
          Float8_ fd;
          fd.x = fma(a.x, b.x, fc.x);
          fd.y = fma(a.y, b.y, fc.y);
          fd.z = fma(a.z, b.z, fc.z);
          fd.w = fma(a.w, b.w, fc.w);
          return fd;
        }
        ```
        **EN:** These overloads are the important bridge from packed FP16 storage to FP32 accumulation. They are what allow generic routines like `qk_dot_` to multiply packed half vectors but keep the intermediate result in a widened format with much better numerical behavior.

        **CN:** 这些重载是打包 FP16 存储与 FP32 累计之间最关键的桥梁。正因为有它们，`qk_dot_` 之类的泛型逻辑才能对 half 向量做打包乘法，同时把中间结果保留在数值表现更好的扩宽格式中。

        ### [reductions and conversion back / 归约与回写转换]
        ```cpp
        template <>
        inline __device__ float sum(uint4 v) {
          uint32_t c = add(v.x, v.y);
          c = add(c, v.z);
          c = add(c, v.w);
          return sum(c);
        }

        inline __device__ void from_float(uint4& dst, Float8_ src) {
          dst.x = float2_to_half2(src.x);
          dst.y = float2_to_half2(src.y);
          dst.z = float2_to_half2(src.z);
          dst.w = float2_to_half2(src.w);
        }

        inline __device__ Float8_ to_float(uint4 u) {
          Float8_ tmp;
          tmp.x = half2_to_float2(u.x);
          tmp.y = half2_to_float2(u.y);
          tmp.z = half2_to_float2(u.z);
          tmp.w = half2_to_float2(u.w);
          return tmp;
        }
        ```
        **EN:** `sum` progressively collapses packed FP16 lanes into a scalar float, which is exactly what the dot-product code needs after vectorized multiplication. `from_float` and `to_float` provide the two-way bridge between packed storage and widened computation.

        **CN:** `sum` 会逐步把打包 FP16 lane 压缩成标量 float，这正是点积逻辑在向量化乘法之后所需要的行为。`from_float` 和 `to_float` 则构成了打包存储与扩宽计算之间的双向桥梁。

        ### [specialized zero / 专用清零]
        ```cpp
        inline __device__ void zero(uint16_t& dst) { dst = uint16_t(0); }
        ```
        **EN:** FP16 scalar zeroing is specialized because the generic word-based `zero` helper in `attention_generic.cuh` is aimed at 4-byte-multiple packed types.

        **CN:** 之所以专门给 FP16 标量提供 `zero`，是因为 `attention_generic.cuh` 中的通用清零函数主要面向大小为 4 字节整数倍的打包类型。

        ## Key Concepts / 关键概念
        - **EN:** FP16 values are manipulated as raw packed integers to match hardware packed instructions closely.
        - **CN:** FP16 值以原始打包整数形式处理，以更贴近底层硬件打包指令。
        - **EN:** The file exposes both storage-precision operations and widened FP32 accumulation operations.
        - **CN:** 该文件同时提供了“存储精度”的运算和“扩宽到 FP32 的累计”运算。
        - **EN:** CUDA and ROCm code paths differ mainly in the inline assembly or intrinsic spelling, not in the high-level arithmetic contract.
        - **CN:** CUDA 与 ROCm 路径主要区别在于汇编/指令写法，高层算术契约基本一致。

        ## Dependencies / 依赖关系
        - **EN:** Depends on `attention_generic.cuh` for template declarations and on `dtype_float32.cuh` for `Float4_`/`Float8_` accumulator structs.
        - **CN:** 依赖 `attention_generic.cuh` 提供模板声明，并依赖 `dtype_float32.cuh` 提供 `Float4_`/`Float8_` 累计结构。
        - **EN:** Supplies the FP16 specializations consumed by `attention_utils.cuh` and `attention_kernels.cuh` through `attention_dtypes.h`.
        - **CN:** 通过 `attention_dtypes.h` 向 `attention_utils.cuh` 和 `attention_kernels.cuh` 提供 FP16 特化实现。
