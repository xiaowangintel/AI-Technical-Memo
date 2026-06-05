# NativeKernels.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/kernels/NativeKernels.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime runtime behavior for NativeKernels, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 runtime 子模块里与 NativeKernels 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#include <torch/nativert/kernels/KernelRegistry.h>

#include <ATen/native/NonSymbolicBC.h>

namespace torch::nativert {
```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 6-15
```cpp

REGISTER_NATIVE_CPU_KERNEL("torch.ops.aten.slice.Tensor", aten_slice_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& dim = KernelInput(1).toInt();
  const auto& start = KernelInput(2).toOptional<int64_t>();
  const auto& end = KernelInput(3).toOptional<int64_t>();
  const auto& step = KernelInput(4).toInt();
  KernelOutput(0) = at::native::slice(self, dim, start, end, step);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `slice`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `slice`。

### Lines 16-23
```cpp
REGISTER_NATIVE_CPU_KERNEL("torch.ops.aten.sym_size.int", aten_sym_size_int, {
  const auto& self = KernelInput(0).toTensor();
  const auto dim = KernelInput(1).toInt();
  auto& out = KernelOutput(0);
  TORCH_CHECK(dim >= 0 && dim < self.dim(), "Invalid dimension");
  out = self.sym_size(dim);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `dim`, `sym_size`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `dim`, `sym_size`。

### Lines 24-29
```cpp
REGISTER_NATIVE_CPU_KERNEL("torch.ops.aten.reshape.default", aten_reshape, {
  const auto& self = KernelInput(0).toTensor();
  const auto& shape = KernelInput(1).toIntVector();
  KernelOutput(0) = at::native::reshape(self, shape);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toIntVector`, `KernelOutput`, `reshape`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toIntVector`, `KernelOutput`, `reshape`。

### Lines 30-35
```cpp
REGISTER_NATIVE_CPU_KERNEL("torch.ops.aten.view.default", aten_view, {
  const auto& self = KernelInput(0).toTensor();
  const auto& size = KernelInput(1).toIntVector();
  KernelOutput(0) = at::native::view(self, size);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toIntVector`, `KernelOutput`, `view`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toIntVector`, `KernelOutput`, `view`。

### Lines 36-41
```cpp
REGISTER_NATIVE_CPU_KERNEL("torch.ops.aten.permute.default", aten_permute, {
  const auto& self = KernelInput(0).toTensor();
  const auto& dims = KernelInput(1).toDimVector();
  KernelOutput(0) = at::native::permute(self, dims);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toDimVector`, `KernelOutput`, `permute`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toDimVector`, `KernelOutput`, `permute`。

### Lines 42-48
```cpp
REGISTER_NATIVE_CPU_KERNEL("torch.ops.aten.select.int", aten_select, {
  const auto& self = KernelInput(0).toTensor();
  const auto dim = KernelInput(1).toInt();
  const auto index = KernelInput(2).toInt();
  KernelOutput(0) = at::native::select(self, dim, index);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `select`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `select`。

### Lines 49-55
```cpp
REGISTER_NATIVE_CPU_KERNEL("torch.ops.aten.split.Tensor", aten_split_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto split_size = KernelInput(1).toInt();
  const auto dim = KernelInput(2).toInt();
  KernelOutput(0) = at::native::split(self, split_size, dim);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `split`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `split`。

### Lines 56-65
```cpp
REGISTER_NATIVE_CPU_KERNEL(
    "torch.ops.aten.split_with_sizes.default",
    aten_split_with_sizes,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& split_sizes = KernelInput(1).toIntList();
      const auto dim = KernelInput(2).toInt();
      KernelOutput(0) =
          at::native::split_with_sizes(self, split_sizes.vec(), dim);
    })
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toIntList`, `toInt`, `KernelOutput`, `split_with_sizes`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toIntList`, `toInt`, `KernelOutput`, `split_with_sizes`, `...`。

### Lines 66-75
```cpp

REGISTER_NATIVE_CPU_KERNEL(
    "torch.ops.aten.tensor_split.sections",
    aten_tensor_split_sections,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto sections = KernelInput(1).toInt();
      const auto dim = KernelInput(2).toInt();
      KernelOutput(0) =
          at::native::tensor_split_sections_symint(self, sections, dim);
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `tensor_split_sections_symint`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `tensor_split_sections_symint`。

### Lines 76-82
```cpp
    })

REGISTER_NATIVE_CPU_KERNEL("torch.ops.aten.item.default", aten_item, {
  const auto& self = KernelInput(0).toTensor();
  KernelOutput(0) = at::native::item(self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `item`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `item`。

### Lines 83-87
```cpp
REGISTER_NATIVE_CPU_KERNEL("torch.ops.aten.narrow.default", aten_narrow, {
  const auto& self = KernelInput(0).toTensor();
  const auto dim = KernelInput(1).toInt();
  int64_t start = 0;
  if (KernelInput(2).isScalar()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `isScalar`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `isScalar`。

### Lines 88-95
```cpp
    start = KernelInput(2).toInt();
  } else {
    auto& t = KernelInput(2).toTensor();
    start = t.item<int64_t>();
  }
  const auto length = KernelInput(3).toInt();
  TORCH_CHECK(self.dim() > 0, "narrow() cannot be applied to a 0-dim tensor.");
  auto cur_size = self.sizes()[dim];
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: `KernelInput`, `toInt`, `toTensor`, `dim`, `narrow`, `sizes`.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：`KernelInput`, `toInt`, `toTensor`, `dim`, `narrow`, `sizes`。

### Lines 96-105
```cpp
  if (start != cur_size && start < 0) {
    start = at::maybe_wrap_dim(start, cur_size);
  }
  TORCH_CHECK(
      length >= 0 && start <= cur_size - length,
      "start (",
      start,
      ") + length (",
      length,
      ") exceeds dimension size (",
```
- EN: This block checks invariants or expected outcomes; handles conditional control flow. Key symbols: `maybe_wrap_dim`, `start`, `length`, `size`.
- CN: 该代码块检查不变量或预期结果；处理条件控制流。关键符号：`maybe_wrap_dim`, `start`, `length`, `size`。

### Lines 106-110
```cpp
      cur_size,
      ").");
  KernelOutput(0) = at::native::slice(self, dim, start, start + length, 1);
})

```
- EN: This block implements local helper logic for NativeKernels. Key symbols: `KernelOutput`, `slice`.
- CN: 该代码块实现与 NativeKernels 相关的局部辅助逻辑。关键符号：`KernelOutput`, `slice`。

### Lines 111-111
```cpp
} // namespace torch::nativert
```
- EN: This block implements local helper logic for NativeKernels. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 NativeKernels 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。
- EN: Header-level integration — the translation unit depends on nearby runtime and framework declarations.
  CN: 头文件级集成——该翻译单元依赖周边运行时与框架声明。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/nativert/kernels/KernelRegistry.h`, `ATen/native/NonSymbolicBC.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `slice`, `dim`, `sym_size`, `toIntVector`, `reshape`, `view`, `...`
