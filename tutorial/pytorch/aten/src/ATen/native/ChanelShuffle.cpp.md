# ChanelShuffle.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/ChanelShuffle.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Chanel Shuffle. It also wires backend dispatch paths.
- **Purpose (CN)**: 实现或声明与 chanel、shuffle 相关的 ATen 原生逻辑。 它还负责连接不同后端的调度路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/NamedTensorUtils.h>
0003: #if defined(C10_MOBILE) && defined(USE_XNNPACK)
0004: #include <ATen/native/xnnpack/Engine.h>
0005: #endif
0006: #include <c10/util/Exception.h>
0007: 
0008: #include <ATen/native/cpu/ChannelShuffleKernel.h>
0009: 
0010: #ifndef AT_PER_OPERATOR_HEADERS
0011: #include <ATen/Functions.h>
0012: #include <ATen/NativeFunctions.h>
0013: #else
0014: #include <ATen/ops/channel_shuffle_native.h>
0015: #include <ATen/ops/empty.h>
0016: #include <ATen/ops/native_channel_shuffle.h>
0017: #include <ATen/ops/native_channel_shuffle_native.h>
0018: #endif
0019: 
0020: namespace at::native {
0021: 
0022: Tensor channel_shuffle_cpu(const Tensor& self, int64_t groups) {
0023:   TORCH_CHECK(self.dim() > 2,
0024:               "channel_shuffle expects input with > 2 dims, but got input with sizes ",
0025:               self.sizes());
0026:   int64_t c = self.size(1);
0027:   TORCH_CHECK(groups > 0,
0028:               "Number of groups to divide channels in must be positive.",
0029:               " Value of groups:", groups);
0030:   TORCH_CHECK((c % groups) == 0,
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, macro-based glue. Notable symbols: defined, channel_shuffle_cpu, TORCH_CHECK, dim.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、宏定义或宏调用。 值得关注的符号包括：defined, channel_shuffle_cpu, TORCH_CHECK, dim。

### Lines 31-60 / 第 31-60 行
```cpp
0031:               "Number of channels must be divisible by groups. Got ",
0032:               c, " channels and ", groups, " groups.");
0033: 
0034:   Tensor output;
0035:   if (self.numel() == 0) {
0036:     output = self.alias();
0037:   } else {
0038:     auto memory_format = self.suggest_memory_format();
0039:     output = at::empty({0}, self.options());
0040:     output.resize_(self.sizes(), memory_format);
0041:     auto input = self.contiguous(memory_format);
0042:     channel_shuffle_kernel(kCPU, output, input, groups);
0043:   }
0044:   return namedinference::propagate_names_if_nonempty(
0045:       output,
0046:       self.has_names() ? self.names() : at::ArrayRef<Dimname>{});
0047: }
0048: 
0049: Tensor channel_shuffle(const Tensor& self, int64_t groups) {
0050:   TORCH_CHECK(self.dim() > 2,
0051:               "channel_shuffle expects input with > 2 dims, but got input with sizes ",
0052:               self.sizes());
0053:   int64_t c = self.size(1);
0054:   TORCH_CHECK(groups > 0,
0055:               "Number of groups to divide channels in must be positive.",
0056:               " Value of groups:", groups);
0057:   TORCH_CHECK((c % groups) == 0,
0058:               "Number of channels must be divisible by groups. Got ",
0059:               c, " channels and ", groups, " groups.");
0060: 
```
- **EN**: Lines 31-60 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: numel, alias, suggest_memory_format, empty.
- **CN**: 第 31-60 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：numel, alias, suggest_memory_format, empty。

### Lines 61-90 / 第 61-90 行
```cpp
0061: #if defined(C10_MOBILE) && defined(USE_XNNPACK)
0062:   if (self.is_contiguous(MemoryFormat::ChannelsLast) &&
0063:       xnnpack::use_channel_shuffle(self, groups)) {
0064:     auto output = self.numel() == 0 ? self.alias() : xnnpack::channel_shuffle(self, groups);
0065:     return output;
0066:   }
0067: #endif
0068: 
0069:   auto output = self.numel() == 0 ? self.alias() : at::native_channel_shuffle(self, groups);
0070:   return namedinference::propagate_names_if_nonempty(
0071:       output,
0072:       self.has_names() ? self.names() : at::ArrayRef<Dimname>{});
0073: }
0074: 
0075: Tensor math_channel_shuffle(const Tensor& self, int64_t groups) {
0076:   int64_t b = self.size(0);
0077:   int64_t c = self.size(1);
0078:   int64_t oc = c / groups;
0079: 
0080:   auto input_reshaped = self.view({b, groups, oc, -1});
0081:   // TODO: contiguous can be made to preserve the memory format
0082:   // of the input. However since the above reshape clobbers h and w
0083:   // it may not be safe to do that, since channels_last contiguous
0084:   // may think oc and the last dim correspond to h,w?
0085:   // It is not clear, however from initial looking around it feels that
0086:   // this may not be correct.
0087:   // In this case channels last will likely require custom implementation
0088:   // if we want to preserve the memory order.
0089:   // XNNPACK has channel shuffle op for NHWC. For mobile usecase this is good.
0090:   // For server we will have to do a custom implementation.
```
- **EN**: Lines 61-90 mainly cover comments/documentation, state/variable declarations, expressions/calls. Notable symbols: defined, is_contiguous, use_channel_shuffle, numel.
- **CN**: 第 61-90 行主要涉及注释或说明、变量/别名声明、表达式或调用。 值得关注的符号包括：defined, is_contiguous, use_channel_shuffle, numel。

### Lines 91-104 / 第 91-104 行
```cpp
0091:   // For ChannelsFirst, a.k.a Contiguous, memory format we will also need
0092:   // a fast custom implementation perhaps.
0093:   Tensor output_tensor =
0094:       input_reshaped.permute({0 /* b */, 2 /* oc */, 1 /* groups */, 3})
0095:       .contiguous()
0096:       .reshape(self.sizes());
0097:   return namedinference::propagate_names_if_nonempty(
0098:       output_tensor,
0099:       self.has_names() ? self.names() : at::ArrayRef<Dimname>{});
0100: }
0101: 
0102: DEFINE_DISPATCH(channel_shuffle_kernel);
0103: 
0104: } // namespace at::native
```
- **EN**: Lines 91-104 mainly cover expressions/calls, comments/documentation, function signatures/definitions. Notable symbols: permute, contiguous, reshape, sizes.
- **CN**: 第 91-104 行主要涉及表达式或调用、注释或说明、函数签名或实现。 值得关注的符号包括：permute, contiguous, reshape, sizes。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Dispatch stub definition  
  **CN**: 调度桩定义
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现
- **EN**: Native operator implementation path  
  **CN**: 原生算子实现路径

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/NamedTensorUtils.h>`, `<ATen/native/xnnpack/Engine.h>`, `<c10/util/Exception.h>`, `<ATen/native/cpu/ChannelShuffleKernel.h>`, `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/channel_shuffle_native.h>`, `<ATen/ops/empty.h>`, `<ATen/ops/native_channel_shuffle.h>`, `<ATen/ops/native_channel_shuffle_native.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `DEFINE_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`
