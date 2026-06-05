# PPCInstrAltivec.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCInstrAltivec.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCInstrAltivec.td - The PowerPC Altivec Extension. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCInstrAltivec.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===-- PPCInstrAltivec.td - The PowerPC Altivec Extension -*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-10

```tablegen
//===----------------------------------------------------------------------===//
//
// This file describes the Altivec extension to the PowerPC instruction set.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file describes the Altivec extension to the PowerPC instruction set.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file describes the Altivec extension to the PowerPC instruction set.”。

### Lines 11-28

```tablegen
//===----------------------------------------------------------------------===//

// *********************************** NOTE ***********************************
// ** For POWER8 Little Endian, the VSX swap optimization relies on knowing  **
// ** which VMX and VSX instructions are lane-sensitive and which are not.   **
// ** A lane-sensitive instruction relies, implicitly or explicitly, on      **
// ** whether lanes are numbered from left to right.  An instruction like    **
// ** VADDFP is not lane-sensitive, because each lane of the result vector   **
// ** relies only on the corresponding lane of the source vectors.  However, **
// ** an instruction like VMULESB is lane-sensitive, because "even" and      **
// ** "odd" lanes are different for big-endian and little-endian numbering.  **
// **                                                                        **
// ** When adding new VMX and VSX instructions, please consider whether they **
// ** are lane-sensitive.  If so, they must be added to a switch statement   **
// ** in PPCVSXSwapRemoval::gatherVectorInstructions().                      **
// ****************************************************************************
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "*********************************** NOTE ***********************************".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“*********************************** NOTE ***********************************”。

### Lines 29-37

```tablegen
//===----------------------------------------------------------------------===//
// Altivec transformation functions and pattern fragments.
//

// fneg is not legal, and desugared as an xor.
def desugared_fneg : PatFrag<(ops node:$x), (v4f32 (bitconvert (xor (bitconvert $x), 
                             (int_ppc_altivec_vslw (bitconvert (v16i8 immAllOnesV)), 
                             (bitconvert (v16i8 immAllOnesV))))))>; 
```
- **EN**: Adds declarative TableGen records such as `desugared_fneg` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `desugared_fneg`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 38-45

