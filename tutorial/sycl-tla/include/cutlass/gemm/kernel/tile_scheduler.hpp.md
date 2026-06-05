# tile_scheduler.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `include/cutlass/gemm/kernel/tile_scheduler.hpp`
- **Purpose / 用途 (EN):** Defines tile scheduler tags, parameter objects, or architecture-specific scheduler selection logic. Briefly, the file comment says: Utilities for selecting default tile schedulers.
- **Purpose / 用途 (CN):** 定义 Tile 调度标签、参数对象或与架构相关的调度选择逻辑。 文件注释的简要说明是：Utilities for selecting default tile schedulers。
- **Line count / 行数:** 473

## Line-by-Line Analysis / 逐行分析
> The source is grouped into contiguous significant line ranges for readability. / 为了便于阅读，下面按连续的重要代码区间进行分析。

### Lines 1-30

```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
   3 |  * SPDX-License-Identifier: BSD-3-Clause
   4 |  *
   5 |  * Redistribution and use in source and binary forms, with or without
   6 |  * modification, are permitted provided that the following conditions are met:
   7 |  *
   8 |  * 1. Redistributions of source code must retain the above copyright notice, this
   9 |  * list of conditions and the following disclaimer.
  10 |  *
  11 |  * 2. Redistributions in binary form must reproduce the above copyright notice,
  12 |  * this list of conditions and the following disclaimer in the documentation
  13 |  * and/or other materials provided with the distribution.
  14 |  *
  15 |  * 3. Neither the name of the copyright holder nor the names of its
  16 |  * contributors may be used to endorse or promote products derived from
  17 |  * this software without specific prior written permission.
  18 |  *
  19 |  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  20 |  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  21 |  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
  22 |  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
  23 |  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  24 |  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
  25 |  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
  26 |  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
  27 |  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  28 |  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  29 |  *
  30 |  **************************************************************************************************/
```
**EN:** This opening comment records the license, copyright ownership, and redistribution conditions that apply to the header.
**CN:** 开头注释说明了该头文件适用的许可证、版权归属以及再分发条件。

### Lines 32-32

```cpp
  32 | #pragma once
```
**EN:** This directive makes the header idempotent so repeated inclusion does not create duplicate definitions.
**CN:** 该指令让头文件只生效一次，避免重复包含导致的重复定义。

### Lines 34-36

```cpp
  34 | /*! \file
  35 |     \brief Utilities for selecting default tile schedulers
  36 | */
```
**EN:** This file-level comment names the header and introduces the high-level role of the kernel component that follows.
**CN:** 这一段文件级注释给出了头文件说明，并概括了后续内核组件的整体作用。

### Lines 38-39

