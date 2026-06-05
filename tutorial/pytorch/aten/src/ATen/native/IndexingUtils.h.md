# IndexingUtils.h — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/IndexingUtils.h`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Indexing Utils. As a header, it exposes declarations and shared helpers.
- **Purpose (CN)**: 实现或声明与 索引、utils 相关的 ATen 原生逻辑。 作为头文件，它暴露声明与共享辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #pragma once
0002: #include <ATen/ExpandUtils.h>
0003: #include <ATen/native/CanUse32BitIndexMath.h>
0004: #include <ATen/native/TensorIterator.h>
0005: #include <ATen/core/IListRef.h>
0006: #include <c10/util/irange.h>
0007: 
0008: #ifndef AT_PER_OPERATOR_HEADERS
0009: #include <ATen/Functions.h>
0010: #else
0011: #include <ATen/ops/empty.h>
0012: #include <ATen/ops/nonzero.h>
0013: #endif
0014: 
0015: namespace at::native {
0016: 
0017: [[noreturn]]
0018: static void invalid_mask(const Tensor & self, int64_t idx, const Tensor & mask, int64_t maskIdx) {
0019:   TORCH_CHECK_INDEX(false, "The shape of the mask ", mask.sizes(), " at index ", maskIdx,
0020:   " does not match the shape of the indexed tensor ", self.sizes(), " at index ", idx);
0021: }
0022: 
0023: [[maybe_unused]] static std::vector<Tensor> expandTensors(
0024:     const Tensor& self,
0025:     IOptTensorListRef indices,
0026:     bool ensure_same_device = false) {
0027:   // If indices come in as ByteTensor or BoolTensor (masks), expand them into
0028:   // the equivalent indexing by LongTensors
0029:   std::vector<Tensor> result;
0030:   for (const auto& index_opt : indices) {
```
- **EN**: Lines 1-30 mainly cover header inclusion, expressions/calls, conditional compilation. Notable symbols: invalid_mask, TORCH_CHECK_INDEX, sizes, expandTensors.
- **CN**: 第 1-30 行主要涉及头文件包含、表达式或调用、预处理条件。 值得关注的符号包括：invalid_mask, TORCH_CHECK_INDEX, sizes, expandTensors。

### Lines 31-60 / 第 31-60 行
```cpp
0031:     if (!index_opt.has_value()) {
0032:       result.emplace_back();
0033:     } else {
0034:       const auto& index = *index_opt;
0035:       if (index.scalar_type() == kByte || index.scalar_type() == kBool) {
0036:         if (index.scalar_type() == kByte) {
0037:           TORCH_WARN("indexing with dtype torch.uint8 is now deprecated," \
0038:           " please use a dtype torch.bool instead.");
0039:         }
0040:         // The sizes of the ByteTensor mask or bool tensor must match the sizes of the
0041:         // corresponding dimensions in self
0042:         for (const auto j : c10::irange(index.dim())) {
0043:           int64_t srcIdx = static_cast<int64_t>(result.size() + j);
0044:           if (index.size(j) != self.size(srcIdx)) {
0045:             invalid_mask(self, srcIdx, index, j);
0046:           }
0047:         }
0048:         // Replace with nonzeros
0049:         at::Tensor nonzero;
0050:         if (ensure_same_device && index.device() != self.device()) {
0051:           bool non_blocking = index.is_cpu() && self.device().is_cuda();
0052:           auto out = at::empty({0}, index.options().dtype(kLong).pinned_memory(non_blocking));
0053:           nonzero = at::nonzero_out(out, index).to(self.device(), non_blocking);
0054:         } else {
0055:           nonzero = index.nonzero();
0056:         }
0057:         for (const auto j : c10::irange(index.dim())) {
0058:           result.emplace_back(nonzero.select(1, j));
0059:         }
0060:       } else if (ensure_same_device && index.device() != self.device()) {
```
- **EN**: Lines 31-60 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: has_value, emplace_back, scalar_type, TORCH_WARN.
- **CN**: 第 31-60 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：has_value, emplace_back, scalar_type, TORCH_WARN。

### Lines 61-90 / 第 61-90 行
```cpp
0061:         result.emplace_back(index.to(self.device()));
0062:       } else {
0063:         result.emplace_back(index);
0064:       }
0065:     }
0066:   }
0067:   return result;
0068: }
0069: 
0070: [[maybe_unused]] static void checkIndexTensorTypes(
0071:     IOptTensorListRef indices,
0072:     bool allow_int = false) {
0073:   for (const auto& tensor : indices) {
0074:     if (tensor.has_value() && tensor->defined()) {
0075:       auto scalarType = tensor->scalar_type();
0076:       if (allow_int) {
0077:         if (scalarType != kLong && scalarType != kByte && scalarType != kBool && scalarType != kInt) {
0078:             TORCH_CHECK_INDEX(false, "tensors used as indices must be long, int, byte or bool tensors");
0079:         }
0080:       } else {
0081:         if (scalarType != kLong && scalarType != kByte && scalarType != kBool) {
0082:             TORCH_CHECK_INDEX(false, "tensors used as indices must be long, byte or bool tensors");
0083:         }
0084:       }
0085:     }
0086:   }
0087: }
0088: 
0089: inline torch::List<std::optional<Tensor>> toListOfOptionalTensors(ArrayRef<Tensor> list) {
0090:   torch::List<std::optional<Tensor>> result;
```
- **EN**: Lines 61-90 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: emplace_back, to, device, checkIndexTensorTypes.
- **CN**: 第 61-90 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：emplace_back, to, device, checkIndexTensorTypes。

### Lines 91-120 / 第 91-120 行
```cpp
0091:   result.reserve(list.size());
0092:   for (const Tensor& a : list) {
0093:     result.push_back(a);
0094:   }
0095:   return result;
0096: }
0097: 
0098: inline torch::List<std::optional<Tensor>> toListOfOptionalTensors(ArrayRef<IValue> list) {
0099:   torch::List<std::optional<Tensor>> result;
0100:   result.reserve(list.size());
0101:   for (const IValue& a : list) {
0102:     result.push_back(a.isTensor() ? std::optional<Tensor>(a.toTensor()) : std::optional<Tensor>());
0103:   }
0104:   return result;
0105: }
0106: 
0107: [[maybe_unused]] static bool hasContiguousSubspace(TensorList tl) {
0108:   // true if all the non-null tensors are adjacent
0109:   auto isDefined = [](const Tensor & tensor){ return tensor.defined(); };
0110:   auto isNull = [](const Tensor & tensor){ return !tensor.defined(); };
0111:   auto start = std::find_if(tl.begin(), tl.end(), isDefined);
0112:   if (start == tl.end()) {
0113:     return true;
0114:   }
0115:   auto stop = std::find_if(tl.rbegin(), tl.rend(), isDefined);
0116:   auto it = std::find_if(start, stop.base(), isNull);
0117:   return it == stop.base();
0118: }
0119: 
0120: // Transposes the tensor and indices together so that all the non-null indices
```
- **EN**: Lines 91-120 mainly cover state/variable declarations, expressions/calls, return paths. Notable symbols: reserve, size, push_back, toListOfOptionalTensors.
- **CN**: 第 91-120 行主要涉及变量/别名声明、表达式或调用、返回路径。 值得关注的符号包括：reserve, size, push_back, toListOfOptionalTensors。

### Lines 121-150 / 第 121-150 行
```cpp
0121: // index the first k dimensions of the tensor. Returns the transposed tensor
0122: // and the reordered indices. For example:
0123: // transposeToFront(tensor, {nullptr, a, nullptr, b})
0124: // returns
0125: // tensor.permute([1, 3, 0, 2]), {a, b, nullptr, nullptr}
0126: [[maybe_unused]] static std::tuple<Tensor, std::vector<Tensor>> transposeToFront(
0127:     const Tensor& self,
0128:     TensorList indices) {
0129:   std::vector<int64_t> dims;
0130:   std::vector<Tensor> transposedIndices;
0131:   dims.reserve(self.dim());
0132:   for (const auto i : c10::irange(self.dim())) {
0133:     if (indices[i].defined()) {
0134:       dims.push_back(i);
0135:       transposedIndices.emplace_back(indices[i]);
0136:     }
0137:   }
0138:   for (const auto i : c10::irange(self.dim())) {
0139:     if (!indices[i].defined()) {
0140:       dims.push_back(i);
0141:       transposedIndices.emplace_back();
0142:     }
0143:   }
0144:   return std::make_tuple(self.permute(dims), std::move(transposedIndices));
0145: }
0146: 
0147: inline std::tuple<Tensor, std::vector<Tensor>, std::vector<int64_t>>
0148: transposeToFrontAndInvPerm(const Tensor& self, TensorList indices) {
0149:   std::vector<int64_t> dims;
0150:   std::vector<int64_t> invPerm;
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: transposeToFront, permute, reserve, dim.
- **CN**: 第 121-150 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：transposeToFront, permute, reserve, dim。

### Lines 151-180 / 第 151-180 行
```cpp
0151:   std::vector<Tensor> transposedIndices;
0152:   dims.reserve(self.dim());
0153:   invPerm.resize(self.dim());
0154:   for (const auto i : c10::irange(self.dim())) {
0155:     if (indices[i].defined()) {
0156:       dims.push_back(i);
0157:       transposedIndices.emplace_back(indices[i]);
0158:     }
0159:   }
0160:   for (const auto i : c10::irange(self.dim())) {
0161:     if (!indices[i].defined()) {
0162:       dims.push_back(i);
0163:       transposedIndices.emplace_back();
0164:     }
0165:   }
0166:   for (const auto i : c10::irange(self.dim())) {
0167:     invPerm[dims[i]] = i;
0168:   }
0169:   return std::make_tuple(self.permute(dims), std::move(transposedIndices), std::move(invPerm));
0170: }
0171: 
0172: struct AdvancedIndex {
0173:   AdvancedIndex(const Tensor& src, TensorList indices);
0174: 
0175:   Tensor src;
0176:   std::vector<Tensor> indices;
0177:   DimVector indexed_sizes;
0178:   DimVector indexed_strides;
0179:   int64_t dims_before;
0180:   int64_t dims_after;
```
- **EN**: Lines 151-180 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: reserve, dim, resize, irange.
- **CN**: 第 151-180 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：reserve, dim, resize, irange。

### Lines 181-184 / 第 181-184 行
```cpp
0181: };
0182: 
0183: 
0184: } //namespace at::native
```
- **EN**: Lines 181-184 mainly cover state/variable declarations, expressions/calls.
- **CN**: 第 181-184 行主要涉及变量/别名声明、表达式或调用。

## Key Concepts / 关键概念
- **EN**: TensorIterator-driven traversal  
  **CN**: 基于 TensorIterator 的遍历
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现
- **EN**: Native operator implementation path  
  **CN**: 原生算子实现路径

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/ExpandUtils.h>`, `<ATen/native/CanUse32BitIndexMath.h>`, `<ATen/native/TensorIterator.h>`, `<ATen/core/IListRef.h>`, `<c10/util/irange.h>`, `<ATen/Functions.h>`, `<ATen/ops/empty.h>`, `<ATen/ops/nonzero.h>`
- **Macros / 宏**: `TORCH_CHECK`, `TORCH_WARN`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
