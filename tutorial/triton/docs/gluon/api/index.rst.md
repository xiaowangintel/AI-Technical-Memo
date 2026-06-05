# index.rst — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** `/root/xw/triton/docs/gluon/api/index.rst`
- **EN:** Gluon API reference page for index.rst.
- **CN:** index.rst 对应的 Gluon API 参考页。

## Content Analysis / 内容分析
### API Reference
**EN:** This section organizes the Gluon API surface into runtime, language, and vendor-specific reference pages. It links to 4 related pages: Runtime API <runtime>, Language API <language>, NVIDIA Intrinsics <nvidia>, and AMD Intrinsics <amd>. Notable prose emphasis: Runtime API <runtime> Language API <language> NVIDIA Intrinsics <nvidia> AMD Intrinsics <amd>
**CN:** 本节将 Gluon API 组织为运行时、语言层和厂商专用参考页面。 它链接到 4 个相关页面：Runtime API <runtime>、Language API <language>、NVIDIA Intrinsics <nvidia>、AMD Intrinsics <amd>。 其中反复出现的技术关键词包括 Intrinsics、toctree、maxdepth、caption、Gluon、hidden。

### Core APIs
**EN:** This section distinguishes the fundamental Gluon runtime and language interfaces from lower-level vendor intrinsics. Key listed points include Runtime API: JIT decorators, result handling, and host-side and Language API: the core Gluon programming model, layouts,. Notable prose emphasis: tensor descriptors used to launch Gluon kernels. memory operations, math operations, and compile-time helpers.
**CN:** 本节区分了基础的 Gluon 运行时/语言接口与更底层的厂商专用内建。 列出的重点包括 Runtime API: JIT decorators, result handling, and host-side、Language API: the core Gluon programming model, layouts,。 其中反复出现的技术关键词包括 Runtime API <runtime>、Language API <language>、Gluon、operations、Runtime、JIT。

### Target Intrinsics
**EN:** This section highlights the architecture-specific API families that expose backend-specific GPU capabilities. Key listed points include NVIDIA Intrinsics: target-specific APIs for NVIDIA Ampere, and AMD Intrinsics: target-specific APIs for AMD CDNA and RDNA GPU. Notable prose emphasis: Hopper, and Blackwell GPUs. generations.
**CN:** 本节强调暴露后端专用 GPU 能力的体系结构相关 API 家族。 列出的重点包括 NVIDIA Intrinsics: target-specific APIs for NVIDIA Ampere,、AMD Intrinsics: target-specific APIs for AMD CDNA and RDNA GPU。 其中反复出现的技术关键词包括 NVIDIA Intrinsics <nvidia>、AMD Intrinsics <amd>、NVIDIA、Intrinsics、target-specific、APIs。

## Key Concepts / 关键概念
- **EN:** API Reference  **CN:** API 参考
- **EN:** Core APIs  **CN:** 核心 API
- **EN:** Target Intrinsics  **CN:** 目标专用内建
- **EN:** Runtime API <runtime>  **CN:** Runtime API <runtime>
- **EN:** Language API <language>  **CN:** Language API <language>
- **EN:** NVIDIA Intrinsics <nvidia>  **CN:** NVIDIA Intrinsics <nvidia>

## Related Files / 相关文件
- `/root/xw/triton/docs/gluon/api/runtime.rst`
- `/root/xw/triton/docs/gluon/api/language.rst`
- `/root/xw/triton/docs/gluon/api/nvidia.rst`
- `/root/xw/triton/docs/gluon/api/amd.rst`
