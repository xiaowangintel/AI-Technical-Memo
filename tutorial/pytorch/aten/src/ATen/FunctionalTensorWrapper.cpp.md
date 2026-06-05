# FunctionalTensorWrapper.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/FunctionalTensorWrapper.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines shared ATen abstractions, tensor utilities, device helpers, or interop glue used across subsystems. This specific file centers on `FunctionalTensorWrapper.cpp`. Descriptor/handle lifecycle management is important here. Transform-aware tensor semantics used by functorch are part of the file scope. Tensor metadata shaping and storage/layout checks are part of the local logic.
- **Purpose (CN)**: 定义跨子系统复用的共享 ATen 抽象、张量工具、设备辅助逻辑或互操作胶水代码。 该文件具体围绕 `FunctionalTensorWrapper.cpp` 展开。 描述符/句柄的生命周期管理是这里的重要内容。 该文件范围内还包含 functorch 所需的变换感知张量语义。 张量元数据组织以及存储/布局检查也是局部逻辑的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-47 / 第 1-47 行

```cpp
0001: 
0002: #include <ATen/FunctionalTensorWrapper.h>
0003: 
0004: #include <ATen/core/IListRef.h>
0005: #include <ATen/core/LegacyTypeDispatch.h>
0006: #include <c10/util/Exception.h>
0007: 
0008: #include <c10/util/irange.h>
0009: 
0010: #ifndef AT_PER_OPERATOR_HEADERS
0011: #include <ATen/Functions.h>
0012: #else
0013: #include <ATen/ops/_propagate_xla_data.h>
0014: #include <ATen/ops/_to_copy.h>
0015: #endif
0016: 
0017: namespace at {
0018: 
0019: void FunctionalTensorWrapper::set_constructor_metadata() {
0020:   TORCH_INTERNAL_ASSERT(value_.defined());
0021:   // Note: "level" is a concept that we don't know how to compute in core.
0022:   // For now I'm retroactively setting this in functorch,
0023:   // but once Open Multiple Dispatch lands we should be able to calculate this in core.
0024:   level_ = -1;
0025:   // mirror all of the generic tensor metadata onto the wrapper
0026:   copy_generic_tensor_metadata(value_.getIntrusivePtr().get(), this);
0027:   refresh_numel();
0028:   refresh_contiguous();
0029:   storage_access_should_throw_ = false;
0030:   // In general, the sizes/stride metadata on a tensor can change as it is mutated,
0031:   // and these changes need to be reflected in the metadata of the wrapper.
0032:   set_allow_tensor_metadata_change(true);
0033:   key_set_ = c10::DispatchKeySet(c10::DispatchKey::Functionalize) | value_.key_set();
0034:   // All of the keys corresponding to functorch transforms should not be copied over.
0035:   // Functorch transforms all have their own wrapper tensors (e.g. BatchedTensorImpl) which expect
0036:   // to participate in the functorch transforms.
0037:   key_set_ = key_set_ - c10::functorch_transforms_ks - c10::python_ks;
0038:   // We override a bunch of _custom(), so make sure they get called
0039:   // TODO: metadata copying may not actually be necessary then
0040:   set_custom_sizes_strides(SizesStridesPolicy::CustomSizes);
0041:   set_custom_device(true);
0042:   // E.g. when running torch.compile under inference mode, we need to make sure that
0043:   // for any inputs that were created outside of inference mode (so they are not inference tensors),
0044:   // then the functional wrappers that we wrap them with should also not be inference tensors.
0045:   version_counter_ = value_.unsafeGetTensorImpl()->version_counter();
0046: }
0047: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `set_constructor_metadata`, `copy_generic_tensor_metadata`, `refresh_numel`, `refresh_contiguous`, `set_allow_tensor_metadata_change`, `set_custom_sizes_strides`, `set_custom_device`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`set_constructor_metadata`, `copy_generic_tensor_metadata`, `refresh_numel`, `refresh_contiguous`, `set_allow_tensor_metadata_change`, `set_custom_sizes_strides`, `set_custom_device`。

### Lines 48-107 / 第 48-107 行

```cpp
0048: FunctionalTensorWrapper::FunctionalTensorWrapper(const Tensor& value)
0049:   : c10::TensorImpl(
0050:       c10::Storage(c10::make_intrusive<functionalization::FunctionalStorageImpl>(value)),
0051:       c10::DispatchKeySet(DispatchKey::Functionalize) | value.key_set(),
0052:       value.dtype()
0053:     ),
0054:     value_(value)
0055: {
0056:   TORCH_INTERNAL_ASSERT(!at::functionalization::impl::isFunctionalTensor(value_));
0057:   TORCH_INTERNAL_ASSERT(!value_.key_set().has(c10::DispatchKey::Functionalize));
0058:   set_constructor_metadata();
0059: }
0060: 
0061: void FunctionalTensorWrapper::freeze_storage() const {
0062:   functional_storage_impl()->freeze();
0063: }
0064: 
0065: // Note [Functionalization: Alias Removal]
0066: // When someone calls a view() op during the functionalization pass, e.g. 'b = a.view(...)',
0067: // we link `b` and `a` to a shared Alias object to preserve the aliasing relationship.
0068: //
0069: // How do we do that?
0070: //
0071: // Every FunctionalTensorWrapper contains a dummy FunctionalStorageImpl, which subclasses from c10::StorageImpl.
0072: // It doesn't contain any data (similar to MetaTensor storage), but it contains an Alias object that knows about the base tensor.
0073: // When a tensor is created through a view operation, both the new and old tensor point to the same FunctionalStorageImpl.
0074: //
0075: // As mutations are applied to any of the views, we also queue each mutation up on the Alias object, so we can replay them.
0076: // When the user requests a tensor that's had a view taken, we check if it's up to date.
0077: // If it's not up to date, we first replay all of the queued up mutations onto the alias, and then re-apply the current view
0078: // on top of the newly updated alias.
0079: //
0080: // Why do we queue up and lazily run mutations on the alias, instead of updating the alias eagerly?
0081: // This behavior was taken from pytorch/xla, which the alias-removal logic was inspired from.
0082: // One benefit of the laziness is that we save work in the cases where a user has multiple views and mutates one of them,
0083: // but never uses the other views later in the program (in which case we'll never update the alias).
0084: // It also has downsides though: repeatedly applying mutations to the same view without syncing
0085: // will silently use up more and more memory as more mutations are queued up.
0086: //
0087: // Corresponding diagram:
0088: //
0089: // b = a.view(...)
0090: //
0091: //        a                                                    b
0092: //        |                                                    |     If the user asks for b and it’s out of date,
0093: //       \/                                                    \/    We regenerate b by replaying it’s views from the alias.
0094: // . - - - - - - - - - - - - - .                    . - - - - - - - - - - - - - .
0095: // |  FunctionalTensorWrapper  |                    |  FunctionalTensorWrapper  |
0096: // . - - - - - - - - - - - - - .                    . - - - - - - - - - - - - - .
0097: // |     value   |   storage   |                    |    storage    |   Value   |
0098: // . - - - - - - - - - - - - - .                    . - - - - - - - - - - - - - .
0099: //          |                   \                  /                      |
0100: //          |                     \              /                        |
0101: //          |                       . - - - - - - - - - - - - .           |
0102: //          |                       |  FunctionalStorageImpl  |           |
0103: //          |                       . - - - - - - - - - - - - .           |
0104: //          |                       |         Alias           |           |
0105: //          |                       . - - - - - - - - - - - - .           |
0106: //          |                       /     mutations to a or b             |
0107: //          |                     /       are queued onto Alias           |
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; manages memory allocation, buffers, or ownership boundaries; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `FunctionalTensorWrapper`, `set_constructor_metadata`, `freeze_storage`, `functional_storage_impl`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；管理内存分配、缓冲区或所有权边界；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`FunctionalTensorWrapper`, `set_constructor_metadata`, `freeze_storage`, `functional_storage_impl`。

