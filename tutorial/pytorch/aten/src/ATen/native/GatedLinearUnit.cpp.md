# GatedLinearUnit.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/GatedLinearUnit.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Gated Linear Unit. It also wires backend dispatch paths.
- **Purpose (CN)**: 实现或声明与 gated、线性、unit 相关的 ATen 原生逻辑。 它还负责连接不同后端的调度路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/TensorIterator.h>
0004: #include <ATen/TensorOperators.h>
0005: #include <ATen/native/Activation.h>
0006: 
0007: #ifndef AT_PER_OPERATOR_HEADERS
0008: #include <ATen/Functions.h>
0009: #include <ATen/NativeFunctions.h>
0010: #else
0011: #include <ATen/ops/cat.h>
0012: #include <ATen/ops/empty.h>
0013: #include <ATen/ops/glu_backward_native.h>
0014: #include <ATen/ops/glu_backward_jvp_native.h>
0015: #include <ATen/ops/glu_jvp_native.h>
0016: #include <ATen/ops/glu_native.h>
0017: #include <ATen/ops/sigmoid.h>
0018: #endif
0019: 
0020: namespace at::meta {
0021: 
0022: TORCH_META_FUNC(glu) (
0023:     const Tensor& self, int64_t dim
0024: ) {
0025:   // this can't pass anyway because a 0-dimensional tensor has "size" 1, which
0026:   // can't be evenly halved, but give a nicer error message here.
0027:   TORCH_CHECK(self.dim() > 0, "glu does not support 0-dimensional tensors");
0028:   auto wrap_dim = maybe_wrap_dim(dim, self.dim());
0029:   const int64_t nIn = self.size(wrap_dim);
0030:   TORCH_CHECK(nIn % 2 == 0, "Halving dimension must be even, but dimension ",
```
- **EN**: Lines 1-30 mainly cover header inclusion, macro-based glue, conditional compilation. Notable symbols: TORCH_META_FUNC, TORCH_CHECK, dim, maybe_wrap_dim.
- **CN**: 第 1-30 行主要涉及头文件包含、宏定义或宏调用、预处理条件。 值得关注的符号包括：TORCH_META_FUNC, TORCH_CHECK, dim, maybe_wrap_dim。

### Lines 31-60 / 第 31-60 行
```cpp
0031:               wrap_dim, " is size ", nIn);
0032: 
0033:   // size output to half of input
0034:   const int64_t selfSize = nIn / 2;
0035:   Tensor firstHalf = self.narrow(wrap_dim, 0, selfSize);
0036:   Tensor secondHalf = self.narrow(wrap_dim, selfSize, selfSize);
0037:   build_borrowing_binary_op(maybe_get_output(), firstHalf, secondHalf);
0038: }
0039: } // namespace at::meta
0040: 
0041: namespace at::native {
0042: 
0043: // NOLINTNEXTLINE(cppcoreguidelines-avoid-non-const-global-variables)
0044: DEFINE_DISPATCH(glu_stub);
0045: // NOLINTNEXTLINE(cppcoreguidelines-avoid-non-const-global-variables)
0046: DEFINE_DISPATCH(glu_backward_stub);
0047: // NOLINTNEXTLINE(cppcoreguidelines-avoid-non-const-global-variables)
0048: DEFINE_DISPATCH(glu_jvp_stub);
0049: 
0050: TORCH_IMPL_FUNC(glu_out) (const Tensor& self, int64_t dim, const Tensor& out) {
0051:   glu_stub(device_type(), *this);
0052: }
0053: 
0054: Tensor& glu_backward_cpu_out(const Tensor& grad_output, const Tensor& input,
0055:                              int64_t dim, Tensor& grad_input) {
0056:   TORCH_CHECK(input.dim() > 0, "glu does not support 0-dimensional tensors");
0057:   auto wrap_dim = maybe_wrap_dim(dim, input.dim());
0058:   const int64_t nIn = input.size(wrap_dim);
0059:   TORCH_CHECK(nIn % 2 == 0, "Halving dimension must be even, but dimension ",
0060:               wrap_dim, " is size ", nIn);
```
- **EN**: Lines 31-60 mainly cover state/variable declarations, macro-based glue, comments/documentation. Notable symbols: narrow, build_borrowing_binary_op, maybe_get_output, NOLINTNEXTLINE.
- **CN**: 第 31-60 行主要涉及变量/别名声明、宏定义或宏调用、注释或说明。 值得关注的符号包括：narrow, build_borrowing_binary_op, maybe_get_output, NOLINTNEXTLINE。

### Lines 61-90 / 第 61-90 行
```cpp
0061: 
0062:   grad_input.resize_as_(input);
0063:   const int64_t inputSize = nIn / 2;
0064:   // half tensor
0065:   Tensor firstHalf = input.narrow(wrap_dim, 0, inputSize);
0066:   Tensor secondHalf = input.narrow(wrap_dim, inputSize, inputSize);
0067:   Tensor gradInputfirstHalf = grad_input.narrow(wrap_dim, 0, inputSize);
0068:   Tensor gradInputsecondHalf = grad_input.narrow(wrap_dim, inputSize, inputSize);
0069: 
0070:   at::sigmoid_out(gradInputfirstHalf, secondHalf);
0071:   // for second gradinput half, can get a better performance by fusion
0072:   auto iter = at::TensorIteratorConfig()
0073:     .add_output(gradInputsecondHalf)
0074:     .add_const_input(gradInputfirstHalf)
0075:     .add_const_input(firstHalf)
0076:     .add_const_input(grad_output)
0077:     .build();
0078:   glu_backward_stub(iter.device_type(), iter);
0079:   gradInputfirstHalf.mul_(grad_output);
0080:   return grad_input;
0081: }
0082: 
0083: Tensor glu_backward_cpu(const Tensor& grad_output, const Tensor& input, int64_t dim) {
0084:   auto grad_input = at::empty({0}, input.options());
0085:   return glu_backward_cpu_out(grad_output, input, dim, grad_input);
0086: }
0087: 
0088: Tensor glu_jvp(
0089:     const Tensor& glu,
0090:     const Tensor& x,
```
- **EN**: Lines 61-90 mainly cover state/variable declarations, function signatures/definitions, expressions/calls. Notable symbols: resize_as_, narrow, sigmoid_out, TensorIteratorConfig.
- **CN**: 第 61-90 行主要涉及变量/别名声明、函数签名或实现、表达式或调用。 值得关注的符号包括：resize_as_, narrow, sigmoid_out, TensorIteratorConfig。

### Lines 91-120 / 第 91-120 行
```cpp
0091:     const Tensor& dx,
0092:     int64_t dim
0093: ) {
0094:   dim = maybe_wrap_dim(dim, x.dim());
0095:   const auto glu_size = glu.size(dim);
0096:   const auto b = x.narrow(dim, glu_size, glu_size);
0097:   const auto da = dx.narrow(dim, 0, glu_size);
0098:   const auto db = dx.narrow(dim, glu_size, glu_size);
0099:   auto dglu = at::empty_like(glu);
0100:   auto iter = at::TensorIteratorConfig()
0101:     .add_output(dglu)
0102:     .add_const_input(glu)
0103:     .add_const_input(b)
0104:     .add_const_input(da)
0105:     .add_const_input(db)
0106:     .build();
0107:   glu_jvp_stub(iter.device_type(), iter);
0108:   return dglu;
0109: }
0110: 
0111: Tensor glu_backward_jvp(
0112:     const Tensor& grad_x,
0113:     const Tensor& grad_glu,
0114:     const Tensor& x,
0115:     const Tensor& dgrad_glu,
0116:     const Tensor& dx,
0117:     int64_t dim
0118: ) {
0119:   dim = maybe_wrap_dim(dim, x.dim());
0120:   const auto glu_size = grad_glu.size(dim);
```
- **EN**: Lines 91-120 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: maybe_wrap_dim, dim, size, narrow.
- **CN**: 第 91-120 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：maybe_wrap_dim, dim, size, narrow。

### Lines 121-150 / 第 121-150 行
```cpp
0121:   const auto a = x.narrow(dim, 0, glu_size);
0122:   const auto b = x.narrow(dim, glu_size, glu_size);
0123:   const auto da = dx.narrow(dim, 0, glu_size);
0124:   const auto db = dx.narrow(dim, glu_size, glu_size);
0125:   // grad_x_a = grad_glu * sigmoid(b)
0126:   const auto grad_x_a = grad_x.narrow(dim, 0, glu_size);
0127:   // grad_x_b = grad_x_a * a * (1 - sigmoid(b))
0128:   const auto grad_x_b = grad_x.narrow(dim, glu_size, glu_size);
0129: 
0130:   const auto sig_b = at::sigmoid(b);
0131:   // TODO: use glu from forward.
0132:   // TODO: fuse kernels.
0133:   const auto glu = a * sig_b;
0134:   const auto db_neg_sig_b = db - db * sig_b;
0135: 
0136:   // dgrad_x_a = d(grad_glu * sigmoid(b))
0137:   //           = dgrad_glu * sigmoid(b) + grad_glu * sigmoid(b) * (1 - sigmoid(b)) * db
0138:   //           = dgrad_glu * sig_b + grad_x_a * (db - db * sig_b)
0139:   //           = dgrad_glu * sig_b + grad_x_a * db_neg_sig_b
0140:   const auto dgrad_x_a = dgrad_glu * sig_b + grad_x_a * db_neg_sig_b;
0141: 
0142:   // dgrad_x_b = d(grad_glu * sigmoid(b) * a * (1 - sigmoid(b))
0143:   //           =  d(grad_glu * sigmoid(b)) * a * (1 - sigmoid(b))
0144:   //            + grad_glu * sigmoid(b) * da * (1 - sigmoid(b))
0145:   //            - grad_glu * sigmoid(b) * a * sigmoid(b) * (1 - sigmoid(b)) * db
0146:   //          =   dgrad_x_a * a * (1 - sigmoid(b))
0147:   //           + (grad_glu * sigmoid(b)) * (da * (1 - sigmoid(b)) - a * sigmoid(b) * (1 - sigmoid(b)) * db)
0148:   //          = dgrad_x_a * (a - glu) + grad_x_a * (da - da * sig_b - glu * db_neg_sig_b
0149:   const auto dgrad_x_b = dgrad_x_a * (a - glu) + grad_x_a * (da - da * sig_b - glu * db_neg_sig_b);
0150: 
```
- **EN**: Lines 121-150 mainly cover comments/documentation, state/variable declarations. Notable symbols: narrow, sigmoid, d.
- **CN**: 第 121-150 行主要涉及注释或说明、变量/别名声明。 值得关注的符号包括：narrow, sigmoid, d。

### Lines 151-155 / 第 151-155 行
```cpp
0151:   return at::cat({dgrad_x_a, dgrad_x_b}, dim);
0152: }
0153: 
0154: 
0155: } // namespace at::native
```
- **EN**: Lines 151-155 mainly cover return paths, expressions/calls, namespace structuring. Notable symbols: cat.
- **CN**: 第 151-155 行主要涉及返回路径、表达式或调用、命名空间组织。 值得关注的符号包括：cat。

## Key Concepts / 关键概念
- **EN**: TensorIterator-driven traversal  
  **CN**: 基于 TensorIterator 的遍历
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Dispatch stub definition  
  **CN**: 调度桩定义
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/TensorIterator.h>`, `<ATen/TensorOperators.h>`, `<ATen/native/Activation.h>`, `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/cat.h>`, `<ATen/ops/empty.h>`, `<ATen/ops/glu_backward_native.h>`, `<ATen/ops/glu_backward_jvp_native.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `DEFINE_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`
