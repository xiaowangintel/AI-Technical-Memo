# _lower_utils.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/backends/_lower_utils.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime backends behavior for lower utils, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 backends 子模块里与 lower utils 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```python
import types

import torch
import torch.utils._pytree as pytree
from torch.export import ExportedProgram
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.utils._pytree`, `torch.export`; external imports: `types`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.utils._pytree`, `torch.export`；外部导入：`types`。

### Lines 6-10
```python
from torch.export.pt2_archive._package import AOTI_FILES, package_pt2
from torch.types import FileLike

from ._lowered_aoti_module import LoweredBackendModule

```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.export.pt2_archive._package`, `torch.types`, `._lowered_aoti_module`; external imports: none.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.export.pt2_archive._package`, `torch.types`, `._lowered_aoti_module`；外部导入：无。

### Lines 11-20
```python

def get_new_ep_with_flat_inputs_outputs(ep: ExportedProgram) -> ExportedProgram:
    class FlattenedModule(torch.nn.Module):
        def __init__(
            self,
            original_module: torch.fx.GraphModule,
            in_spec: pytree.TreeSpec,
            out_spec: pytree.TreeSpec,
        ) -> None:
            super().__init__()
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `get_new_ep_with_flat_inputs_outputs`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`get_new_ep_with_flat_inputs_outputs`。

### Lines 21-25
```python
            self.original_module = original_module
            self.in_spec = in_spec
            self.out_spec = out_spec

        def forward(self, *flat_inputs):  # type: ignore[no-untyped-def]
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `forward`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`forward`。

### Lines 26-32
```python
            # Unflatten inputs to original structure
            inputs = pytree.tree_unflatten(flat_inputs, self.in_spec)
            args, kwargs = inputs
            outputs = self.original_module(*args, **kwargs)
            # Flatten outputs
            flat_outputs, _ = pytree.tree_flatten(outputs)
            return tuple(flat_outputs)
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 33-40
```python

    flattened_module = FlattenedModule(
        ep.module(), ep.call_spec.in_spec, ep.call_spec.out_spec
    )
    args, kwargs = ep.example_inputs
    flat_inputs, _ = pytree.tree_flatten((args, kwargs))
    flat_ep = torch.export.export(flattened_module, tuple(flat_inputs))

```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 41-50
```python
    return flat_ep


def lower_exported_program(
    exported_program: ExportedProgram, model_name: str, backend_id: str
) -> tuple[ExportedProgram, AOTI_FILES]:
    """
    Lower an exported program to AOTInductor and return a delegate ExportedProgram
    with the `executorch_call_delegate` HOP
    """
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `lower_exported_program`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`lower_exported_program`。

### Lines 51-55
```python
    args, kwargs = exported_program.example_inputs
    out_spec = exported_program.call_spec.out_spec
    flat_ep = get_new_ep_with_flat_inputs_outputs(exported_program)
    flat_inputs, _ = pytree.tree_flatten((args, kwargs))

```
- EN: This block advances ONNX export translation; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：无明显局部符号。

### Lines 56-63
```python
    aoti_files = torch._inductor.aot_compile(
        flat_ep.module(), tuple(flat_inputs), options={"aot_inductor.package": True}
    )
    if not isinstance(aoti_files, list):
        raise AssertionError(
            f"aoti_files must be a list, got {type(aoti_files).__name__}"
        )

```
- EN: This block checks invariants or expected outcomes; bridges to backend-specific execution artifacts; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；桥接到特定后端的执行产物；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 64-68
```python
    lowered_aoti_module = LoweredBackendModule(
        flat_ep, backend_id, module_name=model_name
    )

    def patched_forward(self, *args, **kwargs):  # type: ignore[no-untyped-def]
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `patched_forward`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`patched_forward`。

### Lines 69-73
```python
        flat_inputs, _ = pytree.tree_flatten((args, kwargs))
        flat_outputs = torch._higher_order_ops.executorch_call_delegate(
            self, *flat_inputs
        )
        if out_spec is not None and flat_outputs is not None:
```
- EN: This block coordinates runtime execution state; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块协调运行时执行状态；处理条件控制流。关键符号：无明显局部符号。

### Lines 74-79
```python
            return pytree.tree_unflatten(flat_outputs, out_spec)
        else:
            return flat_outputs

    lowered_aoti_module.forward = types.MethodType(patched_forward, lowered_aoti_module)  # type: ignore[method-assign]

```
- EN: This block handles tensor metadata or sample values; bridges to backend-specific execution artifacts; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；桥接到特定后端的执行产物；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 80-84
```python
    aoti_delegate_ep = torch.export.export(lowered_aoti_module, args, kwargs)

    return aoti_delegate_ep, aoti_files


```
- EN: This block advances ONNX export translation; coordinates runtime execution state; bridges to backend-specific execution artifacts; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；协调运行时执行状态；桥接到特定后端的执行产物；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 85-94
```python
def package_nativert_with_aoti_delegate(
    f: FileLike,
    model_name: str,
    backend_id: str,
    original_ep: ExportedProgram,
    delegate_ep: ExportedProgram,
    delegate_files: AOTI_FILES,
) -> None:
    """
    Package a pt2 archive file that can be consumed by NativeRT with AOTI Delegate
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `package_nativert_with_aoti_delegate`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`package_nativert_with_aoti_delegate`。

### Lines 95-104
```python
    """
    package_pt2(
        f,
        exported_programs={
            model_name: original_ep,
            f"{model_name}-{backend_id}": delegate_ep,
        },
        aoti_files={f"{model_name}-{backend_id}": delegate_files},  # type: ignore[dict-item]
    )
    return
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; coordinates runtime execution state; bridges to backend-specific execution artifacts. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；协调运行时执行状态；桥接到特定后端的执行产物。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Runtime execution — the file schedules or executes native runtime programs and delegates.
  CN: 运行时执行——该文件会调度或执行原生运行时程序及其 delegate。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.utils._pytree`, `torch.export`, `torch.export.pt2_archive._package`, `torch.types`, `._lowered_aoti_module`
- External imports / 外部导入: `types`
- Representative symbols / 代表性符号: `get_new_ep_with_flat_inputs_outputs`, `lower_exported_program`, `package_nativert_with_aoti_delegate`
