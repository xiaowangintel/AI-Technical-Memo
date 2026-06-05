# SystemZFeatures.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZFeatures.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```tablegen
   1: //===-- SystemZ.td - SystemZ processors and features ---------*- tblgen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // Feature definitions.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: class SystemZFeature<string extname, string intname, dag featdag, string desc>
  14:   : Predicate<"Subtarget->has"#intname#"()">,
  15:     AssemblerPredicate<featdag, extname>,
  16:     SubtargetFeature<extname, "Has"#intname, "true", desc>;
  17: 
  18: class SystemZMissingFeature<string intname>
  19:   : Predicate<"!Subtarget->has"#intname#"()">;
  20: 
  21: class SystemZFeatureList<list<SystemZFeature> x> {
  22:   list<SystemZFeature> List = x;
  23: }
  24: 
  25: class SystemZFeatureAdd<list<SystemZFeature> x, list<SystemZFeature> y>
  26:   : SystemZFeatureList<!listconcat(x, y)>;
  27: 
  28: // This feature is added as a subtarget feature whenever the function is
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `SystemZFeature`, `SystemZMissingFeature`, `SystemZFeatureList`, `SystemZFeatureAdd`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `SystemZFeature`, `SystemZMissingFeature`, `SystemZFeatureList`, `SystemZFeatureAdd` 等 TableGen 记录。

### Lines 29-56 / 第 29-56 行
```tablegen
  29: // compiled to use soft-float.
  30: def FeatureSoftFloat : SystemZFeature<
  31:   "soft-float", "SoftFloat", (all_of FeatureSoftFloat),
  32:   "Use software emulation for floating point"
  33: >;
  34: 
  35: def FeatureBackChain : SystemZFeature<
  36:   "backchain", "BackChain", (all_of FeatureBackChain),
  37:   "Store the address of the caller's frame into the callee's stack frame"
  38: >;
  39: 
  40: def FeatureUnalignedSymbols : SystemZFeature<
  41:   "unaligned-symbols", "UnalignedSymbols", (all_of FeatureUnalignedSymbols),
  42:   "Don't apply the ABI minimum alignment to external symbols."
  43: >;
  44: 
  45: //===----------------------------------------------------------------------===//
  46: //
  47: // New features added in the Ninth Edition of the z/Architecture
  48: //
  49: //===----------------------------------------------------------------------===//
  50: 
  51: def FeatureDistinctOps : SystemZFeature<
  52:   "distinct-ops", "DistinctOps", (all_of FeatureDistinctOps),
  53:   "Assume that the distinct-operands facility is installed"
  54: >;
  55: 
  56: def FeatureFastSerialization : SystemZFeature<
```
- **EN**: This block declares or refines TableGen records such as `FeatureSoftFloat`, `FeatureBackChain`, `FeatureUnalignedSymbols`, `FeatureDistinctOps`, `FeatureFastSerialization`.
- **CN**: 该代码块声明或细化了 `FeatureSoftFloat`, `FeatureBackChain`, `FeatureUnalignedSymbols`, `FeatureDistinctOps`, `FeatureFastSerialization` 等 TableGen 记录。

