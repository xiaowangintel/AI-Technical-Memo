# VEInstrPatternsVec.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VEInstrPatternsVec.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines backend metadata in TableGen DSL that is later expanded into generated tables and matcher code.
  - **CN**: 使用 TableGen DSL 定义后端元数据，随后会展开为生成的表和匹配代码。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- VEInstrPatternsVec.td - VEC_-type SDNodes and isel for VE Target --===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-14
```tablegen
//
// This file describes the VEC_* prefixed intermediate SDNodes and their
// isel patterns.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 15-27
```tablegen
// Instruction format superclass
//===----------------------------------------------------------------------===//

// Sub-register replication for packed broadcast.
def: Pat<(i64 (repl_f32 f32:$val)),
            (ORrr
              (SRLri (f2l $val), 32),
              (zero_i32 (f2l $val)))>;
def: Pat<(i64 (repl_i32 i32:$val)),
            (ORrr
              (zero_f32 (i2l $val)),
              (SLLri (i2l $val), 32))>;

```
- **EN**: Implements logic around `Pat<`, `SRLri`, `zero_i32`, `zero_f32`, ....
- **CN**: 围绕 `Pat<`, `SRLri`, `zero_i32`, `zero_f32`, ... 实现具体逻辑。

### Lines 28-41
```tablegen
///// Mask Load & Store /////

// Store for v256i1, v512i1 are implemented in 2 ways.  These STVM/STVM512
// pseudo instruction is used for frameindex related load/store instructions.
// Custom Lowering is used for other load/store instructions.

def : Pat<(v256i1 (load ADDRrii:$addr)),
          (LDVMrii ADDRrii:$addr)>;
def : Pat<(v512i1 (load ADDRrii:$addr)),
          (LDVM512rii ADDRrii:$addr)>;
def : Pat<(store v256i1:$vx, ADDRrii:$addr),
          (STVMrii ADDRrii:$addr, $vx)>;
def : Pat<(store v512i1:$vx, ADDRrii:$addr),
          (STVM512rii ADDRrii:$addr, $vx)>;
```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 42-48
```tablegen

multiclass vbrd_elem32<ValueType v32, ValueType s32, SDPatternOperator ImmOp,
                       SDNodeXForm ImmCast, OutPatFrag SuperRegCast> {
  // VBRDil
  def : Pat<(v32 (vec_broadcast (s32 ImmOp:$sy), i32:$vl)),
            (VBRDil (ImmCast $sy), i32:$vl)>;

```
- **EN**: Declares TableGen records such as `vbrd_elem32`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `vbrd_elem32` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 49-59
```tablegen
  // VBRDrl
  def : Pat<(v32 (vec_broadcast s32:$sy, i32:$vl)),
            (VBRDrl (SuperRegCast $sy), i32:$vl)>;
}

multiclass vbrd_elem64<ValueType v64, ValueType s64,
                       SDPatternOperator ImmOp, SDNodeXForm ImmCast> {
  // VBRDil
  def : Pat<(v64 (vec_broadcast (s64 ImmOp:$sy), i32:$vl)),
            (VBRDil (ImmCast $sy), i32:$vl)>;

```
- **EN**: Declares TableGen records such as `vbrd_elem64`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `vbrd_elem64` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 60-73
```tablegen
  // VBRDrl
  def : Pat<(v64 (vec_broadcast s64:$sy, i32:$vl)),
            (VBRDrl s64:$sy, i32:$vl)>;
}

multiclass extract_insert_elem32<ValueType v32, ValueType s32,
                                 OutPatFrag SubRegCast,
                                 OutPatFrag SuperRegCast> {
  // LVSvi
  def: Pat<(s32 (extractelt v32:$vec, uimm7:$idx)),
           (SubRegCast (LVSvi v32:$vec, (ULO7 $idx)))>;
  // LVSvr
  def: Pat<(s32 (extractelt v32:$vec, i64:$idx)),
           (SubRegCast (LVSvr v32:$vec, $idx))>;
```
- **EN**: Declares TableGen records such as `extract_insert_elem32` for the backend description.
- **CN**: 为后端描述声明了 `extract_insert_elem32` 等 TableGen 记录。

### Lines 74-82
```tablegen

  // LSVir
  def: Pat<(v32 (insertelt v32:$vec, s32:$val, uimm7:$idx)),
           (LSVir_v (ULO7 $idx), (SuperRegCast $val), $vec)>;
  // LSVrr
  def: Pat<(v32 (insertelt v32:$vec, s32:$val, i64:$idx)),
           (LSVrr_v $idx, (SuperRegCast $val), $vec)>;
}

```
- **EN**: Implements logic around `Pat<`, `LSVir_v`.
- **CN**: 围绕 `Pat<`, `LSVir_v` 实现具体逻辑。

### Lines 83-90
```tablegen
multiclass extract_insert_elem64<ValueType v64, ValueType s64> {
  // LVSvi
  def: Pat<(s64 (extractelt v64:$vec, uimm7:$idx)),
           (LVSvi v64:$vec, (ULO7 $idx))>;
  // LVSvr
  def: Pat<(s64 (extractelt v64:$vec, i64:$idx)),
           (LVSvr v64:$vec, $idx)>;

```
- **EN**: Declares TableGen records such as `extract_insert_elem64` for the backend description.
- **CN**: 为后端描述声明了 `extract_insert_elem64` 等 TableGen 记录。

### Lines 91-98
```tablegen
  // LSVir
  def: Pat<(v64 (insertelt v64:$vec, s64:$val, uimm7:$idx)),
           (LSVir_v (ULO7 $idx), $val, $vec)>;
  // LSVrr
  def: Pat<(v64 (insertelt v64:$vec, s64:$val, i64:$idx)),
           (LSVrr_v $idx, $val, $vec)>;
}

