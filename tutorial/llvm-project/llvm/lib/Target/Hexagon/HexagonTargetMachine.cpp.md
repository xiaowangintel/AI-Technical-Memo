# HexagonTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonTargetMachine.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): #include "llvm/CodeGen/MIRParser/MIParser.h
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===-- HexagonTargetMachine.cpp - Define TargetMachine for Hexagon -------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // Implements the info about Hexagon target spec.
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #include "HexagonTargetMachine.h"
    14: #include "Hexagon.h"
    15: #include "HexagonISelLowering.h"
    16: #include "HexagonLoopIdiomRecognition.h"
    17: #include "HexagonMachineFunctionInfo.h"
    18: #include "HexagonMachineScheduler.h"
    19: #include "HexagonTargetObjectFile.h"
    20: #include "HexagonTargetTransformInfo.h"
    21: #include "HexagonVectorLoopCarriedReuse.h"
    22: #include "TargetInfo/HexagonTargetInfo.h"
    23: #include "llvm/CodeGen/MIRParser/MIParser.h"
    24: #include "llvm/CodeGen/Passes.h"
    25: #include "llvm/CodeGen/TargetPassConfig.h"
    26: #include "llvm/CodeGen/VLIWMachineScheduler.h"
    27: #include "llvm/MC/TargetRegistry.h"
    28: #include "llvm/Passes/PassBuilder.h"
    29: #include "llvm/Support/CommandLine.h"
    30: #include "llvm/Support/Compiler.h"
    31: #include "llvm/Transforms/Scalar.h"
    32: #include <optional>
    33: 
    34: using namespace llvm;
    35: 
    36: static cl::opt<bool>
    37:     EnableCExtOpt("hexagon-cext", cl::Hidden, cl::init(true),
    38:                   cl::desc("Enable Hexagon constant-extender optimization"));
    39: 
    40: static cl::opt<bool> EnableRDFOpt("rdf-opt", cl::Hidden, cl::init(true),
    41:                                   cl::desc("Enable RDF-based optimizations"));
    42: 
    43: cl::opt<unsigned> RDFFuncBlockLimit(
    44:     "rdf-bb-limit", cl::Hidden, cl::init(1000),
    45:     cl::desc("Basic block limit for a function for RDF optimizations"));
    46: 
    47: static cl::opt<bool>
    48:     DisableHardwareLoops("disable-hexagon-hwloops", cl::Hidden,
    49:                          cl::desc("Disable Hardware Loops for Hexagon target"));
    50: 