### Lines 57-84 / 第 57-84 行
```tablegen
  57:   "fast-serialization", "FastSerialization", (all_of FeatureFastSerialization),
  58:   "Assume that the fast-serialization facility is installed"
  59: >;
  60: 
  61: def FeatureFPExtension : SystemZFeature<
  62:   "fp-extension", "FPExtension", (all_of FeatureFPExtension),
  63:   "Assume that the floating-point extension facility is installed"
  64: >;
  65: 
  66: def FeatureHighWord : SystemZFeature<
  67:   "high-word", "HighWord", (all_of FeatureHighWord),
  68:   "Assume that the high-word facility is installed"
  69: >;
  70: 
  71: def FeatureInterlockedAccess1 : SystemZFeature<
  72:   "interlocked-access1", "InterlockedAccess1", (all_of FeatureInterlockedAccess1),
  73:   "Assume that interlocked-access facility 1 is installed"
  74: >;
  75: def FeatureNoInterlockedAccess1 : SystemZMissingFeature<"InterlockedAccess1">;
  76: 
  77: def FeatureLoadStoreOnCond : SystemZFeature<
  78:   "load-store-on-cond", "LoadStoreOnCond", (all_of FeatureLoadStoreOnCond),
  79:   "Assume that the load/store-on-condition facility is installed"
  80: >;
  81: def FeatureNoLoadStoreOnCond : SystemZMissingFeature<"LoadStoreOnCond">;
  82: 
  83: def FeaturePopulationCount : SystemZFeature<
  84:   "population-count", "PopulationCount", (all_of FeaturePopulationCount),
```
- **EN**: This block declares or refines TableGen records such as `FeatureFPExtension`, `FeatureHighWord`, `FeatureInterlockedAccess1`, `FeatureNoInterlockedAccess1`, `FeatureLoadStoreOnCond`, `FeatureNoLoadStoreOnCond`.
- **CN**: 该代码块声明或细化了 `FeatureFPExtension`, `FeatureHighWord`, `FeatureInterlockedAccess1`, `FeatureNoInterlockedAccess1`, `FeatureLoadStoreOnCond`, `FeatureNoLoadStoreOnCond` 等 TableGen 记录。

### Lines 85-112 / 第 85-112 行
```tablegen
  85:   "Assume that the population-count facility is installed"
  86: >;
  87: 
  88: def FeatureMessageSecurityAssist3 : SystemZFeature<
  89:   "message-security-assist-extension3", "MessageSecurityAssist3", (all_of FeatureMessageSecurityAssist3),
  90:   "Assume that the message-security-assist extension facility 3 is installed"
  91: >;
  92: 
  93: def FeatureMessageSecurityAssist4 : SystemZFeature<
  94:   "message-security-assist-extension4", "MessageSecurityAssist4", (all_of FeatureMessageSecurityAssist4),
  95:   "Assume that the message-security-assist extension facility 4 is installed"
  96: >;
  97: 
  98: def FeatureResetReferenceBitsMultiple : SystemZFeature<
  99:   "reset-reference-bits-multiple", "ResetReferenceBitsMultiple", (all_of FeatureResetReferenceBitsMultiple),
 100:   "Assume that the reset-reference-bits-multiple facility is installed"
 101: >;
 102: 
 103: def Arch9NewFeatures : SystemZFeatureList<[
 104:     FeatureDistinctOps,
 105:     FeatureFastSerialization,
 106:     FeatureFPExtension,
 107:     FeatureHighWord,
 108:     FeatureInterlockedAccess1,
 109:     FeatureLoadStoreOnCond,
 110:     FeaturePopulationCount,
 111:     FeatureMessageSecurityAssist3,
 112:     FeatureMessageSecurityAssist4,
```
- **EN**: This block declares or refines TableGen records such as `FeatureMessageSecurityAssist3`, `FeatureMessageSecurityAssist4`, `FeatureResetReferenceBitsMultiple`, `Arch9NewFeatures`.
- **CN**: 该代码块声明或细化了 `FeatureMessageSecurityAssist3`, `FeatureMessageSecurityAssist4`, `FeatureResetReferenceBitsMultiple`, `Arch9NewFeatures` 等 TableGen 记录。

