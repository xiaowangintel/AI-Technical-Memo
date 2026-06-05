# HexagonSubtarget.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonSubtarget.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file declares the Hexagon specific subclass of TargetSubtarget.
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonSubtarget.h - Define Subtarget for the Hexagon ----*- C++ -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file declares the Hexagon specific subclass of TargetSubtarget.
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #ifndef LLVM_LIB_TARGET_HEXAGON_HEXAGONSUBTARGET_H
    14: #define LLVM_LIB_TARGET_HEXAGON_HEXAGONSUBTARGET_H
    15: 
    16: #include "HexagonDepArch.h"
    17: #include "HexagonFrameLowering.h"
    18: #include "HexagonISelLowering.h"
    19: #include "HexagonInstrInfo.h"
    20: #include "HexagonRegisterInfo.h"
    21: #include "HexagonSelectionDAGInfo.h"
    22: #include "llvm/ADT/SmallSet.h"
    23: #include "llvm/ADT/StringRef.h"
    24: #include "llvm/CodeGen/ScheduleDAGMutation.h"
    25: #include "llvm/CodeGen/TargetSubtargetInfo.h"
    26: #include "llvm/MC/MCInstrItineraries.h"
    27: #include "llvm/Support/Alignment.h"
    28: #include <memory>
    29: #include <string>
    30: #include <vector>
    31: 
    32: #define GET_SUBTARGETINFO_HEADER
    33: #include "HexagonGenSubtargetInfo.inc"
    34: 
    35: namespace llvm {
    36: 
    37: class MachineInstr;
    38: class SDep;
    39: class SUnit;
    40: class TargetMachine;
    41: class Triple;
    42: 
    43: class HexagonSubtarget : public HexagonGenSubtargetInfo {
    44:   virtual void anchor();
    45: 
    46:   bool UseHVX64BOps = false;
    47:   bool UseHVX128BOps = false;
    48: 
    49:   bool UseAudioOps = false;
    50:   bool UseCompound = false;
```
- EN: It imports headers such as HexagonDepArch.h, HexagonFrameLowering.h, HexagonISelLowering.h, HexagonInstrInfo.h, ... (16 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as MachineInstr, SDep, SUnit, TargetMachine, ... (6 total), which carry the state or API of this component.
- CN: 这里引入了 HexagonDepArch.h, HexagonFrameLowering.h, HexagonISelLowering.h, HexagonInstrInfo.h, ... (16 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 MachineInstr, SDep, SUnit, TargetMachine, ... (6 total) 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51:   bool UseLongCalls = false;
    52:   bool UseMemops = false;
    53:   bool UsePackets = false;
    54:   bool UseNewValueJumps = false;
    55:   bool UseNewValueStores = false;
    56:   bool UseSmallData = false;
    57:   bool UseZRegOps = false;
    58:   bool UseHVXIEEEFPOps = false;
    59:   bool UseHVXQFloatOps = false;
    60:   bool UseHVXFloatingPoint = false;
    61:   bool UseCabac = false;
    62: 
    63:   bool HasPreV65 = false;
    64:   bool HasMemNoShuf = false;
    65:   bool EnableDuplex = false;
    66:   bool ReservedR19 = false;
    67:   bool NoreturnStackElim = false;
    68: 
    69: public:
    70:   Hexagon::ArchEnum HexagonArchVersion;
    71:   Hexagon::ArchEnum HexagonHVXVersion = Hexagon::ArchEnum::NoArch;
    72:   CodeGenOptLevel OptLevel;
    73:   /// True if the target should use Back-Skip-Back scheduling. This is the
    74:   /// default for V60.
    75:   bool UseBSBScheduling;
    76: 
    77:   struct UsrOverflowMutation : public ScheduleDAGMutation {
    78:     void apply(ScheduleDAGInstrs *DAG) override;
    79:   };
    80:   struct HVXMemLatencyMutation : public ScheduleDAGMutation {
    81:     void apply(ScheduleDAGInstrs *DAG) override;
    82:   };
    83:   struct CallMutation : public ScheduleDAGMutation {
    84:     void apply(ScheduleDAGInstrs *DAG) override;
    85:   private:
    86:     bool shouldTFRICallBind(const HexagonInstrInfo &HII,
    87:           const SUnit &Inst1, const SUnit &Inst2) const;
    88:   };
    89:   struct BankConflictMutation : public ScheduleDAGMutation {
    90:     void apply(ScheduleDAGInstrs *DAG) override;
    91:   };
    92: 
    93: private:
    94:   enum HexagonProcFamilyEnum { Others, TinyCore };
    95: 
    96:   std::string CPUString;
    97:   HexagonProcFamilyEnum HexagonProcFamily = Others;
    98:   Triple TargetTriple;
    99: 
   100:   // The following objects can use the TargetTriple, so they must be
```
- EN: It declares types such as UsrOverflowMutation, HVXMemLatencyMutation, CallMutation, BankConflictMutation, ... (5 total), which carry the state or API of this component. It declares or implements routines such as apply, shouldTFRICallBind, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonArchVersion, HexagonHVXVersion, HexagonInstrInfo, HexagonProcFamilyEnum, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里声明了 UsrOverflowMutation, HVXMemLatencyMutation, CallMutation, BankConflictMutation, ... (5 total) 等类型，用来承载该组件的状态或接口。 这里声明或实现了 apply, shouldTFRICallBind 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonArchVersion, HexagonHVXVersion, HexagonInstrInfo, HexagonProcFamilyEnum, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:   // declared after it.
   102:   HexagonInstrInfo InstrInfo;
   103:   HexagonTargetLowering TLInfo;
   104:   HexagonSelectionDAGInfo TSInfo;
   105:   HexagonFrameLowering FrameLowering;
   106:   InstrItineraryData InstrItins;
   107: 
   108: public:
   109:   HexagonSubtarget(const Triple &TT, StringRef CPU, StringRef FS,
   110:                    const TargetMachine &TM);
   111: 
   112:   const Triple &getTargetTriple() const { return TargetTriple; }
   113:   bool isEnvironmentMusl() const {
   114:     return TargetTriple.getEnvironment() == Triple::Musl;
   115:   }
   116: 
   117:   /// getInstrItins - Return the instruction itineraries based on subtarget
   118:   /// selection.
   119:   const InstrItineraryData *getInstrItineraryData() const override {
   120:     return &InstrItins;
   121:   }
   122:   const HexagonInstrInfo *getInstrInfo() const override { return &InstrInfo; }
   123:   const HexagonRegisterInfo *getRegisterInfo() const override {
   124:     return &InstrInfo.getRegisterInfo();
   125:   }
   126:   const HexagonTargetLowering *getTargetLowering() const override {
   127:     return &TLInfo;
   128:   }
   129:   const HexagonFrameLowering *getFrameLowering() const override {
   130:     return &FrameLowering;
   131:   }
   132:   const HexagonSelectionDAGInfo *getSelectionDAGInfo() const override {
   133:     return &TSInfo;
   134:   }
   135: 
   136:   HexagonSubtarget &initializeSubtargetDependencies(StringRef CPU,
   137:                                                     StringRef FS);
   138: 
   139:   /// ParseSubtargetFeatures - Parses features string setting specified
   140:   /// subtarget options.  Definition of function is auto generated by tblgen.
   141:   void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);
   142: 
   143:   bool isXRaySupported() const override { return true; }
   144: 
   145:   bool hasV5Ops() const {
   146:     return getHexagonArchVersion() >= Hexagon::ArchEnum::V5;
   147:   }
   148:   bool hasV5OpsOnly() const {
   149:     return getHexagonArchVersion() == Hexagon::ArchEnum::V5;
   150:   }
```
- EN: It declares or implements routines such as HexagonSubtarget, getTargetTriple, isEnvironmentMusl, getInstrItineraryData, ... (14 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonTargetLowering, HexagonSelectionDAGInfo, HexagonFrameLowering, ... (6 total), showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonSubtarget, getTargetTriple, isEnvironmentMusl, getInstrItineraryData, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonTargetLowering, HexagonSelectionDAGInfo, HexagonFrameLowering, ... (6 total)，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   bool hasV55Ops() const {
   152:     return getHexagonArchVersion() >= Hexagon::ArchEnum::V55;
   153:   }
   154:   bool hasV55OpsOnly() const {
   155:     return getHexagonArchVersion() == Hexagon::ArchEnum::V55;
   156:   }
   157:   bool hasV60Ops() const {
   158:     return getHexagonArchVersion() >= Hexagon::ArchEnum::V60;
   159:   }
   160:   bool hasV60OpsOnly() const {
   161:     return getHexagonArchVersion() == Hexagon::ArchEnum::V60;
   162:   }
   163:   bool hasV62Ops() const {
   164:     return getHexagonArchVersion() >= Hexagon::ArchEnum::V62;
   165:   }
   166:   bool hasV62OpsOnly() const {
   167:     return getHexagonArchVersion() == Hexagon::ArchEnum::V62;
   168:   }
   169:   bool hasV65Ops() const {
   170:     return getHexagonArchVersion() >= Hexagon::ArchEnum::V65;
   171:   }
   172:   bool hasV65OpsOnly() const {
   173:     return getHexagonArchVersion() == Hexagon::ArchEnum::V65;
   174:   }
   175:   bool hasV66Ops() const {
   176:     return getHexagonArchVersion() >= Hexagon::ArchEnum::V66;
   177:   }
   178:   bool hasV66OpsOnly() const {
   179:     return getHexagonArchVersion() == Hexagon::ArchEnum::V66;
   180:   }
   181:   bool hasV67Ops() const {
   182:     return getHexagonArchVersion() >= Hexagon::ArchEnum::V67;
   183:   }
   184:   bool hasV67OpsOnly() const {
   185:     return getHexagonArchVersion() == Hexagon::ArchEnum::V67;
   186:   }
   187:   bool hasV68Ops() const {
   188:     return getHexagonArchVersion() >= Hexagon::ArchEnum::V68;
   189:   }
   190:   bool hasV68OpsOnly() const {
   191:     return getHexagonArchVersion() == Hexagon::ArchEnum::V68;
   192:   }
   193:   bool hasV69Ops() const {
   194:     return getHexagonArchVersion() >= Hexagon::ArchEnum::V69;
   195:   }
   196:   bool hasV69OpsOnly() const {
   197:     return getHexagonArchVersion() == Hexagon::ArchEnum::V69;
   198:   }
   199:   bool hasV71Ops() const {
   200:     return getHexagonArchVersion() >= Hexagon::ArchEnum::V71;
```
- EN: It declares or implements routines such as hasV55Ops, hasV55OpsOnly, hasV60Ops, hasV60OpsOnly, ... (17 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 hasV55Ops, hasV55OpsOnly, hasV60Ops, hasV60OpsOnly, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 201-250 / 第 201-250 行

```cpp
   201:   }
   202:   bool hasV71OpsOnly() const {
   203:     return getHexagonArchVersion() == Hexagon::ArchEnum::V71;
   204:   }
   205:   bool hasV73Ops() const {
   206:     return getHexagonArchVersion() >= Hexagon::ArchEnum::V73;
   207:   }
   208:   bool hasV73OpsOnly() const {
   209:     return getHexagonArchVersion() == Hexagon::ArchEnum::V73;
   210:   }
   211:   bool hasV75Ops() const {
   212:     return getHexagonArchVersion() >= Hexagon::ArchEnum::V75;
   213:   }
   214:   bool hasV75OpsOnly() const {
   215:     return getHexagonArchVersion() == Hexagon::ArchEnum::V75;
   216:   }
   217:   bool hasV79Ops() const {
   218:     return getHexagonArchVersion() >= Hexagon::ArchEnum::V79;
   219:   }
   220:   bool hasV79OpsOnly() const {
   221:     return getHexagonArchVersion() == Hexagon::ArchEnum::V79;
   222:   }
   223:   bool useHVXV79Ops() const {
   224:     return HexagonHVXVersion >= Hexagon::ArchEnum::V79;
   225:   }
   226:   bool hasV81Ops() const {
   227:     return getHexagonArchVersion() >= Hexagon::ArchEnum::V81;
   228:   }
   229:   bool hasV81OpsOnly() const {
   230:     return getHexagonArchVersion() == Hexagon::ArchEnum::V81;
   231:   }
   232:   bool useHVXV81Ops() const {
   233:     return HexagonHVXVersion >= Hexagon::ArchEnum::V81;
   234:   }
   235: 
   236:   bool useAudioOps() const { return UseAudioOps; }
   237:   bool useCompound() const { return UseCompound; }
   238:   bool useLongCalls() const { return UseLongCalls; }
   239:   bool useMemops() const { return UseMemops; }
   240:   bool usePackets() const { return UsePackets; }
   241:   bool useNewValueJumps() const { return UseNewValueJumps; }
   242:   bool useNewValueStores() const { return UseNewValueStores; }
   243:   bool useSmallData() const { return UseSmallData; }
   244:   bool useZRegOps() const { return UseZRegOps; }
   245:   bool useCabac() const { return UseCabac; }
   246: 
   247:   bool isTinyCore() const { return HexagonProcFamily == TinyCore; }
   248:   bool isTinyCoreWithDuplex() const { return isTinyCore() && EnableDuplex; }
   249: 
   250:   bool useHVXIEEEFPOps() const { return UseHVXIEEEFPOps && useHVXOps(); }
```
- EN: It declares or implements routines such as hasV71OpsOnly, hasV73Ops, hasV73OpsOnly, hasV75Ops, ... (25 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonHVXVersion, HexagonProcFamily, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 hasV71OpsOnly, hasV73Ops, hasV73OpsOnly, hasV75Ops, ... (25 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonHVXVersion, HexagonProcFamily，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251:   bool useHVXQFloatOps() const {
   252:     return UseHVXQFloatOps && HexagonHVXVersion >= Hexagon::ArchEnum::V68;
   253:   }
   254:   bool useHVXFloatingPoint() const { return UseHVXFloatingPoint; }
   255:   bool useHVXOps() const {
   256:     return HexagonHVXVersion > Hexagon::ArchEnum::NoArch;
   257:   }
   258:   bool useHVXV60Ops() const {
   259:     return HexagonHVXVersion >= Hexagon::ArchEnum::V60;
   260:   }
   261:   bool useHVXV62Ops() const {
   262:     return HexagonHVXVersion >= Hexagon::ArchEnum::V62;
   263:   }
   264:   bool useHVXV65Ops() const {
   265:     return HexagonHVXVersion >= Hexagon::ArchEnum::V65;
   266:   }
   267:   bool useHVXV66Ops() const {
   268:     return HexagonHVXVersion >= Hexagon::ArchEnum::V66;
   269:   }
   270:   bool useHVXV67Ops() const {
   271:     return HexagonHVXVersion >= Hexagon::ArchEnum::V67;
   272:   }
   273:   bool useHVXV68Ops() const {
   274:     return HexagonHVXVersion >= Hexagon::ArchEnum::V68;
   275:   }
   276:   bool useHVXV69Ops() const {
   277:     return HexagonHVXVersion >= Hexagon::ArchEnum::V69;
   278:   }
   279:   bool useHVXV71Ops() const {
   280:     return HexagonHVXVersion >= Hexagon::ArchEnum::V71;
   281:   }
   282:   bool useHVXV73Ops() const {
   283:     return HexagonHVXVersion >= Hexagon::ArchEnum::V73;
   284:   }
   285:   bool useHVX128BOps() const { return useHVXOps() && UseHVX128BOps; }
   286:   bool useHVX64BOps() const { return useHVXOps() && UseHVX64BOps; }
   287: 
   288:   bool hasMemNoShuf() const { return HasMemNoShuf; }
   289:   bool hasReservedR19() const { return ReservedR19; }
   290:   bool usePredicatedCalls() const;
   291: 
   292:   bool noreturnStackElim() const { return NoreturnStackElim; }
   293: 
   294:   bool useBSBScheduling() const { return UseBSBScheduling; }
   295:   bool enableMachineScheduler() const override;
   296: 
   297:   // Always use the TargetLowering default scheduler.
   298:   // FIXME: This will use the vliw scheduler which is probably just hurting
   299:   // compiler time and will be removed eventually anyway.
   300:   bool enableMachineSchedDefaultSched() const override { return false; }
```
- EN: It declares or implements routines such as useHVXQFloatOps, useHVXFloatingPoint, useHVXOps, useHVXV60Ops, ... (21 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonHVXVersion, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 useHVXQFloatOps, useHVXFloatingPoint, useHVXOps, useHVXV60Ops, ... (21 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonHVXVersion，说明了它与同级后端组件的连接关系。

### Lines 301-350 / 第 301-350 行

```cpp
   301: 
   302:   // For use with PostRAScheduling: get the anti-dependence breaking that should
   303:   // be performed before post-RA scheduling.
   304:   AntiDepBreakMode getAntiDepBreakMode() const override { return ANTIDEP_ALL; }
   305:   /// True if the subtarget should run a scheduler after register
   306:   /// allocation.
   307:   bool enablePostRAScheduler() const override { return true; }
   308: 
   309:   bool enableSubRegLiveness() const override;
   310: 
   311:   const std::string &getCPUString () const { return CPUString; }
   312: 
   313:   const Hexagon::ArchEnum &getHexagonArchVersion() const {
   314:     return HexagonArchVersion;
   315:   }
   316: 
   317:   void getPostRAMutations(
   318:       std::vector<std::unique_ptr<ScheduleDAGMutation>> &Mutations)
   319:       const override;
   320: 
   321:   void getSMSMutations(
   322:       std::vector<std::unique_ptr<ScheduleDAGMutation>> &Mutations)
   323:       const override;
   324: 
   325:   /// Enable use of alias analysis during code generation (during MI
   326:   /// scheduling, DAGCombine, etc.).
   327:   bool useAA() const override;
   328: 
   329:   /// Perform target specific adjustments to the latency of a schedule
   330:   /// dependency.
   331:   void adjustSchedDependency(SUnit *Def, int DefOpIdx, SUnit *Use, int UseOpIdx,
   332:                              SDep &Dep,
   333:                              const TargetSchedModel *SchedModel) const override;
   334: 
   335:   unsigned getVectorLength() const {
   336:     assert(useHVXOps());
   337:     if (useHVX64BOps())
   338:       return 64;
   339:     if (useHVX128BOps())
   340:       return 128;
   341:     llvm_unreachable("Invalid HVX vector length settings");
   342:   }
   343: 
   344:   ArrayRef<MVT> getHVXElementTypes() const {
   345:     static MVT Types[] = {MVT::i8, MVT::i16, MVT::i32};
   346:     static MVT TypesV68[] = {MVT::i8, MVT::i16, MVT::i32, MVT::f16, MVT::f32};
   347:     static MVT TypesV81[] = {MVT::i8,  MVT::i16,  MVT::i32,
   348:                              MVT::f16, MVT::bf16, MVT::f32};
   349: 
   350:     if (useHVXV81Ops() && useHVXFloatingPoint())
```
- EN: It declares or implements routines such as getAntiDepBreakMode, enablePostRAScheduler, enableSubRegLiveness, getCPUString, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonArchVersion, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getAntiDepBreakMode, enablePostRAScheduler, enableSubRegLiveness, getCPUString, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonArchVersion，说明了它与同级后端组件的连接关系。

### Lines 351-385 / 第 351-385 行

```cpp
   351:       return ArrayRef(TypesV81);
   352:     if (useHVXV68Ops() && useHVXFloatingPoint())
   353:       return ArrayRef(TypesV68);
   354:     return ArrayRef(Types);
   355:   }
   356: 
   357:   bool isHVXElementType(MVT Ty, bool IncludeBool = false) const;
   358:   bool isHVXVectorType(EVT VecTy, bool IncludeBool = false) const;
   359:   bool isTypeForHVX(Type *VecTy, bool IncludeBool = false) const;
   360: 
   361:   Align getTypeAlignment(MVT Ty) const {
   362:     if (isHVXVectorType(Ty, true))
   363:       return Align(getVectorLength());
   364:     return Align(std::max<unsigned>(1, Ty.getSizeInBits() / 8));
   365:   }
   366: 
   367:   unsigned getL1CacheLineSize() const;
   368:   unsigned getL1PrefetchDistance() const;
   369: 
   370:   Intrinsic::ID getIntrinsicId(unsigned Opc) const;
   371: 
   372: private:
   373:   // Helper function responsible for increasing the latency only.
   374:   int updateLatency(MachineInstr &SrcInst, MachineInstr &DstInst,
   375:                     bool IsArtificial, int Latency) const;
   376:   void restoreLatency(SUnit *Src, SUnit *Dst) const;
   377:   void changeLatency(SUnit *Src, SUnit *Dst, unsigned Lat) const;
   378:   bool isBestZeroLatency(SUnit *Src, SUnit *Dst, const HexagonInstrInfo *TII,
   379:                          SmallPtrSet<SUnit *, 4> &ExclSrc,
   380:                          SmallPtrSet<SUnit *, 4> &ExclDst) const;
   381: };
   382: 
   383: } // end namespace llvm
   384: 
   385: #endif // LLVM_LIB_TARGET_HEXAGON_HEXAGONSUBTARGET_H
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as ArrayRef, isHVXElementType, isHVXVectorType, isTypeForHVX, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 ArrayRef, isHVXElementType, isHVXVectorType, isTypeForHVX, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- VLIW packetization / VLIW 成包
- instruction scheduling models / 指令调度模型
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonDepArch.h, HexagonFrameLowering.h, HexagonISelLowering.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSelectionDAGInfo.h, llvm/ADT/SmallSet.h, llvm/ADT/StringRef.h, llvm/CodeGen/ScheduleDAGMutation.h, llvm/CodeGen/TargetSubtargetInfo.h, ... (16 total)`
- Hexagon symbols / Hexagon 符号: `HexagonSubtarget, HexagonDepArch, HexagonFrameLowering, HexagonISelLowering, HexagonInstrInfo, HexagonRegisterInfo, HexagonSelectionDAGInfo, HexagonGenSubtargetInfo, HexagonArchVersion, HexagonHVXVersion, ... (13 total)`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
