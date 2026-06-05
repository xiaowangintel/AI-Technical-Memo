# PrimKernelRegistry.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/kernels/PrimKernelRegistry.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime runtime behavior for PrimKernelRegistry, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 runtime 子模块里与 PrimKernelRegistry 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#include <ATen/record_function.h>

#include <ATen/CPUFunctions.h>
#include <c10/util/irange.h>

```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `ATen/record_function.h`, `ATen/CPUFunctions.h`, `c10/util/irange.h`; external includes: none.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`ATen/record_function.h`, `ATen/CPUFunctions.h`, `c10/util/irange.h`；外部依赖：无。

### Lines 6-10
```cpp
#include <c10/util/Enumerate.h>
#include <torch/nativert/kernels/PrimKernelRegistry.h>

namespace torch::nativert {

```
- EN: This block implements local helper logic for PrimKernelRegistry. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 PrimKernelRegistry 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 11-15
```cpp
C10_DEFINE_REGISTRY(PrimKernelRegistry, OpKernel, const Node*)

namespace {

class OpKernel_prim_listpack : public OpKernel {
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `OpKernel_prim_listpack`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`OpKernel_prim_listpack`。

### Lines 16-25
```cpp
 public:
  explicit OpKernel_prim_listpack(const Node* node)
      : OpKernel(node, OpKernelKind::kPrimKernel) {
    auto listType = node->outputs()[0]->type();
    switch (listType.kind()) {
      case Type::Kind::TensorList:
        type_ = c10::TensorType::get();
        break;
      case Type::Kind::NestedTensorList:
        // For List[List[Tensor]], each element is a List[Tensor]
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `OpKernel_prim_listpack`, `OpKernel`, `outputs`, `type`, `kind`, `get`.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；遍历集合或执行单元。关键符号：`OpKernel_prim_listpack`, `OpKernel`, `outputs`, `type`, `kind`, `get`。

### Lines 26-35
```cpp
        type_ = c10::ListType::create(c10::TensorType::get());
        break;
      case Type::Kind::SymIntList:
        type_ = c10::IntType::get();
        break;
      case Type::Kind::OptionalTensorList:
        type_ = c10::OptionalType::create(c10::TensorType::get());
        break;
      default:
        TORCH_CHECK(false, "Unsupported list type: ", listType);
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: `create`, `get`.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：`create`, `get`。

### Lines 36-43
```cpp
    }
  }

  void computeInternal(ExecutionFrame& executionFrame) const final {
    RECORD_USER_SCOPE("nativert::OpKernel_prim_listpack");
    c10::List<c10::IValue> list(type_);
    list.reserve(numInputs());
    for (size_t i = 0; i < numInputs(); ++i) {
```
- EN: This block coordinates runtime execution state; iterates over collections or execution units. Key symbols: `computeInternal`, `list`, `reserve`, `numInputs`.
- CN: 该代码块协调运行时执行状态；遍历集合或执行单元。关键符号：`computeInternal`, `list`, `reserve`, `numInputs`。

### Lines 44-52
```cpp
      if (KernelInput(i).isNone()) {
        list.emplace_back();
      } else {
        list.push_back(KernelInput(i));
      }
    }
    KernelOutput(0) = std::move(list);
  }

```
- EN: This block handles conditional control flow. Key symbols: `KernelInput`, `isNone`, `emplace_back`, `push_back`, `KernelOutput`, `move`.
- CN: 该代码块处理条件控制流。关键符号：`KernelInput`, `isNone`, `emplace_back`, `push_back`, `KernelOutput`, `move`。

### Lines 53-58
```cpp
 private:
  c10::TypePtr type_;
};

} // namespace

```
- EN: This block implements local helper logic for PrimKernelRegistry. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 PrimKernelRegistry 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 59-63
```cpp
C10_REGISTER_TYPED_CLASS(
    PrimKernelRegistry,
    "prim.ListPack",
    OpKernel_prim_listpack)

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: no dominant local symbols.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：无明显局部符号。

### Lines 64-71
```cpp
REGISTER_PRIM_KERNEL("prim.ListUnpack", prim_listunpack, {
  RECORD_USER_SCOPE("nativert::OpKernel_prim_listunpack");
  auto inputListRef = KernelInput(0).toListRef();
  for (const auto& [i, ivalue] : c10::enumerate(inputListRef)) {
    KernelOutput(i) = ivalue;
  }
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toListRef`, `enumerate`, `KernelOutput`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toListRef`, `enumerate`, `KernelOutput`。

### Lines 72-76
```cpp
// Noop for input and output
REGISTER_PRIM_KERNEL("prim.Input", prim_input, {})
REGISTER_PRIM_KERNEL("prim.Output", prim_output, {})

namespace {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: no dominant local symbols.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：无明显局部符号。

### Lines 77-86
```cpp

class OpKernel_variadic_concat : public OpKernel {
 public:
  explicit OpKernel_variadic_concat(const Node* node)
      : OpKernel(node, OpKernelKind::kPrimKernel) {
    dim_ = !node_->attributes().empty()
        ? constantToIValue(node_->getAttribute("dim").value).toInt()
        : 0;
  }
  void computeInternal(ExecutionFrame& executionFrame) const final {
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `OpKernel_variadic_concat`, `OpKernel`, `attributes`, `empty`, `constantToIValue`, `getAttribute`, `...`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`OpKernel_variadic_concat`, `OpKernel`, `attributes`, `empty`, `constantToIValue`, `getAttribute`, `...`。

### Lines 87-91
```cpp
    {
      const size_t numNodeInps = numInputs();
      auto numCatInps = numNodeInps;
      auto dim = dim_;
      if (KernelInput(numCatInps - 1).isInt()) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `numInputs`, `KernelInput`, `isInt`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`numInputs`, `KernelInput`, `isInt`。

### Lines 92-96
```cpp
        dim = KernelInput(numCatInps - 1).toInt();
        numCatInps--;
      }
      std::vector<at::Tensor> inputs(numCatInps);
      for (const auto i : c10::irange(numCatInps)) {
```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: `KernelInput`, `toInt`, `inputs`, `irange`.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：`KernelInput`, `toInt`, `inputs`, `irange`。

### Lines 97-106
```cpp
        inputs[i] = KernelInput(i).toTensor();
      }

      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::cat(inputs, dim);
        return;
      }
      auto& out_t = KernelOutput(0).toTensor();
      fastResizeToZero(out_t);
      at::cpu::cat_outf(inputs, dim, out_t);
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `cat`, `fastResizeToZero`, `...`.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `cat`, `fastResizeToZero`, `...`。

### Lines 107-112
```cpp
    }
  }

 private:
  int dim_;
};
```
- EN: This block implements local helper logic for PrimKernelRegistry. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 PrimKernelRegistry 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 113-120
```cpp

} // namespace

C10_REGISTER_TYPED_CLASS(
    PrimKernelRegistry,
    "prim.VarConcat",
    OpKernel_variadic_concat)

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: no dominant local symbols.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：无明显局部符号。

### Lines 121-130
```cpp
namespace {

class OpKernel_variadic_stack : public OpKernel {
 public:
  explicit OpKernel_variadic_stack(const Node* node)
      : OpKernel(node, OpKernelKind::kPrimKernel) {
    dim_ = !node_->attributes().empty()
        ? constantToIValue(node_->getAttribute("dim").value).toInt()
        : 0;
  }
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `OpKernel_variadic_stack`, `OpKernel`, `attributes`, `empty`, `constantToIValue`, `getAttribute`, `...`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`OpKernel_variadic_stack`, `OpKernel`, `attributes`, `empty`, `constantToIValue`, `getAttribute`, `...`。

### Lines 131-135
```cpp
  void computeInternal(ExecutionFrame& executionFrame) const final {
    {
      const size_t numNodeInps = numInputs();
      auto numStackInps = numNodeInps;
      auto dim = dim_;
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state. Key symbols: `computeInternal`, `numInputs`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态。关键符号：`computeInternal`, `numInputs`。

### Lines 136-141
```cpp
      if (KernelInput(numStackInps - 1).isInt()) {
        dim = KernelInput(numStackInps - 1).toInt();
        numStackInps--;
      }
      std::vector<at::Tensor> inputs(numStackInps);
      for (const auto i : c10::irange(numStackInps)) {
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: `KernelInput`, `isInt`, `toInt`, `inputs`, `irange`.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：`KernelInput`, `isInt`, `toInt`, `inputs`, `irange`。

### Lines 142-151
```cpp
        inputs[i] = KernelInput(i).toTensor();
      }
      auto& out = KernelOutput(0);
      if (out.isNone()) {
        out = at::native::_stack_cpu(inputs, dim);
        return;
      }
      auto& out_t = out.toTensor();
      fastResizeToZero(out_t);
      at::native::_stack_out_cpu(inputs, dim, out_t);
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `_stack_cpu`, `fastResizeToZero`, `...`.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `_stack_cpu`, `fastResizeToZero`, `...`。

### Lines 152-157
```cpp
    }
  }

 private:
  int64_t dim_;
};
```
- EN: This block implements local helper logic for PrimKernelRegistry. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 PrimKernelRegistry 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 158-164
```cpp
} // namespace

C10_REGISTER_TYPED_CLASS(
    PrimKernelRegistry,
    "prim.VarStack",
    OpKernel_variadic_stack)

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: no dominant local symbols.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：无明显局部符号。

### Lines 165-165
```cpp
} // namespace torch::nativert
```
- EN: This block implements local helper logic for PrimKernelRegistry. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 PrimKernelRegistry 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Runtime execution — the file schedules or executes native runtime programs and delegates.
  CN: 运行时执行——该文件会调度或执行原生运行时程序及其 delegate。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。
- EN: Header-level integration — the translation unit depends on nearby runtime and framework declarations.
  CN: 头文件级集成——该翻译单元依赖周边运行时与框架声明。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `ATen/record_function.h`, `ATen/CPUFunctions.h`, `c10/util/irange.h`, `c10/util/Enumerate.h`, `torch/nativert/kernels/PrimKernelRegistry.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `OpKernel_prim_listpack`, `OpKernel`, `outputs`, `type`, `kind`, `get`, `create`, `computeInternal`, `list`, `reserve`, `...`