### Lines 113-140 / 第 113-140 行
```tablegen
 113:     FeatureResetReferenceBitsMultiple
 114: ]>;
 115: 
 116: //===----------------------------------------------------------------------===//
 117: //
 118: // New features added in the Tenth Edition of the z/Architecture
 119: //
 120: //===----------------------------------------------------------------------===//
 121: 
 122: def FeatureExecutionHint : SystemZFeature<
 123:   "execution-hint", "ExecutionHint", (all_of FeatureExecutionHint),
 124:   "Assume that the execution-hint facility is installed"
 125: >;
 126: 
 127: def FeatureLoadAndTrap : SystemZFeature<
 128:   "load-and-trap", "LoadAndTrap", (all_of FeatureLoadAndTrap),
 129:   "Assume that the load-and-trap facility is installed"
 130: >;
 131: 
 132: def FeatureMiscellaneousExtensions : SystemZFeature<
 133:   "miscellaneous-extensions", "MiscellaneousExtensions", (all_of FeatureMiscellaneousExtensions),
 134:   "Assume that the miscellaneous-extensions facility is installed"
 135: >;
 136: 
 137: def FeatureProcessorAssist : SystemZFeature<
 138:   "processor-assist", "ProcessorAssist", (all_of FeatureProcessorAssist),
 139:   "Assume that the processor-assist facility is installed"
 140: >;
```
- **EN**: This block declares or refines TableGen records such as `FeatureExecutionHint`, `FeatureLoadAndTrap`, `FeatureMiscellaneousExtensions`, `FeatureProcessorAssist`.
- **CN**: 该代码块声明或细化了 `FeatureExecutionHint`, `FeatureLoadAndTrap`, `FeatureMiscellaneousExtensions`, `FeatureProcessorAssist` 等 TableGen 记录。

### Lines 141-168 / 第 141-168 行
```tablegen
 141: 
 142: def FeatureTransactionalExecution : SystemZFeature<
 143:   "transactional-execution", "TransactionalExecution", (all_of FeatureTransactionalExecution),
 144:   "Assume that the transactional-execution facility is installed"
 145: >;
 146: 
 147: def FeatureDFPZonedConversion : SystemZFeature<
 148:   "dfp-zoned-conversion", "DFPZonedConversion", (all_of FeatureDFPZonedConversion),
 149:   "Assume that the DFP zoned-conversion facility is installed"
 150: >;
 151: 
 152: def FeatureEnhancedDAT2 : SystemZFeature<
 153:   "enhanced-dat-2", "EnhancedDAT2", (all_of FeatureEnhancedDAT2),
 154:   "Assume that the enhanced-DAT facility 2 is installed"
 155: >;
 156: 
 157: def Arch10NewFeatures : SystemZFeatureList<[
 158:     FeatureExecutionHint,
 159:     FeatureLoadAndTrap,
 160:     FeatureMiscellaneousExtensions,
 161:     FeatureProcessorAssist,
 162:     FeatureTransactionalExecution,
 163:     FeatureDFPZonedConversion,
 164:     FeatureEnhancedDAT2
 165: ]>;
 166: 
 167: //===----------------------------------------------------------------------===//
 168: //
```
- **EN**: This block declares or refines TableGen records such as `FeatureTransactionalExecution`, `FeatureDFPZonedConversion`, `FeatureEnhancedDAT2`, `Arch10NewFeatures`.
- **CN**: 该代码块声明或细化了 `FeatureTransactionalExecution`, `FeatureDFPZonedConversion`, `FeatureEnhancedDAT2`, `Arch10NewFeatures` 等 TableGen 记录。

