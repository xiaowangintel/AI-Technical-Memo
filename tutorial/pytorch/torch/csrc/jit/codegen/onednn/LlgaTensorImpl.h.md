# LlgaTensorImpl.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/codegen/onednn/LlgaTensorImpl.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Contains code-generation logic or generated declarations that synthesize JIT-related bindings, schemas, or helper code. This specific file centers on `LlgaTensorImpl.h`. Generation of declarations, registrations, or helper code is part of the responsibility.
- **Purpose (CN)**: 包含代码生成逻辑或生成后的声明，用于产出 JIT 相关绑定、模式或辅助代码。 该文件具体围绕 `LlgaTensorImpl.h` 展开。 其职责包含声明、注册或辅助代码的生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

```cpp
#pragma once

#include <ATen/ATen.h>
#include <ATen/Config.h>

#include <oneapi/dnnl/dnnl_graph.hpp>
#include <torch/csrc/jit/ir/ir.h>
#include <utility>

namespace torch::jit::fuser::onednn {

// Engine represents a device and its context. From the device kind, the engine
// knows how to generate code for the target device and what kind of device
// object to be expected. The device id ensures that there is a unique engine
// being created for each device. The device handle passed from PyTorch allows
// oneDNN Graph implementation to work on the device specified by PyTorch, which
// is currently CPU, so we only have one engine.
// Ref:
// https://oneapi-spec.uxlfoundation.org/specifications/oneapi/latest/elements/onednn/source/graph/programming_model#engine
struct Engine {
  // CPU engine singleton
  static dnnl::engine& getEngine();
  Engine(const Engine&) = delete;
  void operator=(const Engine&) = delete;
};

// Stream is the logical abstraction for execution units. It is created on top
// of oneDNN Graph engine. A compiled oneDNN Graph partition is submitted to a
```

- **EN:** It enters or references namespace scopes such as torch::jit::fuser::onednn, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit::fuser::onednn 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** The block declares or refines core types including Engine.
- **CN:** 该代码块声明或细化了 Engine 等核心类型。
- **EN:** Important callable entry points in this range include getEngine.
- **CN:** 这一段的重要可调用入口包括 getEngine。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。

### Lines 29-56 / 第 29-56 行

```cpp
// stream for execution.
struct Stream {
  // CPU stream singleton
  static dnnl::stream& getStream();
  Stream(const Stream&) = delete;
  void operator=(const Stream&) = delete;
};

struct LlgaTensorDesc {
  using desc = dnnl::graph::logical_tensor;

  LlgaTensorDesc(
      size_t tid,
      std::vector<int64_t> sizes,
      std::vector<int64_t> strides,
      desc::data_type dtype,
      desc::property_type property_type)
      : tid_(tid),
        sizes_(std::move(sizes)),
        strides_(std::move(strides)),
        dtype_(dtype),
        property_type_(property_type),
        layout_type_(desc::layout_type::strided),
        layout_id_(-1) {}

  LlgaTensorDesc(const desc& t)
      : tid_(t.get_id()),
        sizes_(t.get_dims()),
```

- **EN:** The block declares or refines core types including Stream, LlgaTensorDesc.
- **CN:** 该代码块声明或细化了 Stream, LlgaTensorDesc 等核心类型。
- **EN:** Alias declarations such as desc simplify later API usage.
- **CN:** desc 等别名声明简化了后续 API 的使用。
- **EN:** Important callable entry points in this range include getStream, LlgaTensorDesc.
- **CN:** 这一段的重要可调用入口包括 getStream, LlgaTensorDesc。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。

### Lines 57-84 / 第 57-84 行

```cpp
        strides_({-1}),
        dtype_(t.get_data_type()),
        property_type_(t.get_property_type()),
        layout_type_(t.get_layout_type()),
        layout_id_(-1) {
    if (is_opaque()) {
      layout_id_ = t.get_layout_id();
    }
    if (is_strided()) {
      strides_ = t.get_strides();
    }
  }

  LlgaTensorDesc(const torch::jit::Value* v)
      : LlgaTensorDesc(
            v->unique(),
            {},
            {},
            desc::data_type::f32,
            get_property_type(v)) {
    if (v->type()->isSubtypeOf(TensorType::get())) {
      auto tt = v->type()->cast<TensorType>();

      if (tt->scalarType()) {
        dtype_ = getLlgaDataType(tt->scalarType().value());
      }

      auto sizes = tt->sizes();
```

- **EN:** Important callable entry points in this range include dtype_, get_property_type.
- **CN:** 这一段的重要可调用入口包括 dtype_, get_property_type。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Branching logic / 分支逻辑。

