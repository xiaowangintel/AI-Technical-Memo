# mma_traits_sm90_gmma_sparse.hpp — Code Analysis / 代码分析
**Source / 源文件**: `include/cute/atom/mma_traits_sm90_gmma_sparse.hpp`
**Purpose / 用途**: Defines trait specializations that map SM90 MMA instructions into CuTe MMA atoms. / 定义将 SM90 MMA 指令映射为 CuTe MMA atom 的 traits 特化。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-141 / 第 1-141 行
**EN**: Sets up the header entry point and pulls in `cute/pointer_sparse.hpp`, `cute/swizzle.hpp`, `cute/tensor_impl.hpp`, `cute/arch/mma_sm90_desc.hpp` for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 建立头文件入口，并引入 `cute/pointer_sparse.hpp`、`cute/swizzle.hpp`、`cute/tensor_impl.hpp`、`cute/arch/mma_sm90_desc.hpp`，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 142-282 / 第 142-282 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 283-423 / 第 283-423 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 424-564 / 第 424-564 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 565-705 / 第 565-705 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 706-846 / 第 706-846 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 847-987 / 第 847-987 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 988-1128 / 第 988-1128 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1129-1269 / 第 1129-1269 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1270-1410 / 第 1270-1410 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1411-1551 / 第 1411-1551 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1552-1692 / 第 1552-1692 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1693-1832 / 第 1693-1832 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1833-1972 / 第 1833-1972 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1973-2112 / 第 1973-2112 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2113-2252 / 第 2113-2252 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2253-2392 / 第 2253-2392 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2393-2532 / 第 2393-2532 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2533-2673 / 第 2533-2673 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2674-2814 / 第 2674-2814 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2815-2954 / 第 2815-2954 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2955-3094 / 第 2955-3094 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 3095-3234 / 第 3095-3234 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 3235-3374 / 第 3235-3374 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 3375-3514 / 第 3375-3514 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 3515-3654 / 第 3515-3654 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 3655-3794 / 第 3655-3794 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 3795-3934 / 第 3795-3934 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 3935-4074 / 第 3935-4074 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 4075-4214 / 第 4075-4214 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 4215-4355 / 第 4215-4355 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 4356-4495 / 第 4356-4495 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 4496-4636 / 第 4496-4636 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 4637-4776 / 第 4637-4776 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 4777-4916 / 第 4777-4916 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 4917-5057 / 第 4917-5057 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 5058-5198 / 第 5058-5198 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 5199-5339 / 第 5199-5339 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 5340-5480 / 第 5340-5480 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 5481-5621 / 第 5481-5621 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 5622-5762 / 第 5622-5762 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 5763-5903 / 第 5763-5903 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 5904-6044 / 第 5904-6044 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 6045-6185 / 第 6045-6185 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 6186-6326 / 第 6186-6326 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 6327-6467 / 第 6327-6467 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 6468-6608 / 第 6468-6608 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 6609-6749 / 第 6609-6749 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 6750-6890 / 第 6750-6890 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 6891-7031 / 第 6891-7031 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 7032-7172 / 第 7032-7172 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 7173-7313 / 第 7173-7313 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 7314-7454 / 第 7314-7454 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 7455-7595 / 第 7455-7595 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 7596-7735 / 第 7596-7735 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 7736-7738 / 第 7736-7738 行
**EN**: Sets up the header entry point and pulls in `mma_traits_sm90_gmma_sparse_ext.hpp` for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 建立头文件入口，并引入 `mma_traits_sm90_gmma_sparse_ext.hpp`，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

---
## Key Concepts / 关键概念
- MMA atoms separate instruction semantics from higher-level tiling and scheduling. / MMA atom 把指令语义与更高层的分块和调度解耦。
- Trait specializations encode opcode shape, element type, register layout, and accumulator conventions. / traits 特化会编码 opcode 形状、元素类型、寄存器布局和累加器约定。
- CuTe builds tiled GEMM from reusable MMA pieces rather than hard-coding one kernel shape. / CuTe 用可复用的 MMA 构件搭建分块 GEMM，而不是写死单一内核形状。

## Dependencies / 依赖项
- `cute/pointer_sparse.hpp`
- `cute/swizzle.hpp`
- `cute/tensor_impl.hpp`
- `cute/arch/mma_sm90_desc.hpp`
- `cute/arch/mma_sm90_gmma_sparse.hpp`
- `cute/atom/mma_traits_sm90_gmma.hpp`
- `cute/atom/mma_traits.hpp`
- `cute/layout_composed.hpp`
- `cute/numeric/integral_constant.hpp`
- `mma_traits_sm90_gmma_sparse_ext.hpp`
