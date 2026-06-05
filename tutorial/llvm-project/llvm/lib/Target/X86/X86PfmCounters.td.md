# X86PfmCounters.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86PfmCounters.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines X86 target definitions for the core X86 backend using LLVM TableGen DSL. / 使用 LLVM TableGen DSL 为X86 后端核心定义X86 目标定义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```tablegen
//===-- X86PfmCounters.td - X86 Hardware Counters ----------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This describes the available hardware counters for various subtargets.
//
//===----------------------------------------------------------------------===//

def UnhaltedCoreCyclesPfmCounter : PfmCounter<"unhalted_core_cycles">;
def UopsIssuedPfmCounter : PfmCounter<"uops_issued:any">;

// No default counters on X86.
def DefaultPfmCounters : ProcPfmCounters {}
def : PfmCountersDefaultBinding<DefaultPfmCounters>;

// Intel X86 Counters.
defvar DefaultIntelPfmValidationCounters = [
  PfmValidationCounter<InstructionRetired, "INSTRUCTIONS_RETIRED">,
  PfmValidationCounter<L1DCacheLoadMiss, "MEM_LOAD_UOPS_RETIRED:L1_MISS">,
  PfmValidationCounter<L1ICacheLoadMiss, "L1-ICACHE-LOAD-MISSES">,
  PfmValidationCounter<DataTLBLoadMiss, "DTLB_LOAD_MISSES:MISS_CAUSES_A_WALK">,
  PfmValidationCounter<DataTLBStoreMiss, "DTLB_STORE_MISSES:MISS_CAUSES_A_WALK">,
  PfmValidationCounter<InstructionTLBLoadMiss, "ITLB_MISSES:MISS_CAUSES_A_WALK">,
  PfmValidationCounter<BranchPredictionMiss, "BRANCH-MISSES">
];

```
**EN:** This section uses LLVM TableGen DSL to describe file header, licensing, and opening context for the core X86 backend. Key symbols include UnhaltedCoreCyclesPfmCounter, UopsIssuedPfmCounter, DefaultPfmCounters. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的文件头、许可证与开场上下文。关键符号包括 UnhaltedCoreCyclesPfmCounter, UopsIssuedPfmCounter, DefaultPfmCounters。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 31-60: TableGen classes, records, and target metadata / TableGen 类、记录与目标元数据
```tablegen
def PentiumPfmCounters : ProcPfmCounters {
  let CycleCounter = PfmCounter<"cpu_clk_unhalted">;
  let UopsCounter = PfmCounter<"uops_retired">;
}
def : PfmCountersBinding<"pentiumpro", PentiumPfmCounters>;
def : PfmCountersBinding<"pentium2", PentiumPfmCounters>;
def : PfmCountersBinding<"pentium3", PentiumPfmCounters>;
def : PfmCountersBinding<"pentium3m", PentiumPfmCounters>;
def : PfmCountersBinding<"pentium-m", PentiumPfmCounters>;

def CorePfmCounters : ProcPfmCounters {
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = PfmCounter<"uops_retired:any">;
}
def : PfmCountersBinding<"yonah", CorePfmCounters>;
def : PfmCountersBinding<"prescott", CorePfmCounters>;

def AtomPfmCounters : ProcPfmCounters {
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = PfmCounter<"uops_retired:any">;
}
def : PfmCountersBinding<"bonnell", AtomPfmCounters>;
def : PfmCountersBinding<"atom", AtomPfmCounters>;