### Lines 108-142 / 第 108-142 行

```cpp
0108: //          |                   /                                         |
0109: //         \/                 /                                           \/
0110: // . - - - - - - - - - - - - - .                             . - - - - - - - - - - - - - - - .
0111: // |        TensorImpl         |                             |             TensorImpl        |
0112: // . - - - - - - - - - - - - - .                             . - - - - - - - - - - - - - - - .
0113: // |   value   |   storage     |                             |    storage    |     Value     |
0114: // . - - - - - - - - - - - - - .                             . - - - - - - - - - - - - - - - .
0115: //          |                                                             |
0116: //          |                                                             |
0117: //          |                                                             |
0118: //          |   In this picture the two tensor views their own storages,  |
0119: //          |   have their own storages, but backends like functorch      |
0120: //         \/   are allowed to re-alias underneath the pass               \/
0121: // . - - - - - - - - - - - - - .                             . - - - - - - - - - - - - - - - .
0122: // |    underlying_storage     |                             |      underlying_storage       |
0123: // . - - - - - - - - - - - - - .                             . - - - - - - - - - - - - - - - .
0124: //
0125: // This constructor is only used by view ops.
0126: // - view_value: The output tensor that we need to wrap.
0127: // - base: The "base" of the view that `view_value` was generated from.
0128: // See Note [Functionalization: Alias Removal Part 2] for more details on the mutation replay logic.
0129: FunctionalTensorWrapper::FunctionalTensorWrapper(
0130:     const Tensor& view_value,
0131:     const FunctionalTensorWrapper* base,
0132:     const std::shared_ptr<functionalization::ViewMeta>& meta)
0133:     : c10::TensorImpl(
0134:           c10::DispatchKeySet(DispatchKey::Functionalize),
0135:           view_value.dtype(),
0136:           base->storage().data_ptr().device()),
0137:       value_(view_value),
0138:       is_multi_output_view_(
0139:           base->is_multi_output_view_ || meta->is_multi_output),
0140:       was_storage_changed_(base->was_storage_changed_),
0141:       is_symbolic_(base->is_symbolic_) {
0142:   TORCH_INTERNAL_ASSERT(!at::functionalization::impl::isFunctionalTensor(value_));
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `FunctionalTensorWrapper`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`FunctionalTensorWrapper`。

### Lines 143-173 / 第 143-173 行

```cpp
0143:   TORCH_INTERNAL_ASSERT(!value_.key_set().has(c10::DispatchKey::Functionalize));
0144:   set_constructor_metadata();
0145:   // Copy the original tensor's ViewMeta vector and push the current one.
0146:   if (!base->view_metas_.empty()) {
0147:       view_metas_ = base->view_metas_;  // copy
0148:   }
0149:   view_metas_.push_back(meta);
0150:   maybe_mark_symbolic(meta.get());
0151:   storage_ = base->storage_; // alias this tensor's storage with the base tensor's
0152: }
0153: 
0154: functionalization::FunctionalStorageImpl* FunctionalTensorWrapper::functional_storage_impl() const {
0155:   return static_cast<functionalization::FunctionalStorageImpl*>(storage_.unsafeGetStorageImpl());
0156: }
0157: 
0158: void FunctionalTensorWrapper::commit_update() {
0159:   auto storage_impl = functional_storage_impl();
0160:   storage_impl->add_update(value_, view_metas_);
0161:   // As an optimization, we used to mark the tensor here as "up-to-date",
0162:   // That way, code like:
0163:   //   x = torch.ones(1'000'000)
0164:   //   x[0].add_(1)
0165:   // doesn't result in an unnecessary materialization of the base.
0166:   // This optimization results in the slice temporarily haven't incorrect
0167:   // stride/storage_offset though, and DCE should handle that optimization anyway.
0168:   // generation_ = storage_impl->generation();
0169: }
0170: 
0171: bool FunctionalTensorWrapper::is_up_to_date() const {
0172:   auto alias_generation = functional_storage_impl()->generation();
0173:   return generation_ == alias_generation;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `set_constructor_metadata`, `maybe_mark_symbolic`, `functional_storage_impl`, `commit_update`, `is_up_to_date`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`set_constructor_metadata`, `maybe_mark_symbolic`, `functional_storage_impl`, `commit_update`, `is_up_to_date`。

### Lines 174-219 / 第 174-219 行

```cpp
0174: }
0175: 
0176: // See Note [Functionalization Pass - Inplace View Ops]
0177: void FunctionalTensorWrapper::mutate_view_meta(const std::shared_ptr<at::functionalization::ViewMeta>& meta) {
0178:   view_metas_.push_back(meta);
0179:   // Manually track the fact that this tensor received a metadata mutation!
0180:   has_metadata_mutation_ = true;
0181:   // Mark this tensor as being symbolic if there are any symbolic inputs used by the view operation.
0182:   maybe_mark_symbolic(meta.get());
0183:   // Note [Functionalization Pass - Inplace View Ops]
0184:   // So, these ops are special - they're mutation AND view ops. They get special codegen.
0185:   // An example is transpose_, e.g. `a.transpose_()`
0186:   // Calling transpose_() should ensure that a gets an alias, and append the new ViewMeta to a's current list of ViewMetas.
0187:   at::AutoDispatchSkipFunctionalize guard;
0188:   value_ = meta->forward(value_);
0189:   TORCH_INTERNAL_ASSERT(!value_.key_set().has(c10::DispatchKey::Functionalize));
0190: }
0191: 
0192: // Note [Functionalization: Mutation Removal]
0193: // Mutation removal is used to take a program like this:
0194: //
0195: // a.add_(b)
0196: //
0197: // and replace it with a slightly different program that has the same semantics:
0198: //
0199: // tmp = a.add(b)
0200: // a.replace_(tmp)
0201: //
0202: // Where the replace_() call is implemented directly in the functionalization pass, so it is transparent to the backend.
0203: // This is useful for backends that aren't able to handle certain types of mutations, like functorch.
0204: //
0205: // Why do we need to wrap every tensor in a FunctionalTensorWrapper? Consider this program:
0206: //
0207: // Before:
0208: // tensor.add_(batched_tensor)
0209: //
0210: // After:
0211: // tmp = tensor.add(batched_tensor)
0212: // tensor.replace_(tmp)
0213: //
0214: // In the above, tmp is a batched tensor (because adding a normal tensor to a batched tensor does broadcasting and creates a batched tensor).
0215: // But we can't just replace the underlying memory backing `tensor` with `tmp` - a batched tensor takes up more space!
0216: // Instead, every input, intermediate and output of the program is wrapped in a FunctionalTensorImpl, which wraps the underlying tensor.
0217: void FunctionalTensorWrapper::replace_(const Tensor& other, bool from_lazy_regenerate) {
0218:   // TODO: going to need to change this if we want nested functionalize() transforms.
0219:   TORCH_INTERNAL_ASSERT(!at::functionalization::impl::isFunctionalTensor(other));
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `mutate_view_meta`, `maybe_mark_symbolic`, `replace_`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`mutate_view_meta`, `maybe_mark_symbolic`, `replace_`。

### Lines 220-251 / 第 220-251 行

```cpp
0220:   value_ = other;
0221:   TORCH_INTERNAL_ASSERT(!value_.key_set().has(c10::DispatchKey::Functionalize));
0222:   // out= ops are allowed to resize the output tensors, mutating both the data and metadata of the tensor.
0223:   // We need to propagate that metadata mutation to the wrapper (new size).
0224:   auto sizes_ = value_.sym_sizes();
0225:   auto strides_ = value_.sym_strides();
0226:   auto storage_offset_ = value_.sym_storage_offset();
0227:   set_sizes_and_strides(sizes_, strides_, storage_offset_);
0228:   if (dtype() != value_.unsafeGetTensorImpl()->dtype() || layout() != value_.unsafeGetTensorImpl()->layout()) {
0229:     // .to() should not re-entrantly go through functionalization.
0230:     at::AutoDispatchSkipFunctionalize guard;
0231:     // and we want _to_copy() to show up in the graph, not the composite .to() operator
0232:     // (this can happen if autograd has already run by the time we enter this code)
0233:     value_ = at::_to_copy(value_, c10::TensorOptions().dtype(dtype()).layout(layout()));
0234:     TORCH_INTERNAL_ASSERT(!value_.key_set().has(c10::DispatchKey::Functionalize));
0235:   }
0236:   // might not be until after the no_grad region is exited.
0237:   // Therefore, replace_() is not unconditionally safe to check the current no_grad state.
0238:   // If this is a lazy regeneration, then it is guaranteed that we have already
0239:   // done the mutation for the storage alias (when we originally performed the mutation),
0240:   // so no counter update may be needed.
0241:   // Example: if a mutation happens to a view under a no_grad,
0242:   // we won't call replace_() on the other alias until the alias is later used, which
0243:   if (!from_lazy_regenerate) {
0244:     mark_mutation();
0245:     if (!at::GradMode::is_enabled() || InferenceMode::is_enabled()) {
0246:       // This mutation happened under no_grad or inference_mode
0247:       mark_mutation_during_no_grad_or_inference_mode();
0248:     }
0249:   }
0250: }
0251: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `set_sizes_and_strides`, `mark_mutation`, `mark_mutation_during_no_grad_or_inference_mode`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`set_sizes_and_strides`, `mark_mutation`, `mark_mutation_during_no_grad_or_inference_mode`。