### Lines 85-112 / 第 85-112 行

```cpp
      if (sizes.sizes()) {
        for (auto d : *sizes.sizes()) {
          sizes_.push_back(d.value_or(DNNL_GRAPH_UNKNOWN_DIM));
        }
      }

      auto strides = tt->strides();
      if (strides.sizes()) {
        for (auto d : *strides.sizes()) {
          strides_.push_back(d.value_or(DNNL_GRAPH_UNKNOWN_DIM));
        }
      }
    }
  }

  LlgaTensorDesc supplementTensorInfo(const at::Tensor& t) const;

  desc::data_type getLlgaDataType(at::ScalarType dt) const;

  at::ScalarType aten_scalar_type() const;

  const std::vector<int64_t>& sizes() const {
    return sizes_;
  }

  const std::vector<int64_t>& strides() const {
    TORCH_CHECK(!is_opaque(), "Cannot get strides on opaque layout");
    return strides_;
```

- **EN:** Important callable entry points in this range include supplementTensorInfo, getLlgaDataType, aten_scalar_type, sizes, strides, TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 supplementTensorInfo, getLlgaDataType, aten_scalar_type, sizes, strides, TORCH_CHECK。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 113-140 / 第 113-140 行

```cpp
  }

  size_t tid() const {
    return tid_;
  }

  LlgaTensorDesc tid(uint64_t new_id) const {
    auto ret = *this;
    ret.tid_ = new_id;
    return ret;
  }

  desc::data_type dtype() const {
    return dtype_;
  }

  LlgaTensorDesc dtype(desc::data_type new_dtype) const {
    return LlgaTensorDesc(tid_, sizes_, strides_, new_dtype, property_type_);
  }

  desc::layout_type layout_type() const {
    return layout_type_;
  }

  LlgaTensorDesc layout_type(desc::layout_type new_layout_type) {
    auto ret = *this;
    ret.layout_type_ = new_layout_type;
    return ret;
```

- **EN:** Important callable entry points in this range include tid, dtype, LlgaTensorDesc, layout_type.
- **CN:** 这一段的重要可调用入口包括 tid, dtype, LlgaTensorDesc, layout_type。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Type system / 类型系统, Operator schema / 算子模式, Declared symbols / 声明的符号, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Type system / 类型系统, Operator schema / 算子模式, Declared symbols / 声明的符号, Result propagation / 结果传递。

### Lines 141-168 / 第 141-168 行

```cpp
  }

  desc::property_type get_property_type(const torch::jit::Value* v) {
    switch (v->node()->kind()) {
      case prim::Constant:
        return desc::property_type::constant;
      default:
        return desc::property_type::variable;
    }
  }

  LlgaTensorDesc any() {
    return layout_type(desc::layout_type::any);
  }

  size_t storage_size() const {
    return logical_tensor().get_mem_size();
  }

  desc logical_tensor() const {
    if (is_dimensionality_unknown()) {
      return desc(
          tid_, dtype_, DNNL_GRAPH_UNKNOWN_NDIMS, layout_type_, property_type_);
    } else if (is_opaque()) {
      return desc(tid_, dtype_, sizes_, layout_id_, property_type_);
    } else if (is_any()) {
      return desc(tid_, dtype_, sizes_, layout_type_, property_type_);
    } else {
```

- **EN:** Important callable entry points in this range include get_property_type, any, layout_type, storage_size, logical_tensor, desc.
- **CN:** 这一段的重要可调用入口包括 get_property_type, any, layout_type, storage_size, logical_tensor, desc。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 169-196 / 第 169-196 行

```cpp
      return desc(tid_, dtype_, sizes_, strides_, property_type_);
    }
  }

  bool is_strided() const {
    return layout_type_ == desc::layout_type::strided;
  }

  bool is_any() const {
    return layout_type_ == desc::layout_type::any;
  }

  bool is_opaque() const {
    return layout_type_ == desc::layout_type::opaque;
  }

  bool operator==(const LlgaTensorDesc& desc) const {
    return tid_ == desc.tid_ && sizes_ == desc.sizes_ &&
        dtype_ == desc.dtype_ && layout_type_ == desc.layout_type_ &&
        ((is_opaque() && layout_id_ == desc.layout_id_) ||
         strides_ == desc.strides_);
  }

  bool operator!=(const LlgaTensorDesc& desc) const {
    return (tid_ != desc.tid_) || (sizes_ != desc.sizes_) ||
        (dtype_ != desc.dtype_) || (layout_type_ != desc.layout_type_) ||
        !((is_opaque() && (layout_id_ == desc.layout_id_)) ||
          (strides_ == desc.strides_));
```

