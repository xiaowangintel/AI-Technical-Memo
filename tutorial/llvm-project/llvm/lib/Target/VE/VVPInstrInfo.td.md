# VVPInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VVPInstrInfo.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines instruction records, encodings, and SelectionDAG patterns in TableGen DSL for this backend.
  - **CN**: 使用 TableGen DSL 定义该后端的指令记录、编码和 SelectionDAG 模式。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-------------- VVPInstrInfo.td - VVP_* SDNode patterns ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-19
```tablegen
//
// This file defines the VE Vector Predicated SDNodes (VVP SDNodes).  VVP
// SDNodes are an intermediate isel layer between the vector SDNodes emitted by
// LLVM and the actual VE vector instructions. For example:
//
//  ADD(x,y)   -->   VVP_ADD(x,y,mask,evl)   -->   VADDSWSXrvml(x,y,mask,evl)
//     ^                      ^                            ^
//  The standard     The VVP layer SDNode.        The VE vector instruction.
//  SDNode.
//
// TODO explain how VVP nodes relate to VP SDNodes once VP ISel is uptream.
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 20-29
```tablegen

// vvp_load(ptr, stride, mask, avl)
def SDTLoadVVP : SDTypeProfile<1, 4, [
  SDTCisVec<0>,
  SDTCisPtrTy<1>,
  SDTCisInt<2>,
  SDTCisVec<3>,
  IsVLVT<4>
]>;

```
- **EN**: Declares TableGen records such as `SDTLoadVVP` for the backend description.
- **CN**: 为后端描述声明了 `SDTLoadVVP` 等 TableGen 记录。

### Lines 30-38
```tablegen
// vvp_store(data, ptr, stride, mask, avl)
def SDTStoreVVP: SDTypeProfile<0, 5, [
  SDTCisVec<0>,
  SDTCisPtrTy<1>,
  SDTCisInt<2>,
  SDTCisVec<3>,
  IsVLVT<4>
]>;

```
- **EN**: Declares TableGen records such as `SDTStoreVVP` for the backend description.
- **CN**: 为后端描述声明了 `SDTStoreVVP` 等 TableGen 记录。

### Lines 39-47
```tablegen
// vvp_scatter(chain, data, addr, mask, avl)
def SDTScatterVVP: SDTypeProfile<0, 4, [
  SDTCisVec<0>,
  SDTCisVec<1>,
  SDTCisVec<2>,
  SDTCisSameNumEltsAs<0, 2>,
  IsVLVT<3>
]>;

```
- **EN**: Declares TableGen records such as `SDTScatterVVP` for the backend description.
- **CN**: 为后端描述声明了 `SDTScatterVVP` 等 TableGen 记录。

### Lines 48-55
```tablegen
// vvp_gather(chain, addr, mask, avl)
def SDTGatherVVP: SDTypeProfile<1, 3, [
  SDTCisVec<0>,
  SDTCisVec<1>,
  SDTCisSameNumEltsAs<0, 2>,
  IsVLVT<3>
]>;

```
- **EN**: Declares TableGen records such as `SDTGatherVVP` for the backend description.
- **CN**: 为后端描述声明了 `SDTGatherVVP` 等 TableGen 记录。

### Lines 56-64
```tablegen
// BinaryOp(x,y,mask,vl)
def SDTIntBinOpVVP : SDTypeProfile<1, 4, [     // vp_add, vp_and, etc.
  SDTCisSameAs<0, 1>,
  SDTCisSameAs<0, 2>,
  SDTCisInt<0>,
  SDTCisSameNumEltsAs<0, 3>,
  IsVLVT<4>
]>;

```
- **EN**: Declares TableGen records such as `SDTIntBinOpVVP` for the backend description.
- **CN**: 为后端描述声明了 `SDTIntBinOpVVP` 等 TableGen 记录。

### Lines 65-73
```tablegen
// UnaryFPOp(x,mask,vl)
def SDTFPUnaryOpVVP : SDTypeProfile<1, 3, [
  SDTCisSameAs<0, 1>,
  SDTCisFP<0>,
  SDTCisInt<2>,
  SDTCisSameNumEltsAs<0, 2>,
  IsVLVT<3>
]>;

```
- **EN**: Declares TableGen records such as `SDTFPUnaryOpVVP` for the backend description.
- **CN**: 为后端描述声明了 `SDTFPUnaryOpVVP` 等 TableGen 记录。

### Lines 74-83
```tablegen
// BinaryFPOp(x,y,mask,vl)
def SDTFPBinOpVVP : SDTypeProfile<1, 4, [      // vvp_fadd, etc.
  SDTCisSameAs<0, 1>,
  SDTCisSameAs<0, 2>,
  SDTCisFP<0>,
  SDTCisInt<3>,
  SDTCisSameNumEltsAs<0, 3>,
  IsVLVT<4>
]>;