### Lines 252-281 / 第 252-281 行

```cpp
0252: bool FunctionalTensorWrapper::has_data_mutation() {
0253:   // Current tensor's data was mutated if its storage saw any mutations.
0254:   return functional_storage_impl()->generation() > 0;
0255: }
0256: 
0257: void FunctionalTensorWrapper::set__impl(const FunctionalTensorWrapper* other) {
0258:   // self.set_(src) will cause self to have all of the tensor properties of self.
0259:   value_ = other->value_;
0260:   generation_ = other->generation_;
0261:   view_metas_ = other->view_metas_;
0262:   is_symbolic_ = other->is_symbolic_;
0263:   // FREEZE the old storage, preventing mutations to it.
0264:   // this is a huge pain to handle properly in all cases, so we ban it.
0265:   functional_storage_impl()->freeze();
0266:   // Unsafely swap out the storage with other's storage,
0267:   // disconnecting `self` with its view chain
0268:   storage_ = other->storage_;
0269:   /// explicitly mark the tensor as having its storage changed from set_()
0270:   // Otherwise, we don't actually have a 100% accurate way to check this.
0271:   // (We could check if the updated value has a new storage than the original value,
0272:   // but this won't also let us uniquely determine if the tensor **also**
0273:   // experienced a data mutation).
0274:   mark_storage_changed();
0275: 
0276:   auto sizes_ = value_.sym_sizes();
0277:   auto strides_ = value_.sym_strides();
0278:   auto storage_offset_ = value_.sym_storage_offset();
0279:   set_sizes_and_strides(sizes_, strides_, storage_offset_);
0280: }
0281: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; manages memory allocation, buffers, or ownership boundaries. Key symbols: `has_data_mutation`, `set__impl`, `functional_storage_impl`, `mark_storage_changed`, `set_sizes_and_strides`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；管理内存分配、缓冲区或所有权边界。关键符号：`has_data_mutation`, `set__impl`, `functional_storage_impl`, `mark_storage_changed`, `set_sizes_and_strides`。

### Lines 282-333 / 第 282-333 行

```cpp
0282: void FunctionalTensorWrapper::storage_resize_(const c10::SymInt& new_size) {
0283:   auto curr_storage_size = value_.unsafeGetTensorImpl()->unsafe_storage().unsafeGetStorageImpl()->sym_nbytes();
0284:   // storage resizing is severely limited: we only support resizing either to zero, or from zero bytes.
0285:   TORCH_CHECK(new_size == 0 || curr_storage_size == 0, "new_size: ", new_size, ". curr_storage_size: ", curr_storage_size);
0286:   // The "functionalization rule" for storage resizing is a giant no-op, mainly because we don't want
0287:   // resize_() calls to actually emit any ops in the functional graph.
0288:   // How does it work?
0289:   // Resizing up (old size == 0):
0290:   //   We do nothing in this case.
0291:   //   The expectation is that for the user code to be valid, the next op that should run against the current tensor "x"
0292:   //   will be a x.copy_(y) (or similar), that will fully overwrite the data of x.
0293:   //   If there are any outstanding aliases of x, we expect them not to be used until after the copy_() call
0294:   //   (otherwise the eager code would be invalid),
0295:   //   and therefore functionalization will regenerate the aliases off of the result of `x.copy(y)`.
0296:   // Resizing down (new size == 0):
0297:   //   We also do nothing in this case. The assumption is that after resizing a tensor down,
0298:   //   it is fully unused in the program (unless it is later resized back up first, has data copied in)
0299:   //   Although it might be saved for backward, which happens in FSDP.
0300:   //   The expected pattern is that the param will then be resized back up from zero in the backward.
0301: 
0302:   // Mark the tensor as having its storage resized.
0303:   // This is so we can detect it for inputs in AOTAutograd and error / emit
0304:   // an input mutation resize_() appropriately
0305:   functional_storage_impl()->mark_inductor_storage_resize(new_size);
0306: }
0307: 
0308: void FunctionalTensorWrapper::maybe_replace_storage(const Tensor& other) {
0309:   // Note [resize_() in functionalization pass]
0310:   // resize_() is a special operator in functionalization because it can reallocate its underlying storage.
0311:   // This function is only ever called in the case that resize_() needs to reallocate its storage to a larger size.
0312:   //
0313:   // However, functionalization currently bans the following code:
0314:   //   a = torch.ones(2)
0315:   //   b = a.view(2)
0316:   //   b.resize_(4) # b is a view tensor, that we are trying to increase the storage size of
0317:   //
0318:   // Why is this code difficult to handle?
0319:   // The functionalization pass currently keeps aliases in sync by making the following assumptions:
0320:   // - The “base” tensor always refers to “all of the data”
0321:   // - Whenever you have b = view_op(a), “b” should always refer to a subset of “a”s memory.
0322:   //
0323:   // The code above breaks that assumption b.resize_(4) actually needs to update "a"
0324:   // to tell it that it is now actually some slice of a pre-existing larger storage.
0325:   // We're also no longer re-generate "b" fully from "a" anymore, since "a" refers to a slice of "b"'s data.
0326:   //
0327:   // This is probably fixable in theory, but:
0328:   // - the fix would likely complicated the functionalization logic quite a bit.
0329:   // - the primary use case for resize_() today is resizing zero-sized tensors in out= variants of operators
0330:   // - resize_() also can give you weird results today if you try to resize_() a weirdly strided tensor.
0331:   //
0332:   // Given all of the above, for now we're just banning the above usage.
0333:   TORCH_CHECK(storage().use_count() == 1, "Attempted to resize a view tensor to a larger size. This is not allowed in the functionalization pass");
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `storage_resize_`, `functional_storage_impl`, `maybe_replace_storage`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`storage_resize_`, `functional_storage_impl`, `maybe_replace_storage`。

### Lines 334-368 / 第 334-368 行

```cpp
0334:   TORCH_CHECK(view_metas_.empty(), "Attempted to resize a view tensor to a larger size. This is not allowed in the functionalization pass");
0335:   // If this tensor is not a view (and has no outstanding views taken out on it),
0336:   // Then it's safe to throw out the old storage and replace it with the new, larger one.
0337:   storage_ = c10::Storage(c10::make_intrusive<functionalization::FunctionalStorageImpl>(other));
0338:   value_ = other;
0339:   TORCH_INTERNAL_ASSERT(!value_.key_set().has(c10::DispatchKey::Functionalize));
0340:   generation_ = 0;
0341:   // And update the metadata on the wrapper to reflect the new sizes and strides
0342:   set_sizes_and_strides(value_.sizes(), value_.strides());
0343:   refresh_numel();
0344:   // (Technically we should be guaranteed that the tensor was already contiguous,
0345:   // since it's guaranteed not to have been a view. Doesn't hurt to run though)
0346:   refresh_contiguous();
0347:   // Swapping out the storage of a tensor (aka from a resize_() call) will update the sizes and strides of the tensor,
0348:   // so we need to record the fact that metadata was mutated.
0349:   has_metadata_mutation_ = true;
0350: }
0351: 
0352: void FunctionalTensorWrapper::_unsafe_reset_storage() {
0353:   // Reset the storage with the current value_ tensor as the base
0354:   storage_ = c10::Storage(c10::make_intrusive<functionalization::FunctionalStorageImpl>(value_));
0355:   // Reset the generation so that it matches the new storage
0356:   generation_ = 0;
0357:   // Clear any pre-existing view metas so that base and value_ are semantically the same
0358:   view_metas_.clear();
0359: }
0360: 
0361: void FunctionalTensorWrapper::sync_() {
0362:   if (is_up_to_date()) {
0363:     return;
0364:   }
0365:   apply_updates();
0366:   regenerate_from_base();
0367: }
0368: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `set_sizes_and_strides`, `refresh_numel`, `refresh_contiguous`, `_unsafe_reset_storage`, `sync_`, `apply_updates`, `regenerate_from_base`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`set_sizes_and_strides`, `refresh_numel`, `refresh_contiguous`, `_unsafe_reset_storage`, `sync_`, `apply_updates`, `regenerate_from_base`。

### Lines 369-406 / 第 369-406 行

```cpp
0369: const std::vector<std::shared_ptr<functionalization::ViewMeta>>& FunctionalTensorWrapper::view_metas() const {
0370:   return view_metas_;
0371: }
0372: 
0373: void FunctionalTensorWrapper::regenerate_from_base() {
0374:   at::AutoDispatchSkipFunctionalize guard;
0375:   auto storage_impl = functional_storage_impl();
0376:   auto t = storage_impl->base();
0377: 
0378:   TORCH_INTERNAL_ASSERT(!at::functionalization::impl::isFunctionalTensor(t));
0379:   t = at::functionalization::impl::apply_view_meta_sequence(t, view_metas_);
0380:   TORCH_INTERNAL_ASSERT(!at::functionalization::impl::isFunctionalTensor(t));
0381: 
0382:   replace_(t, /*from_lazy_regenerate=*/true);
0383:   generation_ = storage_impl->generation();
0384: }
0385: 
0386: bool FunctionalTensorWrapper::apply_updates() {
0387:   // Apply all updates on alias_
0388:   auto storage_impl = functional_storage_impl();
0389:   return storage_impl->apply_updates();
0390: }
0391: 
0392: const char* FunctionalTensorWrapper::tensorimpl_type_name() const {
0393:     return "FunctionalTensorWrapper";
0394: }
0395: 
0396: void FunctionalTensorWrapper::copy_tensor_metadata(
0397:     const FunctionalTensorWrapper* src_impl,
0398:     FunctionalTensorWrapper* dest_impl,
0399:     const c10::VariableVersion& version_counter,
0400:     bool allow_tensor_metadata_change) {
0401:     TensorImpl::copy_tensor_metadata(
0402:         src_impl,
0403:         dest_impl,
0404:         version_counter,
0405:         allow_tensor_metadata_change);
0406: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `view_metas`, `regenerate_from_base`, `replace_`, `apply_updates`, `tensorimpl_type_name`, `copy_tensor_metadata`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`view_metas`, `regenerate_from_base`, `replace_`, `apply_updates`, `tensorimpl_type_name`, `copy_tensor_metadata`。

### Lines 407-436 / 第 407-436 行

```cpp
0407:     // FunctionalTensorWrapper-specific fields.
0408:     dest_impl->value_ = src_impl->value_;
0409:     dest_impl->level_ = src_impl->level_;
0410:     dest_impl->has_metadata_mutation_ = src_impl->has_metadata_mutation_;
0411:     dest_impl->is_multi_output_view_ = src_impl->is_multi_output_view_;
0412:     dest_impl->was_storage_changed_ = src_impl->was_storage_changed_;
0413:     dest_impl->is_symbolic_ = src_impl->is_symbolic_;
0414:     dest_impl->generation_ = src_impl->generation_;
0415:     dest_impl->view_metas_ = src_impl->view_metas_;
0416: }
0417: 
0418: 
0419: void FunctionalTensorWrapper::copy_tensor_metadata_and_refresh(
0420:     const FunctionalTensorWrapper* src_impl,
0421:     FunctionalTensorWrapper* dest_impl,
0422:     const c10::VariableVersion& version_counter,
0423:     bool allow_tensor_metadata_change) const {
0424:     copy_tensor_metadata(src_impl, dest_impl, version_counter, allow_tensor_metadata_change);
0425:     dest_impl->refresh_numel();
0426:     dest_impl->refresh_contiguous();
0427: }
0428: 
0429: template <typename VariableVersion>
0430: c10::intrusive_ptr<TensorImpl> FunctionalTensorWrapper::shallow_copy_and_detach_core(
0431:     VariableVersion&& version_counter,
0432:     bool allow_tensor_metadata_change) const {
0433:   if (key_set_.has(DispatchKey::Python) &&
0434:       !c10::impl::tls_is_dispatch_key_excluded(DispatchKey::Python)) {
0435:     auto r = pyobj_slot_.load_pyobj_interpreter()->detach(this);
0436:     if (r) {
```

- **EN:** Builds a reusable template/helper layer around `FunctionalTensorWrapper`. Key symbols: `copy_tensor_metadata_and_refresh`, `copy_tensor_metadata`, `shallow_copy_and_detach_core`.
- **CN:** 围绕 `FunctionalTensorWrapper` 构建可复用的模板或辅助层。关键符号：`copy_tensor_metadata_and_refresh`, `copy_tensor_metadata`, `shallow_copy_and_detach_core`。

### Lines 437-476 / 第 437-476 行

```cpp
0437:       r->set_version_counter(std::forward<VariableVersion>(version_counter));
0438:       r->set_allow_tensor_metadata_change(allow_tensor_metadata_change);
0439:       return r;
0440:     }
0441:   }
0442: 
0443:   auto impl = c10::make_intrusive<FunctionalTensorWrapper>(value_);
0444:   copy_tensor_metadata_and_refresh(
0445:       /*src_impl=*/this,
0446:       /*dest_impl=*/impl.get(),
0447:       /*version_counter=*/std::forward<VariableVersion>(version_counter),
0448:       /*allow_tensor_metadata_change=*/allow_tensor_metadata_change);
0449:   return impl;
0450: }
0451: 
0452: c10::intrusive_ptr<TensorImpl> FunctionalTensorWrapper::shallow_copy_and_detach(
0453:     const c10::VariableVersion& version_counter,
0454:     bool allow_tensor_metadata_change) const {
0455:   return shallow_copy_and_detach_core(
0456:       version_counter, allow_tensor_metadata_change);
0457: }
0458: 
0459: c10::intrusive_ptr<TensorImpl> FunctionalTensorWrapper::shallow_copy_and_detach(
0460:     c10::VariableVersion&& version_counter,
0461:     bool allow_tensor_metadata_change) const {
0462:   return shallow_copy_and_detach_core(
0463:       std::move(version_counter), allow_tensor_metadata_change);
0464: }
0465: 
0466: void FunctionalTensorWrapper::shallow_copy_from(const c10::intrusive_ptr<TensorImpl>& impl) {
0467:     AT_ASSERT(has_compatible_shallow_copy_type(impl->key_set()));
0468:     auto functional_impl =
0469:         static_cast<FunctionalTensorWrapper*>(impl.get());
0470:     copy_tensor_metadata_and_refresh(
0471:         /*src_impl=*/functional_impl,
0472:         /*dest_impl=*/this,
0473:         /*version_counter=*/version_counter(),
0474:         /*allow_tensor_metadata_change=*/allow_tensor_metadata_change());
0475: }
0476: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `copy_tensor_metadata_and_refresh`, `shallow_copy_and_detach`, `shallow_copy_and_detach_core`, `shallow_copy_from`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`copy_tensor_metadata_and_refresh`, `shallow_copy_and_detach`, `shallow_copy_and_detach_core`, `shallow_copy_from`。

### Lines 477-506 / 第 477-506 行

```cpp
0477: 
0478: c10::Device FunctionalTensorWrapper::device_custom() const {
0479:   // The storage pointer already uses the underlying tensor custom device (if
0480:   // applicable) to extract the device. So, we dont have to recurse again by
0481:   // doing value_.unsafeGetTensorImpl()->device().
0482:   return storage().data_ptr().device();
0483: }
0484: at::IntArrayRef FunctionalTensorWrapper::sizes_custom() const {
0485:   return value_.unsafeGetTensorImpl()->sizes();
0486: }
0487: at::IntArrayRef FunctionalTensorWrapper::strides_custom() const {
0488:   return value_.unsafeGetTensorImpl()->strides();
0489: }
0490: int64_t FunctionalTensorWrapper::dim_custom() const {
0491:   return value_.unsafeGetTensorImpl()->dim();
0492: }
0493: int64_t FunctionalTensorWrapper::numel_custom() const {
0494:   return value_.unsafeGetTensorImpl()->numel();
0495: }
0496: c10::SymBool FunctionalTensorWrapper::sym_is_contiguous_custom(at::MemoryFormat memory_format) const {
0497:   return value_.unsafeGetTensorImpl()->sym_is_contiguous(memory_format);
0498: }
0499: c10::SymIntArrayRef FunctionalTensorWrapper::sym_sizes_custom() const {
0500:   return value_.unsafeGetTensorImpl()->sym_sizes();
0501: }
0502: c10::SymIntArrayRef FunctionalTensorWrapper::sym_strides_custom() const {
0503:   return value_.unsafeGetTensorImpl()->sym_strides();
0504: }
0505: c10::SymInt FunctionalTensorWrapper::sym_size_custom(int64_t d) const {
0506:   return value_.unsafeGetTensorImpl()->sym_size(d);
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `device_custom`, `storage`, `sizes_custom`, `strides_custom`, `dim_custom`, `numel_custom`, `sym_is_contiguous_custom`, `sym_sizes_custom`, `...`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`device_custom`, `storage`, `sizes_custom`, `strides_custom`, `dim_custom`, `numel_custom`, `sym_is_contiguous_custom`, `sym_sizes_custom`, `...`。

### Lines 507-538 / 第 507-538 行

```cpp
0507: }
0508: c10::SymInt FunctionalTensorWrapper::sym_storage_offset_custom() const {
0509:   return value_.unsafeGetTensorImpl()->sym_storage_offset();
0510: }
0511: c10::Layout FunctionalTensorWrapper::layout_impl() const {
0512:   return value_.unsafeGetTensorImpl()->layout();
0513: }
0514: 
0515: namespace functionalization {
0516: namespace impl {
0517: 
0518: Tensor to_functional_tensor(const Tensor& tensor) {
0519:   // Note [Wrapped Numbers <> Functionalization]
0520:   if (tensor.unsafeGetTensorImpl()->is_wrapped_number()) {
0521:       return tensor;
0522:   }
0523:   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(!isFunctionalTensor(tensor));
0524:   return at::detail::make_tensor<FunctionalTensorWrapper>(tensor);
0525: }
0526: std::optional<Tensor> to_functional_tensor(const std::optional<Tensor>& tensor) {
0527:   if (tensor.has_value()) {
0528:     return to_functional_tensor(*tensor);
0529:   }
0530:   return std::nullopt;
0531: }
0532: c10::List<::std::optional<Tensor>> to_functional_tensor(const c10::List<::std::optional<Tensor>>& t_list) {
0533:   c10::List<::std::optional<Tensor>> outputs;
0534:   outputs.reserve(t_list.size());
0535:   for (const auto i : c10::irange(t_list.size())) {
0536:     outputs.push_back(to_functional_tensor(t_list[i]));
0537:   }
0538:   return outputs;
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `sym_storage_offset_custom`, `layout_impl`, `to_functional_tensor`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`sym_storage_offset_custom`, `layout_impl`, `to_functional_tensor`。

### Lines 539-568 / 第 539-568 行

```cpp
0539: }
0540: std::vector<Tensor> to_functional_tensor(ITensorListRef t_list) {
0541:   std::vector<Tensor> outputs;
0542:   outputs.reserve(t_list.size());
0543:   for (const auto& tensor : t_list) {
0544:     outputs.push_back(to_functional_tensor(tensor));
0545:   }
0546:   return outputs;
0547: }
0548: 
0549: Tensor from_functional_tensor(const Tensor& tensor, bool assert_functional) {
0550:   // Note [Wrapped Numbers <> Functionalization]
0551:   if (!tensor.defined() || tensor.unsafeGetTensorImpl()->is_wrapped_number()) {
0552:       return tensor;
0553:   }
0554:   if (isFunctionalTensor(tensor)) {
0555:     auto impl = unsafeGetFunctionalWrapper(tensor);
0556:     return impl->value();
0557:   } else {
0558:     // If the current tensor is not functional, then raise an error
0559:     // if assert_functional is true. Otherwise, return the input.
0560:     TORCH_INTERNAL_ASSERT(!assert_functional)
0561:     return tensor;
0562:   }
0563: }
0564: std::optional<Tensor> from_functional_tensor(const std::optional<Tensor>& t, bool assert_functional) {
0565:   if (t.has_value()) {
0566:     return from_functional_tensor(*t, assert_functional);
0567:   }
0568:   return std::nullopt;
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `to_functional_tensor`, `from_functional_tensor`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`to_functional_tensor`, `from_functional_tensor`。

### Lines 569-604 / 第 569-604 行

```cpp
0569: }
0570: std::vector<Tensor> from_functional_tensor(ITensorListRef t_list) {
0571:   std::vector<Tensor> outputs;
0572:   outputs.reserve(t_list.size());
0573:   for (const auto& tensor : t_list) {
0574:     // from_functional_tensor(Tensor) has asserts to make sure you don't accidentally call
0575:     // it on a non-functional input,
0576:     // but from_functional_tensor(TensorList) can receive a list containing both
0577:     // functional and non-functional tensors.
0578:     // Example of when that can happen: torch.cat(function_input_tensor, global_state_tensor).
0579:     // When that happens, we're okay with only unwrapping the functional tensors.
0580:     outputs.push_back(from_functional_tensor(tensor, /*assert_functional=*/false));
0581:   }
0582:   return outputs;
0583: }
0584: c10::List<::std::optional<Tensor>> from_functional_tensor(const c10::List<::std::optional<Tensor>>& t_list) {
0585:   c10::List<::std::optional<Tensor>> outputs;
0586:   outputs.reserve(t_list.size());
0587:   for (const auto i : c10::irange(t_list.size())) {
0588:     outputs.push_back(from_functional_tensor(t_list[i], /*assert_functional=*/false));
0589:   }
0590:   return outputs;
0591: }
0592: 
0593: void sync(const Tensor& t) {
0594:   if (t.unsafeGetTensorImpl()->is_wrapped_number()) {
0595:     // Note [Wrapped Numbers <> Functionalization]
0596:     // Unfortunately, we can't easily guarantee that wrapped numbers (scalar-tensors)
0597:     // get wrapped up in a FunctionalTensorWrapper object, since they skip the dispatcher.
0598:     // That shouldn't matter, since I don't think we're allowed to assign to wrapped numbers anyway.
0599:     return;
0600:   }
0601:   // Not every tensor that hits a functionalization kernel is necessarily a functional tensor.
0602:   // For example, xla_tensor.copy_(cpu_tensor) needs to hit the functionalization kernel
0603:   // to sync xla_tensor, but not cpu_tensor.
0604:   if (!at::functionalization::impl::isFunctionalTensor(t)) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `from_functional_tensor`, `sync`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`from_functional_tensor`, `sync`。

### Lines 605-635 / 第 605-635 行

```cpp
0605:     return;
0606:   }
0607:   auto functional_impl = at::functionalization::impl::unsafeGetFunctionalWrapper(t);
0608:   functional_impl->sync_();
0609: }
0610: void sync(const std::optional<Tensor>& t) {
0611:   if (t.has_value()) {
0612:     sync(*t);
0613:   }
0614: }
0615: void sync(ITensorListRef t_list) {
0616:   for (const auto& t : t_list) {
0617:     sync(t);
0618:   }
0619: }
0620: void sync(const c10::List<::std::optional<Tensor>>& t_list) {
0621:   for (const auto i : c10::irange(t_list.size())) {
0622:     sync(t_list[i]);
0623:   }
0624: }
0625: 
0626: void replace_(const Tensor& functional_tensor, const Tensor& other) {
0627:   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(isFunctionalTensor(functional_tensor));
0628:   unsafeGetFunctionalWrapper(functional_tensor)->replace_(other);
0629: }
0630: 
0631: void replace_(const ITensorListRef functional_tensor, ITensorListRef other) {
0632:   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(functional_tensor.size() == other.size());
0633:   auto functional_tensor_it = functional_tensor.begin();
0634:   auto other_it = other.begin();
0635:   for ([[maybe_unused]] const auto i : c10::irange(functional_tensor.size())) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values. Key symbols: `sync`, `replace_`, `unsafeGetFunctionalWrapper`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值。关键符号：`sync`, `replace_`, `unsafeGetFunctionalWrapper`。

### Lines 636-667 / 第 636-667 行

```cpp
0636:     replace_(*functional_tensor_it++, *other_it++);
0637:   }
0638: }
0639: 
0640: void propagate_xla_data(const Tensor& functional_tensor, const Tensor& other) {
0641:   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(isFunctionalTensor(functional_tensor));
0642:   if (functional_tensor.key_set().has(c10::DispatchKey::XLA)) {
0643:     at::_propagate_xla_data(at::functionalization::impl::unsafeGetFunctionalWrapper(functional_tensor)
0644:         ->value(), other);
0645:   }
0646: }
0647: 
0648: void propagate_xla_data(const ITensorListRef functional_tensor, ITensorListRef other) {
0649:   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(functional_tensor.size() == other.size());
0650:   auto functional_tensor_it = functional_tensor.begin();
0651:   auto other_it = other.begin();
0652:   for ([[maybe_unused]] const auto i : c10::irange(functional_tensor.size())) {
0653:     propagate_xla_data(*functional_tensor_it++, *other_it++);
0654:   }
0655: }
0656: 
0657: void propagate_xla_data_direct(const Tensor& tensor, const Tensor& other) {
0658:   if (tensor.key_set().has(c10::DispatchKey::XLA)) {
0659:     at::_propagate_xla_data(tensor, other);
0660:   }
0661:  }
0662: 
0663: void propagate_xla_data_direct(const ITensorListRef tensor,
0664:                                ITensorListRef other) {
0665:   auto tensor_it = tensor.begin();
0666:   auto other_it = other.begin();
0667:   for ([[maybe_unused]] const auto i : c10::irange(tensor.size())) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values. Key symbols: `replace_`, `propagate_xla_data`, `_propagate_xla_data`, `propagate_xla_data_direct`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值。关键符号：`replace_`, `propagate_xla_data`, `_propagate_xla_data`, `propagate_xla_data_direct`。

### Lines 668-697 / 第 668-697 行

```cpp
0668:     propagate_xla_data_direct(*tensor_it++, *other_it++);
0669:   }
0670: }
0671: 
0672: void commit_update(const Tensor& functional_tensor) {
0673:   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(isFunctionalTensor(functional_tensor));
0674:   unsafeGetFunctionalWrapper(functional_tensor)->commit_update();
0675: }
0676: 
0677: void commit_update(ITensorListRef functional_tensor) {
0678:   for (const auto& t : functional_tensor) {
0679:     commit_update(t);
0680:   }
0681: }
0682: 
0683: void unsafe_reset_storage(const Tensor& functional_tensor) {
0684:   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(isFunctionalTensor(functional_tensor));
0685:   unsafeGetFunctionalWrapper(functional_tensor)->_unsafe_reset_storage();
0686: }
0687: 
0688: void mark_mutation_hidden_from_autograd(const Tensor& functional_tensor) {
0689:   TORCH_CHECK(isFunctionalTensor(functional_tensor));
0690:   unsafeGetFunctionalWrapper(functional_tensor)->mark_mutation_hidden_from_autograd();
0691: }
0692: 
0693: bool are_all_mutations_hidden_from_autograd(const Tensor& functional_tensor) {
0694:   TORCH_CHECK(isFunctionalTensor(functional_tensor));
0695:   return unsafeGetFunctionalWrapper(functional_tensor)->are_all_mutations_hidden_from_autograd();
0696: }
0697: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `propagate_xla_data_direct`, `commit_update`, `unsafeGetFunctionalWrapper`, `unsafe_reset_storage`, `mark_mutation_hidden_from_autograd`, `are_all_mutations_hidden_from_autograd`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`propagate_xla_data_direct`, `commit_update`, `unsafeGetFunctionalWrapper`, `unsafe_reset_storage`, `mark_mutation_hidden_from_autograd`, `are_all_mutations_hidden_from_autograd`。