- **EN:** Important callable entry points in this range include desc, is_strided, is_any, is_opaque.
- **CN:** 这一段的重要可调用入口包括 desc, is_strided, is_any, is_opaque。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Type system / 类型系统, Operator schema / 算子模式, Registration / 注册机制, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Type system / 类型系统, Operator schema / 算子模式, Registration / 注册机制, Result propagation / 结果传递。

### Lines 197-224 / 第 197-224 行

```cpp
  }

  static size_t hash(const LlgaTensorDesc& desc) {
    return c10::get_hash(
        desc.tid_,
        desc.sizes_,
        desc.dtype_,
        desc.layout_type_,
        desc.layout_id_);
  }

  void set_compute_inplace() {
    compute_inplace_ = true;
  }

  void set_input_tensor_index(size_t index) {
    input_tensor_index_ = index;
  }

  bool reuses_input_tensor() {
    return compute_inplace_;
  }

  size_t get_input_tensor_index() {
    return input_tensor_index_;
  }

 private:
```

- **EN:** Important callable entry points in this range include hash, get_hash, set_compute_inplace, set_input_tensor_index, reuses_input_tensor, get_input_tensor_index.
- **CN:** 这一段的重要可调用入口包括 hash, get_hash, set_compute_inplace, set_input_tensor_index, reuses_input_tensor, get_input_tensor_index。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Result propagation / 结果传递。

### Lines 225-252 / 第 225-252 行

```cpp
  bool is_dimensionality_unknown() const {
    return sizes_.empty();
  }

  size_t tid_;
  std::vector<int64_t> sizes_;
  std::vector<int64_t> strides_;
  desc::data_type dtype_;
  desc::property_type property_type_;
  desc::layout_type layout_type_;
  size_t layout_id_;
  // If this is an output tensor, and querying the compiled partition would
  // determine that this tensor would reuse its input tensor, then
  // compute_inplace would be true, and input_tensor_index would be the index of
  // the corresponding input tensor in inputSpecs_ of the LlgaKernel object.
  bool compute_inplace_ = false;
  size_t input_tensor_index_{};
};

// Initially, oneDNN Graph also used to have blocked layout for tensors between
// partitions, and the LlgaTensorImpl wrapper helped us bypass guard checks.
// oneDNN Graph has switched over to using strided tensors between partitions,
// but this wrapper still helps us bypass guard checks because the strides of
// tensors between partitions would be different from the ones the guard is
// otherwise expecting.
struct TORCH_API LlgaTensorImpl : public c10::TensorImpl {
  LlgaTensorImpl(
      at::Storage&& storage,
```

- **EN:** The block declares or refines core types including LlgaTensorImpl.
- **CN:** 该代码块声明或细化了 LlgaTensorImpl 等核心类型。
- **EN:** Important callable entry points in this range include is_dimensionality_unknown.
- **CN:** 这一段的重要可调用入口包括 is_dimensionality_unknown。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 253-272 / 第 253-272 行

```cpp
      const caffe2::TypeMeta& data_type,
      const LlgaTensorDesc& desc);

  const LlgaTensorDesc& desc() const {
    return desc_;
  }

  static at::Tensor llga_to_aten_tensor(LlgaTensorImpl* llgaImpl);

 private:
  LlgaTensorDesc desc_;
};

at::Tensor empty_llga(
    const LlgaTensorDesc& desc,
    const c10::TensorOptions& options);

dnnl::graph::tensor llga_from_aten_tensor(const at::Tensor& tensor);

} // namespace torch::jit::fuser::onednn
```

- **EN:** Important callable entry points in this range include desc, llga_to_aten_tensor, empty_llga, llga_from_aten_tensor.
- **CN:** 这一段的重要可调用入口包括 desc, llga_to_aten_tensor, empty_llga, llga_from_aten_tensor。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Namespace scoping / 命名空间作用域, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Namespace scoping / 命名空间作用域, Result propagation / 结果传递。

## Key Concepts / 关键概念

- **Generated code pipeline** — 代码生成流水线
- **Graph IR** — 图中间表示
- **Nodes and values** — 节点与值
- **Control-flow blocks** — 控制流块
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Module API** — 模块 API
- **Backend integration** — 后端集成
- **Optimization pass** — 优化 pass
- **Registration** — 注册机制

## Dependencies / 依赖关系

- `ATen/ATen.h`
- `ATen/Config.h`
- `torch/csrc/jit/ir/ir.h`
