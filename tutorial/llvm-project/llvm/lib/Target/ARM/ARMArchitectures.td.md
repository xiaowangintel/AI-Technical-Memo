# ARMArchitectures.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMArchitectures.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `ARMArchitectures` in LLVM TableGen DSL for the ARM backend, covering architecture/version metadata.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMArchitectures`，涵盖体系结构/版本元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3
```tablegen
class Architecture<string fname, string aname, list<SubtargetFeature> features>
  : SubtargetFeature<fname, "ARMArch", aname,
                     !strconcat(aname, " architecture"), features>;
```
- EN: Declares reusable TableGen class `Architecture` for `ARMArchitectures`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMArchitectures` 声明可复用的 TableGen 类 `Architecture`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5-7
```tablegen
//===----------------------------------------------------------------------===//
// ARM architectures
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 9-9
```tablegen
def ARMv4     : Architecture<"armv4",     "ARMv4",    []>;
```
- EN: Defines TableGen record `ARMv4` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv4`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 11-11
```tablegen
def ARMv4t    : Architecture<"armv4t",    "ARMv4t",   [HasV4TOps]>;
```
- EN: Defines TableGen record `ARMv4t` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv4t`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 13-13
```tablegen
def ARMv5t    : Architecture<"armv5t",    "ARMv5t",   [HasV5TOps]>;
```
- EN: Defines TableGen record `ARMv5t` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv5t`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 15-15
```tablegen
def ARMv5te   : Architecture<"armv5te",   "ARMv5te",  [HasV5TEOps]>;
```
- EN: Defines TableGen record `ARMv5te` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv5te`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 17-17
```tablegen
def ARMv5tej  : Architecture<"armv5tej",  "ARMv5tej", [HasV5TEOps]>;
```
- EN: Defines TableGen record `ARMv5tej` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv5tej`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 19-20
```tablegen
def ARMv6     : Architecture<"armv6",     "ARMv6",    [HasV6Ops,
                                                       FeatureDSP]>;
```
- EN: Defines TableGen record `ARMv6` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv6`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 22-23
```tablegen
def ARMv6t2   : Architecture<"armv6t2",   "ARMv6t2",  [HasV6T2Ops,
                                                       FeatureDSP]>;
```
- EN: Defines TableGen record `ARMv6t2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv6t2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 25-25
```tablegen
def ARMv6k    : Architecture<"armv6k",    "ARMv6k",   [HasV6KOps]>;
```
- EN: Defines TableGen record `ARMv6k` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv6k`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 27-28
```tablegen
def ARMv6kz   : Architecture<"armv6kz",   "ARMv6kz",  [HasV6KOps,
                                                       FeatureTrustZone]>;
```
- EN: Defines TableGen record `ARMv6kz` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv6kz`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 30-35
```tablegen
def ARMv6m    : Architecture<"armv6-m",   "ARMv6m",   [HasV6MOps,
                                                       FeatureNoARM,
                                                       ModeThumb,
                                                       FeatureDB,
                                                       FeatureMClass,
                                                       FeatureStrictAlign]>;
```
- EN: Defines TableGen record `ARMv6m` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv6m`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 37-42
```tablegen
def ARMv6sm   : Architecture<"armv6s-m",  "ARMv6sm",  [HasV6MOps,
                                                       FeatureNoARM,
                                                       ModeThumb,
                                                       FeatureDB,
                                                       FeatureMClass,
                                                       FeatureStrictAlign]>;
```
- EN: Defines TableGen record `ARMv6sm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv6sm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 44-49
```tablegen
def ARMv7a    : Architecture<"armv7-a",   "ARMv7a",   [HasV7Ops,
                                                       FeatureNEON,
                                                       FeatureDB,
                                                       FeatureDSP,
                                                       FeatureAClass,
                                                       FeaturePerfMon]>;
```
- EN: Defines TableGen record `ARMv7a` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv7a`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 51-59
```tablegen
def ARMv7ve   : Architecture<"armv7ve",   "ARMv7ve",  [HasV7Ops,
                                                       FeatureNEON,
                                                       FeatureDB,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureAClass,
                                                       FeaturePerfMon]>;
```
- EN: Defines TableGen record `ARMv7ve` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv7ve`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 61-66
```tablegen
def ARMv7r    : Architecture<"armv7-r",   "ARMv7r",   [HasV7Ops,
                                                       FeatureDB,
                                                       FeatureDSP,
                                                       FeatureHWDivThumb,
                                                       FeatureRClass,
                                                       FeaturePerfMon]>;
