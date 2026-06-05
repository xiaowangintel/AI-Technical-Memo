# VVPInstrPatternsVec.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VVPInstrPatternsVec.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines backend metadata in TableGen DSL that is later expanded into generated tables and matcher code.
  - **CN**: 使用 TableGen DSL 定义后端元数据，随后会展开为生成的表和匹配代码。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```tablegen
//===----------- VVPInstrPatternsVec.td - VVP_* SDNode patterns -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes how VVP_* SDNodes are lowered to machine instructions.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 13-36
```tablegen
//===----------------------------------------------------------------------===//
//
// VVP SDNode definitions.
//
//===----------------------------------------------------------------------===//
include "VVPInstrInfo.td"

multiclass VectorStore<ValueType DataVT,
    ValueType PtrVT, ValueType MaskVT,
    string STWithMask, string STNoMask> {
  // Unmasked (imm stride).
  def : Pat<(vvp_store
               DataVT:$val, PtrVT:$addr,
               (i64 simm7:$stride), (MaskVT true_mask), i32:$avl),
            (!cast<Instruction>(STNoMask#"irvl")
               (LO7 $stride), $addr, $val, $avl)>;
  // Unmasked.
  def : Pat<(vvp_store
               DataVT:$val, PtrVT:$addr,
               i64:$stride, (MaskVT true_mask), i32:$avl),
            (!cast<Instruction>(STNoMask#"rrvl")
               $stride, $addr, $val, $avl)>;
  // Masked (imm stride).
  def : Pat<(vvp_store
```
- **EN**: Imports shared TableGen building blocks such as `VVPInstrInfo.td` so this file can reuse operand, format, or helper definitions.
- **CN**: 导入共享的 TableGen 构件，例如 `VVPInstrInfo.td`，以便复用操作数、格式或辅助定义。

### Lines 37-48
```tablegen
               DataVT:$val, PtrVT:$addr,
               (i64 simm7:$stride), MaskVT:$mask, i32:$avl),
            (!cast<Instruction>(STWithMask#"irvml")
               (LO7 $stride), $addr, $val, $mask, $avl)>;
  // Masked.
  def : Pat<(vvp_store
               DataVT:$val, PtrVT:$addr,
               i64:$stride, MaskVT:$mask, i32:$avl),
            (!cast<Instruction>(STWithMask#"rrvml")
               $stride, $addr, $val, $mask, $avl)>;
}

```
- **EN**: Implements logic around `cast<Instruction>`, `Pat<`.
- **CN**: 围绕 `cast<Instruction>`, `Pat<` 实现具体逻辑。

### Lines 49-72
```tablegen
defm : VectorStore<v256f64, i64, v256i1, "VST",  "VST">;
defm : VectorStore<v256i64, i64, v256i1, "VST",  "VST">;
defm : VectorStore<v256f32, i64, v256i1, "VSTU", "VSTU">;
defm : VectorStore<v256i32, i64, v256i1, "VSTL", "VSTL">;

multiclass VectorLoad<ValueType DataVT,
    ValueType PtrVT, ValueType MaskVT,
    string GTWithMask, string LDNoMask> {
  // Unmasked (imm stride).
  def : Pat<(DataVT (vvp_load
               PtrVT:$addr, (i64 simm7:$stride),
               (MaskVT true_mask), i32:$avl)),
            (!cast<Instruction>(LDNoMask#"irl")
               (LO7 $stride), $addr, $avl)>;
  // Unmasked.
  def : Pat<(DataVT (vvp_load
               PtrVT:$addr, i64:$stride,
               (MaskVT true_mask), i32:$avl)),
            (!cast<Instruction>(LDNoMask#"rrl")
               $stride, PtrVT:$addr, $avl)>;
  // Masked (imm stride).
  def : Pat<(DataVT (vvp_load
               PtrVT:$addr, (i64 simm7:$stride),
               MaskVT:$mask, i32:$avl)),
```
- **EN**: Declares TableGen records such as `VectorLoad` for the backend description.
- **CN**: 为后端描述声明了 `VectorLoad` 等 TableGen 记录。

### Lines 73-91
```tablegen
            (!cast<Instruction>(GTWithMask#"vizml")
               (VADDULrvml $addr,
                  (VMULULivml (LO7 $stride), (VSEQl $avl), $mask, $avl),
                  $mask, $avl),
               0, 0,
               $mask,
               $avl)>;
  // Masked.
  def : Pat<(DataVT (vvp_load
               PtrVT:$addr, i64:$stride, MaskVT:$mask, i32:$avl)),
            (!cast<Instruction>(GTWithMask#"vizml")
               (VADDULrvml $addr,
                  (VMULULrvml $stride, (VSEQl $avl), $mask, $avl),
                  $mask, $avl),
               0, 0,
               $mask,
               $avl)>;
}

```
- **EN**: Implements logic around `cast<Instruction>`, `VMULULivml`, `Pat<`.
- **CN**: 围绕 `cast<Instruction>`, `VMULULivml`, `Pat<` 实现具体逻辑。

### Lines 92-109
```tablegen
defm : VectorLoad<v256f64, i64, v256i1, "VGT",    "VLD">;
defm : VectorLoad<v256i64, i64, v256i1, "VGT",    "VLD">;
defm : VectorLoad<v256f32, i64, v256i1, "VGTU",   "VLDU">;
defm : VectorLoad<v256i32, i64, v256i1, "VGTLZX", "VLDLZX">;

// Vector Gather and scatter
multiclass VectorGather<ValueType DataVT,
    ValueType PtrVT, ValueType MaskVT,
    string GTPrefix> {
  // Unmasked.
  def : Pat<(DataVT (vvp_gather
               PtrVT:$addr, (MaskVT true_mask), i32:$avl)),
            (!cast<Instruction>(GTPrefix#"vizl") $addr, 0, 0, $avl)>;
  // Masked.
  def : Pat<(DataVT (vvp_gather PtrVT:$addr, MaskVT:$mask, i32:$avl)),
            (!cast<Instruction>(GTPrefix#"vizml") $addr, 0, 0, $mask, $avl)>;
}

```
- **EN**: Declares TableGen records such as `VectorGather` for the backend description.
- **CN**: 为后端描述声明了 `VectorGather` 等 TableGen 记录。

### Lines 110-127
```tablegen
defm : VectorGather<v256f64, v256i64, v256i1, "VGT">;
defm : VectorGather<v256i64, v256i64, v256i1, "VGT">;
defm : VectorGather<v256f32, v256i64, v256i1, "VGTU">;
defm : VectorGather<v256i32, v256i64, v256i1, "VGTLZX">;

multiclass VectorScatter<ValueType DataVT,
    ValueType PtrVT, ValueType MaskVT,
    string SCPrefix> {
  // Unmasked.
  def : Pat<(vvp_scatter
               DataVT:$data, PtrVT:$addr, (MaskVT true_mask), i32:$avl),
            (!cast<Instruction>(SCPrefix#"vizvl") $addr, 0, 0, $data, $avl)>; 
  // Masked.
  def : Pat<(vvp_scatter
               DataVT:$data, PtrVT:$addr, MaskVT:$mask, i32:$avl),
            (!cast<Instruction>(SCPrefix#"vizvml") $addr, 0, 0, $data, $mask, $avl)>;
}

```
- **EN**: Declares TableGen records such as `VectorScatter` for the backend description.
- **CN**: 为后端描述声明了 `VectorScatter` 等 TableGen 记录。

### Lines 128-140
```tablegen
defm : VectorScatter<v256f64, v256i64, v256i1, "VSC">;
defm : VectorScatter<v256i64, v256i64, v256i1, "VSC">;
defm : VectorScatter<v256f32, v256i64, v256i1, "VSCU">;
defm : VectorScatter<v256i32, v256i64, v256i1, "VSCL">;


/// FNEG {
// Directly modify the sign bit to flip the sign.

// Set sign bits in a pack of <2 x f32>.
def packed_fneg_imm     : OutPatFrag<(ins ),
                          (i64 (SLLri (i64 (ORim 1, (i32 32))), 31))>;

```
- **EN**: Declares TableGen records such as `packed_fneg_imm`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `packed_fneg_imm` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 141-153
```tablegen

multiclass FNeg<ValueType DataVT> {
  // Masked with select.
  def : Pat<(vvp_select (vvp_fneg DataVT:$vx, (v256i1 srcvalue), (i32 srcvalue)),
                        DataVT:$vfalse,
                        v256i1:$mask,
                        i32:$avl),
            (VXORmvml_v (i32 1), $vx, $mask, $avl, $vfalse)>;

  // Unmasked.
  def : Pat<(vvp_fneg DataVT:$vx, (v256i1 true_mask), i32:$avl),
            (VXORmvl (i32 1), $vx, $avl)>;

```
- **EN**: Declares TableGen records such as `FNeg` for the backend description.
- **CN**: 为后端描述声明了 `FNeg` 等 TableGen 记录。

### Lines 154-170
```tablegen
  // Masked.
  def : Pat<(vvp_fneg DataVT:$vx, v256i1:$mask, i32:$avl),
            (VXORmvml (i32 1), $vx, $mask, $avl)>;
}

defm: FNeg<v256f32>;
defm: FNeg<v256f64>;

///// Packed FNeg /////

// Masked with select.
def : Pat<(vvp_select (vvp_fneg v512f32:$vx, (v512i1 srcvalue), (i32 srcvalue)),
                      v512f32:$vfalse,
                      v512i1:$mask,
                      i32:$avl),
          (v512f32 (PVXORrvml_v (packed_fneg_imm ), $vx, $mask, $avl, $vfalse))>;

```
- **EN**: Implements logic around `Pat<`, `VXORmvml`, `v512f32`.
- **CN**: 围绕 `Pat<`, `VXORmvml`, `v512f32` 实现具体逻辑。

### Lines 171-194
```tablegen
// Unmasked.
def : Pat<(vvp_fneg v512f32:$vx, (v512i1 true_mask), i32:$avl),
          (v512f32 (PVXORrvl (packed_fneg_imm ), $vx, $avl))>;

// Masked.
def : Pat<(vvp_fneg v512f32:$vx, v512i1:$mask, i32:$avl),
          (v512f32 (PVXORrvml (packed_fneg_imm ), $vx, $mask, $avl))>;

/// } FNEG

multiclass Binary_rv<SDPatternOperator OpNode,
    ValueType ScalarVT, ValueType DataVT,
    ValueType MaskVT, string OpBaseName> {
  // Masked with passthru, broadcast.
  def : Pat<(vvp_select
                (OpNode
                    (any_broadcast ScalarVT:$sx),
                    DataVT:$vy,
                    (MaskVT srcvalue),
                    (i32 srcvalue)),
                DataVT:$vfalse,
                MaskVT:$mask,
                i32:$pivot),
            (!cast<Instruction>(OpBaseName#"rvml_v")
```
- **EN**: Declares TableGen records such as `Binary_rv`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Binary_rv` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 195-216
```tablegen
                ScalarVT:$sx,
                $vy,
                $mask,
                $pivot,
                $vfalse)>;

  // Unmasked, broadcast.
  def : Pat<(OpNode
                (any_broadcast ScalarVT:$sx), DataVT:$vy,
                (MaskVT true_mask),
                i32:$avl),
            (!cast<Instruction>(OpBaseName#"rvl")
                ScalarVT:$sx, $vy, $avl)>;
  // Masked, broadcast.
  def : Pat<(OpNode
                (any_broadcast ScalarVT:$sx), DataVT:$vy,
                MaskVT:$mask,
                i32:$avl),
            (!cast<Instruction>(OpBaseName#"rvml")
                ScalarVT:$sx, $vy, $mask, $avl)>;
}

```
- **EN**: Implements logic around `Pat<`, `cast<Instruction>`.
- **CN**: 围绕 `Pat<`, `cast<Instruction>` 实现具体逻辑。

### Lines 217-236
```tablegen
multiclass Binary_vr<SDPatternOperator OpNode,
    ValueType ScalarVT, ValueType DataVT,
    ValueType MaskVT, string OpBaseName> {
  // Masked with passthru, broadcast.
  def : Pat<(vvp_select
                (OpNode
                    DataVT:$vx,
                    (any_broadcast ScalarVT:$sy),
                    (MaskVT srcvalue),
                    (i32 srcvalue)),
                DataVT:$vfalse,
                MaskVT:$mask,
                i32:$pivot),
            (!cast<Instruction>(OpBaseName#"vrml_v")
                $vx,
                ScalarVT:$sy,
                $mask,
                $pivot,
                $vfalse)>;

```
- **EN**: Declares TableGen records such as `Binary_vr`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Binary_vr` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 237-252
```tablegen
  // Unmasked, broadcast.
  def : Pat<(OpNode
                DataVT:$vx, (any_broadcast ScalarVT:$sy),
                (MaskVT true_mask),
                i32:$avl),
            (!cast<Instruction>(OpBaseName#"vrl")
                $vx, ScalarVT:$sy, $avl)>;
  // Masked, broadcast.
  def : Pat<(OpNode
                DataVT:$vx, (any_broadcast ScalarVT:$sy),
                MaskVT:$mask,
                i32:$avl),
            (!cast<Instruction>(OpBaseName#"vrml")
                $vx, ScalarVT:$sy, $mask, $avl)>;
}

```
- **EN**: Implements logic around `Pat<`, `cast<Instruction>`.
- **CN**: 围绕 `Pat<`, `cast<Instruction>` 实现具体逻辑。

### Lines 253-272
```tablegen
multiclass Binary_vv<SDPatternOperator OpNode,
    ValueType DataVT,
    ValueType MaskVT, string OpBaseName> {
  // Masked with passthru, broadcast.
  def : Pat<(vvp_select
                (OpNode
                    DataVT:$vx,
                    DataVT:$vy,
                    (MaskVT srcvalue),
                    (i32 srcvalue)),
                DataVT:$vfalse,
                MaskVT:$mask,
                i32:$pivot),
            (!cast<Instruction>(OpBaseName#"vvml_v")
                $vx,
                $vy,
                $mask,
                $pivot,
                $vfalse)>;

```
- **EN**: Declares TableGen records such as `Binary_vv`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Binary_vv` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 273-292
```tablegen
  // Masked with select.
  // TODO

  // Unmasked.
  def : Pat<(OpNode
                DataVT:$vx, DataVT:$vy,
                (MaskVT true_mask),
                i32:$avl),
            (!cast<Instruction>(OpBaseName#"vvl")
                $vx, $vy, $avl)>;

  // Masked.
  def : Pat<(OpNode
                DataVT:$vx, DataVT:$vy,
                MaskVT:$mask,
                i32:$avl),
            (!cast<Instruction>(OpBaseName#"vvml")
                $vx, $vy, $mask, $avl)>;
}

```
- **EN**: Implements logic around `Pat<`, `cast<Instruction>`.
- **CN**: 围绕 `Pat<`, `cast<Instruction>` 实现具体逻辑。

### Lines 293-308
```tablegen
multiclass Binary_rv_vv<
    SDPatternOperator OpNode,
    ValueType ScalarVT, ValueType DataVT, ValueType MaskVT,
    string OpBaseName> {
  defm : Binary_rv<OpNode, ScalarVT, DataVT, MaskVT, OpBaseName>;
  defm : Binary_vv<OpNode, DataVT, MaskVT, OpBaseName>;
}

multiclass Binary_vr_vv<
    SDPatternOperator OpNode,
    ValueType ScalarVT, ValueType DataVT, ValueType MaskVT,
    string OpBaseName> {
  defm : Binary_vr<OpNode, ScalarVT, DataVT, MaskVT, OpBaseName>;
  defm : Binary_vv<OpNode, DataVT, MaskVT, OpBaseName>;
}

```
- **EN**: Declares TableGen records such as `Binary_rv_vv`, `Binary_vr_vv`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Binary_rv_vv`, `Binary_vr_vv` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 309-329
```tablegen
multiclass Binary_rv_vr_vv<
    SDPatternOperator OpNode,
    ValueType ScalarVT, ValueType DataVT, ValueType MaskVT,
    string OpBaseName> {
  defm : Binary_rv<OpNode, ScalarVT, DataVT, MaskVT, OpBaseName>;
  defm : Binary_vr_vv<OpNode, ScalarVT, DataVT, MaskVT, OpBaseName>;
}

// Expand both 64bit and 32 bit variant (256 elements)
multiclass Binary_rv_vv_ShortLong<
    SDPatternOperator OpNode,
    ValueType LongScalarVT, ValueType LongDataVT, string LongOpBaseName,
    ValueType ShortScalarVT, ValueType ShortDataVT, string ShortOpBaseName> {
  defm : Binary_rv_vv<OpNode,
                      LongScalarVT, LongDataVT, v256i1,
                      LongOpBaseName>;
  defm : Binary_rv_vv<OpNode,
                      ShortScalarVT, ShortDataVT, v256i1,
                      ShortOpBaseName>;
}

```
- **EN**: Declares TableGen records such as `Binary_rv_vr_vv`, `Binary_rv_vv_ShortLong`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Binary_rv_vr_vv`, `Binary_rv_vv_ShortLong` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 330-341
```tablegen
multiclass Binary_vr_vv_ShortLong<
    SDPatternOperator OpNode,
    ValueType LongScalarVT, ValueType LongDataVT, string LongOpBaseName,
    ValueType ShortScalarVT, ValueType ShortDataVT, string ShortOpBaseName> {
  defm : Binary_vr_vv<OpNode,
                      LongScalarVT, LongDataVT, v256i1,
                      LongOpBaseName>;
  defm : Binary_vr_vv<OpNode,
                      ShortScalarVT, ShortDataVT, v256i1,
                      ShortOpBaseName>;
}

```
- **EN**: Declares TableGen records such as `Binary_vr_vv_ShortLong`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Binary_vr_vv_ShortLong` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 342-353
```tablegen
multiclass Binary_rv_vr_vv_ShortLong<
    SDPatternOperator OpNode,
    ValueType LongScalarVT, ValueType LongDataVT, string LongOpBaseName,
    ValueType ShortScalarVT, ValueType ShortDataVT, string ShortOpBaseName> {
  defm : Binary_rv_vr_vv<OpNode,
                      LongScalarVT, LongDataVT, v256i1,
                      LongOpBaseName>;
  defm : Binary_rv_vr_vv<OpNode,
                      ShortScalarVT, ShortDataVT, v256i1,
                      ShortOpBaseName>;
}

```
- **EN**: Declares TableGen records such as `Binary_rv_vr_vv_ShortLong`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Binary_rv_vr_vv_ShortLong` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 354-377
```tablegen
defm : Binary_rv_vv_ShortLong<c_vvp_add,
                              i64, v256i64, "VADDSL",
                              i32, v256i32, "VADDSWSX">;
defm : Binary_rv_vv_ShortLong<vvp_sub,
                              i64, v256i64, "VSUBSL",
                              i32, v256i32, "VSUBSWSX">;
defm : Binary_rv_vv_ShortLong<c_vvp_mul,
                              i64, v256i64, "VMULSL",
                              i32, v256i32, "VMULSWSX">;
defm : Binary_rv_vr_vv_ShortLong<vvp_sdiv,
                              i64, v256i64, "VDIVSL",
                              i32, v256i32, "VDIVSWSX">;
defm : Binary_rv_vr_vv_ShortLong<vvp_udiv,
                              i64, v256i64, "VDIVUL",
                              i32, v256i32, "VDIVUW">;
defm : Binary_rv_vv_ShortLong<c_vvp_and,
                              i64, v256i64, "VAND",
                              i32, v256i32, "PVANDLO">;
defm : Binary_rv_vv_ShortLong<c_vvp_or,
                              i64, v256i64, "VOR",
                              i32, v256i32, "PVORLO">;
defm : Binary_rv_vv_ShortLong<c_vvp_xor,
                              i64, v256i64, "VXOR",
                              i32, v256i32, "PVXORLO">;
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 378-400
```tablegen
defm : Binary_vr_vv_ShortLong<vvp_shl,
                              i64, v256i64, "VSLL",
                              i32, v256i32, "PVSLLLO">;
defm : Binary_vr_vv_ShortLong<vvp_sra,
                              i64, v256i64, "VSRAL",
                              i32, v256i32, "PVSRALO">;
defm : Binary_vr_vv_ShortLong<vvp_srl,
                              i64, v256i64, "VSRL",
                              i32, v256i32, "PVSRLLO">;

defm : Binary_rv_vv_ShortLong<c_vvp_fadd,
                              f64, v256f64, "VFADDD",
                              f32, v256f32, "PVFADDUP">;
defm : Binary_rv_vv_ShortLong<c_vvp_fmul,
                              f64, v256f64, "VFMULD",
                              f32, v256f32, "PVFMULUP">;
defm : Binary_rv_vv_ShortLong<vvp_fsub,
                              f64, v256f64, "VFSUBD",
                              f32, v256f32, "PVFSUBUP">;
defm : Binary_rv_vr_vv_ShortLong<vvp_fdiv,
                              f64, v256f64, "VFDIVD",
                              f32, v256f32, "VFDIVS">;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 401-418
```tablegen
defm : Binary_rv_vv<c_vvp_and,
                    i64, v512i32, v512i1, "PVAND">;
defm : Binary_rv_vv<c_vvp_or,
                    i64, v512i32, v512i1, "PVOR">;
defm : Binary_rv_vv<c_vvp_xor,
                    i64, v512i32, v512i1, "PVXOR">;

defm : Binary_rv_vv<c_vvp_add,
                    i64, v512i32, v512i1, "PVADDU">;
defm : Binary_rv_vv<vvp_sub,
                    i64, v512i32, v512i1, "PVSUBU">;
defm : Binary_vr_vv<vvp_srl,
                    i64, v512i32, v512i1, "PVSRL">;
defm : Binary_vr_vv<vvp_sra,
                    i64, v512i32, v512i1, "PVSRA">;
defm : Binary_vr_vv<vvp_shl,
                    i64, v512i32, v512i1, "PVSLL">;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 419-438
```tablegen
defm : Binary_rv_vv<c_vvp_fadd,
                    i64, v512f32, v512i1, "PVFADD">;
defm : Binary_rv_vv<c_vvp_fmul,
                    i64, v512f32, v512i1, "PVFMUL">;
defm : Binary_rv_vv<vvp_fsub,
                    i64, v512f32, v512i1, "PVFSUB">;

multiclass Ternary_vvv<
    SDPatternOperator OpNode, ValueType DataVT,
    ValueType MaskVT, string OpBaseName> {
  // Masked with passthru.
  def : Pat<(vvp_select
              (OpNode DataVT:$vx, DataVT:$vy, DataVT:$vz,
                (MaskVT srcvalue), (i32 srcvalue)),
              DataVT:$vfalse,
              MaskVT:$mask,
              i32:$avl),
            (!cast<Instruction>(OpBaseName#"vvvml_v")
              $vx, $vy, $vz, $mask, $avl, $vfalse)>;

```
- **EN**: Declares TableGen records such as `Ternary_vvv`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Ternary_vvv` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 439-451
```tablegen
  // Unmasked.
  def : Pat<(OpNode DataVT:$vx, DataVT:$vy, DataVT:$vz,
              (MaskVT true_mask), i32:$avl),
            (!cast<Instruction>(OpBaseName#"vvvl")
              $vx, $vy, $vz, $avl)>;

  // Masked.
  def : Pat<(OpNode DataVT:$vx, DataVT:$vy, DataVT:$vz,
              MaskVT:$mask, i32:$avl),
            (!cast<Instruction>(OpBaseName#"vvvml")
              $vx, $vy, $vz, $mask, $avl)>;
}

```
- **EN**: Implements logic around `Pat<`, `cast<Instruction>`.
- **CN**: 围绕 `Pat<`, `cast<Instruction>` 实现具体逻辑。

### Lines 452-466
```tablegen
multiclass Ternary_rvv<
    SDPatternOperator OpNode,
    ValueType ScalarVT, ValueType DataVT,
    ValueType MaskVT, string OpBaseName> {
  // Masked with passthru, broadcast first.
  def : Pat<(vvp_select
              (OpNode
                (any_broadcast ScalarVT:$sx), DataVT:$vy, DataVT:$vz,
                (MaskVT srcvalue), (i32 srcvalue)),
              DataVT:$vfalse,
              MaskVT:$mask,
              i32:$avl),
            (!cast<Instruction>(OpBaseName#"rvvml_v")
              $sx, $vy, $vz, $mask, $avl, $vfalse)>;

```
- **EN**: Declares TableGen records such as `Ternary_rvv`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Ternary_rvv` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 467-481
```tablegen
  // Unmasked, broadcast first.
  def : Pat<(OpNode
              (any_broadcast ScalarVT:$sx), DataVT:$vy, DataVT:$vz,
              (MaskVT true_mask), i32:$avl),
            (!cast<Instruction>(OpBaseName#"rvvl")
              $sx, $vy, $vz, $avl)>;

  // Masked, broadcast first.
  def : Pat<(OpNode 
              (any_broadcast ScalarVT:$sx), DataVT:$vy, DataVT:$vz,
              MaskVT:$mask, i32:$avl),
            (!cast<Instruction>(OpBaseName#"rvvml")
              $sx, $vy, $vz, $mask, $avl)>;
}

```
- **EN**: Implements logic around `Pat<`, `cast<Instruction>`.
- **CN**: 围绕 `Pat<`, `cast<Instruction>` 实现具体逻辑。

### Lines 482-497
```tablegen
multiclass Ternary_vrv<
    SDPatternOperator OpNode,
    ValueType ScalarVT, ValueType DataVT,
    ValueType MaskVT, string OpBaseName> {
  // Masked with passthru, broadcast second.
  def : Pat<(vvp_select
              (OpNode
                DataVT:$vx, (any_broadcast ScalarVT:$sy), DataVT:$vz,
                (MaskVT srcvalue), (i32 srcvalue)),
              DataVT:$vfalse,
              MaskVT:$mask,
              i32:$avl),
            (!cast<Instruction>(OpBaseName#"vrvml_v")
              $vx, $sy, $vz,
              $mask, $avl, $vfalse)>;

```
- **EN**: Declares TableGen records such as `Ternary_vrv`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Ternary_vrv` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 498-512
```tablegen
  // Unmasked, broadcast second.
  def : Pat<(OpNode
              DataVT:$vx, (any_broadcast ScalarVT:$sy), DataVT:$vz,
              (MaskVT true_mask), i32:$avl),
            (!cast<Instruction>(OpBaseName#"vrvl")
              $vx, $sy, $vz, $avl)>;

  // Masked, broadcast second.
  def : Pat<(OpNode
              DataVT:$vx, (any_broadcast ScalarVT:$sy), DataVT:$vz,
              MaskVT:$mask, i32:$avl),
            (!cast<Instruction>(OpBaseName#"vrvml")
              $vx, $sy, $vz, $mask, $avl)>;
}

```
- **EN**: Implements logic around `Pat<`, `cast<Instruction>`.
- **CN**: 围绕 `Pat<`, `cast<Instruction>` 实现具体逻辑。

### Lines 513-532
```tablegen
multiclass Ternary_rvv_vrv_vvv<
    SDPatternOperator OpNode,
    ValueType ScalarVT, ValueType DataVT,
    ValueType MaskVT, string OpBaseName> {
  defm : Ternary_rvv<OpNode, ScalarVT, DataVT, MaskVT, OpBaseName>;
  defm : Ternary_vrv<OpNode, ScalarVT, DataVT, MaskVT, OpBaseName>;
  defm : Ternary_vvv<OpNode, DataVT, MaskVT, OpBaseName>;
}

// Expand both 64bit and 32 bit variant (256 elements)
multiclass Ternary_ShortLong<
    SDPatternOperator OpNode,
    ValueType LongScalarVT, ValueType LongDataVT, string LongOpBaseName,
    ValueType ShortScalarVT, ValueType ShortDataVT, string ShortOpBaseName> {
  defm : Ternary_rvv_vrv_vvv<OpNode, LongScalarVT, LongDataVT,
                             v256i1, LongOpBaseName>;
  defm : Ternary_rvv_vrv_vvv<OpNode, ShortScalarVT, ShortDataVT,
                             v256i1, ShortOpBaseName>;
}

```
- **EN**: Declares TableGen records such as `Ternary_rvv_vrv_vvv`, `Ternary_ShortLong`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Ternary_rvv_vrv_vvv`, `Ternary_ShortLong` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 533-550
```tablegen
defm : Ternary_ShortLong<c_vvp_ffma,
                         f64, v256f64, "VFMADD", f32, v256f32, "VFMADS">;
defm : Ternary_rvv_vrv_vvv<c_vvp_ffma,
                           i64, v512f32, v512i1, "PVFMAD">;

multiclass Merge_mvv<
    SDPatternOperator OpNode,
    ValueType DataVT, ValueType MaskVT,
    string OpBaseName> {
  // Masked.
  def : Pat<(OpNode
                DataVT:$vtrue, DataVT:$vfalse,
                MaskVT:$vm,
                i32:$avl),
            (!cast<Instruction>(OpBaseName#"vvml_v")
                $vfalse, $vtrue, $vm, $avl, $vfalse)>;
}

```
- **EN**: Declares TableGen records such as `Merge_mvv`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Merge_mvv` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 551-562
```tablegen
multiclass Merge_mvv_ShortLong<
    SDPatternOperator OpNode,
    ValueType LongDataVT, ValueType ShortDataVT,
    string OpBaseName> {
  defm : Merge_mvv<OpNode,
                   LongDataVT, v256i1,
                   OpBaseName>;
  defm : Merge_mvv<OpNode,
                   ShortDataVT, v256i1,
                   OpBaseName>;
}

```
- **EN**: Declares TableGen records such as `Merge_mvv_ShortLong`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Merge_mvv_ShortLong` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 563-586
```tablegen
defm : Merge_mvv_ShortLong<vvp_select,
                           v256f64,
                           v256f32, "VMRG">;
defm : Merge_mvv_ShortLong<vvp_select,
                           v256i64,
                           v256i32, "VMRG">;

multiclass Set_CC<ValueType DataVT, string FmkBaseName, string CmpBaseName, SDPatternOperator CCMatcher, SDNodeXForm CCConv> {
  // Unmasked.
  def : Pat<(v256i1 (vvp_setcc
              DataVT:$LHS, DataVT:$RHS, CCMatcher:$cond, (v256i1 true_mask), i32:$vl)),
              (!cast<Instruction>(FmkBaseName#"vl")
                (CCConv $cond),
                (!cast<Instruction>(CmpBaseName#"vvl")
                  $LHS, $RHS, $vl),
                $vl)>;
  // Masked.
  def : Pat<(v256i1 (vvp_setcc
              DataVT:$LHS, DataVT:$RHS, CCMatcher:$cond, v256i1:$vm, i32:$vl)),
              (!cast<Instruction>(FmkBaseName#"vml")
                (CCConv $cond),
                (!cast<Instruction>(CmpBaseName#"vvl")
                  $LHS, $RHS, $vl),
                $vm, $vl)>;
```
- **EN**: Declares TableGen records such as `Set_CC`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Set_CC` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 587-607
```tablegen
}

defm : Set_CC<v256i64,"VFMKL","VCMPUL",CCUIOp,icond2cc>;
defm : Set_CC<v256i64,"VFMKL","VCMPSL",CCSIOp,icond2cc>;
defm : Set_CC<v256f64,"VFMKL","VFCMPD",cond,fcond2cc>;

defm : Set_CC<v256i32,"VFMKW","VCMPUW",CCUIOp,icond2cc>;
defm : Set_CC<v256i32,"VFMKW","VCMPSWZX",CCSIOp,icond2cc>;
defm : Set_CC<v256f32,"VFMKS","VFCMPS",cond,fcond2cc>;

multiclass Reduce_GenericInt<ValueType VectorVT,
    RegisterClass ResRC, ValueType ResVT,
    string VVPRedOp, string RedInstName> {
  // Unmasked.
  def : Pat <(ResVT (!cast<SDPatternOperator>("vvp_reduce_"#VVPRedOp)
                VectorVT:$vx, (v256i1 true_mask), i32:$vl)),
             (COPY_TO_REGCLASS
               (!cast<Instruction>("LVSvi")
                 (!cast<Instruction>(RedInstName#"vl") $vx, $vl), 0),
                 ResRC)>;

```
- **EN**: Declares TableGen records such as `Reduce_GenericInt`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Reduce_GenericInt` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 608-626
```tablegen
  // Masked.
  def : Pat <(ResVT (!cast<SDPatternOperator>("vvp_reduce_"#VVPRedOp)
                VectorVT:$vx, v256i1:$vm, i32:$vl)),
             (COPY_TO_REGCLASS
                (!cast<Instruction>("LVSvi")
                   (!cast<Instruction>(RedInstName#"vml") $vx, $vm, $vl), 0),
                   ResRC)>;
}

multiclass IntReduce_ShortLong<ValueType VectorVT,
    RegisterClass ResRC, ValueType ResVT,
    string SumSuffix, string MinMaxSuffix> {
  defm: Reduce_GenericInt<VectorVT, ResRC, ResVT, "or",  "VROR">;
  defm: Reduce_GenericInt<VectorVT, ResRC, ResVT, "and", "VRAND">;
  defm: Reduce_GenericInt<VectorVT, ResRC, ResVT, "xor", "VRXOR">;
  defm: Reduce_GenericInt<VectorVT, ResRC, ResVT, "add", "VSUM"#SumSuffix>;
  defm: Reduce_GenericInt<VectorVT, ResRC, ResVT, "smax", "VRMAX"#MinMaxSuffix>;
}

```
- **EN**: Declares TableGen records such as `IntReduce_ShortLong`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `IntReduce_ShortLong` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 627-628
```tablegen
defm: IntReduce_ShortLong<v256i64, I64, i64, "L","SLFST">;
defm: IntReduce_ShortLong<v256i32, I32, i32, "WSX","SWFSTSX">;
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `VVPInstrInfo.td`
- **LLVM subsystems / LLVM 子系统**: TableGen generators
