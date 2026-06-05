# language.rst — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** `/root/xw/triton/docs/gluon/api/language.rst`
- **EN:** Gluon API reference page for language.rst.
- **CN:** language.rst 对应的 Gluon API 参考页。

## Content Analysis / 内容分析
### Language API
**EN:** This section groups the Gluon language surface by programming concepts such as types, memory, math, and debugging.
**CN:** 本节按类型、内存、数学和调试等编程概念组织 Gluon 语言接口。 其中反复出现的技术关键词包括 currentmodule、triton.experimental.gluon.language。

### Types
**EN:** This section enumerates the data abstractions exposed by the language layer. It indexes 3 documented entries, including tensor, shared_memory_descriptor, and distributed_type.
**CN:** 本节枚举语言层暴露的数据抽象。 它通过 autosummary 汇总了 3 个文档条目，例如 tensor、shared_memory_descriptor、distributed_type。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、tensor、shared_memory_descriptor。

### Programming Model
**EN:** This section describes execution-model primitives that control how Triton or Gluon programs map onto hardware. It indexes 6 documented entries, including program_id, num_programs, num_warps, num_ctas, warp_specialize, and barrier.
**CN:** 本节描述控制 Triton/Gluon 程序如何映射到硬件的执行模型原语。 它通过 autosummary 汇总了 6 个文档条目，例如 program_id、num_programs、num_warps、num_ctas、warp_specialize、barrier。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、program_id、num_programs。

### Creation Ops
**EN:** This section gathers APIs for constructing tensors, ranges, or shared-memory backed values. It indexes 8 documented entries, including allocate_shared_memory, arange, cast, full, full_like, and zeros.
**CN:** 本节汇总用于构造张量、索引范围或共享内存值的 API。 它通过 autosummary 汇总了 8 个文档条目，例如 allocate_shared_memory、arange、cast、full、full_like、zeros。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、allocate_shared_memory、arange。

### Layout Ops
**EN:** This section groups APIs that manipulate layout metadata and memory-access organization. It indexes 4 documented entries, including bank_conflicts, convert_layout, set_auto_layout, and to_linear_layout.
**CN:** 本节汇总操作布局元数据和内存访问组织的 API。 它通过 autosummary 汇总了 4 个文档条目，例如 bank_conflicts、convert_layout、set_auto_layout、to_linear_layout。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、bank_conflicts、convert_layout。

### Shape Manipulation Ops
**EN:** This section covers reshaping, broadcasting, splitting, and other tensor-structure transforms. It indexes 8 documented entries, including broadcast, expand_dims, join, map_elementwise, permute, and ravel.
**CN:** 本节涵盖 reshape、broadcast、split 等张量结构变换。 它通过 autosummary 汇总了 8 个文档条目，例如 broadcast、expand_dims、join、map_elementwise、permute、ravel。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、broadcast、expand_dims。

### Memory Ops
**EN:** This section lists the core memory read/write operations used inside kernels. It indexes 2 documented entries, including load and store.
**CN:** 本节列出内核内部使用的核心读写操作。 它通过 autosummary 汇总了 2 个文档条目，例如 load、store。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、load、store。

### Atomic Ops
**EN:** This section collects synchronization-sensitive atomic read-modify-write primitives. It indexes 8 documented entries, including atomic_add, atomic_and, atomic_cas, atomic_max, atomic_min, and atomic_or.
**CN:** 本节汇总与同步密切相关的原子读改写原语。 它通过 autosummary 汇总了 8 个文档条目，例如 atomic_add、atomic_and、atomic_cas、atomic_max、atomic_min、atomic_or。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、atomic_add、atomic_and。

### Linear Algebra Ops
**EN:** This section groups the matrix and dot-product style operators used for kernel math. It indexes 1 documented entries, including dot_fma.
**CN:** 本节汇总用于内核数学计算的矩阵与点积类操作。 它通过 autosummary 汇总了 1 个文档条目，例如 dot_fma。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、dot_fma。

