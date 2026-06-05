# LlgaTensorImpl.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/codegen/onednn/LlgaTensorImpl.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Contains code-generation logic or generated declarations that synthesize JIT-related bindings, schemas, or helper code. This specific file centers on `LlgaTensorImpl.cpp`. Generation of declarations, registrations, or helper code is part of the responsibility.
- **Purpose (CN)**: 包含代码生成逻辑或生成后的声明，用于产出 JIT 相关绑定、模式或辅助代码。 该文件具体围绕 `LlgaTensorImpl.cpp` 展开。 其职责包含声明、注册或辅助代码的生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
#include <ATen/Config.h>

#if AT_MKLDNN_ENABLED()
#include <c10/core/CPUAllocator.h>
#include <torch/csrc/jit/codegen/onednn/LlgaTensorImpl.h>

namespace torch::jit::fuser::onednn {

// Non-default dnnl::graph::allocator needs an allocator.
// We would let it use c10::GetCPUAllocator's allocator,
// which uses posix_memalign with 64 byte alignment-size.
static void* pytorch_default_allocator(size_t size, size_t alignment) {
  static c10::Allocator* c10_allocator = c10::GetCPUAllocator();
  return c10_allocator->raw_allocate(size);
}

// Non-default dnnl::graph::allocator needs a deallocator.
// We would let it use c10::GetCPUAllocator's deallocator.
static void pytorch_default_deallocator(void* buf) {
  static c10::Allocator* c10_allocator = c10::GetCPUAllocator();
```

- **EN:** It enters or references namespace scopes such as torch::jit::fuser::onednn, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit::fuser::onednn 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Important callable entry points in this range include pytorch_default_allocator, pytorch_default_deallocator.
- **CN:** 这一段的重要可调用入口包括 pytorch_default_allocator, pytorch_default_deallocator。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 21-40 / 第 21-40 行

```cpp
  c10_allocator->raw_deallocate(buf);
}

dnnl::engine& Engine::getEngine() {
  // Even if the default PyTorch CPU allocator would change, we'd still use the
  // stale value. In practice, we don't expect users to change the CPU allocator
  // dynamically anyway, as users preload jemalloc/tcmalloc at runtime, if they
  // would like to. But this behavior might need to be changed, as some models
  // work better with tcmalloc, while others work better with jemalloc, so
  // switching the CPU allocator at runtime can be useful.
  static dnnl::graph::allocator alloc{
      pytorch_default_allocator, pytorch_default_deallocator};
  static dnnl::engine cpu_engine = dnnl::graph::make_engine_with_allocator(
      dnnl::engine::kind::cpu, /* index = */ 0, alloc);
  return cpu_engine;
}

dnnl::stream& Stream::getStream() {
  static dnnl::stream cpu_stream{Engine::getEngine()};
  return cpu_stream;
```

- **EN:** Important callable entry points in this range include getEngine, getStream.
- **CN:** 这一段的重要可调用入口包括 getEngine, getStream。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Nodes and values / 节点与值, Operator schema / 算子模式, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Nodes and values / 节点与值, Operator schema / 算子模式, Result propagation / 结果传递。

### Lines 41-60 / 第 41-60 行

```cpp
}

LlgaTensorImpl::LlgaTensorImpl(
    at::Storage&& storage,
    const caffe2::TypeMeta& data_type,
    const LlgaTensorDesc& desc)
    : at::TensorImpl(
          std::move(storage),
          c10::DispatchKeySet(c10::DispatchKey::MkldnnCPU),
          data_type),
      desc_(desc) {
  set_sizes_and_strides(desc.sizes(), desc.strides());
  refresh_numel();
}

at::Tensor LlgaTensorImpl::llga_to_aten_tensor(LlgaTensorImpl* llgaImpl) {
  auto aten_tensor = at::detail::make_tensor<TensorImpl>(
      std::move(llgaImpl->storage_),
      c10::DispatchKeySet(c10::DispatchKey::CPU),
      llgaImpl->data_type_);
```

- **EN:** Important callable entry points in this range include LlgaTensorImpl, set_sizes_and_strides, refresh_numel, llga_to_aten_tensor, move.
- **CN:** 这一段的重要可调用入口包括 LlgaTensorImpl, set_sizes_and_strides, refresh_numel, llga_to_aten_tensor, move。
- **EN:** Concepts touched here: Type system / 类型系统, Declared symbols / 声明的符号.
- **CN:** 这里涉及的概念包括：Type system / 类型系统, Declared symbols / 声明的符号。

### Lines 61-80 / 第 61-80 行

```cpp
  auto impl = aten_tensor.unsafeGetTensorImpl();
  impl->set_storage_offset(llgaImpl->storage_offset_);
  impl->set_sizes_and_strides(llgaImpl->sizes(), llgaImpl->strides());
  return aten_tensor;
}

at::Tensor empty_llga(
    const LlgaTensorDesc& desc,
    const c10::TensorOptions& options) {
  auto nbytes = desc.storage_size();

  auto allocator = at::GetCPUAllocator();
  auto storage_impl = c10::make_intrusive<c10::StorageImpl>(
      c10::StorageImpl::use_byte_size_t(),
      nbytes,
      allocator->allocate(nbytes),
      allocator,
      /*resizable=*/false);

  return at::detail::make_tensor<LlgaTensorImpl>(
```

- **EN:** Important callable entry points in this range include empty_llga, use_byte_size_t.
- **CN:** 这一段的重要可调用入口包括 empty_llga, use_byte_size_t。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Operator schema / 算子模式, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Operator schema / 算子模式, Result propagation / 结果传递。

### Lines 81-100 / 第 81-100 行

```cpp
      std::move(storage_impl), options.dtype(), desc);
}

static const LlgaTensorDesc& get_llga_desc(const at::Tensor& tensor) {
  TORCH_INTERNAL_ASSERT(
      tensor.is_mkldnn(), "get_llga_desc expects Mkldnn tensor input");
  return static_cast<LlgaTensorImpl*>(tensor.unsafeGetTensorImpl())->desc();
}

dnnl::graph::tensor llga_from_aten_tensor(const at::Tensor& tensor) {
  return {
      get_llga_desc(tensor).logical_tensor(),
      torch::jit::fuser::onednn::Engine::getEngine(),
      tensor.data_ptr()};
}

using data_type = dnnl::graph::logical_tensor::data_type;

data_type LlgaTensorDesc::getLlgaDataType(at::ScalarType dt) const {
  switch (dt) {
```

- **EN:** Alias declarations such as data_type simplify later API usage.
- **CN:** data_type 等别名声明简化了后续 API 的使用。
- **EN:** Important callable entry points in this range include move, get_llga_desc, TORCH_INTERNAL_ASSERT, llga_from_aten_tensor, getLlgaDataType.
- **CN:** 这一段的重要可调用入口包括 move, get_llga_desc, TORCH_INTERNAL_ASSERT, llga_from_aten_tensor, getLlgaDataType。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 101-120 / 第 101-120 行

```cpp
    case at::ScalarType::Float:
      return data_type::f32;
    case at::ScalarType::BFloat16:
      return data_type::bf16;
    case at::kInt:
      return data_type::s32;
    case at::ScalarType::QInt8:
      return data_type::s8;
    case at::ScalarType::QUInt8:
      return data_type::u8;
    default:
      // If a dtype is unsupported, oneDNN Graph will make that op a wildcard in
      // the graph construction stage. Then when we would execute oneDNN Graph
      // kernels pertaining to oneDNN Graph partitions, such an op would not be
      // inside a oneDNN Graph partition, so we would not encounter inputs with
      // unsupported dtypes at the time of executing compiled partitions.
      return data_type::undef;
  }
}

```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Type system / 类型系统, Operator schema / 算子模式, Backend integration / 后端集成, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Type system / 类型系统, Operator schema / 算子模式, Backend integration / 后端集成, Result propagation / 结果传递。

### Lines 121-140 / 第 121-140 行

```cpp
LlgaTensorDesc LlgaTensorDesc::supplementTensorInfo(const at::Tensor& t) const {
  if (t.is_mkldnn()) {
    // if input tensor is of mkldnn, it's originated from an upstream
    // LLGA partition which carries opaque layout info
    return get_llga_desc(t).tid(tid_);
  } else {
    // if input tensor is not an mkldnn tensor, use default layout
    auto sizes = t.sizes().vec();
    auto strides = t.strides().vec();
    auto dtype = getLlgaDataType(t.scalar_type());
    return {tid_, sizes, strides, dtype, property_type_};
  }
}

at::ScalarType LlgaTensorDesc::aten_scalar_type() const {
  switch (dtype_) {
    case data_type::f32:
      return at::ScalarType::Float;
    case data_type::bf16:
      return at::ScalarType::BFloat16;
```

- **EN:** Important callable entry points in this range include supplementTensorInfo, get_llga_desc, aten_scalar_type.
- **CN:** 这一段的重要可调用入口包括 supplementTensorInfo, get_llga_desc, aten_scalar_type。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 141-154 / 第 141-154 行

```cpp
    case data_type::s32:
      return at::kInt;
    case data_type::s8:
      return at::ScalarType::QInt8;
    case data_type::u8:
      return at::ScalarType::QUInt8;
    default:
      TORCH_CHECK(false, "Invalid data type ", static_cast<size_t>(dtype_));
  }
}

} // namespace torch::jit::fuser::onednn

#endif // AT_MKLDNN_ENABLED()
```

- **EN:** Important callable entry points in this range include TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 TORCH_CHECK。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Declared symbols / 声明的符号, Macro control flow / 宏控制流, Namespace scoping / 命名空间作用域.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Declared symbols / 声明的符号, Macro control flow / 宏控制流, Namespace scoping / 命名空间作用域。

## Key Concepts / 关键概念

- **Generated code pipeline** — 代码生成流水线
- **Graph IR** — 图中间表示
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Backend integration** — 后端集成
- **Code generation** — 代码生成
- **Core symbols: data_type, pytorch_default_allocator, pytorch_default_deallocator, getEngine, getStream, LlgaTensorImpl, set_sizes_and_strides, refresh_numel** — 核心符号：data_type、pytorch_default_allocator、pytorch_default_deallocator、getEngine、getStream、LlgaTensorImpl、set_sizes_and_strides、refresh_numel

## Dependencies / 依赖关系

- `ATen/Config.h`
- `c10/core/CPUAllocator.h`
- `torch/csrc/jit/codegen/onednn/LlgaTensorImpl.h`
