# GCNProcessors.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/GCNProcessors.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This TableGen DSL file defines GCNProcessors records for the LLVM AMDGPU backend. It describes target features, instructions, predicates, or generated metadata consumed by LLVM table generators. / 该 TableGen DSL 文件为 LLVM AMDGPU 后端定义 GCNProcessors 相关记录。它描述目标特性、指令、谓词或供 LLVM 表生成器使用的元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, comments, and TableGen overview
```tablegen
//===-- GCNProcessors.td - GCN Processor definitions ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

// The code produced for "generic" is only useful for tests and cannot
// reasonably be expected to execute on any particular target.
def : ProcessorModel<"generic", NoSchedModel,
  []
>;

def : ProcessorModel<"generic-hsa", NoSchedModel,
  [FeatureFlatAddressSpace]
>;

//===------------------------------------------------------------===//
// GCN GFX6 (Southern Islands (SI)).
//===------------------------------------------------------------===//

def : ProcessorModel<"gfx600", SIFullSpeedModel,
  FeatureISAVersion6_0_0.Features
>;

def : ProcessorModel<"tahiti", SIFullSpeedModel,
  FeatureISAVersion6_0_0.Features
>;

def : ProcessorModel<"gfx601", SIQuarterSpeedModel,
  FeatureISAVersion6_0_1.Features
>;

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 35-66: TableGen records and backend metadata
```tablegen
def : ProcessorModel<"pitcairn", SIQuarterSpeedModel,
  FeatureISAVersion6_0_1.Features
>;

def : ProcessorModel<"verde", SIQuarterSpeedModel,
  FeatureISAVersion6_0_1.Features
>;

def : ProcessorModel<"gfx602", SIQuarterSpeedModel,
  FeatureISAVersion6_0_2.Features
>;

def : ProcessorModel<"hainan", SIQuarterSpeedModel,
  FeatureISAVersion6_0_2.Features
>;

def : ProcessorModel<"oland", SIQuarterSpeedModel,
  FeatureISAVersion6_0_2.Features
>;

//===------------------------------------------------------------===//
// GCN GFX7 (Sea Islands (CI)).
//===------------------------------------------------------------===//

def : ProcessorModel<"gfx700", SIQuarterSpeedModel,
  FeatureISAVersion7_0_0.Features
>;

def : ProcessorModel<"kaveri", SIQuarterSpeedModel,
  FeatureISAVersion7_0_0.Features
>;

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 67-98: TableGen records and backend metadata
```tablegen
def : ProcessorModel<"gfx701", SIFullSpeedModel,
  FeatureISAVersion7_0_1.Features
>;

def : ProcessorModel<"hawaii", SIFullSpeedModel,
  FeatureISAVersion7_0_1.Features
>;

def : ProcessorModel<"gfx702", SIQuarterSpeedModel,
  FeatureISAVersion7_0_2.Features
>;

def : ProcessorModel<"gfx703", SIQuarterSpeedModel,
  FeatureISAVersion7_0_3.Features
>;

def : ProcessorModel<"kabini", SIQuarterSpeedModel,
  FeatureISAVersion7_0_3.Features
>;

def : ProcessorModel<"mullins", SIQuarterSpeedModel,
  FeatureISAVersion7_0_3.Features
>;

def : ProcessorModel<"gfx704", SIQuarterSpeedModel,
  FeatureISAVersion7_0_4.Features
>;

def : ProcessorModel<"bonaire", SIQuarterSpeedModel,
  FeatureISAVersion7_0_4.Features
>;

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 99-130: TableGen records and backend metadata
```tablegen
def : ProcessorModel<"gfx705", SIQuarterSpeedModel,
  FeatureISAVersion7_0_5.Features
>;

//===------------------------------------------------------------===//
// GCN GFX8 (Volcanic Islands (VI)).
//===------------------------------------------------------------===//

def : ProcessorModel<"gfx801", SIQuarterSpeedModel,
  FeatureISAVersion8_0_1.Features
>;

def : ProcessorModel<"carrizo", SIQuarterSpeedModel,
  FeatureISAVersion8_0_1.Features
>;

def : ProcessorModel<"gfx802", SIQuarterSpeedModel,
  FeatureISAVersion8_0_2.Features
>;

def : ProcessorModel<"iceland", SIQuarterSpeedModel,
  FeatureISAVersion8_0_2.Features
>;

