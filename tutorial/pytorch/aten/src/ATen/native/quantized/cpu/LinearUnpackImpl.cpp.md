# LinearUnpackImpl.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/LinearUnpackImpl.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU linear algebra or matrix-multiplication support paths in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 线性代数或矩阵乘法支持路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/Context.h>
 4: #include <ATen/cpp_custom_type_hack.h>
 5: #include <ATen/native/quantized/cpu/fbgemm_utils.h>
 6: #include <ATen/native/quantized/PackedParams.h>
 7: #include <ATen/native/quantized/cpu/OnednnUtils.h>
 8: #include <ATen/native/quantized/cpu/QnnpackUtils.h>
 9: #include <torch/custom_class.h>
10: #include <torch/library.h>
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/cpp_custom_type_hack.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/cpp_custom_type_hack.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 12-20
```cpp
12: #ifndef AT_PER_OPERATOR_HEADERS
13: #include <ATen/Functions.h>
14: #include <ATen/NativeFunctions.h>
15: #else
16: #include <ATen/ops/_empty_affine_quantized.h>
17: #include <ATen/ops/_empty_per_channel_affine_quantized.h>
18: #include <ATen/ops/empty.h>
19: #include <ATen/ops/from_blob.h>
20: #endif
```
- EN: This range pulls in required headers, including `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/_empty_affine_quantized.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/_empty_affine_quantized.h`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 22-27
```cpp
22: #ifdef USE_FBGEMM
23: std::tuple<at::Tensor, std::optional<at::Tensor>> PackedLinearWeight::unpack() {
24:   auto packB = w.get();
25:
26:   int64_t N = static_cast<int64_t>(packB->numCols());
27:   int64_t K = static_cast<int64_t>(packB->numRows());
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `unpack`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `unpack`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 29-37
```cpp
29:   at::Tensor weight_origin;
30:   if (q_scheme == c10::kPerTensorAffine) {
31:     weight_origin = at::_empty_affine_quantized(
32:         {N, K}, at::device(c10::kCPU).dtype(c10::kQInt8), w_scale[0], w_zp[0]);
33:   } else if (q_scheme == c10::kPerChannelAffine) {
34:     auto scales = at::from_blob(
35:         w_scale.data(), w_scale.size(), at::device(c10::kCPU).dtype(c10::kFloat));
36:     auto zero_points = at::from_blob(
37:         w_zp.data(), w_zp.size(), at::device(c10::kCPU).dtype(c10::kInt));
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 39-45
```cpp
39:     weight_origin = at::_empty_per_channel_affine_quantized(
40:         {N, K},
41:         scales.toType(c10::kDouble),
42:         zero_points.toType(c10::kLong),
43:         0, // The output channel axis is 0
44:         at::device(c10::kCPU).dtype(c10::kQInt8));
45:   }
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 47-52
```cpp
47:   int8_t* weight_ptr_int8 =
48:       reinterpret_cast<int8_t*>(weight_origin.data_ptr<c10::qint8>());
49:
50:   // packB->printPackedMatrix("packedB inside fbgemm_unpack
51:   // (QLinearUnpackWeightInt8): ");
52:   packB->unpack(weight_ptr_int8);
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 54-60
```cpp
54:   return std::tuple<at::Tensor, std::optional<at::Tensor>>(
55:       weight_origin, bias_);
56: }
57: #endif // USE_FBGEMM
58:
59: #ifdef USE_PYTORCH_QNNPACK
60: std::tuple<at::Tensor, std::optional<at::Tensor>> PackedLinearWeightsQnnp::
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 61-71
```cpp
61:     unpack() {
62:   if (orig_weight.defined()) {
63:     return std::tuple<at::Tensor, std::optional<at::Tensor>>(
64:         orig_weight, bias_);
65:   } else {
66:     // Unpacking requires reverting *make_zero_points_and_scales_tensor*
67:     // function in QnnpackUtils.h Please refer for a detail mechanism.
68:     // https://github.com/pytorch/pytorch/blob/master/aten/src/ATen/native/quantized/cpu/QnnpackUtils.h#L469
69:     // w_scales and w_zero_points are different from original scales & zero
70:     // points with padding & casting etc
71:     at::Tensor weight_origin;
```
- EN: The main symbol in this range is `unpack`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `unpack`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 73-79
```cpp
73:     float* weight_scales_data = w_scales.data_ptr<float>();
74:     if (q_scheme == c10::kPerTensorAffine) {
75:       weight_origin = at::_empty_affine_quantized(
76:           weight_sizes,
77:           at::device(c10::kCPU).dtype(c10::kQInt8),
78:           static_cast<double>(weight_scales_data[0]),
79:           (int64_t)w_zero_points[0] - 128);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 80-87
```cpp
80:     } else if (q_scheme == c10::kPerChannelAffine) {
81:       auto scales = at::from_blob(
82:           weight_scales_data,
83:           w_scales.sizes()[0] - kPaddingChannels,
84:           at::device(c10::kCPU).dtype(c10::kFloat));
85:
86:       at::Tensor zero_points = at::empty(
87:           static_cast<int64_t>(w_zero_points.size() - kPaddingChannels), at::device(c10::kCPU).dtype(c10::kLong));
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 88-95
```cpp
88:       for (const auto i : c10::irange(zero_points.numel())) {
89:         zero_points[i] = ((int64_t)w_zero_points[i] - 128);
90:       }
91:       weight_origin = at::_empty_per_channel_affine_quantized(
92:                           weight_sizes,
93:                           scales,
94:                           zero_points.toType(c10::kLong),
95:                           0, // The output channel axis is 0
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 96-103
```cpp
 96:                           at::device(c10::kCPU).dtype(c10::kQInt8))
 97:                           .contiguous();
 98:     } else {
 99:       TORCH_INTERNAL_ASSERT(false, "Unsupported quantization scheme.");
100:     }
101:     int8_t* weight_ptr_int8 =
102:         reinterpret_cast<int8_t*>(weight_origin.data_ptr<c10::qint8>());
103:     w->unpackWeights(w_zero_points.data(), weight_ptr_int8);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 104-111
```cpp
104:     // See for the subtraction 128
105:     // https://github.com/pytorch/pytorch/blob/master/aten/src/ATen/native/quantized/cpu/qlinear_dynamic.cpp#L319
106:     auto wt_numel = weight_origin.numel();
107:     for (const auto i : c10::irange(wt_numel)) {
108:       weight_ptr_int8[i] = (int8_t)(weight_ptr_int8[i] - 128);
109:     }
110:
111:     return std::tuple<at::Tensor, std::optional<at::Tensor>>(
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 112-120
```cpp
112:         weight_origin, bias_);
113:   }
114: }
115: #endif // USE_PYTORCH_QNNPACK
116:
117: #ifdef USE_FBGEMM
118: std::tuple<at::Tensor, std::optional<at::Tensor>> PackedLinearWeightFp16::
119:     unpack() {
120:   auto& packed_weight_ptr = w;
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `unpack`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `unpack`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 122-129
```cpp
122:   auto nrows = packed_weight_ptr->numRows();
123:   auto ncols = packed_weight_ptr->numCols();
124:
125:   at::Tensor unpacked_weight =
126:       at::empty({ncols, nrows}, at::kHalf, c10::MemoryFormat::Contiguous);
127:   packed_weight_ptr->unpack(
128:       static_cast<fbgemm::float16*>(unpacked_weight.data_ptr()),
129:       fbgemm::matrix_op_t::Transpose);
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 131-137
```cpp
131:   return std::make_tuple(unpacked_weight.to(at::kFloat), bias_);
132: }
133: #endif // USE_FBGEMM
134:
135: #if AT_MKLDNN_ENABLED()
136: std::tuple<at::Tensor, std::optional<at::Tensor>> PackedLinearWeightsOnednn::unpack() {
137:   return std::tuple<at::Tensor, std::optional<at::Tensor>>(
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 138-140
```cpp
138:       orig_weight_, orig_bias_);
139: }
140: #endif // #if AT_MKLDNN_ENABLED()
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Low-level memory access / 底层内存访问
- Namespace-scoped helper structure / 命名空间内辅助结构
- Operator-specific control flow / 算子特定控制流

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/cpp_custom_type_hack.h`, `ATen/native/quantized/cpu/fbgemm_utils.h`, `ATen/native/quantized/PackedParams.h`, `ATen/native/quantized/cpu/OnednnUtils.h`, `ATen/native/quantized/cpu/QnnpackUtils.h`, `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/_empty_affine_quantized.h`
- Standard or third-party headers / 标准库或第三方头文件: `torch/custom_class.h`, `torch/library.h`
- Key helper symbols / 关键辅助符号: `fbgemm`
