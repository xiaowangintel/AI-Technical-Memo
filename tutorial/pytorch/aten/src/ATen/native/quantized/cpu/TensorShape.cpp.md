# TensorShape.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/TensorShape.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares quantized CPU operator implementations, scale/zero-point handling, and packed weights in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了量化 CPU 算子实现、scale/zero-point 处理与打包权重逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/core/List.h>
 4: #include <ATen/Dispatch.h>
 5: #include <ATen/WrapDimUtils.h>
 6: #include <ATen/core/IListRef.h>
 7: #include <ATen/native/cpu/Loops.h>
 8: #include <ATen/native/quantized/cpu/QuantizedOps.h>
 9: #include <ATen/native/TensorIterator.h>
10: #include <ATen/native/TensorShape.h>
11: #include <c10/util/irange.h>
12: #include <torch/library.h>
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/core/List.h`, `ATen/Dispatch.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/core/List.h`, `ATen/Dispatch.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 14-26
```cpp
14: #ifndef AT_PER_OPERATOR_HEADERS
15: #include <ATen/Functions.h>
16: #include <ATen/NativeFunctions.h>
17: #else
18: #include <ATen/ops/cat.h>
19: #include <ATen/ops/cat_native.h>
20: #include <ATen/ops/copy_native.h>
21: #include <ATen/ops/quantize_per_tensor.h>
22: #include <ATen/ops/zeros_like_ops.h>
23: #endif
24:
25: #include <algorithm>
26: #include <vector>
```
- EN: This range pulls in required headers, including `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/cat.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/cat.h`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 28-38
```cpp
28: namespace at::native {
29:
30: DEFINE_DISPATCH(qcat_nhwc_stub);
31: DEFINE_DISPATCH(qcat_relu_nhwc_stub);
32:
33: namespace {
34:
35: bool is_cat_nhwc_fast_path(const MaterializedITensorListRef& qxs, int64_t dim) {
36:   TORCH_CHECK(!qxs.empty());
37:   bool is_fast_path = dim == 1;
38:   // NOLINTNEXTLINE(performance-implicit-conversion-in-loop)
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `is_cat_nhwc_fast_path`, which contributes directly to this file's operator logic. The math and shape handling relate to convolution-style operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `is_cat_nhwc_fast_path`，它们直接构成本文件的算子逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 39-51
```cpp
39:   for (const at::Tensor& qx : qxs) {
40:     is_fast_path &= qx.dim() == 4;
41:     is_fast_path &= qx.is_contiguous(c10::MemoryFormat::ChannelsLast);
42:   }
43:   return is_fast_path;
44: }
45:
46: bool is_valid_quantization_scheme(const Tensor& t) {
47:   const auto qtype = t.qscheme();
48:   return (qtype == kPerTensorAffine) || (qtype == kPerTensorSymmetric);
49: }
50:
51: #define QPARAM_THRESHOLD 1e-04
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `is_valid_quantization_scheme`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `is_valid_quantization_scheme`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 53-63
```cpp
53: bool all_inputs_sharing_qparams(const MaterializedITensorListRef& qxs) {
54:   bool is_valid = true;
55:   for (const auto i : c10::irange(1, qxs.size())) {
56:     is_valid &= qxs[0].get().is_quantized();
57:     is_valid &= qxs[i].get().is_quantized() == qxs[0].get().is_quantized();
58:     is_valid &= qxs[i].get().qscheme() == qxs[0].get().qscheme();
59:     is_valid &= qxs[i].get().dtype() == qxs[0].get().dtype();
60:     if (qxs[0].get().qscheme() == kPerTensorAffine) {
61:         is_valid &= fabs(qxs[i].get().q_scale() - qxs[0].get().q_scale()) < QPARAM_THRESHOLD;
62:       is_valid &= qxs[i].get().q_zero_point() == qxs[0].get().q_zero_point();
63:     } else if (qxs[0].get().qscheme() == kPerChannelAffine) {
```
- EN: The main symbol in this range is `all_inputs_sharing_qparams`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `all_inputs_sharing_qparams`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 64-75
```cpp
64:         is_valid &= qxs[i].get().q_per_channel_scales().isclose(qxs[0].get().q_per_channel_scales(), 0, QPARAM_THRESHOLD, false).all().item().to<bool>();
65:       is_valid &= qxs[i].get().q_per_channel_zero_points().equal(qxs[0].get().q_per_channel_zero_points());
66:     } else {
67:         TORCH_CHECK(false, "Unrecognized qscheme:", toString(qxs[0].get().qscheme()));
68:     }
69:   }
70:   return is_valid;
71: }
72:
73: /* Quantized concatenation.
74:  *
75:  * Note: This function uses a dequantization.
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 76-89
```cpp
76:  */
77: template <bool ReLUFused>
78: Tensor quantized_cat_impl(
79:     const MaterializedITensorListRef& qxs,
80:     int64_t dim,
81:     double scale,
82:     int64_t zero_point) {
83:   if (is_cat_nhwc_fast_path(qxs, dim)) {
84:     if (ReLUFused) {
85:       return qcat_relu_nhwc_stub(at::kCPU, qxs, dim, scale, zero_point);
86:     } else {
87:       return qcat_nhwc_stub(at::kCPU, qxs, dim, scale, zero_point);
88:     }
89:   }
```
- EN: The main symbol in this range is `quantized_cat_impl`, `qcat_relu_nhwc_stub`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `quantized_cat_impl`, `qcat_relu_nhwc_stub`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 91-101
```cpp
 91:   const auto x_dtype = qxs[0].get().scalar_type();
 92:   const auto x_qscheme = qxs[0].get().qscheme();
 93:   std::vector<Tensor> xs;
 94:   xs.reserve(qxs.size());
 95:   // NOLINTNEXTLINE(performance-implicit-conversion-in-loop)
 96:   for (const at::Tensor& qx : qxs) {
 97:     TORCH_CHECK(x_dtype == qx.scalar_type(), "All dtypes must be the same.");
 98:     TORCH_CHECK(
 99:         x_qscheme == qx.qscheme(), "Quantization schemes must be the same.");
100:     xs.push_back(qx.dequantize());
101:   }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 102-114
```cpp
102:   const Tensor y = at::cat(xs, dim);
103:   Tensor qy;
104:   AT_DISPATCH_QINT_TYPES(x_dtype, "qcat", [&]() {
105:     qy = at::quantize_per_tensor(y, scale, zero_point, SCALAR_TYPE);
106:     if (ReLUFused) {
107:       auto iter = TensorIterator::unary_op(qy, qy);
108:       cpu_kernel(iter, [&](scalar_t value) -> scalar_t {
109:         return scalar_t(std::max<underlying_t>(value.val_, zero_point));
110:       });
111:     }
112:   });
113:   return qy;
114: }
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 116-126
```cpp
116: template <bool ReLUFused>
117: Tensor quantized_cat_impl(
118:     ITensorListRef qxs,
119:     int64_t dim,
120:     double scale,
121:     int64_t zero_point) {
122:   return quantized_cat_impl<ReLUFused>(qxs.materialize(), dim, scale, zero_point);
123: }
124:
125: template <bool ReLUFused = false>
126: Tensor qcat(
```
- EN: The main symbol in this range is `quantized_cat_impl`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `quantized_cat_impl`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 127-137
```cpp
127:     const c10::List<Tensor>& qxs,
128:     int64_t dim,
129:     std::optional<double> scale,
130:     std::optional<int64_t> zero_point) {
131:   TORCH_CHECK(is_valid_quantization_scheme(qxs[0]),
132:               "Only per-tensor quantization is supported in 'cat'!")
133:   double _scale = scale.has_value() ? scale.value() : qxs.get(0).q_scale();
134:   int64_t _zero_point =
135:       zero_point.has_value() ? zero_point.value() : qxs.get(0).q_zero_point();
136:   return quantized_cat_impl<ReLUFused>(qxs, dim, _scale, _zero_point);
137: }
```
- EN: The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 139-149
```cpp
139: template <bool ReLUFused = false>
140: Tensor qcat_out(const c10::List<Tensor>& qxs, int64_t dim, Tensor out) {
141:   TORCH_CHECK(is_valid_quantization_scheme(qxs[0]),
142:               "Only per-tensor quantization is supported in 'cat'!")
143:   TORCH_CHECK(is_valid_quantization_scheme(out),
144:               "Only per-tensor quantization is supported in 'cat'!")
145:   auto out_ =
146:       quantized_cat_impl<ReLUFused>(qxs, dim, out.q_scale(), out.q_zero_point());
147:   at::native::copy_(out, out_, /*non_blocking=*/false);
148:   return out;
149: }
```
- EN: The main symbol in this range is `qcat_out`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `qcat_out`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 151-163
```cpp
151: } // namespace
152:
153: TORCH_LIBRARY_IMPL(quantized, QuantizedCPU, m) {
154:   m.impl(TORCH_SELECTIVE_NAME("quantized::cat"), TORCH_FN(qcat<false>));
155:   m.impl(TORCH_SELECTIVE_NAME("quantized::cat_relu"), TORCH_FN(qcat<true>));
156:   m.impl(TORCH_SELECTIVE_NAME("quantized::cat_out"), TORCH_FN(qcat_out<false>));
157:   m.impl(TORCH_SELECTIVE_NAME("quantized::cat_relu_out"), TORCH_FN(qcat_out<true>));
158: }
159:
160: Tensor cat_quantized_cpu(const ITensorListRef& qxs, int64_t dim) {
161:   auto materialized = qxs.materialize();
162:   TORCH_CHECK(is_valid_quantization_scheme(materialized[0]),
163:               "Only per-tensor quantization is supported in 'cat'!");
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `cat_quantized_cpu`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `cat_quantized_cpu`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 165-174
```cpp
165:   if (!all_inputs_sharing_qparams(materialized)) {
166:       // TODO: if possible change this warning to an error T194501002
167:       TORCH_WARN("All inputs of this cat operator must share the same quantization parameters. Otherwise large numerical inaccuracies may occur.");
168:   }
169:   check_cat_no_zero_dim(materialized);
170:   dim = legacy_cat_wrap_dim(dim, materialized);
171:   double _scale = materialized[0].get().q_scale();
172:   int64_t _zero_point = materialized[0].get().q_zero_point();
173:   return quantized_cat_impl<false>(materialized, dim, _scale, _zero_point);
174: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 176-187
```cpp
176: Tensor& cat_out_quantized_cpu(const ITensorListRef& qxs, int64_t dim, Tensor& out) {
177:   auto materialized = qxs.materialize();
178:   TORCH_CHECK(is_valid_quantization_scheme(materialized[0]),
179:               "Only per-tensor quantization is supported in 'cat'!")
180:   TORCH_CHECK(is_valid_quantization_scheme(out),
181:               "Only per-tensor quantization is supported in 'cat'!")
182:   check_cat_no_zero_dim(materialized);
183:   dim = legacy_cat_wrap_dim(dim, materialized);
184:   auto out_ = quantized_cat_impl<false>(qxs, dim, out.q_scale(), out.q_zero_point());
185:   at::native::copy_(out, out_, /*non_blocking=*/false);
186:   return out;
187: }
```
- EN: The main symbol in this range is `cat_out_quantized_cpu`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `cat_out_quantized_cpu`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 189-189
```cpp
189: }  // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Dispatcher registration / 调度器注册
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/core/List.h`, `ATen/Dispatch.h`, `ATen/WrapDimUtils.h`, `ATen/core/IListRef.h`, `ATen/native/cpu/Loops.h`, `ATen/native/quantized/cpu/QuantizedOps.h`, `ATen/native/TensorIterator.h`, `ATen/native/TensorShape.h`, `ATen/Functions.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `torch/library.h`, `algorithm`, `vector`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `cpu_kernel`