def : ProcessorModel<"tonga", SIQuarterSpeedModel,
  FeatureISAVersion8_0_2.Features
>;

def : ProcessorModel<"gfx803", SIQuarterSpeedModel,
  FeatureISAVersion8_0_3.Features
>;

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 131-162: TableGen records and backend metadata
```tablegen
def : ProcessorModel<"fiji", SIQuarterSpeedModel,
  FeatureISAVersion8_0_3.Features
>;

def : ProcessorModel<"polaris10", SIQuarterSpeedModel,
  FeatureISAVersion8_0_3.Features
>;

def : ProcessorModel<"polaris11", SIQuarterSpeedModel,
  FeatureISAVersion8_0_3.Features
>;

def : ProcessorModel<"gfx805", SIQuarterSpeedModel,
  FeatureISAVersion8_0_5.Features
>;

def : ProcessorModel<"tongapro", SIQuarterSpeedModel,
  FeatureISAVersion8_0_5.Features
>;

def : ProcessorModel<"gfx810", SIQuarterSpeedModel,
  FeatureISAVersion8_1_0.Features
>;

def : ProcessorModel<"stoney", SIQuarterSpeedModel,
  FeatureISAVersion8_1_0.Features
>;

//===------------------------------------------------------------===//
// GCN GFX9.
//===------------------------------------------------------------===//

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 163-194: TableGen records and backend metadata
```tablegen
def : ProcessorModel<"gfx900", SIQuarterSpeedModel,
  FeatureISAVersion9_0_0.Features
>;

def : ProcessorModel<"gfx902", SIQuarterSpeedModel,
  FeatureISAVersion9_0_2.Features
>;

def : ProcessorModel<"gfx904", SIQuarterSpeedModel,
  FeatureISAVersion9_0_4.Features
>;

def : ProcessorModel<"gfx906", SIQuarterSpeedModel,
  FeatureISAVersion9_0_6.Features
>;

def : ProcessorModel<"gfx908", SIQuarterSpeedModel,
  FeatureISAVersion9_0_8.Features
>;

def : ProcessorModel<"gfx909", SIQuarterSpeedModel,
  FeatureISAVersion9_0_9.Features
>;

def : ProcessorModel<"gfx90a", SIDPFullSpeedModel,
  FeatureISAVersion9_0_A.Features
>;

def : ProcessorModel<"gfx90c", SIQuarterSpeedModel,
  FeatureISAVersion9_0_C.Features
>;

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 195-228: TableGen records and backend metadata
```tablegen
def : ProcessorModel<"gfx942", SIDPGFX942FullSpeedModel,
  FeatureISAVersion9_4_2.Features
>;

def : ProcessorModel<"gfx950", SIDPGFX950FullSpeedModel,
  FeatureISAVersion9_5_0.Features
>;

// [gfx900, gfx902, gfx904, gfx906, gfx909, gfx90c]
def : ProcessorModel<"gfx9-generic", SIQuarterSpeedModel,
  FeatureISAVersion9_Generic.Features
>;

// [gfx942]
def : ProcessorModel<"gfx9-4-generic", SIDPGFX942FullSpeedModel,
  FeatureISAVersion9_4_Generic.Features
>;

//===----------------------------------------------------------------------===//
// GCN GFX10.
//===----------------------------------------------------------------------===//

def : ProcessorModel<"gfx1010", GFX10SpeedModel,
  FeatureISAVersion10_1_0.Features
>;

def : ProcessorModel<"gfx1011", GFX10SpeedModel,
  FeatureISAVersion10_1_1.Features
>;

def : ProcessorModel<"gfx1012", GFX10SpeedModel,
  FeatureISAVersion10_1_2.Features
>;

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 229-260: TableGen records and backend metadata
```tablegen
def : ProcessorModel<"gfx1013", GFX10SpeedModel,
  FeatureISAVersion10_1_3.Features
>;

def : ProcessorModel<"gfx1030", GFX10SpeedModel,
  FeatureISAVersion10_3_0.Features
>;

def : ProcessorModel<"gfx1031", GFX10SpeedModel,
  FeatureISAVersion10_3_0.Features
>;

def : ProcessorModel<"gfx1032", GFX10SpeedModel,
  FeatureISAVersion10_3_0.Features
>;

def : ProcessorModel<"gfx1033", GFX10SpeedModel,
  FeatureISAVersion10_3_0.Features
>;