def SLMPfmCounters : ProcPfmCounters {
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = PfmCounter<"uops_retired:any">;
  let IssueCounters = [
    PfmIssueCounter<"SLM_MEC_RSV", "mem_uop_retired:any_ld + mem_uop_retired:any_st">
  ];
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen classes, records, and target metadata for the core X86 backend. Key symbols include PentiumPfmCounters, CorePfmCounters, AtomPfmCounters, SLMPfmCounters. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 类、记录与目标元数据。关键符号包括 PentiumPfmCounters, CorePfmCounters, AtomPfmCounters, SLMPfmCounters。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 61-90: TableGen classes, records, and target metadata / TableGen 类、记录与目标元数据
```tablegen
}
def : PfmCountersBinding<"silvermont", SLMPfmCounters>;
def : PfmCountersBinding<"goldmont", SLMPfmCounters>;
def : PfmCountersBinding<"goldmont-plus", SLMPfmCounters>;
def : PfmCountersBinding<"tremont", SLMPfmCounters>;

def KnightPfmCounters : ProcPfmCounters {
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = PfmCounter<"uops_retired:all">;
}
def : PfmCountersBinding<"knl", KnightPfmCounters>;
def : PfmCountersBinding<"knm", KnightPfmCounters>;

def Core2PfmCounters : ProcPfmCounters {
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = PfmCounter<"uops_retired:any">;
  let IssueCounters = [
    PfmIssueCounter<"SBPort0",  "rs_uops_dispatched_cycles:port_0">,
    PfmIssueCounter<"SBPort1",  "rs_uops_dispatched_cycles:port_1">,
    PfmIssueCounter<"SBPort23", "rs_uops_dispatched_cycles:port_2 + rs_uops_dispatched_cycles:port_3">,
    PfmIssueCounter<"SBPort4",  "rs_uops_dispatched_cycles:port_4">,
    PfmIssueCounter<"SBPort5",  "rs_uops_dispatched_cycles:port_5">
  ];
}
def : PfmCountersBinding<"core2", Core2PfmCounters>;
def : PfmCountersBinding<"penryn", Core2PfmCounters>;

def NehalemPfmCounters : ProcPfmCounters {
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = PfmCounter<"uops_retired:any">;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen classes, records, and target metadata for the core X86 backend. Key symbols include KnightPfmCounters, Core2PfmCounters, NehalemPfmCounters. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 类、记录与目标元数据。关键符号包括 KnightPfmCounters, Core2PfmCounters, NehalemPfmCounters。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 91-120: TableGen classes, records, and target metadata / TableGen 类、记录与目标元数据
```tablegen
  let IssueCounters = [
    PfmIssueCounter<"SBPort0",  "uops_executed:port0">,
    PfmIssueCounter<"SBPort1",  "uops_executed:port1">,
    PfmIssueCounter<"SBPort23", "uops_executed:port2_core + uops_executed:port3_core">,
    PfmIssueCounter<"SBPort4",  "uops_executed:port4_core">,
    PfmIssueCounter<"SBPort5",  "uops_executed:port5">
  ];
}
def : PfmCountersBinding<"nehalem", NehalemPfmCounters>;
def : PfmCountersBinding<"corei7", NehalemPfmCounters>;
def : PfmCountersBinding<"westmere", NehalemPfmCounters>;

def SandyBridgePfmCounters : ProcPfmCounters {
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = UopsIssuedPfmCounter;
  let IssueCounters = [
    PfmIssueCounter<"SBPort0",  "uops_dispatched_port:port_0">,
    PfmIssueCounter<"SBPort1",  "uops_dispatched_port:port_1">,
    PfmIssueCounter<"SBPort23", "uops_dispatched_port:port_2 + uops_dispatched_port:port_3">,
    PfmIssueCounter<"SBPort4",  "uops_dispatched_port:port_4">,
    PfmIssueCounter<"SBPort5",  "uops_dispatched_port:port_5">
  ];
  let ValidationCounters = DefaultIntelPfmValidationCounters;
}
def : PfmCountersBinding<"sandybridge", SandyBridgePfmCounters>;
def : PfmCountersBinding<"ivybridge", SandyBridgePfmCounters>;