### Indexing Ops
**EN:** This section covers data selection, masking, and coordinate-based access helpers. It indexes 2 documented entries, including gather and where.
**CN:** 本节涵盖数据选择、掩码和基于坐标的访问辅助函数。 它通过 autosummary 汇总了 2 个文档条目，例如 gather、where。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、gather、where。

### Math Ops
**EN:** This section lists scalar and tensor math primitives ranging from elementary functions to utility arithmetic. It indexes 25 documented entries, including abs, add, cdiv, ceil, clamp, and cos.
**CN:** 本节列出从基础函数到实用算术的标量/张量数学原语。 它通过 autosummary 汇总了 25 个文档条目，例如 abs、add、cdiv、ceil、clamp、cos。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、abs、add。

### Reduction Ops
**EN:** This section groups APIs that combine values across dimensions or lanes. It indexes 6 documented entries, including reduce, reduce_or, sum, max, min, and xor_sum.
**CN:** 本节汇总跨维度或跨 lane 聚合数值的 API。 它通过 autosummary 汇总了 6 个文档条目，例如 reduce、reduce_or、sum、max、min、xor_sum。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、reduce、reduce_or。

### Scan Ops
**EN:** This section presents prefix-style and histogram-like aggregation utilities. It indexes 2 documented entries, including associative_scan and histogram.
**CN:** 本节介绍前缀扫描和直方图等聚合工具。 它通过 autosummary 汇总了 2 个文档条目，例如 associative_scan、histogram。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、associative_scan、histogram。

### Layout Classes
**EN:** This section lists layout classes that capture how data is distributed or stored. It indexes 11 documented entries, including AutoLayout, BlockedLayout, CoalescedLayout, DotOperandLayout, DistributedLinearLayout, and NVMMADistributedLayout.
**CN:** 本节列出描述数据如何分布或存储的布局类。 它通过 autosummary 汇总了 11 个文档条目，例如 AutoLayout、BlockedLayout、CoalescedLayout、DotOperandLayout、DistributedLinearLayout、NVMMADistributedLayout。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、AutoLayout、BlockedLayout。

### Iterators
**EN:** This section documents iterator-like helpers for static or dynamic loop construction. It indexes 1 documented entries, including static_range.
**CN:** 本节记录用于静态或动态循环构造的迭代器式辅助函数。 它通过 autosummary 汇总了 1 个文档条目，例如 static_range。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、static_range。

### Inline Assembly
**EN:** This section exposes inline assembly hooks for the rare cases where higher-level primitives are insufficient. It indexes 1 documented entries, including inline_asm_elementwise.
**CN:** 本节暴露内联汇编接口，用于高层原语不足以表达需求的少数场景。 它通过 autosummary 汇总了 1 个文档条目，例如 inline_asm_elementwise。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、inline_asm_elementwise。

### Compiler Hints and Debugging
**EN:** This section collects compile-time assertions, layout hints, and debug printing utilities. It indexes 8 documented entries, including assume, max_constancy, max_contiguous, multiple_of, static_assert, and static_print.
**CN:** 本节汇总编译期断言、布局提示和调试打印工具。 它通过 autosummary 汇总了 8 个文档条目，例如 assume、max_constancy、max_contiguous、multiple_of、static_assert、static_print。 其中反复出现的技术关键词包括 autosummary、toctree、generated、nosignatures、assume、max_constancy。

## Key Concepts / 关键概念
- **EN:** Language API  **CN:** 语言 API
- **EN:** Types  **CN:** 类型
- **EN:** Programming Model  **CN:** 编程模型
- **EN:** Creation Ops  **CN:** 创建操作
- **EN:** Layout Ops  **CN:** 布局操作
- **EN:** Shape Manipulation Ops  **CN:** 形状变换操作

## Related Files / 相关文件
- `/root/xw/triton/docs/gluon/api/index.rst`
- `/root/xw/triton/docs/gluon/index.rst`
