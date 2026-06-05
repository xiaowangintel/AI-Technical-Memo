# Hexagon.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/Hexagon.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Hexagon.td - Describe the Hexagon Target Machine --*- tablegen
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```tablegen
     1: //===-- Hexagon.td - Describe the Hexagon Target Machine --*- tablegen -*--===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This is the top level entry point for the Hexagon target.
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: //===----------------------------------------------------------------------===//
    14: // Target-independent interfaces which we are implementing
    15: //===----------------------------------------------------------------------===//
    16: 
    17: include "llvm/Target/Target.td"
    18: 
    19: //===----------------------------------------------------------------------===//
    20: // Hexagon Subtarget features.
    21: //===----------------------------------------------------------------------===//
    22: 
    23: // Hexagon Architectures
    24: include "HexagonDepArch.td"
    25: 
    26: def ProcTinyCore: SubtargetFeature<"tinycore", "HexagonProcFamily",
    27:       "TinyCore", "Hexagon Tiny Core">;
    28: 
    29: // Hexagon ISA Extensions
    30: def ExtensionZReg: SubtargetFeature<"zreg", "UseZRegOps", "true",
    31:       "Hexagon ZReg extension instructions">;
    32: def ExtensionHVXQFloat: SubtargetFeature<"hvx-qfloat", "UseHVXQFloatOps",
    33:       "true", "Hexagon HVX QFloating point instructions">;
    34: 
    35: def ExtensionHVX: SubtargetFeature<"hvx", "HexagonHVXVersion",
    36:       "Hexagon::ArchEnum::V60", "Hexagon HVX instructions">;
    37: def ExtensionHVXV60: SubtargetFeature<"hvxv60", "HexagonHVXVersion",
    38:       "Hexagon::ArchEnum::V60", "Hexagon HVX instructions",
    39:       [ExtensionHVX]>;
    40: def ExtensionHVXV62: SubtargetFeature<"hvxv62", "HexagonHVXVersion",
    41:       "Hexagon::ArchEnum::V62", "Hexagon HVX instructions",
    42:       [ExtensionHVX, ExtensionHVXV60]>;
    43: def ExtensionHVXV65: SubtargetFeature<"hvxv65", "HexagonHVXVersion",
    44:       "Hexagon::ArchEnum::V65", "Hexagon HVX instructions",
    45:       [ExtensionHVX, ExtensionHVXV60, ExtensionHVXV62]>;
    46: def ExtensionHVXV66: SubtargetFeature<"hvxv66", "HexagonHVXVersion",
    47:       "Hexagon::ArchEnum::V66", "Hexagon HVX instructions",
    48:       [ExtensionHVX, ExtensionHVXV60, ExtensionHVXV62, ExtensionHVXV65,
    49:        ExtensionZReg]>;
    50: def ExtensionHVXV67: SubtargetFeature<"hvxv67", "HexagonHVXVersion",
```
- EN: It includes TableGen fragments such as llvm/Target/Target.td, HexagonDepArch.td so records can build on shared target definitions. It defines declarative TableGen records like ProcTinyCore, ExtensionZReg, ExtensionHVXQFloat, ExtensionHVX, ExtensionHVXV60, ... (9 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonDepArch, HexagonProcFamily, HexagonHVXVersion, showing how the code connects to sibling backend components.
- CN: 这里包含了 llvm/Target/Target.td, HexagonDepArch.td 等 TableGen 片段，使记录可以复用共享目标定义。 这里定义了 ProcTinyCore, ExtensionZReg, ExtensionHVXQFloat, ExtensionHVX, ExtensionHVXV60, ... (9 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonDepArch, HexagonProcFamily, HexagonHVXVersion，说明了它与同级后端组件的连接关系。

### Lines 51-100 / 第 51-100 行

```tablegen
    51:       "Hexagon::ArchEnum::V67", "Hexagon HVX instructions",
    52:       [ExtensionHVXV60, ExtensionHVXV62, ExtensionHVXV65, ExtensionHVXV66]>;
    53: def ExtensionHVXV68: SubtargetFeature<"hvxv68", "HexagonHVXVersion",
    54:       "Hexagon::ArchEnum::V68", "Hexagon HVX instructions",
    55:       [ExtensionHVXV60, ExtensionHVXV62, ExtensionHVXV65, ExtensionHVXV66,
    56:        ExtensionHVXV67]>;
    57: def ExtensionHVXV69: SubtargetFeature<"hvxv69", "HexagonHVXVersion",
    58:       "Hexagon::ArchEnum::V69", "Hexagon HVX instructions",
    59:       [ExtensionHVXV60, ExtensionHVXV62, ExtensionHVXV65, ExtensionHVXV66,
    60:        ExtensionHVXV67, ExtensionHVXV68]>;
    61: def ExtensionHVXV71: SubtargetFeature<"hvxv71", "HexagonHVXVersion",
    62:       "Hexagon::ArchEnum::V71", "Hexagon HVX instructions",
    63:       [ExtensionHVXV60, ExtensionHVXV62, ExtensionHVXV65, ExtensionHVXV66,
    64:        ExtensionHVXV67, ExtensionHVXV68, ExtensionHVXV69]>;
    65: def ExtensionHVXV73: SubtargetFeature<"hvxv73", "HexagonHVXVersion",
    66:       "Hexagon::ArchEnum::V73", "Hexagon HVX instructions",
    67:       [ExtensionHVXV60, ExtensionHVXV62, ExtensionHVXV65, ExtensionHVXV66,
    68:        ExtensionHVXV67, ExtensionHVXV68, ExtensionHVXV69, ExtensionHVXV71]>;
    69: 
    70: def ExtensionHVXV75: SubtargetFeature<"hvxv75", "HexagonHVXVersion",
    71:       "Hexagon::ArchEnum::V75", "Hexagon HVX instructions",
    72:       [ExtensionHVXV60, ExtensionHVXV62, ExtensionHVXV65, ExtensionHVXV66,
    73:        ExtensionHVXV67, ExtensionHVXV68, ExtensionHVXV69, ExtensionHVXV71,
    74:        ExtensionHVXV73]>;
    75: 
    76: def ExtensionHVXV79: SubtargetFeature<"hvxv79", "HexagonHVXVersion",
    77:       "Hexagon::ArchEnum::V79", "Hexagon HVX instructions",
    78:       [ExtensionHVXV60, ExtensionHVXV62, ExtensionHVXV65, ExtensionHVXV66,
    79:        ExtensionHVXV67, ExtensionHVXV68, ExtensionHVXV69, ExtensionHVXV71,
    80:        ExtensionHVXV73, ExtensionHVXV75]>;
    81: 
    82: def ExtensionHVXV81: SubtargetFeature<"hvxv81", "HexagonHVXVersion",
    83:       "Hexagon::ArchEnum::V81", "Hexagon HVX instructions",
    84:       [ExtensionHVXV65, ExtensionHVXV66, ExtensionHVXV67,
    85:        ExtensionHVXV68, ExtensionHVXV69, ExtensionHVXV71,
    86:        ExtensionHVXV73, ExtensionHVXV75, ExtensionHVXV79]>;
    87: 
    88: def ExtensionHVX64B: SubtargetFeature<"hvx-length64b", "UseHVX64BOps",
    89:       "true", "Hexagon HVX 64B instructions", [ExtensionHVX]>;
    90: def ExtensionHVX128B: SubtargetFeature<"hvx-length128b", "UseHVX128BOps",
    91:       "true", "Hexagon HVX 128B instructions", [ExtensionHVX]>;
    92: 
    93: def ExtensionAudio: SubtargetFeature<"audio", "UseAudioOps", "true",
    94:       "Hexagon Audio extension instructions">;
    95: 
    96: def ExtensionHVXIEEEFP: SubtargetFeature<"hvx-ieee-fp", "UseHVXIEEEFPOps",
    97:       "true", "Hexagon HVX IEEE floating point instructions">;
    98: 
    99: def FeatureCompound: SubtargetFeature<"compound", "UseCompound", "true",
   100:       "Use compound instructions">;
```
- EN: It defines declarative TableGen records like ExtensionHVXV68, ExtensionHVXV69, ExtensionHVXV71, ExtensionHVXV73, ExtensionHVXV75, ... (12 total); these records are consumed by TableGen instead of executed directly. Notable Hexagon symbols referenced here include HexagonHVXVersion, showing how the code connects to sibling backend components.
- CN: 这里定义了 ExtensionHVXV68, ExtensionHVXV69, ExtensionHVXV71, ExtensionHVXV73, ExtensionHVXV75, ... (12 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里引用的重要 Hexagon 符号包括 HexagonHVXVersion，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```tablegen
   101: def FeaturePackets: SubtargetFeature<"packets", "UsePackets", "true",
   102:       "Support for instruction packets">;
   103: def FeaturePreV65: SubtargetFeature<"prev65", "HasPreV65", "true",
   104:       "Support features deprecated in v65">;
   105: def FeatureLongCalls: SubtargetFeature<"long-calls", "UseLongCalls", "true",
   106:       "Use constant-extended calls">;
   107: def FeatureMemNoShuf: SubtargetFeature<"mem_noshuf", "HasMemNoShuf", "false",
   108:       "Supports mem_noshuf feature">;
   109: def FeatureMemops: SubtargetFeature<"memops", "UseMemops", "true",
   110:       "Use memop instructions">;
   111: def FeatureNVJ: SubtargetFeature<"nvj", "UseNewValueJumps", "true",
   112:       "Support for new-value jumps", [FeaturePackets]>;
   113: def FeatureNVS: SubtargetFeature<"nvs", "UseNewValueStores", "true",
   114:       "Support for new-value stores", [FeaturePackets]>;
   115: def FeatureSmallData: SubtargetFeature<"small-data", "UseSmallData", "true",
   116:       "Allow GP-relative addressing of global variables">;
   117: def FeatureDuplex: SubtargetFeature<"duplex", "EnableDuplex", "true",
   118:       "Enable generation of duplex instruction">;
   119: def FeatureReservedR19: SubtargetFeature<"reserved-r19", "ReservedR19",
   120:       "true", "Reserve register R19">;
   121: def FeatureNoreturnStackElim: SubtargetFeature<"noreturn-stack-elim",
   122:       "NoreturnStackElim", "true",
   123:       "Eliminate stack allocation in a noreturn function when possible">;
   124: def FeatureCabac: SubtargetFeature<"cabac", "UseCabac", "false",
   125:       "Emit the CABAC instruction">;
   126: 
   127: //===----------------------------------------------------------------------===//
   128: // Hexagon Instruction Predicate Definitions.
   129: //===----------------------------------------------------------------------===//
   130: 
   131: def UseMEMOPS          : Predicate<"HST->useMemops()">;
   132: def UseHVX64B          : Predicate<"HST->useHVX64BOps()">,
   133:                          AssemblerPredicate<(all_of ExtensionHVX64B)>;
   134: def UseHVX128B         : Predicate<"HST->useHVX128BOps()">,
   135:                          AssemblerPredicate<(all_of ExtensionHVX128B)>;
   136: def UseHVX             : Predicate<"HST->useHVXOps()">,
   137:                          AssemblerPredicate<(all_of ExtensionHVXV60)>;
   138: def UseHVXV60          : Predicate<"HST->useHVXV60Ops()">,
   139:                          AssemblerPredicate<(all_of ExtensionHVXV60)>;
   140: def UseHVXV62          : Predicate<"HST->useHVXV62Ops()">,
   141:                          AssemblerPredicate<(all_of ExtensionHVXV62)>;
   142: def UseHVXV65          : Predicate<"HST->useHVXV65Ops()">,
   143:                          AssemblerPredicate<(all_of ExtensionHVXV65)>;
   144: def UseHVXV66          : Predicate<"HST->useHVXV66Ops()">,
   145:                          AssemblerPredicate<(all_of ExtensionHVXV66)>;
   146: def UseHVXV67          : Predicate<"HST->useHVXV67Ops()">,
   147:                          AssemblerPredicate<(all_of ExtensionHVXV67)>;
   148: def UseHVXV68          : Predicate<"HST->useHVXV68Ops()">,
   149:                          AssemblerPredicate<(all_of ExtensionHVXV68)>;
   150: def UseHVXV69          : Predicate<"HST->useHVXV69Ops()">,
```
- EN: It defines declarative TableGen records like FeaturePackets, FeaturePreV65, FeatureLongCalls, FeatureMemNoShuf, FeatureMemops, ... (23 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里定义了 FeaturePackets, FeaturePreV65, FeatureLongCalls, FeatureMemNoShuf, FeatureMemops, ... (23 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 151-200 / 第 151-200 行

```tablegen
   151:                          AssemblerPredicate<(all_of ExtensionHVXV69)>;
   152: def UseHVXV71          : Predicate<"HST->useHVXV71Ops()">,
   153:                          AssemblerPredicate<(all_of ExtensionHVXV71)>;
   154: def UseHVXV73          : Predicate<"HST->useHVXV73Ops()">,
   155:                          AssemblerPredicate<(all_of ExtensionHVXV73)>;
   156: def UseHVXV75          : Predicate<"HST->useHVXV75Ops()">,
   157:                          AssemblerPredicate<(all_of ExtensionHVXV75)>;
   158: def UseHVXV79          : Predicate<"HST->useHVXV79Ops()">,
   159:                          AssemblerPredicate<(all_of ExtensionHVXV79)>;
   160: def UseHVXV81          : Predicate<"HST->useHVXV81Ops()">,
   161:                          AssemblerPredicate<(all_of ExtensionHVXV81)>;
   162: def UseAudio           : Predicate<"HST->useAudioOps()">,
   163:                          AssemblerPredicate<(all_of ExtensionAudio)>;
   164: def UseZReg            : Predicate<"HST->useZRegOps()">,
   165:                          AssemblerPredicate<(all_of ExtensionZReg)>;
   166: def UseCompound        : Predicate<"HST->useCompound()">;
   167: def HasPreV65          : Predicate<"HST->hasPreV65()">,
   168:                          AssemblerPredicate<(all_of FeaturePreV65)>;
   169: def UseHVXIEEEFP       : Predicate<"HST->useHVXIEEEFPOps()">,
   170:                          AssemblerPredicate<(all_of ExtensionHVXIEEEFP)>;
   171: def UseHVXQFloat       : Predicate<"HST->useHVXQFloatOps()">,
   172:                          AssemblerPredicate<(all_of ExtensionHVXQFloat)>;
   173: def UseHVXFloatingPoint: Predicate<"HST->useHVXFloatingPoint()">;
   174: def HasMemNoShuf       : Predicate<"HST->hasMemNoShuf()">,
   175:                          AssemblerPredicate<(all_of FeatureMemNoShuf)>;
   176: def NotOptTinyCore     : Predicate<"!HST->isTinyCore() ||"
   177:                                    "MF->getFunction().hasOptSize()"> {
   178:   let RecomputePerFunction = 1;
   179: }
   180: def UseSmallData       : Predicate<"HST->useSmallData()">;
   181: def UseCabac           : Predicate<"HST->useCabac()">,
   182:                          AssemblerPredicate<(any_of FeatureCabac)>;
   183: 
   184: def : HwModePredicateProlog<[{
   185:   const auto *HST = static_cast<const HexagonSubtarget *>(this);
   186: }]>;
   187: def Hvx64:  HwMode<[UseHVX64B]>;
   188: def Hvx128: HwMode<[UseHVX128B]>;
   189: 
   190: //===----------------------------------------------------------------------===//
   191: // Classes used for relation maps.
   192: //===----------------------------------------------------------------------===//
   193: 
   194: // The classes below should remain in hierarchical order...
   195: class ImmRegShl;
   196: // ImmRegRel - Filter class used to relate instructions having reg-reg form
   197: // with their reg-imm counterparts.
   198: class ImmRegRel;
   199: // PredRel - Filter class used to relate non-predicated instructions with their
   200: // predicated forms.
```
- EN: It declares types such as ImmRegShl, ImmRegRel, which carry the state or API of this component. It defines declarative TableGen records like UseHVXV71, UseHVXV73, UseHVXV75, UseHVXV79, UseHVXV81, ... (20 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明了 ImmRegShl, ImmRegRel 等类型，用来承载该组件的状态或接口。 这里定义了 UseHVXV71, UseHVXV73, UseHVXV75, UseHVXV79, UseHVXV81, ... (20 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```tablegen
   201: class PredRel;
   202: class PredNewRel: PredRel;
   203: // NewValueRel - Filter class used to relate regular store instructions with
   204: // their new-value store form.
   205: class NewValueRel: PredNewRel;
   206: class AddrModeRel: NewValueRel;
   207: class PostInc_BaseImm;
   208: class IntrinsicsRel;
   209: // ... through here.
   210: 
   211: //===----------------------------------------------------------------------===//
   212: // Generate mapping table to relate non-predicate instructions with their
   213: // predicated formats - true and false.
   214: //
   215: 
   216: def getPredOpcode : InstrMapping {
   217:   let FilterClass = "PredRel";
   218:   // Instructions with the same BaseOpcode and isNVStore values form a row.
   219:   let RowFields = ["BaseOpcode", "isNVStore", "PNewValue", "isBrTaken", "isNT"];
   220:   // Instructions with the same predicate sense form a column.
   221:   let ColFields = ["PredSense"];
   222:   // The key column is the unpredicated instructions.
   223:   let KeyCol = [""];
   224:   // Value columns are PredSense=true and PredSense=false
   225:   let ValueCols = [["true"], ["false"]];
   226: }
   227: 
   228: //===----------------------------------------------------------------------===//
   229: // Generate mapping table to relate predicate-true instructions with their
   230: // predicate-false forms
   231: //
   232: def getFalsePredOpcode : InstrMapping {
   233:   let FilterClass = "PredRel";
   234:   let RowFields = ["BaseOpcode", "PNewValue", "isNVStore", "isBrTaken", "isNT"];
   235:   let ColFields = ["PredSense"];
   236:   let KeyCol = ["true"];
   237:   let ValueCols = [["false"]];
   238: }
   239: 
   240: //===----------------------------------------------------------------------===//
   241: // Generate mapping table to relate predicate-false instructions with their
   242: // predicate-true forms
   243: //
   244: def getTruePredOpcode : InstrMapping {
   245:   let FilterClass = "PredRel";
   246:   let RowFields = ["BaseOpcode", "PNewValue", "isNVStore", "isBrTaken", "isNT"];
   247:   let ColFields = ["PredSense"];
   248:   let KeyCol = ["false"];
   249:   let ValueCols = [["true"]];
   250: }
```
- EN: It declares types such as PredRel, PredNewRel:, NewValueRel:, AddrModeRel:, ... (6 total), which carry the state or API of this component. It defines declarative TableGen records like PredRel, PredNewRel, NewValueRel, AddrModeRel, PostInc_BaseImm, ... (9 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 PredRel, PredNewRel:, NewValueRel:, AddrModeRel:, ... (6 total) 等类型，用来承载该组件的状态或接口。 这里定义了 PredRel, PredNewRel, NewValueRel, AddrModeRel, PostInc_BaseImm, ... (9 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 251-300 / 第 251-300 行

```tablegen
   251: 
   252: //===----------------------------------------------------------------------===//
   253: // Generate mapping table to relate predicated instructions with their .new
   254: // format.
   255: //
   256: def getPredNewOpcode : InstrMapping {
   257:   let FilterClass = "PredNewRel";
   258:   let RowFields = ["BaseOpcode", "PredSense", "isNVStore", "isBrTaken"];
   259:   let ColFields = ["PNewValue"];
   260:   let KeyCol = [""];
   261:   let ValueCols = [["new"]];
   262: }
   263: 
   264: //===----------------------------------------------------------------------===//
   265: // Generate mapping table to relate .new predicated instructions with their old
   266: // format.
   267: //
   268: def getPredOldOpcode : InstrMapping {
   269:   let FilterClass = "PredNewRel";
   270:   let RowFields = ["BaseOpcode", "PredSense", "isNVStore", "isBrTaken"];
   271:   let ColFields = ["PNewValue"];
   272:   let KeyCol = ["new"];
   273:   let ValueCols = [[""]];
   274: }
   275: 
   276: //===----------------------------------------------------------------------===//
   277: // Generate mapping table to relate store instructions with their new-value
   278: // format.
   279: //
   280: def getNewValueOpcode : InstrMapping {
   281:   let FilterClass = "NewValueRel";
   282:   let RowFields = ["BaseOpcode", "PredSense", "PNewValue", "addrMode", "isNT"];
   283:   let ColFields = ["NValueST"];
   284:   let KeyCol = ["false"];
   285:   let ValueCols = [["true"]];
   286: }
   287: 
   288: //===----------------------------------------------------------------------===//
   289: // Generate mapping table to relate new-value store instructions with their old
   290: // format.
   291: //
   292: def getNonNVStore : InstrMapping {
   293:   let FilterClass = "NewValueRel";
   294:   let RowFields = ["BaseOpcode", "PredSense", "PNewValue", "addrMode", "isNT"];
   295:   let ColFields = ["NValueST"];
   296:   let KeyCol = ["true"];
   297:   let ValueCols = [["false"]];
   298: }
   299: 
   300: def changeAddrMode_abs_io: InstrMapping {
```
- EN: It defines declarative TableGen records like getPredNewOpcode, getPredOldOpcode, getNewValueOpcode, getNonNVStore, changeAddrMode_abs_io; these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 getPredNewOpcode, getPredOldOpcode, getNewValueOpcode, getNonNVStore, changeAddrMode_abs_io 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 301-350 / 第 301-350 行

```tablegen
   301:   let FilterClass = "AddrModeRel";
   302:   let RowFields = ["CextOpcode", "PredSense", "PNewValue", "isNVStore",
   303:                    "isFloat"];
   304:   let ColFields = ["addrMode"];
   305:   let KeyCol = ["Absolute"];
   306:   let ValueCols = [["BaseImmOffset"]];
   307: }
   308: 
   309: def changeAddrMode_io_abs: InstrMapping {
   310:   let FilterClass = "AddrModeRel";
   311:   let RowFields = ["CextOpcode", "PredSense", "PNewValue", "isNVStore",
   312:                    "isFloat"];
   313:   let ColFields = ["addrMode"];
   314:   let KeyCol = ["BaseImmOffset"];
   315:   let ValueCols = [["Absolute"]];
   316: }
   317: 
   318: def changeAddrMode_io_rr: InstrMapping {
   319:   let FilterClass = "AddrModeRel";
   320:   let RowFields = ["CextOpcode", "PredSense", "PNewValue", "isNVStore"];
   321:   let ColFields = ["addrMode"];
   322:   let KeyCol = ["BaseImmOffset"];
   323:   let ValueCols = [["BaseRegOffset"]];
   324: }
   325: 
   326: def changeAddrMode_rr_io: InstrMapping {
   327:   let FilterClass = "AddrModeRel";
   328:   let RowFields = ["CextOpcode", "PredSense", "PNewValue", "isNVStore"];
   329:   let ColFields = ["addrMode"];
   330:   let KeyCol = ["BaseRegOffset"];
   331:   let ValueCols = [["BaseImmOffset"]];
   332: }
   333: 
   334: def changeAddrMode_pi_io: InstrMapping {
   335:   let FilterClass = "PostInc_BaseImm";
   336:   let RowFields = ["CextOpcode", "PredSense", "PNewValue", "isNVStore"];
   337:   let ColFields = ["addrMode"];
   338:   let KeyCol = ["PostInc"];
   339:   let ValueCols = [["BaseImmOffset"]];
   340: }
   341: 
   342: def changeAddrMode_io_pi: InstrMapping {
   343:   let FilterClass = "PostInc_BaseImm";
   344:   let RowFields = ["CextOpcode", "PredSense", "PNewValue", "isNVStore"];
   345:   let ColFields = ["addrMode"];
   346:   let KeyCol = ["BaseImmOffset"];
   347:   let ValueCols = [["PostInc"]];
   348: }
   349: 
   350: def changeAddrMode_rr_ur: InstrMapping {
```
- EN: It defines declarative TableGen records like changeAddrMode_io_abs, changeAddrMode_io_rr, changeAddrMode_rr_io, changeAddrMode_pi_io, changeAddrMode_io_pi, ... (6 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 changeAddrMode_io_abs, changeAddrMode_io_rr, changeAddrMode_rr_io, changeAddrMode_pi_io, changeAddrMode_io_pi, ... (6 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 351-400 / 第 351-400 行

```tablegen
   351:   let FilterClass = "ImmRegShl";
   352:   let RowFields = ["CextOpcode", "PredSense", "PNewValue", "isNVStore"];
   353:   let ColFields = ["addrMode"];
   354:   let KeyCol = ["BaseRegOffset"];
   355:   let ValueCols = [["BaseLongOffset"]];
   356: }
   357: 
   358: def changeAddrMode_ur_rr: InstrMapping {
   359:   let FilterClass = "ImmRegShl";
   360:   let RowFields = ["CextOpcode", "PredSense", "PNewValue", "isNVStore"];
   361:   let ColFields = ["addrMode"];
   362:   let KeyCol = ["BaseLongOffset"];
   363:   let ValueCols = [["BaseRegOffset"]];
   364: }
   365: 
   366: def getRegForm : InstrMapping {
   367:   let FilterClass = "ImmRegRel";
   368:   let RowFields = ["CextOpcode", "PredSense", "PNewValue"];
   369:   let ColFields = ["InputType"];
   370:   let KeyCol = ["imm"];
   371:   let ValueCols = [["reg"]];
   372: }
   373: 
   374: def notTakenBranchPrediction : InstrMapping {
   375:   let FilterClass = "PredRel";
   376:   let RowFields = ["BaseOpcode", "PNewValue",  "PredSense", "isBranch", "isPredicated"];
   377:   let ColFields = ["isBrTaken"];
   378:   let KeyCol = ["true"];
   379:   let ValueCols = [["false"]];
   380: }
   381: 
   382: def takenBranchPrediction : InstrMapping {
   383:   let FilterClass = "PredRel";
   384:   let RowFields = ["BaseOpcode", "PNewValue",  "PredSense", "isBranch", "isPredicated"];
   385:   let ColFields = ["isBrTaken"];
   386:   let KeyCol = ["false"];
   387:   let ValueCols = [["true"]];
   388: }
   389: 
   390: def getRealHWInstr : InstrMapping {
   391:   let FilterClass = "IntrinsicsRel";
   392:   let RowFields = ["BaseOpcode"];
   393:   let ColFields = ["InstrType"];
   394:   let KeyCol = ["Pseudo"];
   395:   let ValueCols = [["Pseudo"], ["Real"]];
   396: }
   397: //===----------------------------------------------------------------------===//
   398: // Register File, Instruction Descriptions
   399: //===----------------------------------------------------------------------===//
   400: include "HexagonSchedule.td"
```
- EN: It includes TableGen fragments such as HexagonSchedule.td so records can build on shared target definitions. It defines declarative TableGen records like changeAddrMode_ur_rr, getRegForm, notTakenBranchPrediction, takenBranchPrediction, getRealHWInstr; these records are consumed by TableGen instead of executed directly. Notable Hexagon symbols referenced here include HexagonSchedule, showing how the code connects to sibling backend components.
- CN: 这里包含了 HexagonSchedule.td 等 TableGen 片段，使记录可以复用共享目标定义。 这里定义了 changeAddrMode_ur_rr, getRegForm, notTakenBranchPrediction, takenBranchPrediction, getRealHWInstr 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里引用的重要 Hexagon 符号包括 HexagonSchedule，说明了它与同级后端组件的连接关系。

### Lines 401-450 / 第 401-450 行

```tablegen
   401: include "HexagonRegisterInfo.td"
   402: include "HexagonOperands.td"
   403: include "HexagonDepOperands.td"
   404: include "HexagonDepITypes.td"
   405: include "HexagonInstrFormats.td"
   406: include "HexagonDepInstrFormats.td"
   407: include "HexagonDepInstrInfo.td"
   408: include "HexagonCallingConv.td"
   409: include "HexagonPseudo.td"
   410: include "HexagonPatterns.td"
   411: include "HexagonPatternsHVX.td"
   412: include "HexagonPatternsV65.td"
   413: include "HexagonDepMappings.td"
   414: include "HexagonIntrinsics.td"
   415: 
   416: defm : RemapAllTargetPseudoPointerOperands<IntRegs>;
   417: 
   418: def HexagonInstrInfo : InstrInfo;
   419: 
   420: //===----------------------------------------------------------------------===//
   421: // Hexagon processors supported.
   422: //===----------------------------------------------------------------------===//
   423: 
   424: class Proc<string Name, SchedMachineModel Model,
   425:            list<SubtargetFeature> Features>
   426:  : ProcessorModel<Name, Model, Features>;
   427: 
   428: def : Proc<"generic", HexagonModelV60,
   429:            [ArchV5, ArchV55, ArchV60,
   430:             FeatureCompound, FeatureDuplex, FeaturePreV65, FeatureMemops,
   431:             FeatureNVJ, FeatureNVS, FeaturePackets, FeatureSmallData,
   432:             FeatureCabac]>;
   433: def : Proc<"hexagonv5",  HexagonModelV5,
   434:            [ArchV5,
   435:             FeatureCompound, FeatureDuplex, FeaturePreV65, FeatureMemops,
   436:             FeatureNVJ, FeatureNVS, FeaturePackets, FeatureSmallData,
   437:             FeatureCabac]>;
   438: def : Proc<"hexagonv55", HexagonModelV55,
   439:            [ArchV5, ArchV55,
   440:             FeatureCompound, FeatureDuplex, FeaturePreV65, FeatureMemops,
   441:             FeatureNVJ, FeatureNVS, FeaturePackets, FeatureSmallData,
   442:             FeatureCabac]>;
   443: def : Proc<"hexagonv60", HexagonModelV60,
   444:            [ArchV5, ArchV55, ArchV60,
   445:             FeatureCompound, FeatureDuplex, FeaturePreV65, FeatureMemops,
   446:             FeatureNVJ, FeatureNVS, FeaturePackets, FeatureSmallData,
   447:             FeatureCabac]>;
   448: def : Proc<"hexagonv62", HexagonModelV62,
   449:            [ArchV5, ArchV55, ArchV60, ArchV62,
   450:             FeatureCompound, FeatureDuplex, FeaturePreV65, FeatureMemops,
```
- EN: It includes TableGen fragments such as HexagonRegisterInfo.td, HexagonOperands.td, HexagonDepOperands.td, HexagonDepITypes.td, ... (14 total) so records can build on shared target definitions. It declares types such as Proc, which carry the state or API of this component. It defines declarative TableGen records like HexagonInstrInfo, Proc; these records are consumed by TableGen instead of executed directly. Notable Hexagon symbols referenced here include HexagonRegisterInfo, HexagonOperands, HexagonDepOperands, HexagonDepITypes, ... (19 total), showing how the code connects to sibling backend components.
- CN: 这里包含了 HexagonRegisterInfo.td, HexagonOperands.td, HexagonDepOperands.td, HexagonDepITypes.td, ... (14 total) 等 TableGen 片段，使记录可以复用共享目标定义。 这里声明了 Proc 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonInstrInfo, Proc 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里引用的重要 Hexagon 符号包括 HexagonRegisterInfo, HexagonOperands, HexagonDepOperands, HexagonDepITypes, ... (19 total)，说明了它与同级后端组件的连接关系。

### Lines 451-500 / 第 451-500 行

```tablegen
   451:             FeatureNVJ, FeatureNVS, FeaturePackets, FeatureSmallData,
   452:             FeatureCabac]>;
   453: def : Proc<"hexagonv65", HexagonModelV65,
   454:            [ArchV5, ArchV55, ArchV60, ArchV62, ArchV65,
   455:             FeatureCompound, FeatureDuplex, FeatureMemNoShuf, FeatureMemops,
   456:             FeatureNVJ, FeatureNVS, FeaturePackets, FeatureSmallData,
   457:             FeatureCabac]>;
   458: def : Proc<"hexagonv66", HexagonModelV66,
   459:            [ArchV5, ArchV55, ArchV60, ArchV62, ArchV65, ArchV66,
   460:             FeatureCompound, FeatureDuplex, FeatureMemNoShuf, FeatureMemops,
   461:             FeatureNVJ, FeatureNVS, FeaturePackets, FeatureSmallData,
   462:             FeatureCabac]>;
   463: def : Proc<"hexagonv67", HexagonModelV67,
   464:            [ArchV5, ArchV55, ArchV60, ArchV62, ArchV65, ArchV66, ArchV67,
   465:             FeatureCompound, FeatureDuplex, FeatureMemNoShuf, FeatureMemops,
   466:             FeatureNVJ, FeatureNVS, FeaturePackets, FeatureSmallData,
   467:             FeatureCabac]>;
   468: def : Proc<"hexagonv68", HexagonModelV68,
   469:            [ArchV5, ArchV55, ArchV60, ArchV62, ArchV65, ArchV66, ArchV67,
   470:             ArchV68,
   471:             FeatureCompound, FeatureDuplex, FeatureMemNoShuf, FeatureMemops,
   472:             FeatureNVJ, FeatureNVS, FeaturePackets, FeatureSmallData,
   473:             FeatureCabac]>;
   474: def : Proc<"hexagonv69", HexagonModelV69,
   475:            [ArchV5, ArchV55, ArchV60, ArchV62, ArchV65, ArchV66, ArchV67,
   476:             ArchV68, ArchV69,
   477:             FeatureCompound, FeatureDuplex, FeatureMemNoShuf, FeatureMemops,
   478:             FeatureNVJ, FeatureNVS, FeaturePackets, FeatureSmallData,
   479:             FeatureCabac]>;
   480: def : Proc<"hexagonv71", HexagonModelV71,
   481:            [ArchV5, ArchV55, ArchV60, ArchV62, ArchV65, ArchV66, ArchV67,
   482:             ArchV68, ArchV69, ArchV71,
   483:             FeatureCompound, FeatureDuplex, FeatureMemNoShuf, FeatureMemops,
   484:             FeatureNVJ, FeatureNVS, FeaturePackets, FeatureSmallData,
   485:             FeatureCabac]>;
   486: def : Proc<"hexagonv73", HexagonModelV73,
   487:            [ArchV5, ArchV55, ArchV60, ArchV62, ArchV65, ArchV66, ArchV67,
   488:             ArchV68, ArchV69, ArchV71, ArchV73,
   489:             FeatureCompound, FeatureDuplex, FeatureMemNoShuf, FeatureMemops,
   490:             FeatureNVJ, FeatureNVS, FeaturePackets, FeatureSmallData]>;
   491: def : Proc<"hexagonv75", HexagonModelV75,
   492:            [ArchV5, ArchV55, ArchV60, ArchV62, ArchV65, ArchV66, ArchV67,
   493:            ArchV68, ArchV69, ArchV71, ArchV73, ArchV75, FeatureCompound,
   494:            FeatureDuplex, FeatureMemNoShuf, FeatureMemops, FeatureNVJ,
   495:            FeatureNVS, FeaturePackets, FeatureSmallData]>;
   496: def : Proc<"hexagonv79", HexagonModelV79,
   497:            [ArchV5, ArchV55, ArchV60, ArchV62, ArchV65, ArchV66, ArchV67,
   498:            ArchV68, ArchV69, ArchV71, ArchV73, ArchV75, ArchV79,
   499:            FeatureCompound, FeatureDuplex, FeatureMemNoShuf, FeatureMemops,
   500:            FeatureNVJ, FeatureNVS, FeaturePackets, FeatureSmallData]>;
```
- EN: Notable Hexagon symbols referenced here include HexagonModelV65, HexagonModelV66, HexagonModelV67, HexagonModelV68, ... (9 total), showing how the code connects to sibling backend components.
- CN: 这里引用的重要 Hexagon 符号包括 HexagonModelV65, HexagonModelV66, HexagonModelV67, HexagonModelV68, ... (9 total)，说明了它与同级后端组件的连接关系。

### Lines 501-549 / 第 501-549 行

```tablegen
   501: def : Proc<"hexagonv81", HexagonModelV81,
   502:            [ArchV5, ArchV55, ArchV60, ArchV62, ArchV65, ArchV66, ArchV67,
   503:             ArchV68, ArchV69, ArchV71, ArchV73, ArchV75, ArchV79, ArchV81,
   504:             FeatureCompound, FeatureDuplex, FeatureMemNoShuf, FeatureMemops,
   505:             FeatureNVJ, FeatureNVS, FeaturePackets, FeatureSmallData]>;
   506: 
   507: // Need to update the correct features for tiny core.
   508: // Disable NewValueJumps since the packetizer is unable to handle a packet with
   509: // a new value jump and another SLOT0 instruction.
   510: def : Proc<"hexagonv67t", HexagonModelV67T,
   511:            [ArchV5, ArchV55, ArchV60, ArchV62, ArchV65, ArchV66, ArchV67,
   512:             ProcTinyCore, ExtensionAudio,
   513:             FeatureCompound, FeatureMemNoShuf, FeatureMemops,
   514:             FeatureNVS, FeaturePackets, FeatureSmallData]>;
   515: 
   516: def : Proc<"hexagonv71t", HexagonModelV71T,
   517:            [ArchV5, ArchV55, ArchV60, ArchV62, ArchV65, ArchV66, ArchV67,
   518:             ArchV68, ArchV69, ArchV71,
   519:             ProcTinyCore, ExtensionAudio,
   520:             FeatureCompound, FeatureMemNoShuf, FeatureMemops,
   521:             FeatureNVS, FeaturePackets, FeatureSmallData]>;
   522: 
   523: //===----------------------------------------------------------------------===//
   524: // Declare the target which we are implementing
   525: //===----------------------------------------------------------------------===//
   526: 
   527: def HexagonAsmParser : AsmParser {
   528:   let ShouldEmitMatchRegisterAltName = 1;
   529:   bit HasMnemonicFirst = 0;
   530: }
   531: 
   532: def HexagonAsmParserVariant : AsmParserVariant {
   533:   int Variant = 0;
   534:   string TokenizingCharacters = "#()=:.<>!+*-|^&";
   535:   string BreakCharacters = "";
   536: }
   537: 
   538: def HexagonAsmWriter : AsmWriter {
   539:   string AsmWriterClassName  = "InstPrinter";
   540:   bit isMCAsmWriter = 1;
   541: }
   542: 
   543: def Hexagon : Target {
   544:   let InstructionSet = HexagonInstrInfo;
   545:   let AssemblyParsers = [HexagonAsmParser];
   546:   let AssemblyParserVariants = [HexagonAsmParserVariant];
   547:   let AssemblyWriters = [HexagonAsmWriter];
   548:   let AllowRegisterRenaming = 1;
   549: }
```
- EN: It defines declarative TableGen records like HexagonAsmParser, HexagonAsmParserVariant, HexagonAsmWriter, Hexagon; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonModelV81, HexagonModelV67T, HexagonModelV71T, HexagonAsmParser, ... (7 total), showing how the code connects to sibling backend components.
- CN: 这里定义了 HexagonAsmParser, HexagonAsmParserVariant, HexagonAsmWriter, Hexagon 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonModelV81, HexagonModelV67T, HexagonModelV71T, HexagonAsmParser, ... (7 total)，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- register modeling / 寄存器建模
- subtarget feature gating / 子目标特性控制
- generated metadata tables / 生成的元数据表
- TableGen DSL records / TableGen DSL 记录

## Dependencies / 依赖关系

- Included TableGen files / 包含的 TableGen 文件: `llvm/Target/Target.td, HexagonDepArch.td, HexagonSchedule.td, HexagonRegisterInfo.td, HexagonOperands.td, HexagonDepOperands.td, HexagonDepITypes.td, HexagonInstrFormats.td, HexagonDepInstrFormats.td, HexagonDepInstrInfo.td, ... (17 total)`
- Hexagon symbols / Hexagon 符号: `HexagonDepArch, HexagonProcFamily, HexagonHVXVersion, HexagonSubtarget, HexagonSchedule, HexagonRegisterInfo, HexagonOperands, HexagonDepOperands, HexagonDepITypes, HexagonInstrFormats, ... (39 total)`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
