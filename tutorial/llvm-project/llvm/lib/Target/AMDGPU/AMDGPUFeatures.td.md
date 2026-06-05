# AMDGPUFeatures.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUFeatures.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This TableGen DSL file defines AMDGPUFeatures records for the LLVM AMDGPU backend. It describes target features, instructions, predicates, or generated metadata consumed by LLVM table generators. / 该 TableGen DSL 文件为 LLVM AMDGPU 后端定义 AMDGPUFeatures 相关记录。它描述目标特性、指令、谓词或供 LLVM 表生成器使用的元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: File banner, comments, and TableGen overview
```tablegen
//===-- AMDGPUFeatures.td - AMDGPU Feature Definitions -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

def FeatureFP64 : SubtargetFeature<"fp64",
  "HasFP64",
  "true",
  "Enable double precision operations"
>;

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins. Main symbols: `FeatureFP64`.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。 主要符号：`FeatureFP64`。

### Lines 15-30: Defines TableGen record FeatureFMA
```tablegen
def FeatureFMA : SubtargetFeature<"fmaf",
  "HasFMA",
  "true",
  "Enable single precision FMA (not as fast as mul+add, but fused)"
>;

class SubtargetFeatureFlatOffsetBits <int Value> : SubtargetFeature<
  "flat-offset-bits-" # Value,
  "FlatOffsetBitWidth",
  !cast<string>(Value),
  "Number of bits for flat offset encoding"
>;

def FeatureFlatOffsetBits12 : SubtargetFeatureFlatOffsetBits<12>;
def FeatureFlatOffsetBits24 : SubtargetFeatureFlatOffsetBits<24>;

```
**EN:** This section contains concrete logic for TableGen record FeatureFMA. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `FeatureFMA`, `SubtargetFeatureFlatOffsetBits`, `FeatureFlatOffsetBits12`.
**CN:** 本节包含与 TableGen record FeatureFMA 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`FeatureFMA`, `SubtargetFeatureFlatOffsetBits`, `FeatureFlatOffsetBits12`。

### Lines 31-44: Defines TableGen class SubtargetFeatureAddressableLocalMemorySize
```tablegen
// Addressable local memory size is the maximum number of bytes of LDS that can
// be allocated to a single workgroup.
class SubtargetFeatureAddressableLocalMemorySize <int Value> : SubtargetFeature<
  "addressablelocalmemorysize"#Value,
  "AddressableLocalMemorySize",
  !cast<string>(Value),
  "The size of local memory in bytes"
>;

def FeatureAddressableLocalMemorySize32768 : SubtargetFeatureAddressableLocalMemorySize<32768>;
def FeatureAddressableLocalMemorySize65536 : SubtargetFeatureAddressableLocalMemorySize<65536>;
def FeatureAddressableLocalMemorySize163840 : SubtargetFeatureAddressableLocalMemorySize<163840>;
def FeatureAddressableLocalMemorySize327680 : SubtargetFeatureAddressableLocalMemorySize<327680>;

```
**EN:** This section contains concrete logic for TableGen class SubtargetFeatureAddressableLocalMemorySize. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SubtargetFeatureAddressableLocalMemorySize`, `FeatureAddressableLocalMemorySize32768`, `FeatureAddressableLocalMemorySize65536`.
**CN:** 本节包含与 TableGen class SubtargetFeatureAddressableLocalMemorySize 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SubtargetFeatureAddressableLocalMemorySize`, `FeatureAddressableLocalMemorySize32768`, `FeatureAddressableLocalMemorySize65536`。

### Lines 45-61: Defines TableGen class SubtargetFeatureWavefrontSize
```tablegen
class SubtargetFeatureWavefrontSize <int ValueLog2> : SubtargetFeature<
  "wavefrontsize"#!shl(1, ValueLog2),
  "WavefrontSizeLog2",
  !cast<string>(ValueLog2),
  "The number of threads per wavefront"
>;

def FeatureWavefrontSize16 : SubtargetFeatureWavefrontSize<4>;
def FeatureWavefrontSize32 : SubtargetFeatureWavefrontSize<5>;
def FeatureWavefrontSize64 : SubtargetFeatureWavefrontSize<6>;

class SubtargetFeatureGeneration <string Value, string FeatureName,
                                 string Subtarget,
                                  list<SubtargetFeature> Implies> :
        SubtargetFeature <FeatureName, "Gen", Subtarget#"::"#Value,
                          Value#" GPU generation", Implies>;

```
**EN:** This section contains concrete logic for TableGen class SubtargetFeatureWavefrontSize. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SubtargetFeatureWavefrontSize`, `FeatureWavefrontSize16`, `FeatureWavefrontSize32`.
**CN:** 本节包含与 TableGen class SubtargetFeatureWavefrontSize 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SubtargetFeatureWavefrontSize`, `FeatureWavefrontSize16`, `FeatureWavefrontSize32`。

## Key Concepts / 关键概念
- **Language / 语言**: TableGen DSL
- **Primary symbols / 主要符号**: `FeatureFP64`, `FeatureFMA`, `SubtargetFeatureFlatOffsetBits`, `FeatureFlatOffsetBits12`, `FeatureFlatOffsetBits24`, `SubtargetFeatureAddressableLocalMemorySize`
- **Main themes / 核心主题**: subtarget modeling / 子目标建模; feature description / 特性描述
- **Build role / 构建角色**: Generates target records consumed by LLVM TableGen backends / 生成供 LLVM TableGen 后端消费的目标记录

## Dependencies / 依赖关系
- No direct include statements in this file / 此文件中没有直接的 include 语句