```tablegen
def vpkuhum_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                              (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVPKUHUMShuffleMask(cast<ShuffleVectorSDNode>(N), 0, *CurDAG);
}]>;
def vpkuwum_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                              (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVPKUWUMShuffleMask(cast<ShuffleVectorSDNode>(N), 0, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vpkuhum_shuffle`, `vpkuwum_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vpkuhum_shuffle`, `vpkuwum_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 46-53

```tablegen
def vpkudum_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                              (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVPKUDUMShuffleMask(cast<ShuffleVectorSDNode>(N), 0, *CurDAG);
}]>;
def vpkuhum_unary_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                    (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVPKUHUMShuffleMask(cast<ShuffleVectorSDNode>(N), 1, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vpkudum_shuffle`, `vpkuhum_unary_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vpkudum_shuffle`, `vpkuhum_unary_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 54-64

```tablegen
def vpkuwum_unary_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                    (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVPKUWUMShuffleMask(cast<ShuffleVectorSDNode>(N), 1, *CurDAG);
}]>;
def vpkudum_unary_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                    (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVPKUDUMShuffleMask(cast<ShuffleVectorSDNode>(N), 1, *CurDAG);
}]>;

// These fragments are provided for little-endian, where the inputs must be
// swapped for correct semantics.
```
- **EN**: Adds declarative TableGen records such as `vpkuwum_unary_shuffle`, `vpkudum_unary_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vpkuwum_unary_shuffle`, `vpkudum_unary_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 65-72

```tablegen
def vpkuhum_swapped_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                      (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVPKUHUMShuffleMask(cast<ShuffleVectorSDNode>(N), 2, *CurDAG);
}]>;
def vpkuwum_swapped_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                      (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVPKUWUMShuffleMask(cast<ShuffleVectorSDNode>(N), 2, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vpkuhum_swapped_shuffle`, `vpkuwum_swapped_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vpkuhum_swapped_shuffle`, `vpkuwum_swapped_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 73-81

```tablegen
def vpkudum_swapped_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                      (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVPKUDUMShuffleMask(cast<ShuffleVectorSDNode>(N), 2, *CurDAG);
}]>;

def vmrglb_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                             (vector_shuffle (v16i8 node:$lhs), node:$rhs), [{
  return PPC::isVMRGLShuffleMask(cast<ShuffleVectorSDNode>(N), 1, 0, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vpkudum_swapped_shuffle`, `vmrglb_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vpkudum_swapped_shuffle`, `vmrglb_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 82-89

```tablegen
def vmrglh_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                             (vector_shuffle (v16i8 node:$lhs), node:$rhs), [{
  return PPC::isVMRGLShuffleMask(cast<ShuffleVectorSDNode>(N), 2, 0, *CurDAG);
}]>;
def vmrglw_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                             (vector_shuffle (v16i8 node:$lhs), node:$rhs), [{
  return PPC::isVMRGLShuffleMask(cast<ShuffleVectorSDNode>(N), 4, 0, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vmrglh_shuffle`, `vmrglw_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vmrglh_shuffle`, `vmrglw_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 90-97

```tablegen
def vmrghb_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                             (vector_shuffle (v16i8 node:$lhs), node:$rhs), [{
  return PPC::isVMRGHShuffleMask(cast<ShuffleVectorSDNode>(N), 1, 0, *CurDAG);
}]>;
def vmrghh_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                             (vector_shuffle (v16i8 node:$lhs), node:$rhs), [{
  return PPC::isVMRGHShuffleMask(cast<ShuffleVectorSDNode>(N), 2, 0, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vmrghb_shuffle`, `vmrghh_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vmrghb_shuffle`, `vmrghh_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 98-107

```tablegen
def vmrghw_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                             (vector_shuffle (v16i8 node:$lhs), node:$rhs), [{
  return PPC::isVMRGHShuffleMask(cast<ShuffleVectorSDNode>(N), 4, 0, *CurDAG);
}]>;


def vmrglb_unary_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                               (vector_shuffle (v16i8 node:$lhs), node:$rhs), [{
  return PPC::isVMRGLShuffleMask(cast<ShuffleVectorSDNode>(N), 1, 1, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vmrghw_shuffle`, `vmrglb_unary_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vmrghw_shuffle`, `vmrglb_unary_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 108-115

```tablegen
def vmrglh_unary_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                   (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGLShuffleMask(cast<ShuffleVectorSDNode>(N), 2, 1, *CurDAG);
}]>;
def vmrglw_unary_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                   (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGLShuffleMask(cast<ShuffleVectorSDNode>(N), 4, 1, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vmrglh_unary_shuffle`, `vmrglw_unary_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vmrglh_unary_shuffle`, `vmrglw_unary_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 116-123

```tablegen
def vmrghb_unary_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                   (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGHShuffleMask(cast<ShuffleVectorSDNode>(N), 1, 1, *CurDAG);
}]>;
def vmrghh_unary_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                   (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGHShuffleMask(cast<ShuffleVectorSDNode>(N), 2, 1, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vmrghb_unary_shuffle`, `vmrghh_unary_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vmrghb_unary_shuffle`, `vmrghh_unary_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 124-131

```tablegen
def vmrghw_unary_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                   (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGHShuffleMask(cast<ShuffleVectorSDNode>(N), 4, 1, *CurDAG);
}]>;


// These fragments are provided for little-endian, where the inputs must be
// swapped for correct semantics.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "These fragments are provided for little-endian, where the inputs must be". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“These fragments are provided for little-endian, where the inputs must be”。 这些声明会进入生成式模式匹配逻辑。

### Lines 132-139

```tablegen
def vmrglb_swapped_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                               (vector_shuffle (v16i8 node:$lhs), node:$rhs), [{
  return PPC::isVMRGLShuffleMask(cast<ShuffleVectorSDNode>(N), 1, 2, *CurDAG);
}]>;
def vmrglh_swapped_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                   (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGLShuffleMask(cast<ShuffleVectorSDNode>(N), 2, 2, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vmrglb_swapped_shuffle`, `vmrglh_swapped_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vmrglb_swapped_shuffle`, `vmrglh_swapped_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 140-147

```tablegen
def vmrglw_swapped_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                   (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGLShuffleMask(cast<ShuffleVectorSDNode>(N), 4, 2, *CurDAG);
}]>;
def vmrghb_swapped_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                   (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGHShuffleMask(cast<ShuffleVectorSDNode>(N), 1, 2, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vmrglw_swapped_shuffle`, `vmrghb_swapped_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vmrglw_swapped_shuffle`, `vmrghb_swapped_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 148-157

```tablegen
def vmrghh_swapped_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                   (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGHShuffleMask(cast<ShuffleVectorSDNode>(N), 2, 2, *CurDAG);
}]>;
def vmrghw_swapped_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                   (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGHShuffleMask(cast<ShuffleVectorSDNode>(N), 4, 2, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vmrghh_swapped_shuffle`, `vmrghw_swapped_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vmrghh_swapped_shuffle`, `vmrghw_swapped_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 158-165

```tablegen
def vmrgew_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                             (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGEOShuffleMask(cast<ShuffleVectorSDNode>(N), true, 0, *CurDAG);
}]>;
def vmrgow_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                             (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGEOShuffleMask(cast<ShuffleVectorSDNode>(N), false, 0, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vmrgew_shuffle`, `vmrgow_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vmrgew_shuffle`, `vmrgow_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 166-173

```tablegen
def vmrgew_unary_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                   (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGEOShuffleMask(cast<ShuffleVectorSDNode>(N), true, 1, *CurDAG);
}]>;
def vmrgow_unary_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                   (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGEOShuffleMask(cast<ShuffleVectorSDNode>(N), false, 1, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vmrgew_unary_shuffle`, `vmrgow_unary_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vmrgew_unary_shuffle`, `vmrgow_unary_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 174-184

```tablegen
def vmrgew_swapped_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                     (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGEOShuffleMask(cast<ShuffleVectorSDNode>(N), true, 2, *CurDAG);
}]>;
def vmrgow_swapped_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                     (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVMRGEOShuffleMask(cast<ShuffleVectorSDNode>(N), false, 2, *CurDAG);
}]>;
```
- **EN**: Adds declarative TableGen records such as `vmrgew_swapped_shuffle`, `vmrgow_swapped_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vmrgew_swapped_shuffle`, `vmrgow_swapped_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 185-195

```tablegen
def VSLDOI_get_imm : SDNodeXForm<vector_shuffle, [{
  return getI32Imm(PPC::isVSLDOIShuffleMask(N, 0, *CurDAG), SDLoc(N));
}]>;
def vsldoi_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                             (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVSLDOIShuffleMask(N, 0, *CurDAG) != -1;
}], VSLDOI_get_imm>;


/// VSLDOI_unary* - These are used to match vsldoi(X,X), which is turned into
/// vector_shuffle(X,undef,mask) by the dag combiner.
```
- **EN**: Adds declarative TableGen records such as `VSLDOI_get_imm`, `vsldoi_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSLDOI_get_imm`, `vsldoi_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 196-206

```tablegen
def VSLDOI_unary_get_imm : SDNodeXForm<vector_shuffle, [{
  return getI32Imm(PPC::isVSLDOIShuffleMask(N, 1, *CurDAG), SDLoc(N));
}]>;
def vsldoi_unary_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                   (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVSLDOIShuffleMask(N, 1, *CurDAG) != -1;
}], VSLDOI_unary_get_imm>;


/// VSLDOI_swapped* - These fragments are provided for little-endian, where
/// the inputs must be swapped for correct semantics.
```
- **EN**: Adds declarative TableGen records such as `VSLDOI_unary_get_imm`, `vsldoi_unary_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSLDOI_unary_get_imm`, `vsldoi_unary_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 207-216

```tablegen
def VSLDOI_swapped_get_imm : SDNodeXForm<vector_shuffle, [{
  return getI32Imm(PPC::isVSLDOIShuffleMask(N, 2, *CurDAG), SDLoc(N));
}]>;
def vsldoi_swapped_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                                     (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isVSLDOIShuffleMask(N, 2, *CurDAG) != -1;
}], VSLDOI_get_imm>;


// VSPLT*_get_imm xform function: convert vector_shuffle mask to VSPLT* imm.
```
- **EN**: Adds declarative TableGen records such as `VSLDOI_swapped_get_imm`, `vsldoi_swapped_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSLDOI_swapped_get_imm`, `vsldoi_swapped_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 217-223

```tablegen
def VSPLTB_get_imm : SDNodeXForm<vector_shuffle, [{
  return getI32Imm(PPC::getSplatIdxForPPCMnemonics(N, 1, *CurDAG), SDLoc(N));
}]>;
def vspltb_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                             (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isSplatShuffleMask(cast<ShuffleVectorSDNode>(N), 1);
}], VSPLTB_get_imm>;
```
- **EN**: Adds declarative TableGen records such as `VSPLTB_get_imm`, `vspltb_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSPLTB_get_imm`, `vspltb_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 224-230

```tablegen
def VSPLTH_get_imm : SDNodeXForm<vector_shuffle, [{
  return getI32Imm(PPC::getSplatIdxForPPCMnemonics(N, 2, *CurDAG), SDLoc(N));
}]>;
def vsplth_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                             (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isSplatShuffleMask(cast<ShuffleVectorSDNode>(N), 2);
}], VSPLTH_get_imm>;
```
- **EN**: Adds declarative TableGen records such as `VSPLTH_get_imm`, `vsplth_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSPLTH_get_imm`, `vsplth_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 231-240

```tablegen
def VSPLTW_get_imm : SDNodeXForm<vector_shuffle, [{
  return getI32Imm(PPC::getSplatIdxForPPCMnemonics(N, 4, *CurDAG), SDLoc(N));
}]>;
def vspltw_shuffle : PatFrag<(ops node:$lhs, node:$rhs),
                             (vector_shuffle node:$lhs, node:$rhs), [{
  return PPC::isSplatShuffleMask(cast<ShuffleVectorSDNode>(N), 4);
}], VSPLTW_get_imm>;


// VSPLTISB_get_imm xform function: convert build_vector to VSPLTISB imm.
```
- **EN**: Adds declarative TableGen records such as `VSPLTW_get_imm`, `vspltw_shuffle` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSPLTW_get_imm`, `vspltw_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 241-248

```tablegen
def VSPLTISB_get_imm : SDNodeXForm<build_vector, [{
  return PPC::get_VSPLTI_elt(N, 1, *CurDAG);
}]>;
def vecspltisb : PatLeaf<(build_vector), [{
  return PPC::get_VSPLTI_elt(N, 1, *CurDAG).getNode() != nullptr;
}], VSPLTISB_get_imm>;

// VSPLTISH_get_imm xform function: convert build_vector to VSPLTISH imm.
```
- **EN**: Adds declarative TableGen records such as `VSPLTISB_get_imm`, `vecspltisb` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSPLTISB_get_imm`, `vecspltisb`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 249-256

```tablegen
def VSPLTISH_get_imm : SDNodeXForm<build_vector, [{
  return PPC::get_VSPLTI_elt(N, 2, *CurDAG);
}]>;
def vecspltish : PatLeaf<(build_vector), [{
  return PPC::get_VSPLTI_elt(N, 2, *CurDAG).getNode() != nullptr;
}], VSPLTISH_get_imm>;

// VSPLTISW_get_imm xform function: convert build_vector to VSPLTISW imm.
```
- **EN**: Adds declarative TableGen records such as `VSPLTISH_get_imm`, `vecspltish` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSPLTISH_get_imm`, `vecspltish`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 257-263

```tablegen
def VSPLTISW_get_imm : SDNodeXForm<build_vector, [{
  return PPC::get_VSPLTI_elt(N, 4, *CurDAG);
}]>;
def vecspltisw : PatLeaf<(build_vector), [{
  return PPC::get_VSPLTI_elt(N, 4, *CurDAG).getNode() != nullptr;
}], VSPLTISW_get_imm>;
```
- **EN**: Adds declarative TableGen records such as `VSPLTISW_get_imm`, `vecspltisw` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSPLTISW_get_imm`, `vecspltisw`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 264-275

```tablegen
def immEQOneV : PatLeaf<(build_vector), [{
  if (ConstantSDNode *C = cast<BuildVectorSDNode>(N)->getConstantSplatNode())
    return C->isOne();
  return false;
}]>;

def VSRVSRO : PatFrag<(ops node:$input, node:$shift), 
                      (int_ppc_altivec_vsr 
                        (int_ppc_altivec_vsro node:$input, node:$shift), 
                        node:$shift), 
                      [{ return N->getOperand(1).hasOneUse(); }]>;
```
- **EN**: Adds declarative TableGen records such as `immEQOneV`, `VSRVSRO` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `immEQOneV`, `VSRVSRO`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 276-286

```tablegen
//===----------------------------------------------------------------------===//
// Helpers for defining instructions that directly correspond to intrinsics.

// VA1a_Int_Ty - A VAForm_1a intrinsic definition of specific type.
class VA1a_Int_Ty<bits<6> xo, string opc, Intrinsic IntID, ValueType Ty>
  : VAForm_1a<xo, (outs vrrc:$RT), (ins vrrc:$RA, vrrc:$RB, vrrc:$RC),
              !strconcat(opc, " $RT, $RA, $RB, $RC"), IIC_VecFP,
                       [(set Ty:$RT, (IntID Ty:$RA, Ty:$RB, Ty:$RC))]>;

// VA1a_Int_Ty2 - A VAForm_1a intrinsic definition where the type of the
// inputs doesn't match the type of the output.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Helpers for defining instructions that directly correspond to intrinsics.". Notable symbols in this range include `VA1a_Int_Ty`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Helpers for defining instructions that directly correspond to intrinsics.”。 该区间中较显眼的符号包括 `VA1a_Int_Ty`。

### Lines 287-294

```tablegen
class VA1a_Int_Ty2<bits<6> xo, string opc, Intrinsic IntID, ValueType OutTy,
                   ValueType InTy>
  : VAForm_1a<xo, (outs vrrc:$RT), (ins vrrc:$RA, vrrc:$RB, vrrc:$RC),
              !strconcat(opc, " $RT, $RA, $RB, $RC"), IIC_VecFP,
                       [(set OutTy:$RT, (IntID InTy:$RA, InTy:$RB, InTy:$RC))]>;

// VA1a_Int_Ty3 - A VAForm_1a intrinsic definition where there are two
// input types and an output type.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "VA1a_Int_Ty3 - A VAForm_1a intrinsic definition where there are two". Notable symbols in this range include `VA1a_Int_Ty2`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“VA1a_Int_Ty3 - A VAForm_1a intrinsic definition where there are two”。 该区间中较显眼的符号包括 `VA1a_Int_Ty2`。

### Lines 295-302

```tablegen
class VA1a_Int_Ty3<bits<6> xo, string opc, Intrinsic IntID, ValueType OutTy,
                   ValueType In1Ty, ValueType In2Ty>
  : VAForm_1a<xo, (outs vrrc:$RT), (ins vrrc:$RA, vrrc:$RB, vrrc:$RC),
              !strconcat(opc, " $RT, $RA, $RB, $RC"), IIC_VecFP,
                       [(set OutTy:$RT,
                         (IntID In1Ty:$RA, In1Ty:$RB, In2Ty:$RC))]>;

// VX1_Int_Ty - A VXForm_1 intrinsic definition of specific type.
```
- **EN**: Declares a backend-facing type `VA1a_Int_Ty3` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VA1a_Int_Ty3`，并勾勒出周边代码会依赖的接口或状态。

### Lines 303-309

```tablegen
class VX1_Int_Ty<bits<11> xo, string opc, Intrinsic IntID, ValueType Ty>
  : VXForm_1<xo, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
             !strconcat(opc, " $VD, $VA, $VB"), IIC_VecFP,
             [(set Ty:$VD, (IntID Ty:$VA, Ty:$VB))]>;

// VX1_Int_Ty2 - A VXForm_1 intrinsic definition where the type of the
// inputs doesn't match the type of the output.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "VX1_Int_Ty2 - A VXForm_1 intrinsic definition where the type of the".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“VX1_Int_Ty2 - A VXForm_1 intrinsic definition where the type of the”。

### Lines 310-317

```tablegen
class VX1_Int_Ty2<bits<11> xo, string opc, Intrinsic IntID, ValueType OutTy,
                  ValueType InTy>
  : VXForm_1<xo, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
             !strconcat(opc, " $VD, $VA, $VB"), IIC_VecFP,
             [(set OutTy:$VD, (IntID InTy:$VA, InTy:$VB))]>;

// VX1_Int_Ty3 - A VXForm_1 intrinsic definition where there are two
// input types and an output type.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "VX1_Int_Ty3 - A VXForm_1 intrinsic definition where there are two". Notable symbols in this range include `VX1_Int_Ty2`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“VX1_Int_Ty3 - A VXForm_1 intrinsic definition where there are two”。 该区间中较显眼的符号包括 `VX1_Int_Ty2`。

### Lines 318-324

```tablegen
class VX1_Int_Ty3<bits<11> xo, string opc, Intrinsic IntID, ValueType OutTy,
                  ValueType In1Ty, ValueType In2Ty>
  : VXForm_1<xo, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
             !strconcat(opc, " $VD, $VA, $VB"), IIC_VecFP,
             [(set OutTy:$VD, (IntID In1Ty:$VA, In2Ty:$VB))]>;

// VX2_Int_SP - A VXForm_2 intrinsic definition of vector single-precision type.
```
- **EN**: Declares a backend-facing type `VX1_Int_Ty3` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VX1_Int_Ty3`，并勾勒出周边代码会依赖的接口或状态。

### Lines 325-331

```tablegen
class VX2_Int_SP<bits<11> xo, string opc, Intrinsic IntID>
  : VXForm_2<xo, (outs vrrc:$VD), (ins vrrc:$VB),
             !strconcat(opc, " $VD, $VB"), IIC_VecFP,
             [(set v4f32:$VD, (IntID v4f32:$VB))]>;

// VX2_Int_Ty2 - A VXForm_2 intrinsic definition where the type of the
// inputs doesn't match the type of the output.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "VX2_Int_Ty2 - A VXForm_2 intrinsic definition where the type of the".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“VX2_Int_Ty2 - A VXForm_2 intrinsic definition where the type of the”。

### Lines 332-342

```tablegen
class VX2_Int_Ty2<bits<11> xo, string opc, Intrinsic IntID, ValueType OutTy,
                  ValueType InTy>
  : VXForm_2<xo, (outs vrrc:$VD), (ins vrrc:$VB),
             !strconcat(opc, " $VD, $VB"), IIC_VecFP,
             [(set OutTy:$VD, (IntID InTy:$VB))]>;

class VXBX_Int_Ty<bits<11> xo, string opc, Intrinsic IntID, ValueType Ty>
  : VXForm_BX<xo, (outs vrrc:$VD), (ins vrrc:$VA),
             !strconcat(opc, " $VD, $VA"), IIC_VecFP,
             [(set Ty:$VD, (IntID Ty:$VA))]>;
```
- **EN**: Declares a backend-facing type `VX2_Int_Ty2`, `VXBX_Int_Ty` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VX2_Int_Ty2`, `VXBX_Int_Ty`，并勾勒出周边代码会依赖的接口或状态。

### Lines 343-348

```tablegen
class VXCR_Int_Ty<bits<11> xo, string opc, Intrinsic IntID, ValueType Ty>
  : VXForm_CR<xo, (outs vrrc:$VD), (ins vrrc:$VA, u1imm:$ST, u4imm:$SIX),
              !strconcat(opc, " $VD, $VA, $ST, $SIX"), IIC_VecFP,
              [(set Ty:$VD, (IntID Ty:$VA, u1imm_timm:$ST,
                                           u4imm_timm:$SIX))]>;
```
- **EN**: Declares a backend-facing type `VXCR_Int_Ty` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXCR_Int_Ty`，并勾勒出周边代码会依赖的接口或状态。

### Lines 349-356

```tablegen
//===----------------------------------------------------------------------===//
// Instruction Definitions.

let Predicates = [HasAltivec] in {

def DSS      : DSS_Form<0, 822, (outs), (ins u5imm:$STRM),
                        "dss $STRM", IIC_LdStLoad /*FIXME*/, [(int_ppc_altivec_dss imm:$STRM)]>,
                        Deprecated<DeprecatedDST> {
```
- **EN**: Adds declarative TableGen records such as `DSS` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DSS`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 357-363

```tablegen
  let RA = 0;
  let RB = 0;
}

def DSSALL   : DSS_Form<1, 822, (outs), (ins),
                        "dssall", IIC_LdStLoad /*FIXME*/, []>,
                        Deprecated<DeprecatedDST> {
```
- **EN**: Adds declarative TableGen records such as `DSSALL` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DSSALL`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 364-373

```tablegen
  let STRM = 0;
  let RA = 0;
  let RB = 0;
}

def DST      : DSS_Form<0, 342, (outs), (ins u5imm:$STRM, gprc:$RA, gprc:$RB),
                        "dst $RA, $RB, $STRM", IIC_LdStLoad /*FIXME*/,
                        [(int_ppc_altivec_dst i32:$RA, i32:$RB, imm:$STRM)]>,
                        Deprecated<DeprecatedDST>;
```
- **EN**: Adds declarative TableGen records such as `DST` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DST`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 374-383

```tablegen
def DSTT     : DSS_Form<1, 342, (outs), (ins u5imm:$STRM, gprc:$RA, gprc:$RB),
                        "dstt $RA, $RB, $STRM", IIC_LdStLoad /*FIXME*/,
                        [(int_ppc_altivec_dstt i32:$RA, i32:$RB, imm:$STRM)]>,
                        Deprecated<DeprecatedDST>;

def DSTST    : DSS_Form<0, 374, (outs), (ins u5imm:$STRM, gprc:$RA, gprc:$RB),
                        "dstst $RA, $RB, $STRM", IIC_LdStLoad /*FIXME*/,
                        [(int_ppc_altivec_dstst i32:$RA, i32:$RB, imm:$STRM)]>,
                        Deprecated<DeprecatedDST>;
```
- **EN**: Adds declarative TableGen records such as `DSTT`, `DSTST` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DSTT`, `DSTST`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 384-390

```tablegen
def DSTSTT   : DSS_Form<1, 374, (outs), (ins u5imm:$STRM, gprc:$RA, gprc:$RB),
                        "dststt $RA, $RB, $STRM", IIC_LdStLoad /*FIXME*/,
                        [(int_ppc_altivec_dststt i32:$RA, i32:$RB, imm:$STRM)]>,
                        Deprecated<DeprecatedDST>;

let isCodeGenOnly = 1 in {
  // The very same instructions as above, but formally matching 64bit registers.
```
- **EN**: Adds declarative TableGen records such as `DSTSTT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DSTSTT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 391-400

```tablegen
  def DST64    : DSS_Form<0, 342, (outs), (ins u5imm:$STRM, g8rc:$RA, gprc:$RB),
                          "dst $RA, $RB, $STRM", IIC_LdStLoad /*FIXME*/,
                          [(int_ppc_altivec_dst i64:$RA, i32:$RB, imm:$STRM)]>,
                          Deprecated<DeprecatedDST>;

  def DSTT64   : DSS_Form<1, 342, (outs), (ins u5imm:$STRM, g8rc:$RA, gprc:$RB),
                          "dstt $RA, $RB, $STRM", IIC_LdStLoad /*FIXME*/,
                          [(int_ppc_altivec_dstt i64:$RA, i32:$RB, imm:$STRM)]>,
                          Deprecated<DeprecatedDST>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 401-413

```tablegen
  def DSTST64  : DSS_Form<0, 374, (outs), (ins u5imm:$STRM, g8rc:$RA, gprc:$RB),
                          "dstst $RA, $RB, $STRM", IIC_LdStLoad /*FIXME*/,
                          [(int_ppc_altivec_dstst i64:$RA, i32:$RB,
                                                  imm:$STRM)]>,
                          Deprecated<DeprecatedDST>;

  def DSTSTT64 : DSS_Form<1, 374, (outs), (ins u5imm:$STRM, g8rc:$RA, gprc:$RB),
                          "dststt $RA, $RB, $STRM", IIC_LdStLoad /*FIXME*/,
                          [(int_ppc_altivec_dststt i64:$RA, i32:$RB,
                                                   imm:$STRM)]>,
                          Deprecated<DeprecatedDST>;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 414-422

```tablegen
let hasSideEffects = 1 in {
  def MFVSCR : VXForm_4<1540, (outs vrrc:$VD), (ins),
                        "mfvscr $VD", IIC_LdStStore,
                        [(set v8i16:$VD, (int_ppc_altivec_mfvscr))]>;
  def MTVSCR : VXForm_5<1604, (outs), (ins vrrc:$VB),
                        "mtvscr $VB", IIC_LdStLoad,
                        [(int_ppc_altivec_mtvscr v4i32:$VB)]>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 423-429

```tablegen
let PPC970_Unit = 2, mayLoad = 1, mayStore = 0 in {  // Loads.
def LVEBX: XForm_1_memOp<31,   7, (outs vrrc:$RST), (ins (memrr $RA, $RB):$addr),
                   "lvebx $RST, $addr", IIC_LdStLoad,
                   [(set v16i8:$RST, (int_ppc_altivec_lvebx ForceXForm:$addr))]>;
def LVEHX: XForm_1_memOp<31,  39, (outs vrrc:$RST), (ins (memrr $RA, $RB):$addr),
                   "lvehx $RST, $addr", IIC_LdStLoad,
                   [(set v8i16:$RST, (int_ppc_altivec_lvehx ForceXForm:$addr))]>;
```
- **EN**: Adds declarative TableGen records such as `LVEBX`, `LVEHX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LVEBX`, `LVEHX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 430-440

```tablegen
def LVEWX: XForm_1_memOp<31,  71, (outs vrrc:$RST), (ins (memrr $RA, $RB):$addr),
                   "lvewx $RST, $addr", IIC_LdStLoad,
                   [(set v4i32:$RST, (int_ppc_altivec_lvewx ForceXForm:$addr))]>;
def LVX  : XForm_1_memOp<31, 103, (outs vrrc:$RST), (ins (memrr $RA, $RB):$addr),
                   "lvx $RST, $addr", IIC_LdStLoad,
                   [(set v4i32:$RST, (int_ppc_altivec_lvx ForceXForm:$addr))]>;
def LVXL : XForm_1_memOp<31, 359, (outs vrrc:$RST), (ins (memrr $RA, $RB):$addr),
                   "lvxl $RST, $addr", IIC_LdStLoad,
                   [(set v4i32:$RST, (int_ppc_altivec_lvxl ForceXForm:$addr))]>;
}
```
- **EN**: Adds declarative TableGen records such as `LVEWX`, `LVX`, `LVXL` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LVEWX`, `LVX`, `LVXL`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 441-449

```tablegen
def LVSL : XForm_1_memOp<31,   6, (outs vrrc:$RST), (ins (memrr $RA, $RB):$addr),
                   "lvsl $RST, $addr", IIC_LdStLoad,
                   [(set v16i8:$RST, (int_ppc_altivec_lvsl ForceXForm:$addr))]>,
                   PPC970_Unit_LSU;
def LVSR : XForm_1_memOp<31,  38, (outs vrrc:$RST), (ins (memrr $RA, $RB):$addr),
                   "lvsr $RST, $addr", IIC_LdStLoad,
                   [(set v16i8:$RST, (int_ppc_altivec_lvsr ForceXForm:$addr))]>,
                   PPC970_Unit_LSU;
```
- **EN**: Adds declarative TableGen records such as `LVSL`, `LVSR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LVSL`, `LVSR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 450-456

```tablegen
let PPC970_Unit = 2, mayStore = 1, mayLoad = 0 in {   // Stores.
def STVEBX: XForm_8_memOp<31, 135, (outs), (ins vrrc:$RST, (memrr $RA, $RB):$addr),
                   "stvebx $RST, $addr", IIC_LdStStore,
                   [(int_ppc_altivec_stvebx v16i8:$RST, ForceXForm:$addr)]>;
def STVEHX: XForm_8_memOp<31, 167, (outs), (ins vrrc:$RST, (memrr $RA, $RB):$addr),
                   "stvehx $RST, $addr", IIC_LdStStore,
                   [(int_ppc_altivec_stvehx v8i16:$RST, ForceXForm:$addr)]>;
```
- **EN**: Adds declarative TableGen records such as `STVEBX`, `STVEHX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STVEBX`, `STVEHX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 457-467

```tablegen
def STVEWX: XForm_8_memOp<31, 199, (outs), (ins vrrc:$RST, (memrr $RA, $RB):$addr),
                   "stvewx $RST, $addr", IIC_LdStStore,
                   [(int_ppc_altivec_stvewx v4i32:$RST, ForceXForm:$addr)]>;
def STVX  : XForm_8_memOp<31, 231, (outs), (ins vrrc:$RST, (memrr $RA, $RB):$addr),
                   "stvx $RST, $addr", IIC_LdStStore,
                   [(int_ppc_altivec_stvx v4i32:$RST, ForceXForm:$addr)]>;
def STVXL : XForm_8_memOp<31, 487, (outs), (ins vrrc:$RST, (memrr $RA, $RB):$addr),
                   "stvxl $RST, $addr", IIC_LdStStore,
                   [(int_ppc_altivec_stvxl v4i32:$RST, ForceXForm:$addr)]>;
}
```
- **EN**: Adds declarative TableGen records such as `STVEWX`, `STVX`, `STVXL` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STVEWX`, `STVX`, `STVXL`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 468-476

```tablegen
let PPC970_Unit = 5 in {  // VALU Operations.
// VA-Form instructions.  3-input AltiVec ops.
let isCommutable = 1 in {
def VMADDFP : VAForm_1<46, (outs vrrc:$RT), (ins vrrc:$RA, vrrc:$RC, vrrc:$RB),
                       "vmaddfp $RT, $RA, $RC, $RB", IIC_VecFP,
                       [(set v4f32:$RT,
                        (fma v4f32:$RA, v4f32:$RC, v4f32:$RB))]>;

// fneg is not legal, hence we have to match on the desugared version. 
```
- **EN**: Adds declarative TableGen records such as `VMADDFP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VMADDFP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 477-483

```tablegen
def VNMSUBFP: VAForm_1<47, (outs vrrc:$RT), (ins vrrc:$RA, vrrc:$RC, vrrc:$RB),
                       "vnmsubfp $RT, $RA, $RC, $RB", IIC_VecFP,
                       [(set v4f32:$RT, (desugared_fneg (fma v4f32:$RA, v4f32:$RC,
                                                  (desugared_fneg v4f32:$RB))))]>;

let hasSideEffects = 1 in {
  def VMHADDSHS  : VA1a_Int_Ty<32, "vmhaddshs", int_ppc_altivec_vmhaddshs, v8i16>;
```
- **EN**: Adds declarative TableGen records such as `VNMSUBFP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VNMSUBFP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 484-491

```tablegen
  def VMHRADDSHS : VA1a_Int_Ty<33, "vmhraddshs", int_ppc_altivec_vmhraddshs,
                               v8i16>;
}
def VMLADDUHM  : VA1a_Int_Ty<34, "vmladduhm", int_ppc_altivec_vmladduhm, v8i16>;
} // isCommutable

def VPERM      : VA1a_Int_Ty3<43, "vperm", int_ppc_altivec_vperm,
                              v4i32, v4i32, v16i8>;
```
- **EN**: Adds declarative TableGen records such as `VMLADDUHM`, `VPERM` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VMLADDUHM`, `VPERM`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 492-500

```tablegen
def VSEL       : VA1a_Int_Ty<42, "vsel",  int_ppc_altivec_vsel, v4i32>;

// Shuffles.
def VSLDOI  : VAForm_2<44, (outs vrrc:$RT), (ins vrrc:$RA, vrrc:$RB, u4imm:$SH),
                       "vsldoi $RT, $RA, $RB, $SH", IIC_VecFP,
                       [(set v16i8:$RT,
                         (PPCvecshl v16i8:$RA, v16i8:$RB, imm32SExt16:$SH))]>;

// VX-Form instructions.  AltiVec arithmetic ops.
```
- **EN**: Adds declarative TableGen records such as `VSEL`, `VSLDOI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSEL`, `VSLDOI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 501-508

```tablegen
let isCommutable = 1 in {
def VADDFP : VXForm_1<10, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vaddfp $VD, $VA, $VB", IIC_VecFP,
                      [(set v4f32:$VD, (fadd v4f32:$VA, v4f32:$VB))]>;

def VADDUBM : VXForm_1<0, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vaddubm $VD, $VA, $VB", IIC_VecGeneral,
                      [(set v16i8:$VD, (add v16i8:$VA, v16i8:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VADDFP`, `VADDUBM` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VADDFP`, `VADDUBM`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 509-515

```tablegen
def VADDUHM : VXForm_1<64, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vadduhm $VD, $VA, $VB", IIC_VecGeneral,
                      [(set v8i16:$VD, (add v8i16:$VA, v8i16:$VB))]>;
def VADDUWM : VXForm_1<128, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vadduwm $VD, $VA, $VB", IIC_VecGeneral,
                      [(set v4i32:$VD, (add v4i32:$VA, v4i32:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VADDUHM`, `VADDUWM` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VADDUHM`, `VADDUWM`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 516-524

```tablegen
def VADDCUW : VX1_Int_Ty<384, "vaddcuw", int_ppc_altivec_vaddcuw, v4i32>;
def VADDSBS : VX1_Int_Ty<768, "vaddsbs", int_ppc_altivec_vaddsbs, v16i8>;
def VADDSHS : VX1_Int_Ty<832, "vaddshs", int_ppc_altivec_vaddshs, v8i16>;
def VADDSWS : VX1_Int_Ty<896, "vaddsws", int_ppc_altivec_vaddsws, v4i32>;
def VADDUBS : VX1_Int_Ty<512, "vaddubs", int_ppc_altivec_vaddubs, v16i8>;
def VADDUHS : VX1_Int_Ty<576, "vadduhs", int_ppc_altivec_vadduhs, v8i16>;
def VADDUWS : VX1_Int_Ty<640, "vadduws", int_ppc_altivec_vadduws, v4i32>;
} // isCommutable
```
- **EN**: Adds declarative TableGen records such as `VADDCUW`, `VADDSBS`, `VADDSHS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VADDCUW`, `VADDSBS`, `VADDSHS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 525-533

```tablegen
let isCommutable = 1 in
def VAND : VXForm_1<1028, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                    "vand $VD, $VA, $VB", IIC_VecFP,
                    [(set v4i32:$VD, (and v4i32:$VA, v4i32:$VB))]>;
def VANDC : VXForm_1<1092, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                     "vandc $VD, $VA, $VB", IIC_VecFP,
                     [(set v4i32:$VD, (and v4i32:$VA,
                                           (vnot v4i32:$VB)))]>;
```
- **EN**: Adds declarative TableGen records such as `VAND`, `VANDC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VAND`, `VANDC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 534-541

```tablegen
def VCFSX  : VXForm_1<842, (outs vrrc:$VD), (ins u5imm:$VA, vrrc:$VB),
                      "vcfsx $VD, $VB, $VA", IIC_VecFP,
                      [(set v4f32:$VD,
                             (int_ppc_altivec_vcfsx v4i32:$VB, u5imm_timm:$VA))]>;
def VCFUX  : VXForm_1<778, (outs vrrc:$VD), (ins u5imm:$VA, vrrc:$VB),
                      "vcfux $VD, $VB, $VA", IIC_VecFP,
                      [(set v4f32:$VD,
                             (int_ppc_altivec_vcfux v4i32:$VB, u5imm_timm:$VA))]>;
```
- **EN**: Adds declarative TableGen records such as `VCFSX`, `VCFUX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCFSX`, `VCFUX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 542-553

```tablegen
def VCTSXS : VXForm_1<970, (outs vrrc:$VD), (ins u5imm:$VA, vrrc:$VB),
                      "vctsxs $VD, $VB, $VA", IIC_VecFP,
                      [(set v4i32:$VD,
                             (int_ppc_altivec_vctsxs v4f32:$VB, u5imm_timm:$VA))]>;
def VCTUXS : VXForm_1<906, (outs vrrc:$VD), (ins u5imm:$VA, vrrc:$VB),
                      "vctuxs $VD, $VB, $VA", IIC_VecFP,
                      [(set v4i32:$VD,
                             (int_ppc_altivec_vctuxs v4f32:$VB, u5imm_timm:$VA))]>;

// Defines with the UIM field set to 0 for floating-point
// to integer (fp_to_sint/fp_to_uint) conversions and integer
// to floating-point (sint_to_fp/uint_to_fp) conversions.
```
- **EN**: Adds declarative TableGen records such as `VCTSXS`, `VCTUXS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCTSXS`, `VCTUXS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 554-562

```tablegen
let isCodeGenOnly = 1, VA = 0 in {
def VCFSX_0 : VXForm_1<842, (outs vrrc:$VD), (ins vrrc:$VB),
                       "vcfsx $VD, $VB, 0", IIC_VecFP,
                       [(set v4f32:$VD,
                             (int_ppc_altivec_vcfsx v4i32:$VB, 0))]>;
def VCTUXS_0 : VXForm_1<906, (outs vrrc:$VD), (ins vrrc:$VB),
                        "vctuxs $VD, $VB, 0", IIC_VecFP,
                        [(set v4i32:$VD,
                               (int_ppc_altivec_vctuxs v4f32:$VB, 0))]>;
```
- **EN**: Adds declarative TableGen records such as `VCFSX_0`, `VCTUXS_0` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCFSX_0`, `VCTUXS_0`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 563-571

```tablegen
def VCFUX_0 : VXForm_1<778, (outs vrrc:$VD), (ins vrrc:$VB),
                       "vcfux $VD, $VB, 0", IIC_VecFP,
                       [(set v4f32:$VD,
                               (int_ppc_altivec_vcfux v4i32:$VB, 0))]>;
def VCTSXS_0 : VXForm_1<970, (outs vrrc:$VD), (ins vrrc:$VB),
                      "vctsxs $VD, $VB, 0", IIC_VecFP,
                      [(set v4i32:$VD,
                             (int_ppc_altivec_vctsxs v4f32:$VB, 0))]>;
}
```
- **EN**: Adds declarative TableGen records such as `VCFUX_0`, `VCTSXS_0` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCFUX_0`, `VCTSXS_0`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 572-578

```tablegen
def VEXPTEFP : VX2_Int_SP<394, "vexptefp", int_ppc_altivec_vexptefp>;
def VLOGEFP  : VX2_Int_SP<458, "vlogefp",  int_ppc_altivec_vlogefp>;

let isCommutable = 1 in {
def VAVGSB : VX1_Int_Ty<1282, "vavgsb", int_ppc_altivec_vavgsb, v16i8>;
def VAVGSH : VX1_Int_Ty<1346, "vavgsh", int_ppc_altivec_vavgsh, v8i16>;
def VAVGSW : VX1_Int_Ty<1410, "vavgsw", int_ppc_altivec_vavgsw, v4i32>;
```
- **EN**: Adds declarative TableGen records such as `VEXPTEFP`, `VLOGEFP`, `VAVGSB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VEXPTEFP`, `VLOGEFP`, `VAVGSB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 579-585

```tablegen
def VAVGUB : VX1_Int_Ty<1026, "vavgub", int_ppc_altivec_vavgub, v16i8>;
def VAVGUH : VX1_Int_Ty<1090, "vavguh", int_ppc_altivec_vavguh, v8i16>;
def VAVGUW : VX1_Int_Ty<1154, "vavguw", int_ppc_altivec_vavguw, v4i32>;

def VMAXFP : VX1_Int_Ty<1034, "vmaxfp", int_ppc_altivec_vmaxfp, v4f32>;
def VMAXSB : VX1_Int_Ty< 258, "vmaxsb", int_ppc_altivec_vmaxsb, v16i8>;
def VMAXSH : VX1_Int_Ty< 322, "vmaxsh", int_ppc_altivec_vmaxsh, v8i16>;
```
- **EN**: Adds declarative TableGen records such as `VAVGUB`, `VAVGUH`, `VAVGUW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VAVGUB`, `VAVGUH`, `VAVGUW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 586-592

```tablegen
def VMAXSW : VX1_Int_Ty< 386, "vmaxsw", int_ppc_altivec_vmaxsw, v4i32>;
def VMAXUB : VX1_Int_Ty<   2, "vmaxub", int_ppc_altivec_vmaxub, v16i8>;
def VMAXUH : VX1_Int_Ty<  66, "vmaxuh", int_ppc_altivec_vmaxuh, v8i16>;
def VMAXUW : VX1_Int_Ty< 130, "vmaxuw", int_ppc_altivec_vmaxuw, v4i32>;
def VMINFP : VX1_Int_Ty<1098, "vminfp", int_ppc_altivec_vminfp, v4f32>;
def VMINSB : VX1_Int_Ty< 770, "vminsb", int_ppc_altivec_vminsb, v16i8>;
def VMINSH : VX1_Int_Ty< 834, "vminsh", int_ppc_altivec_vminsh, v8i16>;
```
- **EN**: Adds declarative TableGen records such as `VMAXSW`, `VMAXUB`, `VMAXUH` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VMAXSW`, `VMAXUB`, `VMAXUH`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 593-601

```tablegen
def VMINSW : VX1_Int_Ty< 898, "vminsw", int_ppc_altivec_vminsw, v4i32>;
def VMINUB : VX1_Int_Ty< 514, "vminub", int_ppc_altivec_vminub, v16i8>;
def VMINUH : VX1_Int_Ty< 578, "vminuh", int_ppc_altivec_vminuh, v8i16>;
def VMINUW : VX1_Int_Ty< 642, "vminuw", int_ppc_altivec_vminuw, v4i32>;
} // isCommutable

def VMRGHB : VXForm_1< 12, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vmrghb $VD, $VA, $VB", IIC_VecFP,
                      [(set v16i8:$VD, (vmrghb_shuffle v16i8:$VA, v16i8:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VMINSW`, `VMINUB`, `VMINUH` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VMINSW`, `VMINUB`, `VMINUH`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 602-610

```tablegen
def VMRGHH : VXForm_1< 76, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vmrghh $VD, $VA, $VB", IIC_VecFP,
                      [(set v16i8:$VD, (vmrghh_shuffle v16i8:$VA, v16i8:$VB))]>;
def VMRGHW : VXForm_1<140, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vmrghw $VD, $VA, $VB", IIC_VecFP,
                      [(set v16i8:$VD, (vmrghw_shuffle v16i8:$VA, v16i8:$VB))]>;
def VMRGLB : VXForm_1<268, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vmrglb $VD, $VA, $VB", IIC_VecFP,
                      [(set v16i8:$VD, (vmrglb_shuffle v16i8:$VA, v16i8:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VMRGHH`, `VMRGHW`, `VMRGLB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VMRGHH`, `VMRGHW`, `VMRGLB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 611-617

```tablegen
def VMRGLH : VXForm_1<332, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vmrglh $VD, $VA, $VB", IIC_VecFP,
                      [(set v16i8:$VD, (vmrglh_shuffle v16i8:$VA, v16i8:$VB))]>;
def VMRGLW : VXForm_1<396, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vmrglw $VD, $VA, $VB", IIC_VecFP,
                      [(set v16i8:$VD, (vmrglw_shuffle v16i8:$VA, v16i8:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VMRGLH`, `VMRGLW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VMRGLH`, `VMRGLW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 618-625

```tablegen
def VMSUMMBM : VA1a_Int_Ty3<37, "vmsummbm", int_ppc_altivec_vmsummbm,
                            v4i32, v16i8, v4i32>;
def VMSUMSHM : VA1a_Int_Ty3<40, "vmsumshm", int_ppc_altivec_vmsumshm,
                            v4i32, v8i16, v4i32>;
def VMSUMUBM : VA1a_Int_Ty3<36, "vmsumubm", int_ppc_altivec_vmsumubm,
                            v4i32, v16i8, v4i32>;
def VMSUMUHM : VA1a_Int_Ty3<38, "vmsumuhm", int_ppc_altivec_vmsumuhm,
                            v4i32, v8i16, v4i32>;
```
- **EN**: Adds declarative TableGen records such as `VMSUMMBM`, `VMSUMSHM`, `VMSUMUBM` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VMSUMMBM`, `VMSUMSHM`, `VMSUMUBM`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 626-632

```tablegen
let hasSideEffects = 1 in {
  def VMSUMSHS : VA1a_Int_Ty3<41, "vmsumshs", int_ppc_altivec_vmsumshs,
                              v4i32, v8i16, v4i32>;
  def VMSUMUHS : VA1a_Int_Ty3<39, "vmsumuhs", int_ppc_altivec_vmsumuhs,
                              v4i32, v8i16, v4i32>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 633-639

```tablegen
let isCommutable = 1 in {
def VMULESB : VX1_Int_Ty2<776, "vmulesb", int_ppc_altivec_vmulesb,
                          v8i16, v16i8>;
def VMULESH : VX1_Int_Ty2<840, "vmulesh", int_ppc_altivec_vmulesh,
                          v4i32, v8i16>;
def VMULEUB : VX1_Int_Ty2<520, "vmuleub", int_ppc_altivec_vmuleub,
                          v8i16, v16i8>;
```
- **EN**: Adds declarative TableGen records such as `VMULESB`, `VMULESH`, `VMULEUB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VMULESB`, `VMULESH`, `VMULEUB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 640-647

```tablegen
def VMULEUH : VX1_Int_Ty2<584, "vmuleuh", int_ppc_altivec_vmuleuh,
                          v4i32, v8i16>;
def VMULOSB : VX1_Int_Ty2<264, "vmulosb", int_ppc_altivec_vmulosb,
                          v8i16, v16i8>;
def VMULOSH : VX1_Int_Ty2<328, "vmulosh", int_ppc_altivec_vmulosh,
                          v4i32, v8i16>;
def VMULOUB : VX1_Int_Ty2<  8, "vmuloub", int_ppc_altivec_vmuloub,
                          v8i16, v16i8>;
```
- **EN**: Adds declarative TableGen records such as `VMULEUH`, `VMULOSB`, `VMULOSH` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VMULEUH`, `VMULOSB`, `VMULOSH`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 648-654

```tablegen
def VMULOUH : VX1_Int_Ty2< 72, "vmulouh", int_ppc_altivec_vmulouh,
                          v4i32, v8i16>;
} // isCommutable

def VREFP     : VX2_Int_SP<266, "vrefp",     int_ppc_altivec_vrefp>;
def VRFIM     : VX2_Int_SP<714, "vrfim",     int_ppc_altivec_vrfim>;
def VRFIN     : VX2_Int_SP<522, "vrfin",     int_ppc_altivec_vrfin>;
```
- **EN**: Adds declarative TableGen records such as `VMULOUH`, `VREFP`, `VRFIM` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VMULOUH`, `VREFP`, `VRFIM`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 655-663

```tablegen
def VRFIP     : VX2_Int_SP<650, "vrfip",     int_ppc_altivec_vrfip>;
def VRFIZ     : VX2_Int_SP<586, "vrfiz",     int_ppc_altivec_vrfiz>;
def VRSQRTEFP : VX2_Int_SP<330, "vrsqrtefp", int_ppc_altivec_vrsqrtefp>;

def VSUBCUW : VX1_Int_Ty<1408, "vsubcuw", int_ppc_altivec_vsubcuw, v4i32>;

def VSUBFP  : VXForm_1<74, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vsubfp $VD, $VA, $VB", IIC_VecGeneral,
                      [(set v4f32:$VD, (fsub v4f32:$VA, v4f32:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VRFIP`, `VRFIZ`, `VRSQRTEFP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VRFIP`, `VRFIZ`, `VRSQRTEFP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 664-673

```tablegen
def VSUBUBM : VXForm_1<1024, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vsububm $VD, $VA, $VB", IIC_VecGeneral,
                      [(set v16i8:$VD, (sub v16i8:$VA, v16i8:$VB))]>;
def VSUBUHM : VXForm_1<1088, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vsubuhm $VD, $VA, $VB", IIC_VecGeneral,
                      [(set v8i16:$VD, (sub v8i16:$VA, v8i16:$VB))]>;
def VSUBUWM : VXForm_1<1152, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vsubuwm $VD, $VA, $VB", IIC_VecGeneral,
                      [(set v4i32:$VD, (sub v4i32:$VA, v4i32:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VSUBUBM`, `VSUBUHM`, `VSUBUWM` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSUBUBM`, `VSUBUHM`, `VSUBUWM`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 674-680

```tablegen
def VSUBSBS : VX1_Int_Ty<1792, "vsubsbs" , int_ppc_altivec_vsubsbs, v16i8>;
def VSUBSHS : VX1_Int_Ty<1856, "vsubshs" , int_ppc_altivec_vsubshs, v8i16>;
def VSUBSWS : VX1_Int_Ty<1920, "vsubsws" , int_ppc_altivec_vsubsws, v4i32>;
def VSUBUBS : VX1_Int_Ty<1536, "vsububs" , int_ppc_altivec_vsububs, v16i8>;
def VSUBUHS : VX1_Int_Ty<1600, "vsubuhs" , int_ppc_altivec_vsubuhs, v8i16>;
def VSUBUWS : VX1_Int_Ty<1664, "vsubuws" , int_ppc_altivec_vsubuws, v4i32>;
```
- **EN**: Adds declarative TableGen records such as `VSUBSBS`, `VSUBSHS`, `VSUBSWS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSUBSBS`, `VSUBSHS`, `VSUBSWS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 681-688

```tablegen
let hasSideEffects = 1 in {
  def VSUMSWS : VX1_Int_Ty<1928, "vsumsws" , int_ppc_altivec_vsumsws, v4i32>;
  def VSUM2SWS: VX1_Int_Ty<1672, "vsum2sws", int_ppc_altivec_vsum2sws, v4i32>;

  def VSUM4SBS: VX1_Int_Ty3<1800, "vsum4sbs", int_ppc_altivec_vsum4sbs,
                            v4i32, v16i8, v4i32>;
  def VSUM4SHS: VX1_Int_Ty3<1608, "vsum4shs", int_ppc_altivec_vsum4shs,
                            v4i32, v8i16, v4i32>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 689-696

```tablegen
  def VSUM4UBS: VX1_Int_Ty3<1544, "vsum4ubs", int_ppc_altivec_vsum4ubs,
                            v4i32, v16i8, v4i32>;
}

def VNOR : VXForm_1<1284, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                    "vnor $VD, $VA, $VB", IIC_VecFP,
                    [(set v4i32:$VD, (vnot (or v4i32:$VA,
                                               v4i32:$VB)))]>;
```
- **EN**: Adds declarative TableGen records such as `VNOR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VNOR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 697-705

```tablegen
let isCommutable = 1 in {
def VOR : VXForm_1<1156, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vor $VD, $VA, $VB", IIC_VecFP,
                      [(set v4i32:$VD, (or v4i32:$VA, v4i32:$VB))]>;
def VXOR : VXForm_1<1220, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vxor $VD, $VA, $VB", IIC_VecFP,
                      [(set v4i32:$VD, (xor v4i32:$VA, v4i32:$VB))]>;
} // isCommutable
```
- **EN**: Adds declarative TableGen records such as `VOR`, `VXOR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VOR`, `VXOR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 706-713

```tablegen
def VRLB   : VX1_Int_Ty<   4, "vrlb", int_ppc_altivec_vrlb, v16i8>;
def VRLH   : VX1_Int_Ty<  68, "vrlh", int_ppc_altivec_vrlh, v8i16>;
def VRLW   : VXForm_1<132, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vrlw $VD, $VA, $VB", IIC_VecFP, []>;

def VSL    : VX1_Int_Ty< 452, "vsl" , int_ppc_altivec_vsl,  v4i32 >;
def VSLO   : VX1_Int_Ty<1036, "vslo", int_ppc_altivec_vslo, v4i32>;
```
- **EN**: Adds declarative TableGen records such as `VRLB`, `VRLH`, `VRLW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VRLB`, `VRLH`, `VRLW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 714-721

```tablegen
def VSLB   : VX1_Int_Ty< 260, "vslb", int_ppc_altivec_vslb, v16i8>;
def VSLH   : VX1_Int_Ty< 324, "vslh", int_ppc_altivec_vslh, v8i16>;
def VSLW   : VX1_Int_Ty< 388, "vslw", int_ppc_altivec_vslw, v4i32>;

def VSPLTB : VXForm_1<524, (outs vrrc:$VD), (ins u5imm:$VA, vrrc:$VB),
                      "vspltb $VD, $VB, $VA", IIC_VecPerm,
                      [(set v16i8:$VD,
                        (vspltb_shuffle:$VA v16i8:$VB, (undef)))]>;
```
- **EN**: Adds declarative TableGen records such as `VSLB`, `VSLH`, `VSLW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSLB`, `VSLH`, `VSLW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 722-729

```tablegen
def VSPLTH : VXForm_1<588, (outs vrrc:$VD), (ins u5imm:$VA, vrrc:$VB),
                      "vsplth $VD, $VB, $VA", IIC_VecPerm,
                      [(set v16i8:$VD,
                        (vsplth_shuffle:$VA v16i8:$VB, (undef)))]>;
def VSPLTW : VXForm_1<652, (outs vrrc:$VD), (ins u5imm:$VA, vrrc:$VB),
                      "vspltw $VD, $VB, $VA", IIC_VecPerm,
                      [(set v16i8:$VD,
                        (vspltw_shuffle:$VA v16i8:$VB, (undef)))]>;
```
- **EN**: Adds declarative TableGen records such as `VSPLTH`, `VSPLTW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSPLTH`, `VSPLTW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 730-736

```tablegen
let isCodeGenOnly = 1, hasSideEffects = 0 in {
  def VSPLTBs : VXForm_1<524, (outs vrrc:$VD), (ins u5imm:$VA, vfrc:$VB),
                         "vspltb $VD, $VB, $VA", IIC_VecPerm, []>;
  def VSPLTHs : VXForm_1<588, (outs vrrc:$VD), (ins u5imm:$VA, vfrc:$VB),
                         "vsplth $VD, $VB, $VA", IIC_VecPerm, []>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 737-743

```tablegen
def VSR    : VX1_Int_Ty< 708, "vsr"  , int_ppc_altivec_vsr,  v4i32>;
def VSRO   : VX1_Int_Ty<1100, "vsro" , int_ppc_altivec_vsro, v4i32>;

def VSRAB  : VX1_Int_Ty< 772, "vsrab", int_ppc_altivec_vsrab, v16i8>;
def VSRAH  : VX1_Int_Ty< 836, "vsrah", int_ppc_altivec_vsrah, v8i16>;
def VSRAW  : VX1_Int_Ty< 900, "vsraw", int_ppc_altivec_vsraw, v4i32>;
def VSRB   : VX1_Int_Ty< 516, "vsrb" , int_ppc_altivec_vsrb , v16i8>;
```
- **EN**: Adds declarative TableGen records such as `VSR`, `VSRO`, `VSRAB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSR`, `VSRO`, `VSRAB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 744-750

```tablegen
def VSRH   : VX1_Int_Ty< 580, "vsrh" , int_ppc_altivec_vsrh , v8i16>;
def VSRW   : VX1_Int_Ty< 644, "vsrw" , int_ppc_altivec_vsrw , v4i32>;


def VSPLTISB : VXForm_3<780, (outs vrrc:$VD), (ins s5imm:$IMM),
                       "vspltisb $VD, $IMM", IIC_VecPerm,
                       [(set v16i8:$VD, (v16i8 vecspltisb:$IMM))]>;
```
- **EN**: Adds declarative TableGen records such as `VSRH`, `VSRW`, `VSPLTISB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSRH`, `VSRW`, `VSPLTISB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 751-758

```tablegen
def VSPLTISH : VXForm_3<844, (outs vrrc:$VD), (ins s5imm:$IMM),
                       "vspltish $VD, $IMM", IIC_VecPerm,
                       [(set v8i16:$VD, (v8i16 vecspltish:$IMM))]>;
def VSPLTISW : VXForm_3<908, (outs vrrc:$VD), (ins s5imm:$IMM),
                       "vspltisw $VD, $IMM", IIC_VecPerm,
                       [(set v4i32:$VD, (v4i32 vecspltisw:$IMM))]>;

// Vector Pack.
```
- **EN**: Adds declarative TableGen records such as `VSPLTISH`, `VSPLTISW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSPLTISH`, `VSPLTISW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 759-765

```tablegen
def VPKPX   : VX1_Int_Ty2<782, "vpkpx", int_ppc_altivec_vpkpx,
                          v8i16, v4i32>;
let hasSideEffects = 1 in {
  def VPKSHSS : VX1_Int_Ty2<398, "vpkshss", int_ppc_altivec_vpkshss,
                            v16i8, v8i16>;
  def VPKSHUS : VX1_Int_Ty2<270, "vpkshus", int_ppc_altivec_vpkshus,
                            v16i8, v8i16>;
```
- **EN**: Adds declarative TableGen records such as `VPKPX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VPKPX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 766-774

```tablegen
  def VPKSWSS : VX1_Int_Ty2<462, "vpkswss", int_ppc_altivec_vpkswss,
                            v8i16, v4i32>;
  def VPKSWUS : VX1_Int_Ty2<334, "vpkswus", int_ppc_altivec_vpkswus,
                            v8i16, v4i32>;
  def VPKUHUS : VX1_Int_Ty2<142, "vpkuhus", int_ppc_altivec_vpkuhus,
                            v16i8, v8i16>;
  def VPKUWUS : VX1_Int_Ty2<206, "vpkuwus", int_ppc_altivec_vpkuwus,
                            v8i16, v4i32>;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 775-784

```tablegen
def VPKUHUM : VXForm_1<14, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                       "vpkuhum $VD, $VA, $VB", IIC_VecFP,
                       [(set v16i8:$VD,
                         (vpkuhum_shuffle v16i8:$VA, v16i8:$VB))]>;
def VPKUWUM : VXForm_1<78, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                       "vpkuwum $VD, $VA, $VB", IIC_VecFP,
                       [(set v16i8:$VD,
                         (vpkuwum_shuffle v16i8:$VA, v16i8:$VB))]>;

// Vector Unpack.
```
- **EN**: Adds declarative TableGen records such as `VPKUHUM`, `VPKUWUM` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VPKUHUM`, `VPKUWUM`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 785-792

```tablegen
def VUPKHPX : VX2_Int_Ty2<846, "vupkhpx", int_ppc_altivec_vupkhpx,
                          v4i32, v8i16>;
def VUPKHSB : VX2_Int_Ty2<526, "vupkhsb", int_ppc_altivec_vupkhsb,
                          v8i16, v16i8>;
def VUPKHSH : VX2_Int_Ty2<590, "vupkhsh", int_ppc_altivec_vupkhsh,
                          v4i32, v8i16>;
def VUPKLPX : VX2_Int_Ty2<974, "vupklpx", int_ppc_altivec_vupklpx,
                          v4i32, v8i16>;
```
- **EN**: Adds declarative TableGen records such as `VUPKHPX`, `VUPKHSB`, `VUPKHSH` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VUPKHPX`, `VUPKHSB`, `VUPKHSH`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 793-800

```tablegen
def VUPKLSB : VX2_Int_Ty2<654, "vupklsb", int_ppc_altivec_vupklsb,
                          v8i16, v16i8>;
def VUPKLSH : VX2_Int_Ty2<718, "vupklsh", int_ppc_altivec_vupklsh,
                          v4i32, v8i16>;


// Altivec Comparisons.
```
- **EN**: Adds declarative TableGen records such as `VUPKLSB`, `VUPKLSH` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VUPKLSB`, `VUPKLSH`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 801-808

```tablegen
class VCMP<bits<10> xo, string asmstr, ValueType Ty>
  : VXRForm_1<xo, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB), asmstr,
              IIC_VecFPCompare,
              [(set Ty:$VD, (Ty (PPCvcmp Ty:$VA, Ty:$VB, xo)))]>;
class VCMP_rec<bits<10> xo, string asmstr, ValueType Ty>
  : VXRForm_1<xo, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB), asmstr,
              IIC_VecFPCompare,
              [(set Ty:$VD, (Ty (PPCvcmp_rec Ty:$VA, Ty:$VB, xo)))]> {
```
- **EN**: Declares a backend-facing type `VCMP`, `VCMP_rec` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VCMP`, `VCMP_rec`，并勾勒出周边代码会依赖的接口或状态。

### Lines 809-815

```tablegen
  let Defs = [CR6];
  let RC = 1;
}

// f32 element comparisons.0
def VCMPBFP   : VCMP <966, "vcmpbfp $VD, $VA, $VB"  , v4f32>;
def VCMPBFP_rec  : VCMP_rec<966, "vcmpbfp. $VD, $VA, $VB" , v4f32>;
```
- **EN**: Adds declarative TableGen records such as `VCMPBFP`, `VCMPBFP_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCMPBFP`, `VCMPBFP_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 816-823

```tablegen
def VCMPEQFP  : VCMP <198, "vcmpeqfp $VD, $VA, $VB" , v4f32>;
def VCMPEQFP_rec : VCMP_rec<198, "vcmpeqfp. $VD, $VA, $VB", v4f32>;
def VCMPGEFP  : VCMP <454, "vcmpgefp $VD, $VA, $VB" , v4f32>;
def VCMPGEFP_rec : VCMP_rec<454, "vcmpgefp. $VD, $VA, $VB", v4f32>;
def VCMPGTFP  : VCMP <710, "vcmpgtfp $VD, $VA, $VB" , v4f32>;
def VCMPGTFP_rec : VCMP_rec<710, "vcmpgtfp. $VD, $VA, $VB", v4f32>;

// i8 element comparisons.
```
- **EN**: Adds declarative TableGen records such as `VCMPEQFP`, `VCMPEQFP_rec`, `VCMPGEFP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCMPEQFP`, `VCMPEQFP_rec`, `VCMPGEFP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 824-831

```tablegen
def VCMPEQUB  : VCMP <  6, "vcmpequb $VD, $VA, $VB" , v16i8>;
def VCMPEQUB_rec : VCMP_rec<  6, "vcmpequb. $VD, $VA, $VB", v16i8>;
def VCMPGTSB  : VCMP <774, "vcmpgtsb $VD, $VA, $VB" , v16i8>;
def VCMPGTSB_rec : VCMP_rec<774, "vcmpgtsb. $VD, $VA, $VB", v16i8>;
def VCMPGTUB  : VCMP <518, "vcmpgtub $VD, $VA, $VB" , v16i8>;
def VCMPGTUB_rec : VCMP_rec<518, "vcmpgtub. $VD, $VA, $VB", v16i8>;

// i16 element comparisons.
```
- **EN**: Adds declarative TableGen records such as `VCMPEQUB`, `VCMPEQUB_rec`, `VCMPGTSB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCMPEQUB`, `VCMPEQUB_rec`, `VCMPGTSB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 832-839

```tablegen
def VCMPEQUH  : VCMP < 70, "vcmpequh $VD, $VA, $VB" , v8i16>;
def VCMPEQUH_rec : VCMP_rec< 70, "vcmpequh. $VD, $VA, $VB", v8i16>;
def VCMPGTSH  : VCMP <838, "vcmpgtsh $VD, $VA, $VB" , v8i16>;
def VCMPGTSH_rec : VCMP_rec<838, "vcmpgtsh. $VD, $VA, $VB", v8i16>;
def VCMPGTUH  : VCMP <582, "vcmpgtuh $VD, $VA, $VB" , v8i16>;
def VCMPGTUH_rec : VCMP_rec<582, "vcmpgtuh. $VD, $VA, $VB", v8i16>;

// i32 element comparisons.
```
- **EN**: Adds declarative TableGen records such as `VCMPEQUH`, `VCMPEQUH_rec`, `VCMPGTSH` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCMPEQUH`, `VCMPEQUH_rec`, `VCMPGTSH`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 840-846

```tablegen
def VCMPEQUW  : VCMP <134, "vcmpequw $VD, $VA, $VB" , v4i32>;
def VCMPEQUW_rec : VCMP_rec<134, "vcmpequw. $VD, $VA, $VB", v4i32>;
def VCMPGTSW  : VCMP <902, "vcmpgtsw $VD, $VA, $VB" , v4i32>;
def VCMPGTSW_rec : VCMP_rec<902, "vcmpgtsw. $VD, $VA, $VB", v4i32>;
def VCMPGTUW  : VCMP <646, "vcmpgtuw $VD, $VA, $VB" , v4i32>;
def VCMPGTUW_rec : VCMP_rec<646, "vcmpgtuw. $VD, $VA, $VB", v4i32>;
```
- **EN**: Adds declarative TableGen records such as `VCMPEQUW`, `VCMPEQUW_rec`, `VCMPGTSW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCMPEQUW`, `VCMPEQUW_rec`, `VCMPGTSW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 847-855

```tablegen
let isCodeGenOnly = 1, isMoveImm = 1, isAsCheapAsAMove = 1,
    isReMaterializable = 1 in {

def V_SET0B : VXForm_setzero<1220, (outs vrrc:$VD), (ins),
                      "vxor $VD, $VD, $VD", IIC_VecFP,
                      [(set v16i8:$VD, (v16i8 immAllZerosV))]>;
def V_SET0H : VXForm_setzero<1220, (outs vrrc:$VD), (ins),
                      "vxor $VD, $VD, $VD", IIC_VecFP,
                      [(set v8i16:$VD, (v8i16 immAllZerosV))]>;
```
- **EN**: Adds declarative TableGen records such as `V_SET0B`, `V_SET0H` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `V_SET0B`, `V_SET0H`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 856-863

```tablegen
def V_SET0  : VXForm_setzero<1220, (outs vrrc:$VD), (ins),
                      "vxor $VD, $VD, $VD", IIC_VecFP,
                      [(set v4i32:$VD, (v4i32 immAllZerosV))]>;

let IMM=-1 in {
def V_SETALLONESB : VXForm_3<908, (outs vrrc:$VD), (ins),
                      "vspltisw $VD, -1", IIC_VecFP,
                      [(set v16i8:$VD, (v16i8 immAllOnesV))]>;
```
- **EN**: Adds declarative TableGen records such as `V_SET0`, `V_SETALLONESB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `V_SET0`, `V_SETALLONESB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 864-873

```tablegen
def V_SETALLONESH : VXForm_3<908, (outs vrrc:$VD), (ins),
                      "vspltisw $VD, -1", IIC_VecFP,
                      [(set v8i16:$VD, (v8i16 immAllOnesV))]>;
def V_SETALLONES  : VXForm_3<908, (outs vrrc:$VD), (ins),
                      "vspltisw $VD, -1", IIC_VecFP,
                      [(set v4i32:$VD, (v4i32 immAllOnesV))]>;
}
}
} // VALU Operations.
```
- **EN**: Adds declarative TableGen records such as `V_SETALLONESH`, `V_SETALLONES` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `V_SETALLONESH`, `V_SETALLONES`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 874-883

```tablegen
//===----------------------------------------------------------------------===//
// Additional Altivec Patterns
//

// Extended mnemonics
def : InstAlias<"vmr $vD, $vA", (VOR vrrc:$vD, vrrc:$vA, vrrc:$vA)>;
def : InstAlias<"vnot $vD, $vA", (VNOR vrrc:$vD, vrrc:$vA, vrrc:$vA)>;

// This is a nop on all supported architectures and the AIX assembler
// doesn't support it (and will not be updated to support it).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Additional Altivec Patterns". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Additional Altivec Patterns”。 这些声明会进入生成式模式匹配逻辑。

### Lines 884-891

```tablegen
let Predicates = [IsAIX] in
def : Pat<(int_ppc_altivec_dssall), (NOP)>;
let Predicates = [NotAIX] in
def : Pat<(int_ppc_altivec_dssall), (DSSALL)>;

// Rotates.
def : Pat<(v16i8 (rotl v16i8:$vA, v16i8:$vB)),
          (v16i8 (VRLB v16i8:$vA, v16i8:$vB))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 892-901

```tablegen
def : Pat<(v8i16 (rotl v8i16:$vA, v8i16:$vB)),
          (v8i16 (VRLH v8i16:$vA, v8i16:$vB))>;
let Predicates = [IsNotISAFuture] in {
def : Pat<(v4i32 (rotl v4i32:$vA, v4i32:$vB)),
          (v4i32 (VRLW v4i32:$vA, v4i32:$vB))>;
def : Pat<(v4i32 (int_ppc_altivec_vrlw v4i32:$vA, v4i32:$vB)),
          (v4i32 (VRLW v4i32:$vA, v4i32:$vB))>;
}

// Multiply
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 902-908

```tablegen
def : Pat<(mul v8i16:$vA, v8i16:$vB), (VMLADDUHM $vA, $vB, (v8i16(V_SET0H)))>;

// Add
def : Pat<(add (mul v8i16:$vA, v8i16:$vB), v8i16:$vC), (VMLADDUHM $vA, $vB, $vC)>;


// Fused negated multiply-subtract
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Add".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Add”。

### Lines 909-915

```tablegen
def : Pat<(v4f32 (desugared_fneg
                    (int_ppc_altivec_vmaddfp v4f32:$RA, v4f32:$RC,
                         (desugared_fneg v4f32:$RB)))),
          (VNMSUBFP $RA, $RC, $RB)>;

// Saturating adds/subtracts.
def : Pat<(v16i8 (saddsat v16i8:$vA, v16i8:$vB)), (v16i8 (VADDSBS $vA, $vB))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 916-922

```tablegen
def : Pat<(v16i8 (uaddsat v16i8:$vA, v16i8:$vB)), (v16i8 (VADDUBS $vA, $vB))>;
def : Pat<(v8i16 (saddsat v8i16:$vA, v8i16:$vB)), (v8i16 (VADDSHS $vA, $vB))>;
def : Pat<(v8i16 (uaddsat v8i16:$vA, v8i16:$vB)), (v8i16 (VADDUHS $vA, $vB))>;
def : Pat<(v4i32 (saddsat v4i32:$vA, v4i32:$vB)), (v4i32 (VADDSWS $vA, $vB))>;
def : Pat<(v4i32 (uaddsat v4i32:$vA, v4i32:$vB)), (v4i32 (VADDUWS $vA, $vB))>;
def : Pat<(v16i8 (ssubsat v16i8:$vA, v16i8:$vB)), (v16i8 (VSUBSBS $vA, $vB))>;
def : Pat<(v16i8 (usubsat v16i8:$vA, v16i8:$vB)), (v16i8 (VSUBUBS $vA, $vB))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 923-931

```tablegen
def : Pat<(v8i16 (ssubsat v8i16:$vA, v8i16:$vB)), (v8i16 (VSUBSHS $vA, $vB))>;
def : Pat<(v8i16 (usubsat v8i16:$vA, v8i16:$vB)), (v8i16 (VSUBUHS $vA, $vB))>;
def : Pat<(v4i32 (ssubsat v4i32:$vA, v4i32:$vB)), (v4i32 (VSUBSWS $vA, $vB))>;
def : Pat<(v4i32 (usubsat v4i32:$vA, v4i32:$vB)), (v4i32 (VSUBUWS $vA, $vB))>;

// Loads.
def : Pat<(v4i32 (load ForceXForm:$src)), (LVX ForceXForm:$src)>;

// Stores.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 932-938

```tablegen
def : Pat<(store v4i32:$rS, ForceXForm:$dst),
          (STVX $rS, ForceXForm:$dst)>;

// Bit conversions.
def : Pat<(v16i8 (bitconvert (v8i16 VRRC:$src))), (v16i8 VRRC:$src)>;
def : Pat<(v16i8 (bitconvert (v4i32 VRRC:$src))), (v16i8 VRRC:$src)>;
def : Pat<(v16i8 (bitconvert (v4f32 VRRC:$src))), (v16i8 VRRC:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 939-945

```tablegen
def : Pat<(v16i8 (bitconvert (v2i64 VRRC:$src))), (v16i8 VRRC:$src)>;
def : Pat<(v16i8 (bitconvert (v1i128 VRRC:$src))), (v16i8 VRRC:$src)>;

def : Pat<(v8i16 (bitconvert (v16i8 VRRC:$src))), (v8i16 VRRC:$src)>;
def : Pat<(v8i16 (bitconvert (v4i32 VRRC:$src))), (v8i16 VRRC:$src)>;
def : Pat<(v8i16 (bitconvert (v4f32 VRRC:$src))), (v8i16 VRRC:$src)>;
def : Pat<(v8i16 (bitconvert (v2i64 VRRC:$src))), (v8i16 VRRC:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 946-953

```tablegen
def : Pat<(v8i16 (bitconvert (v1i128 VRRC:$src))), (v8i16 VRRC:$src)>;

def : Pat<(v4i32 (bitconvert (v16i8 VRRC:$src))), (v4i32 VRRC:$src)>;
def : Pat<(v4i32 (bitconvert (v8i16 VRRC:$src))), (v4i32 VRRC:$src)>;
def : Pat<(v4i32 (bitconvert (v4f32 VRRC:$src))), (v4i32 VRRC:$src)>;
def : Pat<(v4i32 (bitconvert (v2i64 VRRC:$src))), (v4i32 VRRC:$src)>;
def : Pat<(v4i32 (bitconvert (v1i128 VRRC:$src))), (v4i32 VRRC:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 954-960

```tablegen
def : Pat<(v4f32 (bitconvert (v16i8 VRRC:$src))), (v4f32 VRRC:$src)>;
def : Pat<(v4f32 (bitconvert (v8i16 VRRC:$src))), (v4f32 VRRC:$src)>;
def : Pat<(v4f32 (bitconvert (v4i32 VRRC:$src))), (v4f32 VRRC:$src)>;
def : Pat<(v4f32 (bitconvert (v2i64 VRRC:$src))), (v4f32 VRRC:$src)>;
def : Pat<(v4f32 (bitconvert (v1i128 VRRC:$src))), (v4f32 VRRC:$src)>;

def : Pat<(v2i64 (bitconvert (v16i8 VRRC:$src))), (v2i64 VRRC:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 961-967

```tablegen
def : Pat<(v2i64 (bitconvert (v8i16 VRRC:$src))), (v2i64 VRRC:$src)>;
def : Pat<(v2i64 (bitconvert (v4i32 VRRC:$src))), (v2i64 VRRC:$src)>;
def : Pat<(v2i64 (bitconvert (v4f32 VRRC:$src))), (v2i64 VRRC:$src)>;
def : Pat<(v2i64 (bitconvert (v1i128 VRRC:$src))), (v2i64 VRRC:$src)>;

def : Pat<(v1i128 (bitconvert (v16i8 VRRC:$src))), (v1i128 VRRC:$src)>;
def : Pat<(v1i128 (bitconvert (v8i16 VRRC:$src))), (v1i128 VRRC:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 968-974

```tablegen
def : Pat<(v1i128 (bitconvert (v4i32 VRRC:$src))), (v1i128 VRRC:$src)>;
def : Pat<(v1i128 (bitconvert (v4f32 VRRC:$src))), (v1i128 VRRC:$src)>;
def : Pat<(v1i128 (bitconvert (v2i64 VRRC:$src))), (v1i128 VRRC:$src)>;

def : Pat<(f128 (bitconvert (v16i8 VRRC:$src))), (f128 VRRC:$src)>;
def : Pat<(f128 (bitconvert (v8i16 VRRC:$src))), (f128 VRRC:$src)>;
def : Pat<(f128 (bitconvert (v4i32 VRRC:$src))), (f128 VRRC:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 975-981

```tablegen
def : Pat<(f128 (bitconvert (v4f32 VRRC:$src))), (f128 VRRC:$src)>;
def : Pat<(f128 (bitconvert (v2f64 VRRC:$src))), (f128 VRRC:$src)>;

def : Pat<(v16i8 (bitconvert (f128 VRRC:$src))), (v16i8 VRRC:$src)>;
def : Pat<(v8i16 (bitconvert (f128 VRRC:$src))), (v8i16 VRRC:$src)>;
def : Pat<(v4i32 (bitconvert (f128 VRRC:$src))), (v4i32 VRRC:$src)>;
def : Pat<(v4f32 (bitconvert (f128 VRRC:$src))), (v4f32 VRRC:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 982-988

```tablegen
def : Pat<(v2f64 (bitconvert (f128 VRRC:$src))), (v2f64 VRRC:$src)>;

// Max/Min
def : Pat<(v16i8 (umax v16i8:$src1, v16i8:$src2)),
          (v16i8 (VMAXUB $src1, $src2))>;
def : Pat<(v16i8 (smax v16i8:$src1, v16i8:$src2)),
          (v16i8 (VMAXSB $src1, $src2))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 989-996

```tablegen
def : Pat<(v8i16 (umax v8i16:$src1, v8i16:$src2)),
          (v8i16 (VMAXUH $src1, $src2))>;
def : Pat<(v8i16 (smax v8i16:$src1, v8i16:$src2)),
          (v8i16 (VMAXSH $src1, $src2))>;
def : Pat<(v4i32 (umax v4i32:$src1, v4i32:$src2)),
          (v4i32 (VMAXUW $src1, $src2))>;
def : Pat<(v4i32 (smax v4i32:$src1, v4i32:$src2)),
          (v4i32 (VMAXSW $src1, $src2))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 997-1004

```tablegen
def : Pat<(v16i8 (umin v16i8:$src1, v16i8:$src2)),
          (v16i8 (VMINUB $src1, $src2))>;
def : Pat<(v16i8 (smin v16i8:$src1, v16i8:$src2)),
          (v16i8 (VMINSB $src1, $src2))>;
def : Pat<(v8i16 (umin v8i16:$src1, v8i16:$src2)),
          (v8i16 (VMINUH $src1, $src2))>;
def : Pat<(v8i16 (smin v8i16:$src1, v8i16:$src2)),
          (v8i16 (VMINSH $src1, $src2))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1005-1012

```tablegen
def : Pat<(v4i32 (umin v4i32:$src1, v4i32:$src2)),
          (v4i32 (VMINUW $src1, $src2))>;
def : Pat<(v4i32 (smin v4i32:$src1, v4i32:$src2)),
          (v4i32 (VMINSW $src1, $src2))>;

// Shuffles.

// Match vsldoi(x,x), vpkuwum(x,x), vpkuhum(x,x)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Shuffles.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Shuffles.”。

### Lines 1013-1025

```tablegen
def:Pat<(vsldoi_unary_shuffle:$in v16i8:$vA, undef),
        (VSLDOI $vA, $vA, (VSLDOI_unary_get_imm $in))>;
def:Pat<(vpkuwum_unary_shuffle v16i8:$vA, undef),
        (VPKUWUM $vA, $vA)>;
def:Pat<(vpkuhum_unary_shuffle v16i8:$vA, undef),
        (VPKUHUM $vA, $vA)>;
def:Pat<(vsldoi_shuffle:$SH v16i8:$vA, v16i8:$vB),
        (VSLDOI v16i8:$vA, v16i8:$vB, (VSLDOI_get_imm $SH))>;


// Match vsldoi(y,x), vpkuwum(y,x), vpkuhum(y,x), i.e., swapped operands.
// These fragments are matched for little-endian, where the inputs must
// be swapped for correct semantics.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1026-1033

```tablegen
def:Pat<(vsldoi_swapped_shuffle:$in v16i8:$vA, v16i8:$vB),
        (VSLDOI $vB, $vA, (VSLDOI_swapped_get_imm $in))>;
def:Pat<(vpkuwum_swapped_shuffle v16i8:$vA, v16i8:$vB),
        (VPKUWUM $vB, $vA)>;
def:Pat<(vpkuhum_swapped_shuffle v16i8:$vA, v16i8:$vB),
        (VPKUHUM $vB, $vA)>;

// Match vmrg*(x,x)
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1034-1041

```tablegen
def:Pat<(vmrglb_unary_shuffle v16i8:$vA, undef),
        (VMRGLB $vA, $vA)>;
def:Pat<(vmrglh_unary_shuffle v16i8:$vA, undef),
        (VMRGLH $vA, $vA)>;
def:Pat<(vmrglw_unary_shuffle v16i8:$vA, undef),
        (VMRGLW $vA, $vA)>;
def:Pat<(vmrghb_unary_shuffle v16i8:$vA, undef),
        (VMRGHB $vA, $vA)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1042-1049

```tablegen
def:Pat<(vmrghh_unary_shuffle v16i8:$vA, undef),
        (VMRGHH $vA, $vA)>;
def:Pat<(vmrghw_unary_shuffle v16i8:$vA, undef),
        (VMRGHW $vA, $vA)>;

// Match vmrg*(y,x), i.e., swapped operands.  These fragments
// are matched for little-endian, where the inputs must be
// swapped for correct semantics.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Match vmrg*(y,x), i.e., swapped operands.  These fragments".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Match vmrg*(y,x), i.e., swapped operands.  These fragments”。

### Lines 1050-1057

```tablegen
def:Pat<(vmrglb_swapped_shuffle v16i8:$vA, v16i8:$vB),
        (VMRGLB $vB, $vA)>;
def:Pat<(vmrglh_swapped_shuffle v16i8:$vA, v16i8:$vB),
        (VMRGLH $vB, $vA)>;
def:Pat<(vmrglw_swapped_shuffle v16i8:$vA, v16i8:$vB),
        (VMRGLW $vB, $vA)>;
def:Pat<(vmrghb_swapped_shuffle v16i8:$vA, v16i8:$vB),
        (VMRGHB $vB, $vA)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1058-1065

```tablegen
def:Pat<(vmrghh_swapped_shuffle v16i8:$vA, v16i8:$vB),
        (VMRGHH $vB, $vA)>;
def:Pat<(vmrghw_swapped_shuffle v16i8:$vA, v16i8:$vB),
        (VMRGHW $vB, $vA)>;

// Logical Operations
def : Pat<(vnot v4i32:$vA), (VNOR $vA, $vA)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1066-1074

```tablegen
def : Pat<(vnot (or v4i32:$A, v4i32:$B)),
          (VNOR $A, $B)>;
def : Pat<(and v4i32:$A, (vnot v4i32:$B)),
          (VANDC $A, $B)>;

def : Pat<(fmul v4f32:$vA, v4f32:$vB),
          (VMADDFP $vA, $vB,
             (v4i32 (VSLW (v4i32 (V_SETALLONES)), (v4i32 (V_SETALLONES)))))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1075-1082

```tablegen
def : Pat<(PPCfnmsub v4f32:$A, v4f32:$B, v4f32:$C),
          (VNMSUBFP $A, $B, $C)>;

def : Pat<(int_ppc_altivec_vmaddfp v4f32:$A, v4f32:$B, v4f32:$C),
          (VMADDFP $A, $B, $C)>;
def : Pat<(int_ppc_altivec_vnmsubfp v4f32:$A, v4f32:$B, v4f32:$C),
          (VNMSUBFP $A, $B, $C)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1083-1091

```tablegen
def : Pat<(PPCvperm v16i8:$vA, v16i8:$vB, v16i8:$vC),
          (VPERM $vA, $vB, $vC)>;
def : Pat<(PPCvperm v2f64:$vA, v2f64:$vB, v16i8:$vC),
          (VPERM $vA, $vB, $vC)>;

def : Pat<(PPCfre v4f32:$A), (VREFP $A)>;
def : Pat<(PPCfrsqrte v4f32:$A), (VRSQRTEFP $A)>;

// Vector shifts
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1092-1099

```tablegen
def : Pat<(v16i8 (shl v16i8:$vA, v16i8:$vB)),
          (v16i8 (VSLB $vA, $vB))>;
def : Pat<(v8i16 (shl v8i16:$vA, v8i16:$vB)),
          (v8i16 (VSLH $vA, $vB))>;
def : Pat<(v4i32 (shl v4i32:$vA, v4i32:$vB)),
          (v4i32 (VSLW $vA, $vB))>;
def : Pat<(v1i128 (shl v1i128:$vA, v1i128:$vB)),
          (v1i128 (VSL (v16i8 (VSLO $vA, $vB)), (v16i8 (VSPLTB 15, $vB))))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1100-1108

```tablegen
def : Pat<(v16i8 (PPCshl v16i8:$vA, v16i8:$vB)),
          (v16i8 (VSLB $vA, $vB))>;
def : Pat<(v8i16 (PPCshl v8i16:$vA, v8i16:$vB)),
          (v8i16 (VSLH $vA, $vB))>;
def : Pat<(v4i32 (PPCshl v4i32:$vA, v4i32:$vB)),
          (v4i32 (VSLW $vA, $vB))>;
def : Pat<(v1i128 (PPCshl v1i128:$vA, v1i128:$vB)),
          (v1i128 (VSL (v16i8 (VSLO $vA, $vB)), (v16i8 (VSPLTB 15, $vB))))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1109-1116

```tablegen
def : Pat<(v16i8 (srl v16i8:$vA, v16i8:$vB)),
          (v16i8 (VSRB $vA, $vB))>;
def : Pat<(v8i16 (srl v8i16:$vA, v8i16:$vB)),
          (v8i16 (VSRH $vA, $vB))>;
def : Pat<(v4i32 (srl v4i32:$vA, v4i32:$vB)),
          (v4i32 (VSRW $vA, $vB))>;
def : Pat<(v1i128 (srl v1i128:$vA, v1i128:$vB)),
          (v1i128 (VSR (v16i8 (VSRO $vA, $vB)), (v16i8 (VSPLTB 15, $vB))))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1117-1125

```tablegen
def : Pat<(v16i8 (PPCsrl v16i8:$vA, v16i8:$vB)),
          (v16i8 (VSRB $vA, $vB))>;
def : Pat<(v8i16 (PPCsrl v8i16:$vA, v8i16:$vB)),
          (v8i16 (VSRH $vA, $vB))>;
def : Pat<(v4i32 (PPCsrl v4i32:$vA, v4i32:$vB)),
          (v4i32 (VSRW $vA, $vB))>;
def : Pat<(v1i128 (PPCsrl v1i128:$vA, v1i128:$vB)),
          (v1i128 (VSR (v16i8 (VSRO $vA, $vB)), (v16i8 (VSPLTB 15, $vB))))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1126-1133

```tablegen
def : Pat<(v16i8 (sra v16i8:$vA, v16i8:$vB)),
          (v16i8 (VSRAB $vA, $vB))>;
def : Pat<(v8i16 (sra v8i16:$vA, v8i16:$vB)),
          (v8i16 (VSRAH $vA, $vB))>;
def : Pat<(v4i32 (sra v4i32:$vA, v4i32:$vB)),
          (v4i32 (VSRAW $vA, $vB))>;
def : Pat<(v16i8 (PPCsra v16i8:$vA, v16i8:$vB)),
          (v16i8 (VSRAB $vA, $vB))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1134-1141

```tablegen
def : Pat<(v8i16 (PPCsra v8i16:$vA, v8i16:$vB)),
          (v8i16 (VSRAH $vA, $vB))>;
def : Pat<(v4i32 (PPCsra v4i32:$vA, v4i32:$vB)),
          (v4i32 (VSRAW $vA, $vB))>;

// Float to integer and integer to float conversions
def : Pat<(v4i32 (fp_to_sint v4f32:$vA)),
           (VCTSXS_0 $vA)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1142-1149

```tablegen
def : Pat<(v4i32 (fp_to_uint v4f32:$vA)),
           (VCTUXS_0 $vA)>;
def : Pat<(v4f32 (sint_to_fp v4i32:$vA)),
           (VCFSX_0 $vA)>;
def : Pat<(v4f32 (uint_to_fp v4i32:$vA)),
           (VCFUX_0 $vA)>;

// Floating-point rounding
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1150-1159

```tablegen
def : Pat<(v4f32 (ffloor v4f32:$vA)),
          (VRFIM $vA)>;
def : Pat<(v4f32 (fceil v4f32:$vA)),
          (VRFIP $vA)>;
def : Pat<(v4f32 (ftrunc v4f32:$vA)),
          (VRFIZ $vA)>;
def : Pat<(v4f32 (fnearbyint v4f32:$vA)),
          (VRFIN $vA)>;

// Vector selection
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1160-1167

```tablegen
def : Pat<(v16i8 (vselect v16i8:$vA, v16i8:$vB, v16i8:$vC)),
          (VSEL $vC, $vB, $vA)>;
def : Pat<(v8i16 (vselect v8i16:$vA, v8i16:$vB, v8i16:$vC)),
          (VSEL $vC, $vB, $vA)>;
def : Pat<(v4i32 (vselect v4i32:$vA, v4i32:$vB, v4i32:$vC)),
          (VSEL $vC, $vB, $vA)>;
def : Pat<(v2i64 (vselect v2i64:$vA, v2i64:$vB, v2i64:$vC)),
          (VSEL $vC, $vB, $vA)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1168-1175

```tablegen
def : Pat<(v4f32 (vselect v4i32:$vA, v4f32:$vB, v4f32:$vC)),
          (VSEL $vC, $vB, $vA)>;
def : Pat<(v2f64 (vselect v2i64:$vA, v2f64:$vB, v2f64:$vC)),
          (VSEL $vC, $vB, $vA)>;
def : Pat<(v1i128 (vselect v1i128:$vA, v1i128:$vB, v1i128:$vC)),
          (VSEL $vC, $vB, $vA)>;

// Vector Integer Average Instructions
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1176-1183

```tablegen
def : Pat<(v4i32 (avgceils v4i32:$src1, v4i32:$src2)),
          (v4i32 (VAVGSW $src1, $src2))>;
def : Pat<(v8i16 (avgceils v8i16:$src1, v8i16:$src2)),
          (v8i16 (VAVGSH $src1, $src2))>;
def : Pat<(v16i8 (avgceils v16i8:$src1, v16i8:$src2)),
          (v16i8 (VAVGSB $src1, $src2))>;
def : Pat<(v4i32 (avgceilu v4i32:$src1, v4i32:$src2)),
          (v4i32 (VAVGUW $src1, $src2))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1184-1190

```tablegen
def : Pat<(v8i16 (avgceilu v8i16:$src1, v8i16:$src2)),
          (v8i16 (VAVGUH $src1, $src2))>;
def : Pat<(v16i8 (avgceilu v16i8:$src1, v16i8:$src2)),
          (v16i8 (VAVGUB $src1, $src2))>;

def : Pat<(v16i8 (shl v16i8:$vA, (v16i8 (immEQOneV)))),
          (v16i8 (VADDUBM $vA, $vA))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1191-1198

```tablegen
def : Pat<(v8i16 (shl v8i16:$vA, (v8i16 (immEQOneV)))),
          (v8i16 (VADDUHM $vA, $vA))>;
def : Pat<(v4i32 (shl v4i32:$vA, (v4i32 (immEQOneV)))),
          (v4i32 (VADDUWM $vA, $vA))>;

} // end HasAltivec

// [PO VRT VRA VRB 1 PS XO], "_o" means CR6 is set.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1199-1206

```tablegen
class VX_VT5_VA5_VB5_PS1_XO9_o<bits<9> xo, string opc, list<dag> pattern>
  : VX_RD5_RSp5_PS1_XO9<xo,
                   (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB, u1imm:$PS),
                   !strconcat(opc, " $VD, $VA, $VB, $PS"), IIC_VecFP, pattern> {
  let Defs = [CR6];
}

// [PO VRT VRA VRB 1 / XO]
```
- **EN**: Declares a backend-facing type `VX_VT5_VA5_VB5_PS1_XO9_o` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VX_VT5_VA5_VB5_PS1_XO9_o`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1207-1213

```tablegen
class VX_VT5_VA5_VB5_XO9_o<bits<9> xo, string opc, list<dag> pattern>
  : VX_RD5_RSp5_PS1_XO9<xo, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                        !strconcat(opc, " $VD, $VA, $VB"), IIC_VecFP, pattern> {
  let Defs = [CR6];
  let PS = 0;
}
```
- **EN**: Declares a backend-facing type `VX_VT5_VA5_VB5_XO9_o` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VX_VT5_VA5_VB5_XO9_o`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1214-1220

```tablegen
let Predicates = [HasP8Altivec] in {

let isCommutable = 1 in {
def VMULESW : VX1_Int_Ty2<904, "vmulesw", int_ppc_altivec_vmulesw,
                          v2i64, v4i32>;
def VMULEUW : VX1_Int_Ty2<648, "vmuleuw", int_ppc_altivec_vmuleuw,
                          v2i64, v4i32>;
```
- **EN**: Adds declarative TableGen records such as `VMULESW`, `VMULEUW` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VMULESW`, `VMULEUW`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1221-1227

```tablegen
def VMULOSW : VX1_Int_Ty2<392, "vmulosw", int_ppc_altivec_vmulosw,
                          v2i64, v4i32>;
def VMULOUW : VX1_Int_Ty2<136, "vmulouw", int_ppc_altivec_vmulouw,
                          v2i64, v4i32>;
def VMULUWM : VXForm_1<137, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                       "vmuluwm $VD, $VA, $VB", IIC_VecGeneral,
                       [(set v4i32:$VD, (mul v4i32:$VA, v4i32:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VMULOSW`, `VMULOUW`, `VMULUWM` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VMULOSW`, `VMULOUW`, `VMULUWM`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1228-1234

```tablegen
def VMAXSD : VX1_Int_Ty<450, "vmaxsd", int_ppc_altivec_vmaxsd, v2i64>;
def VMAXUD : VX1_Int_Ty<194, "vmaxud", int_ppc_altivec_vmaxud, v2i64>;
def VMINSD : VX1_Int_Ty<962, "vminsd", int_ppc_altivec_vminsd, v2i64>;
def VMINUD : VX1_Int_Ty<706, "vminud", int_ppc_altivec_vminud, v2i64>;
} // isCommutable

// Vector merge
```
- **EN**: Adds declarative TableGen records such as `VMAXSD`, `VMAXUD`, `VMINSD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VMAXSD`, `VMAXUD`, `VMINSD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1235-1244

```tablegen
def VMRGEW : VXForm_1<1932, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vmrgew $VD, $VA, $VB", IIC_VecFP,
                      [(set v16i8:$VD,
                            (v16i8 (vmrgew_shuffle v16i8:$VA, v16i8:$VB)))]>;
def VMRGOW : VXForm_1<1676, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vmrgow $VD, $VA, $VB", IIC_VecFP,
                      [(set v16i8:$VD,
                            (v16i8 (vmrgow_shuffle v16i8:$VA, v16i8:$VB)))]>;

// Match vmrgew(x,x) and vmrgow(x,x)
```
- **EN**: Adds declarative TableGen records such as `VMRGEW`, `VMRGOW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VMRGEW`, `VMRGOW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1245-1252

```tablegen
def:Pat<(vmrgew_unary_shuffle v16i8:$vA, undef),
        (VMRGEW $vA, $vA)>;
def:Pat<(vmrgow_unary_shuffle v16i8:$vA, undef),
        (VMRGOW $vA, $vA)>;

// Match vmrgew(y,x) and vmrgow(y,x), i.e., swapped operands.  These fragments
// are matched for little-endian, where the inputs must be swapped for correct
// semantics.w
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Match vmrgew(y,x) and vmrgow(y,x), i.e., swapped operands.  These fragments".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Match vmrgew(y,x) and vmrgow(y,x), i.e., swapped operands.  These fragments”。

### Lines 1253-1260

```tablegen
def:Pat<(vmrgew_swapped_shuffle v16i8:$vA, v16i8:$vB),
        (VMRGEW $vB, $vA)>;
def:Pat<(vmrgow_swapped_shuffle v16i8:$vA, v16i8:$vB),
        (VMRGOW $vB, $vA)>;

// Vector rotates.
def VRLD : VX1_Int_Ty<196, "vrld", int_ppc_altivec_vrld, v2i64>;
```
- **EN**: Adds declarative TableGen records such as `VRLD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VRLD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1261-1268

```tablegen
def : Pat<(v2i64 (rotl v2i64:$vA, v2i64:$vB)),
          (v2i64 (VRLD v2i64:$vA, v2i64:$vB))>;

// Vector shifts
def VSLD : VXForm_1<1476, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                    "vsld $VD, $VA, $VB", IIC_VecGeneral, []>;
def VSRD : VXForm_1<1732, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                   "vsrd $VD, $VA, $VB", IIC_VecGeneral, []>;
```
- **EN**: Adds declarative TableGen records such as `VSLD`, `VSRD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSLD`, `VSRD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1269-1275

```tablegen
def VSRAD : VXForm_1<964, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                    "vsrad $VD, $VA, $VB", IIC_VecGeneral, []>;

def : Pat<(v2i64 (shl v2i64:$vA, v2i64:$vB)),
          (v2i64 (VSLD $vA, $vB))>;
def : Pat<(v2i64 (PPCshl v2i64:$vA, v2i64:$vB)),
          (v2i64 (VSLD $vA, $vB))>;
```
- **EN**: Adds declarative TableGen records such as `VSRAD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSRAD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1276-1285

```tablegen
def : Pat<(v2i64 (srl v2i64:$vA, v2i64:$vB)),
          (v2i64 (VSRD $vA, $vB))>;
def : Pat<(v2i64 (PPCsrl v2i64:$vA, v2i64:$vB)),
          (v2i64 (VSRD $vA, $vB))>;
def : Pat<(v2i64 (sra v2i64:$vA, v2i64:$vB)),
          (v2i64 (VSRAD $vA, $vB))>;
def : Pat<(v2i64 (PPCsra v2i64:$vA, v2i64:$vB)),
          (v2i64 (VSRAD $vA, $vB))>;

// Vector Integer Arithmetic Instructions
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1286-1295

```tablegen
let isCommutable = 1 in {
def VADDUDM : VXForm_1<192, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                       "vaddudm $VD, $VA, $VB", IIC_VecGeneral,
                       [(set v2i64:$VD, (add v2i64:$VA, v2i64:$VB))]>;
def VADDUQM : VXForm_1<256, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                       "vadduqm $VD, $VA, $VB", IIC_VecGeneral,
                       [(set v1i128:$VD, (add v1i128:$VA, v1i128:$VB))]>;
} // isCommutable

// Vector Quadword Add
```
- **EN**: Adds declarative TableGen records such as `VADDUDM`, `VADDUQM` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VADDUDM`, `VADDUQM`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1296-1305

```tablegen
def VADDEUQM : VA1a_Int_Ty<60, "vaddeuqm", int_ppc_altivec_vaddeuqm, v1i128>;
def VADDCUQ  : VX1_Int_Ty<320, "vaddcuq", int_ppc_altivec_vaddcuq, v1i128>;
def VADDECUQ : VA1a_Int_Ty<61, "vaddecuq", int_ppc_altivec_vaddecuq, v1i128>;

// Vector Doubleword Subtract
def VSUBUDM : VXForm_1<1216, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                       "vsubudm $VD, $VA, $VB", IIC_VecGeneral,
                       [(set v2i64:$VD, (sub v2i64:$VA, v2i64:$VB))]>;

// Vector Quadword Subtract
```
- **EN**: Adds declarative TableGen records such as `VADDEUQM`, `VADDCUQ`, `VADDECUQ` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VADDEUQM`, `VADDCUQ`, `VADDECUQ`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1306-1313

```tablegen
def VSUBUQM : VXForm_1<1280, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                       "vsubuqm $VD, $VA, $VB", IIC_VecGeneral,
                       [(set v1i128:$VD, (sub v1i128:$VA, v1i128:$VB))]>;
def VSUBEUQM : VA1a_Int_Ty<62, "vsubeuqm", int_ppc_altivec_vsubeuqm, v1i128>;
def VSUBCUQ  : VX1_Int_Ty<1344, "vsubcuq", int_ppc_altivec_vsubcuq, v1i128>;
def VSUBECUQ : VA1a_Int_Ty<63, "vsubecuq", int_ppc_altivec_vsubecuq, v1i128>;

// Count Leading Zeros
```
- **EN**: Adds declarative TableGen records such as `VSUBUQM`, `VSUBEUQM`, `VSUBCUQ` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSUBUQM`, `VSUBEUQM`, `VSUBCUQ`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1314-1322

```tablegen
def VCLZB : VXForm_2<1794, (outs vrrc:$VD), (ins vrrc:$VB),
                     "vclzb $VD, $VB", IIC_VecGeneral,
                     [(set v16i8:$VD, (ctlz v16i8:$VB))]>;
def VCLZH : VXForm_2<1858, (outs vrrc:$VD), (ins vrrc:$VB),
                     "vclzh $VD, $VB", IIC_VecGeneral,
                     [(set v8i16:$VD, (ctlz v8i16:$VB))]>;
def VCLZW : VXForm_2<1922, (outs vrrc:$VD), (ins vrrc:$VB),
                     "vclzw $VD, $VB", IIC_VecGeneral,
                     [(set v4i32:$VD, (ctlz v4i32:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VCLZB`, `VCLZH`, `VCLZW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCLZB`, `VCLZH`, `VCLZW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1323-1330

```tablegen
def VCLZD : VXForm_2<1986, (outs vrrc:$VD), (ins vrrc:$VB),
                     "vclzd $VD, $VB", IIC_VecGeneral,
                     [(set v2i64:$VD, (ctlz v2i64:$VB))]>;

// Population Count
def VPOPCNTB : VXForm_2<1795, (outs vrrc:$VD), (ins vrrc:$VB),
                        "vpopcntb $VD, $VB", IIC_VecGeneral,
                        [(set v16i8:$VD, (ctpop v16i8:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VCLZD`, `VPOPCNTB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCLZD`, `VPOPCNTB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1331-1340

```tablegen
def VPOPCNTH : VXForm_2<1859, (outs vrrc:$VD), (ins vrrc:$VB),
                        "vpopcnth $VD, $VB", IIC_VecGeneral,
                        [(set v8i16:$VD, (ctpop v8i16:$VB))]>;
def VPOPCNTW : VXForm_2<1923, (outs vrrc:$VD), (ins vrrc:$VB),
                        "vpopcntw $VD, $VB", IIC_VecGeneral,
                        [(set v4i32:$VD, (ctpop v4i32:$VB))]>;
def VPOPCNTD : VXForm_2<1987, (outs vrrc:$VD), (ins vrrc:$VB),
                        "vpopcntd $VD, $VB", IIC_VecGeneral,
                        [(set v2i64:$VD, (ctpop v2i64:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VPOPCNTH`, `VPOPCNTW`, `VPOPCNTD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VPOPCNTH`, `VPOPCNTW`, `VPOPCNTD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1341-1351

```tablegen
let isCommutable = 1 in {
// FIXME: Use AddedComplexity > 400 to ensure these patterns match before the
//        VSX equivalents. We need to fix this up at some point. Two possible
//        solutions for this problem:
//        1. Disable Altivec patterns that compete with VSX patterns using the
//           !HasVSX predicate. This essentially favours VSX over Altivec, in
//           hopes of reducing register pressure (larger register set using VSX
//           instructions than VMX instructions)
//        2. Employ a more disciplined use of AddedComplexity, which would provide
//           more fine-grained control than option 1. This would be beneficial
//           if we find situations where Altivec is really preferred over VSX.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "FIXME: Use AddedComplexity > 400 to ensure these patterns match before the".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“FIXME: Use AddedComplexity > 400 to ensure these patterns match before the”。

### Lines 1352-1359

```tablegen
def VEQV  : VXForm_1<1668, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                     "veqv $VD, $VA, $VB", IIC_VecGeneral,
                     [(set v4i32:$VD, (vnot (xor v4i32:$VA, v4i32:$VB)))]>;
def VNAND : VXForm_1<1412, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                     "vnand $VD, $VA, $VB", IIC_VecGeneral,
                     [(set v4i32:$VD, (vnot (and v4i32:$VA, v4i32:$VB)))]>;
} // isCommutable
```
- **EN**: Adds declarative TableGen records such as `VEQV`, `VNAND` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VEQV`, `VNAND`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1360-1366

```tablegen
def VORC : VXForm_1<1348, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                      "vorc $VD, $VA, $VB", IIC_VecGeneral,
                      [(set v4i32:$VD, (or v4i32:$VA,
                                           (vnot v4i32:$VB)))]>;

// i64 element comparisons.
def VCMPEQUD  : VCMP <199, "vcmpequd $VD, $VA, $VB" , v2i64>;
```
- **EN**: Adds declarative TableGen records such as `VORC`, `VCMPEQUD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VORC`, `VCMPEQUD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1367-1373

```tablegen
def VCMPEQUD_rec : VCMP_rec<199, "vcmpequd. $VD, $VA, $VB", v2i64>;
def VCMPGTSD  : VCMP <967, "vcmpgtsd $VD, $VA, $VB" , v2i64>;
def VCMPGTSD_rec : VCMP_rec<967, "vcmpgtsd. $VD, $VA, $VB", v2i64>;
def VCMPGTUD  : VCMP <711, "vcmpgtud $VD, $VA, $VB" , v2i64>;
def VCMPGTUD_rec : VCMP_rec<711, "vcmpgtud. $VD, $VA, $VB", v2i64>;

// The cryptography instructions that do not require Category:Vector.Crypto
```
- **EN**: Adds declarative TableGen records such as `VCMPEQUD_rec`, `VCMPGTSD`, `VCMPGTSD_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCMPEQUD_rec`, `VCMPGTSD`, `VCMPGTSD_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1374-1381

```tablegen
def VPMSUMB : VX1_Int_Ty<1032, "vpmsumb",
                         int_ppc_altivec_crypto_vpmsumb, v16i8>;
def VPMSUMH : VX1_Int_Ty<1096, "vpmsumh",
                         int_ppc_altivec_crypto_vpmsumh, v8i16>;
def VPMSUMW : VX1_Int_Ty<1160, "vpmsumw",
                         int_ppc_altivec_crypto_vpmsumw, v4i32>;
def VPMSUMD : VX1_Int_Ty<1224, "vpmsumd",
                         int_ppc_altivec_crypto_vpmsumd, v2i64>;
```
- **EN**: Adds declarative TableGen records such as `VPMSUMB`, `VPMSUMH`, `VPMSUMW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VPMSUMB`, `VPMSUMH`, `VPMSUMW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1382-1388

```tablegen
def VPERMXOR : VAForm_1<45, (outs vrrc:$RT), (ins vrrc:$RA, vrrc:$RB, vrrc:$RC),
                        "vpermxor $RT, $RA, $RB, $RC", IIC_VecFP, []>;

// Vector doubleword integer pack and unpack.
let hasSideEffects = 1 in {
  def VPKSDSS : VX1_Int_Ty2<1486, "vpksdss", int_ppc_altivec_vpksdss,
                            v4i32, v2i64>;
```
- **EN**: Adds declarative TableGen records such as `VPERMXOR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VPERMXOR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1389-1397

```tablegen
  def VPKSDUS : VX1_Int_Ty2<1358, "vpksdus", int_ppc_altivec_vpksdus,
                            v4i32, v2i64>;
  def VPKUDUS : VX1_Int_Ty2<1230, "vpkudus", int_ppc_altivec_vpkudus,
                            v4i32, v2i64>;
}
def VPKUDUM : VXForm_1<1102, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                       "vpkudum $VD, $VA, $VB", IIC_VecFP,
                       [(set v16i8:$VD,
                         (vpkudum_shuffle v16i8:$VA, v16i8:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VPKUDUM` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VPKUDUM`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1398-1404

```tablegen
def VUPKHSW : VX2_Int_Ty2<1614, "vupkhsw", int_ppc_altivec_vupkhsw,
                          v2i64, v4i32>;
def VUPKLSW : VX2_Int_Ty2<1742, "vupklsw", int_ppc_altivec_vupklsw,
                          v2i64, v4i32>;
def BCDADD_rec : VX_VT5_VA5_VB5_PS1_XO9_o<1,  "bcdadd." , []>;
def BCDSUB_rec : VX_VT5_VA5_VB5_PS1_XO9_o<65, "bcdsub." , []>;
```
- **EN**: Adds declarative TableGen records such as `VUPKHSW`, `VUPKLSW`, `BCDADD_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VUPKHSW`, `VUPKLSW`, `BCDADD_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1405-1412

```tablegen
def : Pat<(v16i8 (int_ppc_bcdadd v16i8:$vA, v16i8:$vB, u1imm_timm:$PS)),
          (BCDADD_rec $vA, $vB, $PS)>;
def : Pat<(v16i8 (int_ppc_bcdsub v16i8:$vA, v16i8:$vB, u1imm_timm:$PS)),
          (BCDSUB_rec $vA, $vB, $PS)>;

// Shuffle patterns for unary and swapped (LE) vector pack modulo.
def:Pat<(vpkudum_unary_shuffle v16i8:$vA, undef),
        (VPKUDUM $vA, $vA)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1413-1421

```tablegen
def:Pat<(vpkudum_swapped_shuffle v16i8:$vA, v16i8:$vB),
        (VPKUDUM $vB, $vA)>;

def VGBBD : VX2_Int_Ty2<1292, "vgbbd", int_ppc_altivec_vgbbd, v16i8, v16i8>;
def VBPERMQ : VX1_Int_Ty2<1356, "vbpermq", int_ppc_altivec_vbpermq,
                          v2i64, v16i8>;
} // end HasP8Altivec

// Crypto instructions (from builtins)
```
- **EN**: Adds declarative TableGen records such as `VGBBD`, `VBPERMQ` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VGBBD`, `VBPERMQ`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1422-1428

```tablegen
let Predicates = [HasP8Crypto] in {
def VSHASIGMAW : VXCR_Int_Ty<1666, "vshasigmaw",
                              int_ppc_altivec_crypto_vshasigmaw, v4i32>;
def VSHASIGMAD : VXCR_Int_Ty<1730, "vshasigmad",
                              int_ppc_altivec_crypto_vshasigmad, v2i64>;
def VCIPHER : VX1_Int_Ty<1288, "vcipher", int_ppc_altivec_crypto_vcipher,
                         v2i64>;
```
- **EN**: Adds declarative TableGen records such as `VSHASIGMAW`, `VSHASIGMAD`, `VCIPHER` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSHASIGMAW`, `VSHASIGMAD`, `VCIPHER`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1429-1438

```tablegen
def VCIPHERLAST : VX1_Int_Ty<1289, "vcipherlast",
                              int_ppc_altivec_crypto_vcipherlast, v2i64>;
def VNCIPHER : VX1_Int_Ty<1352, "vncipher",
                          int_ppc_altivec_crypto_vncipher, v2i64>;
def VNCIPHERLAST : VX1_Int_Ty<1353, "vncipherlast",
                              int_ppc_altivec_crypto_vncipherlast, v2i64>;
def VSBOX : VXBX_Int_Ty<1480, "vsbox", int_ppc_altivec_crypto_vsbox, v2i64>;
} // HasP8Crypto

// The following altivec instructions were introduced in Power ISA 3.0
```
- **EN**: Adds declarative TableGen records such as `VCIPHERLAST`, `VNCIPHER`, `VNCIPHERLAST` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCIPHERLAST`, `VNCIPHER`, `VNCIPHERLAST`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1439-1445

```tablegen
let Predicates = [HasP9Altivec] in {

// Vector Multiply-Sum
def VMSUMUDM : VA1a_Int_Ty3<35, "vmsumudm", int_ppc_altivec_vmsumudm,
                            v1i128, v2i64, v1i128>;

// i8 element comparisons.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Vector Multiply-Sum". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Vector Multiply-Sum”。 这些声明会进入生成式模式匹配逻辑。

### Lines 1446-1452

```tablegen
def VCMPNEB   : VCMP   <  7, "vcmpneb $VD, $VA, $VB"  , v16i8>;
def VCMPNEB_rec  : VCMP_rec  <  7, "vcmpneb. $VD, $VA, $VB" , v16i8>;
def VCMPNEZB  : VCMP <263, "vcmpnezb $VD, $VA, $VB" , v16i8>;
def VCMPNEZB_rec : VCMP_rec<263, "vcmpnezb. $VD, $VA, $VB", v16i8>;

// i16 element comparisons.
def VCMPNEH   : VCMP < 71, "vcmpneh $VD, $VA, $VB"  , v8i16>;
```
- **EN**: Adds declarative TableGen records such as `VCMPNEB`, `VCMPNEB_rec`, `VCMPNEZB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCMPNEB`, `VCMPNEB_rec`, `VCMPNEZB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1453-1459

```tablegen
def VCMPNEH_rec  : VCMP_rec< 71, "vcmpneh. $VD, $VA, $VB" , v8i16>;
def VCMPNEZH  : VCMP <327, "vcmpnezh $VD, $VA, $VB" , v8i16>;
def VCMPNEZH_rec : VCMP_rec<327, "vcmpnezh. $VD, $VA, $VB", v8i16>;

// i32 element comparisons.
def VCMPNEW   : VCMP <135, "vcmpnew $VD, $VA, $VB"  , v4i32>;
def VCMPNEW_rec  : VCMP_rec<135, "vcmpnew. $VD, $VA, $VB" , v4i32>;
```
- **EN**: Adds declarative TableGen records such as `VCMPNEH_rec`, `VCMPNEZH`, `VCMPNEZH_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCMPNEH_rec`, `VCMPNEZH`, `VCMPNEZH_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1460-1469

```tablegen
def VCMPNEZW  : VCMP <391, "vcmpnezw $VD, $VA, $VB" , v4i32>;
def VCMPNEZW_rec : VCMP_rec<391, "vcmpnezw. $VD, $VA, $VB", v4i32>;

// VX-Form: [PO VRT / UIM VRB XO].
// We use VXForm_1 to implement it, that is, we use "VRA" (5 bit) to represent
// "/ UIM" (1 + 4 bit)
class VX1_VT5_UIM5_VB5<bits<11> xo, string opc, list<dag> pattern>
  : VXForm_1<xo, (outs vrrc:$VD), (ins u4imm:$VA, vrrc:$VB),
             !strconcat(opc, " $VD, $VB, $VA"), IIC_VecGeneral, pattern>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "VX-Form: [PO VRT / UIM VRB XO].". Notable symbols in this range include `VCMPNEZW`, `VCMPNEZW_rec`, `VX1_VT5_UIM5_VB5`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“VX-Form: [PO VRT / UIM VRB XO].”。 该区间中较显眼的符号包括 `VCMPNEZW`, `VCMPNEZW_rec`, `VX1_VT5_UIM5_VB5`。

### Lines 1470-1476

```tablegen
class VX1_RT5_RA5_VB5<bits<11> xo, string opc, list<dag> pattern>
  : VXForm_1<xo, (outs g8rc:$VD), (ins g8rc:$VA, vrrc:$VB),
             !strconcat(opc, " $VD, $VA, $VB"), IIC_VecGeneral, pattern>;

// Vector Extract Unsigned
def VEXTRACTUB : VX1_VT5_UIM5_VB5<525, "vextractub", []>;
def VEXTRACTUH : VX1_VT5_UIM5_VB5<589, "vextractuh", []>;
```
- **EN**: Declares a backend-facing type `VX1_RT5_RA5_VB5`, `VEXTRACTUB`, `VEXTRACTUH` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VX1_RT5_RA5_VB5`, `VEXTRACTUB`, `VEXTRACTUH`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1477-1483

```tablegen
def VEXTRACTUW : VX1_VT5_UIM5_VB5<653, "vextractuw", []>;
def VEXTRACTD  : VX1_VT5_UIM5_VB5<717, "vextractd" , []>;

// Vector Extract Unsigned Byte/Halfword/Word Left/Right-Indexed
let hasSideEffects = 0 in {
def VEXTUBLX : VX1_RT5_RA5_VB5<1549, "vextublx", []>, ZExt32To64;
def VEXTUBRX : VX1_RT5_RA5_VB5<1805, "vextubrx", []>, ZExt32To64;
```
- **EN**: Adds declarative TableGen records such as `VEXTRACTUW`, `VEXTRACTD`, `VEXTUBLX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VEXTRACTUW`, `VEXTRACTD`, `VEXTUBLX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1484-1490

```tablegen
def VEXTUHLX : VX1_RT5_RA5_VB5<1613, "vextuhlx", []>, ZExt32To64;
def VEXTUHRX : VX1_RT5_RA5_VB5<1869, "vextuhrx", []>, ZExt32To64;
def VEXTUWLX : VX1_RT5_RA5_VB5<1677, "vextuwlx", []>, ZExt32To64;
def VEXTUWRX : VX1_RT5_RA5_VB5<1933, "vextuwrx", []>, ZExt32To64;
}

// Vector Insert Element Instructions
```
- **EN**: Adds declarative TableGen records such as `VEXTUHLX`, `VEXTUHRX`, `VEXTUWLX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VEXTUHLX`, `VEXTUHRX`, `VEXTUWLX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1491-1502

```tablegen
def VINSERTB : VXForm_1<781, (outs vrrc:$VD),
                        (ins vrrc:$VDi, u4imm:$VA, vrrc:$VB),
                        "vinsertb $VD, $VB, $VA", IIC_VecGeneral,
                        [(set v16i8:$VD, (PPCvecinsert v16i8:$VDi, v16i8:$VB,
                                                      imm32SExt16:$VA))]>,
                        RegConstraint<"$VDi = $VD">;
def VINSERTH : VXForm_1<845, (outs vrrc:$VD),
                        (ins vrrc:$VDi, u4imm:$VA, vrrc:$VB),
                        "vinserth $VD, $VB, $VA", IIC_VecGeneral,
                        [(set v8i16:$VD, (PPCvecinsert v8i16:$VDi, v8i16:$VB,
                                                      imm32SExt16:$VA))]>,
                        RegConstraint<"$VDi = $VD">;
```
- **EN**: Adds declarative TableGen records such as `VINSERTB`, `VINSERTH` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VINSERTB`, `VINSERTH`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1503-1513

```tablegen
def VINSERTW : VX1_VT5_UIM5_VB5<909, "vinsertw", []>;
def VINSERTD : VX1_VT5_UIM5_VB5<973, "vinsertd", []>;

class VX_VT5_EO5_VB5<bits<11> xo, bits<5> eo, string opc, list<dag> pattern>
  : VXForm_RD5_XO5_RS5<xo, eo, (outs vrrc:$VD), (ins vrrc:$VB),
                       !strconcat(opc, " $VD, $VB"), IIC_VecGeneral, pattern>;
class VX_VT5_EO5_VB5s<bits<11> xo, bits<5> eo, string opc, list<dag> pattern>
  : VXForm_RD5_XO5_RS5<xo, eo, (outs vfrc:$VD), (ins vfrc:$VB),
                       !strconcat(opc, " $VD, $VB"), IIC_VecGeneral, pattern>;

// Vector Count Leading/Trailing Zero LSB. Result is placed into GPR[RD]
```
- **EN**: Declares a backend-facing type `VINSERTW`, `VINSERTD`, `VX_VT5_EO5_VB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VINSERTW`, `VINSERTD`, `VX_VT5_EO5_VB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1514-1522

```tablegen
def VCLZLSBB : VXForm_RD5_XO5_RS5<1538, 0, (outs gprc:$VD), (ins vrrc:$VB),
                                  "vclzlsbb $VD, $VB", IIC_VecGeneral,
                                  [(set i32:$VD, (int_ppc_altivec_vclzlsbb
                                     v16i8:$VB))]>;
def VCTZLSBB : VXForm_RD5_XO5_RS5<1538, 1, (outs gprc:$VD), (ins vrrc:$VB),
                                  "vctzlsbb $VD, $VB", IIC_VecGeneral,
                                  [(set i32:$VD, (int_ppc_altivec_vctzlsbb
                                     v16i8:$VB))]>;
// Vector Count Trailing Zeros
```
- **EN**: Adds declarative TableGen records such as `VCLZLSBB`, `VCTZLSBB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCLZLSBB`, `VCTZLSBB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1523-1532

```tablegen
def VCTZB : VX_VT5_EO5_VB5<1538, 28, "vctzb",
                           [(set v16i8:$VD, (cttz v16i8:$VB))]>;
def VCTZH : VX_VT5_EO5_VB5<1538, 29, "vctzh",
                           [(set v8i16:$VD, (cttz v8i16:$VB))]>;
def VCTZW : VX_VT5_EO5_VB5<1538, 30, "vctzw",
                           [(set v4i32:$VD, (cttz v4i32:$VB))]>;
def VCTZD : VX_VT5_EO5_VB5<1538, 31, "vctzd",
                           [(set v2i64:$VD, (cttz v2i64:$VB))]>;

// Vector Extend Sign
```
- **EN**: Adds declarative TableGen records such as `VCTZB`, `VCTZH`, `VCTZW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VCTZB`, `VCTZH`, `VCTZW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1533-1540

```tablegen
def VEXTSB2W : VX_VT5_EO5_VB5<1538, 16, "vextsb2w",
                              [(set v4i32:$VD, (int_ppc_altivec_vextsb2w v16i8:$VB))]>;
def VEXTSH2W : VX_VT5_EO5_VB5<1538, 17, "vextsh2w",
                              [(set v4i32:$VD, (int_ppc_altivec_vextsh2w v8i16:$VB))]>;
def VEXTSB2D : VX_VT5_EO5_VB5<1538, 24, "vextsb2d",
                              [(set v2i64:$VD, (int_ppc_altivec_vextsb2d v16i8:$VB))]>;
def VEXTSH2D : VX_VT5_EO5_VB5<1538, 25, "vextsh2d",
                              [(set v2i64:$VD, (int_ppc_altivec_vextsh2d v8i16:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VEXTSB2W`, `VEXTSH2W`, `VEXTSB2D` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VEXTSB2W`, `VEXTSH2W`, `VEXTSB2D`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1541-1547

```tablegen
def VEXTSW2D : VX_VT5_EO5_VB5<1538, 26, "vextsw2d",
                              [(set v2i64:$VD, (int_ppc_altivec_vextsw2d v4i32:$VB))]>;
let isCodeGenOnly = 1 in {
  def VEXTSB2Ws : VX_VT5_EO5_VB5s<1538, 16, "vextsb2w", []>;
  def VEXTSH2Ws : VX_VT5_EO5_VB5s<1538, 17, "vextsh2w", []>;
  def VEXTSB2Ds : VX_VT5_EO5_VB5s<1538, 24, "vextsb2d", []>;
  def VEXTSH2Ds : VX_VT5_EO5_VB5s<1538, 25, "vextsh2d", []>;
```
- **EN**: Adds declarative TableGen records such as `VEXTSW2D` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VEXTSW2D`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1548-1554

```tablegen
  def VEXTSW2Ds : VX_VT5_EO5_VB5s<1538, 26, "vextsw2d", []>;
}

def : Pat<(v4i32 (sext_inreg v4i32:$VRB, v4i8)), (v4i32 (VEXTSB2W $VRB))>;
def : Pat<(v4i32 (sext_inreg v4i32:$VRB, v4i16)), (v4i32 (VEXTSH2W $VRB))>;
def : Pat<(v2i64 (sext_inreg v2i64:$VRB, v2i8)), (v2i64 (VEXTSB2D $VRB))>;
def : Pat<(v2i64 (sext_inreg v2i64:$VRB, v2i16)), (v2i64 (VEXTSH2D $VRB))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1555-1561

```tablegen
def : Pat<(v2i64 (sext_inreg v2i64:$VRB, v2i32)), (v2i64 (VEXTSW2D $VRB))>;

// Vector Integer Negate
def VNEGW : VX_VT5_EO5_VB5<1538, 6, "vnegw",
                           [(set v4i32:$VD,
                            (sub (v4i32 immAllZerosV), v4i32:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VNEGW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VNEGW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1562-1568

```tablegen
def VNEGD : VX_VT5_EO5_VB5<1538, 7, "vnegd",
                           [(set v2i64:$VD,
                            (sub (v2i64 immAllZerosV), v2i64:$VB))]>;

// Vector Parity Byte
def VPRTYBW : VX_VT5_EO5_VB5<1538, 8, "vprtybw", [(set v4i32:$VD,
                            (int_ppc_altivec_vprtybw v4i32:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VNEGD`, `VPRTYBW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VNEGD`, `VPRTYBW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1569-1576

```tablegen
def VPRTYBD : VX_VT5_EO5_VB5<1538,  9, "vprtybd", [(set v2i64:$VD,
                            (int_ppc_altivec_vprtybd v2i64:$VB))]>;
def VPRTYBQ : VX_VT5_EO5_VB5<1538, 10, "vprtybq", [(set v1i128:$VD,
                            (int_ppc_altivec_vprtybq v1i128:$VB))]>;

// Vector (Bit) Permute (Right-indexed)
def VBPERMD : VX1_Int_Ty3<1484, "vbpermd", int_ppc_altivec_vbpermd,
                          v2i64, v2i64, v16i8>;
```
- **EN**: Adds declarative TableGen records such as `VPRTYBD`, `VPRTYBQ`, `VBPERMD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VPRTYBD`, `VPRTYBQ`, `VBPERMD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1577-1584

```tablegen
def VPERMR : VAForm_1a<59, (outs vrrc:$RT), (ins vrrc:$RA, vrrc:$RB, vrrc:$RC),
                       "vpermr $RT, $RA, $RB, $RC", IIC_VecFP, []>;

class VX1_VT5_VA5_VB5<bits<11> xo, string opc, list<dag> pattern>
  : VXForm_1<xo, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
             !strconcat(opc, " $VD, $VA, $VB"), IIC_VecFP, pattern>;

// Vector Rotate Left Mask/Mask-Insert
```
- **EN**: Declares a backend-facing type `VPERMR`, `VX1_VT5_VA5_VB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VPERMR`, `VX1_VT5_VA5_VB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1585-1594

```tablegen
def VRLWNM : VX1_VT5_VA5_VB5<389, "vrlwnm",
                             [(set v4i32:$VD,
                                 (int_ppc_altivec_vrlwnm v4i32:$VA,
                                                         v4i32:$VB))]>;
def VRLWMI : VXForm_1<133, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB, vrrc:$VDi),
                      "vrlwmi $VD, $VA, $VB", IIC_VecFP,
                      [(set v4i32:$VD,
                         (int_ppc_altivec_vrlwmi v4i32:$VA, v4i32:$VB,
                                                 v4i32:$VDi))]>,
                      RegConstraint<"$VDi = $VD">;
```
- **EN**: Adds declarative TableGen records such as `VRLWNM`, `VRLWMI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VRLWNM`, `VRLWMI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1595-1606

```tablegen
def VRLDNM : VX1_VT5_VA5_VB5<453, "vrldnm",
                             [(set v2i64:$VD,
                                 (int_ppc_altivec_vrldnm v2i64:$VA,
                                                         v2i64:$VB))]>;
def VRLDMI : VXForm_1<197, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB, vrrc:$VDi),
                      "vrldmi $VD, $VA, $VB", IIC_VecFP,
                      [(set v2i64:$VD,
                         (int_ppc_altivec_vrldmi v2i64:$VA, v2i64:$VB,
                                                 v2i64:$VDi))]>,
                      RegConstraint<"$VDi = $VD">;

// Vector Shift Left/Right
```
- **EN**: Adds declarative TableGen records such as `VRLDNM`, `VRLDMI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VRLDNM`, `VRLDMI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1607-1614

```tablegen
def VSLV : VX1_VT5_VA5_VB5<1860, "vslv",
                           [(set v16i8 : $VD, (int_ppc_altivec_vslv v16i8 : $VA, v16i8 : $VB))]>;
def VSRV : VX1_VT5_VA5_VB5<1796, "vsrv",
                           [(set v16i8 : $VD, (int_ppc_altivec_vsrv v16i8 : $VA, v16i8 : $VB))]>;

// Vector Multiply-by-10 (& Write Carry) Unsigned Quadword
def VMUL10UQ   : VXForm_BX<513, (outs vrrc:$VD), (ins vrrc:$VA),
                           "vmul10uq $VD, $VA", IIC_VecFP, []>;
```
- **EN**: Adds declarative TableGen records such as `VSLV`, `VSRV`, `VMUL10UQ` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VSLV`, `VSRV`, `VMUL10UQ`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1615-1624

```tablegen
def VMUL10CUQ  : VXForm_BX<  1, (outs vrrc:$VD), (ins vrrc:$VA),
                           "vmul10cuq $VD, $VA", IIC_VecFP, []>;

// Vector Multiply-by-10 Extended (& Write Carry) Unsigned Quadword
def VMUL10EUQ  : VX1_VT5_VA5_VB5<577, "vmul10euq" , []>;
def VMUL10ECUQ : VX1_VT5_VA5_VB5< 65, "vmul10ecuq", []>;

// Decimal Integer Format Conversion Instructions

// [PO VRT EO VRB 1 PS XO], "_o" means CR6 is set.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Vector Multiply-by-10 Extended (& Write Carry) Unsigned Quadword". Notable symbols in this range include `VMUL10CUQ`, `VMUL10EUQ`, `VMUL10ECUQ`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Vector Multiply-by-10 Extended (& Write Carry) Unsigned Quadword”。 该区间中较显眼的符号包括 `VMUL10CUQ`, `VMUL10EUQ`, `VMUL10ECUQ`。

### Lines 1625-1632

```tablegen
class VX_VT5_EO5_VB5_PS1_XO9_o<bits<5> eo, bits<9> xo, string opc,
                               list<dag> pattern>
  : VX_RD5_EO5_RS5_PS1_XO9<eo, xo, (outs vrrc:$VD), (ins vrrc:$VB, u1imm:$PS),
                        !strconcat(opc, " $VD, $VB, $PS"), IIC_VecFP, pattern> {
  let Defs = [CR6];
}

// [PO VRT EO VRB 1 / XO]
```
- **EN**: Declares a backend-facing type `VX_VT5_EO5_VB5_PS1_XO9_o` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VX_VT5_EO5_VB5_PS1_XO9_o`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1633-1641

```tablegen
class VX_VT5_EO5_VB5_XO9_o<bits<5> eo, bits<9> xo, string opc,
                           list<dag> pattern>
  : VX_RD5_EO5_RS5_PS1_XO9<eo, xo, (outs vrrc:$VD), (ins vrrc:$VB),
                           !strconcat(opc, " $VD, $VB"), IIC_VecFP, pattern> {
  let Defs = [CR6];
  let PS = 0;
}

// Decimal Convert From/to National/Zoned/Signed-QWord
```
- **EN**: Declares a backend-facing type `VX_VT5_EO5_VB5_XO9_o` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VX_VT5_EO5_VB5_XO9_o`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1642-1649

```tablegen
def BCDCFN_rec  : VX_VT5_EO5_VB5_PS1_XO9_o<7, 385, "bcdcfn." ,
                  [(set v16i8:$VD, (int_ppc_national2packed v16i8:$VB, u1imm_timm:$PS))]>;
def BCDCFZ_rec  : VX_VT5_EO5_VB5_PS1_XO9_o<6, 385, "bcdcfz." , 
                  [(set v16i8:$VD, (int_ppc_zoned2packed v16i8:$VB, u1imm_timm:$PS))]>;
def BCDCTN_rec  : VX_VT5_EO5_VB5_XO9_o    <5, 385, "bcdctn." , 
                  [(set v16i8:$VD, (int_ppc_packed2national v16i8:$VB))]>;
def BCDCTZ_rec  : VX_VT5_EO5_VB5_PS1_XO9_o<4, 385, "bcdctz." , 
                  [(set v16i8:$VD, (int_ppc_packed2zoned v16i8:$VB, u1imm_timm:$PS))]>;
```
- **EN**: Adds declarative TableGen records such as `BCDCFN_rec`, `BCDCFZ_rec`, `BCDCTN_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BCDCFN_rec`, `BCDCFZ_rec`, `BCDCTN_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1650-1657

```tablegen
def BCDCFSQ_rec : VX_VT5_EO5_VB5_PS1_XO9_o<2, 385, "bcdcfsq.", []>;
def BCDCTSQ_rec : VX_VT5_EO5_VB5_XO9_o    <0, 385, "bcdctsq.", []>;

// Decimal Copy-Sign/Set-Sign
let Defs = [CR6] in
def BCDCPSGN_rec : VX1_VT5_VA5_VB5<833, "bcdcpsgn.",
    [(set v16i8:$VD, (int_ppc_bcdcopysign v16i8:$VA, v16i8:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `BCDCFSQ_rec`, `BCDCTSQ_rec`, `BCDCPSGN_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BCDCFSQ_rec`, `BCDCTSQ_rec`, `BCDCPSGN_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1658-1666

```tablegen
def BCDSETSGN_rec : VX_VT5_EO5_VB5_PS1_XO9_o<31, 385, "bcdsetsgn.",
    [(set v16i8:$VD, (int_ppc_bcdsetsign v16i8:$VB, i32:$PS))]>;

// Decimal Shift/Unsigned-Shift/Shift-and-Round
def BCDS_rec :  VX_VT5_VA5_VB5_PS1_XO9_o<193, "bcds." , []>;
def BCDUS_rec : VX_VT5_VA5_VB5_XO9_o    <129, "bcdus.", []>;
def BCDSR_rec : VX_VT5_VA5_VB5_PS1_XO9_o<449, "bcdsr.", []>;

// Decimal (Unsigned) Truncate
```
- **EN**: Adds declarative TableGen records such as `BCDSETSGN_rec`, `BCDS_rec`, `BCDUS_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BCDSETSGN_rec`, `BCDS_rec`, `BCDUS_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1667-1673

```tablegen
def BCDTRUNC_rec :  VX_VT5_VA5_VB5_PS1_XO9_o<257, "bcdtrunc." , []>;
def BCDUTRUNC_rec : VX_VT5_VA5_VB5_XO9_o    <321, "bcdutrunc.", []>;

// Patterns mapping BCD PPCISD nodes to machine instructions
def : Pat<(PPCbcds v4i32:$Shift, v16i8:$Src, i32:$PS),
          (BCDS_rec $Shift, $Src, $PS)>;
```
- **EN**: Adds declarative TableGen records such as `BCDTRUNC_rec`, `BCDUTRUNC_rec` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BCDTRUNC_rec`, `BCDUTRUNC_rec`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1674-1682

```tablegen
def : Pat<(PPCbcdsr v4i32:$ShiftRound, v16i8:$Src, i32:$PS),
          (BCDSR_rec $ShiftRound, $Src, $PS)>;

def : Pat<(PPCbcdtrunc v4i32:$Trunc, v16i8:$Src, i32:$PS),
          (BCDTRUNC_rec $Trunc, $Src, $PS)>;

def : Pat<(PPCbcdutrunc v4i32:$Trunc, v16i8:$Src),
          (BCDUTRUNC_rec $Trunc, $Src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1683-1689

```tablegen
def : Pat<(PPCbcdus v4i32:$Shift, v16i8:$Src),
          (BCDUS_rec $Shift, $Src)>;

// Absolute Difference
def VABSDUB : VXForm_1<1027, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                       "vabsdub $VD, $VA, $VB", IIC_VecGeneral,
                       [(set v16i8:$VD, (int_ppc_altivec_vabsdub v16i8:$VA, v16i8:$VB))]>;
```
- **EN**: Adds declarative TableGen records such as `VABSDUB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VABSDUB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1690-1697

```tablegen
def VABSDUH : VXForm_1<1091, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                       "vabsduh $VD, $VA, $VB", IIC_VecGeneral,
                       [(set v8i16:$VD, (int_ppc_altivec_vabsduh v8i16:$VA, v8i16:$VB))]>;
def VABSDUW : VXForm_1<1155, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                       "vabsduw $VD, $VA, $VB", IIC_VecGeneral,
                       [(set v4i32:$VD, (int_ppc_altivec_vabsduw v4i32:$VA, v4i32:$VB))]>;

} // end HasP9Altivec
```
- **EN**: Adds declarative TableGen records such as `VABSDUH`, `VABSDUW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VABSDUH`, `VABSDUW`，LLVM 随后会把它们展开成生成表或辅助代码。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