### Lines 169-196 / 第 169-196 行
```tablegen
 169: // New features added in the Eleventh Edition of the z/Architecture
 170: //
 171: //===----------------------------------------------------------------------===//
 172: 
 173: def FeatureLoadAndZeroRightmostByte : SystemZFeature<
 174:   "load-and-zero-rightmost-byte", "LoadAndZeroRightmostByte", (all_of FeatureLoadAndZeroRightmostByte),
 175:   "Assume that the load-and-zero-rightmost-byte facility is installed"
 176: >;
 177: 
 178: def FeatureLoadStoreOnCond2 : SystemZFeature<
 179:   "load-store-on-cond-2", "LoadStoreOnCond2", (all_of FeatureLoadStoreOnCond2),
 180:   "Assume that the load/store-on-condition facility 2 is installed"
 181: >;
 182: 
 183: def FeatureMessageSecurityAssist5 : SystemZFeature<
 184:   "message-security-assist-extension5", "MessageSecurityAssist5", (all_of FeatureMessageSecurityAssist5),
 185:   "Assume that the message-security-assist extension facility 5 is installed"
 186: >;
 187: 
 188: def FeatureDFPPackedConversion : SystemZFeature<
 189:   "dfp-packed-conversion", "DFPPackedConversion", (all_of FeatureDFPPackedConversion),
 190:   "Assume that the DFP packed-conversion facility is installed"
 191: >;
 192: 
 193: def FeatureVector : SystemZFeature<
 194:   "vector", "Vector", (all_of FeatureVector),
 195:   "Assume that the vectory facility is installed"
 196: >;
```
- **EN**: This block declares or refines TableGen records such as `FeatureLoadAndZeroRightmostByte`, `FeatureLoadStoreOnCond2`, `FeatureMessageSecurityAssist5`, `FeatureDFPPackedConversion`, `FeatureVector`.
- **CN**: 该代码块声明或细化了 `FeatureLoadAndZeroRightmostByte`, `FeatureLoadStoreOnCond2`, `FeatureMessageSecurityAssist5`, `FeatureDFPPackedConversion`, `FeatureVector` 等 TableGen 记录。

### Lines 197-224 / 第 197-224 行
```tablegen
 197: def FeatureNoVector : SystemZMissingFeature<"Vector">;
 198: 
 199: def NoVecHwMode : HwMode<[FeatureNoVector]>;
 200: 
 201: def Arch11NewFeatures : SystemZFeatureList<[
 202:     FeatureLoadAndZeroRightmostByte,
 203:     FeatureLoadStoreOnCond2,
 204:     FeatureMessageSecurityAssist5,
 205:     FeatureDFPPackedConversion,
 206:     FeatureVector
 207: ]>;
 208: 
 209: //===----------------------------------------------------------------------===//
 210: //
 211: // New features added in the Twelfth Edition of the z/Architecture
 212: //
 213: //===----------------------------------------------------------------------===//
 214: 
 215: def FeatureMiscellaneousExtensions2 : SystemZFeature<
 216:   "miscellaneous-extensions-2", "MiscellaneousExtensions2", (all_of FeatureMiscellaneousExtensions2),
 217:   "Assume that the miscellaneous-extensions facility 2 is installed"
 218: >;
 219: 
 220: def FeatureGuardedStorage : SystemZFeature<
 221:   "guarded-storage", "GuardedStorage", (all_of FeatureGuardedStorage),
 222:   "Assume that the guarded-storage facility is installed"
 223: >;
 224: 
```
- **EN**: This block declares or refines TableGen records such as `FeatureNoVector`, `NoVecHwMode`, `Arch11NewFeatures`, `FeatureMiscellaneousExtensions2`, `FeatureGuardedStorage`.
- **CN**: 该代码块声明或细化了 `FeatureNoVector`, `NoVecHwMode`, `Arch11NewFeatures`, `FeatureMiscellaneousExtensions2`, `FeatureGuardedStorage` 等 TableGen 记录。

