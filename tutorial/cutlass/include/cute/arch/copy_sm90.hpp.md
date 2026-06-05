# copy_sm90.hpp — Code Analysis / 代码分析
**Source / 源文件**: `include/cute/arch/copy_sm90.hpp`
**Purpose / 用途**: Defines SM90 copy instructions and the CuTe wrappers that expose them. / 定义 SM90 拷贝指令以及暴露这些指令的 CuTe 包装。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-32 / 第 1-32 行
**EN**: Sets up the header entry point and pulls in the required dependencies for copy atoms, tiled movement, and asynchronous transfer pipelines.
**CN**: 建立头文件入口，并引入 所需依赖，以支撑 copy atom、分块搬运与异步传输流水线。

### Lines 33-59 / 第 33-59 行
**EN**: Sets up the header entry point and pulls in `cute/config.hpp`, `cute/arch/config.hpp`, `cute/arch/copy.hpp` for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 建立头文件入口，并引入 `cute/config.hpp`、`cute/arch/config.hpp`、`cute/arch/copy.hpp`，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 60-84 / 第 60-84 行
**EN**: Defines `SM90_U32x2_STSM_N` and `SM90_U32x4_STSM_N` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_U32x2_STSM_N`、`SM90_U32x4_STSM_N` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 85-108 / 第 85-108 行
**EN**: Defines `SM90_U16x2_STSM_T` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_U16x2_STSM_T` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 109-135 / 第 109-135 行
**EN**: Defines `SM90_U16x4_STSM_T` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_U16x4_STSM_T` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 136-159 / 第 136-159 行
**EN**: Defines `SM90_U16x8_STSM_T` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_U16x8_STSM_T` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 160-186 / 第 160-186 行
**EN**: Adds compile-time guards and selection logic around copy atoms, tiled movement, and asynchronous transfer pipelines. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 围绕 copy atom、分块搬运与异步传输流水线 加入编译期保护与选择逻辑。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 187-211 / 第 187-211 行
**EN**: Adds compile-time guards and selection logic around copy atoms, tiled movement, and asynchronous transfer pipelines. Compile-time checks prevent incompatible shapes, traits, or hardware modes from composing. Templates keep rank, tile shape, and instruction selection visible to the compiler.
**CN**: 围绕 copy atom、分块搬运与异步传输流水线 加入编译期保护与选择逻辑。 编译期检查会阻止不兼容的 shape、traits 或硬件模式被错误组合。 模板让 rank、tile 形状和指令选择都保持对编译器可见。

### Lines 212-219 / 第 212-219 行
**EN**: Sets up the header entry point and pulls in `cute/arch/copy_sm90_desc.hpp`, `cute/arch/copy_sm90_tma.hpp` for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details.
**CN**: 建立头文件入口，并引入 `cute/arch/copy_sm90_desc.hpp`、`cute/arch/copy_sm90_tma.hpp`，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。

---
## Key Concepts / 关键概念
- Copy atoms describe how data moves between memory spaces at the instruction level. / copy atom 描述了数据如何在不同内存空间之间以指令粒度移动。
- Traits capture alignment, vector width, source/destination layout, and pipeline behavior. / traits 会捕获对齐、向量宽度、源/目标布局以及流水线行为。
- Higher-level tiled copies are assembled by composing these low-level movement rules. / 更高层的 tiled copy 是通过组合这些底层搬运规则构造出来的。

## Dependencies / 依赖项
- `cute/config.hpp`
- `cute/arch/config.hpp`
- `cute/arch/copy.hpp`
- `cute/arch/copy_sm90_desc.hpp`
- `cute/arch/copy_sm90_tma.hpp`