```
- **EN**: Implements logic around `Pat<`, `LSVir_v`.
- **CN**: 围绕 `Pat<`, `LSVir_v` 实现具体逻辑。

### Lines 99-105
```tablegen
multiclass patterns_elem32<ValueType v32, ValueType s32,
                           SDPatternOperator ImmOp, SDNodeXForm ImmCast,
                           OutPatFrag SubRegCast, OutPatFrag SuperRegCast> {
  defm : vbrd_elem32<v32, s32, ImmOp, ImmCast, SuperRegCast>;
  defm : extract_insert_elem32<v32, s32, SubRegCast, SuperRegCast>;
}

```
- **EN**: Declares TableGen records such as `patterns_elem32`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `patterns_elem32` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 106-114
```tablegen
multiclass patterns_elem64<ValueType v64, ValueType s64,
                           SDPatternOperator ImmOp, SDNodeXForm ImmCast> {
  defm : vbrd_elem64<v64, s64, ImmOp, ImmCast>;
  defm : extract_insert_elem64<v64, s64>;
}

defm : patterns_elem32<v256i32, i32, simm7, LO7, l2i, i2l>;
defm : patterns_elem32<v256f32, f32, simm7fp, LO7FP, l2f, f2l>;

```
- **EN**: Declares TableGen records such as `patterns_elem64`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `patterns_elem64` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 115-122
```tablegen
defm : patterns_elem64<v256i64, i64, simm7, LO7>;
defm : patterns_elem64<v256f64, f64, simm7fp, LO7FP>;

defm : vbrd_elem64<v512i32, i64, simm7, LO7>;
defm : vbrd_elem64<v512f32, i64, simm7, LO7>;
defm : vbrd_elem64<v512i32, f64, simm7fp, LO7FP>;
defm : vbrd_elem64<v512f32, f64, simm7fp, LO7FP>;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 123-129
```tablegen
class Mask_Binary<ValueType MaskVT, SDPatternOperator MaskOp, string InstName> :
  Pat<(MaskVT (MaskOp MaskVT:$ma, MaskVT:$mb)), (!cast<Instruction>(InstName#"mm") $ma, $mb)>;

def: Mask_Binary<v256i1, and, "ANDM">;
def: Mask_Binary<v256i1, or,  "ORM">;
def: Mask_Binary<v256i1, xor, "XORM">;

```
- **EN**: Declares TableGen records such as `Mask_Binary`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Mask_Binary` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 130-142
```tablegen
///// Packing support /////

// v256i1 <> v512i1
def : Pat<(v256i1 (vec_unpack_lo v512i1:$vm, (i32 srcvalue))),
          (EXTRACT_SUBREG $vm, sub_vm_odd)>;
def : Pat<(v256i1 (vec_unpack_hi v512i1:$vm, (i32 srcvalue))),
          (EXTRACT_SUBREG $vm, sub_vm_even)>;
def : Pat<(v512i1 (vec_pack v256i1:$vlo, v256i1:$vhi, (i32 srcvalue))),
          (INSERT_SUBREG (INSERT_SUBREG
                         (v512i1 (IMPLICIT_DEF)),
                         $vlo, sub_vm_odd),
                         $vhi, sub_vm_even)>;

```
- **EN**: Implements logic around `Pat<`, `INSERT_SUBREG`, `v512i1`.
- **CN**: 围绕 `Pat<`, `INSERT_SUBREG`, `v512i1` 实现具体逻辑。

### Lines 143-150
```tablegen
// v256.32 <> v512.32
multiclass Packing<ValueType PackVT> {
  // no-op unpacks
  def : Pat<(v256i32 (vec_unpack_lo PackVT:$vp, (i32 srcvalue))),
            (COPY_TO_REGCLASS $vp, V64)>;
  def : Pat<(v256f32 (vec_unpack_hi PackVT:$vp, (i32 srcvalue))),
            (COPY_TO_REGCLASS $vp, V64)>;

```
- **EN**: Declares TableGen records such as `Packing` for the backend description.
- **CN**: 为后端描述声明了 `Packing` 等 TableGen 记录。

### Lines 151-157
```tablegen
  // shuffle unpacks
  def : Pat<(v256f32 (vec_unpack_lo PackVT:$vp, i32:$avl)),
            (VSHFvvil $vp, $vp, 4, $avl)>; // always pick lo
  def : Pat<(v256i32 (vec_unpack_hi PackVT:$vp, i32:$avl)),
            (VSHFvvil $vp, $vp, 0, $avl)>; // always pick hi
}

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 158-164
```tablegen
defm : Packing<v512i32>;
defm : Packing<v512f32>;

def : Pat<(v512i32 (vec_pack v256i32:$vlo, v256i32:$vhi, i32:$avl)),
          (VSHFvvil $vlo, $vhi, 13, $avl)>;
def : Pat<(v512f32 (vec_pack v256f32:$vlo, v256f32:$vhi, i32:$avl)),
          (VSHFvvil $vlo, $vhi, 8, $avl)>;
```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