### Lines 225-252 / 第 225-252 行
```tablegen
 225: def FeatureMessageSecurityAssist7 : SystemZFeature<
 226:   "message-security-assist-extension7", "MessageSecurityAssist7", (all_of FeatureMessageSecurityAssist7),
 227:   "Assume that the message-security-assist extension facility 7 is installed"
 228: >;
 229: 
 230: def FeatureMessageSecurityAssist8 : SystemZFeature<
 231:   "message-security-assist-extension8", "MessageSecurityAssist8", (all_of FeatureMessageSecurityAssist8),
 232:   "Assume that the message-security-assist extension facility 8 is installed"
 233: >;
 234: 
 235: def FeatureVectorEnhancements1 : SystemZFeature<
 236:   "vector-enhancements-1", "VectorEnhancements1", (all_of FeatureVectorEnhancements1),
 237:   "Assume that the vector enhancements facility 1 is installed"
 238: >;
 239: def FeatureNoVectorEnhancements1 : SystemZMissingFeature<"VectorEnhancements1">;
 240: 
 241: def FeatureVectorPackedDecimal : SystemZFeature<
 242:   "vector-packed-decimal", "VectorPackedDecimal", (all_of FeatureVectorPackedDecimal),
 243:   "Assume that the vector packed decimal facility is installed"
 244: >;
 245: 
 246: def FeatureInsertReferenceBitsMultiple : SystemZFeature<
 247:   "insert-reference-bits-multiple", "InsertReferenceBitsMultiple", (all_of FeatureInsertReferenceBitsMultiple),
 248:   "Assume that the insert-reference-bits-multiple facility is installed"
 249: >;
 250: 
 251: def FeatureTestPendingExternalInterruption : SystemZFeature<
 252:   "test-pending-external-interruption", "TestPendingExternalInterruption", (all_of FeatureTestPendingExternalInterruption),
```
- **EN**: This block declares or refines TableGen records such as `FeatureMessageSecurityAssist7`, `FeatureMessageSecurityAssist8`, `FeatureVectorEnhancements1`, `FeatureNoVectorEnhancements1`, `FeatureVectorPackedDecimal`, `FeatureInsertReferenceBitsMultiple`.
- **CN**: 该代码块声明或细化了 `FeatureMessageSecurityAssist7`, `FeatureMessageSecurityAssist8`, `FeatureVectorEnhancements1`, `FeatureNoVectorEnhancements1`, `FeatureVectorPackedDecimal`, `FeatureInsertReferenceBitsMultiple` 等 TableGen 记录。

### Lines 253-280 / 第 253-280 行
```tablegen
 253:   "Assume that the test-pending-external-interruption facility is installed"
 254: >;
 255: 
 256: def Arch12NewFeatures : SystemZFeatureList<[
 257:     FeatureMiscellaneousExtensions2,
 258:     FeatureGuardedStorage,
 259:     FeatureMessageSecurityAssist7,
 260:     FeatureMessageSecurityAssist8,
 261:     FeatureVectorEnhancements1,
 262:     FeatureVectorPackedDecimal,
 263:     FeatureInsertReferenceBitsMultiple,
 264:     FeatureTestPendingExternalInterruption
 265: ]>;
 266: 
 267: //===----------------------------------------------------------------------===//
 268: //
 269: // New features added in the Thirteenth Edition of the z/Architecture
 270: //
 271: //===----------------------------------------------------------------------===//
 272: 
 273: def FeatureMiscellaneousExtensions3 : SystemZFeature<
 274:   "miscellaneous-extensions-3", "MiscellaneousExtensions3", (all_of FeatureMiscellaneousExtensions3),
 275:   "Assume that the miscellaneous-extensions facility 3 is installed"
 276: >;
 277: 
 278: def FeatureMessageSecurityAssist9 : SystemZFeature<
 279:   "message-security-assist-extension9", "MessageSecurityAssist9", (all_of FeatureMessageSecurityAssist9),
 280:   "Assume that the message-security-assist extension facility 9 is installed"
```
- **EN**: This block declares or refines TableGen records such as `Arch12NewFeatures`, `FeatureMiscellaneousExtensions3`, `FeatureMessageSecurityAssist9`.
- **CN**: 该代码块声明或细化了 `Arch12NewFeatures`, `FeatureMiscellaneousExtensions3`, `FeatureMessageSecurityAssist9` 等 TableGen 记录。