```
- EN: It imports headers such as HexagonTargetMachine.h, Hexagon.h, HexagonISelLowering.h, HexagonLoopIdiomRecognition.h, ... (20 total), establishing the LLVM/Hexagon APIs used below. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as EnableCExtOpt, EnableRDFOpt, RDFFuncBlockLimit, DisableHardwareLoops, translating Hexagon-specific policy into reusable code paths. Command-line options appear here to gate diagnostics or target-specific tuning behavior.
- CN: 这里引入了 HexagonTargetMachine.h, Hexagon.h, HexagonISelLowering.h, HexagonLoopIdiomRecognition.h, ... (20 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 EnableCExtOpt, EnableRDFOpt, RDFFuncBlockLimit, DisableHardwareLoops 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里出现了命令行选项，用于控制诊断开关或目标相关的调优行为。

### Lines 51-100 / 第 51-100 行

```cpp
    51: static cl::opt<bool> EnableMCR("hexagon-mcr", cl::Hidden, cl::init(true),
    52:                                cl::desc("Enable the machine combiner pass"));
    53: 
    54: static cl::opt<bool>
    55:     DisableAModeOpt("disable-hexagon-amodeopt", cl::Hidden,
    56:                     cl::desc("Disable Hexagon Addressing Mode Optimization"));
    57: 
    58: static cl::opt<bool>
    59:     DisableHexagonCFGOpt("disable-hexagon-cfgopt", cl::Hidden,
    60:                          cl::desc("Disable Hexagon CFG Optimization"));
    61: 
    62: static cl::opt<bool>
    63:     DisableHCP("disable-hcp", cl::Hidden,
    64:                cl::desc("Disable Hexagon constant propagation"));
    65: 
    66: static cl::opt<bool> DisableHexagonMask(
    67:     "disable-mask", cl::Hidden,
    68:     cl::desc("Disable Hexagon specific Mask generation pass"));
    69: 
    70: static cl::opt<bool> DisableHexagonLiveVars(
    71:     "disable-hlv", cl::Hidden,
    72:     cl::desc("Disable Hexagon specific post-RA live-variable analysis"));
    73: static cl::opt<bool> DisableStoreWidening("disable-store-widen", cl::Hidden,
    74:                                           cl::init(false),
    75:                                           cl::desc("Disable store widening"));
    76: 
    77: static cl::opt<bool> DisableLoadWidening("disable-load-widen", cl::Hidden,
    78:                                          cl::desc("Disable load widening"));
    79: 
    80: static cl::opt<bool> EnableExpandCondsets("hexagon-expand-condsets",
    81:                                           cl::init(true), cl::Hidden,
    82:                                           cl::desc("Early expansion of MUX"));
    83: 
    84: static cl::opt<bool> EnableTfrCleanup("hexagon-tfr-cleanup", cl::init(true),
    85:                                       cl::Hidden,
    86:                                       cl::desc("Cleanup of TFRs/COPYs"));
    87: 
    88: static cl::opt<bool> EnableEarlyIf("hexagon-eif", cl::init(true), cl::Hidden,
    89:                                    cl::desc("Enable early if-conversion"));
    90: 
    91: static cl::opt<bool> EnableCopyHoist("hexagon-copy-hoist", cl::init(true),
    92:                                      cl::Hidden,
    93:                                      cl::desc("Enable Hexagon copy hoisting"));
    94: 
    95: static cl::opt<bool>
    96:     EnableGenInsert("hexagon-insert", cl::init(true), cl::Hidden,
    97:                     cl::desc("Generate \"insert\" instructions"));
    98: 
    99: static cl::opt<bool>
   100:     EnableCommGEP("hexagon-commgep", cl::init(true), cl::Hidden,
```
- EN: It declares or implements routines such as EnableMCR, DisableAModeOpt, DisableHexagonCFGOpt, DisableHCP, ... (13 total), translating Hexagon-specific policy into reusable code paths. Command-line options appear here to gate diagnostics or target-specific tuning behavior.
- CN: 这里声明或实现了 EnableMCR, DisableAModeOpt, DisableHexagonCFGOpt, DisableHCP, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里出现了命令行选项，用于控制诊断开关或目标相关的调优行为。

### Lines 101-150 / 第 101-150 行

```cpp
   101:                   cl::desc("Enable commoning of GEP instructions"));
   102: 
   103: static cl::opt<bool>
   104:     EnableGenExtract("hexagon-extract", cl::init(true), cl::Hidden,
   105:                      cl::desc("Generate \"extract\" instructions"));
   106: 
   107: static cl::opt<bool> EnableGenMux(
   108:     "hexagon-mux", cl::init(true), cl::Hidden,
   109:     cl::desc("Enable converting conditional transfers into MUX instructions"));
   110: 
   111: static cl::opt<bool>
   112:     EnableGenPred("hexagon-gen-pred", cl::init(true), cl::Hidden,
   113:                   cl::desc("Enable conversion of arithmetic operations to "
   114:                            "predicate instructions"));
   115: 
   116: static cl::opt<bool>
   117:     EnableLoopPrefetch("hexagon-loop-prefetch", cl::Hidden,
   118:                        cl::desc("Enable loop data prefetch on Hexagon"));
   119: 
   120: static cl::opt<bool>
   121:     DisableHSDR("disable-hsdr", cl::init(false), cl::Hidden,
   122:                 cl::desc("Disable splitting double registers"));
   123: 
   124: static cl::opt<bool>
   125:     EnableGenMemAbs("hexagon-mem-abs", cl::init(true), cl::Hidden,
   126:                     cl::desc("Generate absolute set instructions"));
   127: 
   128: static cl::opt<bool> EnableBitSimplify("hexagon-bit", cl::init(true),
   129:                                        cl::Hidden,
   130:                                        cl::desc("Bit simplification"));
   131: 
   132: static cl::opt<bool> EnableLoopResched("hexagon-loop-resched", cl::init(true),
   133:                                        cl::Hidden,
   134:                                        cl::desc("Loop rescheduling"));
   135: 
   136: static cl::opt<bool> HexagonNoOpt("hexagon-noopt", cl::init(false), cl::Hidden,
   137:                                   cl::desc("Disable backend optimizations"));
   138: 
   139: static cl::opt<bool>
   140:     EnableVectorPrint("enable-hexagon-vector-print", cl::Hidden,
   141:                       cl::desc("Enable Hexagon Vector print instr pass"));
   142: 
   143: static cl::opt<bool>
   144:     EnableVExtractOpt("hexagon-opt-vextract", cl::Hidden, cl::init(true),
   145:                       cl::desc("Enable vextract optimization"));
   146: 
   147: static cl::opt<bool>
   148:     EnableVectorCombine("hexagon-vector-combine", cl::Hidden, cl::init(true),
   149:                         cl::desc("Enable HVX vector combining"));
   150: 
