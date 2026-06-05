# SISchedule.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SISchedule.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This TableGen DSL file defines SISchedule records for the LLVM AMDGPU backend. It describes target features, instructions, predicates, or generated metadata consumed by LLVM table generators. / 该 TableGen DSL 文件为 LLVM AMDGPU 后端定义 SISchedule 相关记录。它描述目标特性、指令、谓词或供 LLVM 表生成器使用的元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: File banner, comments, and TableGen overview
```tablegen
//===-- SISchedule.td - SI Scheduling definitions -------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// MachineModel definitions for Southern Islands (SI)
//
//===----------------------------------------------------------------------===//

def : PredicateProlog<[{
  const SIInstrInfo *TII =
    static_cast<const SIInstrInfo*>(SchedModel->getInstrInfo());
  (void)TII;
}]>;

def WriteBranch : SchedWrite;
def WriteExport : SchedWrite;
def WriteLDS    : SchedWrite;
def WriteSALU   : SchedWrite;
def WriteSMEM   : SchedWrite;
def WriteVMEM   : SchedWrite;
def WriteBarrier : SchedWrite;

def MIVGPRRead  : SchedRead;
def MIMFMARead  : SchedRead;

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins. Main symbols: `WriteBranch`, `WriteExport`, `WriteLDS`.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。 主要符号：`WriteBranch`, `WriteExport`, `WriteLDS`。

### Lines 30-59: Defines TableGen record Write32Bit
```tablegen
// Normal 16 or 32 bit VALU instructions
def Write32Bit         : SchedWrite;
// Conversion to or from F32 (but not converting F64 to or from F32)
def WriteFloatCvt      : SchedWrite;
// F16 or F32 transcendental instructions (these are quarter rate)
def WriteTrans32       : SchedWrite;
// Other quarter rate VALU instructions
def WriteQuarterRate32 : SchedWrite;

def WriteFloatFMA   : SchedWrite;

// Slow quarter rate f64 instruction.
def WriteDouble : SchedWrite;

// half rate f64 instruction (same as v_add_f64)
def WriteDoubleAdd  : SchedWrite;

// Conversion to or from f64 instruction
def WriteDoubleCvt  : SchedWrite;

// F64 "transcendental" (actually only reciprocal and/or square root)
// instructions
def WriteTrans64    : SchedWrite;

// Half rate 64-bit instructions.
def Write64Bit : SchedWrite;

// Integer multiplications.
def WriteIntMul : SchedWrite;

```
**EN:** This section contains concrete logic for TableGen record Write32Bit. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `Write32Bit`, `WriteFloatCvt`, `WriteTrans32`.
**CN:** 本节包含与 TableGen record Write32Bit 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`Write32Bit`, `WriteFloatCvt`, `WriteTrans32`。

### Lines 60-90: Defines TableGen record Write2PassMAI
```tablegen
// mAI multipass instructions.
def Write2PassMAI  : SchedWrite;
def Write4PassMAI  : SchedWrite;
def Write8PassMAI  : SchedWrite;
def Write16PassMAI : SchedWrite;
def Write4PassDGEMM : SchedWrite;
def Write8PassDGEMM : SchedWrite;
def Write16PassDGEMM : SchedWrite;

// WMMA/SWMMA instructions
def WriteXDL2PassWMMA : SchedWrite;
def WriteXDL4PassWMMA : SchedWrite;
def Write4PassWMMA : SchedWrite;
def Write8PassWMMA : SchedWrite;
def Write16PassWMMA : SchedWrite;

// Scalar float instructions
def WriteSFPU : SchedWrite;

// F16 or F32 pseudo scalar transcendental instructions
def WritePseudoScalarTrans : SchedWrite;

// Some instructions have multiple explicit defs. If we provide actual 
// SchedWrites for each of the explicit defs, then the instruction will 
// be encoded as having multiple MicroOps. The scheduler treats instructions
// with multiple MicroOps as taking multiple cycles to issue. To avoid this
// unwanted scheduling behavior, we can associate defs with DummyWrite, which
// don't have any MicroOps
def WriteVALUDummy : SchedWrite;
def WriteSALUDummy : SchedWrite;

```
**EN:** This section contains concrete logic for TableGen record Write2PassMAI. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `Write2PassMAI`, `Write4PassMAI`, `Write8PassMAI`.
**CN:** 本节包含与 TableGen record Write2PassMAI 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`Write2PassMAI`, `Write4PassMAI`, `Write8PassMAI`。

### Lines 91-119: Defines TableGen class SISchedMachineModel
```tablegen
// FIXME: Should there be a class for instructions which are VALU
// instructions and have VALU rates, but write to the SALU (i.e. VOPC
// instructions)

