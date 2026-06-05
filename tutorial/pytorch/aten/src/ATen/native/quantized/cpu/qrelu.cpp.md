# qrelu.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/qrelu.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares native CPU activation kernels, dtype dispatch, and vectorized elementwise execution paths in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了原生 CPU 激活函数 kernel、数据类型分派以及向量化逐元素执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/Context.h>
 4: #include <ATen/Dispatch.h>
 5: #include <ATen/TensorIterator.h>
 6: #include <ATen/native/cpu/Loops.h>
 7: #include <ATen/native/quantized/AffineQuantizer.h>
 8: #include <ATen/native/quantized/cpu/init_qnnpack.h>
 9: #include <ATen/native/quantized/cpu/QnnpackUtils.h>
10: #include <ATen/native/quantized/cpu/QuantizedOps.h>
11: #include <c10/util/irange.h>
12: #include <caffe2/utils/threadpool/pthreadpool-cpp.h>
13: #include <torch/library.h>
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/Dispatch.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. TensorIterator appears here to standardize elementwise traversal and broadcasting behavior.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/Dispatch.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这里使用 TensorIterator 来统一逐元素遍历与广播行为。

### Lines 15-27
```cpp
15: #ifndef AT_PER_OPERATOR_HEADERS
16: #include <ATen/Functions.h>
17: #include <ATen/NativeFunctions.h>
18: #else
19: #include <ATen/ops/_empty_affine_quantized.h>
20: #include <ATen/ops/_prelu_kernel_native.h>
21: #include <ATen/ops/hardtanh_native.h>
22: #include <ATen/ops/leaky_relu_native.h>
23: #include <ATen/ops/prelu.h>
24: #include <ATen/ops/prelu_native.h>
25: #include <ATen/ops/quantize_per_tensor.h>
26: #include <ATen/ops/relu_native.h>
27: #endif
```
- EN: This range pulls in required headers, including `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/_empty_affine_quantized.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/_empty_affine_quantized.h`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 29-39
```cpp
29: #include <algorithm>
30:
31: namespace at::native {
32:
33: DEFINE_DISPATCH(qrelu_stub);
34: DEFINE_DISPATCH(qrelu_leaky_stub);
35: DEFINE_DISPATCH(qprelu_stub);
36:
37: #ifdef USE_PYTORCH_QNNPACK
38: static Tensor qnnpack_relu(Tensor input) {
39:   Tensor qy;
```
- EN: This range pulls in required headers, including `algorithm`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `algorithm`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 40-52
```cpp
40:   TORCH_CHECK(
41:       input.ndimension() > 0, "qnnpack_relu(): Got empty input tensor");
42:   TORCH_CHECK(input.scalar_type() == c10::kQUInt8,
43:                "qnnpack_relu(): Expected input data type ",
44:                toString(c10::kQUInt8),
45:                " but got ",
46:                toString(input.scalar_type()));
47:
48:   Tensor input_contig = input.contiguous(input.suggest_memory_format());
49:
50:   const auto zero_point = input_contig.q_zero_point();
51:
52:   initQNNPACK();
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 54-66
```cpp
54:   size_t num_elems = 1;
55:   for (const auto i : c10::irange(1, input_contig.ndimension())) {
56:     num_elems *= input_contig.size(i);
57:   }
58:
59:   pytorch_qnnp_operator_t qnnpack_operator{nullptr};
60:
61:   const pytorch_qnnp_status createStatus = pytorch_qnnp_create_clamp_nc_u8(
62:       num_elems /* channels */,
63:       zero_point /* output min */,
64:       std::numeric_limits<uint8_t>::max() /* output max */,
65:       0 /* flags */,
66:       &qnnpack_operator);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 68-80
```cpp
68:   std::unique_ptr<pytorch_qnnp_operator, QnnpackOperatorDeleter>
69:       qnnpack_uniq_ptr(qnnpack_operator);
70:
71:   TORCH_INTERNAL_ASSERT(
72:       createStatus == pytorch_qnnp_status_success,
73:       "failed to create QNNPACK Relu operator");
74:
75:   qy = at::_empty_affine_quantized(
76:       input_contig.sizes(),
77:       at::device(kCPU).dtype(input.scalar_type()),
78:       input_contig.q_scale(),
79:       input_contig.q_zero_point(),
80:       input.suggest_memory_format());
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 82-93
```cpp
82:   const pytorch_qnnp_status setupStatus = pytorch_qnnp_setup_clamp_nc_u8(
83:       qnnpack_operator, /* clamp */
84:       input_contig.size(0) /* batch size */,
85:       (uint8_t*)input_contig.data_ptr<c10::quint8>() /* input data */,
86:       num_elems /* input stride */,
87:       (uint8_t*)qy.data_ptr<c10::quint8>() /* output data */,
88:       num_elems /* output stride */);
89:   TORCH_INTERNAL_ASSERT(
90:       setupStatus == pytorch_qnnp_status_success,
91:       "failed to setup QNNPACK Relu operator");
92:
93:   pthreadpool_t threadpool = caffe2::pthreadpool_();
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range. The code participates in pooling window traversal or pooled-output shape computation.
- CN: 这一段涉及量化数据处理或打包参数逻辑。 这段代码参与池化窗口遍历或池化输出形状计算。