```
- EN: It declares or implements routines such as cl::desc, EnableGenExtract, EnableGenMux, EnableGenPred, ... (13 total), translating Hexagon-specific policy into reusable code paths. Command-line options appear here to gate diagnostics or target-specific tuning behavior. Notable Hexagon symbols referenced here include HexagonNoOpt, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 cl::desc, EnableGenExtract, EnableGenMux, EnableGenPred, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里出现了命令行选项，用于控制诊断开关或目标相关的调优行为。 这里引用的重要 Hexagon 符号包括 HexagonNoOpt，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151: static cl::opt<bool> EnableInitialCFGCleanup(
   152:     "hexagon-initial-cfg-cleanup", cl::Hidden, cl::init(true),
   153:     cl::desc("Simplify the CFG after atomic expansion pass"));
   154: 
   155: static cl::opt<bool> EnableInstSimplify("hexagon-instsimplify", cl::Hidden,
   156:                                         cl::init(true),
   157:                                         cl::desc("Enable instsimplify"));
   158: 
   159: /// HexagonTargetMachineModule - Note that this is used on hosts that
   160: /// cannot link in a library unless there are references into the
   161: /// library.  In particular, it seems that it is not possible to get
   162: /// things to work on Win32 without this.  Though it is unused, do not
   163: /// remove it.
   164: extern "C" int HexagonTargetMachineModule;
   165: int HexagonTargetMachineModule = 0;
   166: 
   167: static ScheduleDAGInstrs *createVLIWMachineSched(MachineSchedContext *C) {
   168:   ScheduleDAGMILive *DAG = new VLIWMachineScheduler(
   169:       C, std::make_unique<HexagonConvergingVLIWScheduler>());
   170:   DAG->addMutation(std::make_unique<HexagonSubtarget::UsrOverflowMutation>());
   171:   DAG->addMutation(std::make_unique<HexagonSubtarget::HVXMemLatencyMutation>());
   172:   DAG->addMutation(std::make_unique<HexagonSubtarget::CallMutation>());
   173:   DAG->addMutation(createCopyConstrainDAGMutation(DAG->TII, DAG->TRI));
   174:   return DAG;
   175: }
   176: 
   177: static MachineSchedRegistry
   178:     SchedCustomRegistry("hexagon", "Run Hexagon's custom scheduler",
   179:                         createVLIWMachineSched);
   180: 
   181: static Reloc::Model getEffectiveRelocModel(std::optional<Reloc::Model> RM) {
   182:   return RM.value_or(Reloc::Static);
   183: }
   184: 
   185: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
   186: LLVMInitializeHexagonTarget() {
   187:   // Register the target.
   188:   RegisterTargetMachine<HexagonTargetMachine> X(getTheHexagonTarget());
   189: 
   190:   PassRegistry &PR = *PassRegistry::getPassRegistry();
   191:   initializeHexagonAsmPrinterPass(PR);
   192:   initializeHexagonBitSimplifyPass(PR);
   193:   initializeHexagonConstExtendersPass(PR);
   194:   initializeHexagonConstPropagationPass(PR);
   195:   initializeHexagonCopyToCombinePass(PR);
   196:   initializeHexagonEarlyIfConversionPass(PR);
   197:   initializeHexagonGenMemAbsolutePass(PR);
   198:   initializeHexagonGenMuxPass(PR);
   199:   initializeHexagonGlobalSchedulerPass(PR);
   200:   initializeHexagonLiveVariablesPass(PR);
```
- EN: It declares or implements routines such as EnableInitialCFGCleanup, EnableInstSimplify, createVLIWMachineSched, VLIWMachineScheduler, ... (21 total), translating Hexagon-specific policy into reusable code paths. Command-line options appear here to gate diagnostics or target-specific tuning behavior. Notable Hexagon symbols referenced here include HexagonTargetMachineModule, HexagonConvergingVLIWScheduler, HexagonSubtarget, HexagonTargetMachine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 EnableInitialCFGCleanup, EnableInstSimplify, createVLIWMachineSched, VLIWMachineScheduler, ... (21 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里出现了命令行选项，用于控制诊断开关或目标相关的调优行为。 这里引用的重要 Hexagon 符号包括 HexagonTargetMachineModule, HexagonConvergingVLIWScheduler, HexagonSubtarget, HexagonTargetMachine，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```cpp
   201:   initializeHexagonHardwareLoopsPass(PR);
   202:   initializeHexagonLoopIdiomRecognizeLegacyPassPass(PR);
   203:   initializeHexagonNewValueJumpPass(PR);
   204:   initializeHexagonOptAddrModePass(PR);
   205:   initializeHexagonPacketizerPass(PR);
   206:   initializeHexagonRDFOptPass(PR);
   207:   initializeHexagonSplitDoubleRegsPass(PR);
   208:   initializeHexagonVectorCombineLegacyPass(PR);
   209:   initializeHexagonVectorLoopCarriedReuseLegacyPassPass(PR);
   210:   initializeHexagonVExtractPass(PR);
   211:   initializeHexagonDAGToDAGISelLegacyPass(PR);
   212:   initializeHexagonLoopReschedulingPass(PR);
   213:   initializeHexagonBranchRelaxationPass(PR);
   214:   initializeHexagonCFGOptimizerPass(PR);
   215:   initializeHexagonCommonGEPPass(PR);
   216:   initializeHexagonCopyHoistingPass(PR);
   217:   initializeHexagonExpandCondsetsPass(PR);
   218:   initializeHexagonLoopAlignPass(PR);
   219:   initializeHexagonTfrCleanupPass(PR);
   220:   initializeHexagonFixupHwLoopsPass(PR);
   221:   initializeHexagonCallFrameInformationPass(PR);
   222:   initializeHexagonGenExtractPass(PR);
   223:   initializeHexagonGenInsertPass(PR);
   224:   initializeHexagonGenPredicatePass(PR);
   225:   initializeHexagonLoadWideningPass(PR);
   226:   initializeHexagonStoreWideningPass(PR);
   227:   initializeHexagonMaskPass(PR);
   228:   initializeHexagonOptimizeSZextendsPass(PR);
   229:   initializeHexagonPeepholePass(PR);
   230:   initializeHexagonSplitConst32AndConst64Pass(PR);
   231:   initializeHexagonVectorPrintPass(PR);
   232:   initializeHexagonQFPOptimizerPass(PR);
   233: }
   234: 
   235: HexagonTargetMachine::HexagonTargetMachine(const Target &T, const Triple &TT,
   236:                                            StringRef CPU, StringRef FS,
   237:                                            const TargetOptions &Options,
   238:                                            std::optional<Reloc::Model> RM,
   239:                                            std::optional<CodeModel::Model> CM,
   240:                                            CodeGenOptLevel OL, bool JIT)
   241:     // Specify the vector alignment explicitly. For v512x1, the calculated
   242:     // alignment would be 512*alignment(i1), which is 512 bytes, instead of
   243:     // the required minimum of 64 bytes.
   244:     : CodeGenTargetMachineImpl(T, TT.computeDataLayout(), TT, CPU, FS, Options,
   245:                                getEffectiveRelocModel(RM),
   246:                                getEffectiveCodeModel(CM, CodeModel::Small),
   247:                                (HexagonNoOpt ? CodeGenOptLevel::None : OL)),
   248:       TLOF(std::make_unique<HexagonTargetObjectFile>()),
   249:       Subtarget(Triple(TT), CPU, FS, *this) {
   250:   initAsmInfo();
```
- EN: It declares or implements routines such as initializeHexagonHardwareLoopsPass, initializeHexagonLoopIdiomRecognizeLegacyPassPass, initializeHexagonNewValueJumpPass, initializeHexagonOptAddrModePass, ... (34 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonTargetMachine, HexagonNoOpt, HexagonTargetObjectFile, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 initializeHexagonHardwareLoopsPass, initializeHexagonLoopIdiomRecognizeLegacyPassPass, initializeHexagonNewValueJumpPass, initializeHexagonOptAddrModePass, ... (34 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonTargetMachine, HexagonNoOpt, HexagonTargetObjectFile，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251: }
   252: 
   253: const HexagonSubtarget *
   254: HexagonTargetMachine::getSubtargetImpl(const Function &F) const {
   255:   AttributeList FnAttrs = F.getAttributes();
   256:   Attribute CPUAttr = FnAttrs.getFnAttr("target-cpu");
   257:   Attribute FSAttr = FnAttrs.getFnAttr("target-features");
   258: 
   259:   std::string CPU =
   260:       CPUAttr.isValid() ? CPUAttr.getValueAsString().str() : TargetCPU;
   261:   std::string FS =
   262:       FSAttr.isValid() ? FSAttr.getValueAsString().str() : TargetFS;
   263: 
   264:   auto &I = SubtargetMap[CPU + FS];
   265:   if (!I) {
   266:     // This needs to be done before we create a new subtarget since any
   267:     // creation will depend on the TM and the code generation flags on the
   268:     // function that reside in TargetOptions.
   269:     resetTargetOptions(F);
   270:     I = std::make_unique<HexagonSubtarget>(TargetTriple, CPU, FS, *this);
   271:   }
   272:   return I.get();
   273: }
   274: 
   275: void HexagonTargetMachine::registerPassBuilderCallbacks(PassBuilder &PB) {
   276: #define GET_PASS_REGISTRY "HexagonPassRegistry.def"
   277: #include "llvm/Passes/TargetPassRegistry.inc"
   278: 
   279:   PB.registerLateLoopOptimizationsEPCallback(
   280:       [=](LoopPassManager &LPM, OptimizationLevel Level) {
   281:         if (Level.getSpeedupLevel() > 0)
   282:           LPM.addPass(HexagonLoopIdiomRecognitionPass());
   283:       });
   284:   PB.registerLoopOptimizerEndEPCallback(
   285:       [=](LoopPassManager &LPM, OptimizationLevel Level) {
   286:         if (Level.getSpeedupLevel() > 0)
   287:           LPM.addPass(HexagonVectorLoopCarriedReusePass());
   288:       });
   289: }
   290: 
   291: TargetTransformInfo
   292: HexagonTargetMachine::getTargetTransformInfo(const Function &F) const {
   293:   return TargetTransformInfo(std::make_unique<HexagonTTIImpl>(this, F));
   294: }
   295: 
   296: MachineFunctionInfo *HexagonTargetMachine::createMachineFunctionInfo(
   297:     BumpPtrAllocator &Allocator, const Function &F,
   298:     const TargetSubtargetInfo *STI) const {
   299:   return HexagonMachineFunctionInfo::create<HexagonMachineFunctionInfo>(
   300:       Allocator, F, STI);
```
- EN: It imports headers such as llvm/Passes/TargetPassRegistry.inc, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as HexagonTargetMachine::getSubtargetImpl, getAttributes, getFnAttr, resetTargetOptions, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里引入了 llvm/Passes/TargetPassRegistry.inc 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 HexagonTargetMachine::getSubtargetImpl, getAttributes, getFnAttr, resetTargetOptions, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 301-350 / 第 301-350 行

```cpp
   301: }
   302: 
   303: yaml::MachineFunctionInfo *
   304: HexagonTargetMachine::createDefaultFuncInfoYAML() const {
   305:   return new yaml::HexagonFunctionInfo();
   306: }
   307: 
   308: yaml::MachineFunctionInfo *
   309: HexagonTargetMachine::convertFuncInfoToYAML(const MachineFunction &MF) const {
   310:   const auto *MFI = MF.getInfo<HexagonMachineFunctionInfo>();
   311:   const auto &TRI = *MF.getSubtarget().getRegisterInfo();
   312:   return new yaml::HexagonFunctionInfo(*MFI, TRI);
   313: }
   314: 
   315: bool HexagonTargetMachine::parseMachineFunctionInfo(
   316:     const yaml::MachineFunctionInfo &MFI_, PerFunctionMIParsingState &PFS,
   317:     SMDiagnostic &Error, SMRange &SourceRange) const {
   318:   const auto &YamlMFI = static_cast<const yaml::HexagonFunctionInfo &>(MFI_);
   319:   MachineFunction &MF = PFS.MF;
   320:   HexagonMachineFunctionInfo *MFI = MF.getInfo<HexagonMachineFunctionInfo>();
   321: 
   322:   MFI->initializeBaseYamlFields(YamlMFI);
   323: 
   324:   // Parse StackAlignBaseReg register name
   325:   if (!YamlMFI.StackAlignBaseReg.Value.empty()) {
   326:     Register Reg;
   327:     if (parseNamedRegisterReference(PFS, Reg, YamlMFI.StackAlignBaseReg.Value,
   328:                                     Error)) {
   329:       SourceRange = YamlMFI.StackAlignBaseReg.SourceRange;
   330:       return true;
   331:     }
   332:     MFI->setStackAlignBaseReg(Reg);
   333:   }
   334: 
   335:   return false;
   336: }
   337: 
   338: HexagonTargetMachine::~HexagonTargetMachine() = default;
   339: 
   340: ScheduleDAGInstrs *
   341: HexagonTargetMachine::createMachineScheduler(MachineSchedContext *C) const {
   342:   return createVLIWMachineSched(C);
   343: }
   344: 
   345: namespace {
   346: /// Hexagon Code Generator Pass Configuration Options.
   347: class HexagonPassConfig : public TargetPassConfig {
   348: public:
   349:   HexagonPassConfig(HexagonTargetMachine &TM, PassManagerBase &PM)
   350:       : TargetPassConfig(TM, PM) {}
```
- EN: It declares types such as HexagonPassConfig, which carry the state or API of this component. It defines declarative TableGen records like HexagonPassConfig; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as HexagonTargetMachine::createDefaultFuncInfoYAML, yaml::HexagonFunctionInfo, HexagonTargetMachine::convertFuncInfoToYAML, getInfo<HexagonMachineFunctionInfo>, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 HexagonPassConfig 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonPassConfig 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 HexagonTargetMachine::createDefaultFuncInfoYAML, yaml::HexagonFunctionInfo, HexagonTargetMachine::convertFuncInfoToYAML, getInfo<HexagonMachineFunctionInfo>, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 351-400 / 第 351-400 行

```cpp
   351: 
   352:   HexagonTargetMachine &getHexagonTargetMachine() const {
   353:     return getTM<HexagonTargetMachine>();
   354:   }
   355: 
   356:   void addIRPasses() override;
   357:   bool addInstSelector() override;
   358:   bool addILPOpts() override;
   359:   void addPreRegAlloc() override;
   360:   void addPostRegAlloc() override;
   361:   void addPreSched2() override;
   362:   void addPreEmitPass() override;
   363: };
   364: } // namespace
   365: 
   366: TargetPassConfig *HexagonTargetMachine::createPassConfig(PassManagerBase &PM) {
   367:   return new HexagonPassConfig(*this, PM);
   368: }
   369: 
   370: void HexagonPassConfig::addIRPasses() {
   371:   TargetPassConfig::addIRPasses();
   372:   bool NoOpt = (getOptLevel() == CodeGenOptLevel::None);
   373: 
   374:   if (!NoOpt) {
   375:     if (EnableInstSimplify)
   376:       addPass(createInstSimplifyLegacyPass());
   377:     addPass(createDeadCodeEliminationPass());
   378:   }
   379: 
   380:   addPass(createAtomicExpandLegacyPass());
   381: 
   382:   if (!NoOpt) {
   383:     if (EnableInitialCFGCleanup)
   384:       addPass(createCFGSimplificationPass(SimplifyCFGOptions()
   385:                                               .forwardSwitchCondToPhi(true)
   386:                                               .convertSwitchRangeToICmp(true)
   387:                                               .convertSwitchToLookupTable(true)
   388:                                               .needCanonicalLoops(false)
   389:                                               .hoistCommonInsts(true)
   390:                                               .sinkCommonInsts(true)));
   391:     if (EnableLoopPrefetch)
   392:       addPass(createLoopDataPrefetchPass());
   393:     if (EnableVectorCombine)
   394:       addPass(createHexagonVectorCombineLegacyPass());
   395:     if (EnableCommGEP)
   396:       addPass(createHexagonCommonGEP());
   397:     // Replace certain combinations of shifts and ands with extracts.
   398:     if (EnableGenExtract)
   399:       addPass(createHexagonGenExtract());
   400:   }
```
- EN: It opens namespaces (TargetPassConfig) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as getHexagonTargetMachine, getTM<HexagonTargetMachine>, addIRPasses, addInstSelector, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTargetMachine, HexagonPassConfig, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（TargetPassConfig），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 getHexagonTargetMachine, getTM<HexagonTargetMachine>, addIRPasses, addInstSelector, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTargetMachine, HexagonPassConfig，说明了它与同级后端组件的连接关系。

### Lines 401-450 / 第 401-450 行

```cpp
   401: }
   402: 
   403: bool HexagonPassConfig::addInstSelector() {
   404:   HexagonTargetMachine &TM = getHexagonTargetMachine();
   405:   bool NoOpt = (getOptLevel() == CodeGenOptLevel::None);
   406: 
   407:   if (!NoOpt)
   408:     addPass(createHexagonOptimizeSZextends());
   409: 
   410:   addPass(createHexagonISelDag(TM, getOptLevel()));
   411: 
   412:   if (!NoOpt) {
   413:     if (EnableVExtractOpt)
   414:       addPass(createHexagonVExtract());
   415:     // Create logical operations on predicate registers.
   416:     if (EnableGenPred)
   417:       addPass(createHexagonGenPredicate());
   418:     // Rotate loops to expose bit-simplification opportunities.
   419:     if (EnableLoopResched)
   420:       addPass(createHexagonLoopRescheduling());
   421:     // Split double registers.
   422:     if (!DisableHSDR)
   423:       addPass(createHexagonSplitDoubleRegs());
   424:     // Bit simplification.
   425:     if (EnableBitSimplify)
   426:       addPass(createHexagonBitSimplify());
   427:     addPass(createHexagonPeephole());
   428:     // Constant propagation.
   429:     if (!DisableHCP) {
   430:       addPass(createHexagonConstPropagationPass());
   431:       addPass(&UnreachableMachineBlockElimID);
   432:     }
   433:     if (EnableGenInsert)
   434:       addPass(createHexagonGenInsert());
   435:     if (EnableEarlyIf)
   436:       addPass(createHexagonEarlyIfConversion());
   437:     addPass(createHexagonQFPOptimizer());
   438:   }
   439: 
   440:   return false;
   441: }
   442: 
   443: bool HexagonPassConfig::addILPOpts() {
   444:   if (EnableMCR)
   445:     addPass(&MachineCombinerID);
   446: 
   447:   return true;
   448: }
   449: 
   450: void HexagonPassConfig::addPreRegAlloc() {
```
- EN: It declares or implements routines such as HexagonPassConfig::addInstSelector, getHexagonTargetMachine, getOptLevel, addPass, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonPassConfig, HexagonTargetMachine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonPassConfig::addInstSelector, getHexagonTargetMachine, getOptLevel, addPass, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonPassConfig, HexagonTargetMachine，说明了它与同级后端组件的连接关系。

### Lines 451-500 / 第 451-500 行

```cpp
   451:   if (getOptLevel() != CodeGenOptLevel::None) {
   452:     if (EnableCExtOpt)
   453:       addPass(createHexagonConstExtenders());
   454:     if (EnableExpandCondsets)
   455:       insertPass(&RegisterCoalescerID, &HexagonExpandCondsetsID);
   456:     if (EnableCopyHoist)
   457:       insertPass(&RegisterCoalescerID, &HexagonCopyHoistingID);
   458:     if (EnableTfrCleanup)
   459:       insertPass(&VirtRegRewriterID, &HexagonTfrCleanupID);
   460:     if (!DisableStoreWidening)
   461:       addPass(createHexagonStoreWidening());
   462:     if (!DisableLoadWidening)
   463:       addPass(createHexagonLoadWidening());
   464:     if (EnableGenMemAbs)
   465:       addPass(createHexagonGenMemAbsolute());
   466:     if (!DisableHardwareLoops)
   467:       addPass(createHexagonHardwareLoops());
   468:   }
   469:   if (TM->getOptLevel() >= CodeGenOptLevel::Default)
   470:     addPass(&MachinePipelinerID);
   471: }
   472: 
   473: void HexagonPassConfig::addPostRegAlloc() {
   474:   if (getOptLevel() != CodeGenOptLevel::None) {
   475:     if (EnableRDFOpt)
   476:       addPass(createHexagonRDFOpt());
   477:     if (!DisableHexagonCFGOpt)
   478:       addPass(createHexagonCFGOptimizer());
   479:     if (!DisableAModeOpt)
   480:       addPass(createHexagonOptAddrMode());
   481:   }
   482: }
   483: 
   484: void HexagonPassConfig::addPreSched2() {
   485:   bool NoOpt = (getOptLevel() == CodeGenOptLevel::None);
   486:   addPass(createHexagonCopyToCombine());
   487:   if (getOptLevel() != CodeGenOptLevel::None)
   488:     addPass(&IfConverterID);
   489:   addPass(createHexagonSplitConst32AndConst64());
   490:   if (!NoOpt && !DisableHexagonMask)
   491:     addPass(createHexagonMask());
   492: 
   493:   if (!NoOpt && !DisableHexagonLiveVars) {
   494:     addPass(&HexagonLiveVariablesID);
   495:   }
   496: }
   497: 
   498: void HexagonPassConfig::addPreEmitPass() {
   499:   bool NoOpt = (getOptLevel() == CodeGenOptLevel::None);
   500: 
```
- EN: It declares or implements routines such as HexagonPassConfig::addPostRegAlloc, HexagonPassConfig::addPreSched2, getOptLevel, addPass, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonExpandCondsetsID, HexagonCopyHoistingID, HexagonTfrCleanupID, HexagonPassConfig, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonPassConfig::addPostRegAlloc, HexagonPassConfig::addPreSched2, getOptLevel, addPass, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonExpandCondsetsID, HexagonCopyHoistingID, HexagonTfrCleanupID, HexagonPassConfig, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 501-531 / 第 501-531 行

```cpp
   501:   if (!NoOpt)
   502:     addPass(createHexagonNewValueJump());
   503: 
   504:   addPass(createHexagonBranchRelaxation());
   505: 
   506:   if (!NoOpt) {
   507:     if (!DisableHardwareLoops)
   508:       addPass(createHexagonFixupHwLoops());
   509:     // Generate MUX from pairs of conditional transfers.
   510:     if (EnableGenMux)
   511:       addPass(createHexagonGenMux());
   512:     if (!DisableHexagonLiveVars)
   513:       addPass(&HexagonLiveVariablesID);
   514:   }
   515: 
   516:   // Packetization is mandatory: it handles gather/scatter at all opt levels.
   517:   addPass(createHexagonPacketizer(NoOpt));
   518: 
   519:   if (!NoOpt) {
   520:     // Global pull-up scheduler
   521:     addPass(createHexagonGlobalScheduler());
   522: 
   523:     addPass(createHexagonLoopAlign());
   524:   }
   525: 
   526:   if (EnableVectorPrint)
   527:     addPass(createHexagonVectorPrint());
   528: 
   529:   // Add CFI instructions if necessary.
   530:   addPass(createHexagonCallFrameInformation());
   531: }
```
- EN: It declares or implements routines such as addPass, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonLiveVariablesID, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 addPass 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonLiveVariablesID，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- instruction scheduling models / 指令调度模型
- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- MC-layer target description / MC 层目标描述
- target pipeline configuration / 目标流水线配置

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonTargetMachine.h, Hexagon.h, HexagonISelLowering.h, HexagonLoopIdiomRecognition.h, HexagonMachineFunctionInfo.h, HexagonMachineScheduler.h, HexagonTargetObjectFile.h, HexagonTargetTransformInfo.h, HexagonVectorLoopCarriedReuse.h, TargetInfo/HexagonTargetInfo.h, ... (21 total)`
- Hexagon symbols / Hexagon 符号: `HexagonTargetMachine, HexagonISelLowering, HexagonLoopIdiomRecognition, HexagonMachineFunctionInfo, HexagonMachineScheduler, HexagonTargetObjectFile, HexagonTargetTransformInfo, HexagonVectorLoopCarriedReuse, HexagonTargetInfo, HexagonNoOpt, ... (23 total)`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
