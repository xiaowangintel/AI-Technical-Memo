# qmatmul.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/qmatmul.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU linear algebra or matrix-multiplication support paths in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 线性代数或矩阵乘法支持路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: #include <ATen/ATen.h>
 2: #include <torch/library.h>
 3:
 4: #ifdef USE_RUY_QMATMUL
 5: #include <ATen/Parallel.h>
 6: #include <ATen/native/quantized/cpu/RuyUtils.h>
 7: #include <ruy/ruy.h>
 8: #endif
 9:
10: namespace at::native {
11:
12: namespace {
```
- EN: This range pulls in required headers, including `ATen/ATen.h`, `torch/library.h`, `ATen/Parallel.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/ATen.h`, `torch/library.h`, `ATen/Parallel.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 14-27
```cpp
14: inline void check_inputs(const Tensor& qa, const Tensor& qb) {
15:   TORCH_CHECK(
16:       qa.scalar_type() == c10::kQInt8 || qa.scalar_type() == c10::kQUInt8,
17:       "MatMul operands should use QInt8 or QUInt8 data types.");
18:   TORCH_CHECK(
19:       qa.scalar_type() == qb.scalar_type(),
20:       "MatMul operands should have same data type.");
21:   TORCH_CHECK(
22:       qa.qscheme() == kPerTensorAffine || qa.qscheme() == kPerTensorSymmetric,
23:       "Only per-tensor quantization is supported in Matmul.");
24:   TORCH_CHECK(
25:       qa.qscheme() == qb.qscheme(),
26:       "Both inputs to Matmul must have the same quantization scheme.");
27: }
```
- EN: The main symbol in this range is `check_inputs`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `check_inputs`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 29-39
```cpp
29: #ifdef USE_RUY_QMATMUL
30:
31: Tensor qmatmul(
32:     const Tensor& qa,
33:     const Tensor& qb,
34:     const double output_scale,
35:     const int64_t output_zero_point) {
36:   check_inputs(qa, qb);
37:
38:   const int64_t num_dims = qa.dim();
39:   const int64_t b_num_dims = qb.dim();
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `qmatmul`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `qmatmul`，它们直接构成本文件的算子逻辑。

### Lines 41-53
```cpp
41:   TORCH_CHECK(
42:       num_dims == b_num_dims,
43:       "MatMul operands should have the same dimensionality. (", num_dims,
44:       " and ", b_num_dims, " provided)");
45:   TORCH_CHECK(
46:       num_dims >= 2,
47:       "Quantized Matmul currently only supports operands which are at least 2-dimensional. (",
48:       num_dims, " provided)");
49:
50:   const int64_t m = qa.size(num_dims - 2);
51:   const int64_t k = qa.size(num_dims - 1);
52:   const int64_t b_k = qb.size(num_dims - 2);
53:   const int64_t n = qb.size(num_dims - 1);
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 55-65
```cpp
55:   TORCH_CHECK(
56:       b_k == k,
57:       "For Quantized Matmul, the size of tensor a (", k,
58:       ") at dimension ", num_dims - 1, " must match the size of tensor b (",
59:       b_k, ") at dimension ", num_dims - 2, ".");
60:
61:   std::vector<int64_t> out_size_vec(num_dims);
62:   size_t num_matmuls = 1;
63:   for (int64_t i = 0; i < num_dims - 2; i++) {
64:     const int64_t dim = qa.size(i);
65:     const int64_t qb_dim = qb.size(i);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 67-77
```cpp
67:     TORCH_CHECK(
68:         dim == qb_dim,
69:         "For Quantized Matmul, the size of tensor a (", dim,
70:         ") must match the size of tensor b (", qb_dim,
71:         ") at dimension ", i);
72:
73:     out_size_vec[i] = dim;
74:     num_matmuls *= dim;
75:   }
76:   out_size_vec[num_dims - 2] = m;
77:   out_size_vec[num_dims - 1] = n;
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 79-89
```cpp
79:   Tensor out = at::_empty_affine_quantized(
80:       IntArrayRef(out_size_vec),
81:       at::device(kCPU)
82:           .dtype(qa.scalar_type())
83:           .memory_format(qa.suggest_memory_format()),
84:       output_scale,
85:       output_zero_point,
86:       std::nullopt);
87:
88:   const Tensor& qa_contig = qa.contiguous();
89:   const Tensor& qb_contig = qb.contiguous();
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 91-103
```cpp
 91:   AT_DISPATCH_QINT_BYTE_TYPES(qa.scalar_type(), "qmatmul", [&] {
 92:     using underlying_t = typename scalar_t::underlying;
 93:
 94:     const underlying_t* qa_data = reinterpret_cast<const underlying_t*>(
 95:         qa_contig.data_ptr<scalar_t>());
 96:     const underlying_t* qb_data = reinterpret_cast<const underlying_t*>(
 97:         qb_contig.data_ptr<scalar_t>());
 98:     underlying_t* out_data =
 99:         reinterpret_cast<underlying_t*>(out.data_ptr<scalar_t>());
100:
101:     const size_t qa_stride = m * k;
102:     const size_t qb_stride = k * n;
103:     const size_t out_stride = m * n;
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 105-115
```cpp
105:     auto matmuls = [&](int64_t begin, int64_t end) {
106:
107:       ruy::Matrix<underlying_t> qa_matrix;
108:       ruy::MakeSimpleLayout(
109:           m, k, ruy::Order::kRowMajor, qa_matrix.mutable_layout());
110:       qa_matrix.set_zero_point(qa.q_zero_point());
111:
112:       ruy::Matrix<underlying_t> qb_matrix;
113:       ruy::MakeSimpleLayout(
114:           k, n, ruy::Order::kRowMajor, qb_matrix.mutable_layout());
115:       qb_matrix.set_zero_point(qb.q_zero_point());
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 117-127
```cpp
117:       ruy::Matrix<underlying_t> out_matrix;
118:       ruy::MakeSimpleLayout(
119:           m, n, ruy::Order::kRowMajor, out_matrix.mutable_layout());
120:       out_matrix.set_zero_point(output_zero_point);
121:
122:       // Requantization explanation:
123:       // https://github.com/google/gemmlowp/blob/e844ffd17118c1e17d94e1ba4354c075a4577b88/doc/quantization.md
124:       const double requantization_scale_inv =
125:           (qa.q_scale() * qb.q_scale()) / output_scale;
126:
127:       ruy::MulParams<int32_t, underlying_t> mul_params;
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 129-139
```cpp
129:       int multiplier_fixedpoint;
130:       int multiplier_exponent;
131:       ruy_utils::quantize_multiplier(requantization_scale_inv,
132:                                      &multiplier_fixedpoint,
133:                                      &multiplier_exponent);
134:       mul_params.set_multiplier_fixedpoint(multiplier_fixedpoint);
135:       mul_params.set_multiplier_exponent(multiplier_exponent);
136:
137:       const underlying_t* qa_subtensor = qa_data + begin * qa_stride;
138:       const underlying_t* qb_subtensor = qb_data + begin * qb_stride;
139:       underlying_t* out_subtensor = out_data + begin * out_stride;
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 141-155
```cpp
141:       for (int64_t i = begin; i < end; i++) {
142:         qa_matrix.set_data(qa_subtensor);
143:         qb_matrix.set_data(qb_subtensor);
144:         out_matrix.set_data(out_subtensor);
145:         ruy::Mul(qa_matrix,
146:                  qb_matrix,
147:                  mul_params,
148:                  ruy_utils::get_ruy_context(),
149:                  &out_matrix);
150:
151:         qa_subtensor += qa_stride;
152:         qb_subtensor += qb_stride;
153:         out_subtensor += out_stride;
154:       }
155:     };
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 157-167
```cpp
157:     at::parallel_for(0, num_matmuls, 1, matmuls);
158:   });
159:
160:   return out;
161: }
162:
163: #else // ifdef USE_RUY_QMATMUL
164:
165: Tensor qmatmul(
166:     const Tensor& qa,
167:     const Tensor& qb,
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 168-182
```cpp
168:     const double output_scale,
169:     const int64_t output_zero_point) {
170:   check_inputs(qa, qb);
171:   Tensor ra = at::dequantize(qa);
172:   Tensor rb = at::dequantize(qb);
173:   Tensor rc = at::matmul(ra, rb);
174:   return at::quantize_per_tensor(
175:       rc, output_scale, output_zero_point, qa.scalar_type());
176: }
177:
178: #endif // ifdef USE_RUY_QMATMUL
179:
180: TORCH_LIBRARY_IMPL(quantized, QuantizedCPU, m) {
181:   m.impl(TORCH_SELECTIVE_NAME("quantized::matmul"), TORCH_FN(qmatmul));
182: }
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 184-186
```cpp
184: } // namespace
185:
186: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- CPU parallelism / CPU 并行
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/ATen.h`, `ATen/Parallel.h`, `ATen/native/quantized/cpu/RuyUtils.h`
- Standard or third-party headers / 标准库或第三方头文件: `torch/library.h`, `ruy/ruy.h`
- Key helper symbols / 关键辅助符号: `parallel_for`