### Lines 698-730 / 第 698-730 行

```cpp
0698: bool are_all_mutations_under_no_grad_or_inference_mode(const Tensor& functional_tensor) {
0699:   TORCH_CHECK(isFunctionalTensor(functional_tensor));
0700:   return unsafeGetFunctionalWrapper(functional_tensor)->are_all_mutations_under_no_grad_or_inference_mode();
0701: }
0702: 
0703: bool isFunctionalTensor(const at::Tensor& tensor) {
0704:    return tensor.unsafeGetTensorImpl()->key_set().has(c10::DispatchKey::Functionalize);
0705: }
0706: 
0707: bool isBaseTensor(const at::Tensor& tensor) {
0708:   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(isFunctionalTensor(tensor));
0709:   return unsafeGetFunctionalWrapper(tensor)->isBaseTensor();
0710: }
0711: 
0712: bool isFunctionalTensor(const std::optional<Tensor>& t) {
0713:   if (t.has_value()) {
0714:     return isFunctionalTensor(*t);
0715:   } else {
0716:     return false;
0717:   }
0718: }
0719: 
0720: bool isFunctionalTensor(const c10::List<::std::optional<Tensor>>& t_list) {
0721:   if (t_list.empty()) { return false; }
0722:   auto functional_count = 0;
0723:   for (const auto i : c10::irange(t_list.size())) {
0724:     auto const & e= t_list[i];
0725:     if (!e.has_value() || !e->defined()) { continue; }
0726:     if (isFunctionalTensor(e)) {
0727:       ++functional_count;
0728:     }
0729:   }
0730:   return functional_count > 0;
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `are_all_mutations_under_no_grad_or_inference_mode`, `unsafeGetFunctionalWrapper`, `isFunctionalTensor`, `isBaseTensor`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`are_all_mutations_under_no_grad_or_inference_mode`, `unsafeGetFunctionalWrapper`, `isFunctionalTensor`, `isBaseTensor`。

### Lines 731-761 / 第 731-761 行

```cpp
0731: }
0732: 
0733: template <typename T>
0734: static bool isFunctionalTensorIListRef(c10::IListRef<T> list) {
0735:   if (list.size() == 0) { return false; }
0736:   auto functional_count = 0;
0737:   for (const auto& tensor : list) {
0738:     if (!tensor.defined()) { continue; }
0739:     if (isFunctionalTensor(tensor)) {
0740:       ++functional_count;
0741:     }
0742:   }
0743:   return functional_count > 0;
0744: }
0745: 
0746: bool isFunctionalTensor(ITensorListRef list) {
0747:   return isFunctionalTensorIListRef(list);
0748: }
0749: 
0750: void freeze_functional_tensor(const Tensor& tensor) {
0751:   TORCH_INTERNAL_ASSERT(at::functionalization::impl::isFunctionalTensor(tensor));
0752:   auto functional_base_impl = at::functionalization::impl::unsafeGetFunctionalWrapper(tensor);
0753:   functional_base_impl->freeze_storage();
0754: }
0755: 
0756: Tensor create_functional_tensor_with_view_meta(
0757:     const at::Tensor& view_to_wrap,
0758:     const at::Tensor& base,
0759:     const std::shared_ptr<functionalization::ViewMeta>& meta,
0760:     int64_t out_idx) {
0761:   TORCH_INTERNAL_ASSERT(!at::functionalization::impl::isFunctionalTensor(view_to_wrap));
```

- **EN:** Builds a reusable template/helper layer around `FunctionalTensorWrapper`. Key symbols: `isFunctionalTensorIListRef`, `isFunctionalTensor`, `freeze_functional_tensor`, `create_functional_tensor_with_view_meta`.
- **CN:** 围绕 `FunctionalTensorWrapper` 构建可复用的模板或辅助层。关键符号：`isFunctionalTensorIListRef`, `isFunctionalTensor`, `freeze_functional_tensor`, `create_functional_tensor_with_view_meta`。

### Lines 762-792 / 第 762-792 行

```cpp
0762:   TORCH_INTERNAL_ASSERT(at::functionalization::impl::isFunctionalTensor(base));
0763:   auto functional_base_impl = at::functionalization::impl::unsafeGetFunctionalWrapper(base);
0764:   auto meta_ = meta;
0765:   if (out_idx != 0) {
0766:     // Note [out_idx in ViewMeta]
0767:     // When a view op outputs multiple tensors, each output needs its own separate ViewMeta.
0768:     // Each ViewMeta also tracks the index of the particular output tensor, which is needed in the reverse function.
0769:     meta_ = meta->to_out_index(out_idx);
0770:   }
0771:   return at::detail::make_tensor<FunctionalTensorWrapper>(view_to_wrap, functional_base_impl, meta_);
0772: }
0773: 
0774: std::vector<Tensor> create_functional_tensor_with_view_meta(
0775:     ITensorListRef view_to_wrap,
0776:     const at::Tensor& base,
0777:     const std::shared_ptr<functionalization::ViewMeta>& meta) {
0778:   std::vector<Tensor> outputs(view_to_wrap.size());
0779:   int64_t i = 0;
0780:   for (const auto& tensor : view_to_wrap) {
0781:     outputs[i] = create_functional_tensor_with_view_meta(tensor, base, meta, i);
0782:     i++;
0783:   }
0784:   return outputs;
0785: }
0786: 
0787: void mutate_view_meta(const at::Tensor& self, const std::shared_ptr<functionalization::ViewMeta>& meta) {
0788:   TORCH_INTERNAL_ASSERT(at::functionalization::impl::isFunctionalTensor(self));
0789:   auto self_impl = at::functionalization::impl::unsafeGetFunctionalWrapper(self);
0790:   self_impl->mutate_view_meta(meta);
0791: }
0792: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `create_functional_tensor_with_view_meta`, `outputs`, `mutate_view_meta`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`create_functional_tensor_with_view_meta`, `outputs`, `mutate_view_meta`。

