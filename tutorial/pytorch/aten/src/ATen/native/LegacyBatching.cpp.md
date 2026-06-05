# LegacyBatching.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/LegacyBatching.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Legacy Batching.
- **Purpose (CN)**: 实现或声明与 legacy、batching 相关的 ATen 原生逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #include <ATen/core/Tensor.h>
0002: #include <ATen/LegacyBatchedTensorImpl.h>
0003: #include <ATen/WrapDimUtils.h>
0004: #include <ATen/LegacyVmapTransforms.h>
0005: 
0006: #ifdef AT_PER_OPERATOR_HEADERS
0007: #include <ATen/ops/_add_batch_dim_native.h>
0008: #include <ATen/ops/_remove_batch_dim_native.h>
0009: #endif
0010: 
0011: namespace at::native {
0012: 
0013: // Adds a batch dimension to the tensor `self` out-of-place
0014: Tensor _add_batch_dim(const Tensor& self, int64_t batch_dim, int64_t level) {
0015:   return addBatchDim(self, level, batch_dim);
0016: }
0017: 
0018: static bool has_level(const Tensor& self, int64_t level) {
0019:   const auto* batched = maybeGetBatchedImpl(self);
0020:   if (!batched) {
0021:     return false;
0022:   }
0023:   auto bdims = batched->bdims();
0024:   auto* it = std::find_if(bdims.begin(), bdims.end(), [&](const BatchDim& bdim) {
0025:     return bdim.level() == level;
0026:   });
0027:   return it != bdims.end();
0028: }
0029: 
0030: // Returns a Tensor with batch dim with level `level` turned into a regular dimension,
```
- **EN**: Lines 1-30 mainly cover header inclusion, return paths, state/variable declarations. Notable symbols: _add_batch_dim, addBatchDim, has_level, maybeGetBatchedImpl.
- **CN**: 第 1-30 行主要涉及头文件包含、返回路径、变量/别名声明。 值得关注的符号包括：_add_batch_dim, addBatchDim, has_level, maybeGetBatchedImpl。

### Lines 31-60 / 第 31-60 行
```cpp
0031: // as well as a logical dim index of where said dimension is in the returned tensor.
0032: // A call to this function is always followed by a call to `movedim`.
0033: //
0034: // Preconditions: A BatchDim with level `level` must exist inside `batched`.
0035: //
0036: // The reason why we want to return the index of where said dimension is in the returned
0037: // tensor is because we want to keep track of which dimension used to be the batch
0038: // dimension so that we can move it to the correct logical dimension specified by
0039: // `out_dims` in vmap. For example, if we had
0040: // >>> x = torch.randn(2, 3, 5)
0041: // >>> vmap(lambda x: x, in_dims=0, out_dims=1)(x)
0042: // then right when we are about to exit the vmap block, x is a BatchedTensor with a
0043: // batch dimension at (physical) index 0. Note that the batch dimension doesn't
0044: // always have to exist at (physical) index 0. When we undo the batch dimension,
0045: // we want to move it to dimension 1 (as specified by out_dims). So we return the
0046: // index at which the batch dim appears so that we can move it to the correct place.
0047: // later down the line via a call to `movedim`.
0048: static std::pair<Tensor,int64_t> remove_existing_batch_dim(
0049:     const BatchedTensorImpl* batched, int64_t level) {
0050:   auto bdims = batched->bdims();
0051:   if (bdims.size() == 1) {
0052:     TORCH_INTERNAL_ASSERT(bdims[0].level() == level);
0053:     return std::make_pair(batched->value(), bdims[0].dim());
0054:   }
0055:   BatchDims new_bdims;
0056:   int64_t newly_exposed_physical_dim = -1;
0057:   new_bdims.reserve(bdims.size() - 1);
0058:   for (const auto& bdim : bdims) {
0059:     if (bdim.level() == level) {
0060:       newly_exposed_physical_dim = bdim.dim();
```
- **EN**: Lines 31-60 mainly cover comments/documentation, state/variable declarations, control-flow checks. Notable symbols: randn, vmap, at, remove_existing_batch_dim.
- **CN**: 第 31-60 行主要涉及注释或说明、变量/别名声明、控制流逻辑。 值得关注的符号包括：randn, vmap, at, remove_existing_batch_dim。

### Lines 61-90 / 第 61-90 行
```cpp
0061:     } else {
0062:       new_bdims.push_back(bdim);
0063:     }
0064:   }
0065:   // Because a BatchDim with level `level` must exist inside `batched,
0066:   // we should have found a `newly_exposed_logical_dim`.
0067:   TORCH_INTERNAL_ASSERT(newly_exposed_physical_dim != -1);
0068:   int64_t num_batch_dims_before_newly_exposed_physical_dim = std::count_if(
0069:       new_bdims.begin(), new_bdims.end(),
0070:       [&](const BatchDim& bdim) {
0071:         return bdim.dim() < newly_exposed_physical_dim;
0072:       });
0073:   int64_t newly_exposed_logical_dim =
0074:       newly_exposed_physical_dim - num_batch_dims_before_newly_exposed_physical_dim;
0075:   auto result_tensor = makeBatched(batched->value(), std::move(new_bdims));
0076:   return std::make_pair(std::move(result_tensor), newly_exposed_logical_dim);
0077: }
0078: 
0079: // at::movedim but may return the original tensor if dst is the same as src.
0080: static Tensor maybe_movedim(const Tensor& self, int64_t src, int64_t dst) {
0081:   auto logical_dim = self.dim();
0082:   src = maybe_wrap_dim(src, logical_dim);
0083:   dst = maybe_wrap_dim(dst, logical_dim);
0084:   if (src == dst) {
0085:     return self;
0086:   }
0087:   return self.movedim(src, dst);
0088: }
0089: 
0090: // Removes the batch dim with level `level` from `self`. If this causes the
```
- **EN**: Lines 61-90 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: push_back, TORCH_INTERNAL_ASSERT, count_if, begin.
- **CN**: 第 61-90 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：push_back, TORCH_INTERNAL_ASSERT, count_if, begin。

### Lines 91-120 / 第 91-120 行
```cpp
0091: // last batch dim to be removed from a BatchedTensor, then this returns a
0092: // regular Tensor.
0093: //
0094: // If the `level` of the batch dim to remove does not exist in `self`, then we
0095: // add the batch dim in. This can happen if `self` didn't interact with a tensor
0096: // inside the vmap level, for example,
0097: //     self = torch.randn(3)
0098: //     y = torch.randn(5)
0099: //     out = vmap(lambda x: vmap(lambda y: x)(y))(self)
0100: //     assert out.shape == (3, 5)
0101: // Inside the inner vmap, `x` is a BatchedTensor with a single batch dimension
0102: // corresponding to the *outer* vmap level and it doesn't have any dimensions that
0103: // correspond to the inner vmap level so we need to create one for the user.
0104: //
0105: // `out_dim` controls where we should put the batch dimension in the output tensor.
0106: Tensor _remove_batch_dim(const Tensor& self, int64_t level, int64_t batch_size, int64_t out_dim) {
0107:   if (!has_level(self, level)) {
0108:     auto self_sizes = self.sizes();
0109:     VmapDimVector expanded_sizes(self_sizes.begin(), self_sizes.end());
0110:     expanded_sizes.insert(expanded_sizes.begin() + out_dim, batch_size);
0111:     return self.expand(expanded_sizes);
0112:   }
0113: 
0114:   // Must be batched if has_level(self, /*any_level*/)
0115:   const auto* batched = maybeGetBatchedImpl(self);
0116:   TORCH_INTERNAL_ASSERT(batched != nullptr);
0117: 
0118:   auto [self_without_bdim, newly_exposed_logical_dim] = remove_existing_batch_dim(batched, level);
0119:   return maybe_movedim(self_without_bdim, newly_exposed_logical_dim, out_dim);
0120: }
```
- **EN**: Lines 91-120 mainly cover comments/documentation, state/variable declarations, return paths. Notable symbols: randn, vmap, _remove_batch_dim, has_level.
- **CN**: 第 91-120 行主要涉及注释或说明、变量/别名声明、返回路径。 值得关注的符号包括：randn, vmap, _remove_batch_dim, has_level。

### Lines 121-122 / 第 121-122 行
```cpp
0121: 
0122: } // namespace at::native
```
- **EN**: Lines 121-122 mainly cover namespace structuring.
- **CN**: 第 121-122 行主要涉及命名空间组织。

## Key Concepts / 关键概念
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现
- **EN**: Native operator implementation path  
  **CN**: 原生算子实现路径

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/LegacyBatchedTensorImpl.h>`, `<ATen/WrapDimUtils.h>`, `<ATen/LegacyVmapTransforms.h>`, `<ATen/ops/_add_batch_dim_native.h>`, `<ATen/ops/_remove_batch_dim_native.h>`
- **Macros / 宏**: None highlighted / 无特别标注
- **Namespaces / 命名空间**: `at::native`, `at::`, `std::`
