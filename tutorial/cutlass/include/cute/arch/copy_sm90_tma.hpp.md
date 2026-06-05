# copy_sm90_tma.hpp — Code Analysis / 代码分析
**Source / 源文件**: `include/cute/arch/copy_sm90_tma.hpp`
**Purpose / 用途**: Defines SM90 TMA-based copy instructions and CuTe wrappers. / 定义 SM90 基于 TMA 的拷贝指令及其 CuTe 包装。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-74 / 第 1-74 行
**EN**: Sets up the header entry point and pulls in `cute/config.hpp`, `cute/arch/config.hpp`, `cute/arch/copy.hpp`, `cute/arch/copy_sm90.hpp` for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 建立头文件入口，并引入 `cute/config.hpp`、`cute/arch/config.hpp`、`cute/arch/copy.hpp`、`cute/arch/copy_sm90.hpp`，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 75-136 / 第 75-136 行
**EN**: Defines `PREFETCH` and `SM90_TMA_LOAD_2D` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `PREFETCH`、`SM90_TMA_LOAD_2D` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 137-198 / 第 137-198 行
**EN**: Defines `PREFETCH` and `SM90_TMA_LOAD_3D` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `PREFETCH`、`SM90_TMA_LOAD_3D` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 199-265 / 第 199-265 行
**EN**: Defines `SM90_TMA_LOAD_4D` and `PREFETCH` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_LOAD_4D`、`PREFETCH` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 266-326 / 第 266-326 行
**EN**: Defines `SM90_TMA_LOAD_5D` and `PREFETCH` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_LOAD_5D`、`PREFETCH` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 327-390 / 第 327-390 行
**EN**: Defines `SM90_TMA_LOAD` and `PREFETCH` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details.
**CN**: 定义 `SM90_TMA_LOAD`、`PREFETCH` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。

