# XtensaFeatures.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaFeatures.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines backend metadata in TableGen DSL that is later expanded into generated tables and matcher code.
  - **CN**: 使用 TableGen DSL 定义后端元数据，随后会展开为生成的表和匹配代码。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```tablegen
//===----------------------------------------------------------------------===//
// Xtensa subtarget features.
//===----------------------------------------------------------------------===//

// Xtensa ISA extensions (Xtensa Options).
def FeatureDensity : SubtargetFeature<"density", "HasDensity", "true",
                                      "Enable Density instructions">;
def HasDensity : Predicate<"Subtarget->hasDensity()">,
                 AssemblerPredicate<(all_of FeatureDensity)>;

```
- **EN**: Declares TableGen records such as `FeatureDensity`, `HasDensity` for the backend description.
- **CN**: 为后端描述声明了 `FeatureDensity`, `HasDensity` 等 TableGen 记录。

### Lines 11-20
```tablegen
def FeatureSingleFloat : SubtargetFeature<"fp", "HasSingleFloat", "true",
                                          "Enable Xtensa Single FP instructions">;
def HasSingleFloat : Predicate<"Subtarget->hasSingleFloat()">,
                     AssemblerPredicate<(all_of FeatureSingleFloat)>;

def FeatureWindowed : SubtargetFeature<"windowed", "HasWindowed", "true",
                                       "Enable Xtensa Windowed Register option">;
def HasWindowed : Predicate<"Subtarget->hasWindowed()">,
                  AssemblerPredicate<(all_of FeatureWindowed)>;

```
- **EN**: Declares TableGen records such as `FeatureSingleFloat`, `HasSingleFloat`, `FeatureWindowed`, `HasWindowed` for the backend description.
- **CN**: 为后端描述声明了 `FeatureSingleFloat`, `HasSingleFloat`, `FeatureWindowed`, `HasWindowed` 等 TableGen 记录。

### Lines 21-30
```tablegen
def FeatureMAC16 : SubtargetFeature<"mac16", "HasMAC16", "true",
                                    "Enable Xtensa MAC16 instructions">;
def HasMAC16 : Predicate<"Subtarget->hasMAC16()">,
               AssemblerPredicate<(all_of FeatureMAC16)>;

def FeatureBoolean          : SubtargetFeature<"bool", "HasBoolean", "true",
                                               "Enable Xtensa Boolean extension">;
def HasBoolean              : Predicate<"Subtarget->hasBoolean()">,
                                         AssemblerPredicate<(all_of FeatureBoolean)>;

```
- **EN**: Declares TableGen records such as `FeatureMAC16`, `HasMAC16`, `FeatureBoolean`, `HasBoolean` for the backend description.
- **CN**: 为后端描述声明了 `FeatureMAC16`, `HasMAC16`, `FeatureBoolean`, `HasBoolean` 等 TableGen 记录。

### Lines 31-40
```tablegen
def FeatureLoop : SubtargetFeature<"loop", "HasLoop", "true",
                                   "Enable Xtensa Loop extension">;
def HasLoop : Predicate<"Subtarget->hasLoop()">,
              AssemblerPredicate<(all_of FeatureLoop)>;

def FeatureSEXT : SubtargetFeature<"sext", "HasSEXT", "true",
                                   "Enable Xtensa Sign Extend option">;
def HasSEXT : Predicate<"Subtarget->hasSEXT()">,
              AssemblerPredicate<(all_of FeatureSEXT)>;

```
- **EN**: Declares TableGen records such as `FeatureLoop`, `HasLoop`, `FeatureSEXT`, `HasSEXT` for the backend description.
- **CN**: 为后端描述声明了 `FeatureLoop`, `HasLoop`, `FeatureSEXT`, `HasSEXT` 等 TableGen 记录。

