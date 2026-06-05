# options.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/03_visualize_layout/options.h`
**Purpose / 用途**: Defines command-line options for the layout visualization tool / 定义布局可视化工具的命令行选项
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-38 / 第 1-38 行
```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

#pragma once

#include <vector>
#include <iostream>

// Cutlass command line parser
#include "cutlass/util/command_line.h"
```
**EN**: License, include guard via `#pragma once`, standard containers/streams, and CUTLASS command-line parser dependency.
**CN**: 许可证、通过 `#pragma once` 的包含保护、标准容器/流，以及 CUTLASS 命令行解析器依赖。

### Lines 40-62 / 第 40-62 行
```cpp
class Options {
public:

  bool help;
  bool good;
  std::vector<int> extent;          ///< extent of tile to fill
  std::vector<int> stride;          ///< stride vector for layout function
  std::vector<int> output_shape;    ///< output shape
  int vectorize;                    ///< sequences of consecutive output elements are concatenated into a vector
                                    ///  if, and only if, they were consecutive in source memory

public:

  /// Options
  Options(): 
    help(false),
    good(true),
    extent({32, 8}),
    stride({32}),
    output_shape({16, 8}), 
    vectorize(1) { 

  }
```
**EN**: `Options` stores parsed state: help request, validity flag, layout extent, optional stride, output matrix shape, and vectorization width. The default constructor uses extent 32x8, stride 32, output shape 16x8, and scalar output.
**CN**: `Options` 保存解析状态：帮助请求、有效性标志、布局范围、可选 stride、输出矩阵形状和向量化宽度。默认构造函数使用 32x8 范围、stride 32、输出形状 16x8 和标量输出。

### Lines 64-83 / 第 64-83 行
```cpp
  /// Constructs from command line parser
  Options(cutlass::CommandLine const & cmd_line): help(false), good(true) {

    if (cmd_line.check_cmd_line_flag("help") ||
        cmd_line.check_cmd_line_flag("h")) {

      help = true;
    }

    if (cmd_line.check_cmd_line_flag("extent")) {
      cmd_line.get_cmd_line_arguments("extent", extent);
    }
    else {
      extent = {32, 8};
    }

    if (cmd_line.check_cmd_line_flag("stride")) {
      cmd_line.get_cmd_line_arguments("stride", stride);
    }
    
```
**EN**: The parser constructor detects `--help`/`-h`, reads `--extent` as a comma-delimited vector, and falls back to `{32,8}` when absent. `--stride` is optional; an empty stride later means use `Layout::packed`.
**CN**: 解析构造函数检测 `--help`/`-h`，把 `--extent` 读为逗号分隔 vector，缺省为 `{32,8}`。`--stride` 可选；空 stride 在后续表示使用 `Layout::packed`。

### Lines 84-99 / 第 84-99 行
```cpp
    int default_output_shape[] = {16, 8}; 

    if (cmd_line.check_cmd_line_flag("output-shape")) {
      cmd_line.get_cmd_line_arguments("output-shape", output_shape);
    }

    for (int i = int(output_shape.size()); i < 2; ++i) {
      output_shape.push_back(default_output_shape[i]);
    }

    if (cmd_line.check_cmd_line_flag("vectorize")) {
      cmd_line.get_cmd_line_argument("vectorize", vectorize);
    }
    else {
      vectorize = 1;
    }
```
**EN**: `--output-shape` may provide one or two dimensions; missing dimensions are filled from `{16,8}`. `--vectorize` selects how many consecutive offsets are printed together; default is 1.
**CN**: `--output-shape` 可以提供一维或二维；缺失维度由 `{16,8}` 补齐。`--vectorize` 选择连续 offset 合并打印的数量；默认 1。

### Lines 101-109 / 第 101-109 行
```cpp
    if (output_shape.front() % vectorize) {

      std::cerr << "Error: --vectorize=" << vectorize 
        << " must divide contiguous elements in --output-shape="
        << output_shape.at(0) << "," << output_shape.at(1) << std::endl;

      good = false;
    }
  }
```
**EN**: The output row width must be divisible by vector length so CSV cells align; failure sets `good=false` and emits a diagnostic.
**CN**: 输出行宽必须能被向量长度整除，以便 CSV 单元对齐；失败时设置 `good=false` 并输出诊断。

### Lines 111-121 / 第 111-121 行
```cpp
  /// Prints usage statement
  static void print_usage(std::ostream &out) {
    out
      << "  Options:\n"
      << "    --help                              Displays this help message.\n"
      << "    --extent=<extent>                   Specifies the layout-specific extent (as comma-delimited array).\n"
      << "    --stride=<stride>                   Specifies the layout-specific stride vector (comma-delimited array)\n"
      << "    --output-shape=<extent>             Specifies the dimensions of a row-major output matrix. \n"
      << "    --vectorize=<vector length>         If possible, vectorizes the output into vectors of consecutive elements\n";
  }
};
```
**EN**: `print_usage` centralizes help text for all supported visualization flags.
**CN**: `print_usage` 集中输出所有可视化标志的帮助文本。

---
## Key Concepts / 关键概念
- Options are intentionally layout-agnostic; rank validation happens in the templated visualizer. / Options 有意保持布局无关；rank 校验在模板化可视化器中完成。
- An empty stride vector encodes “packed layout,” while a present vector constructs `Layout(stride)`. / 空 stride vector 表示“packed layout”；存在的 vector 则构造 `Layout(stride)`。

## Dependencies / 依赖项
- `cutlass/util/command_line.h` — parses flags and comma-delimited integer arguments
- `<vector>` — dynamic rank-sized options
- `<iostream>` — usage and error streams