def : ProcessorModel<"gfx1034", GFX10SpeedModel,
  FeatureISAVersion10_3_0.Features
>;

def : ProcessorModel<"gfx1035", GFX10SpeedModel,
  FeatureISAVersion10_3_0.Features
>;

def : ProcessorModel<"gfx1036", GFX10SpeedModel,
  FeatureISAVersion10_3_0.Features
>;

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 261-294: TableGen records and backend metadata
```tablegen
// [gfx1010, gfx1011, gfx1012, gfx1013]
def : ProcessorModel<"gfx10-1-generic", GFX10SpeedModel,
  FeatureISAVersion10_1_Generic.Features
>;

// [gfx1030, gfx1031, gfx1032, gfx1033, gfx1034, gfx1035, gfx1036]
def : ProcessorModel<"gfx10-3-generic", GFX10SpeedModel,
  FeatureISAVersion10_3_Generic.Features
>;

//===----------------------------------------------------------------------===//
// GCN GFX11.
//===----------------------------------------------------------------------===//

def : ProcessorModel<"gfx1100", GFX11SpeedModel,
  FeatureISAVersion11_0_0.Features
>;

def : ProcessorModel<"gfx1101", GFX11SpeedModel,
  FeatureISAVersion11_0_1.Features
>;

def : ProcessorModel<"gfx1102", GFX11SpeedModel,
  FeatureISAVersion11_0_2.Features
>;

def : ProcessorModel<"gfx1103", GFX11SpeedModel,
  FeatureISAVersion11_0_3.Features
>;

def : ProcessorModel<"gfx1150", GFX11SpeedModel,
  FeatureISAVersion11_5_0.Features
>;

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 295-327: TableGen records and backend metadata
```tablegen
def : ProcessorModel<"gfx1151", GFX11SpeedModel,
  FeatureISAVersion11_5_1.Features
>;

def : ProcessorModel<"gfx1152", GFX11SpeedModel,
  FeatureISAVersion11_5_2.Features
>;

def : ProcessorModel<"gfx1153", GFX11SpeedModel,
  FeatureISAVersion11_5_3.Features
>;

def : ProcessorModel<"gfx1170", GFX11SpeedModel,
  FeatureISAVersion11_7_Common.Features
>;

def : ProcessorModel<"gfx1171", GFX11SpeedModel,
  FeatureISAVersion11_7_Common.Features
>;

def : ProcessorModel<"gfx1172", GFX11SpeedModel,
  FeatureISAVersion11_7_Common.Features
>;

// [gfx1100, gfx1101, gfx1102, gfx1103, gfx1150, gfx1151, gfx1152, gfx1153]
def : ProcessorModel<"gfx11-generic", GFX11SpeedModel,
  FeatureISAVersion11_Generic.Features
>;

//===----------------------------------------------------------------------===//
// GCN GFX12.
//===----------------------------------------------------------------------===//

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 328-360: TableGen records and backend metadata
```tablegen
def : ProcessorModel<"gfx1200", GFX12SpeedModel,
  FeatureISAVersion12.Features
>;

def : ProcessorModel<"gfx1201", GFX12SpeedModel,
  FeatureISAVersion12.Features
>;

// [gfx1200, gfx1201]
def : ProcessorModel<"gfx12-generic", GFX12SpeedModel,
  FeatureISAVersion12_Generic.Features
>;

def : ProcessorModel<"gfx1250", GFX1250SpeedModel,
  FeatureISAVersion12_50.Features
>;

def : ProcessorModel<"gfx1251", GFX1250SpeedModel,
  FeatureISAVersion12_51.Features
>;

// [gfx1250, gfx1251]
def : ProcessorModel<"gfx12-5-generic", GFX1250SpeedModel,
  FeatureISAVersion12_5_Generic.Features
>;

//===----------------------------------------------------------------------===//
// GCN GFX13.
//===----------------------------------------------------------------------===//

def : ProcessorModel<"gfx1310", GFX12SpeedModel,
  FeatureISAVersion13.Features
>;
```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

## Key Concepts / 关键概念
- **Language / 语言**: TableGen DSL
- **Main themes / 核心主题**: scheduling / 调度; feature description / 特性描述
- **Build role / 构建角色**: Generates target records consumed by LLVM TableGen backends / 生成供 LLVM TableGen 后端消费的目标记录

## Dependencies / 依赖关系
- No direct include statements in this file / 此文件中没有直接的 include 语句