### Lines 41-50
```tablegen
def FeatureCLAMPS : SubtargetFeature<"clamps", "HasCLAMPS", "true",
                                     "Enable Xtensa CLAMPS option">;
def HasCLAMPS : Predicate<"Subtarget->hasCLAMPS()">,
                AssemblerPredicate<(all_of FeatureCLAMPS)>;

def FeatureNSA : SubtargetFeature<"nsa", "HasNSA", "true",
                                  "Enable Xtensa NSA option">;
def HasNSA : Predicate<"Subtarget->hasNSA()">,
             AssemblerPredicate<(all_of FeatureNSA)>;

```
- **EN**: Declares TableGen records such as `FeatureCLAMPS`, `HasCLAMPS`, `FeatureNSA`, `HasNSA` for the backend description.
- **CN**: 为后端描述声明了 `FeatureCLAMPS`, `HasCLAMPS`, `FeatureNSA`, `HasNSA` 等 TableGen 记录。

### Lines 51-60
```tablegen
def FeatureMINMAX : SubtargetFeature<"minmax", "HasMINMAX", "true",
                                     "Enable Xtensa MINMAX option">;
def HasMINMAX : Predicate<"Subtarget->hasMINMAX()">,
                AssemblerPredicate<(all_of FeatureMINMAX)>;

def FeatureMul16 : SubtargetFeature<"mul16", "HasMul16", "true",
                                    "Enable Xtensa Mul16 option">;
def HasMul16 : Predicate<"Subtarget->hasMul16()">,
               AssemblerPredicate<(all_of FeatureMul16)>;

```
- **EN**: Declares TableGen records such as `FeatureMINMAX`, `HasMINMAX`, `FeatureMul16`, `HasMul16` for the backend description.
- **CN**: 为后端描述声明了 `FeatureMINMAX`, `HasMINMAX`, `FeatureMul16`, `HasMul16` 等 TableGen 记录。

### Lines 61-70
```tablegen
def FeatureMul32 : SubtargetFeature<"mul32", "HasMul32", "true",
                                    "Enable Xtensa Mul32 option">;
def HasMul32 : Predicate<"Subtarget->hasMul32()">,
               AssemblerPredicate<(all_of FeatureMul32)>;

def FeatureMul32High : SubtargetFeature<"mul32high", "HasMul32High", "true",
                                        "Enable Xtensa Mul32High option">;
def HasMul32High : Predicate<"Subtarget->hasMul32High()">,
                   AssemblerPredicate<(all_of FeatureMul32High)>;

```
- **EN**: Declares TableGen records such as `FeatureMul32`, `HasMul32`, `FeatureMul32High`, `HasMul32High` for the backend description.
- **CN**: 为后端描述声明了 `FeatureMul32`, `HasMul32`, `FeatureMul32High`, `HasMul32High` 等 TableGen 记录。

### Lines 71-80
```tablegen
def FeatureDiv32 : SubtargetFeature<"div32", "HasDiv32", "true",
                                    "Enable Xtensa Div32 option">;
def HasDiv32 : Predicate<"Subtarget->hasDiv32()">,
               AssemblerPredicate<(all_of FeatureDiv32)>;

def FeatureS32C1I : SubtargetFeature<"s32c1i", "HasS32C1I", "true",
                                     "Enable Xtensa S32C1I option">;
def HasS32C1I : Predicate<"Subtarget->hasS32C1I()">,
                AssemblerPredicate<(all_of FeatureS32C1I)>;

```
- **EN**: Declares TableGen records such as `FeatureDiv32`, `HasDiv32`, `FeatureS32C1I`, `HasS32C1I` for the backend description.
- **CN**: 为后端描述声明了 `FeatureDiv32`, `HasDiv32`, `FeatureS32C1I`, `HasS32C1I` 等 TableGen 记录。