class SISchedMachineModel : SchedMachineModel {
  let CompleteModel = 1;
  // MicroOpBufferSize = 1 means that instructions will always be added
  // the ready queue when they become available.  This exposes them
  // to the register pressure analysis.
  let MicroOpBufferSize = 1;
  let IssueWidth = 1;
  let PostRAScheduler = 1;

  // FIXME:Approximate 2 * branch cost.  Try to hack around bad
  // early-ifcvt heuristics. These need improvement to avoid the OOE
  // heuristics.
  int MispredictPenalty = 20;
}

def SIFullSpeedModel : SISchedMachineModel;
def SIQuarterSpeedModel : SISchedMachineModel;
def SIDPFullSpeedModel : SISchedMachineModel;
def SIDPGFX942FullSpeedModel : SISchedMachineModel;
def SIDPGFX950FullSpeedModel : SISchedMachineModel;
def GFX10SpeedModel : SISchedMachineModel;
def GFX11SpeedModel : SISchedMachineModel;
def GFX12SpeedModel : SISchedMachineModel;
def GFX1250SpeedModel : SISchedMachineModel;

```
**EN:** This section contains concrete logic for TableGen class SISchedMachineModel. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SISchedMachineModel`, `SIFullSpeedModel`, `SIQuarterSpeedModel`.
**CN:** 本节包含与 TableGen class SISchedMachineModel 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SISchedMachineModel`, `SIFullSpeedModel`, `SIQuarterSpeedModel`。

### Lines 120-153: Defines TableGen record HWBranch
```tablegen
// XXX: Are the resource counts correct?
def HWBranch : ProcResource<1> {
  let BufferSize = 1;
}
def HWExport : ProcResource<1> {
  let BufferSize = 1;
}
def HWLGKM   : ProcResource<1> {
  let BufferSize = 1;
}
def HWSALU   : ProcResource<1> {
  let BufferSize = 1;
}
def HWVMEM   : ProcResource<1> {
  let BufferSize = 1;
}
def HWVALU   : ProcResource<1> {
  let BufferSize = 1;
}
def HWTransVALU : ProcResource<1> { // Transcendental VALU
  let BufferSize = 1;
}
def HWRC   : ProcResource<1> { // Register destination cache
  let BufferSize = 1;
}
def HWXDL   : ProcResource<1> { // MFMA CU
  let BufferSize = 0;
}

class HWWriteRes<SchedWrite write, list<ProcResourceKind> resources,
                 int latency> : WriteRes<write, resources> {
  let Latency = latency;
}

```
**EN:** This section contains concrete logic for TableGen record HWBranch. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `HWBranch`, `HWExport`, `HWLGKM`.
**CN:** 本节包含与 TableGen record HWBranch 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`HWBranch`, `HWExport`, `HWLGKM`。

### Lines 154-186: Defines TableGen class HWVALUWriteRes
```tablegen
class HWVALUWriteRes<SchedWrite write, int latency> :
  HWWriteRes<write, [HWVALU], latency>;

class UnsupportedWriteRes<SchedWrite write> : WriteRes<write, []> {
  let Unsupported = 1;
}

def PredMIReadVGPR : SchedPredicate<[{TII->hasVGPRUses(*MI)}]>;

def MIReadVGPR : SchedReadVariant<[
      SchedVar<PredMIReadVGPR, [MIVGPRRead]>,
      SchedVar<NoSchedPred, [ReadDefault]>]>;

// The latency numbers are taken from AMD Accelerated Parallel Processing
// guide. They may not be accurate.

