# RISCVSchedSiFive7.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVSchedSiFive7.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for processor scheduling resources, latencies, and pipeline behavior for RISC-V cores. / 使用 TableGen 定义RISC-V 处理器内核的调度资源、时延与流水线行为。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Commentary and design intent / 注释与设计意图
```tablegen
//==- RISCVSchedSiFive7.td - SiFive7 Scheduling Definitions --*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//

/// c is true if mx has the worst case behavior compared to LMULs in MxList.
/// On the SiFive7, the worst case LMUL is the Largest LMUL
/// and the worst case sew is the smallest SEW for that LMUL.
class SiFive7IsWorstCaseMX<string mx, list<string> MxList> {
  defvar LLMUL = LargestLMUL<MxList>.r;
  bit c = !eq(mx, LLMUL);
}
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 19-41: TableGen class SiFive7IsWorstCaseMXSEW<string / TableGen 类 SiFive7IsWorstCaseMXSEW<string
```tablegen
/// c is true if mx and sew have the worst case behavior compared to LMULs in
/// MxList. On the SiFive7, the worst case LMUL is the Largest LMUL
/// and the worst case sew is the smallest SEW for that LMUL.
class SiFive7IsWorstCaseMXSEW<string mx, int sew, list<string> MxList,
                               bit isF = 0> {
  defvar LLMUL = LargestLMUL<MxList>.r;
  defvar SSEW = SmallestSEW<mx, isF>.r;
  bit c = !and(!eq(mx, LLMUL), !eq(sew, SSEW));
}

