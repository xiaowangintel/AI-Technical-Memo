# Transforms.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/X86/Transforms.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR Transforms component. The leading comments describe it as: =- Transforms.h - X86 Dialect Transformation Entrypoints --------*- C++ -*-=//.
- **用途（CN）**: 声明 MLIR Transforms 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
````cpp
//=- Transforms.h - X86 Dialect Transformation Entrypoints --------*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_X86_TRANSFORMS_H
#define MLIR_DIALECT_X86_TRANSFORMS_H

#include "mlir/IR/Value.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 14-80
````cpp
namespace mlir {

class ImplicitLocOpBuilder;
class LLVMConversionTarget;
class LLVMTypeConverter;
class RewritePatternSet;

namespace x86 {

/// Helper class to factor out the creation and extraction of masks from nibs.
struct MaskHelper {
  /// b0 captures the lowest bit, b7 captures the highest bit.
  /// Meant to be used with instructions such as mm256BlendPs.
  template <uint8_t b0, uint8_t b1, uint8_t b2, uint8_t b3, uint8_t b4,
            uint8_t b5, uint8_t b6, uint8_t b7>
  static uint8_t blend() {
    static_assert(b0 <= 1 && b1 <= 1 && b2 <= 1 && b3 <= 1, "overflow");
    static_assert(b4 <= 1 && b5 <= 1 && b6 <= 1 && b7 <= 1, "overflow");
    return static_cast<uint8_t>((b7 << 7) | (b6 << 6) | (b5 << 5) | (b4 << 4) |
                                (b3 << 3) | (b2 << 2) | (b1 << 1) | b0);
  }
  /// b0 captures the lowest bit, b7 captures the highest bit.
  /// Meant to be used with instructions such as mm256BlendPs.
  static void extractBlend(uint8_t mask, uint8_t &b0, uint8_t &b1, uint8_t &b2,
                           uint8_t &b3, uint8_t &b4, uint8_t &b5, uint8_t &b6,
                           uint8_t &b7) {
    b7 = mask & (1 << 7);
    b6 = mask & (1 << 6);
    b5 = mask & (1 << 5);
    b4 = mask & (1 << 4);
    b3 = mask & (1 << 3);
    b2 = mask & (1 << 2);
    b1 = mask & (1 << 1);
    b0 = mask & 1;
  }
  /// b01 captures the lower 2 bits, b67 captures the higher 2 bits.
  /// Meant to be used with instructions such as mm256ShufflePs.
  template <unsigned b67, unsigned b45, unsigned b23, unsigned b01>
  static uint8_t shuffle() {
    static_assert(b01 <= 0x03, "overflow");
    static_assert(b23 <= 0x03, "overflow");
    static_assert(b45 <= 0x03, "overflow");
    static_assert(b67 <= 0x03, "overflow");
    return static_cast<uint8_t>((b67 << 6) | (b45 << 4) | (b23 << 2) | b01);
  }
  /// b01 captures the lower 2 bits, b67 captures the higher 2 bits.
  static void extractShuffle(uint8_t mask, uint8_t &b01, uint8_t &b23,
                             uint8_t &b45, uint8_t &b67) {
    b67 = (mask & (0x03 << 6)) >> 6;
    b45 = (mask & (0x03 << 4)) >> 4;
    b23 = (mask & (0x03 << 2)) >> 2;
    b01 = mask & 0x03;
  }
  /// b03 captures the lower 4 bits, b47 captures the higher 4 bits.
  /// Meant to be used with instructions such as mm256Permute2f128Ps.
  template <unsigned b47, unsigned b03>
  static uint8_t permute() {
    static_assert(b03 <= 0x0f, "overflow");
    static_assert(b47 <= 0x0f, "overflow");
    return static_cast<uint8_t>((b47 << 4) + b03);
  }
  /// b03 captures the lower 4 bits, b47 captures the higher 4 bits.
  static void extractPermute(uint8_t mask, uint8_t &b03, uint8_t &b47) {
    b47 = (mask & (0x0f << 4)) >> 4;
    b03 = mask & 0x0f;
  }
};
````
- **EN**: This C++ declaration introduces `ImplicitLocOpBuilder` and establishes part of the API surface for `Transforms`. Representative entry points here include `blend`, `static_assert`, `extractBlend`, `shuffle`.
- **CN**: 该 C++ 声明引入了 `ImplicitLocOpBuilder`，并构成 `Transforms` API 表面的一部分。 这一段可见的代表性接口包括 `blend`, `static_assert`, `extractBlend`, `shuffle`。

### Lines 81-110
````cpp
//===----------------------------------------------------------------------===//

// A set of patterns for specialized lowering of vector contraction
// operation to vector fused multiply and add (FMA) operation.
void populateVectorContractToFMAPatterns(RewritePatternSet &patterns);

// A set of patterns for lowering 32-bit packed vector contraction operations
// to their corresponding packed-type dot-product operations, ultimately
// targeting the relevant x86 LLVM intrinsics (e.g., BF16 and Int8).
void populateVectorContractToPackedTypeDotProductPatterns(
    RewritePatternSet &patterns);

// A set of patterns for lowering 32-bit packed BF16 vector contraction
// operations to vector fused multiply-add (FMA) operations, following
// the emulation-based approach using BF16 packed operations.
void populateVectorContractBF16ToFMAPatterns(RewritePatternSet &patterns);

// Performs forward scheduling of vector producer ops to minimize their live
// range by placing them at their earliest legal use site.
void populateSinkVectorProducerOpsPatterns(RewritePatternSet &patterns);

// Shuffles FMAs with x86 operations as operands such that FMAs are
// grouped with respect to odd/even packed index.
void populateShuffleVectorFMAOpsPatterns(RewritePatternSet &patterns);

// A set of patterns for lowering 32-bit packed vector contraction operations
// to their corresponding packed-type tiled dot-product operations, using
// AMX ultimately targeting the relevant x86 LLVM intrinsics (e.g., BF16 and
// Int8).
void populateVectorContractToAMXDotProductPatterns(RewritePatternSet &patterns);
````
- **EN**: This block groups callable interfaces such as `populateVectorContractToFMAPatterns`, `populateVectorContractToPackedTypeDotProductPatterns`, `populateVectorContractBF16ToFMAPatterns`, `populateSinkVectorProducerOpsPatterns`, indicating how `Transforms` is queried or updated.
- **CN**: 该代码块聚合了 `populateVectorContractToFMAPatterns`, `populateVectorContractToPackedTypeDotProductPatterns`, `populateVectorContractBF16ToFMAPatterns`, `populateSinkVectorProducerOpsPatterns` 等可调用接口，展示了如何查询或更新 `Transforms`。

### Lines 113-127
````cpp
//===----------------------------------------------------------------------===//
/// Helpers extracted from:
///   - clang/lib/Headers/avxintrin.h
///   - clang/test/CodeGen/X86/avx-builtins.c
///   - clang/test/CodeGen/X86/avx2-builtins.c
///   - clang/test/CodeGen/X86/avx-shuffle-builtins.c
/// as well as the Intel Intrinsics Guide
/// (https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html)
/// make it easier to just implement known good lowerings.
/// All intrinsics correspond 1-1 to the Intel definition.
//===----------------------------------------------------------------------===//

namespace avx2 {

namespace inline_asm {
````
- **EN**: This C++ declaration introduces `avx2` and establishes part of the API surface for `Transforms`.
- **CN**: 该 C++ 声明引入了 `avx2`，并构成 `Transforms` API 表面的一部分。

### Lines 128-137
````cpp
//===----------------------------------------------------------------------===//
/// Methods in the inline_asm namespace  emit calls to LLVM::InlineAsmOp.
//===----------------------------------------------------------------------===//
/// If bit i of `mask` is zero, take f32@i from v1 else take it from v2.
Value mm256BlendPsAsm(ImplicitLocOpBuilder &b, Value v1, Value v2,
                      uint8_t mask);

} // namespace inline_asm

namespace intrin {
````
- **EN**: This C++ declaration introduces `intrin` and establishes part of the API surface for `Transforms`. Representative entry points here include `mm256BlendPsAsm`.
- **CN**: 该 C++ 声明引入了 `intrin`，并构成 `Transforms` API 表面的一部分。 这一段可见的代表性接口包括 `mm256BlendPsAsm`。

### Lines 138-164
````cpp
//===----------------------------------------------------------------------===//
/// Methods in the intrin namespace emulate clang's impl. of X86 intrinsics.
//===----------------------------------------------------------------------===//
/// Lower to vector.shuffle v1, v2, [0, 8, 1, 9, 4, 12, 5, 13].
Value mm256UnpackLoPs(ImplicitLocOpBuilder &b, Value v1, Value v2);

/// Lower to vector.shuffle v1, v2, [0, 8, 1, 9, 4, 12, 5, 13].
Value mm256UnpackHiPs(ImplicitLocOpBuilder &b, Value v1, Value v2);

///                            a  a   b   b  a  a   b   b
/// Take an 8 bit mask, 2 bit for each position of a[0, 3)  **and** b[0, 4):
///                                 0:127    |         128:255
///                            b01  b23  C8  D8  |  b01+4 b23+4 C8+4 D8+4
Value mm256ShufflePs(ImplicitLocOpBuilder &b, Value v1, Value v2, uint8_t mask);

// imm[0:1] out of imm[0:3] is:
//    0             1           2             3
// a[0:127] or a[128:255] or b[0:127] or b[128:255]    |
//          a[0:127] or a[128:255] or b[0:127] or b[128:255]
//             0             1           2             3
// imm[0:1] out of imm[4:7].
Value mm256Permute2f128Ps(ImplicitLocOpBuilder &b, Value v1, Value v2,
                          uint8_t mask);

/// If bit i of `mask` is zero, take f32@i from v1 else take it from v2.
Value mm256BlendPs(ImplicitLocOpBuilder &b, Value v1, Value v2, uint8_t mask);
} // namespace intrin
````
- **EN**: This block groups callable interfaces such as `mm256UnpackLoPs`, `mm256UnpackHiPs`, `mm256ShufflePs`, `mm256Permute2f128Ps`, indicating how `Transforms` is queried or updated.
- **CN**: 该代码块聚合了 `mm256UnpackLoPs`, `mm256UnpackHiPs`, `mm256ShufflePs`, `mm256Permute2f128Ps` 等可调用接口，展示了如何查询或更新 `Transforms`。

### Lines 166-210
````cpp
//===----------------------------------------------------------------------===//
/// Generic lowerings may either use intrin or inline_asm depending on needs.
//===----------------------------------------------------------------------===//
/// 4x8xf32-specific AVX2 transpose lowering.
void transpose4x8xf32(ImplicitLocOpBuilder &ib, MutableArrayRef<Value> vs);

/// 8x8xf32-specific AVX2 transpose lowering.
void transpose8x8xf32(ImplicitLocOpBuilder &ib, MutableArrayRef<Value> vs);

/// Structure to control the behavior of specialized AVX2 transpose lowering.
struct TransposeLoweringOptions {
  bool lower4x8xf32_ = false;
  TransposeLoweringOptions &lower4x8xf32(bool lower = true) {
    lower4x8xf32_ = lower;
    return *this;
  }
  bool lower8x8xf32_ = false;
  TransposeLoweringOptions &lower8x8xf32(bool lower = true) {
    lower8x8xf32_ = lower;
    return *this;
  }
};

/// Options for controlling specialized AVX2 lowerings.
struct LoweringOptions {
  /// Configure specialized vector lowerings.
  TransposeLoweringOptions transposeOptions;
  LoweringOptions &setTransposeOptions(TransposeLoweringOptions options) {
    transposeOptions = options;
    return *this;
  }
};

/// Insert specialized transpose lowering patterns.
void populateSpecializedTransposeLoweringPatterns(
    RewritePatternSet &patterns, LoweringOptions options = LoweringOptions(),
    int benefit = 10);

} // namespace avx2
} // namespace x86

/// Collect a set of patterns to lower X86 ops to ops that map to LLVM
/// intrinsics.
void populateX86LegalizeForLLVMExportPatterns(LLVMTypeConverter &converter,
                                              RewritePatternSet &patterns);
````
- **EN**: This C++ declaration introduces `TransposeLoweringOptions` and establishes part of the API surface for `Transforms`. Representative entry points here include `transpose4x8xf32`, `transpose8x8xf32`, `lower4x8xf32`, `lower8x8xf32`.
- **CN**: 该 C++ 声明引入了 `TransposeLoweringOptions`，并构成 `Transforms` API 表面的一部分。 这一段可见的代表性接口包括 `transpose4x8xf32`, `transpose8x8xf32`, `lower4x8xf32`, `lower8x8xf32`。

### Lines 211-218
````cpp
/// Configure the target to support lowering X86 ops to ops that map to
/// LLVM intrinsics.
void configureX86LegalizeForExportTarget(LLVMConversionTarget &target);

/// Register LLVM conversion interface for X86 dialect.
void registerConvertX86ToLLVMInterface(DialectRegistry &registry);

} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `configureX86LegalizeForExportTarget`, `registerConvertX86ToLLVMInterface`, indicating how `Transforms` is queried or updated.
- **CN**: 该代码块聚合了 `configureX86LegalizeForExportTarget`, `registerConvertX86ToLLVMInterface` 等可调用接口，展示了如何查询或更新 `Transforms`。

### Lines 221-221
````cpp
#endif // MLIR_DIALECT_X86_TRANSFORMS_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/IR/Value.h
