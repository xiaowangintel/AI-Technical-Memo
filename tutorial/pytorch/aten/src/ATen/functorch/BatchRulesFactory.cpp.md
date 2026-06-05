# BatchRulesFactory.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/functorch/BatchRulesFactory.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Supports functorch-style transforms, batching, functionalization, or derivative-aware ATen behavior. This specific file centers on `BatchRulesFactory.cpp`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Transform-aware tensor semantics used by functorch are part of the file scope.
- **Purpose (CN)**: 支撑 functorch 风格的变换、批处理、函数化或与导数相关的 ATen 行为。 该文件具体围绕 `BatchRulesFactory.cpp` 展开。 文件头部注释也概括了其核心职责。 该文件范围内还包含 functorch 所需的变换感知张量语义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行

```cpp
0001: // Copyright (c) Facebook, Inc. and its affiliates.
0002: // All rights reserved.
0003: //
0004: // This source code is licensed under the BSD-style license found in the
0005: // LICENSE file in the root directory of this source tree.
0006: 
0007: #include <ATen/functorch/BatchRulesHelper.h>
0008: #include <c10/core/SymIntArrayRef.h>
0009: 
0010: namespace at::functorch {
0011: 
0012: template <typename A, A a, typename C>
0013: struct NewBlahBatchRuleHelperSymInt;
0014: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `NewBlahBatchRuleHelperSymInt`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`NewBlahBatchRuleHelperSymInt`。

### Lines 15-28 / 第 15-28 行

```cpp
0015: template <typename F, F Func, typename A, typename B, typename... T>
0016: struct NewBlahBatchRuleHelperSymInt<F, Func, typelist<A, B, T...>> {
0017:   static std::tuple<Tensor, std::optional<int64_t>> apply(
0018:       const Tensor& tensor,
0019:       std::optional<int64_t> batch_dim,
0020:       SymIntArrayRef shape,
0021:       T... extra_args) {
0022:     // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
0023:     const auto bdim_size = tensor.sym_size(batch_dim.value());
0024:     c10::SmallVector<c10::SymInt> new_shape;
0025:     new_shape.reserve(shape.size() + 1);
0026:     new_shape.emplace_back(bdim_size);
0027:     new_shape.insert(new_shape.end(), shape.begin(), shape.end());
0028:     return std::make_tuple(Func(tensor, new_shape, std::forward<T>(extra_args)...), 0);
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `NewBlahBatchRuleHelperSymInt`, `apply`, `make_tuple`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`NewBlahBatchRuleHelperSymInt`, `apply`, `make_tuple`。

### Lines 29-47 / 第 29-47 行

```cpp
0029:   }
0030: };
0031: 
0032: template <typename A, A a, typename C>
0033: struct NewBlahBatchRuleHelper;
0034: 
0035: template <typename F, F Func, typename A, typename B, typename... T>
0036: struct NewBlahBatchRuleHelper<F, Func, typelist<A, B, T...>> {
0037:   static std::tuple<Tensor, std::optional<int64_t>> apply(
0038:       const Tensor& tensor,
0039:       std::optional<int64_t> batch_dim,
0040:       IntArrayRef shape,
0041:       T... extra_args) {
0042:     const auto bdim_size = tensor.size(batch_dim.value());
0043:     VmapDimVector new_shape;
0044:     new_shape.reserve(shape.size() + 1);
0045:     new_shape.emplace_back(bdim_size);
0046:     new_shape.insert(new_shape.end(), shape.begin(), shape.end());
0047:     return std::make_tuple(Func(tensor, new_shape, std::forward<T>(extra_args)...), 0);
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `NewBlahBatchRuleHelper`, `apply`, `make_tuple`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`NewBlahBatchRuleHelper`, `apply`, `make_tuple`。

### Lines 48-65 / 第 48-65 行

```cpp
0048:   }
0049: };
0050: 
0051: // USAGE: NEW_BLAH_BATCH_RULE(at::new_zeros)
0052: // INCORRECT USAGE: NEW_BLAH_BATCH_RULE(&at::new_zeros)
0053: // It is important that this macro is not passed a function pointer!!
0054: #define NEW_BLAH_BATCH_RULE(fn) SINGLE_ARG(\
0055:     NewBlahBatchRuleHelper<\
0056:       decltype(&fn),\
0057:       &fn,\
0058:       c10::guts::function_traits<decltype(fn)>::parameter_types>::apply)
0059: 
0060: #define NEW_BLAH_BATCH_RULE_SYMINT(fn) SINGLE_ARG(\
0061:     NewBlahBatchRuleHelperSymInt<\
0062:       decltype(&fn),\
0063:       &fn,\
0064:       c10::guts::function_traits<decltype(fn)>::parameter_types>::apply)
0065: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 66-82 / 第 66-82 行

