# mma_sm90_gmma_sparse_ext.hpp — Code Analysis / 代码分析
**Source / 源文件**: `include/cute/arch/mma_sm90_gmma_sparse_ext.hpp`
**Purpose / 用途**: Defines sparse SM90 GMMA instruction wrappers and CuTe adapters for tiled MMA. / 定义 稀疏 SM90 GMMA 指令包装及其面向分块 MMA 的 CuTe 适配层。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-482 / 第 1-482 行
**EN**: Sets up the header entry point and pulls in `cute/config.hpp`, `cutlass/arch/synclog.hpp` for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 建立头文件入口，并引入 `cute/config.hpp`、`cutlass/arch/synclog.hpp`，以支撑 Tensor Core / MMA 原子与分块 GEMM 编排。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 483-980 / 第 483-980 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 981-1460 / 第 981-1460 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 1461-1969 / 第 1461-1969 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 1970-2450 / 第 1970-2450 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 2451-2931 / 第 2451-2931 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 2932-3444 / 第 2932-3444 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 3445-3932 / 第 3445-3932 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 3933-4448 / 第 3933-4448 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 4449-4955 / 第 4449-4955 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 4956-5437 / 第 4956-5437 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 5438-5920 / 第 5438-5920 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 5921-6400 / 第 5921-6400 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 6401-6912 / 第 6401-6912 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 6913-7392 / 第 6913-7392 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 7393-7884 / 第 7393-7884 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 7885-8372 / 第 7885-8372 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 8373-8888 / 第 8373-8888 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 8889-9395 / 第 8889-9395 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 9396-9877 / 第 9396-9877 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 9878-10360 / 第 9878-10360 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 10361-10840 / 第 10361-10840 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 10841-11352 / 第 10841-11352 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 11353-11832 / 第 11353-11832 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 11833-12324 / 第 11833-12324 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。

### Lines 12325-12804 / 第 12325-12804 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 12805-13284 / 第 12805-13284 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 13285-13767 / 第 13285-13767 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 13768-14258 / 第 13768-14258 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 14259-14741 / 第 14259-14741 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 14742-15222 / 第 14742-15222 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 15223-15706 / 第 15223-15706 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 15707-16233 / 第 15707-16233 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 16234-16714 / 第 16234-16714 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 16715-17195 / 第 16715-17195 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 17196-17700 / 第 17196-17700 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 17701-18205 / 第 17701-18205 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 18206-18685 / 第 18206-18685 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 18686-19182 / 第 18686-19182 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 19183-19702 / 第 19183-19702 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 19703-20205 / 第 19703-20205 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 20206-20724 / 第 20206-20724 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 20725-21209 / 第 20725-21209 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 21210-21711 / 第 21210-21711 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 21712-22199 / 第 21712-22199 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 22200-22680 / 第 22200-22680 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 22681-23160 / 第 22681-23160 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 23161-23657 / 第 23161-23657 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 23658-24176 / 第 23658-24176 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 24177-24661 / 第 24177-24661 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 24662-25163 / 第 24662-25163 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 25164-25651 / 第 25164-25651 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 25652-26132 / 第 25652-26132 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 26133-26612 / 第 26133-26612 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 26613-27109 / 第 26613-27109 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 27110-27628 / 第 27110-27628 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 27629-28113 / 第 27629-28113 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 28114-28615 / 第 28114-28615 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 28616-29103 / 第 28616-29103 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 29104-29584 / 第 29104-29584 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 29585-30064 / 第 29585-30064 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 30065-30554 / 第 30065-30554 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 30555-31046 / 第 30555-31046 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 31047-31526 / 第 31047-31526 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 31527-32040 / 第 31527-32040 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 32041-32521 / 第 32041-32521 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 32522-33026 / 第 32522-33026 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 33027-33512 / 第 33027-33512 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 33513-33992 / 第 33513-33992 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 33993-34498 / 第 33993-34498 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 34499-34988 / 第 34499-34988 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 34989-35468 / 第 34989-35468 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 35469-35948 / 第 35469-35948 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 35949-36433 / 第 35949-36433 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 36434-36919 / 第 36434-36919 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 36920-37399 / 第 36920-37399 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 37400-37912 / 第 37400-37912 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 37913-38396 / 第 37913-38396 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 38397-38900 / 第 38397-38900 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 38901-39404 / 第 38901-39404 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 39405-39888 / 第 39405-39888 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 39889-40400 / 第 39889-40400 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 40401-40886 / 第 40401-40886 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 40887-41378 / 第 40887-41378 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 41379-41859 / 第 41379-41859 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 41860-42348 / 第 41860-42348 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 42349-42854 / 第 42349-42854 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 42855-43335 / 第 42855-43335 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 43336-43822 / 第 43336-43822 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 43823-44302 / 第 43823-44302 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 44303-44782 / 第 44303-44782 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 44783-45298 / 第 44783-45298 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 45299-45783 / 第 45299-45783 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 45784-46265 / 第 45784-46265 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 46266-46745 / 第 46266-46745 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 46746-47257 / 第 46746-47257 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 47258-47744 / 第 47258-47744 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 47745-48228 / 第 47745-48228 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 48229-48711 / 第 48229-48711 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 48712-49191 / 第 48712-49191 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 49192-49672 / 第 49192-49672 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 49673-50155 / 第 49673-50155 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 50156-50653 / 第 50156-50653 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 50654-51159 / 第 50654-51159 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 51160-51656 / 第 51160-51656 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 51657-52155 / 第 51657-52155 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 52156-52642 / 第 52156-52642 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 52643-53130 / 第 52643-53130 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 53131-53624 / 第 53131-53624 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 53625-54107 / 第 53625-54107 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 54108-54597 / 第 54108-54597 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 54598-55085 / 第 54598-55085 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 55086-55565 / 第 55086-55565 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 55566-56048 / 第 55566-56048 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 56049-56542 / 第 56049-56542 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 56543-57022 / 第 56543-57022 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 57023-57512 / 第 57023-57512 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 57513-57992 / 第 57513-57992 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 57993-58472 / 第 57993-58472 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 58473-58957 / 第 58473-58957 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 58958-59443 / 第 58958-59443 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 59444-59923 / 第 59444-59923 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 59924-60436 / 第 59924-60436 行
**EN**: Continues the large catalog of hardware specializations for Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 继续展开面向 Tensor Core / MMA 原子与分块 GEMM 编排 的大型硬件特化目录。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 60437-60445 / 第 60437-60445 行
**EN**: Adds compile-time guards and selection logic around Tensor Core / MMA atoms and tiled GEMM orchestration. Each specialization pins down instruction shape, element type, register packing, and accumulator conventions. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 围绕 Tensor Core / MMA 原子与分块 GEMM 编排 加入编译期保护与选择逻辑。 每个特化都会固定指令形状、元素类型、寄存器打包方式以及累加器约定。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

---
## Key Concepts / 关键概念
- MMA atoms separate instruction semantics from higher-level tiling and scheduling. / MMA atom 把指令语义与更高层的分块和调度解耦。
- Trait specializations encode opcode shape, element type, register layout, and accumulator conventions. / traits 特化会编码 opcode 形状、元素类型、寄存器布局和累加器约定。
- CuTe builds tiled GEMM from reusable MMA pieces rather than hard-coding one kernel shape. / CuTe 用可复用的 MMA 构件搭建分块 GEMM，而不是写死单一内核形状。

## Dependencies / 依赖项
- `cute/config.hpp`
- `cutlass/arch/synclog.hpp`
