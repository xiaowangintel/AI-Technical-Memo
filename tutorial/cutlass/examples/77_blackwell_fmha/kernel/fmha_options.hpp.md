# fmha_options.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/77_blackwell_fmha/kernel/fmha_options.hpp`  
**Purpose / 用途**: Defines the small compile-time option system used to configure FMHA kernels without building enormous monolithic parameter lists. / 定义用于配置 FMHA kernel 的小型编译期选项系统，避免构造过于庞大的参数列表。

---

## Line-by-Line Analysis / 逐行分析

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
```

**EN**: This comment block frames the file: it usually states the target architecture, problem family, usage pattern, or design motivation before the code starts. That context matters because later template choices are easier to understand once the intended hardware path and workload are known. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一段注释用于给文件定调：通常先说明目标架构、问题类型、使用方式或设计动机，再进入具体代码。先理解这些背景，再看后面的模板选择与硬件路径会更清晰。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
#pragma once
```

**EN**: This preprocessor block gates architecture-specific code or closes a specialized compilation path. It keeps the file buildable across toolchains and GPU targets even when the most optimized path only exists on a subset of architectures. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一预处理块负责为特定架构的代码设置编译门槛，或者结束某条专门化路径。这样即便最优路径只存在于部分 GPU 架构上，文件依旧可以在不同工具链与目标上保持可编译。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
#include "cutlass/cutlass.h"

namespace cutlass::fmha::kernel {

template<auto kTag, typename Default, typename... Options>
struct find_option;

template<auto kTag, typename Default>
struct find_option<kTag, Default> {
  using option_value = Default;
};

template<auto kTag, typename Default, typename Option, typename... Options>
struct find_option<kTag, Default, Option, Options...> :
  std::conditional_t<
    Option::tag == kTag,
    Option,
    find_option<kTag, Default, Options...>
  >
{};

template<auto kTag, typename Default, typename... Options>
using find_option_t = typename find_option<kTag, Default, Options...>::option_value;

enum class Tag {
  kIsPersistent,
  kNumMmaWarpGroups,
  kLoadsQSeparately,

  kIsMainloopLocked,
  kIsEpilogueLocked,

  kStagesQ,
  kStagesKV,

  kEpilogueKind,

  kBlocksPerSM,
  kClusterM,
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Exposes scheduler policy as a first-class performance choice. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。把调度策略作为一级性能选择暴露出来。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  kAccQK
};

template<auto kTag, class Value>
struct Option {
  static constexpr auto tag = kTag;
  using option_value = Value;
};

}  // namespace cutlass::fmha::kernel
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

## Key Concepts / 关键概念

- Architecture-tagged specialization (`Sm100`/`Sm103`/`Sm120`) / 基于架构标签的特化（`Sm100`/`Sm103`/`Sm120`）
- Warp-specialized FMHA layering across collective, kernel, and device wrappers / 跨 collective、kernel 与 device wrapper 的 warp-specialized FMHA 分层
- Scheduler policy as a first-class optimization knob / 作为一级优化旋钮的调度策略

## Dependencies / 依赖项

- `cutlass/cutlass.h` — core CUTLASS types, architecture tags, and utilities / CUTLASS 核心类型、架构标签与工具
