# config.h — Code Analysis / 代码分析

**Source / 源文件**: `include/cutlass/arch/config.h`
**Purpose / 用途**: Defines compile-time feature gates that enable or disable architecture-specific CUTLASS instructions and capabilities. / 定义编译期特性开关，用于启用或禁用架构相关的 CUTLASS 指令与能力。

---

## Line-by-Line Analysis / 逐行分析

```cpp
/***************************************************************************************************
```
**EN**: Line 1 starts the file banner comment block.
**CN**: 第 1 行开始文件头部注释块。

```cpp
 * Copyright (c) 2024 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
```
**EN**: Line 2 states the copyright ownership for this header.
**CN**: 第 2 行声明该头文件的版权归属。

```cpp
 * SPDX-License-Identifier: BSD-3-Clause
```
**EN**: Line 3 gives the SPDX license identifier used for automated license tracking.
**CN**: 第 3 行给出 SPDX 许可证标识，便于自动化许可证跟踪。

```cpp
 *
```
**EN**: Line 4 closes or continues the banner-style comment decoration.
**CN**: 第 4 行延续或结束这种横幅样式的注释装饰。

```cpp
 * Redistribution and use in source and binary forms, with or without
```
**EN**: Line 5 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 5 行继续当前注释块，补充说明性或法律文本。

```cpp
 * modification, are permitted provided that the following conditions are met:
```
**EN**: Line 6 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 6 行继续当前注释块，补充说明性或法律文本。

```cpp
 *
```
**EN**: Line 7 closes or continues the banner-style comment decoration.
**CN**: 第 7 行延续或结束这种横幅样式的注释装饰。

```cpp
 * 1. Redistributions of source code must retain the above copyright notice, this
```
**EN**: Line 8 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 8 行继续当前注释块，补充说明性或法律文本。

```cpp
 * list of conditions and the following disclaimer.
```
**EN**: Line 9 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 9 行继续当前注释块，补充说明性或法律文本。

```cpp
 *
```
**EN**: Line 10 closes or continues the banner-style comment decoration.
**CN**: 第 10 行延续或结束这种横幅样式的注释装饰。

```cpp
 * 2. Redistributions in binary form must reproduce the above copyright notice,
```
**EN**: Line 11 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 11 行继续当前注释块，补充说明性或法律文本。

```cpp
 * this list of conditions and the following disclaimer in the documentation
```
**EN**: Line 12 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 12 行继续当前注释块，补充说明性或法律文本。

```cpp
 * and/or other materials provided with the distribution.
```
**EN**: Line 13 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 13 行继续当前注释块，补充说明性或法律文本。

```cpp
 *
```
**EN**: Line 14 closes or continues the banner-style comment decoration.
**CN**: 第 14 行延续或结束这种横幅样式的注释装饰。

```cpp
 * 3. Neither the name of the copyright holder nor the names of its
```
**EN**: Line 15 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 15 行继续当前注释块，补充说明性或法律文本。

```cpp
 * contributors may be used to endorse or promote products derived from
```
**EN**: Line 16 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 16 行继续当前注释块，补充说明性或法律文本。

```cpp
 * this software without specific prior written permission.
```
**EN**: Line 17 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 17 行继续当前注释块，补充说明性或法律文本。

```cpp
 *
```
**EN**: Line 18 closes or continues the banner-style comment decoration.
**CN**: 第 18 行延续或结束这种横幅样式的注释装饰。

```cpp
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
```
**EN**: Line 19 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 19 行继续当前注释块，补充说明性或法律文本。

```cpp
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
```
**EN**: Line 20 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 20 行继续当前注释块，补充说明性或法律文本。

```cpp
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
```
**EN**: Line 21 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 21 行继续当前注释块，补充说明性或法律文本。

```cpp
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
```
**EN**: Line 22 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 22 行继续当前注释块，补充说明性或法律文本。

```cpp
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
```
**EN**: Line 23 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 23 行继续当前注释块，补充说明性或法律文本。

```cpp
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
```
**EN**: Line 24 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 24 行继续当前注释块，补充说明性或法律文本。

```cpp
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
```
**EN**: Line 25 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 25 行继续当前注释块，补充说明性或法律文本。

```cpp
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
```
**EN**: Line 26 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 26 行继续当前注释块，补充说明性或法律文本。

```cpp
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
```
**EN**: Line 27 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 27 行继续当前注释块，补充说明性或法律文本。

```cpp
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```
**EN**: Line 28 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 28 行继续当前注释块，补充说明性或法律文本。

```cpp
 *
```
**EN**: Line 29 closes or continues the banner-style comment decoration.
**CN**: 第 29 行延续或结束这种横幅样式的注释装饰。

```cpp
 **************************************************************************************************/
```
**EN**: Line 30 closes or continues the banner-style comment decoration.
**CN**: 第 30 行延续或结束这种横幅样式的注释装饰。

```cpp

```
**EN**: Line 31 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 31 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
/*! \file
```
**EN**: Line 32 continues the surrounding comment block with explanatory or legal text.
**CN**: 第 32 行继续当前注释块，补充说明性或法律文本。

```cpp
    \brief Definitions for architecture macros
```
**EN**: Line 33 summarizes the main purpose of the header.
**CN**: 第 33 行概括该头文件的主要用途。

```cpp
*/
```
**EN**: Line 34 closes or continues the banner-style comment decoration.
**CN**: 第 34 行延续或结束这种横幅样式的注释装饰。

```cpp