```
- EN: Defines TableGen record `ARMv7r` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv7r`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 68-74
```tablegen
def ARMv7m    : Architecture<"armv7-m",   "ARMv7m",   [HasV7Ops,
                                                       FeatureThumb2,
                                                       FeatureNoARM,
                                                       ModeThumb,
                                                       FeatureDB,
                                                       FeatureHWDivThumb,
                                                       FeatureMClass]>;
```
- EN: Defines TableGen record `ARMv7m` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv7m`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 76-83
```tablegen
def ARMv7em   : Architecture<"armv7e-m",  "ARMv7em",  [HasV7Ops,
                                                       FeatureThumb2,
                                                       FeatureNoARM,
                                                       ModeThumb,
                                                       FeatureDB,
                                                       FeatureHWDivThumb,
                                                       FeatureMClass,
                                                       FeatureDSP]>;
```
- EN: Defines TableGen record `ARMv7em` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv7em`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 85-95
```tablegen
def ARMv8a    : Architecture<"armv8-a",   "ARMv8a",   [HasV8Ops,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCrypto,
                                                       FeatureCRC]>;
```
- EN: Defines TableGen record `ARMv8a` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv8a`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 97-107
```tablegen
def ARMv81a   : Architecture<"armv8.1-a", "ARMv81a",  [HasV8_1aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCrypto,
                                                       FeatureCRC]>;
```
- EN: Defines TableGen record `ARMv81a` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv81a`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 109-120
```tablegen
def ARMv82a   : Architecture<"armv8.2-a", "ARMv82a",  [HasV8_2aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCrypto,
                                                       FeatureCRC,
                                                       FeatureRAS]>;
```
- EN: Defines TableGen record `ARMv82a` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv82a`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 122-133
```tablegen
def ARMv83a   : Architecture<"armv8.3-a", "ARMv83a",  [HasV8_3aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCrypto,
                                                       FeatureCRC,
                                                       FeatureRAS]>;
```
- EN: Defines TableGen record `ARMv83a` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv83a`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 135-147
```tablegen
def ARMv84a   : Architecture<"armv8.4-a", "ARMv84a",  [HasV8_4aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCrypto,
                                                       FeatureCRC,
                                                       FeatureRAS,
                                                       FeatureDotProd]>;
```
- EN: Defines TableGen record `ARMv84a` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv84a`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 149-166
```tablegen
def ARMv85a   : Architecture<"armv8.5-a", "ARMv85a",  [HasV8_5aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCrypto,
                                                       FeatureCRC,
                                                       FeatureRAS,
                                                       FeatureDotProd]>;
def ARMv86a   : Architecture<"armv8.6-a", "ARMv86a",  [HasV8_6aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
```
- EN: Defines TableGen record `ARMv85a` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv85a`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 167-184
```tablegen
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCrypto,
                                                       FeatureCRC,
                                                       FeatureRAS,
                                                       FeatureDotProd]>;
def ARMv87a   : Architecture<"armv8.7-a", "ARMv87a",  [HasV8_7aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCrypto,
```
- EN: Defines TableGen record `ARMv87a` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv87a`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 185-202
```tablegen
                                                       FeatureCRC,
                                                       FeatureRAS,
                                                       FeatureDotProd]>;
def ARMv88a   : Architecture<"armv8.8-a", "ARMv88a",  [HasV8_8aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCrypto,
                                                       FeatureCRC,
                                                       FeatureRAS,
                                                       FeatureDotProd]>;
