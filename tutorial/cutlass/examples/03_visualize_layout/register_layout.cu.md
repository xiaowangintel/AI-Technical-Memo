# register_layout.cu — Code Analysis / 代码分析
**Source / 源文件**: `examples/03_visualize_layout/register_layout.cu`
**Purpose / 用途**: Registers concrete CUTLASS layout visualizers by command-line name / 按命令行名称注册具体 CUTLASS 布局可视化器
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-46 / 第 1-46 行
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

/*! \file
  \brief CUTLASS layout visualization example
*/

#include <map>
#include <memory>

#include "cutlass/layout/matrix.h"
#include "cutlass/layout/pitch_linear.h"
#include "cutlass/layout/tensor_op_multiplicand_sm70.h"
#include "cutlass/layout/tensor_op_multiplicand_sm75.h"
#include "cutlass/layout/tensor_op_multiplicand_sm80.h"

#include "visualize_layout.h"
#include "register_layout.h"
```
**EN**: License, file comment, STL map/unique_ptr support, CUTLASS layout headers for matrix, pitch-linear, and SM70/SM75/SM80 tensor-op layouts, plus local visualizer interfaces.
**CN**: 许可证、文件说明、STL map/unique_ptr 支持，CUTLASS 矩阵、pitch-linear、SM70/SM75/SM80 tensor-op 布局头，以及本地可视化接口。

### Lines 48-63 / 第 48-63 行
```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////