```
**EN**: Line 35 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 35 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
#pragma once
```
**EN**: Line 36 uses `#pragma once` to prevent multiple inclusion of this header.
**CN**: 第 36 行使用 `#pragma once` 防止该头文件被重复包含。

```cpp

```
**EN**: Line 37 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 37 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
#include "cutlass/platform/platform.h"
```
**EN**: Line 38 includes `cutlass/platform/platform.h` so this header can use its declarations, macros, or intrinsics.
**CN**: 第 38 行包含 `cutlass/platform/platform.h`，以便本头文件使用其中的声明、宏或底层 intrinsic。

```cpp

```
**EN**: Line 39 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 39 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Line 40 is a single-line documentation comment describing nearby code: //////////////////////////////////////////////////////////////////////////////////////////////
**CN**: 第 40 行是单行文档注释，用于描述附近代码：//////////////////////////////////////////////////////////////////////////////////////////////

```cpp

```
**EN**: Line 41 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 41 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
// SM90
```
**EN**: Line 42 is an inline comment that explains the nearby code or intent.
**CN**: 第 42 行是行内注释，用于解释附近代码或设计意图。

```cpp
#if (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 0))
```
**EN**: Line 43 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 43 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
  #define CUTLASS_ARCH_MMA_SM90_SUPPORTED 1
```
**EN**: Line 44 defines macro `CUTLASS_ARCH_MMA_SM90_SUPPORTED` to steer later compilation paths.
**CN**: 第 44 行定义宏 `CUTLASS_ARCH_MMA_SM90_SUPPORTED`，用于控制后续的编译路径。

```cpp
  #if (!defined(CUTLASS_ARCH_MMA_SM90_ENABLED) && defined(__CUDA_ARCH__) && __CUDA_ARCH__ == 900)
```
**EN**: Line 45 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 45 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    #define CUTLASS_ARCH_MMA_SM90_ENABLED 1
```
**EN**: Line 46 defines macro `CUTLASS_ARCH_MMA_SM90_ENABLED` to steer later compilation paths.
**CN**: 第 46 行定义宏 `CUTLASS_ARCH_MMA_SM90_ENABLED`，用于控制后续的编译路径。

```cpp

```
**EN**: Line 47 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 47 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    #if (!defined(CUTLASS_ARCH_MMA_SM90A_ENABLED) && defined(__CUDA_ARCH_FEAT_SM90_ALL))
```
**EN**: Line 48 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 48 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
      #define CUTLASS_ARCH_MMA_SM90A_ENABLED 1
```
**EN**: Line 49 defines macro `CUTLASS_ARCH_MMA_SM90A_ENABLED` to steer later compilation paths.
**CN**: 第 49 行定义宏 `CUTLASS_ARCH_MMA_SM90A_ENABLED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 50 ends the current conditional-compilation block.
**CN**: 第 50 行结束当前条件编译块。

```cpp
  #endif
```
**EN**: Line 51 ends the current conditional-compilation block.
**CN**: 第 51 行结束当前条件编译块。

```cpp
#endif
```
**EN**: Line 52 ends the current conditional-compilation block.
**CN**: 第 52 行结束当前条件编译块。

```cpp

```
**EN**: Line 53 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 53 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
#if (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 2))
```
**EN**: Line 54 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 54 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
  #define CUTLASS_ARCH_MMA_SPARSE_SM90_SUPPORTED
```
**EN**: Line 55 defines macro `CUTLASS_ARCH_MMA_SPARSE_SM90_SUPPORTED` to steer later compilation paths.
**CN**: 第 55 行定义宏 `CUTLASS_ARCH_MMA_SPARSE_SM90_SUPPORTED`，用于控制后续的编译路径。

```cpp
#endif
```
**EN**: Line 56 ends the current conditional-compilation block.
**CN**: 第 56 行结束当前条件编译块。

```cpp

```
**EN**: Line 57 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 57 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Line 58 is a single-line documentation comment describing nearby code: //////////////////////////////////////////////////////////////////////////////////////////////
**CN**: 第 58 行是单行文档注释，用于描述附近代码：//////////////////////////////////////////////////////////////////////////////////////////////

```cpp

```
**EN**: Line 59 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 59 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
// Modifiable TMA
```
**EN**: Line 60 is an inline comment that explains the nearby code or intent.
**CN**: 第 60 行是行内注释，用于解释附近代码或设计意图。

```cpp
// tensormap.replace is arch conditional
```
**EN**: Line 61 is an inline comment that explains the nearby code or intent.
**CN**: 第 61 行是行内注释，用于解释附近代码或设计意图。

```cpp
#if (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 3))
```
**EN**: Line 62 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 62 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
  #define CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED 1
```
**EN**: Line 63 defines macro `CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED` to steer later compilation paths.
**CN**: 第 63 行定义宏 `CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED`，用于控制后续的编译路径。

```cpp
  #if (!defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_ENABLED) && \
```
**EN**: Line 64 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 64 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    (defined(__CUDA_ARCH_FEAT_SM90_ALL)  || defined(__CUDA_ARCH_FEAT_SM100_ALL) || \
```
**EN**: Line 65 contributes to the surrounding declaration or implementation logic.
**CN**: 第 65 行为周围的声明或实现逻辑提供组成部分。