```cpp
  38 | #include "cutlass/arch/arch.h"
  39 | #include "cutlass/detail/dependent_false.hpp"
```
**EN:** This include block imports `cutlass/arch/arch.h`, `cutlass/detail/dependent_false.hpp`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/arch/arch.h`, `cutlass/detail/dependent_false.hpp`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 43-47

```cpp
  43 | namespace cutlass::gemm {
  44 | 
  45 | //
  46 | // Tags for specifying tile schedulers
  47 | //
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

### Lines 49-49

```cpp
  49 | struct PersistentScheduler { };
```
**EN:** This block declares or specializes `PersistentScheduler`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `PersistentScheduler`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 51-51

```cpp
  51 | struct StreamKScheduler { };
```
**EN:** This block declares or specializes `StreamKScheduler`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `StreamKScheduler`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 53-53

```cpp
  53 | struct GroupScheduler { }; // Only used for Grouped GEMMs
```
**EN:** This block declares or specializes `GroupScheduler`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `GroupScheduler`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 55-55

```cpp
  55 | struct DynamicPersistentScheduler { };
```
**EN:** This block declares or specializes `DynamicPersistentScheduler`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `DynamicPersistentScheduler`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 57-57

```cpp
  57 | struct StaticPersistentScheduler { };
```
**EN:** This block declares or specializes `StaticPersistentScheduler`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `StaticPersistentScheduler`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 59-60

```cpp
  59 | } // namespace cutlass::gemm
  60 | ////////////////////////////////////////////////////////////////////////////////
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

### Lines 62-63

```cpp
  62 | #include "cutlass/gemm/kernel/sm90_tile_scheduler.hpp"
  63 | #include "cutlass/gemm/kernel/sm100_static_tile_scheduler.hpp" 
```
**EN:** This include block imports `cutlass/gemm/kernel/sm90_tile_scheduler.hpp`, `cutlass/gemm/kernel/sm100_static_tile_scheduler.hpp`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/gemm/kernel/sm90_tile_scheduler.hpp`, `cutlass/gemm/kernel/sm100_static_tile_scheduler.hpp`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 65-74

```cpp
  65 | #include "cutlass/gemm/kernel/sm90_tile_scheduler_stream_k.hpp"
  66 | #include "cutlass/gemm/kernel/sm90_tile_scheduler_group.hpp"
  67 | #include "cutlass/gemm/kernel/sm100_tile_scheduler.hpp"            
  68 | #include "cutlass/gemm/kernel/sm100_tile_scheduler_stream_k.hpp"   
  69 | #include "cutlass/gemm/kernel/sm100_tile_scheduler_group.hpp"
  70 | #if defined (SYCL_INTEL_TARGET)
  71 | #include "cutlass/gemm/kernel/xe_tile_scheduler_streamk.hpp"
  72 | #include "cutlass/gemm/kernel/xe_tile_scheduler_group.hpp"
  73 | #endif
  74 | ////////////////////////////////////////////////////////////////////////////////
```
**EN:** This block continues the scheduler logic, Stream-K flow, Xe backend logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑、Stream-K 流程、Xe 后端逻辑相关逻辑。

### Lines 76-80

```cpp
  76 | namespace cutlass::gemm::kernel::detail {
  77 | 
  78 | //
  79 | // Selectors mapping tile scheduler tag and arch tag to a tile scheduler class
  80 | //
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

### Lines 82-93

```cpp
  82 | template <
  83 |   class TileSchedulerTag,
  84 |   class ArchTag,
  85 |   class TileShape,
  86 |   class ClusterShape
  87 |   , uint32_t SchedulerPipelineStageCount = 2 
  88 |   , class ProblemShapeType = void
  89 | >
  90 | struct TileSchedulerSelector {
  91 |   static_assert(cutlass::detail::dependent_false<ArchTag>,
  92 |       "Could not select a tile scheduler for given parameters.");
  93 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 95-109

```cpp
  95 | template <
  96 |   class ArchTag,
  97 |   class TileShape,
  98 |   class ClusterShape
  99 |   , uint32_t SchedulerPipelineStageCount     
 100 | >
 101 | struct TileSchedulerSelector<
 102 |     PersistentScheduler,
 103 |     ArchTag,
 104 |     TileShape,
 105 |     ClusterShape
 106 |     , SchedulerPipelineStageCount              
 107 |   > {
 108 |   using Scheduler = PersistentTileSchedulerSm90;
 109 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 111-132

```cpp
 111 | // Default (void) for Sm90 maps to PersistentTileSchedulerSm90
 112 | template <
 113 |   class ArchTag,
 114 |   class TileShape,
 115 |   class ClusterShape
 116 |   , uint32_t SchedulerPipelineStageCount     
 117 | >
 118 | struct TileSchedulerSelector<
 119 |     void,
 120 |     ArchTag,
 121 |     TileShape,
 122 |     ClusterShape
 123 |     , SchedulerPipelineStageCount              
 124 |   > {
 125 |   using Scheduler = typename TileSchedulerSelector<
 126 |       PersistentScheduler,
 127 |       ArchTag,
 128 |       TileShape,
 129 |       ClusterShape
 130 |       , SchedulerPipelineStageCount            
 131 |   >::Scheduler;
 132 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 134-147

```cpp
 134 | template <
 135 |   class TileShape,
 136 |   class ClusterShape
 137 |   , uint32_t SchedulerPipelineStageCount     
 138 | >
 139 | struct TileSchedulerSelector<
 140 |     StreamKScheduler,
 141 |     arch::Sm90,
 142 |     TileShape,
 143 |     ClusterShape
 144 |     , SchedulerPipelineStageCount              
 145 |   > {
 146 |   using Scheduler = PersistentTileSchedulerSm90StreamK<TileShape, ClusterShape>;
 147 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 149-163

```cpp
 149 | template <
 150 |   class ArchTag,
 151 |   class TileShape,
 152 |   class ClusterShape, 
 153 |   uint32_t SchedulerPipelineStageCount     
 154 | >
 155 | struct TileSchedulerSelector<
 156 |     StaticPersistentScheduler,
 157 |     ArchTag,
 158 |     TileShape,
 159 |     ClusterShape
 160 |     , SchedulerPipelineStageCount              
 161 |   > {
 162 |   using Scheduler = PersistentTileSchedulerSm90;
 163 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 165-180

```cpp
 165 | template <
 166 |   class TileShape,
 167 |   class ClusterShape, 
 168 |   uint32_t SchedulerPipelineStageCount, 
 169 |   class GroupProblemShape
 170 | >
 171 | struct TileSchedulerSelector<
 172 |     GroupScheduler,
 173 |     arch::Sm90,
 174 |     TileShape,
 175 |     ClusterShape
 176 |     , SchedulerPipelineStageCount              
 177 |     , GroupProblemShape
 178 |   > {
 179 |   using Scheduler = PersistentTileSchedulerSm90Group<GroupProblemShape, SchedulerPipelineStageCount>;
 180 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 182-196

```cpp
 182 | #if defined (SYCL_INTEL_TARGET)
 183 | template <
 184 |   class TileShape,
 185 |   class ClusterShape,
 186 |   uint32_t ThreadsPerBlock
 187 | >
 188 | struct TileSchedulerSelector<
 189 |   StreamKScheduler,
 190 |   arch::IntelXe,
 191 |   TileShape,
 192 |   ClusterShape,
 193 |   ThreadsPerBlock
 194 |   > {
 195 |   using Scheduler = PersistentTileSchedulerXeStreamK<TileShape, ThreadsPerBlock>;
 196 | };
```
**EN:** This conditional-compilation block enables the following code only when the required architecture or backend feature is available.
**CN:** 这一段条件编译逻辑只在目标架构或后端特性可用时启用后续代码。

### Lines 198-217

```cpp
 198 | template <
 199 |   class TileShape,
 200 |   class ClusterShape,
 201 |   uint32_t SchedulerPipelineStageCount,
 202 |   class GroupProblemShape
 203 | >
 204 | struct TileSchedulerSelector<
 205 |     GroupScheduler,
 206 |     arch::IntelXe,
 207 |     TileShape,
 208 |     ClusterShape,
 209 |     SchedulerPipelineStageCount,
 210 |     GroupProblemShape
 211 |   > {
 212 |   using Scheduler = PersistentTileSchedulerXeGroup<GroupProblemShape>;
 213 | };
 214 | template <
 215 |   class TileShape,
 216 |   class ClusterShape
 217 | >
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 218-226

```cpp
 218 | struct TileSchedulerSelector<
 219 |   PersistentScheduler,
 220 |   arch::IntelXe,
 221 |   TileShape,
 222 |   ClusterShape
 223 |   > {
 224 |   using Scheduler = PersistentTileSchedulerSm90;
 225 | };
 226 | #endif
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 228-238

```cpp
 228 | template <class TileShape, class ClusterShape, uint32_t SchedulerPipelineStageCount>
 229 | struct TileSchedulerSelector<
 230 |     PersistentScheduler,
 231 |     arch::Sm100,
 232 |     TileShape,
 233 |     ClusterShape,
 234 |     SchedulerPipelineStageCount> {
 235 |   using Scheduler = PersistentTileSchedulerSm100<
 236 |                         ClusterShape,
 237 |                         SchedulerPipelineStageCount>;
 238 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 240-255

```cpp
 240 | // Ptr-Array kernel may provide a specialized ArrayProblemShape type
 241 | template <class TileShape,
 242 |   class ClusterShape,
 243 |   uint32_t SchedulerPipelineStageCount,
 244 |   class ProblemShape>
 245 | struct TileSchedulerSelector<
 246 |     PersistentScheduler,
 247 |     arch::Sm100,
 248 |     TileShape,
 249 |     ClusterShape,
 250 |     SchedulerPipelineStageCount,
 251 |     ProblemShape> {
 252 |   using Scheduler = PersistentTileSchedulerSm100<
 253 |                         ClusterShape,
 254 |                         SchedulerPipelineStageCount>;
 255 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 257-269

```cpp
 257 | // Default (void) for Sm100 maps to PersistentTileSchedulerSm100
 258 | template <class TileShape, class ClusterShape, uint32_t SchedulerPipelineStageCount>
 259 | struct TileSchedulerSelector<
 260 |     void,
 261 |     arch::Sm100,
 262 |     TileShape,
 263 |     ClusterShape,
 264 |     SchedulerPipelineStageCount> {
 265 |     using Scheduler = PersistentTileSchedulerSm100<
 266 |                 ClusterShape,
 267 |                 SchedulerPipelineStageCount
 268 |                 >;
 269 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 271-290

```cpp
 271 | // Default (void) for Sm100 maps to PersistentTileSchedulerSm100
 272 | // Ptr-Array kernel may provide a specialized ArrayProblemShape type
 273 | template <class TileShape,
 274 |   class ClusterShape,
 275 |   uint32_t SchedulerPipelineStageCount,
 276 |   class ProblemShape>
 277 | struct TileSchedulerSelector<
 278 |     void,
 279 |     arch::Sm100,
 280 |     TileShape,
 281 |     ClusterShape,
 282 |     SchedulerPipelineStageCount,
 283 |     ProblemShape> {
 284 |   using Scheduler = typename TileSchedulerSelector<
 285 |       PersistentScheduler,
 286 |       arch::Sm100,
 287 |       TileShape,
 288 |       ClusterShape,
 289 |       SchedulerPipelineStageCount>::Scheduler;
 290 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 292-308

```cpp
 292 | // SM100 Group tile scheduler
 293 | template <
 294 |   class TileShape,
 295 |   class ClusterShape,
 296 |   uint32_t SchedulerPipelineStageCount,
 297 |   class GroupProblemShape
 298 | >
 299 | struct TileSchedulerSelector<
 300 |     GroupScheduler,
 301 |     arch::Sm100,
 302 |     TileShape,
 303 |     ClusterShape,
 304 |     SchedulerPipelineStageCount,
 305 |     GroupProblemShape
 306 |   > {
 307 |   using Scheduler = PersistentTileSchedulerSm100Group<GroupProblemShape, SchedulerPipelineStageCount>;
 308 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 310-322

```cpp
 310 | // SM100 stream-K scheduler
 311 | template <class TileShape, class ClusterShape, uint32_t SchedulerPipelineStageCount>
 312 | struct TileSchedulerSelector<
 313 |     StreamKScheduler,
 314 |     arch::Sm100,
 315 |     TileShape,
 316 |     ClusterShape,
 317 |     SchedulerPipelineStageCount> {
 318 |   using Scheduler = PersistentTileSchedulerSm100StreamK<
 319 |                         TileShape,
 320 |                         ClusterShape,
 321 |                         SchedulerPipelineStageCount>;
 322 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 324-335

```cpp
 324 | // SM100 dynamic tile scheduler
 325 | template <class TileShape, class ClusterShape, uint32_t SchedulerPipelineStageCount>
 326 | struct TileSchedulerSelector<
 327 |     DynamicPersistentScheduler,
 328 |     arch::Sm100,
 329 |     TileShape,
 330 |     ClusterShape,
 331 |     SchedulerPipelineStageCount> {
 332 |   using Scheduler = PersistentTileSchedulerSm100<
 333 |                         ClusterShape,
 334 |                         SchedulerPipelineStageCount>;
 335 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 337-349

```cpp
 337 | template <
 338 |   class TileShape,
 339 |   class ClusterShape,
 340 |   uint32_t SchedulerPipelineStageCount
 341 | >
 342 | struct TileSchedulerSelector<
 343 |     StaticPersistentScheduler,
 344 |     arch::Sm100,
 345 |     TileShape,
 346 |     ClusterShape,
 347 |     SchedulerPipelineStageCount> {
 348 |   using Scheduler = StaticPersistentTileScheduler100;
 349 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 351-361

```cpp
 351 | template <class TileShape, class ClusterShape, uint32_t SchedulerPipelineStageCount>
 352 | struct TileSchedulerSelector<
 353 |     PersistentScheduler,
 354 |     arch::Sm103,
 355 |     TileShape,
 356 |     ClusterShape,
 357 |     SchedulerPipelineStageCount> {
 358 |   using Scheduler = PersistentTileSchedulerSm100<
 359 |                         ClusterShape,
 360 |                         SchedulerPipelineStageCount>;
 361 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 363-378

```cpp
 363 | // Ptr-Array kernel may provide a specialized ArrayProblemShape type
 364 | template <class TileShape,
 365 |   class ClusterShape,
 366 |   uint32_t SchedulerPipelineStageCount,
 367 |   class ProblemShape>
 368 | struct TileSchedulerSelector<
 369 |     PersistentScheduler,
 370 |     arch::Sm103,
 371 |     TileShape,
 372 |     ClusterShape,
 373 |     SchedulerPipelineStageCount,
 374 |     ProblemShape> {
 375 |   using Scheduler = PersistentTileSchedulerSm100<
 376 |                         ClusterShape,
 377 |                         SchedulerPipelineStageCount>;
 378 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 380-396

```cpp
 380 | // SM103 Group tile scheduler
 381 | template <
 382 |   class TileShape,
 383 |   class ClusterShape,
 384 |   uint32_t SchedulerPipelineStageCount,
 385 |   class GroupProblemShape
 386 | >
 387 | struct TileSchedulerSelector<
 388 |     GroupScheduler,
 389 |     arch::Sm103,
 390 |     TileShape,
 391 |     ClusterShape,
 392 |     SchedulerPipelineStageCount,
 393 |     GroupProblemShape
 394 |   > {
 395 |   using Scheduler = PersistentTileSchedulerSm100Group<GroupProblemShape, SchedulerPipelineStageCount>;
 396 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 398-409

```cpp
 398 | template <class TileShape, class ClusterShape, uint32_t SchedulerPipelineStageCount>
 399 | struct TileSchedulerSelector<
 400 |     StreamKScheduler,
 401 |     arch::Sm103,
 402 |     TileShape,
 403 |     ClusterShape,
 404 |     SchedulerPipelineStageCount> {
 405 |   using Scheduler = PersistentTileSchedulerSm100StreamK<
 406 |                         TileShape,
 407 |                         ClusterShape,
 408 |                         SchedulerPipelineStageCount>;
 409 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 411-423

```cpp
 411 | // Default (void) for Sm120 maps to PersistentTileSchedulerSm100
 412 | template <class TileShape, class ClusterShape, uint32_t SchedulerPipelineStageCount>
 413 | struct TileSchedulerSelector<
 414 |     void,
 415 |     arch::Sm120,
 416 |     TileShape,
 417 |     ClusterShape,
 418 |     SchedulerPipelineStageCount> {
 419 |     using Scheduler = PersistentTileSchedulerSm100<
 420 |                 ClusterShape,
 421 |                 SchedulerPipelineStageCount
 422 |                 >;
 423 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 425-434

```cpp
 425 | // PersistentScheduler for Sm120 maps to PersistentTileSchedulerSm100
 426 | template <class TileShape, class ClusterShape, uint32_t SchedulerPipelineStageCount>
 427 | struct TileSchedulerSelector<
 428 |     PersistentScheduler,
 429 |     arch::Sm120,
 430 |     TileShape,
 431 |     ClusterShape,
 432 |     SchedulerPipelineStageCount> {
 433 |   using Scheduler = PersistentTileSchedulerSm100<ClusterShape, SchedulerPipelineStageCount>;
 434 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 437-449

```cpp
 437 | // StreamKScheduler for Sm120 maps to PersistentTileSchedulerSm100StreamK
 438 | template <class TileShape, class ClusterShape, uint32_t SchedulerPipelineStageCount>
 439 | struct TileSchedulerSelector<
 440 |     StreamKScheduler,
 441 |     arch::Sm120,
 442 |     TileShape,
 443 |     ClusterShape,
 444 |     SchedulerPipelineStageCount> {
 445 |   using Scheduler = PersistentTileSchedulerSm100StreamK<
 446 |                         TileShape,
 447 |                         ClusterShape,
 448 |                         SchedulerPipelineStageCount>;
 449 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 451-467

```cpp
 451 | // SM120 Group tile scheduler
 452 | template <
 453 |   class TileShape,
 454 |   class ClusterShape,
 455 |   uint32_t SchedulerPipelineStageCount,
 456 |   class GroupProblemShape
 457 | >
 458 | struct TileSchedulerSelector<
 459 |     GroupScheduler,
 460 |     arch::Sm120,
 461 |     TileShape,
 462 |     ClusterShape,
 463 |     SchedulerPipelineStageCount,
 464 |     GroupProblemShape
 465 |   > {
 466 |   using Scheduler = PersistentTileSchedulerSm90Group<GroupProblemShape, SchedulerPipelineStageCount>;
 467 | };
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 471-471

```cpp
 471 | } // namespace cutlass::gemm::kernel::detail
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

## Key Concepts / 关键概念
- Template metaprogramming / 模板元编程
- Tile scheduling / Tile 调度
- Persistent scheduling / 持久化调度
- Stream-K scheduling / Stream-K 调度
- Grouped problem handling / 分组问题处理
- SM90 architecture tuning / SM90 架构调优
- SM100 architecture tuning / SM100 架构调优
- SM103 architecture tuning / SM103 架构调优

## Dependencies / 依赖关系
- **Included headers / 引入头文件:** `cutlass/arch/arch.h`, `cutlass/detail/dependent_false.hpp`, `cutlass/gemm/kernel/sm90_tile_scheduler.hpp`, `cutlass/gemm/kernel/sm100_static_tile_scheduler.hpp`, `cutlass/gemm/kernel/sm90_tile_scheduler_stream_k.hpp`, `cutlass/gemm/kernel/sm90_tile_scheduler_group.hpp`, `cutlass/gemm/kernel/sm100_tile_scheduler.hpp`, `cutlass/gemm/kernel/sm100_tile_scheduler_stream_k.hpp`, `cutlass/gemm/kernel/sm100_tile_scheduler_group.hpp`, `cutlass/gemm/kernel/xe_tile_scheduler_streamk.hpp`, `cutlass/gemm/kernel/xe_tile_scheduler_group.hpp`
- **Subsystems / 子系统:** Tile schedulers / Tile 调度器, Grouped problem support / 分组问题支持
- **Peer kernel headers / 同级内核头文件:** `cutlass/gemm/kernel/sm90_tile_scheduler.hpp`, `cutlass/gemm/kernel/sm100_static_tile_scheduler.hpp`, `cutlass/gemm/kernel/sm90_tile_scheduler_stream_k.hpp`, `cutlass/gemm/kernel/sm90_tile_scheduler_group.hpp`, `cutlass/gemm/kernel/sm100_tile_scheduler.hpp`, `cutlass/gemm/kernel/sm100_tile_scheduler_stream_k.hpp`, `cutlass/gemm/kernel/sm100_tile_scheduler_group.hpp`, `cutlass/gemm/kernel/xe_tile_scheduler_streamk.hpp`, ... (+1 more)