void RegisterLayouts(std::map<std::string, std::unique_ptr<VisualizeLayoutBase> > &layouts) {

  struct {
    char const *name;
    VisualizeLayoutBase *ptr;
  } layout_pairs[] = {

      {"PitchLinear", new VisualizeLayout<cutlass::layout::PitchLinear>},
      {"ColumnMajor", new VisualizeLayout<cutlass::layout::ColumnMajor>},
      {"RowMajor", new VisualizeLayout<cutlass::layout::RowMajor>},
      {"ColumnMajorInterleaved<4>",
       new VisualizeLayout<cutlass::layout::ColumnMajorInterleaved<4>>},
      {"RowMajorInterleaved<4>",
       new VisualizeLayout<cutlass::layout::RowMajorInterleaved<4>>},
```
**EN**: `RegisterLayouts` receives the registry map and starts an array of `{name, ptr}` pairs for basic layouts: pitch-linear, row/column-major, and interleaved row/column-major with interleave factor 4.
**CN**: `RegisterLayouts` 接收注册表 map，并创建 `{name, ptr}` 数组，首先注册基础布局：pitch-linear、行/列主序，以及 interleave factor 为 4 的行/列交错布局。

### Lines 64-88 / 第 64-88 行
```cpp
      // All Ampere/Turing H/Integer matrix multiply tensor core kernels uses the same swizzling
      // layout implementation with different templates.
      //
      // mma.sync.aligned.m8n8k128.s32.b1.b1.s32 Interleaved-256
      // mma.sync.aligned.m16n8k256.s32.b1.b1.s32 Interleaved-256
      {"TensorOpMultiplicand<1,256>",
       new VisualizeLayout<cutlass::layout::TensorOpMultiplicand<1, 256>>},
      // mma.sync.aligned.m8n8k128.s32.b1.b1.s32 TN kblock512
      // mma.sync.aligned.m16n8k256.s32.b1.b1.s32 TN kblock512
      {"TensorOpMultiplicand<1,512>",
       new VisualizeLayout<cutlass::layout::TensorOpMultiplicand<1, 512>>},
      // mma.sync.aligned.m16n8k256.s32.b1.b1.s32 TN kblock1024
      {"TensorOpMultiplicand<1,1024>",
       new VisualizeLayout<cutlass::layout::TensorOpMultiplicand<1, 1024>>},
      // Integer matrix multiply.int4 8832  Interleaved-64
      // Integer matrix multiply.int4 16864 Interleaved-64
      {"TensorOpMultiplicand<4,64>",
       new VisualizeLayout<cutlass::layout::TensorOpMultiplicand<4, 64>>},
      // Integer matrix multiply.int4 8832  TN kblock128
      // Integer matrix multiply.int4 16864 TN kblock128
      {"TensorOpMultiplicand<4,128>",
       new VisualizeLayout<cutlass::layout::TensorOpMultiplicand<4, 128>>},
      // Integer matrix multiply.int4 16864 TN kblock256
      {"TensorOpMultiplicand<4,256>",
       new VisualizeLayout<cutlass::layout::TensorOpMultiplicand<4, 256>>},
```
**EN**: The first tensor-op registrations cover binary and int4 multiplicand swizzles. `TensorOpMultiplicand<ElementSizeBits, Crosswise>` template parameters describe element bit width and the k-block/crosswise grouping used by tensor-core shared-memory layouts.
**CN**: 第一批 tensor-op 注册覆盖 binary 和 int4 乘数布局 swizzle。`TensorOpMultiplicand<ElementSizeBits, Crosswise>` 模板参数描述元素位宽以及 Tensor Core 共享内存布局使用的 k-block/crosswise 分组。

### Lines 89-114 / 第 89-114 行
```cpp
      // Integer matrix multiply 8816  Interleaved-32
      // Integer matrix multiply 16832 Interleaved-32
      {"TensorOpMultiplicand<8,32>",
       new VisualizeLayout<cutlass::layout::TensorOpMultiplicand<8, 32>>},
      // Integer matrix multiply 8816  TN kblock64
      // Integer matrix multiply 16832 TN kblock64
      {"TensorOpMultiplicand<8,64>",
       new VisualizeLayout<cutlass::layout::TensorOpMultiplicand<8, 64>>},
      // Integer matrix multiply 16832 TN kblock128
      {"TensorOpMultiplicand<8,128>",
       new VisualizeLayout<cutlass::layout::TensorOpMultiplicand<8, 128>>},
      // Matrix Multiply 1688  TN kblock32
      // Matrix multiply 16816 TN kblock32
      {"TensorOpMultiplicand<16,32>",
       new VisualizeLayout<cutlass::layout::TensorOpMultiplicand<16, 32>>},
      // Matrix multiply 1688  NT
      // Matrix multiply 16816 NT
      // Matrix multiply 16816 TN kblock64
      {"TensorOpMultiplicand<16,64>",
       new VisualizeLayout<cutlass::layout::TensorOpMultiplicand<16, 64>>},
      // Matrix multiply 1688.TF32 TN kblock16
      {"TensorOpMultiplicand<32,16>",
       new VisualizeLayout<cutlass::layout::TensorOpMultiplicand<32, 16>>},
      // Matrix multiply 1688.TF32 TN kblock32
      {"TensorOpMultiplicand<32,32>",
       new VisualizeLayout<cutlass::layout::TensorOpMultiplicand<32, 32>>},
```
**EN**: Additional `TensorOpMultiplicand` entries cover int8, FP16/BF16-like 16-bit, and TF32 32-bit variants with different k-block sizes matching mma instruction shapes noted in comments.
**CN**: 后续 `TensorOpMultiplicand` 条目覆盖 int8、类似 FP16/BF16 的 16 位、以及 TF32 32 位变体，不同 k-block 大小与注释中的 mma 指令形状匹配。

### Lines 115-138 / 第 115-138 行
```cpp
      // Matrix multiply 1688 NT
      {"TensorOpMultiplicandCongruous<32,32>",
       new VisualizeLayout<
           cutlass::layout::TensorOpMultiplicandCongruous<32, 32>>},
      // Matrix multiply 884 NT
      {"TensorOpMultiplicandCongruous<64,16>",
       new VisualizeLayout<
           cutlass::layout::TensorOpMultiplicandCongruous<64, 16>>},
      // Matrix multiply 884 TN
      {"TensorOpMultiplicand64bCrosswise",
       new VisualizeLayout<cutlass::layout::TensorOpMultiplicand64bCrosswise>},
      {"TensorOpMultiplicandCongruous<128,4>",
       new VisualizeLayout<
           cutlass::layout::TensorOpMultiplicandCongruous<128, 4>>},
      {"TensorOpMultiplicandCrosswise<128,4>",
       new VisualizeLayout<
           cutlass::layout::TensorOpMultiplicandCrosswise<128, 4>>},
      {"VoltaTensorOpMultiplicandCongruous<16>",
       new VisualizeLayout<
           cutlass::layout::VoltaTensorOpMultiplicandCongruous<16>>},
      {"VoltaTensorOpMultiplicandCrosswise<16,32>",
       new VisualizeLayout<
           cutlass::layout::VoltaTensorOpMultiplicandCrosswise<16, 32>>}
  };
```
**EN**: Congruous/crosswise aliases and Volta-specific layouts are registered. These classes encode architecture-specific shared-memory permutations for Tensor Core operands, so the same `VisualizeLayout` template can display their coordinate-to-offset mapping.
**CN**: 注册 congruous/crosswise 别名和 Volta 专用布局。这些类编码 Tensor Core 操作数的架构特定共享内存排列，因此同一个 `VisualizeLayout` 模板可显示其坐标到 offset 的映射。

### Lines 140-145 / 第 140-145 行
```cpp
  for (auto layout : layout_pairs) {
    layouts.emplace(std::string(layout.name), std::unique_ptr<VisualizeLayoutBase>(layout.ptr));
  }
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The loop transfers raw pointers into `std::unique_ptr<VisualizeLayoutBase>` and inserts them under string keys. Ownership is centralized in the registry map.
**CN**: 循环把裸指针转交给 `std::unique_ptr<VisualizeLayoutBase>`，并以字符串键插入；所有权集中在注册表 map 中。

---
## Key Concepts / 关键概念
- Layout registration decouples command-line strings from template instantiations that must be known at compile time. / 布局注册把命令行字符串与必须在编译期确定的模板实例解耦。
- Tensor-op multiplicand layouts model shared-memory swizzles required by `mma.sync` operand loading. / Tensor-op multiplicand 布局模拟 `mma.sync` 操作数加载所需的共享内存 swizzle。
- `VisualizeLayout<Layout>` type-erases through `VisualizeLayoutBase` for runtime selection. / `VisualizeLayout<Layout>` 通过 `VisualizeLayoutBase` 类型擦除以支持运行时选择。

## Dependencies / 依赖项
- `cutlass/layout/matrix.h` — row/column and interleaved matrix layouts
- `cutlass/layout/pitch_linear.h` — pitch-linear layout
- `cutlass/layout/tensor_op_multiplicand_sm70.h` — Volta tensor-op shared layouts
- `cutlass/layout/tensor_op_multiplicand_sm75.h` — Turing tensor-op shared layouts
- `cutlass/layout/tensor_op_multiplicand_sm80.h` — Ampere tensor-op shared layouts
- `visualize_layout.h` — templated visualizer
- `register_layout.h` — registry interface and base class