// The latency values are 1 / (operations / cycle) / 4.
multiclass SICommonWriteRes {

  let RetireOOO = 1 in { // llvm-mca specific flag
  def : HWWriteRes<WriteBranch,  [HWBranch], 8>;
  def : HWWriteRes<WriteExport,  [HWExport], 4>;
  def : HWWriteRes<WriteLDS,     [HWLGKM],   5>; // Can be between 2 and 64
  def : HWWriteRes<WriteSALU,    [HWSALU],   1>;
  def : HWWriteRes<WriteSMEM,    [HWLGKM],   5>;
  def : HWWriteRes<WriteVMEM,    [HWVMEM],   80>;
  def : HWWriteRes<WriteBarrier, [HWBranch], 500>; // XXX: Guessed ???

  def : HWVALUWriteRes<Write32Bit,         1>;
  def : HWVALUWriteRes<WriteFloatCvt,      4>;
  def : HWVALUWriteRes<WriteTrans32,       4>;
  def : HWVALUWriteRes<WriteQuarterRate32, 4>;

```
**EN:** This section contains concrete logic for TableGen class HWVALUWriteRes. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `HWVALUWriteRes`, `UnsupportedWriteRes`, `PredMIReadVGPR`.
**CN:** 本节包含与 TableGen class HWVALUWriteRes 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`HWVALUWriteRes`, `UnsupportedWriteRes`, `PredMIReadVGPR`。

### Lines 187-213: Adjusts TableGen properties and predicates
```tablegen
  let ReleaseAtCycles = [4] in
  def : HWVALUWriteRes<Write4PassDGEMM,    4>;
  let ReleaseAtCycles = [8] in
  def : HWVALUWriteRes<Write8PassDGEMM,    8>;
  let ReleaseAtCycles = [16] in
  def : HWVALUWriteRes<Write16PassDGEMM,  16>;

  let ReleaseAtCycles = [2] in
  def : HWWriteRes<Write2PassMAI,  [HWXDL], 2>;
  let ReleaseAtCycles = [4] in
  def : HWWriteRes<Write4PassMAI,  [HWXDL], 4>;
  let ReleaseAtCycles = [8] in
  def : HWWriteRes<Write8PassMAI,  [HWXDL], 8>;
  let ReleaseAtCycles = [16] in
  def : HWWriteRes<Write16PassMAI, [HWXDL], 16>;

  let NumMicroOps = 0 in {
  def : HWWriteRes<WriteVALUDummy, [HWVALU],  1>;
  def : HWWriteRes<WriteSALUDummy, [HWSALU],  1>;
  }

  def : UnsupportedWriteRes<WriteSFPU>;
  def : UnsupportedWriteRes<WritePseudoScalarTrans>;
  } // End RetireOOO = 1

  def : ReadAdvance<MIVGPRRead, -2>;

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 214-240: Defines TableGen record PredIsVGPR32Copy
```tablegen
  // Technically mfma reads can be from 0 to 4 cycles but that does not make
  // sense to model because its register setup is huge. In particular if we
  // properly model read advance as -2 for a vgpr read it will result in a
  // bad scheduling of acc writes before that mfma. To avoid it we would
  // need to consume 2 or 4 more vgprs to be initialized before the acc
  // write sequence. Just assume worst case here.
  def : ReadAdvance<MIMFMARead, -4>;
}

def PredIsVGPR32Copy : SchedPredicate<[{TII->isVGPRCopy(*MI) && TII->getOpSize(*MI, 0) <= 32}]>;
def PredIsVGPR64Copy : SchedPredicate<[{TII->isVGPRCopy(*MI) && TII->getOpSize(*MI, 0) > 32}]>;
def WriteCopy : SchedWriteVariant<[
    SchedVar<PredIsVGPR32Copy, [Write32Bit]>,
    SchedVar<PredIsVGPR64Copy, [Write64Bit]>,
    SchedVar<NoSchedPred, [WriteSALU]>]>;

// Check if any matrix inputs are interpreted as f8 in an f8f6f4 mfma
// instruction.
def PredIsF8_MFMA_SCALE : SchedPredicate<[{
  TII->getNamedOperand(*MI, AMDGPU::OpName::cbsz)->getImm() <= AMDGPU::MFMAScaleFormats::FP8_E5M2 ||
  TII->getNamedOperand(*MI, AMDGPU::OpName::blgp)->getImm() <= AMDGPU::MFMAScaleFormats::FP8_E5M2
}]>;

let SchedModel = SIFullSpeedModel in {

defm : SICommonWriteRes;

```
**EN:** This section contains concrete logic for TableGen record PredIsVGPR32Copy. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `PredIsVGPR32Copy`, `PredIsVGPR64Copy`, `WriteCopy`.
**CN:** 本节包含与 TableGen record PredIsVGPR32Copy 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`PredIsVGPR32Copy`, `PredIsVGPR64Copy`, `WriteCopy`。

### Lines 241-274: Adjusts TableGen properties and predicates
```tablegen
let RetireOOO = 1 in { // llvm-mca specific flag
def : HWVALUWriteRes<Write64Bit,       2>;
def : HWVALUWriteRes<WriteIntMul,      4>;
def : HWVALUWriteRes<WriteFloatFMA,    1>;
def : HWVALUWriteRes<WriteDouble,      4>;
def : HWVALUWriteRes<WriteDoubleAdd,   2>;
def : HWVALUWriteRes<WriteDoubleCvt,   4>;
def : HWVALUWriteRes<WriteTrans64,     4>;
} // End RetireOOO = 1

def : InstRW<[WriteCopy], (instrs COPY)>;

} // End SchedModel = SIFullSpeedModel

let SchedModel = SIQuarterSpeedModel in {

defm : SICommonWriteRes;

let RetireOOO = 1 in { // llvm-mca specific flag
def : HWVALUWriteRes<Write64Bit,       2>;
def : HWVALUWriteRes<WriteIntMul,      4>;
def : HWVALUWriteRes<WriteFloatFMA,    16>;
def : HWVALUWriteRes<WriteDouble,      16>;
def : HWVALUWriteRes<WriteDoubleAdd,    8>;
def : HWVALUWriteRes<WriteDoubleCvt,    4>;
def : HWVALUWriteRes<WriteTrans64,     16>;
} // End RetireOOO = 1

def : InstRW<[WriteCopy], (instrs COPY)>;
def : InstRW<[Write64Bit, MIReadVGPR], (instregex "^V_ACCVGPR_WRITE_B32_e64$")>;
def : InstRW<[Write2PassMAI,  MIMFMARead], (instregex "^V_MFMA_..._4X4X")>;
def : InstRW<[Write8PassMAI,  MIMFMARead], (instregex "^V_MFMA_..._16X16X")>;
def : InstRW<[Write16PassMAI, MIMFMARead], (instregex "^V_MFMA_..._32X32X")>;

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 275-304: Adjusts TableGen properties and predicates
```tablegen
}  // End SchedModel = SIQuarterSpeedModel