def HaswellPfmCounters : ProcPfmCounters {
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = UopsIssuedPfmCounter;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen classes, records, and target metadata for the core X86 backend. Key symbols include SandyBridgePfmCounters, HaswellPfmCounters. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 类、记录与目标元数据。关键符号包括 SandyBridgePfmCounters, HaswellPfmCounters。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 121-150: TableGen classes, records, and target metadata / TableGen 类、记录与目标元数据
```tablegen
  let IssueCounters = [
    PfmIssueCounter<"HWPort0", "uops_executed_port:port_0">,
    PfmIssueCounter<"HWPort1", "uops_executed_port:port_1">,
    PfmIssueCounter<"HWPort2", "uops_executed_port:port_2">,
    PfmIssueCounter<"HWPort3", "uops_executed_port:port_3">,
    PfmIssueCounter<"HWPort4", "uops_executed_port:port_4">,
    PfmIssueCounter<"HWPort5", "uops_executed_port:port_5">,
    PfmIssueCounter<"HWPort6", "uops_executed_port:port_6">,
    PfmIssueCounter<"HWPort7", "uops_executed_port:port_7">
  ];
  let ValidationCounters = DefaultIntelPfmValidationCounters;
}
def : PfmCountersBinding<"haswell", HaswellPfmCounters>;

def BroadwellPfmCounters : ProcPfmCounters {
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = UopsIssuedPfmCounter;
  let IssueCounters = [
    PfmIssueCounter<"BWPort0", "uops_executed_port:port_0">,
    PfmIssueCounter<"BWPort1", "uops_executed_port:port_1">,
    PfmIssueCounter<"BWPort2", "uops_executed_port:port_2">,
    PfmIssueCounter<"BWPort3", "uops_executed_port:port_3">,
    PfmIssueCounter<"BWPort4", "uops_executed_port:port_4">,
    PfmIssueCounter<"BWPort5", "uops_executed_port:port_5">,
    PfmIssueCounter<"BWPort6", "uops_executed_port:port_6">,
    PfmIssueCounter<"BWPort7", "uops_executed_port:port_7">
  ];
  let ValidationCounters = DefaultIntelPfmValidationCounters;
}
def : PfmCountersBinding<"broadwell", BroadwellPfmCounters>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen classes, records, and target metadata for the core X86 backend. Key symbols include BroadwellPfmCounters. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 类、记录与目标元数据。关键符号包括 BroadwellPfmCounters。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 151-180: TableGen classes, records, and target metadata / TableGen 类、记录与目标元数据
```tablegen

def SkylakeClientPfmCounters : ProcPfmCounters {
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = UopsIssuedPfmCounter;
  let IssueCounters = [
    PfmIssueCounter<"SKLPort0", "uops_dispatched_port:port_0">,
    PfmIssueCounter<"SKLPort1", "uops_dispatched_port:port_1">,
    PfmIssueCounter<"SKLPort2", "uops_dispatched_port:port_2">,
    PfmIssueCounter<"SKLPort3", "uops_dispatched_port:port_3">,
    PfmIssueCounter<"SKLPort4", "uops_dispatched_port:port_4">,
    PfmIssueCounter<"SKLPort5", "uops_dispatched_port:port_5">,
    PfmIssueCounter<"SKLPort6", "uops_dispatched_port:port_6">,
    PfmIssueCounter<"SKLPort7", "uops_dispatched_port:port_7">
  ];
  let ValidationCounters = DefaultIntelPfmValidationCounters;
}
def : PfmCountersBinding<"skylake", SkylakeClientPfmCounters>;

def SkylakeServerPfmCounters : ProcPfmCounters {
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = UopsIssuedPfmCounter;
  let IssueCounters = [
    PfmIssueCounter<"SKXPort0", "uops_dispatched_port:port_0">,
    PfmIssueCounter<"SKXPort1", "uops_dispatched_port:port_1">,
    PfmIssueCounter<"SKXPort2", "uops_dispatched_port:port_2">,
    PfmIssueCounter<"SKXPort3", "uops_dispatched_port:port_3">,
    PfmIssueCounter<"SKXPort4", "uops_dispatched_port:port_4">,
    PfmIssueCounter<"SKXPort5", "uops_dispatched_port:port_5">,
    PfmIssueCounter<"SKXPort6", "uops_dispatched_port:port_6">,
    PfmIssueCounter<"SKXPort7", "uops_dispatched_port:port_7">
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen classes, records, and target metadata for the core X86 backend. Key symbols include SkylakeClientPfmCounters, SkylakeServerPfmCounters. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 类、记录与目标元数据。关键符号包括 SkylakeClientPfmCounters, SkylakeServerPfmCounters。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 181-210: TableGen classes, records, and target metadata / TableGen 类、记录与目标元数据
```tablegen
  ];
  let ValidationCounters = DefaultIntelPfmValidationCounters;
}
def : PfmCountersBinding<"skylake-avx512", SkylakeServerPfmCounters>;
def : PfmCountersBinding<"cascadelake", SkylakeServerPfmCounters>;
def : PfmCountersBinding<"cannonlake", SkylakeServerPfmCounters>;

def IceLakePfmCounters : ProcPfmCounters {
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = UopsIssuedPfmCounter;
  let IssueCounters = [
    PfmIssueCounter<"ICXPort0",  "uops_dispatched:port_0">,
    PfmIssueCounter<"ICXPort1",  "uops_dispatched:port_1">,
    PfmIssueCounter<"ICXPort23", "uops_dispatched:port_2_3">,
    PfmIssueCounter<"ICXPort49", "uops_dispatched:port_4_9">,
    PfmIssueCounter<"ICXPort5",  "uops_dispatched:port_5">,
    PfmIssueCounter<"ICXPort6",  "uops_dispatched:port_6">,
    PfmIssueCounter<"ICXPort78", "uops_dispatched:port_7_8">
  ];
  let ValidationCounters = DefaultIntelPfmValidationCounters;
}
def : PfmCountersBinding<"icelake-client", IceLakePfmCounters>;
def : PfmCountersBinding<"icelake-server", IceLakePfmCounters>;
def : PfmCountersBinding<"rocketlake", IceLakePfmCounters>;
def : PfmCountersBinding<"tigerlake", IceLakePfmCounters>;

def AlderLakePfmCounters : ProcPfmCounters {
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = UopsIssuedPfmCounter;
  let IssueCounters = [
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen classes, records, and target metadata for the core X86 backend. Key symbols include IceLakePfmCounters, AlderLakePfmCounters. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 类、记录与目标元数据。关键符号包括 IceLakePfmCounters, AlderLakePfmCounters。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 211-240: TableGen classes, records, and target metadata / TableGen 类、记录与目标元数据
```tablegen
    PfmIssueCounter<"ADLPPort00", "uops_dispatched:port_0">,
    PfmIssueCounter<"ADLPPort01", "uops_dispatched:port_1">,
    PfmIssueCounter<"ADLPPort02_03_10", "uops_dispatched:port_2_3_10">,
    PfmIssueCounter<"ADLPPort04_09", "uops_dispatched:port_4_9">,
    PfmIssueCounter<"ADLPPort05_11", "uops_dispatched:port_5_11">,
    PfmIssueCounter<"ADLPPort06", "uops_dispatched:port_6">,
    PfmIssueCounter<"ADLPPort07_08", "uops_dispatched:port_7_8">
  ];
  let ValidationCounters = DefaultIntelPfmValidationCounters;
}
def : PfmCountersBinding<"alderlake", AlderLakePfmCounters>;

def SapphireRapidsPfmCounters : ProcPfmCounters {
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = UopsIssuedPfmCounter;
  let IssueCounters = [
    PfmIssueCounter<"SPRPort00", "uops_dispatched:port_0">,
    PfmIssueCounter<"SPRPort01", "uops_dispatched:port_1">,
    PfmIssueCounter<"SPRPort02_03_10", "uops_dispatched:port_2_3_10">,
    PfmIssueCounter<"SPRPort04_09", "uops_dispatched:port_4_9">,
    PfmIssueCounter<"SPRPort05_11", "uops_dispatched:port_5_11">,
    PfmIssueCounter<"SPRPort06", "uops_dispatched:port_6">,
    PfmIssueCounter<"SPRPort07_08", "uops_dispatched:port_7_8">,
  ];
  let ValidationCounters = DefaultIntelPfmValidationCounters;
}
def : PfmCountersBinding<"sapphirerapids", SapphireRapidsPfmCounters>;
def : PfmCountersBinding<"emeraldrapids", SapphireRapidsPfmCounters>;

def LunarLakePfmCounters : ProcPfmCounters {
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen classes, records, and target metadata for the core X86 backend. Key symbols include SapphireRapidsPfmCounters, LunarLakePfmCounters. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 类、记录与目标元数据。关键符号包括 SapphireRapidsPfmCounters, LunarLakePfmCounters。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 241-270: TableGen classes, records, and target metadata / TableGen 类、记录与目标元数据
```tablegen
  let CycleCounter = UnhaltedCoreCyclesPfmCounter;
  let UopsCounter = UopsIssuedPfmCounter;
  let IssueCounters = [
    // Refer: https://perfmon-events.intel.com/ section Lunar Lake Hybrid Event
    // ALU Dispatch - Any of ALUs with latency 1 cycle that is not jmp or Shift.
    PfmIssueCounter<"LNLPVPort02_03", "uops_dispatched:alu">,
    PfmIssueCounter<"LNLPPort00_01_02_03_04_05", "uops_dispatched:int_eu_all">,
    PfmIssueCounter<"LNLPPort00_02_04", "uops_dispatched:jmp">,
    PfmIssueCounter<"LNLPPort20_21_22", "uops_dispatched:load">,
    PfmIssueCounter<"LNLPPort01_03_05", "uops_dispatched:shift">,
    // Slow Dispatch - If uops latency > 1, counted as slow. TBD
    // PfmIssueCounter<"LNLPPort01_03_05", "uops_dispatched:slow">,
    PfmIssueCounter<"LNLPPort25_26_27", "uops_dispatched:sta">,
    PfmIssueCounter<"LNLPPort10_11", "uops_dispatched:std">
  ];
  let ValidationCounters = DefaultIntelPfmValidationCounters;
}
def : PfmCountersBinding<"lunarlake", LunarLakePfmCounters>;

// AMD X86 Counters.
defvar DefaultAMDPfmValidationCounters = [
  PfmValidationCounter<InstructionRetired, "RETIRED_INSTRUCTIONS">,
  PfmValidationCounter<L1DCacheLoadMiss, "L1-DCACHE-LOAD-MISSES">,
  PfmValidationCounter<L1DCacheStoreMiss, "L1-DCACHE-STORE-MISSES">,
  PfmValidationCounter<L1ICacheLoadMiss, "L1-ICACHE-LOAD-MISSES">,
  PfmValidationCounter<DataTLBLoadMiss, "DTLB-LOAD-MISSES">,
  PfmValidationCounter<InstructionTLBLoadMiss, "ITLB-LOAD-MISSES">,
  PfmValidationCounter<BranchPredictionMiss, "BRANCH-MISSES">
];

```
**EN:** This section uses LLVM TableGen DSL to describe tablegen classes, records, and target metadata for the core X86 backend. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 类、记录与目标元数据。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 271-300: TableGen classes, records, and target metadata / TableGen 类、记录与目标元数据
```tablegen
// Set basic counters for AMD cpus that we know libpfm4 supports.
def DefaultAMDPfmCounters : ProcPfmCounters {
  let CycleCounter = PfmCounter<"cpu_clk_unhalted">;
  let UopsCounter = PfmCounter<"retired_uops">;
}
def : PfmCountersBinding<"athlon", DefaultAMDPfmCounters>;
def : PfmCountersBinding<"athlon-tbird", DefaultAMDPfmCounters>;
def : PfmCountersBinding<"athlon-4", DefaultAMDPfmCounters>;
def : PfmCountersBinding<"athlon-xp", DefaultAMDPfmCounters>;
def : PfmCountersBinding<"athlon-mp", DefaultAMDPfmCounters>;
def : PfmCountersBinding<"k8", DefaultAMDPfmCounters>;
def : PfmCountersBinding<"opteron", DefaultAMDPfmCounters>;
def : PfmCountersBinding<"athlon64", DefaultAMDPfmCounters>;
def : PfmCountersBinding<"athlon-fx", DefaultAMDPfmCounters>;
def : PfmCountersBinding<"k8-sse3", DefaultAMDPfmCounters>;
def : PfmCountersBinding<"opteron-sse3", DefaultAMDPfmCounters>;
def : PfmCountersBinding<"athlon64-sse3", DefaultAMDPfmCounters>;
def : PfmCountersBinding<"amdfam10", DefaultAMDPfmCounters>;
def : PfmCountersBinding<"barcelona", DefaultAMDPfmCounters>;

def BdVer2PfmCounters : ProcPfmCounters {
  let CycleCounter = PfmCounter<"cpu_clk_unhalted">;
  let UopsCounter = PfmCounter<"retired_uops">;
  let IssueCounters = [
    PfmIssueCounter<"PdFPU0", "dispatched_fpu_ops:ops_pipe0 + dispatched_fpu_ops:ops_dual_pipe0">,
    PfmIssueCounter<"PdFPU1", "dispatched_fpu_ops:ops_pipe1 + dispatched_fpu_ops:ops_dual_pipe1">,
    PfmIssueCounter<"PdFPU2", "dispatched_fpu_ops:ops_pipe2 + dispatched_fpu_ops:ops_dual_pipe2">,
    PfmIssueCounter<"PdFPU3", "dispatched_fpu_ops:ops_pipe3 + dispatched_fpu_ops:ops_dual_pipe3">
  ];
}
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen classes, records, and target metadata for the core X86 backend. Key symbols include DefaultAMDPfmCounters, BdVer2PfmCounters. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 类、记录与目标元数据。关键符号包括 DefaultAMDPfmCounters, BdVer2PfmCounters。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 301-330: TableGen classes, records, and target metadata / TableGen 类、记录与目标元数据
```tablegen
def : PfmCountersBinding<"bdver1", BdVer2PfmCounters>;
def : PfmCountersBinding<"bdver2", BdVer2PfmCounters>;

def BdVer3PfmCounters : ProcPfmCounters {
  let CycleCounter = PfmCounter<"cpu_clk_unhalted">;
  let UopsCounter = PfmCounter<"retired_uops">;
  let IssueCounters = [
    PfmIssueCounter<"SrFPU0", "dispatched_fpu_ops:ops_pipe0 + dispatched_fpu_ops:ops_dual_pipe0">,
    PfmIssueCounter<"SrFPU1", "dispatched_fpu_ops:ops_pipe1 + dispatched_fpu_ops:ops_dual_pipe1">,
    PfmIssueCounter<"SrFPU2", "dispatched_fpu_ops:ops_pipe2 + dispatched_fpu_ops:ops_dual_pipe2">
  ];
}
def : PfmCountersBinding<"bdver3", BdVer3PfmCounters>;
def : PfmCountersBinding<"bdver4", BdVer3PfmCounters>;

def BtVer1PfmCounters : ProcPfmCounters {
  let CycleCounter = PfmCounter<"cpu_clk_unhalted">;
  let UopsCounter = PfmCounter<"retired_uops">;
  let IssueCounters = [
    PfmIssueCounter<"BtFPU0", "dispatched_fpu:pipe0">,
    PfmIssueCounter<"BtFPU1", "dispatched_fpu:pipe1">
  ];
}
def : PfmCountersBinding<"btver1", BtVer1PfmCounters>;

def BtVer2PfmCounters : ProcPfmCounters {
  let CycleCounter = PfmCounter<"cpu_clk_unhalted">;
  let UopsCounter = PfmCounter<"retired_uops">;
  let IssueCounters = [
    PfmIssueCounter<"JFPU0", "dispatched_fpu:pipe0">,
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen classes, records, and target metadata for the core X86 backend. Key symbols include BdVer3PfmCounters, BtVer1PfmCounters, BtVer2PfmCounters. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 类、记录与目标元数据。关键符号包括 BdVer3PfmCounters, BtVer1PfmCounters, BtVer2PfmCounters。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 331-360: TableGen classes, records, and target metadata / TableGen 类、记录与目标元数据
```tablegen
    PfmIssueCounter<"JFPU1", "dispatched_fpu:pipe1">
  ];
}
def : PfmCountersBinding<"btver2", BtVer2PfmCounters>;

def ZnVer1PfmCounters : ProcPfmCounters {
  let CycleCounter = PfmCounter<"cycles_not_in_halt">;
  let UopsCounter = PfmCounter<"retired_uops">;
  let IssueCounters = [
    PfmIssueCounter<"ZnFPU0", "fpu_pipe_assignment:total0">,
    PfmIssueCounter<"ZnFPU1", "fpu_pipe_assignment:total1">,
    PfmIssueCounter<"ZnFPU2", "fpu_pipe_assignment:total2">,
    PfmIssueCounter<"ZnFPU3", "fpu_pipe_assignment:total3">,
    PfmIssueCounter<"ZnAGU", "ls_dispatch:ld_st_dispatch + ls_dispatch:ld_dispatch + ls_dispatch:store_dispatch">,
    PfmIssueCounter<"ZnDivider", "div_op_count">
  ];
  let ValidationCounters = DefaultAMDPfmValidationCounters;
}
def : PfmCountersBinding<"znver1", ZnVer1PfmCounters>;

def ZnVer2PfmCounters : ProcPfmCounters {
  let CycleCounter = PfmCounter<"cycles_not_in_halt">;
  let UopsCounter = PfmCounter<"retired_uops">;
  let IssueCounters = [
    PfmIssueCounter<"Zn2AGU", "ls_dispatch:ld_st_dispatch + ls_dispatch:ld_dispatch + ls_dispatch:store_dispatch">,
    PfmIssueCounter<"Zn2Divider", "div_op_count">
  ];
  let ValidationCounters = DefaultAMDPfmValidationCounters;
}
def : PfmCountersBinding<"znver2", ZnVer2PfmCounters>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen classes, records, and target metadata for the core X86 backend. Key symbols include ZnVer1PfmCounters, ZnVer2PfmCounters. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 类、记录与目标元数据。关键符号包括 ZnVer1PfmCounters, ZnVer2PfmCounters。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 361-390: TableGen classes, records, and target metadata / TableGen 类、记录与目标元数据
```tablegen

def ZnVer3PfmCounters : ProcPfmCounters {
  let CycleCounter = PfmCounter<"cycles_not_in_halt">;
  let UopsCounter = PfmCounter<"retired_ops">;
  let IssueCounters = [
    PfmIssueCounter<"Zn3Int", "ops_type_dispatched_from_decoder:int_disp_retire_mode">,
    PfmIssueCounter<"Zn3FPU", "ops_type_dispatched_from_decoder:fp_disp_retire_mode">,
    PfmIssueCounter<"Zn3Load", "ls_dispatch:ld_dispatch">,
    PfmIssueCounter<"Zn3Store", "ls_dispatch:store_dispatch">,
    PfmIssueCounter<"Zn3Divider", "div_op_count">
  ];
  let ValidationCounters = DefaultAMDPfmValidationCounters;
}
def : PfmCountersBinding<"znver3", ZnVer3PfmCounters>;

def ZnVer4PfmCounters : ProcPfmCounters {
  let CycleCounter = PfmCounter<"cycles_not_in_halt">;
  let UopsCounter = PfmCounter<"retired_ops">;
  let IssueCounters = [
    PfmIssueCounter<"Zn4Int", "ops_type_dispatched_from_decoder:int_disp_retire_mode">,
    PfmIssueCounter<"Zn4FPU", "ops_type_dispatched_from_decoder:fp_disp_retire_mode">,
    PfmIssueCounter<"Zn4Load", "ls_dispatch:ld_dispatch">,
    PfmIssueCounter<"Zn4Store", "ls_dispatch:store_dispatch">,
    PfmIssueCounter<"Zn4Divider", "div_op_count">,
    PfmIssueCounter<"Zn4AGU", "ls_dispatch:ld_st_dispatch + ls_dispatch:ld_dispatch + ls_dispatch:store_dispatch">
  ];
  let ValidationCounters = DefaultAMDPfmValidationCounters;
}
def : PfmCountersBinding<"znver4", ZnVer4PfmCounters>;
def : PfmCountersBinding<"znver5", ZnVer4PfmCounters>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen classes, records, and target metadata for the core X86 backend. Key symbols include ZnVer3PfmCounters, ZnVer4PfmCounters. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 类、记录与目标元数据。关键符号包括 ZnVer3PfmCounters, ZnVer4PfmCounters。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 391-391: TableGen classes, records, and target metadata / TableGen 类、记录与目标元数据
```tablegen
def : PfmCountersBinding<"znver6", ZnVer4PfmCounters>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen classes, records, and target metadata for the core X86 backend. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 类、记录与目标元数据。这些记录随后会展开为 X86 后端使用的生成表。

## Key Concepts / 关键概念
- Primary topic: X86 target definitions. / 核心主题：X86 目标定义。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: UnhaltedCoreCyclesPfmCounter, UopsIssuedPfmCounter, DefaultPfmCounters, PentiumPfmCounters, CorePfmCounters, AtomPfmCounters. / 重要符号：UnhaltedCoreCyclesPfmCounter, UopsIssuedPfmCounter, DefaultPfmCounters, PentiumPfmCounters, CorePfmCounters, AtomPfmCounters。
- This file is written in TableGen DSL and contributes generated target tables. / 该文件使用 TableGen DSL 编写，并参与生成目标描述表。

## Dependencies / 依赖关系
- Consumed by LLVM TableGen to emit generated matcher, encoding, register, or scheduling tables. / 由 LLVM TableGen 消费，以生成匹配器、编码、寄存器或调度相关表。
- Closely coupled with sibling X86 .td fragments that assemble the full target description. / 与同级 X86 .td 片段紧密协作，共同组成完整目标描述。
