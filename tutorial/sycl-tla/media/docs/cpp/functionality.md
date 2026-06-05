# functionality — Documentation Analysis / 文档分析

## Source / 来源
- Source: `/root/xw/sycl-tla/media/docs/cpp/functionality.md`
- Purpose: Summarizes CUTLASS functional coverage across GEMM, implicit GEMM convolution, Tensor Core warp instructions, and WMMA-backed warp APIs. / 总结 CUTLASS 在 GEMM、隐式 GEMM 卷积、Tensor Core warp 指令与 WMMA warp API 方面的功能覆盖范围。

## Content Analysis / 内容分析
### Functionality
**EN:** The opening block defines the notation used in all later tables: layout abbreviations (`N`, `T`, `NHWC`, `NCxHWx`), datatype shorthands, and opcode classes such as SIMT, TensorOp, Sparse TensorOp, and WMMA. It also states the baseline requirement for CUTLASS 3.x style kernels.
**CN:** 开头部分先定义后续表格中的缩写：布局记号（`N`、`T`、`NHWC`、`NCxHWx`）、数据类型简称，以及 SIMT、TensorOp、Sparse TensorOp、WMMA 等操作类别，并说明 CUTLASS 3.x 内核的基础版本要求。

### Device-level GEMM
**EN:** This section is mainly a capability matrix. Rather than teaching GEMM APIs, it tells the reader which combinations of architecture, toolkit, datatype, and layout are already implemented and points to unit tests as the most concrete reference.
**CN:** 本节更像“能力清单”而不是 API 教程：它说明哪些架构、工具链、数据类型与布局组合已经被实现，并把对应单元测试作为最直接的参考入口。

### CUTLASS 3.x Kernels
**EN:** The 3.x table is intentionally narrow and modern, focusing on SM90a TensorOp kernels for f16, bf16, f32/tf32, and s8. The message is that 3.x documents current high-end kernels first, with example tests standing in for template recipes.
**CN:** 3.x 表格范围较集中，主要覆盖面向 SM90a 的现代 TensorOp 内核，涉及 f16、bf16、f32/tf32 与 s8。它强调的是“当前高端内核支持面”，并用测试示例替代冗长的模板说明。

### CUTLASS 2.x Kernels
**EN:** The legacy 2.x table is much broader. It spans SM50 through SM90 and includes SIMT, WMMA, TensorOp, and sparse TensorOp variants, covering floating-point, integer, binary, and complex cases. This makes it useful as a compatibility catalog for older code paths.
**CN:** 传统 2.x 表格覆盖更广，从 SM50 到 SM90，包含 SIMT、WMMA、TensorOp 和稀疏 TensorOp，并覆盖浮点、整数、二值与复数场景，因此更适合作为旧代码路径的兼容性总表。

### Device-level Implicit GEMM convolution
**EN:** This section mirrors the GEMM catalog format for convolution kernels, especially forward propagation. It highlights the importance of tensor layouts (`NHWC`, `NCxHWx`) and shows where Tensor Core support becomes available by datatype and architecture.
**CN:** 本节用与 GEMM 类似的目录式表格介绍卷积内核，重点是前向传播。它突出张量布局（`NHWC`、`NCxHWx`）的重要性，并展示不同数据类型在不同架构上何时获得 Tensor Core 支持。

### Warp-level Matrix Multiply with Tensor Cores
**EN:** The first half maps instruction shapes to supported warp tile shapes. The second half is more practical: it explains which shared-memory layouts must be paired with each operand and global-memory layout so that Tensor Core instructions can be fed efficiently.
**CN:** 前半部分给出指令形状与 warp 级 tile 形状的对应关系；后半部分更偏实现细节，说明不同操作数与全局内存布局应搭配什么共享内存布局，才能高效驱动 Tensor Core 指令。

### Warp-level Matrix Multiply with CUDA WMMA API
**EN:** The WMMA section presents a simpler interface layer. Compared with the TensorOp section, the key takeaway is that WMMA exposes target-independent warp MMA APIs, while still requiring canonical row-major or column-major shared-memory alignment.
**CN:** WMMA 部分展示了一个更直接的接口层。与前面的 TensorOp 章节相比，这里的重点是：WMMA 提供较为通用的 warp 级矩阵乘 API，但依然要求规范的行主序或列主序共享内存对齐。

## Key Concepts / 关键概念
- Capability tables / 功能覆盖表
- Compute capability and CUDA version gating / 架构与 CUDA 版本门槛
- SIMT vs TensorOp vs SpTensorOp vs WMMA / 不同执行路径对比
- Layout-sensitive kernel availability / 与布局强相关的内核支持
- Unit tests as reference instantiations / 以单元测试作为实例入口

## Related Files / 相关文件
- `include/cutlass/gemm/` — GEMM core components / GEMM 核心组件
- `include/cutlass/conv/` — convolution operator support / 卷积算子支持
- `include/cutlass/layout/` — layout types referenced by the tables / 表格中反复引用的布局类型
- `test/unit/gemm/device/` — example kernel instantiations / 示例内核实例化
- `media/docs/cpp/layout.md` — companion layout background / 配套布局背景文档