### Lines 281-308 / 第 281-308 行
```tablegen
 281: >;
 282: 
 283: def FeatureVectorEnhancements2 : SystemZFeature<
 284:   "vector-enhancements-2", "VectorEnhancements2", (all_of FeatureVectorEnhancements2),
 285:   "Assume that the vector enhancements facility 2 is installed"
 286: >;
 287: 
 288: def FeatureVectorPackedDecimalEnhancement : SystemZFeature<
 289:   "vector-packed-decimal-enhancement", "VectorPackedDecimalEnhancement", (all_of FeatureVectorPackedDecimalEnhancement),
 290:   "Assume that the vector packed decimal enhancement facility is installed"
 291: >;
 292: 
 293: def FeatureEnhancedSort : SystemZFeature<
 294:   "enhanced-sort", "EnhancedSort", (all_of FeatureEnhancedSort),
 295:   "Assume that the enhanced-sort facility is installed"
 296: >;
 297: 
 298: def FeatureDeflateConversion : SystemZFeature<
 299:   "deflate-conversion", "DeflateConversion", (all_of FeatureDeflateConversion),
 300:   "Assume that the deflate-conversion facility is installed"
 301: >;
 302: 
 303: def Arch13NewFeatures : SystemZFeatureList<[
 304:     FeatureMiscellaneousExtensions3,
 305:     FeatureMessageSecurityAssist9,
 306:     FeatureVectorEnhancements2,
 307:     FeatureVectorPackedDecimalEnhancement,
 308:     FeatureEnhancedSort,
```
- **EN**: This block declares or refines TableGen records such as `FeatureVectorEnhancements2`, `FeatureVectorPackedDecimalEnhancement`, `FeatureEnhancedSort`, `FeatureDeflateConversion`, `Arch13NewFeatures`.
- **CN**: 该代码块声明或细化了 `FeatureVectorEnhancements2`, `FeatureVectorPackedDecimalEnhancement`, `FeatureEnhancedSort`, `FeatureDeflateConversion`, `Arch13NewFeatures` 等 TableGen 记录。

### Lines 309-336 / 第 309-336 行
```tablegen
 309:     FeatureDeflateConversion
 310: ]>;
 311: 
 312: //===----------------------------------------------------------------------===//
 313: //
 314: // New features added in the Fourteenth Edition of the z/Architecture
 315: //
 316: //===----------------------------------------------------------------------===//
 317: 
 318: def FeatureVectorPackedDecimalEnhancement2 : SystemZFeature<
 319:   "vector-packed-decimal-enhancement-2", "VectorPackedDecimalEnhancement2", (all_of FeatureVectorPackedDecimalEnhancement2),
 320:   "Assume that the vector packed decimal enhancement facility 2 is installed"
 321: >;
 322: 
 323: def FeatureNNPAssist : SystemZFeature<
 324:   "nnp-assist", "NNPAssist", (all_of FeatureNNPAssist),
 325:   "Assume that the NNP-assist facility is installed"
 326: >;
 327: 
 328: def FeatureBEAREnhancement : SystemZFeature<
 329:   "bear-enhancement", "BEAREnhancement", (all_of FeatureBEAREnhancement),
 330:   "Assume that the BEAR-enhancement facility is installed"
 331: >;
 332: 
 333: def FeatureResetDATProtection : SystemZFeature<
 334:   "reset-dat-protection", "ResetDATProtection", (all_of FeatureResetDATProtection),
 335:   "Assume that the reset-DAT-protection facility is installed"
 336: >;
```
- **EN**: This block declares or refines TableGen records such as `FeatureVectorPackedDecimalEnhancement2`, `FeatureNNPAssist`, `FeatureBEAREnhancement`, `FeatureResetDATProtection`.
- **CN**: 该代码块声明或细化了 `FeatureVectorPackedDecimalEnhancement2`, `FeatureNNPAssist`, `FeatureBEAREnhancement`, `FeatureResetDATProtection` 等 TableGen 记录。