/// Number of DLEN parts = (LMUL * VLEN) / DLEN.
/// Since DLEN = VLEN / 2, Num DLEN parts = 2 * LMUL.
class SiFive7GetCyclesDefault<string mx> {
  int c = !cond(
    !eq(mx, "M1") : 2,
    !eq(mx, "M2") : 4,
    !eq(mx, "M4") : 8,
    !eq(mx, "M8") : 16,
    !eq(mx, "MF2") : 1,
    !eq(mx, "MF4") : 1,
    !eq(mx, "MF8") : 1
  );
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 42-64: TableGen class SiFive7GetCyclesNarrowing<string / TableGen 类 SiFive7GetCyclesNarrowing<string
```tablegen

class SiFive7GetCyclesNarrowing<string mx> {
  int c = !cond(
    !eq(mx, "M1") : 4,
    !eq(mx, "M2") : 8,
    !eq(mx, "M4") : 16,
    !eq(mx, "MF2") : 2,
    !eq(mx, "MF4") : 1,
    !eq(mx, "MF8") : 1
  );
}

class SiFive7GetCyclesVMask<string mx> {
  int c = !cond(
    !eq(mx, "M1") : 1,
    !eq(mx, "M2") : 1,
    !eq(mx, "M4") : 1,
    !eq(mx, "M8") : 2,
    !eq(mx, "MF2") : 1,
    !eq(mx, "MF4") : 1,
    !eq(mx, "MF8") : 1
  );
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 65-87: TableGen class SiFive7GetMaskLoadStoreCycles<string / TableGen 类 SiFive7GetMaskLoadStoreCycles<string
```tablegen

/// VLDM and VSTM can't read/write more than 2 DLENs of data.
/// 2 DLENs when LMUL=8. 1 DLEN for all other DLENs
class SiFive7GetMaskLoadStoreCycles<string mx> {
  int c = !cond(
    !eq(mx, "M8")  : 2,
    true : 1
  );
}

// Cycles for nf=2 segmented loads and stores are calculated using the
// formula (2 * VLEN * LMUL) / DLEN = 4 * LMUL
class SiFive7GetCyclesSegmentedSeg2<string mx> {
  int c = !cond(
    !eq(mx, "M1") :  4,
    !eq(mx, "M2") :  8,
    !eq(mx, "M4") :  16,
    !eq(mx, "M8") :  32,
    !eq(mx, "MF2") : 2,
    !eq(mx, "MF4") : 1,
    !eq(mx, "MF8") : 1
  );
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 88-107: TableGen class SiFive7GetCyclesSegmented<string / TableGen 类 SiFive7GetCyclesSegmented<string
```tablegen

// Cycles for segmented loads and stores are calculated using the
// formula vl * ceil((SEW * nf) / DLEN), where SEW * nf is the segment size.
class SiFive7GetCyclesSegmented<string mx, int sew, int nf, int VLEN> {
  defvar DLEN = !div(VLEN, 2);
  // (VLEN * LMUL) / SEW
  defvar VLUpperBound  = !cond(
    !eq(mx, "M1") : !div(VLEN, sew),
    !eq(mx, "M2") : !div(!mul(VLEN, 2), sew),
    !eq(mx, "M4") : !div(!mul(VLEN, 4), sew),
    !eq(mx, "M8") : !div(!mul(VLEN, 8), sew),
    !eq(mx, "MF2") : !div(!div(VLEN, 2), sew),
    !eq(mx, "MF4") : !div(!div(VLEN, 4), sew),
    !eq(mx, "MF8") : !div(!div(VLEN, 8), sew),
  );
  // We can calculate ceil(a/b) using (a + b - 1) / b.
  defvar a = !mul(sew, nf);
  defvar b = DLEN;
  int c = !mul(VLUpperBound, !div(!sub(!add(a, b), 1), b));
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 108-125: TableGen class SiFive7GetCyclesOnePerElement<string / TableGen 类 SiFive7GetCyclesOnePerElement<string
```tablegen

class SiFive7GetCyclesOnePerElement<string mx, int sew, int VLEN> {
  // c = ceil(VLEN / SEW) * LMUL
  // Note: c >= 1 since the smallest VLEN is 512 / 8 = 8, and the
  // largest division performed on VLEN is in MF8 case with division
  // by 8. Therefore, there is no need to ceil the result.
  int numElements = !div(VLEN, sew);
  int c = !cond(
    !eq(mx, "M1")  : numElements,
    !eq(mx, "M2")  : !mul(numElements, 2),
    !eq(mx, "M4")  : !mul(numElements, 4),
    !eq(mx, "M8")  : !mul(numElements, 8),
    !eq(mx, "MF2") : !div(numElements, 2),
    !eq(mx, "MF4") : !div(numElements, 4),
    !eq(mx, "MF8") : !div(numElements, 8)
  );
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 126-155: TableGen class SiFive7GetDivOrSqrtFactor<int / TableGen 类 SiFive7GetDivOrSqrtFactor<int
```tablegen
class SiFive7GetDivOrSqrtFactor<int sew> {
  int c = !cond(
    // TODO: Add SchedSEWSetFP upstream and remove the SEW=8 case.
    !eq(sew, 8) : 15,
    !eq(sew, 16) : 15,
    !eq(sew, 32) : 28,
    !eq(sew, 64) : 57
  );
}

/// Cycles for reductions take approximately VL*SEW/DLEN + 5(4 + log(DLEN/SEW))
/// cycles.
class SiFive7GetReductionCycles<string mx, int sew, int VLEN> {
  // VLUpperBound*SEW/DLEN is equivalent to 2*LMUL since
  // VLUpperBound=(VLEN*LMUL)/SEW.
  defvar DLEN = !div(VLEN, 2);
  defvar TwoTimesLMUL = !cond(
    !eq(mx, "M1") : 2,
    !eq(mx, "M2") : 4,
    !eq(mx, "M4") : 8,
    !eq(mx, "M8") : 16,
    !eq(mx, "MF2") : 1,
    !eq(mx, "MF4") : 1,
    !eq(mx, "MF8") : 1
  );
  int c = !add(
    TwoTimesLMUL,
    !mul(5, !add(4, !logtwo(!div(DLEN, sew))))
  );
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 156-174: TableGen class SiFive7GetOrderedReductionCycles<string / TableGen 类 SiFive7GetOrderedReductionCycles<string
```tablegen

/// Cycles for ordered reductions take approximately 6*VL cycles
class SiFive7GetOrderedReductionCycles<string mx, int sew, int VLEN> {
  // (VLEN * LMUL) / SEW
  defvar VLUpperBound  = !cond(
    !eq(mx, "M1") : !div(VLEN, sew),
    !eq(mx, "M2") : !div(!mul(VLEN, 2), sew),
    !eq(mx, "M4") : !div(!mul(VLEN, 4), sew),
    !eq(mx, "M8") : !div(!mul(VLEN, 8), sew),
    !eq(mx, "MF2") : !div(!div(VLEN, 2), sew),
    !eq(mx, "MF4") : !div(!div(VLEN, 4), sew),
    !eq(mx, "MF8") : !div(!div(VLEN, 8), sew),
  );
  int c = !mul(6, VLUpperBound);
}

class isSingleDLEN<string mx> {
  bit c = !or(!eq(mx, "MF2"), !or(!eq(mx, "MF4"), !eq(mx, "MF8")));
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 175-207: TableGen class SiFive7GetCyclesVRGatherVV<string / TableGen 类 SiFive7GetCyclesVRGatherVV<string
```tablegen

class SiFive7GetCyclesVRGatherVV<string mx, int sew, int VLEN,
                                 bit hasFastGather> {
  // if (hasFastGather && isSingleDLEN(mx))
  //   c = 1;
  //  else if (hasFastGather && (log2(SEW/8) + log2(LMUL) <= log2(DLEN / 32))
  //   c = LMUL * 2 * ceil(vl * SEW / DLEN);
  //  else
  //   c = vl;

  defvar y = !logtwo(!div(sew, 8));
  defvar x = !cond(
    !eq(mx, "M1") : y,
    !eq(mx, "M2") : !add(y, 1),
    !eq(mx, "M4") : !add(y, 2),
    !eq(mx, "M8") : !add(y, 3),
    // Give isSingleDLEN(mx) cases a garbage value to avoid build failures,
    // even though x will go unused.
    true : 1
  );
  // LMUL * 2 * ceil(vl * SEW / DLEN) = LMUL * 2 * ceil(2 * LMUL)
  defvar z = !cond(
    !eq(mx, "M1") : 4,
    !eq(mx, "M2") : 16,
    !eq(mx, "M4") : 64,
    !eq(mx, "M8") : 256,
    // Give isSingleDLEN(mx) cases a garbage value to avoid build failures,
    // even though z will go unused.
    true : 1
  );
  defvar VLUpperBound = SiFive7GetCyclesOnePerElement<mx, sew, VLEN>.c;
  bit IsSingleDLEN = isSingleDLEN<mx>.c;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 208-228: TableGen class SiFive7GetCyclesVCompress<string / TableGen 类 SiFive7GetCyclesVCompress<string
```tablegen
  int c = !cond(
    !and(hasFastGather, IsSingleDLEN) : 1,
    !and(hasFastGather, !le(x, !logtwo(!div(VLEN, 64)))) : z,
    true: VLUpperBound
  );
}

class SiFive7GetCyclesVCompress<string mx, int sew, int VLEN,
                               bit hasFastGather> {

  // if (hasFastGather && isSingleDLEN(mx))
  //   c = 1
  // else
  //   c = vl
  defvar VLUpperBound = SiFive7GetCyclesOnePerElement<mx, sew, VLEN>.c;
  bit IsSingleDLEN = isSingleDLEN<mx>.c;

  int c = !if(!and(hasFastGather, IsSingleDLEN),
              1,
              VLUpperBound);
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 229-248: TableGen class SiFive7GetSiFiveVFNRClipCycles<string / TableGen 类 SiFive7GetSiFiveVFNRClipCycles<string
```tablegen

class SiFive7GetSiFiveVFNRClipCycles<string mx, int VLEN> {
  int latency = !cond(
    !eq(mx, "MF8"): 7,
    !eq(mx, "MF4"): 8,
    !eq(mx, "MF2"): 10,
    !eq(mx, "M1"): 13,
    !eq(mx, "M2"): 19,
  );

  defvar DLEN = !div(VLEN, 2);
  int occupancy = SiFive7GetCyclesOnePerElement<mx, sew=!div(DLEN, 4),
                                                VLEN=VLEN>.c;
}

class SiFive7FPLatencies {
  int BasicFP16ALU;
  int BasicFP32ALU;
  int BasicFP64ALU;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 249-266: TableGen class SiFive7AnyToGPRBypass<SchedRead / TableGen 类 SiFive7AnyToGPRBypass<SchedRead
```tablegen

class SiFive7AnyToGPRBypass<SchedRead read, int cycles = 2>
    : ReadAdvance<read, cycles, [WriteIALU, WriteIALU32,
                                 WriteShiftImm, WriteShiftImm32,
                                 WriteShiftReg, WriteShiftReg32,
                                 WriteSHXADD, WriteSHXADD32,
                                 WriteRotateImm, WriteRotateImm32,
                                 WriteRotateReg, WriteRotateReg32,
                                 WriteSingleBit, WriteSingleBitImm,
                                 WriteBEXT, WriteBEXTI,
                                 WriteCLZ, WriteCLZ32, WriteCTZ, WriteCTZ32,
                                 WriteCPOP, WriteCPOP32,
                                 WriteREV8, WriteORCB, WriteIMinMax, WriteSFB,
                                 WriteIMul, WriteIMul32,
                                 WriteIDiv, WriteIDiv32,
                                 WriteIRem, WriteIRem32,
                                 WriteLDB, WriteLDH, WriteLDW, WriteLDD]>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 267-287: Commentary and design intent / 注释与设计意图
```tablegen
// The SiFive7 microarchitecture has three kinds of pipelines: A, B, V.
// Pipe A can handle memory, integer alu and vector operations.
// Pipe B can handle integer alu, control flow, integer multiply and divide,
// and floating point computation.
// The V pipeline is modeled by the VCQ, VA, VL, and VS resources. There can
// be one or two VA (Vector Arithmetic).
multiclass SiFive7ProcResources<bit dualVALU = false> {
  let BufferSize = 0 in {
    def PipeA     : ProcResource<1>;
    def PipeB     : ProcResource<1>;

    def IDiv      : ProcResource<1>; // Int Division
    def FDiv      : ProcResource<1>; // FP Division/Sqrt

    // Arithmetic sequencer(s)
    // VA1 can handle any vector airthmetic instruction.
    def VA1     : ProcResource<1>;
    if dualVALU then {
      // VA2 generally can only handle simple vector arithmetic.
      def VA2     : ProcResource<1>;
    }
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 288-308: TableGen record VL / TableGen 记录 VL
```tablegen

    def VL        : ProcResource<1>; // Load sequencer
    def VS        : ProcResource<1>; // Store sequencer
    // The VCQ accepts instructions from the the A Pipe and holds them until the
    // vector unit is ready to dequeue them. The unit dequeues up to one instruction
    // per cycle, in order, as soon as the sequencer for that type of instruction is
    // available. This resource is meant to be used for 1 cycle by all vector
    // instructions, to model that only one vector instruction may be dequeued at a
    // time. The actual dequeueing into the sequencer is modeled by the VA, VL, and
    // VS sequencer resources below. Each of them will only accept a single
    // instruction at a time and remain busy for the number of cycles associated
    // with that instruction.
    def VCQ       : ProcResource<1>; // Vector Command Queue

    def PipeAB : ProcResGroup<[!cast<ProcResource>(NAME#"PipeA"),
                               !cast<ProcResource>(NAME#"PipeB")]>;

    if dualVALU then
    def VA1OrVA2 : ProcResGroup<[!cast<ProcResource>(NAME#"VA1"),
                                 !cast<ProcResource>(NAME#"VA2")]>;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 309-326: Reusable TableGen multiclass SiFive7WriteResBase / 可复用的 TableGen 多类 SiFive7WriteResBase
```tablegen
}

multiclass SiFive7WriteResBase<int VLEN,
    ProcResourceKind PipeA, ProcResourceKind PipeB, ProcResourceKind PipeAB,
    ProcResourceKind IDiv, ProcResourceKind FDiv,
    ProcResourceKind VA1, ProcResourceKind VA1OrVA2,
    ProcResourceKind VL, ProcResourceKind VS,
    ProcResourceKind VCQ,
    SiFive7FPLatencies fpLatencies,
    bit hasFastGather = false> {

  // Branching
  let Latency = 3 in {
    def : WriteRes<WriteJmp, [PipeB]>;
    def : WriteRes<WriteJal, [PipeB]>;
    def : WriteRes<WriteJalr, [PipeB]>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 327-347: TableGen record definition / TableGen 记录定义
```tablegen
  //Short forward branch
  def : WriteRes<WriteSFB, [PipeA, PipeB]> {
    let Latency = 3;
    let NumMicroOps = 2;
  }

  // Integer arithmetic and logic
  let Latency = 3 in {
    def : WriteRes<WriteIALU, [PipeAB]>;
    def : WriteRes<WriteIALU32, [PipeAB]>;
    def : WriteRes<WriteShiftImm, [PipeAB]>;
    def : WriteRes<WriteShiftImm32, [PipeAB]>;
    def : WriteRes<WriteShiftReg, [PipeAB]>;
    def : WriteRes<WriteShiftReg32, [PipeAB]>;
  }

  // Integer multiplication
  let Latency = 3 in {
    def : WriteRes<WriteIMul, [PipeB]>;
    def : WriteRes<WriteIMul32, [PipeB]>;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 348-369: TableGen record SiFive7RV64IDivRemSchedWriteRes / TableGen 记录 SiFive7RV64IDivRemSchedWriteRes
```tablegen

  // IDiv / IRem in a 32-bit core should have the same latency and throughput
  // as IDiv32 / IRem32 in a 64-bit core.
  def SiFive7RV64IDivRemSchedWriteRes : SchedWriteRes<[PipeB, IDiv]> {
    let Latency = 66;
    let ReleaseAtCycles = [1, 65];
  }
  def SiFive7RV32IDivRemSchedWriteRes : SchedWriteRes<[PipeB, IDiv]> {
    let Latency = 34;
    let ReleaseAtCycles = [1, 33];
  }
  def SiFive7IDivRemSchedWriteVariant : SchedWriteVariant<[
    SchedVar<RV32SchedPred, [!cast<SchedWriteRes>(NAME # "SiFive7RV32IDivRemSchedWriteRes")]>,
    SchedVar<NoSchedPred, [!cast<SchedWriteRes>(NAME # "SiFive7RV64IDivRemSchedWriteRes")]>
  ]>;

  // Integer division
  def : SchedAlias<WriteIDiv, !cast<SchedWrite>(NAME # "SiFive7IDivRemSchedWriteVariant")>;
  def : WriteRes<WriteIDiv32,  [PipeB, IDiv]> {
    let Latency = 34;
    let ReleaseAtCycles = [1, 33];
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 370-391: TableGen record definition / TableGen 记录定义
```tablegen

  // Integer remainder
  def : SchedAlias<WriteIRem, !cast<SchedWrite>(NAME # "SiFive7IDivRemSchedWriteVariant")>;
  def : WriteRes<WriteIRem32,  [PipeB, IDiv]> {
    let Latency = 34;
    let ReleaseAtCycles = [1, 33];
  }

  // Bitmanip
  let Latency = 3 in {
    // Rotates are in the late-B ALU.
    def : WriteRes<WriteRotateImm, [PipeB]>;
    def : WriteRes<WriteRotateImm32, [PipeB]>;
    def : WriteRes<WriteRotateReg, [PipeB]>;
    def : WriteRes<WriteRotateReg32, [PipeB]>;

    // clz[w]/ctz[w] are in the late-B ALU.
    def : WriteRes<WriteCLZ, [PipeB]>;
    def : WriteRes<WriteCLZ32, [PipeB]>;
    def : WriteRes<WriteCTZ, [PipeB]>;
    def : WriteRes<WriteCTZ32, [PipeB]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 392-409: TableGen record definition / TableGen 记录定义
```tablegen
    // cpop[w] look exactly like multiply.
    def : WriteRes<WriteCPOP, [PipeB]>;
    def : WriteRes<WriteCPOP32, [PipeB]>;

    // orc.b is in the late-B ALU.
    def : WriteRes<WriteORCB, [PipeB]>;

    // min/max are in the late-B ALU
    def : WriteRes<WriteIMinMax, [PipeB]>;

    // rev8 is in the late-A and late-B ALUs.
    def : WriteRes<WriteREV8, [PipeAB]>;

    // shNadd[.uw] is on the early-B and late-B ALUs.
    def : WriteRes<WriteSHXADD, [PipeB]>;
    def : WriteRes<WriteSHXADD32, [PipeB]>;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 410-428: TableGen record definition / TableGen 记录定义
```tablegen
  // Single-bit instructions
  // BEXT[I] instruction is available on all ALUs and the other instructions
  // are only available on the B pipe.
  let Latency = 3 in {
    def : WriteRes<WriteSingleBit, [PipeB]>;
    def : WriteRes<WriteSingleBitImm, [PipeB]>;
    def : WriteRes<WriteBEXT, [PipeAB]>;
    def : WriteRes<WriteBEXTI, [PipeAB]>;
  }

  // Memory
  def : WriteRes<WriteSTB, [PipeA]>;
  def : WriteRes<WriteSTH, [PipeA]>;
  def : WriteRes<WriteSTW, [PipeA]>;
  def : WriteRes<WriteSTD, [PipeA]>;
  def : WriteRes<WriteFST16, [PipeA]>;
  def : WriteRes<WriteFST32, [PipeA]>;
  def : WriteRes<WriteFST64, [PipeA]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 429-449: TableGen record definition / TableGen 记录定义
```tablegen
  let Latency = 3 in {
    def : WriteRes<WriteLDB, [PipeA]>;
    def : WriteRes<WriteLDH, [PipeA]>;
    def : WriteRes<WriteLDW, [PipeA]>;
    def : WriteRes<WriteLDD, [PipeA]>;

    def : WriteRes<WriteFLD16, [PipeA]>;
    def : WriteRes<WriteFLD32, [PipeA]>;
    def : WriteRes<WriteFLD64, [PipeA]>;
  }

  // Atomic memory
  def : WriteRes<WriteAtomicSTW, [PipeA]>;
  def : WriteRes<WriteAtomicSTD, [PipeA]>;

  let Latency = 3 in {
  def : WriteRes<WriteAtomicW, [PipeA]>;
  def : WriteRes<WriteAtomicD, [PipeA]>;
  def : WriteRes<WriteAtomicLDW, [PipeA]>;
  def : WriteRes<WriteAtomicLDD, [PipeA]>;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 450-472: TableGen record definition / TableGen 记录定义
```tablegen

  // Half precision.
  let Latency = fpLatencies.BasicFP16ALU in {
  def : WriteRes<WriteFAdd16, [PipeB]>;
  def : WriteRes<WriteFMul16, [PipeB]>;
  def : WriteRes<WriteFMA16, [PipeB]>;
  }
  let Latency = 3 in {
  def : WriteRes<WriteFSGNJ16, [PipeB]>;
  def : WriteRes<WriteFMinMax16, [PipeB]>;
  }

  let Latency = 14, ReleaseAtCycles = [1, 13] in {
  def :  WriteRes<WriteFDiv16, [PipeB, FDiv]>;
  def :  WriteRes<WriteFSqrt16, [PipeB, FDiv]>;
  }

  // Single precision.
  let Latency = fpLatencies.BasicFP32ALU in {
    def : WriteRes<WriteFAdd32, [PipeB]>;
    def : WriteRes<WriteFMul32, [PipeB]>;
    def : WriteRes<WriteFMA32, [PipeB]>;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 473-492: TableGen record definition / TableGen 记录定义
```tablegen
  let Latency = 3 in {
    def : WriteRes<WriteFSGNJ32, [PipeB]>;
    def : WriteRes<WriteFMinMax32, [PipeB]>;
  }

  let Latency = 27, ReleaseAtCycles = [1, 26] in {
    def : WriteRes<WriteFDiv32, [PipeB, FDiv]>;
    def : WriteRes<WriteFSqrt32, [PipeB, FDiv]>;
  }

  // Double precision
  let Latency = fpLatencies.BasicFP64ALU in {
    def : WriteRes<WriteFAdd64, [PipeB]>;
    def : WriteRes<WriteFMul64, [PipeB]>;
    def : WriteRes<WriteFMA64, [PipeB]>;
  }
  let Latency = 3 in {
    def : WriteRes<WriteFSGNJ64, [PipeB]>;
    def : WriteRes<WriteFMinMax64, [PipeB]>;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 493-519: TableGen record definition / TableGen 记录定义
```tablegen

  let Latency = 56, ReleaseAtCycles = [1, 55] in {
    def : WriteRes<WriteFDiv64, [PipeB, FDiv]>;
    def : WriteRes<WriteFSqrt64, [PipeB, FDiv]>;
  }

  // Conversions
  let Latency = 3 in {
  def : WriteRes<WriteFCvtI32ToF16, [PipeB]>;
  def : WriteRes<WriteFCvtI32ToF32, [PipeB]>;
  def : WriteRes<WriteFCvtI32ToF64, [PipeB]>;
  def : WriteRes<WriteFCvtI64ToF16, [PipeB]>;
  def : WriteRes<WriteFCvtI64ToF32, [PipeB]>;
  def : WriteRes<WriteFCvtI64ToF64, [PipeB]>;
  def : WriteRes<WriteFCvtF16ToI32, [PipeB]>;
  def : WriteRes<WriteFCvtF16ToI64, [PipeB]>;
  def : WriteRes<WriteFCvtF16ToF32, [PipeB]>;
  def : WriteRes<WriteFCvtF16ToF64, [PipeB]>;
  def : WriteRes<WriteFCvtF32ToI32, [PipeB]>;
  def : WriteRes<WriteFCvtF32ToI64, [PipeB]>;
  def : WriteRes<WriteFCvtF32ToF16, [PipeB]>;
  def : WriteRes<WriteFCvtF32ToF64, [PipeB]>;
  def : WriteRes<WriteFCvtF64ToI32, [PipeB]>;
  def : WriteRes<WriteFCvtF64ToI64, [PipeB]>;
  def : WriteRes<WriteFCvtF64ToF16, [PipeB]>;
  def : WriteRes<WriteFCvtF64ToF32, [PipeB]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 520-539: TableGen record definition / TableGen 记录定义
```tablegen
  def : WriteRes<WriteFClass16, [PipeB]>;
  def : WriteRes<WriteFClass32, [PipeB]>;
  def : WriteRes<WriteFClass64, [PipeB]>;
  def : WriteRes<WriteFCmp16, [PipeB]>;
  def : WriteRes<WriteFCmp32, [PipeB]>;
  def : WriteRes<WriteFCmp64, [PipeB]>;
  def : WriteRes<WriteFMovI16ToF16, [PipeA]>;
  def : WriteRes<WriteFMovF16ToI16, [PipeA]>;
  def : WriteRes<WriteFMovI32ToF32, [PipeA]>;
  def : WriteRes<WriteFMovF32ToI32, [PipeA]>;
  def : WriteRes<WriteFMovI64ToF64, [PipeA]>;
  def : WriteRes<WriteFMovF64ToI64, [PipeA]>;
  }

  // 6. Configuration-Setting Instructions
  let Latency = 3 in {
  def : WriteRes<WriteVSETVLI, [PipeA]>;
  def : WriteRes<WriteVSETIVLI, [PipeA]>;
  def : WriteRes<WriteVSETVL, [PipeA]>;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 540-562: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen

  // 7. Vector Loads and Stores
  // Unit-stride loads and stores can operate at the full bandwidth of the memory
  // pipe. The memory pipe is DLEN bits wide on x280.
  foreach mx = SchedMxList in {
    defvar Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVLDE",    [VCQ, VL], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVLDFF",   [VCQ, VL], mx, IsWorstCase>;
    }
    let Latency = 1, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in
    defm : LMULWriteResMX<"WriteVSTE",    [VCQ, VS], mx, IsWorstCase>;
  }

  foreach mx = SchedMxList in {
    defvar Cycles = SiFive7GetMaskLoadStoreCycles<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in
    defm : LMULWriteResMX<"WriteVLDM",    [VCQ, VL], mx, IsWorstCase>;
    let Latency = 1, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in
    defm : LMULWriteResMX<"WriteVSTM",    [VCQ, VS], mx, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 563-585: Commentary and design intent / 注释与设计意图
```tablegen

  // Strided loads and stores operate at one element per cycle and should be
  // scheduled accordingly. Indexed loads and stores operate at one element per
  // cycle, and they stall the machine until all addresses have been generated,
  // so they cannot be scheduled. Indexed and strided loads and stores have LMUL
  // specific suffixes, but since SEW is already encoded in the name of the
  // resource, we do not need to use LMULSEWXXX constructors. However, we do
  // use the SEW from the name to determine the number of Cycles.

  foreach mx = SchedMxListEEW8 in {
    defvar VLDSX0Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar Cycles = SiFive7GetCyclesOnePerElement<mx, 8, VLEN>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    defm  : LMULWriteResMXVariant<"WriteVLDS8",  VLDSX0Pred,
                                  // Predicated
                                  [VCQ, VL], 4, [0, 1], [1, !add(1, VLDSX0Cycles)],
                                  // Not Predicated
                                  [VCQ, VL], !add(3, Cycles), [0, 1], [1, !add(1, Cycles)],
                                  mx, IsWorstCase>;
    let Latency = !add(3, Cycles), AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVLDUX8", [VCQ, VL], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVLDOX8", [VCQ, VL], mx, IsWorstCase>;
    }
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 586-606: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
    let Latency = 1, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVSTS8",  [VCQ, VS], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSTUX8", [VCQ, VS], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSTOX8", [VCQ, VS], mx, IsWorstCase>;
    }
  }

  foreach mx = SchedMxListEEW16 in {
    defvar VLDSX0Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar Cycles = SiFive7GetCyclesOnePerElement<mx, 16, VLEN>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    defm  : LMULWriteResMXVariant<"WriteVLDS16",  VLDSX0Pred,
                                  // Predicated
                                  [VCQ, VL], 4, [0, 1], [1, !add(1, VLDSX0Cycles)],
                                  // Not Predicated
                                  [VCQ, VL], !add(3, Cycles), [0, 1], [1, !add(1, Cycles)],
                                  mx, IsWorstCase>;
    let Latency = !add(3, Cycles), AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVLDUX16", [VCQ, VL], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVLDOX16", [VCQ, VL], mx, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 607-626: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
    let Latency = 1, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVSTS16",  [VCQ, VS], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSTUX16", [VCQ, VS], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSTOX16", [VCQ, VS], mx, IsWorstCase>;
    }
  }
  foreach mx = SchedMxListEEW32 in {
    defvar VLDSX0Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar Cycles = SiFive7GetCyclesOnePerElement<mx, 32, VLEN>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    defm  : LMULWriteResMXVariant<"WriteVLDS32",  VLDSX0Pred,
                                  // Predicated
                                  [VCQ, VL], 4, [0, 1], [1, !add(1, VLDSX0Cycles)],
                                  // Not Predicated
                                  [VCQ, VL], !add(3, Cycles), [0, 1], [1, !add(1, Cycles)],
                                  mx, IsWorstCase>;
    let Latency = !add(3, Cycles), AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVLDUX32", [VCQ, VL], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVLDOX32", [VCQ, VL], mx, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 627-646: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
    let Latency = 1, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVSTS32",  [VCQ, VS], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSTUX32", [VCQ, VS], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSTOX32", [VCQ, VS], mx, IsWorstCase>;
    }
  }
  foreach mx = SchedMxListEEW64 in {
    defvar VLDSX0Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar Cycles = SiFive7GetCyclesOnePerElement<mx, 64, VLEN>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    defm  : LMULWriteResMXVariant<"WriteVLDS64",  VLDSX0Pred,
                                  // Predicated
                                  [VCQ, VL], 4, [0, 1], [1, !add(1, VLDSX0Cycles)],
                                  // Not Predicated
                                  [VCQ, VL], !add(3, Cycles), [0, 1], [1, !add(1, Cycles)],
                                  mx, IsWorstCase>;
    let Latency = !add(3, Cycles), AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVLDUX64", [VCQ, VL], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVLDOX64", [VCQ, VL], mx, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 647-672: Bulk record instantiation / 批量记录实例化
```tablegen
    let Latency = 1, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVSTS64",  [VCQ, VS], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSTUX64", [VCQ, VS], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSTOX64", [VCQ, VS], mx, IsWorstCase>;
    }
  }

  // VLD*R is LMUL aware
  let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 2)] in
    def : WriteRes<WriteVLD1R,  [VCQ, VL]>;
  let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 4)] in
    def : WriteRes<WriteVLD2R,  [VCQ, VL]>;
  let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 8)] in
    def : WriteRes<WriteVLD4R,  [VCQ, VL]>;
  let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 16)] in
    def : WriteRes<WriteVLD8R,  [VCQ, VL]>;
  // VST*R is LMUL aware
  let Latency = 1, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 2)] in
    def : WriteRes<WriteVST1R,   [VCQ, VS]>;
  let Latency = 1, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 4)] in
    def : WriteRes<WriteVST2R,   [VCQ, VS]>;
  let Latency = 1, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 8)] in
    def : WriteRes<WriteVST4R,   [VCQ, VS]>;
  let Latency = 1, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 16)] in
    def : WriteRes<WriteVST8R,   [VCQ, VS]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 673-697: Commentary and design intent / 注释与设计意图
```tablegen
  // Segmented Loads and Stores
  // Unit-stride segmented loads and stores are effectively converted into strided
  // segment loads and stores. Strided segment loads and stores operate at up to
  // one segment per cycle if the segment fits within one aligned memory beat.
  // Indexed segment loads and stores operate at the same rate as strided ones,
  // but they stall the machine until all addresses have been generated.
  foreach mx = SchedMxList in {
    foreach eew = [8, 16, 32, 64] in {
      defvar Cycles = SiFive7GetCyclesSegmentedSeg2<mx>.c;
      defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
      // Does not chain so set latency high
      let Latency = !add(3, Cycles), AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
        defm : LMULWriteResMX<"WriteVLSEG2e" # eew,   [VCQ, VL], mx, IsWorstCase>;
        defm : LMULWriteResMX<"WriteVLSEGFF2e" # eew, [VCQ, VL], mx, IsWorstCase>;
      }
      let Latency = 1, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in
      defm : LMULWriteResMX<"WriteVSSEG2e" # eew,   [VCQ, VS], mx, IsWorstCase>;
      foreach nf=3-8 in {
        defvar Cycles = SiFive7GetCyclesSegmented<mx, eew, nf, VLEN>.c;
        defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
        // Does not chain so set latency high
        let Latency = !add(3, Cycles), AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
          defm : LMULWriteResMX<"WriteVLSEG" # nf # "e" # eew,   [VCQ, VL], mx, IsWorstCase>;
          defm : LMULWriteResMX<"WriteVLSEGFF" # nf # "e" # eew, [VCQ, VL], mx, IsWorstCase>;
        }
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 698-718: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
        let Latency = 1, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in
        defm : LMULWriteResMX<"WriteVSSEG" # nf # "e" # eew,   [VCQ, VS], mx, IsWorstCase>;
      }
    }
  }
  foreach mx = SchedMxList in {
    foreach nf=2-8 in {
      foreach eew = [8, 16, 32, 64] in {
        defvar Cycles = SiFive7GetCyclesSegmented<mx, eew, nf, VLEN>.c;
        defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
        // Does not chain so set latency high
        let Latency = !add(3, Cycles), AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
          defm : LMULWriteResMX<"WriteVLSSEG" # nf # "e" # eew,  [VCQ, VL], mx, IsWorstCase>;
          defm : LMULWriteResMX<"WriteVLUXSEG" # nf # "e" # eew, [VCQ, VL], mx, IsWorstCase>;
          defm : LMULWriteResMX<"WriteVLOXSEG" # nf # "e" # eew, [VCQ, VL], mx, IsWorstCase>;
        }
        let Latency = 1, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
          defm : LMULWriteResMX<"WriteVSSSEG" # nf # "e" # eew,  [VCQ, VS], mx, IsWorstCase>;
          defm : LMULWriteResMX<"WriteVSUXSEG" # nf # "e" # eew, [VCQ, VS], mx, IsWorstCase>;
          defm : LMULWriteResMX<"WriteVSOXSEG" # nf # "e" # eew, [VCQ, VS], mx, IsWorstCase>;
        }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 719-744: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
      }
    }
  }

  // 11. Vector Integer Arithmetic Instructions
  foreach mx = SchedMxList in {
    defvar Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVIALUV",     [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIALUX",     [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIALUI",     [VCQ, VA1OrVA2], mx, IsWorstCase>;
      // vmadc requires mask
      defm : LMULWriteResMX<"WriteVICALUV",    [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVICALUX",    [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVICALUI",    [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVICALUMV",   [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVICALUMX",   [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVICALUMI",   [VCQ, VA1], mx, IsWorstCase>;
      // min max require merge
      defm : LMULWriteResMX<"WriteVIMinMaxV",  [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIMinMaxX",  [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIMergeV",   [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIMergeX",   [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIMergeI",   [VCQ, VA1], mx, IsWorstCase>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 745-765: Bulk record instantiation / 批量记录实例化
```tablegen
      defm : LMULWriteResMX<"WriteVIMovV",     [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIMovX",     [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIMovI",     [VCQ, VA1OrVA2], mx, IsWorstCase>;

      defm : LMULWriteResMX<"WriteVExtV",      [VCQ, VA1], mx, IsWorstCase>;
    }
    let Latency = 8, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVShiftV",    [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVShiftX",    [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVShiftI",    [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIMulV",     [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIMulX",     [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIMulAddV",  [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIMulAddX",  [VCQ, VA1OrVA2], mx, IsWorstCase>;
    }
    // Mask results can't chain.
    let Latency = !add(Cycles, 3), AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVICmpV",     [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVICmpX",     [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVICmpI",     [VCQ, VA1], mx, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 766-795: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  }

  foreach mx = SchedMxList in {
    foreach sew = SchedSEWSet<mx>.val in {
      // One bit at a time, but up to four elements can be processed at a time.
      // Add 3 to number of elements to ensure the group formed by remainder
      // elements are accounted for.
      defvar Cycles =
          !mul(sew, !div(!add(3, SiFive7GetCyclesOnePerElement<mx, sew, VLEN>.c), 4));
      defvar IsWorstCase = SiFive7IsWorstCaseMXSEW<mx, sew, SchedMxList>.c;
      let Latency = Cycles, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
        defm : LMULSEWWriteResMXSEW<"WriteVIDivV", [VCQ, VA1], mx, sew, IsWorstCase>;
        defm : LMULSEWWriteResMXSEW<"WriteVIDivX", [VCQ, VA1], mx, sew, IsWorstCase>;
      }
    }
  }

  // Widening
  foreach mx = SchedMxListW in {
    defvar Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxListW>.c;
    let Latency = 8, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVIWALUV",    [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIWALUX",    [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIWALUI",    [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIWMulV",    [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIWMulX",    [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIWMulAddV", [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIWMulAddX", [VCQ, VA1OrVA2], mx, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 796-823: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  }
  // Narrowing
  foreach mx = SchedMxListW in {
    defvar Cycles = SiFive7GetCyclesNarrowing<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxListW>.c;
    let Latency = 8, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVNShiftV",   [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVNShiftX",   [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVNShiftI",   [VCQ, VA1OrVA2], mx, IsWorstCase>;
    }
  }

  // 12. Vector Fixed-Point Arithmetic Instructions
  foreach mx = SchedMxList in {
    defvar Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    let Latency = 8, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVSALUV",   [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSALUX",   [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSALUI",   [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVAALUV",   [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVAALUX",   [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSMulV",   [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSMulX",   [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSShiftV", [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSShiftX", [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSShiftI", [VCQ, VA1OrVA2], mx, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 824-859: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  }
  // Narrowing
  foreach mx = SchedMxListW in {
    defvar Cycles = SiFive7GetCyclesNarrowing<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxListW>.c;
    let Latency = 8, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVNClipV",  [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVNClipX",  [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVNClipI",  [VCQ, VA1OrVA2], mx, IsWorstCase>;
    }
  }

  // 13. Vector Floating-Point Instructions
  foreach mx = SchedMxListF in {
    foreach sew = SchedSEWSet<mx, isF=1>.val in {
      defvar IsWorstCase = SiFive7IsWorstCaseMXSEW<mx, sew, SchedMxListF, isF=1>.c;
      if !eq(sew, 64) then {
        defvar SingleElementCycles = SiFive7GetCyclesOnePerElement<mx, sew, VLEN>.c;
        foreach SchedWriteName = ["WriteVFALUV", "WriteVFALUF", "WriteVFMulV", "WriteVFMulF",
                                  "WriteVFMulAddV", "WriteVFMulAddF"] in
        defm : LMULSEWWriteResMXSEWVariant<SchedWriteName, SingleElementVecFP64SchedPred,
                                           // Predicated
                                           [VCQ, VA1], !add(SingleElementCycles, 7), [0, 1], [1, !add(1, SingleElementCycles)],
                                           // Not Predicated
                                           [VCQ, VA1OrVA2], 8, [0, 1], [1, !add(1, SiFive7GetCyclesDefault<mx>.c)],
                                           mx, sew, IsWorstCase>;
        foreach SchedWriteName = ["WriteVFRecpV", "WriteVFCvtIToFV"] in
        defm : LMULSEWWriteResMXSEWVariant<SchedWriteName, SingleElementVecFP64SchedPred,
                                           // Predicated
                                           [VCQ, VA1], !add(SingleElementCycles, 7), [0, 1], [1, !add(1, SingleElementCycles)],
                                           // Not Predicated
                                           [VCQ, VA1], 8, [0, 1], [1, !add(1, SiFive7GetCyclesDefault<mx>.c)],
                                           mx, sew, IsWorstCase>;
        foreach SchedWriteName = ["WriteVFSgnjV", "WriteVFSgnjF"] in
        defm : LMULSEWWriteResMXSEWVariant<SchedWriteName, SingleElementVecFP64SchedPred,
                                           // Predicated
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 860-881: Bulk record instantiation SchedWriteName / 批量记录实例化 SchedWriteName
```tablegen
                                           [VCQ, VA1], !add(SingleElementCycles, 3), [0, 1], [1, !add(1, SingleElementCycles)],
                                           // Not Predicated
                                           [VCQ, VA1OrVA2], 4, [0, 1], [1, !add(1, SiFive7GetCyclesDefault<mx>.c)],
                                           mx, sew, IsWorstCase>;
        foreach SchedWriteName = ["WriteVFMinMaxV", "WriteVFMinMaxF"] in
        defm : LMULSEWWriteResMXSEWVariant<SchedWriteName, SingleElementVecFP64SchedPred,
                                           // Predicated
                                           [VCQ, VA1], !add(SingleElementCycles, 3), [0, 1], [1, !add(1, SingleElementCycles)],
                                           // Not Predicated
                                           [VCQ, VA1], 4, [0, 1], [1, !add(1, SiFive7GetCyclesDefault<mx>.c)],
                                           mx, sew, IsWorstCase>;
      } else {
        let Latency = 8, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, SiFive7GetCyclesDefault<mx>.c)] in {
          defm : LMULSEWWriteResMXSEW<"WriteVFALUV",  [VCQ, VA1OrVA2], mx, sew, IsWorstCase>;
          defm : LMULSEWWriteResMXSEW<"WriteVFALUF",  [VCQ, VA1OrVA2], mx, sew, IsWorstCase>;
          defm : LMULSEWWriteResMXSEW<"WriteVFMulV",  [VCQ, VA1OrVA2], mx, sew, IsWorstCase>;
          defm : LMULSEWWriteResMXSEW<"WriteVFMulF",  [VCQ, VA1OrVA2], mx, sew, IsWorstCase>;
          defm : LMULSEWWriteResMXSEW<"WriteVFMulAddV", [VCQ, VA1OrVA2], mx, sew, IsWorstCase>;
          defm : LMULSEWWriteResMXSEW<"WriteVFMulAddF", [VCQ, VA1OrVA2], mx, sew, IsWorstCase>;
          defm : LMULSEWWriteResMXSEW<"WriteVFRecpV",   [VCQ, VA1], mx, sew, IsWorstCase>;
          defm : LMULSEWWriteResMXSEW<"WriteVFCvtIToFV", [VCQ, VA1], mx, sew, IsWorstCase>;
        }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 882-902: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
        let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, SiFive7GetCyclesDefault<mx>.c)] in {
          defm : LMULSEWWriteResMXSEW<"WriteVFSgnjV",   [VCQ, VA1OrVA2], mx, sew, IsWorstCase>;
          defm : LMULSEWWriteResMXSEW<"WriteVFSgnjF",   [VCQ, VA1OrVA2], mx, sew, IsWorstCase>;
          // min max require merge
          defm : LMULSEWWriteResMXSEW<"WriteVFMinMaxV", [VCQ, VA1], mx, sew, IsWorstCase>;
          defm : LMULSEWWriteResMXSEW<"WriteVFMinMaxF", [VCQ, VA1], mx, sew, IsWorstCase>;
        }
      }
    }
  }
  foreach mx = SchedMxList in {
    defvar Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    let Latency = 8, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVFCvtFToIV",  [VCQ, VA1], mx, IsWorstCase>;
    }
    let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVFClassV",    [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVFMergeV",    [VCQ, VA1OrVA2], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVFMovV",      [VCQ, VA1OrVA2], mx, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 903-920: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
    // Mask results can't chain.
    let Latency = !add(Cycles, 3), AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      // fcmp requires mask
      defm : LMULWriteResMX<"WriteVFCmpV",      [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVFCmpF",      [VCQ, VA1], mx, IsWorstCase>;
    }
  }
  foreach mx = SchedMxListF in {
    foreach sew = SchedSEWSet<mx, isF=1>.val in {
      defvar Cycles = !mul(SiFive7GetDivOrSqrtFactor<sew>.c,
                           !div(SiFive7GetCyclesOnePerElement<mx, sew, VLEN>.c, 4));
      defvar IsWorstCase = SiFive7IsWorstCaseMXSEW<mx, sew, SchedMxListF, 1>.c;
      let Latency = Cycles, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
        defm : LMULSEWWriteResMXSEW<"WriteVFSqrtV", [VCQ, VA1], mx, sew, IsWorstCase>;
        defm : LMULSEWWriteResMXSEW<"WriteVFDivV",  [VCQ, VA1], mx, sew, IsWorstCase>;
        defm : LMULSEWWriteResMXSEW<"WriteVFDivF",  [VCQ, VA1], mx, sew, IsWorstCase>;
      }
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 921-940: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  }

  // Widening
  foreach mx = SchedMxListW in {
    foreach sew = SchedSEWSet<mx, isF=0, isWidening=1>.val in {
      defvar IsWorstCase = SiFive7IsWorstCaseMXSEW<mx, sew, SchedMxListW>.c;
      defvar DefaultCycles = SiFive7GetCyclesDefault<mx>.c;
      if !eq(sew, 32) then {
        defvar SingleElementCycles = SiFive7GetCyclesOnePerElement<mx, sew, VLEN>.c;
        defm : LMULSEWWriteResMXSEWVariant<"WriteVFWCvtIToFV", SingleElementVecFP64SchedPred,
                                           // Predicated
                                           [VCQ, VA1], 8, [0, 1], [1, !add(1, SingleElementCycles)],
                                           // Not Predicated
                                           [VCQ, VA1], 8, [0, 1], [1, !add(1, DefaultCycles)],
                                           mx, sew, IsWorstCase>;
      } else {
        let Latency = 8,
            AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, DefaultCycles)] in
        defm : LMULSEWWriteResMXSEW<"WriteVFWCvtIToFV", [VCQ, VA1], mx, sew, IsWorstCase>;
      }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 941-967: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
    }
  }
  foreach mx = SchedMxListFW in {
    foreach sew = SchedSEWSet<mx, isF=1, isWidening=1>.val in {
      defvar DefaultCycles = SiFive7GetCyclesDefault<mx>.c;
      defvar IsWorstCase = SiFive7IsWorstCaseMXSEW<mx, sew, SchedMxListFW, isF=1>.c;
      let Latency = 8, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, DefaultCycles)] in {
        defm : LMULSEWWriteResMXSEW<"WriteVFWALUV", [VCQ, VA1OrVA2], mx, sew, IsWorstCase>;
        defm : LMULSEWWriteResMXSEW<"WriteVFWALUF", [VCQ, VA1OrVA2], mx, sew, IsWorstCase>;
        defm : LMULSEWWriteResMXSEW<"WriteVFWMulV", [VCQ, VA1OrVA2], mx, sew, IsWorstCase>;
        defm : LMULSEWWriteResMXSEW<"WriteVFWMulF", [VCQ, VA1OrVA2], mx, sew, IsWorstCase>;
        defm : LMULSEWWriteResMXSEW<"WriteVFWMulAddV", [VCQ, VA1OrVA2], mx, sew, IsWorstCase>;
        defm : LMULSEWWriteResMXSEW<"WriteVFWMulAddF", [VCQ, VA1OrVA2], mx, sew, IsWorstCase>;
      }
      if !eq(sew, 32) then {
        defvar SingleElementCycles = SiFive7GetCyclesOnePerElement<mx, sew, VLEN>.c;
        defm : LMULSEWWriteResMXSEWVariant<"WriteVFWCvtFToFV", SingleElementVecFP64SchedPred,
                                           // Predicated
                                           [VCQ, VA1], 8, [0, 1], [1, !add(1, SingleElementCycles)],
                                           // Not Predicated
                                           [VCQ, VA1], 8, [0, 1], [1, !add(1, DefaultCycles)],
                                           mx, sew, IsWorstCase>;
      } else {
        let Latency = 8,
            AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, DefaultCycles)] in
        defm : LMULSEWWriteResMXSEW<"WriteVFWCvtFToFV", [VCQ, VA1], mx, sew, IsWorstCase>;
      }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 968-1000: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
    }
    defvar Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxListFW>.c;
    let Latency = 8, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in
    defm : LMULWriteResMX<"WriteVFWCvtFToIV", [VCQ, VA1], mx, IsWorstCase>;
  }
  // Narrowing
  foreach mx = SchedMxListW in {
    defvar Cycles = SiFive7GetCyclesNarrowing<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxListW>.c;
    let Latency = 8, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVFNCvtFToIV", [VCQ, VA1], mx, IsWorstCase>;
    }
  }
  foreach mx = SchedMxListFW in {
    foreach sew = SchedSEWSet<mx, isF=1, isWidening=1>.val in {
      defvar IsWorstCase = SiFive7IsWorstCaseMXSEW<mx, sew, SchedMxListFW, isF=1>.c;
      defvar DefaultCycles = SiFive7GetCyclesNarrowing<mx>.c;
      if !eq(sew, 32) then {
        defvar SingleElementCycles = SiFive7GetCyclesOnePerElement<mx, sew, VLEN>.c;
        foreach SchedWriteName = ["WriteVFNCvtIToFV", "WriteVFNCvtFToFV"] in
        defm : LMULSEWWriteResMXSEWVariant<SchedWriteName, SingleElementVecFP64SchedPred,
                                           // Predicated
                                           [VCQ, VA1], 8, [0, 1], [1, !add(1, SingleElementCycles)],
                                           // Not Predicated
                                           [VCQ, VA1], 8, [0, 1], [1, !add(1, DefaultCycles)],
                                           mx, sew, IsWorstCase>;
      } else {
        let Latency = 8,
            AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, DefaultCycles)] in {
          defm : LMULSEWWriteResMXSEW<"WriteVFNCvtIToFV", [VCQ, VA1], mx, sew, IsWorstCase>;
          defm : LMULSEWWriteResMXSEW<"WriteVFNCvtFToFV", [VCQ, VA1], mx, sew, IsWorstCase>;
        }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1001-1018: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
      }
    }
  }

  // 14. Vector Reduction Operations
  foreach mx = SchedMxList in {
    foreach sew = SchedSEWSet<mx>.val in {
      defvar Cycles = SiFive7GetReductionCycles<mx, sew, VLEN>.c;
      defvar IsWorstCase = SiFive7IsWorstCaseMXSEW<mx, sew, SchedMxList>.c;
      let Latency = Cycles, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
        defm : LMULSEWWriteResMXSEW<"WriteVIRedV_From", [VCQ, VA1],
                                       mx, sew, IsWorstCase>;
        defm : LMULSEWWriteResMXSEW<"WriteVIRedMinMaxV_From", [VCQ, VA1],
                                       mx, sew, IsWorstCase>;
      }
    }
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1019-1038: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  foreach mx = SchedMxListWRed in {
    foreach sew = SchedSEWSet<mx, 0, 1>.val in {
      defvar Cycles = SiFive7GetReductionCycles<mx, sew, VLEN>.c;
      defvar IsWorstCase = SiFive7IsWorstCaseMXSEW<mx, sew, SchedMxListWRed>.c;
      let Latency = Cycles, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in
      defm : LMULSEWWriteResMXSEW<"WriteVIWRedV_From", [VCQ, VA1],
                                     mx, sew, IsWorstCase>;
    }
  }

  foreach mx = SchedMxListF in {
    foreach sew = SchedSEWSet<mx, 1>.val in {
      defvar RedCycles = SiFive7GetReductionCycles<mx, sew, VLEN>.c;
      defvar IsWorstCase = SiFive7IsWorstCaseMXSEW<mx, sew, SchedMxListF, 1>.c;
      let Latency = RedCycles, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, RedCycles)] in {
        defm : LMULSEWWriteResMXSEW<"WriteVFRedV_From", [VCQ, VA1],
                                       mx, sew, IsWorstCase>;
        defm : LMULSEWWriteResMXSEW<"WriteVFRedMinMaxV_From", [VCQ, VA1],
                                       mx, sew, IsWorstCase>;
      }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1039-1057: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
      defvar OrdRedCycles = SiFive7GetOrderedReductionCycles<mx, sew, VLEN>.c;
      let Latency = OrdRedCycles, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, OrdRedCycles)] in
      defm : LMULSEWWriteResMXSEW<"WriteVFRedOV_From", [VCQ, VA1],
                                     mx, sew, IsWorstCase>;
    }
  }

  foreach mx = SchedMxListFWRed in {
    foreach sew = SchedSEWSet<mx, 1, 1>.val in {
      defvar RedCycles = SiFive7GetReductionCycles<mx, sew, VLEN>.c;
      defvar IsWorstCase = SiFive7IsWorstCaseMXSEW<mx, sew, SchedMxListFWRed, 1>.c;
      let Latency = RedCycles, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, RedCycles)] in
      defm : LMULSEWWriteResMXSEW<"WriteVFWRedV_From", [VCQ, VA1],
                                     mx, sew, IsWorstCase>;
      defvar OrdRedCycles = SiFive7GetOrderedReductionCycles<mx, sew, VLEN>.c;
      let Latency = OrdRedCycles, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, OrdRedCycles)] in
      defm : LMULSEWWriteResMXSEW<"WriteVFWRedOV_From", [VCQ, VA1],
                                     mx, sew, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1058-1077: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  }

  // 15. Vector Mask Instructions
  // Simple mask logical
  foreach mx = SchedMxList in {
    defvar Cycles = SiFive7GetCyclesVMask<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVMALUV", [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVMSFSV", [VCQ, VA1], mx, IsWorstCase>;
    }
  }
  // Simple mask logical used in series
  foreach mx = SchedMxList in {
    defvar Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVIotaV", [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVIdxV", [VCQ, VA1], mx, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1078-1096: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  }
  // Mask reduction
  foreach mx = SchedMxList in {
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    let Latency = 11, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 3)] in {
      defm "" : LMULWriteResMX<"WriteVMFFSV", [VCQ, VA1], mx, IsWorstCase>;
      defm "" : LMULWriteResMX<"WriteVMPopV", [VCQ, VA1], mx, IsWorstCase>;
    }
  }

  // 16. Vector Permutation Instructions
  let Latency = 11, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 3)] in {
    def : WriteRes<WriteVMovXS, [VCQ, VA1]>;
    def : WriteRes<WriteVMovFS, [VCQ, VA1]>;
  }
  let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 1)] in {
    def : WriteRes<WriteVMovSX, [VCQ, VA1OrVA2]>;
    def : WriteRes<WriteVMovSF, [VCQ, VA1OrVA2]>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1097-1117: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  foreach mx = SchedMxList in {
    defvar Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    let Latency = 8, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVRGatherVX",    [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVRGatherVI",    [VCQ, VA1], mx, IsWorstCase>;
    }
  }

  foreach mx = SchedMxList in {
    foreach sew = SchedSEWSet<mx>.val in {
      defvar IsWorstCase = SiFive7IsWorstCaseMXSEW<mx, sew, SchedMxList>.c;
      defvar IsSingleDLEN = isSingleDLEN<mx>.c;

      defvar GatherVVCycles =
        SiFive7GetCyclesVRGatherVV<mx, sew, VLEN, hasFastGather>.c;
      // 7 + DLEN/ SEW
      defvar SlowGatherLat = !add(7, !div(!div(VLEN, 2), sew));
      defvar GatherVVLat = !if(hasFastGather,
                              !add(3, GatherVVCycles), SlowGatherLat);
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1118-1135: Bulk record instantiation / 批量记录实例化
```tablegen
      let Latency = GatherVVLat, AcquireAtCycles = [0, 1],
          ReleaseAtCycles = [1, !add(5, GatherVVCycles)] in
      defm "" : LMULSEWWriteResMXSEW<"WriteVRGatherVV", [VCQ, VA1], mx, sew, IsWorstCase>;

      // VRGatherEI16VV is not improved by fastGather.
      defvar GatherEI16VVCycles = SiFive7GetCyclesOnePerElement<mx, sew, VLEN>.c;
      let Latency = SlowGatherLat, AcquireAtCycles = [0, 1],
          ReleaseAtCycles = [1, !add(5, GatherEI16VVCycles)] in
      defm "" : LMULSEWWriteResMXSEW<"WriteVRGatherEI16VV", [VCQ, VA1], mx, sew, IsWorstCase>;

      defvar CompressCycles = SiFive7GetCyclesVCompress<mx, sew, VLEN, hasFastGather>.c;
      defvar CompressLat = !if(!and(hasFastGather, IsSingleDLEN),
                               4,
                               !add(7, CompressCycles)); // 7 + VL
      let Latency = CompressLat, AcquireAtCycles = [0, 1],
          ReleaseAtCycles = [1, !add(8, CompressCycles)] in
      defm "" : LMULSEWWriteResMXSEW<"WriteVCompressV", [VCQ, VA1], mx, sew, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1136-1159: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  }

  foreach mx = SchedMxList in {
    defvar Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVSlideUpX",   [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSlideDownX", [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVSlideI",     [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVISlide1X",   [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVFSlide1F",   [VCQ, VA1], mx, IsWorstCase>;
    }
  }

  // VMov*V is LMUL Aware
  let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 2)] in
    def : WriteRes<WriteVMov1V,     [VCQ, VA1OrVA2]>;
  let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 4)] in
    def : WriteRes<WriteVMov2V,     [VCQ, VA1OrVA2]>;
  let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 8)] in
    def : WriteRes<WriteVMov4V,     [VCQ, VA1OrVA2]>;
  let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, 16)] in
    def : WriteRes<WriteVMov8V,     [VCQ, VA1OrVA2]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1160-1195: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  // Others
  def : WriteRes<WriteCSR, [PipeB]>;
  def : WriteRes<WriteNop, []>;
  let Latency = 3 in
    def : WriteRes<WriteRdVLENB, [PipeB]>;

  def : InstRW<[WriteIALU], (instrs COPY)>;

  // VCIX
  //
  // In principle we don't know the latency of any VCIX instructions (they
  // depends on a particular coprocessor implementation). However, the default
  // latency of 1 can lead to issues [1]. So instead we set the latency to the
  // default provided by `SiFive7GetCyclesDefault`. This is still not accurate
  // and can lead to suboptimal codegen, but should hopefully be a better
  // starting point.
  //
  // [1] https://github.com/llvm/llvm-project/issues/83391
  foreach mx = SchedMxList in {
    defvar Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxList>.c;
    let Latency = Cycles,
        AcquireAtCycles = [0, 1],
        ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm : LMULWriteResMX<"WriteVC_V_I",   [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_V_X",   [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_V_IV",  [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_V_VV",  [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_V_XV",  [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_V_IVV", [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_V_IVW", [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_V_VVV", [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_V_VVW", [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_V_XVV", [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_V_XVW", [VCQ, VA1], mx, IsWorstCase>;
      foreach f = ["FPR16", "FPR32", "FPR64"] in {
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1196-1215: Bulk record instantiation f / 批量记录实例化 f
```tablegen
        defm : LMULWriteResMX<"WriteVC_V_" # f # "V",  [VCQ, VA1], mx, IsWorstCase>;
        defm : LMULWriteResMX<"WriteVC_V_" # f # "VV", [VCQ, VA1], mx, IsWorstCase>;
        defm : LMULWriteResMX<"WriteVC_V_" # f # "VW", [VCQ, VA1], mx, IsWorstCase>;
      }
      defm : LMULWriteResMX<"WriteVC_I",   [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_X",   [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_IV",  [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_VV",  [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_XV",  [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_IVV", [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_IVW", [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_VVV", [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_VVW", [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_XVV", [VCQ, VA1], mx, IsWorstCase>;
      defm : LMULWriteResMX<"WriteVC_XVW", [VCQ, VA1], mx, IsWorstCase>;
      foreach f = ["FPR16", "FPR32", "FPR64"] in {
        defm : LMULWriteResMX<"WriteVC_" # f # "V",  [VCQ, VA1], mx, IsWorstCase>;
        defm : LMULWriteResMX<"WriteVC_" # f # "VV", [VCQ, VA1], mx, IsWorstCase>;
        defm : LMULWriteResMX<"WriteVC_" # f # "VW", [VCQ, VA1], mx, IsWorstCase>;
      }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1216-1236: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
    }
  }

  foreach mx = !listremove(SchedMxListW, ["M4"]) in {
    defvar Cycles = SiFive7GetSiFiveVFNRClipCycles<mx, VLEN>;
    let Latency = Cycles.latency,
        AcquireAtCycles = [0, 1],
        ReleaseAtCycles = [1, !add(1, Cycles.occupancy)] in
    defm : LMULWriteResMX<"WriteSF_VFNRClipV", [VCQ, VA1], mx,
                          IsWorstCase=!eq(mx, "M2")>;
  }

  // XSfvqmaccdod
  foreach mx = ["M1", "M2", "M4", "M8"] in {
    defvar Cycles = SiFive7GetCyclesDefault<mx>.c;
    let Latency = 8,
        AcquireAtCycles = [0, 1],
        ReleaseAtCycles = [1, !add(1, Cycles)] in
    defm : LMULWriteResMX<"WriteSF_VQMACC_DOD", [VCQ, VA1], mx,
                          IsWorstCase=!eq(mx, "M8")>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1237-1256: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen

  // XSfvqmaccqoq
  foreach mx = ["MF2", "M1", "M2", "M4"] in {
    defvar Cycles = SiFive7GetCyclesDefault<mx>.c;
    let Latency = 8,
        AcquireAtCycles = [0, 1],
        ReleaseAtCycles = [1, !add(1, Cycles)] in
    defm : LMULWriteResMX<"WriteSF_VQMACC_QOQ", [VCQ, VA1], mx,
                          IsWorstCase=!eq(mx, "M4")>;
  }

  // XSfvfwmaccqqq
  foreach mx = SchedMxListFW in {
    defvar Cycles = SiFive7GetCyclesDefault<mx>.c;
    defvar IsWorstCase = SiFive7IsWorstCaseMX<mx, SchedMxListFW>.c;
    let Latency = 8,
        AcquireAtCycles = [0, 1],
        ReleaseAtCycles = [1, !add(1, Cycles)] in
    defm : LMULWriteResMX<"WriteSF_VFWMACC_QQQ", [VCQ, VA1], mx, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1257-1292: Reusable TableGen multiclass SiFive7ReadAdvance / 可复用的 TableGen 多类 SiFive7ReadAdvance
```tablegen
}

//===----------------------------------------------------------------------===//

multiclass SiFive7ReadAdvance {
  // Bypass and advance
  def : SiFive7AnyToGPRBypass<ReadJmp>;
  def : SiFive7AnyToGPRBypass<ReadJalr>;
  def : ReadAdvance<ReadCSR, 0>;
  def : SiFive7AnyToGPRBypass<ReadStoreData>;
  def : ReadAdvance<ReadMemBase, 0>;
  def : SiFive7AnyToGPRBypass<ReadIALU>;
  def : SiFive7AnyToGPRBypass<ReadIALU32>;
  def : SiFive7AnyToGPRBypass<ReadShiftImm>;
  def : SiFive7AnyToGPRBypass<ReadShiftImm32>;
  def : SiFive7AnyToGPRBypass<ReadShiftReg>;
  def : SiFive7AnyToGPRBypass<ReadShiftReg32>;
  def : ReadAdvance<ReadIDiv, 0>;
  def : ReadAdvance<ReadIDiv32, 0>;
  def : ReadAdvance<ReadIRem, 0>;
  def : ReadAdvance<ReadIRem32, 0>;
  def : ReadAdvance<ReadIMul, 0>;
  def : ReadAdvance<ReadIMul32, 0>;
  def : ReadAdvance<ReadAtomicWA, 0>;
  def : ReadAdvance<ReadAtomicWD, 0>;
  def : ReadAdvance<ReadAtomicDA, 0>;
  def : ReadAdvance<ReadAtomicDD, 0>;
  def : ReadAdvance<ReadAtomicLDW, 0>;
  def : ReadAdvance<ReadAtomicLDD, 0>;
  def : ReadAdvance<ReadAtomicSTW, 0>;
  def : ReadAdvance<ReadAtomicSTD, 0>;
  def : ReadAdvance<ReadFStoreData, 0>;
  def : ReadAdvance<ReadFMemBase, 0>;
  def : ReadAdvance<ReadFAdd16, 0>;
  def : ReadAdvance<ReadFAdd32, 0>;
  def : ReadAdvance<ReadFAdd64, 0>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 1293-1328: TableGen record definition / TableGen 记录定义
```tablegen
  def : ReadAdvance<ReadFMul16, 0>;
  def : ReadAdvance<ReadFMA16, 0>;
  def : ReadAdvance<ReadFMA16Addend, 0>;
  def : ReadAdvance<ReadFMul32, 0>;
  def : ReadAdvance<ReadFMul64, 0>;
  def : ReadAdvance<ReadFMA32, 0>;
  def : ReadAdvance<ReadFMA32Addend, 0>;
  def : ReadAdvance<ReadFMA64, 0>;
  def : ReadAdvance<ReadFMA64Addend, 0>;
  def : ReadAdvance<ReadFDiv16, 0>;
  def : ReadAdvance<ReadFDiv32, 0>;
  def : ReadAdvance<ReadFDiv64, 0>;
  def : ReadAdvance<ReadFSqrt16, 0>;
  def : ReadAdvance<ReadFSqrt32, 0>;
  def : ReadAdvance<ReadFSqrt64, 0>;
  def : ReadAdvance<ReadFCmp16, 0>;
  def : ReadAdvance<ReadFCmp32, 0>;
  def : ReadAdvance<ReadFCmp64, 0>;
  def : ReadAdvance<ReadFSGNJ16, 0>;
  def : ReadAdvance<ReadFSGNJ32, 0>;
  def : ReadAdvance<ReadFSGNJ64, 0>;
  def : ReadAdvance<ReadFMinMax16, 0>;
  def : ReadAdvance<ReadFMinMax32, 0>;
  def : ReadAdvance<ReadFMinMax64, 0>;
  def : ReadAdvance<ReadFCvtF16ToI32, 0>;
  def : ReadAdvance<ReadFCvtF16ToI64, 0>;
  def : ReadAdvance<ReadFCvtF32ToI32, 0>;
  def : ReadAdvance<ReadFCvtF32ToI64, 0>;
  def : ReadAdvance<ReadFCvtF64ToI32, 0>;
  def : ReadAdvance<ReadFCvtF64ToI64, 0>;
  def : ReadAdvance<ReadFCvtI32ToF16, 0>;
  def : ReadAdvance<ReadFCvtI32ToF32, 0>;
  def : ReadAdvance<ReadFCvtI32ToF64, 0>;
  def : ReadAdvance<ReadFCvtI64ToF16, 0>;
  def : ReadAdvance<ReadFCvtI64ToF32, 0>;
  def : ReadAdvance<ReadFCvtI64ToF64, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 1329-1347: TableGen record definition / TableGen 记录定义
```tablegen
  def : ReadAdvance<ReadFCvtF32ToF64, 0>;
  def : ReadAdvance<ReadFCvtF64ToF32, 0>;
  def : ReadAdvance<ReadFCvtF16ToF32, 0>;
  def : ReadAdvance<ReadFCvtF32ToF16, 0>;
  def : ReadAdvance<ReadFCvtF16ToF64, 0>;
  def : ReadAdvance<ReadFCvtF64ToF16, 0>;
  def : ReadAdvance<ReadFMovF16ToI16, 0>;
  def : ReadAdvance<ReadFMovI16ToF16, 0>;
  def : ReadAdvance<ReadFMovF32ToI32, 0>;
  def : ReadAdvance<ReadFMovI32ToF32, 0>;
  def : ReadAdvance<ReadFMovF64ToI64, 0>;
  def : ReadAdvance<ReadFMovI64ToF64, 0>;
  def : ReadAdvance<ReadFClass16, 0>;
  def : ReadAdvance<ReadFClass32, 0>;
  def : ReadAdvance<ReadFClass64, 0>;

  def : SiFive7AnyToGPRBypass<ReadSFBJmp, 0>;
  def : SiFive7AnyToGPRBypass<ReadSFBALU, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 1348-1367: TableGen record definition / TableGen 记录定义
```tablegen
  // Bitmanip
  def : SiFive7AnyToGPRBypass<ReadRotateImm>;
  def : SiFive7AnyToGPRBypass<ReadRotateImm32>;
  def : SiFive7AnyToGPRBypass<ReadRotateReg>;
  def : SiFive7AnyToGPRBypass<ReadRotateReg32>;
  def : SiFive7AnyToGPRBypass<ReadCLZ>;
  def : SiFive7AnyToGPRBypass<ReadCLZ32>;
  def : SiFive7AnyToGPRBypass<ReadCTZ>;
  def : SiFive7AnyToGPRBypass<ReadCTZ32>;
  def : ReadAdvance<ReadCPOP, 0>;
  def : ReadAdvance<ReadCPOP32, 0>;
  def : SiFive7AnyToGPRBypass<ReadORCB>;
  def : SiFive7AnyToGPRBypass<ReadIMinMax>;
  def : SiFive7AnyToGPRBypass<ReadREV8>;
  def : SiFive7AnyToGPRBypass<ReadSHXADD>;
  def : SiFive7AnyToGPRBypass<ReadSHXADD32>;
  // Single-bit instructions
  def : SiFive7AnyToGPRBypass<ReadSingleBit>;
  def : SiFive7AnyToGPRBypass<ReadSingleBitImm>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 1368-1403: Bulk record instantiation / 批量记录实例化
```tablegen
  // 6. Configuration-Setting Instructions
  def : ReadAdvance<ReadVSETVLI, 2>;
  def : ReadAdvance<ReadVSETVL, 2>;

  // 7. Vector Loads and Stores
  def : ReadAdvance<ReadVLDX, 0>;
  def : ReadAdvance<ReadVSTX, 0>;
  defm : LMULReadAdvance<"ReadVSTEV", 0>;
  defm : LMULReadAdvance<"ReadVSTM", 0>;
  def : ReadAdvance<ReadVLDSX, 0>;
  def : ReadAdvance<ReadVSTSX, 0>;
  defm : LMULReadAdvance<"ReadVSTS8V", 0>;
  defm : LMULReadAdvance<"ReadVSTS16V", 0>;
  defm : LMULReadAdvance<"ReadVSTS32V", 0>;
  defm : LMULReadAdvance<"ReadVSTS64V", 0>;
  defm : LMULReadAdvance<"ReadVLDUXV", 0>;
  defm : LMULReadAdvance<"ReadVLDOXV", 0>;
  defm : LMULReadAdvance<"ReadVSTUX8", 0>;
  defm : LMULReadAdvance<"ReadVSTUX16", 0>;
  defm : LMULReadAdvance<"ReadVSTUX32", 0>;
  defm : LMULReadAdvance<"ReadVSTUX64", 0>;
  defm : LMULReadAdvance<"ReadVSTUXV", 0>;
  defm : LMULReadAdvance<"ReadVSTUX8V", 0>;
  defm : LMULReadAdvance<"ReadVSTUX16V", 0>;
  defm : LMULReadAdvance<"ReadVSTUX32V", 0>;
  defm : LMULReadAdvance<"ReadVSTUX64V", 0>;
  defm : LMULReadAdvance<"ReadVSTOX8", 0>;
  defm : LMULReadAdvance<"ReadVSTOX16", 0>;
  defm : LMULReadAdvance<"ReadVSTOX32", 0>;
  defm : LMULReadAdvance<"ReadVSTOX64", 0>;
  defm : LMULReadAdvance<"ReadVSTOXV", 0>;
  defm : LMULReadAdvance<"ReadVSTOX8V", 0>;
  defm : LMULReadAdvance<"ReadVSTOX16V", 0>;
  defm : LMULReadAdvance<"ReadVSTOX32V", 0>;
  defm : LMULReadAdvance<"ReadVSTOX64V", 0>;
  // LMUL Aware
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1404-1439: Bulk record instantiation / 批量记录实例化
```tablegen
  def : ReadAdvance<ReadVST1R, 0>;
  def : ReadAdvance<ReadVST2R, 0>;
  def : ReadAdvance<ReadVST4R, 0>;
  def : ReadAdvance<ReadVST8R, 0>;

  // 11. Vector Integer Arithmetic Instructions
  defm : LMULReadAdvance<"ReadVIALUV", 0>;
  defm : LMULReadAdvance<"ReadVIALUX", 0>;
  defm : LMULReadAdvanceW<"ReadVIWALUV", 0>;
  defm : LMULReadAdvanceW<"ReadVIWALUX", 0>;
  defm : LMULReadAdvance<"ReadVExtV", 0>;
  defm : LMULReadAdvance<"ReadVICALUV", 0>;
  defm : LMULReadAdvance<"ReadVICALUX", 0>;
  defm : LMULReadAdvance<"ReadVShiftV", 0>;
  defm : LMULReadAdvance<"ReadVShiftX", 0>;
  defm : LMULReadAdvanceW<"ReadVNShiftV", 0>;
  defm : LMULReadAdvanceW<"ReadVNShiftX", 0>;
  defm : LMULReadAdvance<"ReadVICmpV", 0>;
  defm : LMULReadAdvance<"ReadVICmpX", 0>;
  defm : LMULReadAdvance<"ReadVIMinMaxV", 0>;
  defm : LMULReadAdvance<"ReadVIMinMaxX", 0>;
  defm : LMULReadAdvance<"ReadVIMulV", 0>;
  defm : LMULReadAdvance<"ReadVIMulX", 0>;
  defm : LMULSEWReadAdvance<"ReadVIDivV", 0>;
  defm : LMULSEWReadAdvance<"ReadVIDivX", 0>;
  defm : LMULReadAdvanceW<"ReadVIWMulV", 0>;
  defm : LMULReadAdvanceW<"ReadVIWMulX", 0>;
  defm : LMULReadAdvance<"ReadVIMulAddV", 0>;
  defm : LMULReadAdvance<"ReadVIMulAddX", 0>;
  defm : LMULReadAdvanceW<"ReadVIWMulAddV", 0>;
  defm : LMULReadAdvanceW<"ReadVIWMulAddX", 0>;
  defm : LMULReadAdvance<"ReadVIMergeV", 0>;
  defm : LMULReadAdvance<"ReadVIMergeX", 0>;
  defm : LMULReadAdvance<"ReadVIMovV", 0>;
  defm : LMULReadAdvance<"ReadVIMovX", 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1440-1475: Bulk record instantiation / 批量记录实例化
```tablegen
  // 12. Vector Fixed-Point Arithmetic Instructions
  defm : LMULReadAdvance<"ReadVSALUV", 0>;
  defm : LMULReadAdvance<"ReadVSALUX", 0>;
  defm : LMULReadAdvance<"ReadVAALUV", 0>;
  defm : LMULReadAdvance<"ReadVAALUX", 0>;
  defm : LMULReadAdvance<"ReadVSMulV", 0>;
  defm : LMULReadAdvance<"ReadVSMulX", 0>;
  defm : LMULReadAdvance<"ReadVSShiftV", 0>;
  defm : LMULReadAdvance<"ReadVSShiftX", 0>;
  defm : LMULReadAdvanceW<"ReadVNClipV", 0>;
  defm : LMULReadAdvanceW<"ReadVNClipX", 0>;

  // 13. Vector Floating-Point Instructions
  defm : LMULSEWReadAdvanceF<"ReadVFALUV", 0>;
  defm : LMULSEWReadAdvanceF<"ReadVFALUF", 0>;
  defm : LMULSEWReadAdvanceFW<"ReadVFWALUV", 0>;
  defm : LMULSEWReadAdvanceFW<"ReadVFWALUF", 0>;
  defm : LMULSEWReadAdvanceF<"ReadVFMulV", 0>;
  defm : LMULSEWReadAdvanceF<"ReadVFMulF", 0>;
  defm : LMULSEWReadAdvanceF<"ReadVFDivV", 0>;
  defm : LMULSEWReadAdvanceF<"ReadVFDivF", 0>;
  defm : LMULSEWReadAdvanceFW<"ReadVFWMulV", 0>;
  defm : LMULSEWReadAdvanceFW<"ReadVFWMulF", 0>;
  defm : LMULSEWReadAdvanceF<"ReadVFMulAddV", 0>;
  defm : LMULSEWReadAdvanceF<"ReadVFMulAddF", 0>;
  defm : LMULSEWReadAdvanceFW<"ReadVFWMulAddV", 0>;
  defm : LMULSEWReadAdvanceFW<"ReadVFWMulAddF", 0>;
  defm : LMULSEWReadAdvanceF<"ReadVFSqrtV", 0>;
  defm : LMULSEWReadAdvanceF<"ReadVFRecpV", 0>;
  defm : LMULSEWReadAdvanceF<"ReadVFMinMaxV", 0>;
  defm : LMULSEWReadAdvanceF<"ReadVFMinMaxF", 0>;
  defm : LMULSEWReadAdvanceF<"ReadVFSgnjV", 0>;
  defm : LMULSEWReadAdvanceF<"ReadVFSgnjF", 0>;
  defm : LMULReadAdvance<"ReadVFCmpV", 0>;
  defm : LMULReadAdvance<"ReadVFCmpF", 0>;
  defm : LMULReadAdvance<"ReadVFClassV", 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1476-1501: Bulk record instantiation / 批量记录实例化
```tablegen
  defm : LMULReadAdvance<"ReadVFMergeV", 0>;
  defm : LMULReadAdvance<"ReadVFMergeF", 0>;
  defm : LMULReadAdvance<"ReadVFMovF", 0>;
  defm : LMULSEWReadAdvanceF<"ReadVFCvtIToFV", 0>;
  defm : LMULReadAdvance<"ReadVFCvtFToIV", 0>;
  defm : LMULSEWReadAdvanceW<"ReadVFWCvtIToFV", 0>;
  defm : LMULReadAdvanceFW<"ReadVFWCvtFToIV", 0>;
  defm : LMULSEWReadAdvanceFW<"ReadVFWCvtFToFV", 0>;
  defm : LMULSEWReadAdvanceFW<"ReadVFNCvtIToFV", 0>;
  defm : LMULReadAdvanceW<"ReadVFNCvtFToIV", 0>;
  defm : LMULSEWReadAdvanceFW<"ReadVFNCvtFToFV", 0>;

  // 14. Vector Reduction Operations
  def : ReadAdvance<ReadVIRedV, 0>;
  def : ReadAdvance<ReadVIRedV0, 0>;
  def : ReadAdvance<ReadVIWRedV, 0>;
  def : ReadAdvance<ReadVIWRedV0, 0>;
  def : ReadAdvance<ReadVFRedV, 0>;
  def : ReadAdvance<ReadVFRedV0, 0>;
  def : ReadAdvance<ReadVFRedOV, 0>;
  def : ReadAdvance<ReadVFRedOV0, 0>;
  def : ReadAdvance<ReadVFWRedV, 0>;
  def : ReadAdvance<ReadVFWRedV0, 0>;
  def : ReadAdvance<ReadVFWRedOV, 0>;
  def : ReadAdvance<ReadVFWRedOV0, 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1502-1533: Bulk record instantiation / 批量记录实例化
```tablegen
  // 15. Vector Mask Instructions
  defm : LMULReadAdvance<"ReadVMALUV", 0>;
  defm : LMULReadAdvance<"ReadVMPopV", 0>;
  defm : LMULReadAdvance<"ReadVMFFSV", 0>;
  defm : LMULReadAdvance<"ReadVMSFSV", 0>;
  defm : LMULReadAdvance<"ReadVIotaV", 0>;

  // 16. Vector Permutation Instructions
  def : ReadAdvance<ReadVMovXS, 0>;
  def : ReadAdvance<ReadVMovSX_V, 0>;
  def : ReadAdvance<ReadVMovSX_X, 0>;
  def : ReadAdvance<ReadVMovFS, 0>;
  def : ReadAdvance<ReadVMovSF_V, 0>;
  def : ReadAdvance<ReadVMovSF_F, 0>;
  defm : LMULReadAdvance<"ReadVISlideV", 0>;
  defm : LMULReadAdvance<"ReadVISlideX", 0>;
  defm : LMULReadAdvance<"ReadVFSlideV", 0>;
  defm : LMULReadAdvance<"ReadVFSlideF", 0>;
  defm : LMULSEWReadAdvance<"ReadVRGatherVV_data", 0>;
  defm : LMULSEWReadAdvance<"ReadVRGatherVV_index", 0>;
  defm : LMULSEWReadAdvance<"ReadVRGatherEI16VV_data", 0>;
  defm : LMULSEWReadAdvance<"ReadVRGatherEI16VV_index", 0>;
  defm : LMULReadAdvance<"ReadVRGatherVX_data", 0>;
  defm : LMULReadAdvance<"ReadVRGatherVX_index", 0>;
  defm : LMULReadAdvance<"ReadVRGatherVI_data", 0>;
  defm : LMULSEWReadAdvance<"ReadVCompressV", 0>;
  // LMUL Aware
  def : ReadAdvance<ReadVMov1V, 0>;
  def : ReadAdvance<ReadVMov2V, 0>;
  def : ReadAdvance<ReadVMov4V, 0>;
  def : ReadAdvance<ReadVMov8V, 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1534-1551: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  // XSfvfnrclipxfqf
  defm : LMULReadAdvance<"ReadSF_VFNRClipV", 0>;
  defm : LMULReadAdvance<"ReadSF_VFNRClipF", 0>;

  // SiFive VMACC
  defm : LMULReadAdvance<"ReadSF_VQMACC_DOD",  0>;
  defm : LMULReadAdvance<"ReadSF_VQMACC_QOQ",  0>;
  defm : LMULReadAdvance<"ReadSF_VFWMACC_QQQ", 0>;

  // Others
  def : ReadAdvance<ReadVMask, 0>;
  def : ReadAdvance<ReadVPassthru_WorstCase, 0>;
  foreach mx = SchedMxList in {
    def : ReadAdvance<!cast<SchedRead>("ReadVPassthru_" # mx), 0>;
    foreach sew = SchedSEWSet<mx>.val in
      def : ReadAdvance<!cast<SchedRead>("ReadVPassthru_" # mx  # "_E" # sew), 0>;
  }
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1552-1577: Reusable TableGen multiclass is / 可复用的 TableGen 多类 is
```tablegen

//===----------------------------------------------------------------------===//

/// This multiclass is a "bundle" of (1) processor resources (i.e. pipes) and
/// (2) WriteRes entries. It's parameterized by config values that will
/// eventually be supplied by different SchedMachineModels.
multiclass SiFive7SchedResources<int vlen, bit dualVALU,
                                 SiFive7FPLatencies fpLatencies,
                                 bit hasFastGather> {
  defm SiFive7 : SiFive7ProcResources<dualVALU>;

  // Pull out defs from SiFive7ProcResources so we can refer to them by name.
  defvar SiFive7PipeA = !cast<ProcResource>(NAME # "SiFive7PipeA");
  defvar SiFive7PipeB = !cast<ProcResource>(NAME # "SiFive7PipeB");
  defvar SiFive7PipeAB = !cast<ProcResGroup>(NAME # "SiFive7PipeAB");
  defvar SiFive7IDiv = !cast<ProcResource>(NAME # "SiFive7IDiv");
  defvar SiFive7FDiv = !cast<ProcResource>(NAME # "SiFive7FDiv");
  defvar SiFive7VA1 = !cast<ProcResource>(NAME # "SiFive7VA1");
  // Use SiFive7VA1 for VA1OrVA2 if there is only 1 VALU.
  defvar SiFive7VA1OrVA2 = !if (dualVALU,
                                !cast<ProcResGroup>(NAME # "SiFive7VA1OrVA2"),
                                !cast<ProcResource>(NAME # "SiFive7VA1"));
  defvar SiFive7VL = !cast<ProcResource>(NAME # "SiFive7VL");
  defvar SiFive7VS = !cast<ProcResource>(NAME # "SiFive7VS");
  defvar SiFive7VCQ = !cast<ProcResource>(NAME # "SiFive7VCQ");
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 1578-1603: Bulk record instantiation SiFive7 / 批量记录实例化 SiFive7
```tablegen
  // Define WriteRes records that are the same across all SiFive7 derived
  // SchedModels.
  defm SiFive7
      : SiFive7WriteResBase<vlen, SiFive7PipeA, SiFive7PipeB, SiFive7PipeAB,
                            SiFive7IDiv, SiFive7FDiv, SiFive7VA1,
                            SiFive7VA1OrVA2, SiFive7VL, SiFive7VS,
                            SiFive7VCQ, fpLatencies, hasFastGather>;

  //===----------------------------------------------------------------------===//
  // Bypass and advance

  defm SiFive7 : SiFive7ReadAdvance;
  //===----------------------------------------------------------------------===//
  // Unsupported extensions
  defm : UnsupportedSchedQ;
  // TODO: scheduling info of XSfvfexp* and XSfvfexpa*
  // for SiFive7 will be added in follow-up patches.
  defm : UnsupportedSchedXSfvfexp;
  defm : UnsupportedSchedXSfvfexpa;
  defm : UnsupportedSchedZabha;
  defm : UnsupportedSchedZbc;
  defm : UnsupportedSchedZbkb;
  defm : UnsupportedSchedZbkx;
  defm : UnsupportedSchedZfa;
  defm : UnsupportedSchedZvk;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1604-1621: TableGen class SiFive7SchedMachineModel<int / TableGen 类 SiFive7SchedMachineModel<int
```tablegen

class SiFive7SchedMachineModel<int vlen> : SchedMachineModel {
  let MicroOpBufferSize = 0; // Explicitly set to zero since SiFive7 is in-order.
  let IssueWidth = 2;        // 2 micro-ops are dispatched per cycle.
  let LoadLatency = 3;
  let MispredictPenalty = 3;
  let CompleteModel = 0;
  let EnableIntervals = true;
  let UnsupportedFeatures = [HasStdExtZbkb, HasStdExtZbkc, HasStdExtZbkx,
                             HasStdExtZcmt, HasStdExtZknd, HasStdExtZkne,
                             HasStdExtZknh, HasStdExtZksed, HasStdExtZksh,
                             HasStdExtZkr];
  int VLEN = vlen;
  bit HasDualVALU = false;

  SiFive7FPLatencies FPLatencies;
  bit HasFastGather = false;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 1622-1642: TableGen record SiFive7DefaultFPLatencies / TableGen 记录 SiFive7DefaultFPLatencies
```tablegen
  string Name = !subst("Model", "", !subst("SiFive7", "", NAME));
}

/// Auxiliary config values.
def SiFive7DefaultFPLatencies : SiFive7FPLatencies {
  let BasicFP16ALU = 5;
  let BasicFP32ALU = 5;
  let BasicFP64ALU = 7;
}

def SiFive7LowFPLatencies : SiFive7FPLatencies {
  let BasicFP16ALU = 4;
  let BasicFP32ALU = 4;
  let BasicFP64ALU = 4;
}

/// Models
def SiFive7VLEN128X100Model : SiFive7SchedMachineModel<128> {
  let FPLatencies = SiFive7LowFPLatencies;
  let HasFastGather = true;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 1643-1661: Bulk record instantiation SiFive7VLEN512Model / 批量记录实例化 SiFive7VLEN512Model
```tablegen

def SiFive7VLEN512Model : SiFive7SchedMachineModel<512> {
  let FPLatencies = SiFive7DefaultFPLatencies;
}

def SiFive7VLEN1024X300Model : SiFive7SchedMachineModel<1024> {
  let HasDualVALU = true;
  let FPLatencies = SiFive7LowFPLatencies;
  let HasFastGather = true;
}

/// Binding models to their scheduling resources.
foreach model = [SiFive7VLEN128X100Model, SiFive7VLEN512Model,
                 SiFive7VLEN1024X300Model] in {
  let SchedModel = model in
  defm model.Name : SiFive7SchedResources<model.VLEN, model.HasDualVALU,
                                          model.FPLatencies,
                                          model.HasFastGather>;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1662-1666: Definitions and supporting logic / 定义与支撑逻辑
```tablegen

// Some model name aliases.
defvar SiFive7Model = SiFive7VLEN512Model;
defvar SiFiveX100Model = SiFive7VLEN128X100Model;
defvar SiFiveX390Model = SiFive7VLEN1024X300Model;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

## Key Concepts / 关键概念
- **Scheduling models** / **调度模型**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