### Lines 81-91
```tablegen
// Assume that lock-free native-width atomics are available, even if the target
// and operating system combination would not usually provide them. The user
// is responsible for providing any necessary __sync implementations. Code
// built with this feature is not ABI-compatible with code built without this
// feature, if atomic variables are exposed across the ABI boundary.
def FeatureForcedAtomics    : SubtargetFeature<"forced-atomics", "HasForcedAtomics", "true",
                                               "Assume that lock-free native-width atomics are available">;
def HasForcedAtomics        : Predicate<"Subtarget->hasForcedAtomics()">,
                                         AssemblerPredicate<(all_of FeatureForcedAtomics)>;
def HasAtomicLdSt           : Predicate<"Subtarget->hasS32C1I() || Subtarget->hasForcedAtomics()">;

```
- **EN**: Declares TableGen records such as `FeatureForcedAtomics`, `HasForcedAtomics`, `HasAtomicLdSt` for the backend description.
- **CN**: 为后端描述声明了 `FeatureForcedAtomics`, `HasForcedAtomics`, `HasAtomicLdSt` 等 TableGen 记录。

### Lines 92-101
```tablegen
def FeatureRegionProtection : SubtargetFeature<"regprotect", "HasRegionProtection", "true",
                                               "Enable Xtensa Region Protection option">;
def HasRegionProtection : Predicate<"Subtarget->hasRegionProtection()">,
                          AssemblerPredicate<(all_of FeatureRegionProtection)>;

def FeatureRelocatableVector : SubtargetFeature<"rvector", "HasRelocatableVector", "true",
                                                "Enable Xtensa Relocatable Vector option">;
def HasRelocatableVector : Predicate<"Subtarget->hasRelocatableVector()">,
                           AssemblerPredicate<(all_of FeatureRelocatableVector)>;

```
- **EN**: Declares TableGen records such as `FeatureRegionProtection`, `HasRegionProtection`, `FeatureRelocatableVector`, `HasRelocatableVector` for the backend description.
- **CN**: 为后端描述声明了 `FeatureRegionProtection`, `HasRegionProtection`, `FeatureRelocatableVector`, `HasRelocatableVector` 等 TableGen 记录。

### Lines 102-111
```tablegen
def FeatureMiscSR : SubtargetFeature<"miscsr", "HasMiscSR", "true",
                                     "Enable Xtensa Miscellaneous SR option">;
def HasMiscSR : Predicate<"Subtarget->hasMiscSR()">,
                AssemblerPredicate<(all_of FeatureMiscSR)>;

def FeatureExtendedL32R : SubtargetFeature<"extendedl32r", "HasExtendedL32R", "true",
                                           "Enable Xtensa Extended L32R option">;
def HasExtendedL32R : Predicate<"Subtarget->hasExtendedL32R()">,
                      AssemblerPredicate<(all_of FeatureExtendedL32R)>;

```
- **EN**: Declares TableGen records such as `FeatureMiscSR`, `HasMiscSR`, `FeatureExtendedL32R`, `HasExtendedL32R` for the backend description.
- **CN**: 为后端描述声明了 `FeatureMiscSR`, `HasMiscSR`, `FeatureExtendedL32R`, `HasExtendedL32R` 等 TableGen 记录。

### Lines 112-121
```tablegen
def FeatureDataCache : SubtargetFeature<"dcache", "HasDataCache", "true",
                                        "Enable Xtensa Data Cache option">;
def HasDataCache : Predicate<"Subtarget->hasDataCache()">,
                   AssemblerPredicate<(all_of FeatureDataCache)>;

def FeatureTHREADPTR : SubtargetFeature<"threadptr", "HasTHREADPTR", "true",
                                        "Enable Xtensa THREADPTR option">;
def HasTHREADPTR : Predicate<"Subtarget->hasTHREADPTR()">,
                   AssemblerPredicate<(all_of FeatureTHREADPTR)>;

```
- **EN**: Declares TableGen records such as `FeatureDataCache`, `HasDataCache`, `FeatureTHREADPTR`, `HasTHREADPTR` for the backend description.
- **CN**: 为后端描述声明了 `FeatureDataCache`, `HasDataCache`, `FeatureTHREADPTR`, `HasTHREADPTR` 等 TableGen 记录。