### Lines 337-364 / 第 337-364 行
```tablegen
 337: 
 338: def FeatureProcessorActivityInstrumentation : SystemZFeature<
 339:   "processor-activity-instrumentation", "ProcessorActivityInstrumentation", (all_of FeatureProcessorActivityInstrumentation),
 340:   "Assume that the processor-activity-instrumentation facility is installed"
 341: >;
 342: 
 343: def Arch14NewFeatures : SystemZFeatureList<[
 344:     FeatureVectorPackedDecimalEnhancement2,
 345:     FeatureNNPAssist,
 346:     FeatureBEAREnhancement,
 347:     FeatureResetDATProtection,
 348:     FeatureProcessorActivityInstrumentation
 349: ]>;
 350: 
 351: //===----------------------------------------------------------------------===//
 352: //
 353: // New features added in the Fifteenth Edition of the z/Architecture
 354: //
 355: //===----------------------------------------------------------------------===//
 356: 
 357: def FeatureMiscellaneousExtensions4 : SystemZFeature<
 358:   "miscellaneous-extensions-4", "MiscellaneousExtensions4", (all_of FeatureMiscellaneousExtensions4),
 359:   "Assume that the miscellaneous-extensions facility 4 is installed"
 360: >;
 361: 
 362: def FeatureVectorEnhancements3 : SystemZFeature<
 363:   "vector-enhancements-3", "VectorEnhancements3", (all_of FeatureVectorEnhancements3),
 364:   "Assume that the vector enhancements facility 3 is installed"
```
- **EN**: This block declares or refines TableGen records such as `FeatureProcessorActivityInstrumentation`, `Arch14NewFeatures`, `FeatureMiscellaneousExtensions4`, `FeatureVectorEnhancements3`.
- **CN**: 该代码块声明或细化了 `FeatureProcessorActivityInstrumentation`, `Arch14NewFeatures`, `FeatureMiscellaneousExtensions4`, `FeatureVectorEnhancements3` 等 TableGen 记录。

### Lines 365-392 / 第 365-392 行
```tablegen
 365: >;
 366: 
 367: def FeatureVectorPackedDecimalEnhancement3 : SystemZFeature<
 368:   "vector-packed-decimal-enhancement-3", "VectorPackedDecimalEnhancement3", (all_of FeatureVectorPackedDecimalEnhancement3),
 369:   "Assume that the vector packed decimal enhancement facility 3 is installed"
 370: >;
 371: 
 372: def FeatureMessageSecurityAssist12 : SystemZFeature<
 373:   "message-security-assist-extension12", "MessageSecurityAssist12", (all_of FeatureMessageSecurityAssist12),
 374:   "Assume that the message-security-assist extension facility 12 is installed"
 375: >;
 376: 
 377: def FeatureConcurrentFunctions : SystemZFeature<
 378:   "concurrent-functions", "ConcurrentFunctions", (all_of FeatureConcurrentFunctions),
 379:   "Assume that the concurrent-functions facility is installed"
 380: >;
 381: 
 382: def Arch15NewFeatures : SystemZFeatureList<[
 383:     FeatureMiscellaneousExtensions4,
 384:     FeatureVectorEnhancements3,
 385:     FeatureVectorPackedDecimalEnhancement3,
 386:     FeatureMessageSecurityAssist12,
 387:     FeatureConcurrentFunctions
 388: ]>;
 389: 
 390: //===----------------------------------------------------------------------===//
 391: //
 392: // Cumulative supported and unsupported feature sets
```
- **EN**: This block declares or refines TableGen records such as `FeatureVectorPackedDecimalEnhancement3`, `FeatureMessageSecurityAssist12`, `FeatureConcurrentFunctions`, `Arch15NewFeatures`.
- **CN**: 该代码块声明或细化了 `FeatureVectorPackedDecimalEnhancement3`, `FeatureMessageSecurityAssist12`, `FeatureConcurrentFunctions`, `Arch15NewFeatures` 等 TableGen 记录。

