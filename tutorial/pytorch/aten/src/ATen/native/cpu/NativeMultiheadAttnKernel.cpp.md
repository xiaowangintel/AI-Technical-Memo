# NativeMultiheadAttnKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/NativeMultiheadAttnKernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares support for Native Multihead Attn Kernel in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了 Native Multihead Attn Kernel 的相关支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/Dispatch.h>
 4: #include <ATen/OpMathType.h>
 5: #include <ATen/Parallel.h>
 6: #include <ATen/TensorIterator.h>
 7: #include <ATen/cpu/vec/vec.h>
 8: #include <ATen/cpu/vec/functional.h>
 9: #include <ATen/native/cpu/utils.h>
10: #include <ATen/native/transformers/attention.h>
11: #include <c10/util/irange.h>
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/OpMathType.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/OpMathType.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 13-19
```cpp
13: namespace at::native {
14:
15: namespace {
16:
17: template <typename scalar_t>
18: void cpu_transform_bias_rescale_qkv(
19:     scalar_t* q_k_v_data,
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 20-27
```cpp
20:     const scalar_t* qkv_data,
21:     const scalar_t* qkv_bias_data,
22:     int64_t B,
23:     int64_t T,
24:     int64_t D,
25:     int64_t num_head) {
26:
27:   int64_t dim_per_head = D / num_head;
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 29-36
```cpp
29:   // shapes and strides:
30:   //   qkv      : {B, T, 3, num_head, dim_per_head}
31:   //   qkv_bias : {3, num_head, dim_per_head}
32:   //   q_k_v    : {3, B, num_head, T, dim_per_head}
33:   //
34:   int64_t i_strideB = T * 3 * D;
35:   int64_t i_strideT = 3 * D;
36:   int64_t o_stride = B * num_head * T * dim_per_head;
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 38-47
```cpp
38:   // inv_sqrt_dim_per_head in accumulate type
39:   using acc_t = at::opmath_type<scalar_t>;
40:   using Vec =  vec::Vectorized<acc_t>;
41:   const acc_t s = 1.0 / std::sqrt(static_cast<acc_t>(dim_per_head));
42:
43:   // parallel on {B, num_head, T}
44:   int64_t grain_size = std::max(at::internal::GRAIN_SIZE / (3 * dim_per_head), (int64_t)1);
45:   at::parallel_for(0, B * num_head * T, grain_size, [&](int64_t begin, int64_t end) {
46:     int64_t b{0}, nh{0}, t{0};
47:     data_index_init(begin, b, B, nh, num_head, t, T);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。

### Lines 49-56
```cpp
49:     for (const auto i : c10::irange(begin, end)) {
50:       const scalar_t* q_in_ptr = qkv_data + b * i_strideB + t * i_strideT + 0 * D + nh * dim_per_head;
51:       const scalar_t* k_in_ptr = qkv_data + b * i_strideB + t * i_strideT + 1 * D + nh * dim_per_head;
52:       const scalar_t* v_in_ptr = qkv_data + b * i_strideB + t * i_strideT + 2 * D + nh * dim_per_head;
53:
54:       const scalar_t* q_bias_ptr = qkv_bias_data + 0 * D + nh * dim_per_head;
55:       const scalar_t* k_bias_ptr = qkv_bias_data + 1 * D + nh * dim_per_head;
56:       const scalar_t* v_bias_ptr = qkv_bias_data + 2 * D + nh * dim_per_head;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 58-66
```cpp
58:       // we can use global index i here for output
59:       scalar_t* q_out_ptr = q_k_v_data + 0 * o_stride + i * dim_per_head;
60:       scalar_t* k_out_ptr = q_k_v_data + 1 * o_stride + i * dim_per_head;
61:       scalar_t* v_out_ptr = q_k_v_data + 2 * o_stride + i * dim_per_head;
62:
63:       // q = (q + bias) * inv_sqrt_dim_per_head
64:       vec::map2<scalar_t>(
65:           [s](Vec q, Vec q_bias) { return (q + q_bias) * Vec(s); },
66:           q_out_ptr, q_in_ptr, q_bias_ptr, dim_per_head);
```
- EN: The main symbol in this range is `map2<scalar_t>`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `map2<scalar_t>`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 68-74
```cpp
68:       // k = k + bias
69:       vec::map2<scalar_t>([](Vec k, Vec k_bias) { return k + k_bias; },
70:           k_out_ptr, k_in_ptr, k_bias_ptr, dim_per_head);
71:
72:       // v = v + bias
73:       vec::map2<scalar_t>([](Vec v, Vec v_bias) { return v + v_bias; },
74:           v_out_ptr, v_in_ptr, v_bias_ptr, dim_per_head);
```
- EN: The main symbol in this range is `map2<scalar_t>`, which contributes directly to this file's operator logic. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `map2<scalar_t>`，它们直接构成本文件的算子逻辑。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 76-82
```cpp
76:       // move to the next index
77:       data_index_step(b, B, nh, num_head, t, T);
78:     }
79:   });
80: }
81:
82: void transform_bias_rescale_qkv_kernel_impl(
```
- EN: The main symbol in this range is `transform_bias_rescale_qkv_kernel_impl`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `transform_bias_rescale_qkv_kernel_impl`，它们直接构成本文件的算子逻辑。

### Lines 83-90
```cpp
83:     at::ScalarType type,
84:     void* _q_k_v,
85:     const void* _qkv,
86:     const void* _qkv_bias,
87:     int64_t B,
88:     int64_t T,
89:     int64_t D,
90:     int64_t num_head) {
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 92-98
```cpp
92:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, type, "transform_bias_rescale_qkv", [&] {
93:     scalar_t* q_k_v = static_cast<scalar_t*>(_q_k_v);
94:     const scalar_t* qkv = static_cast<const scalar_t*>(_qkv);
95:     const scalar_t* qkv_bias = static_cast<const scalar_t*>(_qkv_bias);
96:     cpu_transform_bias_rescale_qkv<scalar_t>(
97:         q_k_v,
98:         qkv,
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 99-107
```cpp
 99:         qkv_bias,
100:         B,
101:         T,
102:         D,
103:         num_head);
104:   });
105: }
106:
107: } // anonymous namespace
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 109-111
```cpp
109: REGISTER_DISPATCH(transform_bias_rescale_qkv_stub, &transform_bias_rescale_qkv_kernel_impl)
110:
111: } // at::native
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Dispatcher registration / 调度器注册
- Scalar and dtype abstractions / 标量与数据类型抽象

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/OpMathType.h`, `ATen/Parallel.h`, `ATen/TensorIterator.h`, `ATen/cpu/vec/vec.h`, `ATen/cpu/vec/functional.h`, `ATen/native/cpu/utils.h`, `ATen/native/transformers/attention.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `Vectorized`, `parallel_for`, `REGISTER_DISPATCH`, `AT_DISPATCH_FLOATING_TYPES`, `Scalar`, `ScalarType`
