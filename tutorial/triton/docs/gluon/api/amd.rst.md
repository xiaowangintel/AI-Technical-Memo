# amd.rst — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** `/root/xw/triton/docs/gluon/api/amd.rst`
- **EN:** Gluon API reference page for amd.rst.
- **CN:** amd.rst 对应的 Gluon API 参考页。

## Content Analysis / 内容分析
### AMD
**EN:** This section introduces the AMD branch of the Gluon API and frames the generation-specific pages that follow. Notable prose emphasis: Target-specific Gluon APIs for AMD GPU generations.
**CN:** 本节介绍 Gluon API 中面向 AMD 的分支，并为后续代际页面建立上下文。 其中反复出现的技术关键词包括 Target-specific、Gluon、APIs、AMD、GPU、generations.。

### Common APIs
**EN:** This section collects AMD-wide abstractions that are shared across multiple GPU generations. It indexes 3 documented entries, including AMDMFMALayout, AMDWMMALayout, and warp_pipeline_stage.
**CN:** 本节汇总跨多个 AMD GPU 代际共享的抽象与接口。 它通过 autosummary 汇总了 3 个文档条目，例如 AMDMFMALayout、AMDWMMALayout、warp_pipeline_stage。 其中反复出现的技术关键词包括 currentmodule、triton.experimental.gluon.language.amd、autosummary、toctree、generated、nosignatures。

### GPU Generations
**EN:** This section splits the AMD material by GPU generation so readers can jump to architecture-specific intrinsics. It links to 4 related pages: CDNA 3 <amd.cdna3>, CDNA 4 <amd.cdna4>, RDNA 3 <amd.rdna3>, and RDNA 4 <amd.rdna4>. Notable prose emphasis: CDNA 3 <amd.cdna3> CDNA 4 <amd.cdna4> RDNA 3 <amd.rdna3> RDNA 4 <amd.rdna4>
**CN:** 本节按 GPU 代际组织 AMD 内容，便于读者定位架构专用内建。 它链接到 4 个相关页面：CDNA 3 <amd.cdna3>、CDNA 4 <amd.cdna4>、RDNA 3 <amd.rdna3>、RDNA 4 <amd.rdna4>。 其中反复出现的技术关键词包括 CDNA、RDNA、toctree、maxdepth、amd.cdna3、amd.cdna4。

## Key Concepts / 关键概念
- **EN:** AMD  **CN:** AMD
- **EN:** Common APIs  **CN:** 通用 API
- **EN:** GPU Generations  **CN:** GPU 代际
- **EN:** AMDMFMALayout  **CN:** AMDMFMALayout
- **EN:** AMDWMMALayout  **CN:** AMDWMMALayout
- **EN:** warp_pipeline_stage  **CN:** warp_pipeline_stage

## Related Files / 相关文件
- `/root/xw/triton/docs/gluon/api/index.rst`
- `/root/xw/triton/docs/gluon/index.rst`