```cpp
0066: static std::tuple<Tensor, std::optional<int64_t>> _new_zeros_with_same_feature_meta_batch_rule(
0067:     const Tensor& self, std::optional<int64_t> self_bdim,
0068:     const Tensor& other, std::optional<int64_t> other_bdim,
0069:     int64_t self_num_batch_dims) {
0070:   // The "self, other" naming is too confusing
0071:   // What this function really says is "create a new tangent for this base".
0072:   const auto& base = other;
0073:   const auto& base_bdim = other_bdim;
0074:   const auto& tangent = self;
0075:   const auto& tangent_bdim = self_bdim;
0076: 
0077:   // Three case:
0078:   //          Case 1  Case 2  Case 3
0079:   // base        [6]  [B, 6]  [B, 6]
0080:   // tangent  [B, 5]     [5]  [B, 5]
0081:   // result   [B, 6]  [B, 6]  [B, 6]
0082: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values. Key symbols: `_new_zeros_with_same_feature_meta_batch_rule`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值。关键符号：`_new_zeros_with_same_feature_meta_batch_rule`。

### Lines 83-98 / 第 83-98 行

```cpp
0083:   // Case 2 & 3
0084:   if (base_bdim) {
0085:     auto base_ = moveBatchDimToFront(base, base_bdim);
0086:     Tensor tangent_ = tangent;
0087:     if (tangent_bdim.has_value()) {
0088:       // tangent  [B, K0, K1, 5]
0089:       // base_            [B, 6]
0090:       // We want to move B to after the Ks, so that self_num_batch_dims
0091:       // (which really means tangent_num_batch_dims) isn't interfered with.
0092:       // [B, K0, K1, 6] -> [K0, K1, B, 6]
0093:       //
0094:       // [K0, K1, B, 6], [B, 5], 2 -> [K0, K1, B, 5]
0095:       tangent_ = tangent.movedim(*tangent_bdim, self_num_batch_dims);
0096:     }
0097:     auto result = at::_new_zeros_with_same_feature_meta(tangent_, base_, self_num_batch_dims);
0098:     return std::make_tuple(std::move(result), self_num_batch_dims);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`make_tuple`。

### Lines 99-122 / 第 99-122 行

```cpp
0099:   }
0100: 
0101:   // Case 1:
0102:   auto tangent_ = moveBatchDimToFront(tangent, tangent_bdim);
0103:   auto result = at::_new_zeros_with_same_feature_meta(tangent_, base, self_num_batch_dims + 1);
0104:   return std::make_tuple(std::move(result), 0);
0105: }
0106: 
0107: static std::tuple<Tensor, std::optional<int64_t>> linspace_logspace_batch_rule_helper(
0108:     const at::Tensor& start, std::optional<int64_t> start_bdim,
0109:     const at::Tensor& end, std::optional<int64_t> end_bdim,
0110:     int64_t steps,
0111:     std::optional<double> base,
0112:     std::optional<at::ScalarType> dtype,
0113:     std::optional<at::Layout> layout,
0114:     std::optional<at::Device> device,
0115:     std::optional<bool> pin_memory)
0116: {
0117:   auto batch_size = get_bdim_size2(start, start_bdim, end, end_bdim);
0118:   auto start_ = ensure_has_bdim(start, start_bdim.has_value(), batch_size);
0119:   auto end_ = ensure_has_bdim(end, end_bdim.has_value(), batch_size);
0120:   start_ = moveBatchDimToFront(start_, start_bdim);
0121:   end_ = moveBatchDimToFront(end_, end_bdim);
0122: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `make_tuple`, `linspace_logspace_batch_rule_helper`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`make_tuple`, `linspace_logspace_batch_rule_helper`。

### Lines 123-137 / 第 123-137 行

```cpp
0123:   auto tensor_options = at::TensorOptions().dtype(dtype).layout(layout).device(device).pinned_memory(pin_memory);
0124: 
0125:   Tensor result;
0126:   if (steps == 0){
0127:     result = at::full({batch_size, 0}, 0, tensor_options);
0128:   } else if (steps == 1){
0129:     result = start_.new_empty({batch_size}, tensor_options).copy_(start_).unsqueeze(1);
0130:   } else {
0131:     result = (start_ + at::arange(0, steps, tensor_options).unsqueeze_(1) * (end_ - start_) / (steps - 1)).transpose(0, 1);
0132:   }
0133: 
0134:   if (base){
0135:     result = at::pow(*base, result);
0136:   }
0137: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 138-153 / 第 138-153 行

```cpp
0138:   if (dtype && result.scalar_type() != *dtype){
0139:     result = result.to(*dtype);
0140:   }
0141: 
0142:   return std::make_tuple(std::move(result), 0);
0143: }
0144: 
0145: static std::tuple<Tensor, std::optional<int64_t>> linspace_Tensor_Tensor_batch_rule(
0146:     const at::Tensor& start, std::optional<int64_t> start_bdim,
0147:     const at::Tensor& end, std::optional<int64_t> end_bdim,
0148:     int64_t steps,
0149:     std::optional<at::ScalarType> dtype,
0150:     std::optional<at::Layout> layout,
0151:     std::optional<at::Device> device,
0152:     std::optional<bool> pin_memory){
0153:   return linspace_logspace_batch_rule_helper(start, start_bdim, end, end_bdim, steps, std::nullopt, dtype, layout, device, pin_memory);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `make_tuple`, `linspace_Tensor_Tensor_batch_rule`, `linspace_logspace_batch_rule_helper`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`make_tuple`, `linspace_Tensor_Tensor_batch_rule`, `linspace_logspace_batch_rule_helper`。

### Lines 154-168 / 第 154-168 行

```cpp
0154: }
0155: 
0156: static std::tuple<Tensor, std::optional<int64_t>> linspace_Tensor_Scalar_batch_rule(
0157:     const at::Tensor& start, std::optional<int64_t> start_bdim,
0158:     const at::Scalar& end,
0159:     int64_t steps,
0160:     std::optional<at::ScalarType> dtype,
0161:     std::optional<at::Layout> layout,
0162:     std::optional<at::Device> device,
0163:     std::optional<bool> pin_memory){
0164: 
0165:   auto end_t = at::native::wrapped_scalar_tensor(end, start.device());
0166:   return linspace_logspace_batch_rule_helper(start, start_bdim, end_t, std::nullopt, steps, std::nullopt, dtype, layout, device, pin_memory);
0167: }
0168: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `linspace_Tensor_Scalar_batch_rule`, `linspace_logspace_batch_rule_helper`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`linspace_Tensor_Scalar_batch_rule`, `linspace_logspace_batch_rule_helper`。

