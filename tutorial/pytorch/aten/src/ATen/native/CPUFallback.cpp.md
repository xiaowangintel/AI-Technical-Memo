# CPUFallback.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/CPUFallback.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to CPUFallback.
- **Purpose (CN)**: 实现或声明与 cpufallback 相关的 ATen 原生逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/native/CPUFallback.h>
0003: 
0004: #include <ATen/core/ivalue.h>
0005: #include <ATen/core/stack.h>
0006: #include <ATen/core/dispatch/Dispatcher.h>
0007: 
0008: #include <sstream>
0009: #include <vector>
0010: 
0011: 
0012: #ifndef AT_PER_OPERATOR_HEADERS
0013: #include <ATen/Functions.h>
0014: #else
0015: #include <ATen/ops/_copy_from_and_resize.h>
0016: #include <ATen/ops/_to_cpu.h>
0017: #endif
0018: 
0019: 
0020: namespace at::native {
0021: 
0022: // convenience helper for converting tensors to cpu
0023: 
0024: template<typename T, std::enable_if_t<std::is_same_v<T, at::Tensor> || std::is_same_v<T, std::optional<at::Tensor>>, int> = 1>
0025: static std::vector<T> to_cpu(const std::vector<T>& tensors) {
0026:     // We can't just call at::to_cpu() on the entire list of Tensors
0027:     // Because it will break on undefined tensors. Separate out undefined tensors first.
0028:     const int num = tensors.size();
0029:     std::vector<T> cpu_tensors(num);
0030:     std::vector<at::Tensor> valid_tensors;
```
- **EN**: Lines 1-30 mainly cover header inclusion, state/variable declarations, conditional compilation. Notable symbols: to_cpu, size, cpu_tensors.
- **CN**: 第 1-30 行主要涉及头文件包含、变量/别名声明、预处理条件。 值得关注的符号包括：to_cpu, size, cpu_tensors。

### Lines 31-60 / 第 31-60 行
```cpp
0031:     std::vector<bool> to_translate(num);
0032:     for (const auto i : c10::irange(num)) {
0033:       // Explicitly handling undefined tensors here instead of letting `at::_to_cpu` handle it.
0034:       // Otherwise, we'd need to require all backends with their own implementation of _to_cpu
0035:       // to properly handle undefined tensors.
0036:       if constexpr(std::is_same_v<T, std::optional<at::Tensor>>) {
0037:         if (tensors[i].has_value() && tensors[i].value().defined()) {
0038:           to_translate[i] = true;
0039:           valid_tensors.push_back(tensors[i].value());
0040:         } else {
0041:           cpu_tensors[i] = tensors[i];
0042:         }
0043:       } else {
0044:         if (tensors[i].defined()) {
0045:           to_translate[i] = true;
0046:           valid_tensors.push_back(tensors[i]);
0047:         } else {
0048:           cpu_tensors[i] = tensors[i];
0049:         }
0050:       }
0051:     }
0052:     auto cpu_valid_tensors = at::_to_cpu(valid_tensors);
0053:     for (int i = 0, defined_pos = 0; i < num; ++i) {
0054:       if (to_translate[i]) {
0055:         cpu_tensors[i] = std::move(cpu_valid_tensors[defined_pos++]);
0056:       }
0057:     }
0058:   return cpu_tensors;
0059: }
0060: 
```
- **EN**: Lines 31-60 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: to_translate, irange, constexpr, has_value.
- **CN**: 第 31-60 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：to_translate, irange, constexpr, has_value。

### Lines 61-90 / 第 61-90 行
```cpp
0061: static std::optional<c10::Device> compute_target_device(std::vector<at::Tensor>& t_args, const std::vector<c10::List<at::Tensor>>& tlist_args) {
0062:   // Decide what device to move the output tensor(s) to.
0063:   // The current convention is that we use the first tensor arg to pick the device
0064:   // Barring that, we take the first tensor from a TensorList arg.
0065:   if (!t_args.empty()) {
0066:     return t_args[0].device();
0067:   } else {
0068:     // We need to loop through all of the (potentially multiple) TensorList arguments
0069:     // In case, e.g. the first one is empty but the second is not.
0070:     for (auto& tens_list : tlist_args) {
0071:       for (const auto i : c10::irange(tens_list.size())) {
0072:         return tens_list.get(i).device();
0073:       }
0074:     }
0075:   }
0076:   return std::nullopt;
0077: }
0078: 
0079: static bool validate_tensor_list(const c10::List<at::Tensor>& tensorlist) {
0080:   bool flag = false;
0081: 
0082:   for (const auto& i : c10::irange(tensorlist.size())) {
0083:     if (tensorlist[i].defined())
0084:       flag = true;
0085:   }
0086: 
0087:   return flag;
0088: }
0089: 
0090: void cpu_fallback(const c10::OperatorHandle& op, torch::jit::Stack* stack, bool error_on_views,
```
- **EN**: Lines 61-90 mainly cover expressions/calls, comments/documentation, control-flow checks. Notable symbols: compute_target_device, tensor, empty, device.
- **CN**: 第 61-90 行主要涉及表达式或调用、注释或说明、控制流逻辑。 值得关注的符号包括：compute_target_device, tensor, empty, device。

### Lines 91-120 / 第 91-120 行
```cpp
0091:                   c10::DispatchKey cpu_dispatch_key) {
0092:   TORCH_CHECK(c10::BackendComponent::CPUBit == c10::toBackendComponent(cpu_dispatch_key),
0093:               "Expected CPU backend DispatchKey but got ",
0094:               c10::toString(cpu_dispatch_key));
0095:   auto& schema_args = op.schema().arguments();
0096:   const auto num_arguments = schema_args.size();
0097:   auto arguments = torch::jit::last(stack, num_arguments);
0098:   const auto arguments_begin = stack->size() - num_arguments;
0099: 
0100:   std::vector<at::Tensor> tensor_args;
0101:   std::vector<size_t> tensor_args_indices;
0102: 
0103:   std::vector<c10::List<at::Tensor>> tensorlist_args;
0104:   std::vector<size_t> tensorlist_args_indices;
0105: 
0106:   std::vector<c10::List<std::optional<at::Tensor>>> optional_tensorlist_args;
0107:   std::vector<size_t> optional_tensorlist_args_indices;
0108: 
0109:   std::optional<c10::Device> tgt_device = std::nullopt;
0110:   // save converted cpu tensor for TensorList and optional TensorList
0111:   std::vector<c10::IValue> tensorlist_cpu_args;
0112:   std::vector<c10::IValue> optional_tensorlist_cpu_args;
0113: 
0114:   // Step 1: Convert all non-CPU tensor inputs into CPU tensors
0115:   // and put them on the stack at the correct indices.
0116:   for (const auto idx : c10::irange(arguments.size())) {
0117:     const auto& ivalue = arguments[idx];
0118:     if (ivalue.isTensor()) {
0119:       tensor_args.push_back(ivalue.toTensor());
0120:       tensor_args_indices.push_back(idx);
```
- **EN**: Lines 91-120 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: TORCH_CHECK, toBackendComponent, toString, schema.
- **CN**: 第 91-120 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：TORCH_CHECK, toBackendComponent, toString, schema。

### Lines 121-150 / 第 121-150 行
```cpp
0121:     } else if (ivalue.isTensorList()) {
0122:       // Note: we copy each TensorList argument to CPU individually out of convenience,
0123:       // but XLA would benefit from materializing all tensor and TensorList args onto the CPU at the same time.
0124:       // We can improve this if we need better perf for XLA's CPU fallbacks.
0125:       tensorlist_args.push_back(ivalue.toTensorList());
0126:       tensorlist_args_indices.push_back(idx);
0127:       auto cpu_ivalue = c10::IValue(c10::List<at::Tensor>(to_cpu(ivalue.toTensorVector())));
0128:       tensorlist_cpu_args.push_back(cpu_ivalue);
0129:       (*stack)[arguments_begin + idx] = std::move(cpu_ivalue);
0130:     } else if (ivalue.isOptionalTensorList()) {
0131:       optional_tensorlist_args.push_back(ivalue.toOptionalTensorList());
0132:       optional_tensorlist_args_indices.push_back(idx);
0133:       auto cpu_ivalue = c10::IValue(c10::List<std::optional<at::Tensor>>(to_cpu(ivalue.toOptionalTensorVector())));
0134:       optional_tensorlist_cpu_args.push_back(cpu_ivalue);
0135:       (*stack)[arguments_begin + idx] = c10::IValue(cpu_ivalue);
0136:     } else if (ivalue.isDevice()) {
0137:       tgt_device = ivalue.toDevice();
0138:       (*stack)[arguments_begin + idx] = c10::IValue(c10::Device(kCPU));
0139:     }
0140:   }
0141:   // XLA requires all of the tensor arguments to be gathered up and converted to CPU together.
0142:   auto cpu_tensors = to_cpu(tensor_args);
0143: 
0144:   for (const auto i : c10::irange(tensor_args_indices.size())) {
0145:     auto idx = tensor_args_indices[i];
0146:     (*stack)[arguments_begin + idx] = c10::IValue(cpu_tensors[i]);
0147:   }
0148: 
0149:   // Step 2: Call the underlying CPU implementation of the operator
0150:   op.redispatchBoxed(c10::DispatchKeySet(cpu_dispatch_key), stack);
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, comments/documentation, function signatures/definitions. Notable symbols: isTensorList, push_back, toTensorList, IValue.
- **CN**: 第 121-150 行主要涉及变量/别名声明、注释或说明、函数签名或实现。 值得关注的符号包括：isTensorList, push_back, toTensorList, IValue。

### Lines 151-180 / 第 151-180 行
```cpp
0151: 
0152:   // Step 3: We need to take special care to handle mutable aliases properly:
0153:   // If any input tensors are mutable aliases, we need to
0154:   // directly copy the updated data on the CPU tensors back to the original inputs.
0155:   for (const auto i : c10::irange(tensor_args_indices.size())) {
0156:     auto tensor_idx = tensor_args_indices[i];
0157:     const AliasInfo* alias_info = schema_args[tensor_idx].alias_info();
0158:     if (alias_info != nullptr && alias_info->isWrite()) {
0159:       if (!tensor_args[i].defined()) continue;
0160:       at::_copy_from_and_resize(cpu_tensors[i], tensor_args[i]);
0161:     }
0162:   }
0163: 
0164:   // We also need to explicit reapply input mutations to inputs that are lists
0165:   // of tensors
0166:   for (const auto i : c10::irange(tensorlist_args_indices.size())) {
0167:     auto tensorlist_idx = tensorlist_args_indices[i];
0168:     const AliasInfo* alias_info = schema_args[tensorlist_idx].alias_info();
0169:     if (alias_info != nullptr && alias_info->isWrite()) {
0170:       const auto& cpu_tensors = tensorlist_cpu_args[i].toTensorVector();
0171:       for (const auto idx : c10::irange(tensorlist_args[i].size())) {
0172:         if (!cpu_tensors[idx].defined()) continue;
0173:         at::_copy_from_and_resize(cpu_tensors[idx], tensorlist_args[i][idx]);
0174:       }
0175:     }
0176:   }
0177: 
0178:   // We also need to explicit reapply input mutations to inputs that are lists
0179:   // of optional tensors
0180:   for (const auto i : c10::irange(optional_tensorlist_args_indices.size())) {
```
- **EN**: Lines 151-180 mainly cover control-flow checks, comments/documentation, state/variable declarations. Notable symbols: irange, size, alias_info, isWrite.
- **CN**: 第 151-180 行主要涉及控制流逻辑、注释或说明、变量/别名声明。 值得关注的符号包括：irange, size, alias_info, isWrite。

### Lines 181-210 / 第 181-210 行
```cpp
0181:     auto tensorlist_idx = optional_tensorlist_args_indices[i];
0182:     const AliasInfo* alias_info = schema_args[tensorlist_idx].alias_info();
0183:     if (alias_info != nullptr && alias_info->isWrite()) {
0184:       const auto& cpu_tensors = optional_tensorlist_cpu_args[i].toOptionalTensorList();
0185:       for (const auto idx : c10::irange(optional_tensorlist_args[i].size())) {
0186:         if (cpu_tensors[idx].has_value() && cpu_tensors[idx].value().defined()) {
0187:           const std::optional<at::Tensor>& optional_tensor = optional_tensorlist_args[i][idx];
0188:           at::_copy_from_and_resize(cpu_tensors[idx].value(), optional_tensor.value());
0189:         }
0190:       }
0191:     }
0192:   }
0193: 
0194:   // Step 4: Convert any CPU output tensors back to the original input device.
0195:   // For mutable alias'd outputs, we also need to take special care
0196:   // to move the ORIGINAL input tensor back onto the stack, in place of
0197:   // the temporary CPU output tensor that we created.
0198:   //
0199:   // Note [CPU Fallback Does Not Handle View Operators]
0200:   // Also note that we are incapable of handling immutable aliases properly.
0201:   // Why?
0202:   // Schemas with an immutable alias'd tensor outputs correspond to view operators.
0203:   // For example, the `view_as` schema from native_functions.yaml:
0204:   // `view_as(Tensor(a) self, Tensor other) -> Tensor(a)`
0205:   // We can't handle these ops properly, because view ops are supposed to return
0206:   // a NEW tensor that shares the SAME storage as the original tensor.
0207:   // However, the new tensor that we created cannot share the same storage,
0208:   // since it lives on CPU and the original tensor lives on a different device.
0209:   // Because of that, we warn if someone attempts to call the
0210:   // CPU fallback on a view operator (this is to maintain BC for view ops for XLA
```
- **EN**: Lines 181-210 mainly cover comments/documentation, state/variable declarations, expressions/calls. Notable symbols: alias_info, isWrite, toOptionalTensorList, irange.
- **CN**: 第 181-210 行主要涉及注释或说明、变量/别名声明、表达式或调用。 值得关注的符号包括：alias_info, isWrite, toOptionalTensorList, irange。

### Lines 211-240 / 第 211-240 行
```cpp
0211:   // that fall back to CPU).
0212:   const auto& schema_returns = op.schema().returns();
0213:   const auto& num_returns = schema_returns.size();
0214:   auto returns = torch::jit::last(stack, num_returns);
0215:   const auto returns_begin = stack->size() - num_returns;
0216: 
0217:   if (!tgt_device.has_value()){
0218:     tgt_device = compute_target_device(tensor_args, tensorlist_args);
0219:   }
0220: 
0221:   for (const auto idx : c10::irange(returns.size())) {
0222:     const AliasInfo* alias_info = schema_returns[idx].alias_info();
0223:     if (alias_info != nullptr && alias_info->isWrite()) {
0224:       // Case (1): mutable alias case.
0225:       // Move the input ivalue directly onto the stack in place of
0226:       // the existing cpu output tensor.
0227:       bool found_alias = false;
0228:       if (returns[idx].isTensor() && returns[idx].toTensor().defined()) {
0229:         // We could store some extra metadata on the function schema to avoid
0230:         // the loop here if we need to improve perf.
0231:         for (const auto i : c10::irange(tensor_args_indices.size())) {
0232:           auto input_tensor_idx = tensor_args_indices[i];
0233:           const auto& input_tensor = cpu_tensors[i];
0234:           const AliasInfo* input_alias_info =
0235:               schema_args[input_tensor_idx].alias_info();
0236:           // Checked above; adding assert to guard against breakage of the below
0237:           // condition due to changing the above if test.
0238:           TORCH_INTERNAL_ASSERT_DEBUG_ONLY(alias_info != nullptr);
0239:           if (input_tensor.defined() &&
0240:               (alias_info == input_alias_info ||
```
- **EN**: Lines 211-240 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: schema, returns, size, last.
- **CN**: 第 211-240 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：schema, returns, size, last。

### Lines 241-270 / 第 241-270 行
```cpp
0241:                (input_alias_info != nullptr &&
0242:                 *alias_info == *input_alias_info))) {
0243:             // We've found the original input tensor that aliases with the
0244:             // current output. Wrap it in an IValue and put it directly on the
0245:             // stack.
0246:             (*stack)[returns_begin + idx] = c10::IValue(tensor_args[i]);
0247:             found_alias = true;
0248:             break;
0249:           }
0250:         }
0251:       } else if (
0252:           returns[idx].isTensorList() &&
0253:           validate_tensor_list(returns[idx].toTensorList())) {
0254:         for (const auto i : c10::irange(tensorlist_args_indices.size())) {
0255:           auto input_tensor_idx = tensorlist_args_indices[i];
0256:           const AliasInfo* input_alias_info =
0257:               schema_args[input_tensor_idx].alias_info();
0258:           // Checked above; adding assert to guard against breakage of the below
0259:           // condition due to changing the above if test.
0260:           TORCH_INTERNAL_ASSERT_DEBUG_ONLY(alias_info != nullptr);
0261:           if (validate_tensor_list(tensorlist_args[i]) &&
0262:               (alias_info == input_alias_info ||
0263:                (input_alias_info != nullptr &&
0264:                 *alias_info == *input_alias_info))) {
0265:             // We've found the original input tensor that aliases with the
0266:             // current output. Wrap it in an IValue and put it directly on the
0267:             // stack.
0268:             (*stack)[returns_begin + idx] = c10::IValue(tensorlist_args[i]);
0269:             found_alias = true;
0270:             break;
```
- **EN**: Lines 241-270 mainly cover comments/documentation, state/variable declarations, expressions/calls. Notable symbols: IValue, isTensorList, validate_tensor_list, toTensorList.
- **CN**: 第 241-270 行主要涉及注释或说明、变量/别名声明、表达式或调用。 值得关注的符号包括：IValue, isTensorList, validate_tensor_list, toTensorList。

### Lines 271-300 / 第 271-300 行
```cpp
0271:           }
0272:         }
0273:       }
0274:       TORCH_CHECK(
0275:           found_alias,
0276:           "The operator ",
0277:           op.schema().operator_name(),
0278:           " appears to have invalid alias information. ",
0279:           "Found a return tensor argument with a mismatched mutable alias: ",
0280:           schema_returns[idx]);
0281:     } else {
0282:       if (alias_info != nullptr && !alias_info->isWrite()) {
0283:         // Case (3): immutable alias (view) case.
0284:         // Warn here, since we're copying and not creating a view.
0285:         // If this operator is needed, the backend should provide a kernel for
0286:         // it. See Note [CPU Fallback Does Not Handle View Operators]
0287:         std::stringstream dev_str;
0288:         if (tgt_device) {
0289:           dev_str << *tgt_device;
0290:         } else {
0291:           dev_str << "<none>";
0292:         }
0293:         if (error_on_views) {
0294:           TORCH_CHECK(
0295:               false,
0296:               "The operator ",
0297:               op.schema().operator_name(),
0298:               " appears to be a view operator, ",
0299:               "but it has no implementation for the backend \"",
0300:               dev_str.str(),
```
- **EN**: Lines 271-300 mainly cover expressions/calls, function signatures/definitions, state/variable declarations. Notable symbols: TORCH_CHECK, schema, isWrite, Case.
- **CN**: 第 271-300 行主要涉及表达式或调用、函数签名或实现、变量/别名声明。 值得关注的符号包括：TORCH_CHECK, schema, isWrite, Case。

### Lines 301-330 / 第 301-330 行
```cpp
0301:               "\". View operators don't support ",
0302:               "since the tensor's storage cannot be shared across devices.");
0303:         } else {
0304:           TORCH_WARN(
0305:               false,
0306:               "The operator ",
0307:               op.schema().operator_name(),
0308:               " appears to be a view operator, ",
0309:               "but it has no implementation for the backend \"",
0310:               dev_str.str(),
0311:               "\". View operators don't support falling back to run on the CPU, ",
0312:               "since the tensor's storage cannot be shared across devices.");
0313:         }
0314:       }
0315:       // Case (2): copy case.
0316:       // Copy the cpu output tensor to the original device.
0317: 
0318:       // We technically  might not have a target device, e.g. if you call
0319:       // torch.cat() with an empty list In that case, we shouldn't have any
0320:       // tensors to schlep across devices anyway.
0321:       if (tgt_device) {
0322:         if (returns[idx].isTensor() && returns[idx].toTensor().defined()) {
0323:           (*stack)[returns_begin + idx] =
0324:               c10::IValue(returns[idx].toTensor().to(*tgt_device));
0325:         } else if (
0326:             returns[idx].isTensorList() &&
0327:             validate_tensor_list(returns[idx].toTensorList())) {
0328:           const auto& cpu_tensors = returns[idx].toTensorList().vec();
0329:           std::vector<at::Tensor> tensors;
0330:           tensors.reserve(cpu_tensors.size());
```
- **EN**: Lines 301-330 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: TORCH_WARN, schema, str, Case.
- **CN**: 第 301-330 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：TORCH_WARN, schema, str, Case。

### Lines 331-343 / 第 331-343 行
```cpp
0331: 
0332:           for (const auto& tensor : cpu_tensors) {
0333:             tensors.push_back(tensor.to(*tgt_device));
0334:           }
0335:           (*stack)[returns_begin + idx] =
0336:               c10::IValue(c10::List<at::Tensor>(tensors));
0337:         }
0338:       }
0339:     }
0340:   }
0341: }
0342: 
0343: } // namespace at::native
```
- **EN**: Lines 331-343 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: push_back, to, IValue.
- **CN**: 第 331-343 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：push_back, to, IValue。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现
- **EN**: Dispatch-key aware behavior  
  **CN**: 感知 DispatchKey 的行为控制

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/native/CPUFallback.h>`, `<ATen/core/ivalue.h>`, `<ATen/core/stack.h>`, `<ATen/core/dispatch/Dispatcher.h>`, `<sstream>`, `<vector>`, `<ATen/Functions.h>`, `<ATen/ops/_copy_from_and_resize.h>`, `<ATen/ops/_to_cpu.h>`
- **Macros / 宏**: `TORCH_CHECK`, `TORCH_WARN`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