```
- **EN**: Declares TableGen records such as `SDTFPBinOpVVP` for the backend description.
- **CN**: 为后端描述声明了 `SDTFPBinOpVVP` 等 TableGen 记录。

### Lines 84-94
```tablegen
// TernaryFPOp(x,y,z,mask,vl)
def SDTFPTernaryOpVVP : SDTypeProfile<1, 5, [
  SDTCisSameAs<0, 1>,
  SDTCisSameAs<0, 2>,
  SDTCisSameAs<0, 3>,
  SDTCisFP<0>,
  SDTCisInt<4>,
  SDTCisSameNumEltsAs<0, 4>,
  IsVLVT<5>
]>;

```
- **EN**: Declares TableGen records such as `SDTFPTernaryOpVVP` for the backend description.
- **CN**: 为后端描述声明了 `SDTFPTernaryOpVVP` 等 TableGen 记录。

### Lines 95-103
```tablegen
// Select(OnTrue, OnFalse, SelMask, vl)
def SDTSelectVVP : SDTypeProfile<1, 4, [       // vp_select, vp_merge
  SDTCisVec<0>,
  SDTCisSameNumEltsAs<0, 3>,
  SDTCisSameAs<0, 1>,
  SDTCisSameAs<1, 2>,
  IsVLVT<4>
]>;

```
- **EN**: Declares TableGen records such as `SDTSelectVVP` for the backend description.
- **CN**: 为后端描述声明了 `SDTSelectVVP` 等 TableGen 记录。

### Lines 104-115
```tablegen
// SetCC (lhs, rhs, cc, mask, vl)
def SDTSetCCVVP : SDTypeProfile<1, 5, [        // vp_setcc
  SDTCisVec<0>,
  SDTCisVec<1>,
  SDTCisSameNumEltsAs<0, 1>,
  SDTCisSameAs<1, 2>,
  SDTCisVT<3, OtherVT>,
  SDTCisInt<4>,
  SDTCisSameNumEltsAs<0, 4>,
  IsVLVT<5>
]>;

```
- **EN**: Declares TableGen records such as `SDTSetCCVVP` for the backend description.
- **CN**: 为后端描述声明了 `SDTSetCCVVP` 等 TableGen 记录。

### Lines 116-124
```tablegen
// vvp_reduce(vector, mask, vl)
def SDTReduceVVP : SDTypeProfile<1, 3, [
  SDTCisVec<1>,
  SDTCisInt<2>,
  SDTCisVec<2>,
  SDTCisSameNumEltsAs<1,2>,
  IsVLVT<3>
]>;

```
- **EN**: Declares TableGen records such as `SDTReduceVVP` for the backend description.
- **CN**: 为后端描述声明了 `SDTReduceVVP` 等 TableGen 记录。

### Lines 125-132
```tablegen

// Binary operator commutative pattern.
class vvp_commutative<SDNode RootOp> :
  PatFrags<
  (ops node:$lhs, node:$rhs, node:$mask, node:$vlen),
  [(RootOp node:$lhs, node:$rhs, node:$mask, node:$vlen),
   (RootOp node:$rhs, node:$lhs, node:$mask, node:$vlen)]>;

```
- **EN**: Declares TableGen records such as `vvp_commutative`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `vvp_commutative` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 133-142
```tablegen
class vvp_fma_commutative<SDNode RootOp> :
  PatFrags<
  (ops node:$X, node:$Y, node:$Z, node:$mask, node:$vlen),
  [(RootOp node:$X, node:$Y, node:$Z, node:$mask, node:$vlen),
   (RootOp node:$X, node:$Z, node:$Y, node:$mask, node:$vlen)]>;

// VVP node definitions.
def vvp_add    : SDNode<"VEISD::VVP_ADD",  SDTIntBinOpVVP>;
def c_vvp_add  : vvp_commutative<vvp_add>;

```
- **EN**: Declares TableGen records such as `vvp_fma_commutative`, `vvp_add`, `c_vvp_add` for the backend description.
- **CN**: 为后端描述声明了 `vvp_fma_commutative`, `vvp_add`, `c_vvp_add` 等 TableGen 记录。

### Lines 143-150
```tablegen
def vvp_sub    : SDNode<"VEISD::VVP_SUB",  SDTIntBinOpVVP>;

def vvp_mul    : SDNode<"VEISD::VVP_MUL",  SDTIntBinOpVVP>;
def c_vvp_mul  : vvp_commutative<vvp_mul>;

def vvp_sdiv   : SDNode<"VEISD::VVP_SDIV", SDTIntBinOpVVP>;
def vvp_udiv   : SDNode<"VEISD::VVP_UDIV", SDTIntBinOpVVP>;

```
- **EN**: Declares TableGen records such as `vvp_sub`, `vvp_mul`, `c_vvp_mul`, `vvp_sdiv`, ... for the backend description.
- **CN**: 为后端描述声明了 `vvp_sub`, `vvp_mul`, `c_vvp_mul`, `vvp_sdiv`, ... 等 TableGen 记录。

### Lines 151-159
```tablegen
def vvp_and    : SDNode<"VEISD::VVP_AND",  SDTIntBinOpVVP>;
def c_vvp_and  : vvp_commutative<vvp_and>;

def vvp_or     : SDNode<"VEISD::VVP_OR",  SDTIntBinOpVVP>;
def c_vvp_or   : vvp_commutative<vvp_or>;