### Lines 391-454 / 第 391-454 行
**EN**: Defines `SM90_TMA_LOAD_IM2COL_3D` and `PREFETCH` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_LOAD_IM2COL_3D`、`PREFETCH` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 455-529 / 第 455-529 行
**EN**: Defines `SM90_TMA_LOAD_IM2COL_4D`, `PREFETCH`, and `SM90_TMA_LOAD_IM2COL_5D` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_LOAD_IM2COL_4D`、`PREFETCH`、`SM90_TMA_LOAD_IM2COL_5D` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 530-589 / 第 530-589 行
**EN**: Defines `PREFETCH` and `SM90_TMA_LOAD_IM2COL` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `PREFETCH`、`SM90_TMA_LOAD_IM2COL` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 590-650 / 第 590-650 行
**EN**: Defines `PREFETCH` and `SM90_TMA_LOAD_MULTICAST_1D` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `PREFETCH`、`SM90_TMA_LOAD_MULTICAST_1D` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 651-712 / 第 651-712 行
**EN**: Defines `SM90_TMA_LOAD_MULTICAST_2D` and `SM90_TMA_LOAD_MULTICAST_3D` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_LOAD_MULTICAST_2D`、`SM90_TMA_LOAD_MULTICAST_3D` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 713-772 / 第 713-772 行
**EN**: Defines `SM90_TMA_LOAD_MULTICAST_4D`, `SM90_TMA_LOAD_MULTICAST_5D`, and `SM90_TMA_LOAD_MULTICAST` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_LOAD_MULTICAST_4D`、`SM90_TMA_LOAD_MULTICAST_5D`、`SM90_TMA_LOAD_MULTICAST` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 773-843 / 第 773-843 行
**EN**: Defines `SM90_TMA_LOAD_IM2COL_MULTICAST_3D` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_LOAD_IM2COL_MULTICAST_3D` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 844-907 / 第 844-907 行
**EN**: Defines `SM90_TMA_LOAD_IM2COL_MULTICAST_4D` and `SM90_TMA_LOAD_IM2COL_MULTICAST_5D` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_LOAD_IM2COL_MULTICAST_4D`、`SM90_TMA_LOAD_IM2COL_MULTICAST_5D` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 908-975 / 第 908-975 行
**EN**: Defines `SM90_TMA_LOAD_IM2COL_MULTICAST` and `SM90_TMA_STORE_1D` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_LOAD_IM2COL_MULTICAST`、`SM90_TMA_STORE_1D` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 976-1044 / 第 976-1044 行
**EN**: Defines `SM90_TMA_STORE_2D`, `SM90_TMA_STORE_3D`, and `SM90_TMA_STORE_4D` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_STORE_2D`、`SM90_TMA_STORE_3D`、`SM90_TMA_STORE_4D` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 1045-1110 / 第 1045-1110 行
**EN**: Defines `SM90_TMA_STORE_5D` and `SM90_TMA_STORE` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_STORE_5D`、`SM90_TMA_STORE` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 1111-1182 / 第 1111-1182 行
**EN**: Defines `SM90_TMA_STORE_IM2COL_3D`, `SM90_TMA_STORE_IM2COL_4D`, and `SM90_TMA_STORE_IM2COL_5D` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_STORE_IM2COL_3D`、`SM90_TMA_STORE_IM2COL_4D`、`SM90_TMA_STORE_IM2COL_5D` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 1183-1243 / 第 1183-1243 行
**EN**: Defines `SM90_TMA_STORE_IM2COL` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_STORE_IM2COL` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 1244-1303 / 第 1244-1303 行
**EN**: Defines `SM90_TMA_REDUCE_ADD_1D` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_REDUCE_ADD_1D` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 1304-1368 / 第 1304-1368 行
**EN**: Defines `SM90_TMA_REDUCE_ADD_2D`, `SM90_TMA_REDUCE_ADD_3D`, and `SM90_TMA_REDUCE_ADD_4D` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_REDUCE_ADD_2D`、`SM90_TMA_REDUCE_ADD_3D`、`SM90_TMA_REDUCE_ADD_4D` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 1369-1434 / 第 1369-1434 行
**EN**: Defines `SM90_TMA_REDUCE_ADD_5D` and `SM90_TMA_REDUCE_ADD` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_TMA_REDUCE_ADD_5D`、`SM90_TMA_REDUCE_ADD` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 1435-1495 / 第 1435-1495 行
**EN**: Defines `SM90_BULK_COPY_G2S`, `PREFETCH`, `SM90_BULK_COPY_S2G`, and `SM90_BULK_COPY_AUTO` and related types for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details. Preprocessor guards select the correct path for architecture, compiler, or feature availability.
**CN**: 定义 `SM90_BULK_COPY_G2S`、`PREFETCH`、`SM90_BULK_COPY_S2G`、`SM90_BULK_COPY_AUTO` 等相关类型，以支撑 copy atom、分块搬运与异步传输流水线。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。 预处理器保护会根据架构、编译器或特性可用性选择正确路径。

### Lines 1496-1496 / 第 1496-1496 行
**EN**: Continues the implementation details for copy atoms, tiled movement, and asynchronous transfer pipelines. The code ties CuTe movement rules to TMA descriptors or bulk-transfer details.
**CN**: 继续展开 copy atom、分块搬运与异步传输流水线 的实现细节。 这部分代码把 CuTe 的搬运规则绑定到 TMA 描述符或批量传输细节上。

---
## Key Concepts / 关键概念
- Copy atoms describe how data moves between memory spaces at the instruction level. / copy atom 描述了数据如何在不同内存空间之间以指令粒度移动。
- Traits capture alignment, vector width, source/destination layout, and pipeline behavior. / traits 会捕获对齐、向量宽度、源/目标布局以及流水线行为。
- Higher-level tiled copies are assembled by composing these low-level movement rules. / 更高层的 tiled copy 是通过组合这些底层搬运规则构造出来的。

## Dependencies / 依赖项
- `cute/config.hpp`
- `cute/arch/config.hpp`
- `cute/arch/copy.hpp`
- `cute/arch/copy_sm90.hpp`
- `cutlass/arch/synclog.hpp`