### Lines 393-420 / 第 393-420 行
```tablegen
 393: //
 394: //===----------------------------------------------------------------------===//
 395: 
 396: def Arch8SupportedFeatures
 397:   : SystemZFeatureList<[]>;
 398: def Arch9SupportedFeatures
 399:   : SystemZFeatureAdd<Arch8SupportedFeatures.List,  Arch9NewFeatures.List>;
 400: def Arch10SupportedFeatures
 401:   : SystemZFeatureAdd<Arch9SupportedFeatures.List,  Arch10NewFeatures.List>;
 402: def Arch11SupportedFeatures
 403:   : SystemZFeatureAdd<Arch10SupportedFeatures.List, Arch11NewFeatures.List>;
 404: def Arch12SupportedFeatures
 405:   : SystemZFeatureAdd<Arch11SupportedFeatures.List, Arch12NewFeatures.List>;
 406: def Arch13SupportedFeatures
 407:   : SystemZFeatureAdd<Arch12SupportedFeatures.List, Arch13NewFeatures.List>;
 408: def Arch14SupportedFeatures
 409:   : SystemZFeatureAdd<Arch13SupportedFeatures.List, Arch14NewFeatures.List>;
 410: def Arch15SupportedFeatures
 411:   : SystemZFeatureAdd<Arch14SupportedFeatures.List, Arch15NewFeatures.List>;
 412: 
 413: def Arch15UnsupportedFeatures
 414:   : SystemZFeatureList<[]>;
 415: def Arch14UnsupportedFeatures
 416:   : SystemZFeatureAdd<Arch15UnsupportedFeatures.List, Arch15NewFeatures.List>;
 417: def Arch13UnsupportedFeatures
 418:   : SystemZFeatureAdd<Arch14UnsupportedFeatures.List, Arch14NewFeatures.List>;
 419: def Arch12UnsupportedFeatures
 420:   : SystemZFeatureAdd<Arch13UnsupportedFeatures.List, Arch13NewFeatures.List>;
```
- **EN**: This block declares or refines TableGen records such as `Arch8SupportedFeatures`, `Arch9SupportedFeatures`, `Arch10SupportedFeatures`, `Arch11SupportedFeatures`, `Arch12SupportedFeatures`, `Arch13SupportedFeatures`.
- **CN**: 该代码块声明或细化了 `Arch8SupportedFeatures`, `Arch9SupportedFeatures`, `Arch10SupportedFeatures`, `Arch11SupportedFeatures`, `Arch12SupportedFeatures`, `Arch13SupportedFeatures` 等 TableGen 记录。

### Lines 421-428 / 第 421-428 行
```tablegen
 421: def Arch11UnsupportedFeatures
 422:   : SystemZFeatureAdd<Arch12UnsupportedFeatures.List, Arch12NewFeatures.List>;
 423: def Arch10UnsupportedFeatures
 424:   : SystemZFeatureAdd<Arch11UnsupportedFeatures.List, Arch11NewFeatures.List>;
 425: def Arch9UnsupportedFeatures
 426:   : SystemZFeatureAdd<Arch10UnsupportedFeatures.List, Arch10NewFeatures.List>;
 427: def Arch8UnsupportedFeatures
 428:   : SystemZFeatureAdd<Arch9UnsupportedFeatures.List,  Arch9NewFeatures.List>;
```
- **EN**: This block declares or refines TableGen records such as `Arch11UnsupportedFeatures`, `Arch10UnsupportedFeatures`, `Arch9UnsupportedFeatures`, `Arch8UnsupportedFeatures`.
- **CN**: 该代码块声明或细化了 `Arch11UnsupportedFeatures`, `Arch10UnsupportedFeatures`, `Arch9UnsupportedFeatures`, `Arch8UnsupportedFeatures` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