```cpp
     defined(__CUDA_ARCH_FEAT_SM101_ALL) || defined(__CUDA_ARCH_FEAT_SM120_ALL)))
```
**EN**: Line 66 contributes to the surrounding declaration or implementation logic.
**CN**: 第 66 行为周围的声明或实现逻辑提供组成部分。

```cpp
    #define CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_ENABLED 1
```
**EN**: Line 67 defines macro `CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_ENABLED` to steer later compilation paths.
**CN**: 第 67 行定义宏 `CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_ENABLED`，用于控制后续的编译路径。

```cpp
  #endif
```
**EN**: Line 68 ends the current conditional-compilation block.
**CN**: 第 68 行结束当前条件编译块。

```cpp
#endif
```
**EN**: Line 69 ends the current conditional-compilation block.
**CN**: 第 69 行结束当前条件编译块。

```cpp

```
**EN**: Line 70 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 70 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp

```
**EN**: Line 71 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 71 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Line 72 is a single-line documentation comment describing nearby code: //////////////////////////////////////////////////////////////////////////////////////////////
**CN**: 第 72 行是单行文档注释，用于描述附近代码：//////////////////////////////////////////////////////////////////////////////////////////////

```cpp

```
**EN**: Line 73 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 73 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
// SM90 F64
```
**EN**: Line 74 is an inline comment that explains the nearby code or intent.
**CN**: 第 74 行是行内注释，用于解释附近代码或设计意图。

```cpp
#if (__CUDACC_VER_MAJOR__ > 11 || (__CUDACC_VER_MAJOR__ == 11 && __CUDACC_VER_MINOR__ >= 8))
```
**EN**: Line 75 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 75 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
  #define CUTLASS_ARCH_MMA_SM90_F64_MMA_SUPPORTED 1
```
**EN**: Line 76 defines macro `CUTLASS_ARCH_MMA_SM90_F64_MMA_SUPPORTED` to steer later compilation paths.
**CN**: 第 76 行定义宏 `CUTLASS_ARCH_MMA_SM90_F64_MMA_SUPPORTED`，用于控制后续的编译路径。

```cpp
  #if (!defined(CUTLASS_ARCH_MMA_SM90_F64_MMA_ENABLED) && defined(__CUDA_ARCH__) && __CUDA_ARCH__ >= 900)
```
**EN**: Line 77 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 77 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    #define CUTLASS_ARCH_MMA_SM90_F64_MMA_ENABLED 1
```
**EN**: Line 78 defines macro `CUTLASS_ARCH_MMA_SM90_F64_MMA_ENABLED` to steer later compilation paths.
**CN**: 第 78 行定义宏 `CUTLASS_ARCH_MMA_SM90_F64_MMA_ENABLED`，用于控制后续的编译路径。

```cpp
  #endif
```
**EN**: Line 79 ends the current conditional-compilation block.
**CN**: 第 79 行结束当前条件编译块。

```cpp
#endif
```
**EN**: Line 80 ends the current conditional-compilation block.
**CN**: 第 80 行结束当前条件编译块。

```cpp

```
**EN**: Line 81 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 81 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp

```
**EN**: Line 82 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 82 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Line 83 is a single-line documentation comment describing nearby code: //////////////////////////////////////////////////////////////////////////////////////////////
**CN**: 第 83 行是单行文档注释，用于描述附近代码：//////////////////////////////////////////////////////////////////////////////////////////////

```cpp

```
**EN**: Line 84 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 84 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
// SM100, SM100a
```
**EN**: Line 85 is an inline comment that explains the nearby code or intent.
**CN**: 第 85 行是行内注释，用于解释附近代码或设计意图。

```cpp
#if !CUTLASS_CLANG_CUDA && (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 8))
```
**EN**: Line 86 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 86 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
  #define CUTLASS_ARCH_MMA_SM100_SUPPORTED 1
```
**EN**: Line 87 defines macro `CUTLASS_ARCH_MMA_SM100_SUPPORTED` to steer later compilation paths.
**CN**: 第 87 行定义宏 `CUTLASS_ARCH_MMA_SM100_SUPPORTED`，用于控制后续的编译路径。

```cpp
  #if (!defined(CUTLASS_ARCH_MMA_SM100_ENABLED) && defined(__CUDA_ARCH__) && __CUDA_ARCH__ == 1000)
```
**EN**: Line 88 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 88 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    #define CUTLASS_ARCH_MMA_SM100_ENABLED 1
```
**EN**: Line 89 defines macro `CUTLASS_ARCH_MMA_SM100_ENABLED` to steer later compilation paths.
**CN**: 第 89 行定义宏 `CUTLASS_ARCH_MMA_SM100_ENABLED`，用于控制后续的编译路径。

```cpp

```
**EN**: Line 90 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 90 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    #if (!defined(CUTLASS_ARCH_MMA_SM100A_ENABLED) && defined(__CUDA_ARCH_FEAT_SM100_ALL))
```
**EN**: Line 91 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 91 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
      #define CUTLASS_ARCH_MMA_SM100A_ENABLED 1
```
**EN**: Line 92 defines macro `CUTLASS_ARCH_MMA_SM100A_ENABLED` to steer later compilation paths.
**CN**: 第 92 行定义宏 `CUTLASS_ARCH_MMA_SM100A_ENABLED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 93 ends the current conditional-compilation block.
**CN**: 第 93 行结束当前条件编译块。

```cpp