let SchedModel = SIDPFullSpeedModel in {

defm : SICommonWriteRes;

let RetireOOO = 1 in { // llvm-mca specific flag
def : HWVALUWriteRes<WriteFloatFMA,    1>;
def : HWVALUWriteRes<WriteDouble,      1>;
def : HWVALUWriteRes<WriteDoubleAdd,   1>;
def : HWVALUWriteRes<WriteDoubleCvt,   1>;
def : HWVALUWriteRes<WriteTrans64,     4>;
def : HWVALUWriteRes<WriteIntMul,      1>;
def : HWVALUWriteRes<Write64Bit,       1>;
} // End RetireOOO = 1

def : InstRW<[WriteCopy], (instrs COPY)>;
def : InstRW<[Write64Bit], (instregex "^V_ACCVGPR_WRITE_B32_e64$")>;
def : InstRW<[Write2PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_4X4X")>;
def : InstRW<[Write8PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_16X16X")>;
def : InstRW<[Write16PassMAI,  MIMFMARead], (instregex "^V_MFMA_.32_32X32X")>;
def : InstRW<[Write4PassDGEMM, MIMFMARead], (instregex "^V_MFMA_.64_4X4X")>;
def : InstRW<[Write8PassDGEMM, MIMFMARead], (instregex "^V_MFMA_.64_16X16X")>;

} // End SchedModel = SIDPFullSpeedModel

let SchedModel = SIDPGFX942FullSpeedModel in {

defm : SICommonWriteRes;

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 305-338: Adjusts TableGen properties and predicates
```tablegen
def : HWVALUWriteRes<WriteFloatFMA,    1>;
def : HWVALUWriteRes<WriteDouble,      1>;
def : HWVALUWriteRes<WriteDoubleAdd,   1>;
def : HWVALUWriteRes<WriteDoubleCvt,   1>;
def : HWVALUWriteRes<WriteTrans64,     4>;
def : HWVALUWriteRes<WriteIntMul,      1>;
def : HWVALUWriteRes<Write64Bit,       1>;

def : InstRW<[WriteCopy], (instrs COPY)>;
def : InstRW<[Write64Bit], (instregex "^V_ACCVGPR_WRITE_B32_e64$")>;
def : InstRW<[Write2PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_4X4X")>;

def : InstRW<[Write4PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_16X16X8X")>;
def : InstRW<[Write4PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_16X16X16")>;
def : InstRW<[Write4PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_16X16X32")>;
def : InstRW<[Write8PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_16X16X[14][FBI]")>;

def : InstRW<[Write8PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_32X32X4XF")>;
def : InstRW<[Write8PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_32X32X8")>;
def : InstRW<[Write8PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_32X32X16")>;
def : InstRW<[Write16PassMAI,  MIMFMARead], (instregex "^V_MFMA_.32_32X32X[124][FBI]")>;

def : InstRW<[Write4PassDGEMM, MIMFMARead], (instregex "^V_MFMA_.64_4X4X")>;
def : InstRW<[Write8PassDGEMM, MIMFMARead], (instregex "^V_MFMA_.64_16X16X")>;

def : InstRW<[Write4PassMAI,   MIMFMARead], (instregex "^V_SMFMAC_.32_16X16X")>;
def : InstRW<[Write8PassMAI,   MIMFMARead], (instregex "^V_SMFMAC_.32_32X32X")>;

} // End SchedModel = SIDPGFX942FullSpeedModel


let SchedModel = SIDPGFX950FullSpeedModel in {
defm : SICommonWriteRes;

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 339-369: TableGen records and backend metadata
```tablegen
def : HWVALUWriteRes<WriteFloatFMA,    1>;
def : HWVALUWriteRes<WriteDouble,      1>;
def : HWVALUWriteRes<WriteDoubleAdd,   1>;
def : HWVALUWriteRes<WriteDoubleCvt,   1>;
def : HWVALUWriteRes<WriteTrans64,     4>;
def : HWVALUWriteRes<WriteIntMul,      1>;
def : HWVALUWriteRes<Write64Bit,       1>;

def : InstRW<[WriteCopy], (instrs COPY)>;
def : InstRW<[Write64Bit], (instregex "^V_ACCVGPR_WRITE_B32_e64$")>;
def : InstRW<[Write2PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_4X4X")>;

def : InstRW<[Write4PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_16X16X8X")>;
def : InstRW<[Write4PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_16X16X16")>;
def : InstRW<[Write4PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_16X16X32")>;
def : InstRW<[Write4PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_16X16X64")>;
def : InstRW<[Write8PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_16X16X[14][FBI]")>;

def : InstRW<[Write8PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_32X32X4XF")>;
def : InstRW<[Write8PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_32X32X8")>;
def : InstRW<[Write8PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_32X32X16")>;
def : InstRW<[Write8PassMAI,   MIMFMARead], (instregex "^V_MFMA_.32_32X32X32_")>;
def : InstRW<[Write16PassMAI,  MIMFMARead], (instregex "^V_MFMA_.32_32X32X[124][FBI]")>;

def : InstRW<[Write4PassDGEMM, MIMFMARead], (instregex "^V_MFMA_.64_4X4X")>;
def : InstRW<[Write16PassDGEMM, MIMFMARead], (instregex "^V_MFMA_.64_16X16X")>;

def : InstRW<[Write4PassMAI,   MIMFMARead], (instregex "^V_SMFMAC_.32_16X16X")>;
def : InstRW<[Write8PassMAI,   MIMFMARead], (instregex "^V_SMFMAC_.32_32X32X")>;


```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 370-403: Defines TableGen record WriteMFMAScale_16X16X128_F8F6F4
```tablegen
// If either matrix format is f8, the instruction takes 2x as many
// cycles. TODO: This isn't reflected in MCA.
def WriteMFMAScale_16X16X128_F8F6F4 : SchedWriteVariant<[
    SchedVar<PredIsF8_MFMA_SCALE, [Write8PassMAI]>,
    SchedVar<NoSchedPred, [Write4PassMAI]>]>;
def WriteMFMAScale_32X32X64_F8F6F4 : SchedWriteVariant<[
    SchedVar<PredIsF8_MFMA_SCALE, [Write16PassMAI]>,
    SchedVar<NoSchedPred, [Write8PassMAI]>]>;

def : InstRW<[WriteMFMAScale_16X16X128_F8F6F4, MIMFMARead],
       (instregex "^V_MFMA(_SCALE)?_.32_16X16X128_F8F6F4")>;
def : InstRW<[WriteMFMAScale_32X32X64_F8F6F4,  MIMFMARead],
        (instregex "^V_MFMA(_SCALE)?_.32_32X32X64_F8F6F4")>;

} // End SchedModel = SIDPGFX950FullSpeedModel


let SchedModel = GFX10SpeedModel in {

// The latency values are 1 / (operations / cycle).
// Add 1 stall cycle for VGPR read.
let RetireOOO = 1 in { // llvm-mca specific flag
def : HWWriteRes<Write32Bit,         [HWVALU, HWRC],   5>;
def : HWWriteRes<WriteFloatCvt,      [HWVALU, HWRC],   5>;
def : HWWriteRes<Write64Bit,         [HWVALU, HWRC],   6>;
def : HWWriteRes<WriteTrans32,       [HWTransVALU, HWRC], 10>;
def : HWWriteRes<WriteQuarterRate32, [HWVALU, HWRC],   8>;
def : HWWriteRes<WriteFloatFMA,      [HWVALU, HWRC],   5>;
def : HWWriteRes<WriteDouble,        [HWVALU, HWRC],   22>;
def : HWWriteRes<WriteDoubleAdd,     [HWVALU, HWRC],   22>;
def : HWWriteRes<WriteDoubleCvt,     [HWVALU, HWRC],   22>;
def : HWWriteRes<WriteIntMul,        [HWVALU, HWRC],   8>;
def : HWWriteRes<WriteTrans64,       [HWVALU, HWTransVALU, HWRC], 24>;

```
**EN:** This section contains concrete logic for TableGen record WriteMFMAScale_16X16X128_F8F6F4. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `WriteMFMAScale_16X16X128_F8F6F4`, `WriteMFMAScale_32X32X64_F8F6F4`.
**CN:** 本节包含与 TableGen record WriteMFMAScale_16X16X128_F8F6F4 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`WriteMFMAScale_16X16X128_F8F6F4`, `WriteMFMAScale_32X32X64_F8F6F4`。

### Lines 404-427: Adjusts TableGen properties and predicates
```tablegen
def : HWWriteRes<WriteBranch,        [HWBranch],       32>;
def : HWWriteRes<WriteExport,        [HWExport, HWRC], 16>;
def : HWWriteRes<WriteLDS,           [HWLGKM,   HWRC], 20>;
def : HWWriteRes<WriteSALU,          [HWSALU,   HWRC], 2>;
def : HWWriteRes<WriteSMEM,          [HWLGKM,   HWRC], 20>;
def : HWWriteRes<WriteVMEM,          [HWVMEM,   HWRC], 320>;
def : HWWriteRes<WriteBarrier,       [HWBranch],       2000>;

def : UnsupportedWriteRes<WriteSFPU>;
def : UnsupportedWriteRes<WritePseudoScalarTrans>;

let NumMicroOps = 0 in {
def : HWWriteRes<WriteVALUDummy, [HWVALU],  5>;
def : HWWriteRes<WriteSALUDummy, [HWSALU],  2>;
}

} // End RetireOOO = 1

def : InstRW<[WriteCopy], (instrs COPY)>;

}  // End SchedModel = GFX10SpeedModel

let SchedModel = GFX11SpeedModel in {

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 428-461: Adjusts TableGen properties and predicates
```tablegen
// The latency values are 1 / (operations / cycle).
// Add 1 stall cycle for VGPR read.
let RetireOOO = 1 in { // llvm-mca specific flag
def : HWWriteRes<Write32Bit,         [HWVALU, HWRC],   5>;
def : HWWriteRes<WriteFloatCvt,      [HWVALU, HWRC],   5>;
def : HWWriteRes<Write64Bit,         [HWVALU, HWRC],   6>;
def : HWWriteRes<WriteTrans32,       [HWTransVALU, HWRC], 10>;
def : HWWriteRes<WriteQuarterRate32, [HWVALU, HWRC],   8>;
def : HWWriteRes<WriteFloatFMA,      [HWVALU, HWRC],   5>;
def : HWWriteRes<WriteDouble,        [HWVALU, HWRC],   38>;
def : HWWriteRes<WriteDoubleAdd,     [HWVALU, HWRC],   38>;
def : HWWriteRes<WriteDoubleCvt,     [HWVALU, HWRC],   38>;
def : HWWriteRes<WriteIntMul,        [HWVALU, HWRC],   8>;
def : HWWriteRes<WriteTrans64,       [HWVALU, HWTransVALU, HWRC], 40>;

def : HWWriteRes<WriteBranch,        [HWBranch],       32>;
def : HWWriteRes<WriteExport,        [HWExport, HWRC], 16>;
def : HWWriteRes<WriteLDS,           [HWLGKM,   HWRC], 20>;
def : HWWriteRes<WriteSALU,          [HWSALU,   HWRC], 2>;
def : HWWriteRes<WriteSFPU,          [HWSALU,   HWRC], 4>;
def : HWWriteRes<WriteSMEM,          [HWLGKM,   HWRC], 20>;
def : HWWriteRes<WriteVMEM,          [HWVMEM,   HWRC], 320>;
def : HWWriteRes<WriteBarrier,       [HWBranch],       2000>;
} // End RetireOOO = 1

def : UnsupportedWriteRes<WritePseudoScalarTrans>;

def : InstRW<[WriteCopy], (instrs COPY)>;

let NumMicroOps = 0 in {
def : HWWriteRes<WriteVALUDummy, [HWVALU],  5>;
def : HWWriteRes<WriteSALUDummy, [HWSALU],  2>;
}

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 462-494: Adjusts TableGen properties and predicates
```tablegen
}  // End SchedModel = GFX11SpeedModel

let SchedModel = GFX12SpeedModel in {

def : HWWriteRes<Write32Bit,             [HWVALU, HWRC],   5>;
def : HWWriteRes<WriteFloatCvt,          [HWVALU, HWRC],   5>;
def : HWWriteRes<Write64Bit,             [HWVALU, HWRC],   6>;
def : HWWriteRes<WriteTrans32,           [HWVALU, HWRC],   9>;
def : HWWriteRes<WriteQuarterRate32,     [HWVALU, HWRC],   6>;
def : HWWriteRes<WriteFloatFMA,          [HWVALU, HWRC],   5>;
def : HWWriteRes<WriteDouble,            [HWVALU, HWRC],   38>;
def : HWWriteRes<WriteDoubleAdd,         [HWVALU, HWRC],   38>;
def : HWWriteRes<WriteDoubleCvt,         [HWVALU, HWRC],   38>;
def : HWWriteRes<WriteIntMul,            [HWVALU, HWRC],   8>;
def : HWWriteRes<WriteTrans64,           [HWVALU, HWRC],   40>;
def : HWWriteRes<WritePseudoScalarTrans, [HWVALU, HWRC],   8>;

def : HWWriteRes<WriteBranch,            [HWBranch],       32>;
def : HWWriteRes<WriteExport,            [HWExport, HWRC], 16>;
def : HWWriteRes<WriteLDS,               [HWLGKM,   HWRC], 20>;
def : HWWriteRes<WriteSALU,              [HWSALU,   HWRC], 2>;
def : HWWriteRes<WriteSFPU,              [HWSALU,   HWRC], 4>;
def : HWWriteRes<WriteSMEM,              [HWLGKM,   HWRC], 20>;
def : HWWriteRes<WriteVMEM,              [HWVMEM,   HWRC], 320>;
def : HWWriteRes<WriteBarrier,           [HWBranch],       2000>;

def : InstRW<[WriteCopy], (instrs COPY)>;

let NumMicroOps = 0 in {
def : HWWriteRes<WriteVALUDummy, [HWVALU],  5>;
def : HWWriteRes<WriteSALUDummy, [HWSALU],  2>;
}

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 495-528: Defines TableGen record PredIsF8_WMMA_SCALE
```tablegen
}  // End SchedModel = GFX12SpeedModel

// Check if any matrix inputs are interpreted as f8 in an f8f6f4
// wmma instruction.
def PredIsF8_WMMA_SCALE : SchedPredicate<[{
  TII->getNamedOperand(*MI, AMDGPU::OpName::matrix_a_fmt)->getImm() <= AMDGPU::WMMA::MATRIX_FMT_BF8 ||
  TII->getNamedOperand(*MI, AMDGPU::OpName::matrix_b_fmt)->getImm() <= AMDGPU::WMMA::MATRIX_FMT_BF8
}]>;

// If either matrix format is f8, the instruction takes 2x as many
// cycles. TODO: This isn't reflected in MCA.
def WriteWMMAScale_16X16X128_F8F6F4 : SchedWriteVariant<[
    SchedVar<PredIsF8_WMMA_SCALE, [WriteXDL4PassWMMA]>,
    SchedVar<NoSchedPred, [WriteXDL2PassWMMA]>
]>;

multiclass GFX125xCommonWriteRes {

let ReleaseAtCycles = [8] in
def : HWWriteRes<WriteXDL2PassWMMA, [HWXDL], 8>;
let ReleaseAtCycles = [16] in
def : HWWriteRes<WriteXDL4PassWMMA, [HWXDL], 16>;

def : HWWriteRes<Write4PassWMMA,  [HWVALU], 16>;
def : HWWriteRes<Write8PassWMMA,  [HWVALU], 32>;
def : HWWriteRes<Write16PassWMMA, [HWVALU], 64>;

def : HWWriteRes<Write32Bit,             [HWVALU, HWRC],   5>;
def : HWWriteRes<WriteFloatCvt,          [HWVALU, HWRC],   5>;
def : HWWriteRes<WriteTrans32,           [HWTransVALU, HWRC],   8>;
def : HWWriteRes<WriteQuarterRate32,     [HWVALU, HWRC],   6>;
def : HWWriteRes<WriteFloatFMA,          [HWVALU, HWRC],   5>;
def : HWWriteRes<WritePseudoScalarTrans, [HWVALU, HWRC],   8>;

```
**EN:** This section contains concrete logic for TableGen record PredIsF8_WMMA_SCALE. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `PredIsF8_WMMA_SCALE`, `WriteWMMAScale_16X16X128_F8F6F4`, `GFX125xCommonWriteRes`.
**CN:** 本节包含与 TableGen record PredIsF8_WMMA_SCALE 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`PredIsF8_WMMA_SCALE`, `WriteWMMAScale_16X16X128_F8F6F4`, `GFX125xCommonWriteRes`。

### Lines 529-562: Adjusts TableGen properties and predicates
```tablegen
def : HWWriteRes<WriteBranch,            [HWBranch],       32>;
def : HWWriteRes<WriteExport,            [HWExport, HWRC], 16>;
def : HWWriteRes<WriteLDS,               [HWLGKM,   HWRC], 20>;
def : HWWriteRes<WriteSALU,              [HWSALU,   HWRC], 2>;
def : HWWriteRes<WriteSFPU,              [HWSALU,   HWRC], 4>;
def : HWWriteRes<WriteSMEM,              [HWLGKM,   HWRC], 20>;
def : HWWriteRes<WriteVMEM,              [HWVMEM,   HWRC], 320>;
def : HWWriteRes<WriteBarrier,           [HWBranch],       2000>;

def : InstRW<[WriteCopy], (instrs COPY)>;

def : InstRW<[WriteXDL2PassWMMA], (instregex "^V_[S]*WMMA[C]*_.*_(FP8|BF8|BF16|F16)_w32")>;
def : InstRW<[WriteXDL4PassWMMA], (instregex "^V_[S]*WMMA[C]*_.*_(IU8|IU4)_w32")>;
def : InstRW<[WriteWMMAScale_16X16X128_F8F6F4], (instregex "^V_WMMA_.*_16X16X128_F8F6F4.*_w32")>;
def : InstRW<[Write4PassWMMA],    (instregex "^V_WMMA_F32_16X16X4_F32_w32")>;
def : InstRW<[WriteXDL2PassWMMA], (instregex "^V_WMMA.*_F32_32X16X128_F4")>;

let NumMicroOps = 0 in {
def : HWWriteRes<WriteVALUDummy, [HWVALU],  5>;
def : HWWriteRes<WriteSALUDummy, [HWSALU],  2>;
}

} // End GFX125xCommonWriteRes

let SchedModel = GFX1250SpeedModel in {
defm : GFX125xCommonWriteRes;

def : HWWriteRes<Write64Bit,             [HWVALU, HWRC],   6>;
def : HWWriteRes<WriteIntMul,            [HWVALU, HWRC],   8>;
def : HWWriteRes<WriteDouble,            [HWVALU, HWRC],   37>;
def : HWWriteRes<WriteDoubleAdd,         [HWVALU, HWRC],   37>;
def : HWWriteRes<WriteDoubleCvt,         [HWVALU, HWRC],   37>;
def : HWWriteRes<WriteTrans64,           [HWVALU, HWTransVALU, HWRC], 38>;
} // SchedModel = GFX1250SpeedModel
```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

## Key Concepts / 关键概念
- **Language / 语言**: TableGen DSL
- **Primary symbols / 主要符号**: `WriteBranch`, `WriteExport`, `WriteLDS`, `WriteSALU`, `WriteSMEM`, `WriteVMEM`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; scheduling / 调度; predicates and constraints / 谓词与约束
- **Build role / 构建角色**: Generates target records consumed by LLVM TableGen backends / 生成供 LLVM TableGen 后端消费的目标记录

## Dependencies / 依赖关系
- No direct include statements in this file / 此文件中没有直接的 include 语句