def ARMv89a   : Architecture<"armv8.9-a", "ARMv89a",  [HasV8_9aOps,
                                                       FeatureAClass,
```
- EN: Defines TableGen record `ARMv88a` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv88a`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 203-213
```tablegen
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCrypto,
                                                       FeatureCRC,
                                                       FeatureRAS,
                                                       FeatureDotProd]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 215-232
```tablegen
def ARMv9a   : Architecture<"armv9-a", "ARMv9a",       [HasV9_0aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCRC,
                                                       FeatureRAS,
                                                       FeatureDotProd]>;
def ARMv91a   : Architecture<"armv9.1-a", "ARMv91a",   [HasV9_1aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
```
- EN: Defines TableGen record `ARMv9a` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv9a`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 233-250
```tablegen
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCRC,
                                                       FeatureRAS,
                                                       FeatureDotProd]>;
def ARMv92a   : Architecture<"armv9.2-a", "ARMv92a",  [HasV9_2aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCRC,
                                                       FeatureRAS,
                                                       FeatureDotProd]>;
```
- EN: Defines TableGen record `ARMv92a` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv92a`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 251-268
```tablegen
def ARMv93a   : Architecture<"armv9.3-a", "ARMv93a",  [HasV9_3aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCrypto,
                                                       FeatureCRC,
                                                       FeatureRAS,
                                                       FeatureDotProd]>;
def ARMv94a   : Architecture<"armv9.4-a", "ARMv94a",  [HasV9_4aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
```
- EN: Defines TableGen record `ARMv93a` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv93a`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 269-286
```tablegen
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCRC,
                                                       FeatureRAS,
                                                       FeatureDotProd]>;
def ARMv95a   : Architecture<"armv9.5-a", "ARMv95a",  [HasV9_5aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCRC,
                                                       FeatureRAS,
```
- EN: Defines TableGen record `ARMv95a` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv95a`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 287-304
```tablegen
                                                       FeatureDotProd]>;
def ARMv96a   : Architecture<"armv9.6-a", "ARMv96a",  [HasV9_6aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCRC,
                                                       FeatureRAS,
                                                       FeatureDotProd]>;
def ARMv97a   : Architecture<"armv9.7-a", "ARMv97a",  [HasV9_7aOps,
                                                       FeatureAClass,
                                                       FeatureDB,
                                                       FeatureFPARMv8,
                                                       FeatureNEON,
```
- EN: Defines TableGen record `ARMv96a` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv96a`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 305-320
```tablegen
                                                       FeatureDSP,
                                                       FeatureTrustZone,
                                                       FeatureMP,
                                                       FeatureVirtualization,
                                                       FeatureCRC,
                                                       FeatureRAS,
                                                       FeatureDotProd]>;
def ARMv8r    : Architecture<"armv8-r",   "ARMv8r",   [HasV8Ops,
                                                       FeatureRClass,
                                                       FeatureDB,
                                                       FeatureDFB,
                                                       FeatureDSP,
                                                       FeatureCRC,
                                                       FeatureMP,
                                                       FeatureFPARMv8_D16_SP,
                                                       FeatureVirtualization]>;
```
- EN: Defines TableGen record `ARMv8r` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv8r`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 322-332
```tablegen
def ARMv8mBaseline : Architecture<"armv8-m.base", "ARMv8mBaseline",
                                                      [HasV8MBaselineOps,
                                                       FeatureNoARM,
                                                       ModeThumb,
                                                       FeatureDB,
                                                       FeatureHWDivThumb,
                                                       FeatureV7Clrex,
                                                       Feature8MSecExt,
                                                       FeatureAcquireRelease,
                                                       FeatureMClass,
                                                       FeatureStrictAlign]>;
```
- EN: Defines TableGen record `ARMv8mBaseline` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv8mBaseline`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 334-342
```tablegen
def ARMv8mMainline : Architecture<"armv8-m.main", "ARMv8mMainline",
                                                      [HasV8MMainlineOps,
                                                       FeatureNoARM,
                                                       ModeThumb,
                                                       FeatureDB,
                                                       FeatureHWDivThumb,
                                                       Feature8MSecExt,
                                                       FeatureAcquireRelease,
                                                       FeatureMClass]>;
```
- EN: Defines TableGen record `ARMv8mMainline` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv8mMainline`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 344-354
```tablegen
def ARMv81mMainline : Architecture<"armv8.1-m.main", "ARMv81mMainline",
                                                      [HasV8_1MMainlineOps,
                                                       FeatureNoARM,
                                                       ModeThumb,
                                                       FeatureDB,
                                                       FeatureHWDivThumb,
                                                       Feature8MSecExt,
                                                       FeatureAcquireRelease,
                                                       FeatureMClass,
                                                       FeatureRAS,
                                                       FeatureLOB]>;
```
- EN: Defines TableGen record `ARMv81mMainline` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMv81mMainline`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 356-362
```tablegen
// Aliases
def IWMMXT   : Architecture<"iwmmxt",      "ARMv5te",  [ARMv5te]>;
def IWMMXT2  : Architecture<"iwmmxt2",     "ARMv5te",  [ARMv5te]>;
def XScale   : Architecture<"xscale",      "ARMv5te",  [ARMv5te]>;
def ARMv6j   : Architecture<"armv6j",      "ARMv7a",   [ARMv6]>;
def ARMv7k   : Architecture<"armv7k",      "ARMv7a",   [ARMv7a]>;
def ARMv7s   : Architecture<"armv7s",      "ARMv7a",   [ARMv7a]>;
```
- EN: Defines TableGen record `IWMMXT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IWMMXT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

## Key Concepts / 关键概念

- EN: Primary role: architecture/version metadata.
  - CN: 核心职责：体系结构/版本元数据。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