```
**EN**: Line 94 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 94 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    // SM100f
```
**EN**: Line 95 is an inline comment that explains the nearby code or intent.
**CN**: 第 95 行是行内注释，用于解释附近代码或设计意图。

```cpp
    #if (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9))
```
**EN**: Line 96 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 96 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    #define CUTLASS_ARCH_MMA_SM100F_SUPPORTED 1
```
**EN**: Line 97 defines macro `CUTLASS_ARCH_MMA_SM100F_SUPPORTED` to steer later compilation paths.
**CN**: 第 97 行定义宏 `CUTLASS_ARCH_MMA_SM100F_SUPPORTED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 98 ends the current conditional-compilation block.
**CN**: 第 98 行结束当前条件编译块。

```cpp

```
**EN**: Line 99 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 99 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    #if (!defined(CUTLASS_ARCH_MMA_SM100F_ENABLED) && CUDA_ARCH_FAMILY(1000))
```
**EN**: Line 100 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 100 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
      #define CUTLASS_ARCH_MMA_SM100F_ENABLED CUTLASS_ARCH_MMA_SM100F_SUPPORTED
```
**EN**: Line 101 defines macro `CUTLASS_ARCH_MMA_SM100F_ENABLED` to steer later compilation paths.
**CN**: 第 101 行定义宏 `CUTLASS_ARCH_MMA_SM100F_ENABLED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 102 ends the current conditional-compilation block.
**CN**: 第 102 行结束当前条件编译块。

```cpp
  #endif
```
**EN**: Line 103 ends the current conditional-compilation block.
**CN**: 第 103 行结束当前条件编译块。

```cpp
#endif
```
**EN**: Line 104 ends the current conditional-compilation block.
**CN**: 第 104 行结束当前条件编译块。

```cpp

```
**EN**: Line 105 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 105 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Line 106 is a single-line documentation comment describing nearby code: //////////////////////////////////////////////////////////////////////////////////////////////
**CN**: 第 106 行是单行文档注释，用于描述附近代码：//////////////////////////////////////////////////////////////////////////////////////////////

```cpp

```
**EN**: Line 107 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 107 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
// SM101 and SM101a
```
**EN**: Line 108 is an inline comment that explains the nearby code or intent.
**CN**: 第 108 行是行内注释，用于解释附近代码或设计意图。

```cpp
#if !CUTLASS_CLANG_CUDA && (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 8)
```
**EN**: Line 109 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 109 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
  #define CUTLASS_ARCH_MMA_SM101_SUPPORTED 1
```
**EN**: Line 110 defines macro `CUTLASS_ARCH_MMA_SM101_SUPPORTED` to steer later compilation paths.
**CN**: 第 110 行定义宏 `CUTLASS_ARCH_MMA_SM101_SUPPORTED`，用于控制后续的编译路径。

```cpp
  #if (!defined(CUTLASS_ARCH_MMA_SM101_ENABLED) && defined(__CUDA_ARCH__) && __CUDA_ARCH__ == 1010)
```
**EN**: Line 111 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 111 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    #define CUTLASS_ARCH_MMA_SM101_ENABLED 1
```
**EN**: Line 112 defines macro `CUTLASS_ARCH_MMA_SM101_ENABLED` to steer later compilation paths.
**CN**: 第 112 行定义宏 `CUTLASS_ARCH_MMA_SM101_ENABLED`，用于控制后续的编译路径。

```cpp

```
**EN**: Line 113 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 113 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    #if (!defined(CUTLASS_ARCH_MMA_SM101A_ENABLED) && defined(__CUDA_ARCH_FEAT_SM101_ALL))
```
**EN**: Line 114 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 114 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
      #define CUTLASS_ARCH_MMA_SM101A_ENABLED 1
```
**EN**: Line 115 defines macro `CUTLASS_ARCH_MMA_SM101A_ENABLED` to steer later compilation paths.
**CN**: 第 115 行定义宏 `CUTLASS_ARCH_MMA_SM101A_ENABLED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 116 ends the current conditional-compilation block.
**CN**: 第 116 行结束当前条件编译块。

```cpp

```
**EN**: Line 117 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 117 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    // SM101f
```
**EN**: Line 118 is an inline comment that explains the nearby code or intent.
**CN**: 第 118 行是行内注释，用于解释附近代码或设计意图。

```cpp
    #if !CUTLASS_CLANG_CUDA && (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9)
```
**EN**: Line 119 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 119 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    #define CUTLASS_ARCH_MMA_SM101F_SUPPORTED 1
```
**EN**: Line 120 defines macro `CUTLASS_ARCH_MMA_SM101F_SUPPORTED` to steer later compilation paths.
**CN**: 第 120 行定义宏 `CUTLASS_ARCH_MMA_SM101F_SUPPORTED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 121 ends the current conditional-compilation block.
**CN**: 第 121 行结束当前条件编译块。

