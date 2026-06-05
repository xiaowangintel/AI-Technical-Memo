# ComplexHelper.h — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/ComplexHelper.h`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Complex Helper. As a header, it exposes declarations and shared helpers.
- **Purpose (CN)**: 实现或声明与 complex、helper 相关的 ATen 原生逻辑。 作为头文件，它暴露声明与共享辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #pragma once
0002: 
0003: #include <ATen/core/Tensor.h>
0004: #include <c10/core/SymBool.h>
0005: #include <c10/util/irange.h>
0006: 
0007: #ifndef AT_PER_OPERATOR_HEADERS
0008: #include <ATen/NativeFunctions.h>
0009: #else
0010: #include <ATen/ops/view_as_real_native.h>
0011: #include <ATen/ops/view_as_complex_native.h>
0012: 
0013: #include <utility>
0014: #endif
0015: 
0016: // WARNING: this header contains non-inline functions and should be only
0017: // included from ONE cpp file
0018: 
0019: namespace at::native {
0020: 
0021: // View tensor with new dtype, storage offset, sizes and strides
0022: inline Tensor view_tensor(
0023:     const Tensor &tensor, ScalarType dtype,
0024:     c10::SymInt offset, SymIntArrayRef sizes, SymIntArrayRef strides) {
0025:   Storage storage = tensor.storage();
0026:   auto key_set = tensor.key_set().remove(DispatchKey::Conjugate);
0027:   auto new_tensor = detail::make_tensor<TensorImpl>(
0028:       c10::TensorImpl::VIEW, std::move(storage), key_set, scalarTypeToTypeMeta(dtype));
0029:   auto * impl = new_tensor.unsafeGetTensorImpl();
0030:   impl->set_sizes_and_strides(sizes, strides, offset);
```
- **EN**: Lines 1-30 mainly cover header inclusion, state/variable declarations, conditional compilation. Notable symbols: view_tensor, storage, key_set, remove.
- **CN**: 第 1-30 行主要涉及头文件包含、变量/别名声明、预处理条件。 值得关注的符号包括：view_tensor, storage, key_set, remove。

### Lines 31-60 / 第 31-60 行
```cpp
0031:   return new_tensor;
0032: }
0033: 
0034: inline SymDimVector computeStrideForViewAsReal(SymIntArrayRef oldstride) {
0035:   SymDimVector res(oldstride.size() + 1);
0036:   for (const auto i : c10::irange(oldstride.size())) {
0037:     res[i] = oldstride[i] * 2;
0038:   }
0039:   res.back() = 1;
0040:   return res;
0041: }
0042: 
0043: inline Tensor _view_as_real_physical(const Tensor& self) {
0044:   TORCH_CHECK(self.is_complex(), "view_as_real is only supported for complex tensors");
0045:   auto old_sizes = self.sym_sizes();
0046:   SymDimVector new_sizes(old_sizes.size() + 1);
0047:   std::copy(old_sizes.begin(), old_sizes.end(), new_sizes.begin());
0048:   // last dimension will always have two elements containing the real and imag vals
0049:   new_sizes.back() = 2;
0050:   auto new_strides = computeStrideForViewAsReal(self.sym_strides());
0051:   auto new_storage_offset = self.sym_storage_offset() * 2;
0052:   const auto float_type = c10::toRealValueType(self.scalar_type());
0053:   auto real_tensor = view_tensor(self, float_type, std::move(new_storage_offset), new_sizes, new_strides);
0054:   return real_tensor;
0055: }
0056: 
0057: // expects as input a complex tensor and returns back a tensor
0058: // with corresponding real dtype containing the complex values
0059: // in the last two dimensions
0060: Tensor view_as_real(const Tensor& self) {
```
- **EN**: Lines 31-60 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: computeStrideForViewAsReal, res, size, irange.
- **CN**: 第 31-60 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：computeStrideForViewAsReal, res, size, irange。

### Lines 61-90 / 第 61-90 行
```cpp
0061:   TORCH_CHECK(!self.is_conj(), "view_as_real doesn't work on unresolved conjugated tensors.  To resolve the conjugate tensor so you can view it as real, use self.resolve_conj(); however, be warned that the resulting tensor will NOT alias the original.");
0062:   return _view_as_real_physical(self);
0063: }
0064: 
0065: inline SymDimVector computeStrideForViewAsComplex(
0066:     SymIntArrayRef oldstride,
0067:     SymIntArrayRef oldsizes) {
0068:   const auto dim = oldstride.size();
0069:   TORCH_CHECK(dim > 0, "Tensor must have one or more dimensions");
0070:   TORCH_SYM_CHECK(oldstride[dim - 1].sym_eq(1), "Tensor must have a last dimension with stride 1");
0071: 
0072:   SymDimVector res(dim - 1);
0073:   for (const auto i : c10::irange(res.size())) {
0074:     // Skip divisibility check for singleton dimensions
0075:     TORCH_SYM_CHECK(
0076:         oldsizes[i].sym_eq(1) | (oldstride[i] % 2).sym_eq(0),
0077:         "Tensor must have a stride divisible by 2 for all but last dimension");
0078:     res[i] = oldstride[i] / 2;
0079:   }
0080:   return res;
0081: }
0082: 
0083: // expects as input a float or double tensor with last dimension of size 2
0084: // and returns back a tensor with corresponding complex dtype
0085: Tensor view_as_complex(const Tensor& self) {
0086:   TORCH_CHECK(
0087:     self.scalar_type() == kFloat || self.scalar_type() == kDouble || self.scalar_type() == kHalf,
0088:     "view_as_complex is only supported for half, float and double tensors, but got a tensor of scalar type: ", self.scalar_type());
0089: 
0090:   auto old_sizes = self.sym_sizes();
```
- **EN**: Lines 61-90 mainly cover state/variable declarations, macro-based glue, expressions/calls. Notable symbols: TORCH_CHECK, is_conj, resolve_conj, _view_as_real_physical.
- **CN**: 第 61-90 行主要涉及变量/别名声明、宏定义或宏调用、表达式或调用。 值得关注的符号包括：TORCH_CHECK, is_conj, resolve_conj, _view_as_real_physical。

### Lines 91-104 / 第 91-104 行
```cpp
0091:   TORCH_CHECK(!old_sizes.empty(), "Input tensor must have one or more dimensions");
0092:   TORCH_SYM_CHECK(old_sizes[old_sizes.size()-1].sym_eq(2), "Tensor must have a last dimension of size 2");
0093:   SymDimVector new_sizes(old_sizes.begin(), old_sizes.end() - 1);
0094: 
0095:   const auto new_strides = computeStrideForViewAsComplex(self.sym_strides(), self.sym_sizes());
0096:   const auto complex_type = c10::toComplexType(self.scalar_type());
0097: 
0098:   TORCH_SYM_CHECK((self.sym_storage_offset() % 2).sym_eq(0), "Tensor must have a storage_offset divisible by 2");
0099:   const auto new_storage_offset = self.sym_storage_offset() / 2;
0100: 
0101:   return view_tensor(self, complex_type, new_storage_offset, new_sizes, new_strides);
0102: }
0103: 
0104: } // namespace at::native
```
- **EN**: Lines 91-104 mainly cover state/variable declarations, macro-based glue, return paths. Notable symbols: TORCH_CHECK, empty, TORCH_SYM_CHECK, size.
- **CN**: 第 91-104 行主要涉及变量/别名声明、宏定义或宏调用、返回路径。 值得关注的符号包括：TORCH_CHECK, empty, TORCH_SYM_CHECK, size。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现
- **EN**: Scalar/tensor mixed arithmetic  
  **CN**: 标量与张量混合运算
- **EN**: Dispatch-key aware behavior  
  **CN**: 感知 DispatchKey 的行为控制

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<c10/core/SymBool.h>`, `<c10/util/irange.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/view_as_real_native.h>`, `<ATen/ops/view_as_complex_native.h>`, `<utility>`
- **Macros / 宏**: `TORCH_CHECK`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
