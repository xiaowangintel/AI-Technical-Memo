# ACLUtils.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/ACLUtils.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares quantized CPU operator implementations, scale/zero-point handling, and packed weights in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了量化 CPU 算子实现、scale/zero-point 处理与打包权重逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
 1: #pragma once
 2:
 3: #include <ATen/Config.h>
 4: #if AT_MKLDNN_ACL_ENABLED()
 5:
 6: #include <ATen/native/quantized/cpu/OnednnUtils.h>
 7: #include <arm_compute/core/Error.h>
 8: #include <arm_compute/core/TensorInfo.h>
 9: #include <arm_compute/function_info/ActivationLayerInfo.h>
10: #include <arm_compute/runtime/NEON/functions/NEActivationLayer.h>
11: #include <arm_compute/runtime/NEON/functions/NEArithmeticAddition.h>
12: #include <arm_compute/runtime/NEON/functions/NEGEMMLowpMatrixMultiplyCore.h>
13: #include <arm_compute/runtime/NEON/functions/NEQuantizationLayer.h>
14: #include <arm_compute/runtime/Tensor.h>
15: #include <array>
```
- EN: This range pulls in required headers, including `ATen/Config.h`, `ATen/native/quantized/cpu/OnednnUtils.h`, `arm_compute/core/Error.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/Config.h`, `ATen/native/quantized/cpu/OnednnUtils.h`, `arm_compute/core/Error.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 17-27
```cpp
17: // Utilities for Arm Compute Library (ACL) quantized operations
18: // Provides interfaces to leverage ACL's accelerated kernels for statically and
19: // dynamically quantized matmuls (i.e. qlinear and qlinear_dynamic) These are
20: // utalized through PackedLinearWeightsACL which extends
21: // PackedLinearWeightsOnednn Note that PackedLinearWeightsACL extends rather
22: // than replaces PackedLinearWeightsOnednn for AArch64 because ACL currently
23: // only supports per_tensor weight quantization.
24: namespace at::native::acl_utils {
25:
26: using QuantMatmulCacheKey = std::tuple<
27:     int64_t, // M
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 28-39
```cpp
28:     bool, // FUSE_RELU
29:     int64_t, // NUM_THREADS
30:     double, // INPUT_SCALE
31:     int64_t, // INPUT_OFFSET
32:     double, // OUTPUT_SCALE
33:     int64_t, // OUTPUT_OFFSET
34:     bool // SIGNED_INPUT
35:     >;
36:
37: enum class QuantMatmulCacheKeyIndex {
38:   M,
39:   FUSE_RELU,
```
- EN: The main symbol in this range is `QuantMatmulCacheKeyIndex`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `QuantMatmulCacheKeyIndex`，它们直接构成本文件的算子逻辑。

### Lines 40-52
```cpp
40:   NUM_THREADS,
41:   INPUT_SCALE,
42:   INPUT_OFFSET,
43:   OUTPUT_SCALE,
44:   OUTPUT_OFFSET,
45:   SIGNED_INPUT
46: };
47:
48: // Abstract interface to share common stuff between static/dynamic ACL matmuls.
49: struct QuantMatmul {
50:   arm_compute::NEGEMMLowpMatrixMultiplyCore gemm;
51:   // key for use in the cache
52:   QuantMatmulCacheKey key;
```
- EN: The main symbol in this range is `QuantMatmul`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `QuantMatmul`，它们直接构成本文件的算子逻辑。

### Lines 54-65
```cpp
54:   QuantMatmul(
55:       int64_t weight_dim_0,
56:       int64_t weight_dim_1,
57:       double weight_scale,
58:       int64_t weight_offset,
59:       int8_t* weight_ptr,
60:       std::optional<float*> bias_ptr,
61:       const QuantMatmulCacheKey& cache_key);
62:
63:   virtual ~QuantMatmul();
64:   virtual arm_compute::Status validate() = 0;
65:   virtual void configure() = 0;
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 67-82
```cpp
67:  protected:
68:   arm_compute::Tensor wei_q_tensor_;
69:   std::optional<arm_compute::Tensor> bia_tensor_;
70:   arm_compute::GEMMInfo gemm_info_;
71:   std::optional<arm_compute::ActivationLayerInfo> relu_info_;
72: };
73:
74: struct DynamicQuantMatmul : public QuantMatmul {
75:   arm_compute::Tensor src_q_tensor;
76:   arm_compute::Tensor src_tensor;
77:   arm_compute::Tensor dst_tensor;
78:   arm_compute::NEQuantizationLayer quant;
79:   // We need a ReLU layer here (unlike static quantization) because the ReLU
80:   // cannot be "truly" fused with the GEMM through gemm_info in ACL dynamically
81:   // quantized matmuls.
82:   std::optional<arm_compute::NEActivationLayer> relu;
```
- EN: The main symbol in this range is `DynamicQuantMatmul`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `DynamicQuantMatmul`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 84-96
```cpp
84:   DynamicQuantMatmul(
85:       int64_t weight_dim_0,
86:       int64_t weight_dim_1,
87:       double weight_scale,
88:       int64_t weight_offset,
89:       int8_t* weight_ptr,
90:       std::optional<float*> bias_ptr,
91:       const QuantMatmulCacheKey& cache_key);
92:
93:   ~DynamicQuantMatmul() override;
94:
95:   arm_compute::Status validate() override;
96:   void configure() override;
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 98-113
```cpp
 98:  private:
 99:   at::Tensor src_q_tensor_orig_;
100: };
101:
102: struct StaticQuantMatmul : public QuantMatmul {
103:   arm_compute::Tensor src_q_tensor;
104:   arm_compute::Tensor dst_q_tensor;
105:
106:   StaticQuantMatmul(
107:       int64_t weight_dim_0,
108:       int64_t weight_dim_1,
109:       double weight_scale,
110:       int64_t weight_offset,
111:       int8_t* weight_ptr,
112:       std::optional<float*> bias_ptr,
113:       const QuantMatmulCacheKey& cache_key);
```
- EN: The main symbol in this range is `StaticQuantMatmul`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `StaticQuantMatmul`，它们直接构成本文件的算子逻辑。

### Lines 115-129
```cpp
115:   ~StaticQuantMatmul() override;
116:
117:   arm_compute::Status validate() override;
118:   void configure() override;
119:
120:  private:
121:   std::optional<arm_compute::Tensor> bia_q_tensor_;
122:   std::optional<at::Tensor> bia_q_tensor_orig_;
123: };
124:
125: struct QuantAdd {
126:   arm_compute::Tensor qa_tensor;
127:   arm_compute::Tensor qb_tensor;
128:   arm_compute::Tensor qdst_tensor;
129:   arm_compute::NEArithmeticAddition q_add;
```
- EN: The main symbol in this range is `QuantAdd`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `QuantAdd`，它们直接构成本文件的算子逻辑。

### Lines 131-142
```cpp
131:   QuantAdd(
132:       arm_compute::DataType dtype,
133:       const std::vector<int64_t>& input_dims,
134:       double qa_scale,
135:       int64_t qa_offset,
136:       double qb_scale,
137:       int64_t qb_offset,
138:       double dst_scale,
139:       int64_t dst_offset);
140:
141:   arm_compute::Status validate();
142:   void configure();
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 144-155
```cpp
144:  private:
145:   arm_compute::ConvertPolicy policy{arm_compute::ConvertPolicy::SATURATE};
146: };
147:
148: } // namespace at::native::acl_utils
149: struct PackedLinearWeightsACL : public PackedLinearWeightsOnednn {
150:   using ACLQuantMatmul = at::native::acl_utils::QuantMatmul;
151:   using ACLDynamicQuantMatmul = at::native::acl_utils::DynamicQuantMatmul;
152:   using ACLStaticQuantMatmul = at::native::acl_utils::StaticQuantMatmul;
153:   using ACLQuantMatmulCacheKey = at::native::acl_utils::QuantMatmulCacheKey;
154:   using ACLQuantMatmulCacheKeyIndex =
155:       at::native::acl_utils::QuantMatmulCacheKeyIndex;
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `PackedLinearWeightsACL`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `PackedLinearWeightsACL`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 157-166
```cpp
157:   PackedLinearWeightsACL(
158:       std::unique_ptr<ideep::tensor> weight,
159:       std::optional<ideep::tensor> bias,
160:       at::Tensor orig_weight,
161:       std::optional<at::Tensor> orig_bias);
162:
163:   at::Tensor apply_dynamic(at::Tensor input, bool reduce_range = false)
164:       override;
165:   at::Tensor apply_dynamic_relu(at::Tensor input, bool reduce_range = false)
166:       override;
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 168-182
```cpp
168:   at::Tensor apply(
169:       at::Tensor input,
170:       double output_scale,
171:       int64_t output_zero_point) override;
172:   at::Tensor apply_relu(
173:       at::Tensor input,
174:       double output_scale,
175:       int64_t output_zero_point) override;
176:
177:   template <typename ACLQuantMatmulT>
178:   std::shared_ptr<ACLQuantMatmulT> get_acl_quant_matmul(
179:       const ACLQuantMatmulCacheKey& key) {
180:     return std::dynamic_pointer_cast<ACLQuantMatmulT>(
181:         fetch_or_create_acl_quant_matmul<ACLQuantMatmulT>(key));
182:   }
```
- EN: The main symbol in this range is `get_acl_quant_matmul`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `get_acl_quant_matmul`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 184-195
```cpp
184:  private:
185:   int64_t k_;
186:   int64_t n_;
187:   int64_t weight_zero_point_;
188:   double weight_scale_;
189:
190:   // A 2 element (per layer) cache. Given it's not intended to store more than 2
191:   // elements, we do not need a fancy implementation. The idea behind it is to
192:   // allow for a (configuration free) fast path for autoregressive
193:   // transformer-like models which usually involve 2 input tensor shapes; one
194:   // for the prefill phase and another for the autoregressive phase
195:   std::array<std::shared_ptr<ACLQuantMatmul>, 2> cache_;
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 197-207
```cpp
197:   template <typename ACLQuantMatmulT>
198:   std::shared_ptr<ACLQuantMatmul> fetch_or_create_acl_quant_matmul(
199:       const ACLQuantMatmulCacheKey& key) {
200:     // We're only maintaining a 2 element LRU cache
201:     // hit first
202:     if (cache_[0] != nullptr && cache_[0]->key == key) {
203:       return cache_[0];
204:     }
205:     // hit second
206:     if (cache_[1] != nullptr && cache_[1]->key == key) {
207:       // Update LRU
```
- EN: The main symbol in this range is `fetch_or_create_acl_quant_matmul`, which contributes directly to this file's operator logic. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `fetch_or_create_acl_quant_matmul`，它们直接构成本文件的算子逻辑。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 208-219
```cpp
208:       std::swap(cache_[0], cache_[1]);
209:       return cache_[0];
210:     }
211:     // miss -> replace Least Recently Used - i.e. element at index 1
212:     cache_[1] = create_acl_quant_matmul<ACLQuantMatmulT>(key);
213:     std::swap(cache_[0], cache_[1]);
214:     return cache_[0];
215:   }
216:
217:   template <typename ACLQuantMatmulT>
218:   std::shared_ptr<ACLQuantMatmulT> create_acl_quant_matmul(
219:       const ACLQuantMatmulCacheKey& key) {
```
- EN: The main symbol in this range is `create_acl_quant_matmul`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 这一段的主要符号是 `create_acl_quant_matmul`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 220-231
```cpp
220:     std::optional<float*> bias_ptr;
221:     if (bias_.has_value()) {
222:       bias_ptr = (float*)bias_.value().get_data_handle();
223:     }
224:     auto acl_gemm = std::make_shared<ACLQuantMatmulT>(
225:         k_,
226:         n_,
227:         weight_scale_,
228:         weight_zero_point_,
229:         (int8_t*)weight_.get()->get_data_handle(),
230:         bias_ptr,
231:         key);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 233-245
```cpp
233:     // validate
234:     auto status = acl_gemm->validate();
235:     if (status.error_code() != arm_compute::ErrorCode::OK) {
236:       TORCH_WARN(
237:           "Arm Compute Library's Quantized Matmul Validation Failed: " +
238:           status.error_description());
239:       return nullptr;
240:     }
241:
242:     // configure
243:     acl_gemm->configure();
244:     return acl_gemm;
245:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 247-257
```cpp
247:   template <bool ReluFused>
248:   at::Tensor apply_dynamic_impl(at::Tensor input, bool reduce_range = false);
249:
250:   template <bool ReluFused>
251:   at::Tensor apply_impl(
252:       at::Tensor input,
253:       double output_scale,
254:       int64_t output_zero_point);
255: };
256:
257: #endif // AT_MKLDNN_ACL_ENABLED()
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The main symbol in this range is `apply_dynamic_impl`, `apply_impl`, which contributes directly to this file's operator logic.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 这一段的主要符号是 `apply_dynamic_impl`, `apply_impl`，它们直接构成本文件的算子逻辑。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Shape/container bookkeeping / 形状与容器管理
- Namespace-scoped helper structure / 命名空间内辅助结构
- Operator-specific control flow / 算子特定控制流

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/Config.h`, `ATen/native/quantized/cpu/OnednnUtils.h`
- Standard or third-party headers / 标准库或第三方头文件: `arm_compute/core/Error.h`, `arm_compute/core/TensorInfo.h`, `arm_compute/function_info/ActivationLayerInfo.h`, `arm_compute/runtime/NEON/functions/NEActivationLayer.h`, `arm_compute/runtime/NEON/functions/NEArithmeticAddition.h`, `arm_compute/runtime/NEON/functions/NEGEMMLowpMatrixMultiplyCore.h`, `arm_compute/runtime/NEON/functions/NEQuantizationLayer.h`, `arm_compute/runtime/Tensor.h`, `array`
