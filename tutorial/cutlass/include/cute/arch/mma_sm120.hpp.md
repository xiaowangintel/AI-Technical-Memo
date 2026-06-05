# mma_sm120.hpp — Code Analysis / 代码分析
**Source / 源文件**: `include/cute/arch/mma_sm120.hpp`
**Purpose / 用途**: Defines SM120 MMA instruction wrappers and CuTe hardware bindings. / 定义 SM120 MMA 指令包装及其 CuTe 硬件绑定。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-90 / 第 1-90 行
**EN**: Sets up the header entry point and pulls in `cute/arch/config.hpp`, `cute/arch/mma.hpp`, `cute/numeric/numeric_types.hpp`, `cutlass/detail/dependent_false.hpp` for Tensor Core / MMA atoms and tiled GEMM orchestration. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 建立头文件入口，并引入 `cute/arch/config.hpp`、`cute/arch/mma.hpp`、`cute/numeric/numeric_types.hpp`、`cutlass/detail/dependent_false.hpp`，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 91-182 / 第 91-182 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 183-283 / 第 183-283 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 284-385 / 第 284-385 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 386-476 / 第 386-476 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 477-566 / 第 477-566 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 567-661 / 第 567-661 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 662-752 / 第 662-752 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 753-843 / 第 753-843 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 844-936 / 第 844-936 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 937-1038 / 第 937-1038 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1039-1140 / 第 1039-1140 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1141-1242 / 第 1141-1242 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1243-1344 / 第 1243-1344 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1345-1434 / 第 1345-1434 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1435-1525 / 第 1435-1525 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1526-1617 / 第 1526-1617 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1618-1719 / 第 1618-1719 行
**EN**: Defines `SM120_16x8x32_TN` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1720-1824 / 第 1720-1824 行
**EN**: Defines `SM120_16x8x32_TN`, `SM120_16x8x32_TN_VS`, and `SM120_16x8x64_TN_VS` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `SM120_16x8x32_TN`、`SM120_16x8x32_TN_VS`、`SM120_16x8x64_TN_VS` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 1825-1930 / 第 1825-1930 行
**EN**: Defines `SM120_16x8x32_TN_VS` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN_VS` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 1931-2036 / 第 1931-2036 行
**EN**: Defines `SM120_16x8x32_TN_VS` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN_VS` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2037-2142 / 第 2037-2142 行
**EN**: Defines `SM120_16x8x32_TN_VS` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN_VS` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2143-2248 / 第 2143-2248 行
**EN**: Defines `SM120_16x8x32_TN_VS` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN_VS` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2249-2354 / 第 2249-2354 行
**EN**: Defines `SM120_16x8x32_TN_VS` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN_VS` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2355-2460 / 第 2355-2460 行
**EN**: Defines `SM120_16x8x32_TN_VS` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN_VS` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2461-2566 / 第 2461-2566 行
**EN**: Defines `SM120_16x8x32_TN_VS` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN_VS` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2567-2672 / 第 2567-2672 行
**EN**: Defines `SM120_16x8x32_TN_VS` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN_VS` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2673-2778 / 第 2673-2778 行
**EN**: Defines `SM120_16x8x32_TN_VS` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN_VS` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2779-2884 / 第 2779-2884 行
**EN**: Defines `SM120_16x8x32_TN_VS` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN_VS` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2885-2990 / 第 2885-2990 行
**EN**: Defines `SM120_16x8x32_TN_VS` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN_VS` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 2991-3096 / 第 2991-3096 行
**EN**: Defines `SM120_16x8x32_TN_VS` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x32_TN_VS` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 3097-3186 / 第 3097-3186 行
**EN**: Defines `SM120_16x8x64_TN_VS` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Preprocessor guards select the correct path for architecture, compiler, or feature availability. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SM120_16x8x64_TN_VS` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 3187-3277 / 第 3187-3277 行
**EN**: Defines `SM120_16x8x64_TN_VS`, `ElementA`, `ElementB`, and `ElementC` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM120_16x8x64_TN_VS`、`ElementA`、`ElementB`、`ElementC` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 3278-3278 / 第 3278-3278 行
**EN**: Continues the implementation details for Tensor Core / MMA atoms and tiled GEMM orchestration.
**CN**: 继续展开 Tensor Core / MMA 原子与分块 GEMM 编排 的实现细节。

---
## Key Concepts / 关键概念
- MMA atoms separate instruction semantics from higher-level tiling and scheduling. / MMA atom 把指令语义与更高层的分块和调度解耦。
- Trait specializations encode opcode shape, element type, register layout, and accumulator conventions. / traits 特化会编码 opcode 形状、元素类型、寄存器布局和累加器约定。
- CuTe builds tiled GEMM from reusable MMA pieces rather than hard-coding one kernel shape. / CuTe 用可复用的 MMA 构件搭建分块 GEMM，而不是写死单一内核形状。

## Dependencies / 依赖项
- `cute/arch/config.hpp`
- `cute/arch/mma.hpp`
- `cute/numeric/numeric_types.hpp`
- `cutlass/detail/dependent_false.hpp`
