# copy_atom.hpp — Code Analysis / 代码分析
**Source / 源文件**: `include/cute/atom/copy_atom.hpp`
**Purpose / 用途**: Defines the `Copy_Atom` abstraction that lifts low-level copy traits into reusable tiled copy units. / 定义 `Copy_Atom` 抽象，把底层 copy traits 提升为可复用的分块拷贝单元。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-60 / 第 1-60 行
**EN**: Sets up the header entry point and pulls in `cute/config.hpp`, `cute/tensor_impl.hpp`, `cute/util/type_traits.hpp`, `cute/container/tuple.hpp` for copy atoms, tiled movement, and asynchronous transfer pipelines. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 建立头文件入口，并引入 `cute/config.hpp`、`cute/tensor_impl.hpp`、`cute/util/type_traits.hpp`、`cute/container/tuple.hpp`，以支撑 copy atom、分块搬运与异步传输流水线。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 61-126 / 第 61-126 行
**EN**: Defines `DEngine` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `DEngine` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 127-186 / 第 127-186 行
**EN**: Defines `SEngine`, `DEngine`, `ThrCopy`, and `LayoutCopy_TV` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `SEngine`、`DEngine`、`ThrCopy`、`LayoutCopy_TV` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 187-249 / 第 187-249 行
**EN**: Defines `ShapeTiler_MN` and `TiledCopy` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `ShapeTiler_MN`、`TiledCopy` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 250-309 / 第 250-309 行
**EN**: Implements helpers like `tile2thrfrg` and `retile` for copy atoms, tiled movement, and asynchronous transfer pipelines. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 实现 `tile2thrfrg`、`retile` 等辅助函数，用于 copy atom、分块搬运与异步传输流水线。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 310-374 / 第 310-374 行
**EN**: Defines `ThrCopy` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `ThrCopy` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 375-438 / 第 375-438 行
**EN**: Defines `LayoutCopy_TV` and `Tiler` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `LayoutCopy_TV`、`Tiler` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 439-507 / 第 439-507 行
**EN**: Defines `ThrLayout` and `ValLayout` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `ThrLayout`、`ValLayout` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 508-569 / 第 508-569 行
**EN**: Adds compile-time guards and selection logic around copy atoms, tiled movement, and asynchronous transfer pipelines. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 围绕 copy atom、分块搬运与异步传输流水线 加入编译期保护与选择逻辑。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 570-631 / 第 570-631 行
**EN**: Defines `TiledCopy` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 定义 `TiledCopy` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 632-691 / 第 632-691 行
**EN**: Sets up the header entry point and pulls in `cute/atom/copy_traits_sm50.hpp`, `cute/atom/copy_traits_sm75.hpp`, `cute/atom/copy_traits_sm80.hpp`, `cute/atom/copy_traits_sm90.hpp` for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 建立头文件入口，并引入 `cute/atom/copy_traits_sm50.hpp`、`cute/atom/copy_traits_sm75.hpp`、`cute/atom/copy_traits_sm80.hpp`、`cute/atom/copy_traits_sm90.hpp`，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

---
## Key Concepts / 关键概念
- Copy atoms describe how data moves between memory spaces at the instruction level. / copy atom 描述了数据如何在不同内存空间之间以指令粒度移动。
- Traits capture alignment, vector width, source/destination layout, and pipeline behavior. / traits 会捕获对齐、向量宽度、源/目标布局以及流水线行为。
- Higher-level tiled copies are assembled by composing these low-level movement rules. / 更高层的 tiled copy 是通过组合这些底层搬运规则构造出来的。

## Dependencies / 依赖项
- `cute/config.hpp`
- `cute/tensor_impl.hpp`
- `cute/util/type_traits.hpp`
- `cute/container/tuple.hpp`
- `cute/numeric/integral_constant.hpp`
- `cute/atom/copy_traits.hpp`
- `cute/atom/mma_atom.hpp`
- `cute/atom/copy_traits_sm50.hpp`
- `cute/atom/copy_traits_sm75.hpp`
- `cute/atom/copy_traits_sm80.hpp`
- `cute/atom/copy_traits_sm90.hpp`
- `cute/atom/copy_traits_sm100.hpp`
- `cute/atom/copy_traits_sm90_tma.hpp`
- `cute/atom/copy_traits_sm100_tma.hpp`