```cpp

```
**EN**: Line 122 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 122 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    #if (!defined(CUTLASS_ARCH_MMA_SM101F_ENABLED) && CUDA_ARCH_FAMILY(1010))
```
**EN**: Line 123 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 123 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
      #define CUTLASS_ARCH_MMA_SM101F_ENABLED CUTLASS_ARCH_MMA_SM101F_SUPPORTED
```
**EN**: Line 124 defines macro `CUTLASS_ARCH_MMA_SM101F_ENABLED` to steer later compilation paths.
**CN**: 第 124 行定义宏 `CUTLASS_ARCH_MMA_SM101F_ENABLED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 125 ends the current conditional-compilation block.
**CN**: 第 125 行结束当前条件编译块。

```cpp
  #endif
```
**EN**: Line 126 ends the current conditional-compilation block.
**CN**: 第 126 行结束当前条件编译块。

```cpp
#endif
```
**EN**: Line 127 ends the current conditional-compilation block.
**CN**: 第 127 行结束当前条件编译块。

```cpp

```
**EN**: Line 128 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 128 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Line 129 is a single-line documentation comment describing nearby code: //////////////////////////////////////////////////////////////////////////////////////////////
**CN**: 第 129 行是单行文档注释，用于描述附近代码：//////////////////////////////////////////////////////////////////////////////////////////////

```cpp

```
**EN**: Line 130 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 130 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
// SM110 and SM110a only on 13.0 and above
```
**EN**: Line 131 is an inline comment that explains the nearby code or intent.
**CN**: 第 131 行是行内注释，用于解释附近代码或设计意图。

```cpp
#if !CUTLASS_CLANG_CUDA && (__CUDACC_VER_MAJOR__ > 13 || (__CUDACC_VER_MAJOR__ == 13 && __CUDACC_VER_MINOR__ >= 0))
```
**EN**: Line 132 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 132 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
  #define CUTLASS_ARCH_MMA_SM110_SUPPORTED 1
```
**EN**: Line 133 defines macro `CUTLASS_ARCH_MMA_SM110_SUPPORTED` to steer later compilation paths.
**CN**: 第 133 行定义宏 `CUTLASS_ARCH_MMA_SM110_SUPPORTED`，用于控制后续的编译路径。

```cpp
  #if (!defined(CUTLASS_ARCH_MMA_SM110_ENABLED) && defined(__CUDA_ARCH__) && __CUDA_ARCH__ == 1100)
```
**EN**: Line 134 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 134 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    #define CUTLASS_ARCH_MMA_SM110_ENABLED 1
```
**EN**: Line 135 defines macro `CUTLASS_ARCH_MMA_SM110_ENABLED` to steer later compilation paths.
**CN**: 第 135 行定义宏 `CUTLASS_ARCH_MMA_SM110_ENABLED`，用于控制后续的编译路径。

```cpp

```
**EN**: Line 136 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 136 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    #if (!defined(CUTLASS_ARCH_MMA_SM110A_ENABLED) && defined(__CUDA_ARCH_FEAT_SM110_ALL))
```
**EN**: Line 137 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 137 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
      #define CUTLASS_ARCH_MMA_SM110A_ENABLED 1
```
**EN**: Line 138 defines macro `CUTLASS_ARCH_MMA_SM110A_ENABLED` to steer later compilation paths.
**CN**: 第 138 行定义宏 `CUTLASS_ARCH_MMA_SM110A_ENABLED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 139 ends the current conditional-compilation block.
**CN**: 第 139 行结束当前条件编译块。

```cpp

```
**EN**: Line 140 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 140 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    // SM110f
```
**EN**: Line 141 is an inline comment that explains the nearby code or intent.
**CN**: 第 141 行是行内注释，用于解释附近代码或设计意图。

```cpp
    #if (__CUDACC_VER_MAJOR__ > 13 || (__CUDACC_VER_MAJOR__ == 13 && __CUDACC_VER_MINOR__ >= 0))
```
**EN**: Line 142 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 142 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    #define CUTLASS_ARCH_MMA_SM110F_SUPPORTED 1
```
**EN**: Line 143 defines macro `CUTLASS_ARCH_MMA_SM110F_SUPPORTED` to steer later compilation paths.
**CN**: 第 143 行定义宏 `CUTLASS_ARCH_MMA_SM110F_SUPPORTED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 144 ends the current conditional-compilation block.
**CN**: 第 144 行结束当前条件编译块。

```cpp

```
**EN**: Line 145 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 145 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    #if (!defined(CUTLASS_ARCH_MMA_SM110F_ENABLED) && CUDA_ARCH_FAMILY(1100))
```
**EN**: Line 146 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 146 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
      #define CUTLASS_ARCH_MMA_SM110F_ENABLED CUTLASS_ARCH_MMA_SM110F_SUPPORTED
```
**EN**: Line 147 defines macro `CUTLASS_ARCH_MMA_SM110F_ENABLED` to steer later compilation paths.
**CN**: 第 147 行定义宏 `CUTLASS_ARCH_MMA_SM110F_ENABLED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 148 ends the current conditional-compilation block.
**CN**: 第 148 行结束当前条件编译块。

```cpp
  #endif
```
**EN**: Line 149 ends the current conditional-compilation block.
**CN**: 第 149 行结束当前条件编译块。

```cpp
#endif
```
**EN**: Line 150 ends the current conditional-compilation block.
**CN**: 第 150 行结束当前条件编译块。

```cpp