def vvp_xor    : SDNode<"VEISD::VVP_XOR",  SDTIntBinOpVVP>;
def c_vvp_xor  : vvp_commutative<vvp_xor>;

```
- **EN**: Declares TableGen records such as `vvp_and`, `c_vvp_and`, `vvp_or`, `c_vvp_or`, ... for the backend description.
- **CN**: 为后端描述声明了 `vvp_and`, `c_vvp_and`, `vvp_or`, `c_vvp_or`, ... 等 TableGen 记录。

### Lines 160-172
```tablegen
def vvp_srl    : SDNode<"VEISD::VVP_SRL",  SDTIntBinOpVVP>;
def vvp_sra    : SDNode<"VEISD::VVP_SRA",  SDTIntBinOpVVP>;
def vvp_shl    : SDNode<"VEISD::VVP_SHL",  SDTIntBinOpVVP>;

def vvp_fneg    : SDNode<"VEISD::VVP_FNEG",  SDTFPUnaryOpVVP>;

def vvp_fadd    : SDNode<"VEISD::VVP_FADD",  SDTFPBinOpVVP>;
def c_vvp_fadd  : vvp_commutative<vvp_fadd>;
def vvp_fsub    : SDNode<"VEISD::VVP_FSUB",  SDTFPBinOpVVP>;
def vvp_fmul    : SDNode<"VEISD::VVP_FMUL",  SDTFPBinOpVVP>;
def c_vvp_fmul  : vvp_commutative<vvp_fmul>;
def vvp_fdiv    : SDNode<"VEISD::VVP_FDIV",  SDTFPBinOpVVP>;

```
- **EN**: Declares TableGen records such as `vvp_srl`, `vvp_sra`, `vvp_shl`, `vvp_fneg`, ... for the backend description.
- **CN**: 为后端描述声明了 `vvp_srl`, `vvp_sra`, `vvp_shl`, `vvp_fneg`, ... 等 TableGen 记录。

### Lines 173-180
```tablegen
def vvp_ffma    : SDNode<"VEISD::VVP_FFMA",  SDTFPTernaryOpVVP>;
def c_vvp_ffma  : vvp_fma_commutative<vvp_ffma>;

def vvp_scatter : SDNode<"VEISD::VVP_SCATTER",  SDTScatterVVP,
                         [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;
def vvp_gather  : SDNode<"VEISD::VVP_GATHER",  SDTGatherVVP,
                         [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;

```
- **EN**: Declares TableGen records such as `vvp_ffma`, `c_vvp_ffma`, `vvp_scatter`, `vvp_gather` for the backend description.
- **CN**: 为后端描述声明了 `vvp_ffma`, `c_vvp_ffma`, `vvp_scatter`, `vvp_gather` 等 TableGen 记录。

### Lines 181-187
```tablegen
def vvp_load    : SDNode<"VEISD::VVP_LOAD",  SDTLoadVVP,
                         [SDNPHasChain, SDNPMayLoad, SDNPMemOperand ]>;
def vvp_store   : SDNode<"VEISD::VVP_STORE", SDTStoreVVP,
                         [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;

// Reductions

```
- **EN**: Declares TableGen records such as `vvp_load`, `vvp_store` for the backend description.
- **CN**: 为后端描述声明了 `vvp_load`, `vvp_store` 等 TableGen 记录。

### Lines 188-194
```tablegen
// int reductions
def vvp_reduce_add          : SDNode<"VEISD::VVP_REDUCE_ADD", SDTReduceVVP>;
def vvp_reduce_and          : SDNode<"VEISD::VVP_REDUCE_AND", SDTReduceVVP>;
def vvp_reduce_or           : SDNode<"VEISD::VVP_REDUCE_OR",  SDTReduceVVP>;
def vvp_reduce_xor          : SDNode<"VEISD::VVP_REDUCE_XOR", SDTReduceVVP>;
def vvp_reduce_smax         : SDNode<"VEISD::VVP_REDUCE_SMAX", SDTReduceVVP>;

```
- **EN**: Declares TableGen records such as `vvp_reduce_add`, `vvp_reduce_and`, `vvp_reduce_or`, `vvp_reduce_xor`, ... for the backend description.
- **CN**: 为后端描述声明了 `vvp_reduce_add`, `vvp_reduce_and`, `vvp_reduce_or`, `vvp_reduce_xor`, ... 等 TableGen 记录。

### Lines 195-199
```tablegen

def vvp_select : SDNode<"VEISD::VVP_SELECT", SDTSelectVVP>;

// setcc (lhs, rhs, cc, mask, vl)
def vvp_setcc  : SDNode<"VEISD::VVP_SETCC", SDTSetCCVVP>;
```
- **EN**: Declares TableGen records such as `vvp_select`, `vvp_setcc` for the backend description.
- **CN**: 为后端描述声明了 `vvp_select`, `vvp_setcc` 等 TableGen 记录。

## Key Concepts / 关键概念

- **Instruction semantics / 指令语义**:
  - **EN**: Encodes instruction behavior and helper routines
  - **CN**: 描述指令行为与辅助例程
- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