### Lines 95-105
```cpp
 95:   const pytorch_qnnp_status runStatus =
 96:       pytorch_qnnp_run_operator(qnnpack_operator, threadpool);
 97:
 98:   TORCH_INTERNAL_ASSERT(
 99:       runStatus == pytorch_qnnp_status_success,
100:       "failed to run QNNPACK Relu operator");
101:   return qy;
102: }
103: #endif
104:
105: Tensor relu_quantized_cpu(const Tensor& qx) {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `relu_quantized_cpu`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `relu_quantized_cpu`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 106-117
```cpp
106:   #ifdef USE_PYTORCH_QNNPACK
107:   if (at::globalContext().qEngine() == at::QEngine::QNNPACK && qx.scalar_type() == kQUInt8) {
108:     return qnnpack_relu(qx);
109:   }
110:   #endif
111:   Tensor qy;
112:   qrelu_stub(qx.device().type(), qx, qy);
113:   return qy;
114: }
115: Tensor& relu_quantized_cpu_(Tensor& qx) {
116:   const auto zero_point = qx.q_zero_point();
117:   AT_DISPATCH_QINT_TYPES(qx.scalar_type(), "qrelu", [&]() {
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `relu_quantized_cpu_`, `qrelu_stub`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `relu_quantized_cpu_`, `qrelu_stub`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 118-129
```cpp
118:     using Vec = Vectorized<scalar_t>;
119:     auto iter = TensorIterator::unary_op(qx, qx);
120:     auto zero_point_vec = Vec(scalar_t(zero_point));
121:     cpu_kernel_vec(
122:         iter,
123:         [&](scalar_t value) -> scalar_t {
124:           return scalar_t(std::max<underlying_t>(value.val_, zero_point));
125:         },
126:         [&](Vec value) -> Vec { return value.relu(zero_point_vec); });
127:   });
128:   return qx;
129: }
```
- EN: TensorIterator appears here to standardize elementwise traversal and broadcasting behavior. Vectorized/SIMD execution is used to process multiple values per iteration. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这里使用 TensorIterator 来统一逐元素遍历与广播行为。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 131-146
```cpp
131: Tensor& leaky_relu_out_quantized_cpu(const Tensor& self,
132:                                  const Scalar& negval, Tensor& result) {
133:   qrelu_leaky_stub(self.device().type(), result, self, negval);
134:   return result;
135: }
136:
137: Tensor leaky_relu_quantized_cpu(const Tensor& self, const Scalar& negval) {
138:   const auto qx = self.contiguous(self.suggest_memory_format());
139:   auto qy = at::_empty_affine_quantized(qx.sizes(),
140:       at::device(kCPU).dtype(self.scalar_type()),
141:       qx.q_scale(),
142:       qx.q_zero_point(),
143:       self.suggest_memory_format());
144:   qrelu_leaky_stub(self.device().type(), qy, qx, negval);
145:   return qy;
146: }
```
- EN: The main symbol in this range is `leaky_relu_out_quantized_cpu`, `leaky_relu_quantized_cpu`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `leaky_relu_out_quantized_cpu`, `leaky_relu_quantized_cpu`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 148-161
```cpp
148: Tensor& leaky_relu_quantized_cpu_(Tensor& self, const Scalar& negval) {
149:   qrelu_leaky_stub(self.device().type(), self, self, negval);
150:   return self;
151: }
152:
153: static Tensor _prelu_kernel_quantized_cpu_impl(const Tensor& self, const Tensor& weight,
154:                                 double output_scale, int64_t output_zero_point) {
155:   auto ndim = self.dim();
156:   // for ndim < 1 or > 5, go to reference path
157:   if (ndim > 5 || ndim < 1) {
158:     auto x = self.dequantize();
159:     auto y = at::prelu(x, weight);
160:     return at::quantize_per_tensor(y, output_scale, output_zero_point, c10::kQUInt8);
161:   }
```
- EN: The main symbol in this range is `leaky_relu_quantized_cpu_`, `_prelu_kernel_quantized_cpu_impl`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `leaky_relu_quantized_cpu_`, `_prelu_kernel_quantized_cpu_impl`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 163-173
```cpp
163:   auto qy = at::_empty_affine_quantized(self.sizes(),
164:       at::device(kCPU)
165:         .dtype(self.scalar_type()),
166:       output_scale,
167:       output_zero_point,
168:       self.suggest_memory_format());
169:
170:   qprelu_stub(self.device().type(), qy, self, weight);
171:
172:   return qy;
173: }
```
- EN: The main symbol in this range is `qprelu_stub`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `qprelu_stub`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 175-189
```cpp
175: Tensor _prelu_kernel_quantized_cpu(const Tensor& self, const Tensor& weight) {
176:   return _prelu_kernel_quantized_cpu_impl(self, weight, self.q_scale(), self.q_zero_point());
177: }
178:
179: namespace {
180: Tensor quantized_relu6(const Tensor& qx) {
181:   Tensor qy;
182:   qy = hardtanh_quantized_cpu(qx, 0.0f, 6.0f);
183:   return qy;
184: }
185:
186: Tensor quantized_relu6_(Tensor& qx) {
187:   hardtanh_quantized_cpu_(qx, 0.0f, 6.0f);
188:   return qx;
189: }
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `_prelu_kernel_quantized_cpu`, `quantized_relu6`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `_prelu_kernel_quantized_cpu`, `quantized_relu6`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 191-200
```cpp
191: class QRelu6 final {
192:  public:
193:   static Tensor run(Tensor qx, bool inplace) {
194:     if (inplace) {
195:       return quantized_relu6_(qx);
196:     } else {
197:       return quantized_relu6(qx);
198:     }
199:   }
200: };
```
- EN: The main symbol in this range is `run`, `QRelu6`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `run`, `QRelu6`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 202-212
```cpp
202: class QLeakyRelu final {
203:  public:
204:   static Tensor run(Tensor self, const Scalar& negative_slope, bool inplace, double output_scale, int64_t output_zero_point) {
205:     // inplace argument is ignored now, TODO:support inplace
206:     if (inplace) {
207:       TORCH_WARN("inplace=True is not supported for quantized::leaky_relu yet");
208:     }
209:     const auto qx = self.contiguous(self.suggest_memory_format());
210:     auto qy = at::_empty_affine_quantized(qx.sizes(),
211:       at::device(kCPU).dtype(self.scalar_type()),
212:       output_scale,
```
- EN: The main symbol in this range is `run`, `QLeakyRelu`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `run`, `QLeakyRelu`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 213-225
```cpp
213:       output_zero_point,
214:       self.suggest_memory_format());
215:     qrelu_leaky_stub(self.device().type(), qy, qx, negative_slope);
216:     return qy;
217:   }
218: };
219:
220: class QPRelu final {
221:  public:
222:   static Tensor run(Tensor self, const Tensor& weight, double output_scale, int64_t output_zero_point) {
223:   return _prelu_kernel_quantized_cpu_impl(self, weight, output_scale, output_zero_point);
224:   }
225: };
```
- EN: The main symbol in this range is `run`, `qrelu_leaky_stub`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `run`, `qrelu_leaky_stub`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 227-235
```cpp
227: TORCH_LIBRARY_IMPL(quantized, QuantizedCPU, m) {
228:   m.impl(TORCH_SELECTIVE_NAME("quantized::relu6"), TORCH_FN(QRelu6::run));
229:   m.impl(TORCH_SELECTIVE_NAME("quantized::leaky_relu"), TORCH_FN(QLeakyRelu::run));
230:   m.impl(TORCH_SELECTIVE_NAME("quantized::prelu"), TORCH_FN(QPRelu::run));
231: }
232:
233: } // namespace
234:
235: }  // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- TensorIterator traversal / TensorIterator 遍历
- SIMD vectorization / SIMD 向量化
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Pooling reductions / 池化归约
- Dispatcher registration / 调度器注册

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Context.h`, `ATen/Dispatch.h`, `ATen/TensorIterator.h`, `ATen/native/cpu/Loops.h`, `ATen/native/quantized/AffineQuantizer.h`, `ATen/native/quantized/cpu/init_qnnpack.h`, `ATen/native/quantized/cpu/QnnpackUtils.h`, `ATen/native/quantized/cpu/QuantizedOps.h`, `ATen/Functions.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `caffe2/utils/threadpool/pthreadpool-cpp.h`, `torch/library.h`, `algorithm`
- Key helper symbols / 关键辅助符号: `TensorIterator`, `cpu_kernel`, `cpu_kernel_vec`, `Vectorized`, `Scalar`, `qnnpack`