```
**EN**: Line 151 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 151 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Line 152 is a single-line documentation comment describing nearby code: //////////////////////////////////////////////////////////////////////////////////////////////
**CN**: 第 152 行是单行文档注释，用于描述附近代码：//////////////////////////////////////////////////////////////////////////////////////////////

```cpp

```
**EN**: Line 153 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 153 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
// SM120 and SM120a
```
**EN**: Line 154 is an inline comment that explains the nearby code or intent.
**CN**: 第 154 行是行内注释，用于解释附近代码或设计意图。

```cpp
#if !CUTLASS_CLANG_CUDA && (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 8))
```
**EN**: Line 155 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 155 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
  #define CUTLASS_ARCH_MMA_SM120_SUPPORTED 1
```
**EN**: Line 156 defines macro `CUTLASS_ARCH_MMA_SM120_SUPPORTED` to steer later compilation paths.
**CN**: 第 156 行定义宏 `CUTLASS_ARCH_MMA_SM120_SUPPORTED`，用于控制后续的编译路径。

```cpp
  #if (!defined(CUTLASS_ARCH_MMA_SM120_ENABLED) && defined(__CUDA_ARCH__) && __CUDA_ARCH__ == 1200)
```
**EN**: Line 157 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 157 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    #define CUTLASS_ARCH_MMA_SM120_ENABLED 1
```
**EN**: Line 158 defines macro `CUTLASS_ARCH_MMA_SM120_ENABLED` to steer later compilation paths.
**CN**: 第 158 行定义宏 `CUTLASS_ARCH_MMA_SM120_ENABLED`，用于控制后续的编译路径。

```cpp

```
**EN**: Line 159 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 159 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    #if (!defined(CUTLASS_ARCH_MMA_SM120A_ENABLED) && defined(__CUDA_ARCH_FEAT_SM120_ALL))
```
**EN**: Line 160 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 160 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
      #define CUTLASS_ARCH_MMA_SM120A_ENABLED 1
```
**EN**: Line 161 defines macro `CUTLASS_ARCH_MMA_SM120A_ENABLED` to steer later compilation paths.
**CN**: 第 161 行定义宏 `CUTLASS_ARCH_MMA_SM120A_ENABLED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 162 ends the current conditional-compilation block.
**CN**: 第 162 行结束当前条件编译块。

```cpp

```
**EN**: Line 163 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 163 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    // SM120f
```
**EN**: Line 164 is an inline comment that explains the nearby code or intent.
**CN**: 第 164 行是行内注释，用于解释附近代码或设计意图。

```cpp
    #if (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9))
```
**EN**: Line 165 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 165 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    #define CUTLASS_ARCH_MMA_SM120F_SUPPORTED 1
```
**EN**: Line 166 defines macro `CUTLASS_ARCH_MMA_SM120F_SUPPORTED` to steer later compilation paths.
**CN**: 第 166 行定义宏 `CUTLASS_ARCH_MMA_SM120F_SUPPORTED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 167 ends the current conditional-compilation block.
**CN**: 第 167 行结束当前条件编译块。

```cpp

```
**EN**: Line 168 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 168 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    #if (!defined(CUTLASS_ARCH_MMA_SM120F_ENABLED) && CUDA_ARCH_FAMILY(1200))
```
**EN**: Line 169 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 169 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
      #define CUTLASS_ARCH_MMA_SM120F_ENABLED CUTLASS_ARCH_MMA_SM120F_SUPPORTED
```
**EN**: Line 170 defines macro `CUTLASS_ARCH_MMA_SM120F_ENABLED` to steer later compilation paths.
**CN**: 第 170 行定义宏 `CUTLASS_ARCH_MMA_SM120F_ENABLED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 171 ends the current conditional-compilation block.
**CN**: 第 171 行结束当前条件编译块。

```cpp
  #endif
```
**EN**: Line 172 ends the current conditional-compilation block.
**CN**: 第 172 行结束当前条件编译块。

```cpp
#endif
```
**EN**: Line 173 ends the current conditional-compilation block.
**CN**: 第 173 行结束当前条件编译块。

```cpp

```
**EN**: Line 174 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 174 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
// SM103 and SM103a
```
**EN**: Line 175 is an inline comment that explains the nearby code or intent.
**CN**: 第 175 行是行内注释，用于解释附近代码或设计意图。

```cpp
#if !CUTLASS_CLANG_CUDA && (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9))
```
**EN**: Line 176 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 176 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
  #define CUTLASS_ARCH_MMA_SM103_SUPPORTED 1
```
**EN**: Line 177 defines macro `CUTLASS_ARCH_MMA_SM103_SUPPORTED` to steer later compilation paths.
**CN**: 第 177 行定义宏 `CUTLASS_ARCH_MMA_SM103_SUPPORTED`，用于控制后续的编译路径。

```cpp
  #if (!defined(CUTLASS_ARCH_MMA_SM103_ENABLED) && defined(__CUDA_ARCH__) && __CUDA_ARCH__ == 1030)
```
**EN**: Line 178 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 178 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    #define CUTLASS_ARCH_MMA_SM103_ENABLED 1
```
**EN**: Line 179 defines macro `CUTLASS_ARCH_MMA_SM103_ENABLED` to steer later compilation paths.
**CN**: 第 179 行定义宏 `CUTLASS_ARCH_MMA_SM103_ENABLED`，用于控制后续的编译路径。