### Lines 793-826 / 第 793-826 行

```cpp
0793: Tensor apply_view_meta_sequence(
0794:     const Tensor& base,
0795:     const std::vector<std::shared_ptr<functionalization::ViewMeta>>& sequence) {
0796:   Tensor r = base;
0797:   for (auto& vm : sequence) {
0798:     r = vm->forward(r);
0799:   }
0800:   return r;
0801: }
0802: 
0803: // Note [Propagating strides in the functionalization pass]
0804: // In order to properly compute stride information, the functionalization pass
0805: // calls each {view} reference implementations with meta tensors.
0806: // The output meta tensor's stride info serves as a reference for what the correct strides should be.
0807: void set_sizes_strides_offset(const Tensor& out, const Tensor& reference_out) {
0808:   out.unsafeGetTensorImpl()->set_sizes_and_strides(reference_out.sym_sizes(), reference_out.sym_strides(), reference_out.sym_storage_offset());
0809: }
0810: 
0811: void set_sizes_strides_offset(const std::vector<Tensor>& outs, const std::vector<Tensor>& reference_outs) {
0812:   TORCH_INTERNAL_ASSERT(outs.size() == reference_outs.size());
0813:   for (const auto i : c10::irange(reference_outs.size())) {
0814:     set_sizes_strides_offset(outs[i], reference_outs[i]);
0815:   }
0816: }
0817: 
0818: thread_local static bool _functionalizationReapplyViews;
0819: 
0820: bool getFunctionalizationReapplyViewsTLS() {
0821:   return _functionalizationReapplyViews;
0822: }
0823: void setFunctionalizationReapplyViewsTLS(bool reapply_views) {
0824:   _functionalizationReapplyViews = reapply_views;
0825: }
0826: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `apply_view_meta_sequence`, `set_sizes_strides_offset`, `getFunctionalizationReapplyViewsTLS`, `setFunctionalizationReapplyViewsTLS`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`apply_view_meta_sequence`, `set_sizes_strides_offset`, `getFunctionalizationReapplyViewsTLS`, `setFunctionalizationReapplyViewsTLS`。

### Lines 827-857 / 第 827-857 行

```cpp
0827: } // namespace impl
0828: 
0829: 
0830: // Given an **out-of-place** op that might internally call view/inplace ops,
0831: // This function will "functionalize" it.
0832: // That is, it will call the operator, but removing any intermediate views/mutations
0833: // that are performed inside of it.
0834: // This is useful for LTC/XLA, which would like to reuse some of our composite kernels
0835: // from pytorch core but not have to worry about the view ops that they might call.
0836: // e.g. at::block_diag
0837: void functionalize_op_helper(const c10::OperatorHandle& op, torch::jit::Stack* stack) {
0838:   const auto& schema = op.schema();
0839:   const auto num_arguments = schema.arguments().size();
0840:   const auto arguments_begin = stack->size() - num_arguments;
0841:   auto arguments = torch::jit::last(stack, num_arguments);
0842: 
0843:   // Wrap all tensor-like inputs into FunctionalTensorWrappers.
0844:   // When we re-invoke the dispatcher, this will automatically enable the functionalization pass.
0845:   for (uint64_t idx = 0; idx < num_arguments; ++idx) {
0846:     const auto& ivalue = arguments[idx];
0847:     if (ivalue.isTensor()) {
0848:       const auto& t = ivalue.toTensor();
0849:       if (t.defined()) {
0850:         TORCH_INTERNAL_ASSERT(!at::functionalization::impl::isFunctionalTensor(t),
0851:           "The composite op functionalization fallback expects its inputs all not to be functional tensors");
0852:         auto t_new = c10::IValue(at::functionalization::impl::to_functional_tensor(t));
0853:         (*stack)[arguments_begin + idx] = t_new;
0854:       }
0855:     } else if (ivalue.isTensorList()) {
0856:       auto tensors = ivalue.toTensorList();
0857:       TORCH_INTERNAL_ASSERT(!at::functionalization::impl::isFunctionalTensor(tensors),
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `functionalize_op_helper`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`functionalize_op_helper`。

### Lines 858-899 / 第 858-899 行

```cpp
0858:         "The composite op functionalization fallback expects its inputs all not to be functional tensors");
0859:       auto t_new = c10::IValue(at::functionalization::impl::to_functional_tensor(tensors));
0860:       (*stack)[arguments_begin + idx] = t_new;
0861:     } else if (ivalue.isOptionalTensorList()) {
0862:       auto opt_tensors = ivalue.toOptionalTensorList();
0863:       TORCH_INTERNAL_ASSERT(!at::functionalization::impl::isFunctionalTensor(opt_tensors),
0864:         "The composite op functionalization fallback expects its inputs all not to be functional tensors");
0865:       auto t_new = c10::IValue(at::functionalization::impl::to_functional_tensor(opt_tensors));
0866:       (*stack)[arguments_begin + idx] = t_new;
0867:     } else if (ivalue.isList()) {
0868:       // Handle nested lists containing tensor lists (e.g., Tensor[][]).
0869:       auto list = ivalue.toList();
0870:       for (const auto i : c10::irange(list.size())) {
0871:         const auto& elem = list.get(i);
0872:         if (elem.isTensorList()) {
0873:           auto tensors = elem.toTensorList();
0874:           TORCH_INTERNAL_ASSERT(!at::functionalization::impl::isFunctionalTensor(tensors),
0875:             "The composite op functionalization fallback expects its inputs all not to be functional tensors");
0876:           list.set(i, c10::IValue(at::functionalization::impl::to_functional_tensor(tensors)));
0877:         }
0878:       }
0879:     }
0880:   }
0881: 
0882:   {
0883:     // Today when you call at::empty(device=lazy), the lazy backend decides whether or not to wrap
0884:     // the output in a functional tensor based on TLS.
0885:     // In this code, we're re-entrantly entering functionalization in the same call-stack,
0886:     // so we need to manually fix up TLS as if it hadn't already been called.
0887:     auto curr_tls = c10::impl::tls_local_dispatch_key_set();
0888:     auto tls_reenable_functionalize = c10::impl::PODLocalDispatchKeySet();
0889:     tls_reenable_functionalize.set_included(curr_tls.included_);
0890:     tls_reenable_functionalize.set_excluded(curr_tls.excluded_.remove(c10::DispatchKey::Functionalize));
0891:     c10::impl::ForceDispatchKeyGuard guard_(tls_reenable_functionalize);
0892:     // So, we should probably provide a way to directly call a kernel registered to
0893:     // the `CompositeExplicitAutograd` key.
0894:     // We can't do that today, so this should be a reasonably good proxy
0895:     // (It won't work in cases where an op has both a CompositeExplicitAutograd kernel
0896:     // AND a dedicated meta kernel, but that probably shouldn't ever happen).
0897:     op.redispatchBoxed(c10::DispatchKeySet(c10::DispatchKey::Meta), stack);
0898:   }
0899: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `guard_`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`guard_`。

### Lines 900-936 / 第 900-936 行

```cpp
0900:   const auto num_returns = schema.returns().size();
0901:   const auto returns_begin = stack->size() - num_returns;
0902:   auto returns = torch::jit::last(stack, num_returns);
0903: 
0904:   for (const auto idx : c10::irange(num_returns)) {
0905:     const auto& ivalue = returns[idx];
0906:     if (ivalue.isTensor()) {
0907:       const auto& t = ivalue.toTensor();
0908:       if (!t.defined()) { continue; }
0909:       at::functionalization::impl::sync(t);
0910:       auto t_new = c10::IValue(at::functionalization::impl::from_functional_tensor(t));
0911:       (*stack)[returns_begin + idx] = t_new;
0912:     } else if (ivalue.isTensorList()) {
0913:       auto tensors = ivalue.toTensorList();
0914:       at::functionalization::impl::sync(tensors);
0915:       auto t_new = c10::IValue(at::functionalization::impl::from_functional_tensor(tensors));
0916:       (*stack)[returns_begin + idx] = t_new;
0917:     } else if (ivalue.isOptionalTensorList()) {
0918:       auto opt_tensors = ivalue.toOptionalTensorList();
0919:       at::functionalization::impl::sync(opt_tensors);
0920:       auto t_new = c10::IValue(at::functionalization::impl::from_functional_tensor(opt_tensors));
0921:       (*stack)[returns_begin + idx] = t_new;
0922:     } else if (ivalue.isList()) {
0923:       // Handle nested lists containing tensor lists (e.g., Tensor[][]).
0924:       auto list = ivalue.toList();
0925:       for (const auto i : c10::irange(list.size())) {
0926:         const auto& elem = list.get(i);
0927:         if (elem.isTensorList()) {
0928:           auto tensors = elem.toTensorList();
0929:           at::functionalization::impl::sync(tensors);
0930:           list.set(i, c10::IValue(at::functionalization::impl::from_functional_tensor(tensors)));
0931:         }
0932:       }
0933:     }
0934:   }
0935: }
0936: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions. Key symbols: `sync`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`sync`。

### Lines 937-940 / 第 937-940 行

```cpp
0937: 
0938: 
0939: } // namespace functionalization
0940: } // namespace at
```

- **EN:** Opens or closes namespace scopes so the remaining declarations stay inside the intended ATen/backend module.
- **CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 ATen/后端模块之内。


## Key Concepts / 关键概念
- **Shared ATen infrastructure** — 共享 ATen 基础设施
- **Tensor metadata and value flow** — 张量元数据与数值流
- **Dispatch and backend routing** — 调度与后端路由
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Vendor library descriptor management** — 厂商库描述符管理
- **Memory allocation strategy** — 内存分配策略
- **Device and stream coordination** — 设备与流协调
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **functorch transform support** — functorch 变换支持
- **Runtime validation and invariants** — 运行时校验与不变量

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/FunctionalTensorWrapper.h`, `ATen/core/IListRef.h`, `ATen/core/LegacyTypeDispatch.h`, `c10/util/Exception.h`, `c10/util/irange.h`, `ATen/Functions.h`, `ATen/ops/_propagate_xla_data.h`, `ATen/ops/_to_copy.h`
- **External includes / 外部头文件**: 无
- **Namespaces / 命名空间**: `at`, `functionalization`, `impl`
- **Representative symbols / 代表性符号**: `set_constructor_metadata`, `copy_generic_tensor_metadata`, `refresh_numel`, `refresh_contiguous`, `set_allow_tensor_metadata_change`, `set_custom_sizes_strides`, `set_custom_device`, `FunctionalTensorWrapper`, `freeze_storage`, `functional_storage_impl`, `maybe_mark_symbolic`, `commit_update`, `...`
