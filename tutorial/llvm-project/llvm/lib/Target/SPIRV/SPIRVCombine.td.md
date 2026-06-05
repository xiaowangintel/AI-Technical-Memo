# SPIRVCombine.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVCombine.td`
- Repository: `llvm-project`
- Purpose (EN): SPIRVCombine support code for the LLVM target backend.
- 目的（中文）: 该文件使用 TableGen DSL 描述目标后端元数据，并驱动自动生成代码。
- Language: TableGen DSL (TableGen DSL noted below)
- Note: This file is written in TableGen DSL, so many records are declarative descriptions consumed by LLVM code generators.
- 说明：该文件使用 TableGen DSL 编写，因此许多记录都是供 LLVM 代码生成器消费的声明式描述。

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-43
```tablegen
 1: //=- SPIRVCombine.td - Define SPIRV Combine Rules -------------*-tablegen -*-=//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7:
 8: include "llvm/Target/GlobalISel/Combine.td"
 9:
10:
11: def vector_length_sub_to_distance_lowering : GICombineRule <
12:   (defs root:$root),
13:   (match (wip_match_opcode G_INTRINSIC):$root,
14:           [{ return Helper.matchLengthToDistance(*${root}); }]),
15:   (apply [{ Helper.applySPIRVDistance(*${root}); }])
16: >;
17:
18: def vector_select_to_faceforward_lowering : GICombineRule <
19:   (defs root:$root),
20:   (match (wip_match_opcode G_SELECT):$root,
21:           [{ return Helper.matchSelectToFaceForward(*${root}); }]),
22:   (apply [{ Helper.applySPIRVFaceForward(*${root}); }])
23: >;
24:
25: def matrix_transpose_lowering
26:     : GICombineRule<(defs root:$root),
27:                     (match (wip_match_opcode G_INTRINSIC):$root,
28:                         [{ return Helper.matchMatrixTranspose(*${root}); }]),
29:                     (apply [{ Helper.applyMatrixTranspose(*${root}); }])>;
30:
31: def matrix_multiply_lowering
32:     : GICombineRule<(defs root:$root),
33:                     (match (wip_match_opcode G_INTRINSIC):$root,
34:                         [{ return Helper.matchMatrixMultiply(*${root}); }]),
35:                     (apply [{ Helper.applyMatrixMultiply(*${root}); }])>;
36:
37: def SPIRVPreLegalizerCombiner
38:     : GICombiner<"SPIRVPreLegalizerCombinerImpl",
39:                  [vector_length_sub_to_distance_lowering,
40:                   vector_select_to_faceforward_lowering,
41:                   matrix_transpose_lowering, matrix_multiply_lowering]> {
42:   let CombineAllMethodName = "tryCombineAllImpl";
43: }
```
- EN: This range uses TableGen DSL to describe records such as vector_length_sub_to_distance_lowering, vector_select_to_faceforward_lowering, matrix_transpose_lowering, matrix_multiply_lowering; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 vector_length_sub_to_distance_lowering、vector_select_to_faceforward_lowering、matrix_transpose_lowering、matrix_multiply_lowering 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

## Key Concepts / 关键概念

- EN: TableGen files use declarative records that LLVM expands into generated C++ tables and helper code.
  - CN: TableGen 文件使用声明式记录，LLVM 会将其展开为生成的 C++ 表和辅助代码。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include vector_length_sub_to_distance_lowering, vector_select_to_faceforward_lowering, matrix_transpose_lowering, matrix_multiply_lowering, SPIRVPreLegalizerCombiner, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 vector_length_sub_to_distance_lowering, vector_select_to_faceforward_lowering, matrix_transpose_lowering, matrix_multiply_lowering, SPIRVPreLegalizerCombiner，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Target/GlobalISel/Combine.td`
- TableGen pipeline / TableGen 流水线: records in this file are consumed by LLVM TableGen emitters to produce generated lookup tables and helper code.
- TableGen 流水线说明：该文件中的记录会被 LLVM 的 TableGen 生成器消费，进而产出查找表和辅助代码。