### Lines 169-182 / 第 169-182 行

```cpp
0169: static std::tuple<Tensor, std::optional<int64_t>> linspace_Scalar_Tensor_batch_rule(
0170:     const at::Scalar& start,
0171:     const at::Tensor& end, std::optional<int64_t> end_bdim,
0172:     int64_t steps,
0173:     std::optional<at::ScalarType> dtype,
0174:     std::optional<at::Layout> layout,
0175:     std::optional<at::Device> device,
0176:     std::optional<bool> pin_memory){
0177: 
0178:   auto start_t = at::native::wrapped_scalar_tensor(start, end.device());
0179:   return linspace_logspace_batch_rule_helper(start_t, std::nullopt, end, end_bdim, steps, std::nullopt, dtype, layout, device, pin_memory);
0180: }
0181: 
0182: static std::tuple<Tensor, std::optional<int64_t>> logspace_Tensor_Tensor_batch_rule(
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `linspace_Scalar_Tensor_batch_rule`, `linspace_logspace_batch_rule_helper`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`linspace_Scalar_Tensor_batch_rule`, `linspace_logspace_batch_rule_helper`。

### Lines 183-203 / 第 183-203 行

```cpp
0183:     const at::Tensor& start, std::optional<int64_t> start_bdim,
0184:     const at::Tensor& end, std::optional<int64_t> end_bdim,
0185:     int64_t steps,
0186:     double base,
0187:     std::optional<at::ScalarType> dtype,
0188:     std::optional<at::Layout> layout,
0189:     std::optional<at::Device> device,
0190:     std::optional<bool> pin_memory){
0191:   return linspace_logspace_batch_rule_helper(start, start_bdim, end, end_bdim, steps, base, dtype, layout, device, pin_memory);
0192: }
0193: 
0194: static std::tuple<Tensor, std::optional<int64_t>> logspace_Tensor_Scalar_batch_rule(
0195:     const at::Tensor& start, std::optional<int64_t> start_bdim,
0196:     const at::Scalar& end,
0197:     int64_t steps,
0198:     double base,
0199:     std::optional<at::ScalarType> dtype,
0200:     std::optional<at::Layout> layout,
0201:     std::optional<at::Device> device,
0202:     std::optional<bool> pin_memory){
0203: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `linspace_logspace_batch_rule_helper`, `logspace_Tensor_Scalar_batch_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`linspace_logspace_batch_rule_helper`, `logspace_Tensor_Scalar_batch_rule`。

### Lines 204-217 / 第 204-217 行

```cpp
0204:   auto end_t = at::native::wrapped_scalar_tensor(end, start.device());
0205:   return linspace_logspace_batch_rule_helper(start, start_bdim, end_t, std::nullopt, steps, base, dtype, layout, device, pin_memory);
0206: }
0207: 
0208: static std::tuple<Tensor, std::optional<int64_t>> logspace_Scalar_Tensor_batch_rule(
0209:     const at::Scalar& start,
0210:     const at::Tensor& end, std::optional<int64_t> end_bdim,
0211:     int64_t steps,
0212:     double base,
0213:     std::optional<at::ScalarType> dtype,
0214:     std::optional<at::Layout> layout,
0215:     std::optional<at::Device> device,
0216:     std::optional<bool> pin_memory){
0217: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `linspace_logspace_batch_rule_helper`, `logspace_Scalar_Tensor_batch_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`linspace_logspace_batch_rule_helper`, `logspace_Scalar_Tensor_batch_rule`。

### Lines 218-245 / 第 218-245 行

```cpp
0218:   auto start_t = at::native::wrapped_scalar_tensor(start, end.device());
0219:   return linspace_logspace_batch_rule_helper(start_t, std::nullopt, end, end_bdim, steps, base, dtype, layout, device, pin_memory);
0220: }
0221: 
0222: static bool _has_same_storage_numel_batch_rule(const Tensor& a, const Tensor& b) {
0223:   return true;
0224: }
0225: 
0226: TORCH_LIBRARY_IMPL(aten, FuncTorchBatched, m) {
0227:   m.impl("_has_same_storage_numel", _has_same_storage_numel_batch_rule);
0228:   VMAP_SUPPORT(ones_like, BASIC_UNARY_BATCH_RULE(ATEN_FN(ones_like)));
0229:   VMAP_SUPPORT(zeros_like, BASIC_UNARY_BATCH_RULE(ATEN_FN(zeros_like)));
0230:   VMAP_SUPPORT(empty_like, BASIC_UNARY_BATCH_RULE(ATEN_FN(empty_like)));
0231:   VMAP_SUPPORT(randn_like, BASIC_UNARY_BATCH_RULE(ATEN_FN(randn_like)));
0232:   VMAP_SUPPORT(rand_like, BASIC_UNARY_BATCH_RULE(ATEN_FN(rand_like)));
0233:   VMAP_SUPPORT(full_like, BASIC_UNARY_BATCH_RULE(ATEN_FN(full_like)));
0234:   VMAP_SUPPORT(new_empty, NEW_BLAH_BATCH_RULE_SYMINT(ATEN_FN(new_empty)));
0235:   VMAP_SUPPORT(new_zeros, NEW_BLAH_BATCH_RULE_SYMINT(ATEN_FN(new_zeros)));
0236:   VMAP_SUPPORT(new_ones, NEW_BLAH_BATCH_RULE_SYMINT(ATEN_FN(new_ones)));
0237:   VMAP_SUPPORT(new_full, NEW_BLAH_BATCH_RULE_SYMINT(ATEN_FN(new_full)));
0238:   VMAP_SUPPORT2(linspace, Tensor_Tensor, linspace_Tensor_Tensor_batch_rule);
0239:   VMAP_SUPPORT2(linspace, Tensor_Scalar, linspace_Tensor_Scalar_batch_rule);
0240:   VMAP_SUPPORT2(linspace, Scalar_Tensor, linspace_Scalar_Tensor_batch_rule);
0241:   VMAP_SUPPORT2(logspace, Tensor_Tensor, logspace_Tensor_Tensor_batch_rule);
0242:   VMAP_SUPPORT2(logspace, Tensor_Scalar, logspace_Tensor_Scalar_batch_rule);
0243:   VMAP_SUPPORT2(logspace, Scalar_Tensor, logspace_Scalar_Tensor_batch_rule);
0244:   VMAP_SUPPORT(_new_zeros_with_same_feature_meta, _new_zeros_with_same_feature_meta_batch_rule);
0245:   // Not sure how to add the ones with irregular args to the mix cleanly (i.e. randint takes an extra int parameter)
```

- **EN:** Registers dispatch hooks, operator entry points, or specialization glue so runtime code can discover `BatchRulesFactory` behavior. Symbols: `linspace_logspace_batch_rule_helper`, `_has_same_storage_numel_batch_rule`.
- **CN:** 注册调度钩子、算子入口或特化胶水代码，使运行时能够发现 `BatchRulesFactory` 的行为。符号：`linspace_logspace_batch_rule_helper`, `_has_same_storage_numel_batch_rule`。

### Lines 246-247 / 第 246-247 行

```cpp
0246: }
0247: } // namespace at::functorch
```

- **EN:** Opens or closes namespace scopes so the remaining declarations stay inside the intended ATen/backend module.
- **CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 ATen/后端模块之内。


## Key Concepts / 关键概念
- **functorch-aware tensor transforms** — 面向 functorch 的张量变换
- **Tensor metadata and value flow** — 张量元数据与数值流
- **functorch transform support** — functorch 变换支持
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: NewBlahBatchRuleHelperSymInt, NewBlahBatchRuleHelper, apply, make_tuple, linspace_logspace_batch_rule_helper, linspace_Tensor_Tensor_batch_rule, linspace_Tensor_Scalar_batch_rule, linspace_Scalar_Tensor_batch_rule** — 核心符号：NewBlahBatchRuleHelperSymInt、NewBlahBatchRuleHelper、apply、make_tuple、linspace_logspace_batch_rule_helper、linspace_Tensor_Tensor_batch_rule、linspace_Tensor_Scalar_batch_rule、linspace_Scalar_Tensor_batch_rule

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/functorch/BatchRulesHelper.h`, `c10/core/SymIntArrayRef.h`
- **External includes / 外部头文件**: 无
- **Namespaces / 命名空间**: `at::functorch`
- **Representative symbols / 代表性符号**: `NewBlahBatchRuleHelperSymInt`, `NewBlahBatchRuleHelper`, `apply`, `make_tuple`, `linspace_logspace_batch_rule_helper`, `linspace_Tensor_Tensor_batch_rule`, `linspace_Tensor_Scalar_batch_rule`, `linspace_Scalar_Tensor_batch_rule`, `logspace_Tensor_Tensor_batch_rule`, `logspace_Tensor_Scalar_batch_rule`, `logspace_Scalar_Tensor_batch_rule`, `_has_same_storage_numel_batch_rule`