```cpp

```
**EN**: Line 180 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 180 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    #if (!defined(CUTLASS_ARCH_MMA_SM103A_ENABLED) && defined(__CUDA_ARCH_FEAT_SM103_ALL))
```
**EN**: Line 181 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 181 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
      #define CUTLASS_ARCH_MMA_SM103A_ENABLED 1
```
**EN**: Line 182 defines macro `CUTLASS_ARCH_MMA_SM103A_ENABLED` to steer later compilation paths.
**CN**: 第 182 行定义宏 `CUTLASS_ARCH_MMA_SM103A_ENABLED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 183 ends the current conditional-compilation block.
**CN**: 第 183 行结束当前条件编译块。

```cpp

```
**EN**: Line 184 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 184 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    // SM103f
```
**EN**: Line 185 is an inline comment that explains the nearby code or intent.
**CN**: 第 185 行是行内注释，用于解释附近代码或设计意图。

```cpp
    #if (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9))
```
**EN**: Line 186 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 186 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    #define CUTLASS_ARCH_MMA_SM103F_SUPPORTED 1
```
**EN**: Line 187 defines macro `CUTLASS_ARCH_MMA_SM103F_SUPPORTED` to steer later compilation paths.
**CN**: 第 187 行定义宏 `CUTLASS_ARCH_MMA_SM103F_SUPPORTED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 188 ends the current conditional-compilation block.
**CN**: 第 188 行结束当前条件编译块。

```cpp

```
**EN**: Line 189 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 189 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    #if (!defined(CUTLASS_ARCH_MMA_SM103F_ENABLED) && CUDA_ARCH_FAMILY(1030))
```
**EN**: Line 190 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 190 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
      #define CUTLASS_ARCH_MMA_SM103F_ENABLED CUTLASS_ARCH_MMA_SM103F_SUPPORTED
```
**EN**: Line 191 defines macro `CUTLASS_ARCH_MMA_SM103F_ENABLED` to steer later compilation paths.
**CN**: 第 191 行定义宏 `CUTLASS_ARCH_MMA_SM103F_ENABLED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 192 ends the current conditional-compilation block.
**CN**: 第 192 行结束当前条件编译块。

```cpp
  #endif
```
**EN**: Line 193 ends the current conditional-compilation block.
**CN**: 第 193 行结束当前条件编译块。

```cpp
#endif
```
**EN**: Line 194 ends the current conditional-compilation block.
**CN**: 第 194 行结束当前条件编译块。

```cpp

```
**EN**: Line 195 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 195 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
// SM121 and SM121a
```
**EN**: Line 196 is an inline comment that explains the nearby code or intent.
**CN**: 第 196 行是行内注释，用于解释附近代码或设计意图。

```cpp
#if !CUTLASS_CLANG_CUDA && (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9))
```
**EN**: Line 197 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 197 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
  #define CUTLASS_ARCH_MMA_SM121_SUPPORTED 1
```
**EN**: Line 198 defines macro `CUTLASS_ARCH_MMA_SM121_SUPPORTED` to steer later compilation paths.
**CN**: 第 198 行定义宏 `CUTLASS_ARCH_MMA_SM121_SUPPORTED`，用于控制后续的编译路径。

```cpp
  #if (!defined(CUTLASS_ARCH_MMA_SM121_ENABLED) && defined(__CUDA_ARCH__) && __CUDA_ARCH__ == 1210)
```
**EN**: Line 199 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 199 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    #define CUTLASS_ARCH_MMA_SM121_ENABLED 1
```
**EN**: Line 200 defines macro `CUTLASS_ARCH_MMA_SM121_ENABLED` to steer later compilation paths.
**CN**: 第 200 行定义宏 `CUTLASS_ARCH_MMA_SM121_ENABLED`，用于控制后续的编译路径。

```cpp

```
**EN**: Line 201 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 201 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    #if (!defined(CUTLASS_ARCH_MMA_SM121A_ENABLED) &&\
```
**EN**: Line 202 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 202 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
         (defined(__CUDA_ARCH_FEAT_SM121_ALL) || CUDA_ARCH_CONDITIONAL(1210)))
```
**EN**: Line 203 contributes to the surrounding declaration or implementation logic.
**CN**: 第 203 行为周围的声明或实现逻辑提供组成部分。

```cpp
      #define CUTLASS_ARCH_MMA_SM121A_ENABLED 1
```
**EN**: Line 204 defines macro `CUTLASS_ARCH_MMA_SM121A_ENABLED` to steer later compilation paths.
**CN**: 第 204 行定义宏 `CUTLASS_ARCH_MMA_SM121A_ENABLED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 205 ends the current conditional-compilation block.
**CN**: 第 205 行结束当前条件编译块。

```cpp

```
**EN**: Line 206 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 206 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    // SM121f
```
**EN**: Line 207 is an inline comment that explains the nearby code or intent.
**CN**: 第 207 行是行内注释，用于解释附近代码或设计意图。

```cpp
    #if (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9))
```
**EN**: Line 208 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 208 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
    #define CUTLASS_ARCH_MMA_SM121F_SUPPORTED 1
