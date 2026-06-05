# terminology — Documentation Analysis / 文档分析

## Source / 来源
- Source: `/root/xw/sycl-tla/media/docs/cpp/terminology.md`
- Purpose: Provides a glossary of CUTLASS tensor, layout, storage, iterator, and execution-model vocabulary. / 提供一份 CUTLASS 张量、布局、存储、迭代器与执行模型术语表。

## Content Analysis / 内容分析
### CUTLASS Terminology
**EN:** The page is structured as a flat glossary rather than a narrative document. Its main value is normalization: it aligns CUTLASS-specific meanings of terms like capacity, extent, fragment, layout, tile, and policy with either STL conventions or CUTLASS-specific deviations.
**CN:** 本页采用扁平化词汇表而非叙事式结构。其核心价值在于统一定义：把 capacity、extent、fragment、layout、tile、policy 等术语与 STL 传统或 CUTLASS 特定含义对齐。

### Core tensor and layout vocabulary
**EN:** Terms such as `cute::Layout`, `cute::Tensor`, `Layout`, `TensorRef`, `TensorView`, `Extent`, `Rank`, `Size`, and `Capacity` define the conceptual model for representing multidimensional data and its storage requirements.
**CN:** `cute::Layout`、`cute::Tensor`、`Layout`、`TensorRef`、`TensorView`、`Extent`、`Rank`、`Size`、`Capacity` 等词共同构成了多维数据表示及其存储需求的概念基础。

### Storage and datatype vocabulary
**EN:** Entries like `Element`, `Numeric Type`, `Storage`, `Array<T, N>`, `AlignedBuffer<T, N>`, and `sizeof_bits<T>::value` focus on how data is physically represented, packed, and reasoned about at compile time.
**CN:** `Element`、`Numeric Type`、`Storage`、`Array<T, N>`、`AlignedBuffer<T, N>` 与 `sizeof_bits<T>::value` 等条目更关注数据在物理层面的表示、打包方式以及编译期推导。

### Execution and algorithm vocabulary
**EN:** Terms such as `Fragment`, `Tile`, `Tile Iterator`, `Thread Map`, `Operator`, `Warp`, `Register`, `Residue`, and `Policy` explain how CUTLASS decomposes work across hardware threads and template layers. The deprecation notes also signal the migration path toward CuTe abstractions.
**CN:** `Fragment`、`Tile`、`Tile Iterator`、`Thread Map`、`Operator`、`Warp`、`Register`、`Residue`、`Policy` 等术语解释了 CUTLASS 如何在硬件线程层级和模板层级上拆分计算；其中的弃用说明也体现了向 CuTe 抽象迁移的方向。

## Key Concepts / 关键概念
- Glossary-driven onboarding / 以术语表驱动的入门方式
- Data-model vocabulary / 数据模型术语
- Storage and packing terminology / 存储与打包术语
- Execution hierarchy terminology / 执行层级术语
- CuTe migration notes / 向 CuTe 迁移的提示

## Related Files / 相关文件
- `media/docs/cpp/layout.md` — layout-focused companion document / 与布局相关的配套文档
- `media/docs/cpp/fundamental_types.md` — datatype and container vocabulary / 数据类型与容器词汇来源
- `media/docs/cpp/tile_iterator_concept.md` — formal iterator terminology / 形式化迭代器术语说明
- `include/cutlass/tensor_ref.h` — concrete tensor reference types / 具体张量引用类型