### Lines 122-128
```tablegen
// Xtensa Interrupts Options.
def FeatureHighPriInterrupts : SubtargetFeature<"highpriinterrupts",
                                                "HasHighPriInterrupts", "true",
                                                "Enable Xtensa HighPriInterrupts option">;
def HasHighPriInterrupts : Predicate<"Subtarget->hasHighPriInterrupts()">,
                                      AssemblerPredicate<(all_of FeatureHighPriInterrupts)>;

```
- **EN**: Declares TableGen records such as `FeatureHighPriInterrupts`, `HasHighPriInterrupts` for the backend description.
- **CN**: 为后端描述声明了 `FeatureHighPriInterrupts`, `HasHighPriInterrupts` 等 TableGen 记录。

### Lines 129-137
```tablegen
foreach i = {3-7} in
    def FeatureHighPriInterruptsLevel#i : SubtargetFeature<"highpriinterrupts-level"#i,
         "HasHighPriInterruptsLevel"#i#"", "true", "Enable Xtensa HighPriInterrupts Level"#i, [FeatureHighPriInterrupts]>;

def FeatureInterrupt : SubtargetFeature<"interrupt", "HasInterrupt", "true",
                                        "Enable Xtensa Interrupt option">;
def HasInterrupt : Predicate<"Subtarget->hasInterrupt()">,
                              AssemblerPredicate<(all_of FeatureInterrupt)>;

```
- **EN**: Declares TableGen records such as `FeatureHighPriInterruptsLevel`, `FeatureInterrupt`, `HasInterrupt` for the backend description.
- **CN**: 为后端描述声明了 `FeatureHighPriInterruptsLevel`, `FeatureInterrupt`, `HasInterrupt` 等 TableGen 记录。

### Lines 138-147
```tablegen
def FeatureException : SubtargetFeature<"exception", "HasException", "true",
                                        "Enable Xtensa Exception option">;
def HasException : Predicate<"Subtarget->hasException()">,
                              AssemblerPredicate<(all_of FeatureException)>;

def FeatureDebug : SubtargetFeature<"debug", "HasDebug", "true",
                                    "Enable Xtensa Debug option">;
def HasDebug : Predicate<"Subtarget->hasDebug()">,
                          AssemblerPredicate<(all_of FeatureDebug)>;

```
- **EN**: Declares TableGen records such as `FeatureException`, `HasException`, `FeatureDebug`, `HasDebug` for the backend description.
- **CN**: 为后端描述声明了 `FeatureException`, `HasException`, `FeatureDebug`, `HasDebug` 等 TableGen 记录。

### Lines 148-156
```tablegen
foreach i = {1-3} in
    def FeatureTimers#i : SubtargetFeature<"timers"#i,
         "HasTimers"#i#"", "true", "Enable Xtensa Timers "#i>;

def FeaturePRID : SubtargetFeature<"prid", "HasPRID", "true",
                                   "Enable Xtensa Processor ID option">;
def HasPRID : Predicate<"Subtarget->hasPRID()">,
                         AssemblerPredicate<(all_of FeaturePRID)>;

```
- **EN**: Declares TableGen records such as `FeatureTimers`, `FeaturePRID`, `HasPRID` for the backend description.
- **CN**: 为后端描述声明了 `FeatureTimers`, `FeaturePRID`, `HasPRID` 等 TableGen 记录。

### Lines 157-165
```tablegen
def FeatureCoprocessor : SubtargetFeature<"coprocessor", "HasCoprocessor", "true",
                                          "Enable Xtensa Coprocessor option">;
def HasCoprocessor : Predicate<"Subtarget->hasCoprocessor()">,
                                AssemblerPredicate<(all_of FeatureCoprocessor)>;

def FeatureDFPAccel : SubtargetFeature<"dfpaccel", "HasDFPAccel", "true",
                                       "Enable Xtensa Double Precision FP acceleration">;
def HasDFPAccel : Predicate<"Subtarget->hasDFPAccel()">,
                  AssemblerPredicate<(all_of FeatureDFPAccel)>;
```
- **EN**: Declares TableGen records such as `FeatureCoprocessor`, `HasCoprocessor`, `FeatureDFPAccel`, `HasDFPAccel` for the backend description.
- **CN**: 为后端描述声明了 `FeatureCoprocessor`, `HasCoprocessor`, `FeatureDFPAccel`, `HasDFPAccel` 等 TableGen 记录。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