```
**EN**: Line 209 defines macro `CUTLASS_ARCH_MMA_SM121F_SUPPORTED` to steer later compilation paths.
**CN**: 第 209 行定义宏 `CUTLASS_ARCH_MMA_SM121F_SUPPORTED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 210 ends the current conditional-compilation block.
**CN**: 第 210 行结束当前条件编译块。

```cpp

```
**EN**: Line 211 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 211 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
    #if (!defined(CUTLASS_ARCH_MMA_SM121F_ENABLED) && CUDA_ARCH_FAMILY(1210))
```
**EN**: Line 212 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 212 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
      #define CUTLASS_ARCH_MMA_SM121F_ENABLED CUTLASS_ARCH_MMA_SM121F_SUPPORTED
```
**EN**: Line 213 defines macro `CUTLASS_ARCH_MMA_SM121F_ENABLED` to steer later compilation paths.
**CN**: 第 213 行定义宏 `CUTLASS_ARCH_MMA_SM121F_ENABLED`，用于控制后续的编译路径。

```cpp
    #endif
```
**EN**: Line 214 ends the current conditional-compilation block.
**CN**: 第 214 行结束当前条件编译块。

```cpp
  #endif
```
**EN**: Line 215 ends the current conditional-compilation block.
**CN**: 第 215 行结束当前条件编译块。

```cpp
#endif
```
**EN**: Line 216 ends the current conditional-compilation block.
**CN**: 第 216 行结束当前条件编译块。

```cpp

```
**EN**: Line 217 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 217 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp

```
**EN**: Line 218 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 218 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
#if (defined(CUTLASS_ARCH_MMA_SM100A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM100F_ENABLED) ||\
```
**EN**: Line 219 begins a conditional-compilation branch that depends on compiler, architecture, or feature macros.
**CN**: 第 219 行开始一个条件编译分支，其条件取决于编译器、架构或特性宏。

```cpp
     defined(CUTLASS_ARCH_MMA_SM101A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM101F_ENABLED) ||\
```
**EN**: Line 220 contributes to the surrounding declaration or implementation logic.
**CN**: 第 220 行为周围的声明或实现逻辑提供组成部分。

```cpp
     defined(CUTLASS_ARCH_MMA_SM103A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM103F_ENABLED) ||\
```
**EN**: Line 221 contributes to the surrounding declaration or implementation logic.
**CN**: 第 221 行为周围的声明或实现逻辑提供组成部分。

```cpp
     defined(CUTLASS_ARCH_MMA_SM110A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM110F_ENABLED) ||\
```
**EN**: Line 222 contributes to the surrounding declaration or implementation logic.
**CN**: 第 222 行为周围的声明或实现逻辑提供组成部分。

```cpp
     defined(CUTLASS_ARCH_MMA_SM120A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM120F_ENABLED) ||\
```
**EN**: Line 223 contributes to the surrounding declaration or implementation logic.
**CN**: 第 223 行为周围的声明或实现逻辑提供组成部分。

```cpp
     defined(CUTLASS_ARCH_MMA_SM121A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM121F_ENABLED))
```
**EN**: Line 224 contributes to the surrounding declaration or implementation logic.
**CN**: 第 224 行为周围的声明或实现逻辑提供组成部分。

```cpp
#  define CUTLASS_ARCH_CLC_ENABLED
```
**EN**: Line 225 is a preprocessor directive that shapes how this header is compiled.
**CN**: 第 225 行是一条预处理指令，用于塑造该头文件的编译方式。

```cpp
#endif
```
**EN**: Line 226 ends the current conditional-compilation block.
**CN**: 第 226 行结束当前条件编译块。

```cpp

```
**EN**: Line 227 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 227 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp

```
**EN**: Line 228 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 228 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Line 229 is a single-line documentation comment describing nearby code: //////////////////////////////////////////////////////////////////////////////////////////////
**CN**: 第 229 行是单行文档注释，用于描述附近代码：//////////////////////////////////////////////////////////////////////////////////////////////

```cpp

```
**EN**: Line 230 is intentionally blank, separating nearby declarations into readable sections.
**CN**: 第 230 行是刻意保留的空行，用于把相邻声明分成更易读的段落。

## Key Concepts / 关键概念

- Architecture-specific CUDA abstractions / 架构相关的 CUDA 抽象
- Conditional compilation and feature gating / 条件编译与特性门控

## Dependencies / 依赖项

- Direct includes / 直接包含:
  - `cutlass/platform/platform.h`
- Important macros / 重要宏:
  - `__CUDACC_VER_MAJOR__`
  - `__CUDACC_VER_MINOR__`
  - `CUTLASS_ARCH_MMA_SM90_SUPPORTED`
  - `CUTLASS_ARCH_MMA_SM90_ENABLED`
  - `__CUDA_ARCH__`
  - `CUTLASS_ARCH_MMA_SM90A_ENABLED`
  - `CUTLASS_ARCH_MMA_SPARSE_SM90_SUPPORTED`
  - `CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED`
  - `CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_ENABLED`
  - `CUTLASS_ARCH_MMA_SM90_F64_MMA_SUPPORTED`
  - `CUTLASS_ARCH_MMA_SM90_F64_MMA_ENABLED`
  - `CUTLASS_CLANG_CUDA`
  - `...` (31 more / 其余 31 项)
