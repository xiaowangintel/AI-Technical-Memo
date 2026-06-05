# runtime.rst — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** `/root/xw/triton/docs/gluon/api/runtime.rst`
- **EN:** Gluon API reference page for runtime.rst.
- **CN:** runtime.rst 对应的 Gluon API 参考页。

## Content Analysis / 内容分析
### Runtime API
**EN:** This section presents the Gluon runtime-facing entry points used to define and launch kernels from Python.
**CN:** 本节介绍从 Python 定义和启动 Gluon 内核所需的运行时入口。 其中反复出现的技术关键词包括 currentmodule、triton.experimental.gluon。

### Runtime
**EN:** This section focuses on runtime constructs such as JIT decorators, result handling, and compile-time helpers. It indexes 4 documented entries, including GluonJITFunction, jit, constexpr_function, and must_use_result.
**CN:** 本节聚焦于 JIT 装饰器、结果处理和编译期辅助等运行时构件。 它通过 autosummary 汇总了 4 个文档条目，例如 GluonJITFunction、jit、constexpr_function、must_use_result。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、GluonJITFunction、jit。

### Host-Side Descriptors
**EN:** This section covers host-visible tensor descriptor types that prepare structured arguments for kernels. It indexes 3 documented entries, including nvidia.hopper.TensorDescriptor, nvidia.hopper.TensorDescriptorIm2Col, and nvidia.blackwell.TensorDescriptor.
**CN:** 本节介绍主机侧可见的张量描述符类型，用于为内核准备结构化参数。 它通过 autosummary 汇总了 3 个文档条目，例如 nvidia.hopper.TensorDescriptor、nvidia.hopper.TensorDescriptorIm2Col、nvidia.blackwell.TensorDescriptor。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、nvidia.hopper.TensorDescriptor、nvidia.hopper.TensorDescriptorIm2Col。

## Key Concepts / 关键概念
- **EN:** Runtime API  **CN:** 运行时 API
- **EN:** Runtime  **CN:** 运行时
- **EN:** Host-Side Descriptors  **CN:** 主机侧描述符
- **EN:** GluonJITFunction  **CN:** GluonJITFunction
- **EN:** jit  **CN:** jit
- **EN:** constexpr_function  **CN:** constexpr_function

## Related Files / 相关文件
- `/root/xw/triton/docs/gluon/api/index.rst`
- `/root/xw/triton/docs/gluon/index.rst`
