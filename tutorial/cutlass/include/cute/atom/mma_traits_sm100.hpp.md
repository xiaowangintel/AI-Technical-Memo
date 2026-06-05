# mma_traits_sm100.hpp — Code Analysis / 代码分析
**Source / 源文件**: `include/cute/atom/mma_traits_sm100.hpp`
**Purpose / 用途**: Defines trait specializations that map SM100 MMA instructions into CuTe MMA atoms. / 定义将 SM100 MMA 指令映射为 CuTe MMA atom 的 traits 特化。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-140 / 第 1-140 行
**EN**: Sets up the header entry point and pulls in `cute/config.hpp`, `cute/pointer_sparse.hpp`, `cute/tensor_impl.hpp`, `cute/arch/mma_sm100.hpp` for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 建立头文件入口，并引入 `cute/config.hpp`、`cute/pointer_sparse.hpp`、`cute/tensor_impl.hpp`、`cute/arch/mma_sm100.hpp`，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 141-281 / 第 141-281 行
**EN**: Introduces aliases such as `value_type` to make Tensor Core / MMA atoms and tiled GEMM orchestration easier to express. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 引入 `value_type` 等别名，使 Tensor Core / MMA 原子与分块 GEMM 编排 更易表达。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 282-421 / 第 282-421 行
**EN**: Defines `DescriptorIterator`, `smem_desc`, `sparse_smem_desc`, and `MakeTensor` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `DescriptorIterator`、`smem_desc`、`sparse_smem_desc`、`MakeTensor` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 422-576 / 第 422-576 行
**EN**: Defines `tmem_frg_base` and `tmem_frg` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `tmem_frg_base`、`tmem_frg` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 577-686 / 第 577-686 行
**EN**: Defines `tmem_e_frg` and `tmem_e_frg_ws` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `tmem_e_frg`、`tmem_e_frg_ws` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 687-833 / 第 687-833 行
**EN**: Defines `tmem_sf_frg` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `tmem_sf_frg` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 834-973 / 第 834-973 行
**EN**: Defines `tmem_frg_ws` and `MakeTensor` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `tmem_frg_ws`、`MakeTensor` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 974-1115 / 第 974-1115 行
**EN**: Defines `MakeTensor`, `MMA_Traits`, `TA`, and `TB` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `MakeTensor`、`MMA_Traits`、`TA`、`TB` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 1116-1266 / 第 1116-1266 行
**EN**: Defines `TA`, `TB`, `TC`, and `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `TA`、`TB`、`TC`、`MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 1267-1406 / 第 1267-1406 行
**EN**: Defines `MMA_Traits`, `TA`, `TB`, and `TC` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `MMA_Traits`、`TA`、`TB`、`TC` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 1407-1546 / 第 1407-1546 行
**EN**: Defines `TC`, `MMA_Traits`, `TA`, and `TB` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `TC`、`MMA_Traits`、`TA`、`TB` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 1547-1693 / 第 1547-1693 行
**EN**: Defines `TA`, `TB`, `TC`, and `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `TA`、`TB`、`TC`、`MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 1694-1833 / 第 1694-1833 行
**EN**: Defines `MMA_Traits`, `TA`, `TB`, and `TC` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `MMA_Traits`、`TA`、`TB`、`TC` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 1834-1974 / 第 1834-1974 行
**EN**: Defines `TA`, `TB`, `TC`, and `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `TA`、`TB`、`TC`、`MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 1975-2114 / 第 1975-2114 行
**EN**: Defines `TA`, `TB`, `TC`, and `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `TA`、`TB`、`TC`、`MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 2115-2254 / 第 2115-2254 行
**EN**: Defines `TA`, `TB`, `TC`, and `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `TA`、`TB`、`TC`、`MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 2255-2394 / 第 2255-2394 行
**EN**: Defines `TA`, `TB`, `TC`, and `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `TA`、`TB`、`TC`、`MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 2395-2534 / 第 2395-2534 行
**EN**: Defines `TA`, `TB`, `TC`, and `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `TA`、`TB`、`TC`、`MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 2535-2678 / 第 2535-2678 行
**EN**: Defines `MMA_Traits`, `TA`, `TB`, and `TC` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `MMA_Traits`、`TA`、`TB`、`TC` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 2679-2819 / 第 2679-2819 行
**EN**: Defines `TA`, `TB`, `TC`, and `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `TA`、`TB`、`TC`、`MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 2820-2959 / 第 2820-2959 行
**EN**: Defines `MMA_Traits`, `TA`, `TB`, and `TC` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `MMA_Traits`、`TA`、`TB`、`TC` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 2960-3109 / 第 2960-3109 行
**EN**: Defines `MMA_Traits`, `TA`, `TB`, and `TC` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `MMA_Traits`、`TA`、`TB`、`TC` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 3110-3249 / 第 3110-3249 行
**EN**: Defines `MMA_Traits`, `TA`, `TB`, and `TC` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `MMA_Traits`、`TA`、`TB`、`TC` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 3250-3392 / 第 3250-3392 行
**EN**: Defines `TA`, `TB`, `TC`, and `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `TA`、`TB`、`TC`、`MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 3393-3535 / 第 3393-3535 行
**EN**: Defines `MMA_Traits`, `TA`, `TB`, and `TC` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `MMA_Traits`、`TA`、`TB`、`TC` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 3536-3675 / 第 3536-3675 行
**EN**: Defines `TA`, `TB`, `TC`, and `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `TA`、`TB`、`TC`、`MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 3676-3818 / 第 3676-3818 行
**EN**: Defines `TA`, `TB`, `TC`, and `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `TA`、`TB`、`TC`、`MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 3819-3958 / 第 3819-3958 行
**EN**: Defines `MMA_Traits`, `TA`, `TB`, and `TC` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `MMA_Traits`、`TA`、`TB`、`TC` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 3959-4099 / 第 3959-4099 行
**EN**: Defines `MMA_Traits`, `TA`, `TB`, and `TC` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `MMA_Traits`、`TA`、`TB`、`TC` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 4100-4240 / 第 4100-4240 行
**EN**: Defines `TA`, `TB`, `TC`, and `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `TA`、`TB`、`TC`、`MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 4241-4380 / 第 4241-4380 行
**EN**: Defines `MMA_Traits`, `TA`, `TB`, and `TC` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `MMA_Traits`、`TA`、`TB`、`TC` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 4381-4521 / 第 4381-4521 行
**EN**: Defines `TA`, `TB`, `TC`, and `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `TA`、`TB`、`TC`、`MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 4522-4661 / 第 4522-4661 行
**EN**: Defines `MMA_Traits`, `TD`, `TA`, and `TB` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 定义 `MMA_Traits`、`TD`、`TA`、`TB` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 4662-4733 / 第 4662-4733 行
**EN**: Defines `MMA_Traits` and related types for Tensor Core / MMA atoms and tiled GEMM orchestration. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `MMA_Traits` 等相关类型，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

---
## Key Concepts / 关键概念
- MMA atoms separate instruction semantics from higher-level tiling and scheduling. / MMA atom 把指令语义与更高层的分块和调度解耦。
- Trait specializations encode opcode shape, element type, register layout, and accumulator conventions. / traits 特化会编码 opcode 形状、元素类型、寄存器布局和累加器约定。
- CuTe builds tiled GEMM from reusable MMA pieces rather than hard-coding one kernel shape. / CuTe 用可复用的 MMA 构件搭建分块 GEMM，而不是写死单一内核形状。

## Dependencies / 依赖项
- `cute/config.hpp`
- `cute/pointer_sparse.hpp`
- `cute/tensor_impl.hpp`
- `cute/arch/mma_sm100.hpp`
- `cute/arch/mma_sm100_desc.hpp`
- `cute/arch/mma_sm100_umma.hpp`
- `cute/arch/tmem_allocator_sm100.hpp`
- `cute/atom/mma_traits.hpp`
- `cute/atom/mma_traits_sm90_gmma.hpp`
- `cute/atom/mma_traits_sm90_gmma_sparse.hpp`
- `cute/atom/copy_traits_sm100.hpp`
- `cute/numeric/numeric_types.hpp`
