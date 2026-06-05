# X86ShuffleDecode.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/MCTargetDesc/X86ShuffleDecode.h`
- **Repository**: llvm/llvm-project
- **Purpose**: Declares interfaces and shared data structures for X86 target definitions in the X86 MC target description layer. / 为X86 MC 目标描述层中的X86 目标定义声明接口与共享数据结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===-- X86ShuffleDecode.h - X86 shuffle decode logic -----------*-C++-*---===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Define several functions to decode x86 specific shuffle semantics into a
// generic vector mask.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_X86_UTILS_X86SHUFFLEDECODE_H
#define LLVM_LIB_TARGET_X86_UTILS_X86SHUFFLEDECODE_H

#include <cstdint>

//===----------------------------------------------------------------------===//
//  Vector Mask Decoding
//===----------------------------------------------------------------------===//

namespace llvm {
class APInt;
template <typename T> class ArrayRef;
template <typename T> class SmallVectorImpl;

enum { SM_SentinelUndef = -1, SM_SentinelZero = -2 };

/// Decode a 128-bit INSERTPS instruction as a v4f32 shuffle mask.
```
**EN:** This section declares APIs, helper types, or inline logic related to the X86 MC target description layer. Key symbols include APInt. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 MC 目标描述层相关的 API、辅助类型或内联逻辑。关键符号包括 APInt。这些内容定义了实现文件所依赖的契约。

### Lines 31-60: Supporting logic and local data handling / 辅助逻辑与局部数据处理
```cpp
void DecodeINSERTPSMask(unsigned Imm, SmallVectorImpl<int> &ShuffleMask,
                        bool SrcIsMem);

// Insert the bottom Len elements from a second source into a vector starting at
// element Idx.
void DecodeInsertElementMask(unsigned NumElts, unsigned Idx, unsigned Len,
                             SmallVectorImpl<int> &ShuffleMask);

/// Decode a MOVHLPS instruction as a v2f64/v4f32 shuffle mask.
/// i.e. <3,1> or <6,7,2,3>
void DecodeMOVHLPSMask(unsigned NElts, SmallVectorImpl<int> &ShuffleMask);

/// Decode a MOVLHPS instruction as a v2f64/v4f32 shuffle mask.
/// i.e. <0,2> or <0,1,4,5>
void DecodeMOVLHPSMask(unsigned NElts, SmallVectorImpl<int> &ShuffleMask);

void DecodeMOVSLDUPMask(unsigned NumElts, SmallVectorImpl<int> &ShuffleMask);

void DecodeMOVSHDUPMask(unsigned NumElts, SmallVectorImpl<int> &ShuffleMask);

void DecodeMOVDDUPMask(unsigned NumElts, SmallVectorImpl<int> &ShuffleMask);

void DecodePSLLDQMask(unsigned NumElts, unsigned Imm,
                      SmallVectorImpl<int> &ShuffleMask);

void DecodePSRLDQMask(unsigned NumElts, unsigned Imm,
                      SmallVectorImpl<int> &ShuffleMask);

void DecodePALIGNRMask(unsigned NumElts, unsigned Imm,
                       SmallVectorImpl<int> &ShuffleMask);
```
**EN:** This section declares APIs, helper types, or inline logic related to the X86 MC target description layer. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 MC 目标描述层相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 61-90: Supporting logic and local data handling / 辅助逻辑与局部数据处理
```cpp

void DecodeVALIGNMask(unsigned NumElts, unsigned Imm,
                      SmallVectorImpl<int> &ShuffleMask);

/// Decodes the shuffle masks for pshufd/pshufw/vpermilpd/vpermilps.
void DecodePSHUFMask(unsigned NumElts, unsigned ScalarBits, unsigned Imm,
                     SmallVectorImpl<int> &ShuffleMask);

/// Decodes the shuffle masks for pshufhw.
void DecodePSHUFHWMask(unsigned NumElts, unsigned Imm,
                       SmallVectorImpl<int> &ShuffleMask);

/// Decodes the shuffle masks for pshuflw.
void DecodePSHUFLWMask(unsigned NumElts, unsigned Imm,
                       SmallVectorImpl<int> &ShuffleMask);

/// Decodes a PSWAPD 3DNow! instruction.
void DecodePSWAPMask(unsigned NumElts, SmallVectorImpl<int> &ShuffleMask);

/// Decodes the shuffle masks for shufp*.
void DecodeSHUFPMask(unsigned NumElts, unsigned ScalarBits, unsigned Imm,
                     SmallVectorImpl<int> &ShuffleMask);

/// Decodes the shuffle masks for unpckhps/unpckhpd and punpckh*.
void DecodeUNPCKHMask(unsigned NumElts, unsigned ScalarBits,
                      SmallVectorImpl<int> &ShuffleMask);

/// Decodes the shuffle masks for unpcklps/unpcklpd and punpckl*.
void DecodeUNPCKLMask(unsigned NumElts, unsigned ScalarBits,
                      SmallVectorImpl<int> &ShuffleMask);
```
**EN:** This section declares APIs, helper types, or inline logic related to the X86 MC target description layer. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 MC 目标描述层相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 91-120: Comments and explanatory notes / 注释与说明性文字
```cpp

/// Decodes a broadcast of the first element of a vector.
void DecodeVectorBroadcast(unsigned NumElts, SmallVectorImpl<int> &ShuffleMask);

/// Decodes a broadcast of a subvector to a larger vector type.
void DecodeSubVectorBroadcast(unsigned DstNumElts, unsigned SrcNumElts,
                              SmallVectorImpl<int> &ShuffleMask);

/// Decode a PSHUFB mask from a raw array of constants such as from
/// BUILD_VECTOR.
void DecodePSHUFBMask(ArrayRef<uint64_t> RawMask, const APInt &UndefElts,
                      SmallVectorImpl<int> &ShuffleMask);

/// Decode a BLEND immediate mask into a shuffle mask.
void DecodeBLENDMask(unsigned NumElts, unsigned Imm,
                     SmallVectorImpl<int> &ShuffleMask);

void DecodeVPERM2X128Mask(unsigned NumElts, unsigned Imm,
                          SmallVectorImpl<int> &ShuffleMask);

/// Decode a shuffle packed values at 128-bit granularity
/// (SHUFF32x4/SHUFF64x2/SHUFI32x4/SHUFI64x2)
/// immediate mask into a shuffle mask.
void decodeVSHUF64x2FamilyMask(unsigned NumElts, unsigned ScalarSize,
                               unsigned Imm, SmallVectorImpl<int> &ShuffleMask);

/// Decodes the shuffle masks for VPERMQ/VPERMPD.
void DecodeVPERMMask(unsigned NumElts, unsigned Imm,
                     SmallVectorImpl<int> &ShuffleMask);

```
**EN:** This section declares APIs, helper types, or inline logic related to the X86 MC target description layer. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 MC 目标描述层相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 121-150: Comments and explanatory notes / 注释与说明性文字
```cpp
/// Decode a VPPERM mask from a raw array of constants such as from
/// BUILD_VECTOR.
/// This can only basic masks (permutes + zeros), not any of the other
/// operations that VPPERM can perform.
void DecodeVPPERMMask(ArrayRef<uint64_t> RawMask, const APInt &UndefElts,
                      SmallVectorImpl<int> &ShuffleMask);

/// Decode a zero extension instruction as a shuffle mask.
void DecodeZeroExtendMask(unsigned SrcScalarBits, unsigned DstScalarBits,
                          unsigned NumDstElts, bool IsAnyExtend,
                          SmallVectorImpl<int> &ShuffleMask);

/// Decode a move lower and zero upper instruction as a shuffle mask.
void DecodeZeroMoveLowMask(unsigned NumElts, SmallVectorImpl<int> &ShuffleMask);

/// Decode a scalar float move instruction as a shuffle mask.
void DecodeScalarMoveMask(unsigned NumElts, bool IsLoad,
                          SmallVectorImpl<int> &ShuffleMask);

/// Decode a SSE4A EXTRQ instruction as a shuffle mask.
void DecodeEXTRQIMask(unsigned NumElts, unsigned EltSize, int Len, int Idx,
                      SmallVectorImpl<int> &ShuffleMask);

/// Decode a SSE4A INSERTQ instruction as a shuffle mask.
void DecodeINSERTQIMask(unsigned NumElts, unsigned EltSize, int Len, int Idx,
                        SmallVectorImpl<int> &ShuffleMask);

/// Decode a VPERMILPD/VPERMILPS variable mask from a raw array of constants.
void DecodeVPERMILPMask(unsigned NumElts, unsigned ScalarBits,
                        ArrayRef<uint64_t> RawMask, const APInt &UndefElts,
```
**EN:** This section declares APIs, helper types, or inline logic related to the X86 MC target description layer. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 MC 目标描述层相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 151-167: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
                        SmallVectorImpl<int> &ShuffleMask);

/// Decode a VPERMIL2PD/VPERMIL2PS variable mask from a raw array of constants.
void DecodeVPERMIL2PMask(unsigned NumElts, unsigned ScalarBits, unsigned M2Z,
                         ArrayRef<uint64_t> RawMask, const APInt &UndefElts,
                         SmallVectorImpl<int> &ShuffleMask);

/// Decode a VPERM W/D/Q/PS/PD mask from a raw array of constants.
void DecodeVPERMVMask(ArrayRef<uint64_t> RawMask, const APInt &UndefElts,
                      SmallVectorImpl<int> &ShuffleMask);

/// Decode a VPERMT2 W/D/Q/PS/PD mask from a raw array of constants.
void DecodeVPERMV3Mask(ArrayRef<uint64_t> RawMask, const APInt &UndefElts,
                      SmallVectorImpl<int> &ShuffleMask);
} // llvm namespace

#endif
```
**EN:** This section declares APIs, helper types, or inline logic related to the X86 MC target description layer. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 MC 目标描述层相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

## Key Concepts / 关键概念
- Primary topic: X86 target definitions. / 核心主题：X86 目标定义。
- Subsystem: the X86 MC target description layer. / 所属子系统：X86 MC 目标描述层。
- Notable symbols: APInt. / 重要符号：APInt。
- The file emphasizes declarations, interfaces, and reusable helpers. / 该文件以声明、接口和可复用辅助逻辑为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: cstdint. / 直接包含：cstdint。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
- Implementation files in the same subsystem rely on these declarations. / 同一子系统中的实现文件依赖这些声明。
