# EmbeddingBag.h — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/EmbeddingBag.h`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Embedding Bag. As a header, it exposes declarations and shared helpers.
- **Purpose (CN)**: 实现或声明与 嵌入、bag 相关的 ATen 原生逻辑。 作为头文件，它暴露声明与共享辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #pragma once
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/Config.h>
0004: #include <cstdint>
0005: 
0006: #ifdef USE_FBGEMM
0007: #include <fbgemm/FbgemmEmbedding.h>
0008: #endif
0009: 
0010: namespace at::native {
0011: 
0012: enum class EmbeddingBagMode {
0013:   SUM = 0,
0014:   MEAN = 1,
0015:   MAX = 2,
0016: };
0017: 
0018: [[maybe_unused]] static bool operator==(int64_t op1, EmbeddingBagMode op2) {
0019:   return op1 == static_cast<int64_t>(op2);
0020: }
0021: 
0022: [[maybe_unused]] static bool operator!=(int64_t op1, EmbeddingBagMode op2) {
0023:   return !(op1 == op2);
0024: }
0025: 
0026: void check_arguments(
0027:     const Tensor& weight,
0028:     const Tensor& indices,
0029:     const Tensor& offsets,
0030:     const int64_t mode,
```
- **EN**: Lines 1-30 mainly cover expressions/calls, header inclusion, conditional compilation. Notable symbols: check_arguments.
- **CN**: 第 1-30 行主要涉及表达式或调用、头文件包含、预处理条件。 值得关注的符号包括：check_arguments。

### Lines 31-60 / 第 31-60 行
```cpp
0031:     const std::optional<Tensor>& per_sample_weights,
0032:     bool include_last_offset);
0033: 
0034: void make_bag_size_out(
0035:     Tensor& bag_size_out,
0036:     const Tensor& offsets,
0037:     const Tensor& indices,
0038:     const int64_t mode,
0039:     const bool include_last_offset,
0040:     const bool requires_grad);
0041: 
0042: void make_max_indices_out(
0043:     Tensor& max_indices_out,
0044:     const Tensor& weight,
0045:     const Tensor& indices,
0046:     const Tensor& offsets,
0047:     const Tensor& bag_size,
0048:     const int64_t mode,
0049:     bool include_last_offset);
0050: 
0051: void make_offset2bag_out(
0052:     Tensor& offset2bag,
0053:     Tensor& output,
0054:     const Tensor& weight,
0055:     const Tensor& indices,
0056:     const Tensor& offsets,
0057:     const int64_t mode,
0058:     const std::optional<Tensor>& per_sample_weights,
0059:     const int64_t padding_idx = -1);
0060: 
```
- **EN**: Lines 31-60 mainly cover expressions/calls, state/variable declarations. Notable symbols: make_bag_size_out, make_max_indices_out, make_offset2bag_out.
- **CN**: 第 31-60 行主要涉及表达式或调用、变量/别名声明。 值得关注的符号包括：make_bag_size_out, make_max_indices_out, make_offset2bag_out。

### Lines 61-90 / 第 61-90 行
```cpp
0061: #ifdef USE_FBGEMM
0062: 
0063: template<bool has_weight, typename TIndex, typename TData>
0064: struct _CallbackAndBlockSize {
0065:     using TCallback = typename fbgemm::EmbeddingSpMDMKernelSignature<TData, TIndex, TIndex, TData>::Type;
0066: 
0067:     int64_t blockSize = -1;
0068:     TCallback callback = nullptr;
0069: 
0070:     static TCallback generateCallback(int64_t block_size) {
0071:         return fbgemm::GenerateEmbeddingSpMDM<TData, TIndex, TIndex, TData>(
0072:                 block_size,
0073:                 has_weight,
0074:                 /* normalize_by_lengths */false,
0075:                 /* prefetch */16,
0076:                 /* is_weight_positional */false,
0077:                 /* use_offsets */true);
0078:     }
0079: 
0080:     _CallbackAndBlockSize() = default;
0081: 
0082:     explicit _CallbackAndBlockSize(std::optional<int64_t> maybe_block_size)
0083:       : blockSize(maybe_block_size.value_or(-1))
0084:       , callback(maybe_block_size.has_value() ? generateCallback(maybe_block_size.value()) : nullptr)
0085:     {}
0086: };
0087: 
0088: template<typename... StorageMixins>
0089: struct _EmbeddingBagKernelCacheImpl : private StorageMixins... {
0090: 
```
- **EN**: Lines 61-90 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: generateCallback, _CallbackAndBlockSize, blockSize, value_or.
- **CN**: 第 61-90 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：generateCallback, _CallbackAndBlockSize, blockSize, value_or。

### Lines 91-120 / 第 91-120 行
```cpp
0091:     _EmbeddingBagKernelCacheImpl() = default;
0092:     // use each of the mixins to store corresponding kernel and block size
0093:     explicit _EmbeddingBagKernelCacheImpl(std::optional<int64_t> maybe_block_size)
0094:       : StorageMixins(maybe_block_size)...
0095:     {}
0096: 
0097:     // this method is thread safe (call sites may call from different threads)
0098:     template<bool has_weight, typename TIndex, typename TData>
0099:     typename _CallbackAndBlockSize<has_weight, TIndex, TData>::TCallback
0100:     getCallback(int64_t block_size) const {
0101:         // if the cache doesn't store the kernel for the incoming block size
0102:         // (so it is different from the one stored in corresponding mixin)
0103:         // regenerate the kernel (not writing it into the cache so we avoid locks)
0104:         if (block_size != _CallbackAndBlockSize<has_weight, TIndex, TData>::blockSize) {
0105:             return _CallbackAndBlockSize<has_weight, TIndex, TData>::generateCallback(block_size);
0106:         }
0107:         // else retrieve the cached kernel from the corresponding mixin
0108:         return _CallbackAndBlockSize<has_weight, TIndex, TData>::callback;
0109:     }
0110: };
0111: 
0112: // instantiate the cache with the list of storage mixins
0113: // for each of the 8 _EmbeddingBagKernelCache* usages in the EmbeddingBag.cpp impl file
0114: using _EmbeddingBagKernelCache = _EmbeddingBagKernelCacheImpl<
0115:     _CallbackAndBlockSize<true, int32_t, float>,
0116:     _CallbackAndBlockSize<false, int32_t, float>,
0117:     _CallbackAndBlockSize<true, int64_t, float>,
0118:     _CallbackAndBlockSize<false, int64_t, float>,
0119:     _CallbackAndBlockSize<true, int32_t, unsigned short>,
0120:     _CallbackAndBlockSize<false, int32_t, unsigned short>,
```
- **EN**: Lines 91-120 mainly cover expressions/calls, comments/documentation, state/variable declarations. Notable symbols: _EmbeddingBagKernelCacheImpl, StorageMixins, safe, getCallback.
- **CN**: 第 91-120 行主要涉及表达式或调用、注释或说明、变量/别名声明。 值得关注的符号包括：_EmbeddingBagKernelCacheImpl, StorageMixins, safe, getCallback。

### Lines 121-150 / 第 121-150 行
```cpp
0121:     _CallbackAndBlockSize<true, int64_t, unsigned short>,
0122:     _CallbackAndBlockSize<false, int64_t, unsigned short>>;
0123: #else
0124: struct _EmbeddingBagKernelCache {
0125:     explicit _EmbeddingBagKernelCache(std::optional<int64_t> /* maybe_block_size */) {}
0126: };
0127: #endif
0128: 
0129: void _embedding_bag_cpu_impl_out(Tensor& output, Tensor& offset2bag,
0130:     Tensor& bag_size, Tensor* max_indices,
0131:     const Tensor &weight, const Tensor &indices,
0132:     const Tensor &offsets, const int64_t mode = 0,
0133:     const std::optional<Tensor>& per_sample_weights = std::nullopt,
0134:     bool include_last_offset = false,
0135:     int64_t padding_idx = -1,
0136:     _EmbeddingBagKernelCache* fbgemm_kernel_cache = nullptr);
0137: 
0138: void _embedding_bag_cpu_out(
0139:     at::Tensor& output,
0140:     at::Tensor& offset2bag,
0141:     at::Tensor& bag_size,
0142:     at::Tensor* p_max_indices,
0143:     const at::Tensor& weight,
0144:     const at::Tensor& indices,
0145:     const at::Tensor& offsets,
0146:     const bool scale_grad_by_freq,
0147:     const int64_t mode,
0148:     const bool sparse,
0149:     const std::optional<at::Tensor>& per_sample_weights,
0150:     const bool include_last_offset,
```
- **EN**: Lines 121-150 mainly cover expressions/calls, state/variable declarations, conditional compilation. Notable symbols: _EmbeddingBagKernelCache, _embedding_bag_cpu_impl_out, _embedding_bag_cpu_out.
- **CN**: 第 121-150 行主要涉及表达式或调用、变量/别名声明、预处理条件。 值得关注的符号包括：_EmbeddingBagKernelCache, _embedding_bag_cpu_impl_out, _embedding_bag_cpu_out。

### Lines 151-154 / 第 151-154 行
```cpp
0151:     const std::optional<int64_t>& padding_idx,
0152:     _EmbeddingBagKernelCache* fbgemm_kernel_cache = nullptr);
0153: 
0154: } // namespace at::native
```
- **EN**: Lines 151-154 mainly cover expressions/calls, state/variable declarations, namespace structuring.
- **CN**: 第 151-154 行主要涉及表达式或调用、变量/别名声明、命名空间组织。

## Key Concepts / 关键概念
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现
- **EN**: Native operator implementation path  
  **CN**: 原生算子实现路径

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/Config.h>`, `<cstdint>`, `<fbgemm/FbgemmEmbedding.h>`
- **Macros / 宏**: None highlighted / 无特别标注
- **Namespaces / 命名空间**: `at::native`, `at::`, `std::`
