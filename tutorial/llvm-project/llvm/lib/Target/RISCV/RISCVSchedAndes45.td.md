# RISCVSchedAndes45.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVSchedAndes45.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for processor scheduling resources, latencies, and pipeline behavior for RISC-V cores. / 使用 TableGen 定义RISC-V 处理器内核的调度资源、时延与流水线行为。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Commentary and design intent / 注释与设计意图
```tablegen
//==- RISCVSchedAndes45.td - Andes45 Scheduling Definitions --*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//

defvar Andes45VLEN = 512;
defvar Andes45DLEN = 512;
defvar Andes45VLEN_DLEN_RATIO = !div(Andes45VLEN, Andes45DLEN);

assert !or(!eq(Andes45VLEN_DLEN_RATIO, 1), !eq(Andes45VLEN_DLEN_RATIO, 2)),
       "Andes45VLEN / Andes45DLEN should be 1 or 2";

defvar Andes45BIU_DATA_WIDTH = 512;
defvar Andes45DLEN_BIU_DATA_WIDTH_RATIO = !div(Andes45DLEN, Andes45BIU_DATA_WIDTH);
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 21-44: TableGen class Andes45IsWorstCaseMX<string / TableGen 类 Andes45IsWorstCaseMX<string
```tablegen
assert !or(!eq(Andes45DLEN_BIU_DATA_WIDTH_RATIO, 1), !eq(Andes45DLEN_BIU_DATA_WIDTH_RATIO, 2)),
       "Andes45DLEN / Andes45DLEN_BIU_DATA_WIDTH_RATIO should be 1 or 2";

// HVM region: VLSU_MEM_DW equals DLEN
// Cachable/Non-cachable region: VLSU_MEM_DW equals BIU_DATA_WIDTH
defvar Andes45VLSU_MEM_DW = Andes45BIU_DATA_WIDTH;
defvar Andes45VLEN_VLSU_MEM_DW_RATIO = !div(Andes45VLEN, Andes45VLSU_MEM_DW);

// There are various latency depending on its memory type and status.
defvar VLSU_MEM_LATENCY = 13;

// The worst case LMUL is the largest LMUL.
class Andes45IsWorstCaseMX<string mx, list<string> MxList> {
  defvar LLMUL = LargestLMUL<MxList>.r;
  bit c = !eq(mx, LLMUL);
}

// The worst case is the largest LMUL with the smallest SEW.
class Andes45IsWorstCaseMXSEW<string mx, int sew, list<string> MxList,
                              bit isF = 0> {
  defvar LLMUL = LargestLMUL<MxList>.r;
  defvar SSEW = SmallestSEW<mx, isF>.r;
  bit c = !and(!eq(mx, LLMUL), !eq(sew, SSEW));
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 45-70: TableGen class Andes45GetCyclesLoadStore<string / TableGen 类 Andes45GetCyclesLoadStore<string
```tablegen

// (VLEN/VLSU_MEM_DW)*EMUL
class Andes45GetCyclesLoadStore<string mx> {
  int c = !cond(
    !eq(mx, "M1") : !mul(Andes45VLEN_VLSU_MEM_DW_RATIO, 1),
    !eq(mx, "M2") : !mul(Andes45VLEN_VLSU_MEM_DW_RATIO, 2),
    !eq(mx, "M4") : !mul(Andes45VLEN_VLSU_MEM_DW_RATIO, 4),
    !eq(mx, "M8") : !mul(Andes45VLEN_VLSU_MEM_DW_RATIO, 8),
    !eq(mx, "MF2") : !mul(Andes45VLEN_VLSU_MEM_DW_RATIO, 1),
    !eq(mx, "MF4") : !mul(Andes45VLEN_VLSU_MEM_DW_RATIO, 1),
    !eq(mx, "MF8") : !mul(Andes45VLEN_VLSU_MEM_DW_RATIO, 1)
  );
}

class Andes45GetCyclesOnePerElement<string mx, int sew> {
  defvar VL = !div(Andes45VLEN, sew);
  int c = !cond(
    !eq(mx, "M1")  : VL,
    !eq(mx, "M2")  : !mul(VL, 2),
    !eq(mx, "M4")  : !mul(VL, 4),
    !eq(mx, "M8")  : !mul(VL, 8),
    !eq(mx, "MF2") : !div(VL, 2),
    !eq(mx, "MF4") : !div(VL, 4),
    !eq(mx, "MF8") : !div(VL, 8)
  );
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 71-97: TableGen class Andes45GetLMULValue<string / TableGen 类 Andes45GetLMULValue<string
```tablegen

// When fractional LMUL is used, the LMUL used in calculation is 1.
class Andes45GetLMULValue<string mx> {
  int c = !cond(
    !eq(mx, "M1") : 1,
    !eq(mx, "M2") : 2,
    !eq(mx, "M4") : 4,
    !eq(mx, "M8") : 8,
    !eq(mx, "MF2") : 1,
    !eq(mx, "MF4") : 1,
    !eq(mx, "MF8") : 1
  );
}

// (VLEN/DLEN)*LMUL
// When fractional LMUL is used, the LMUL used in calculation is 1.
class Andes45GetCyclesDefault<string mx> {
  int c = !cond(
    !eq(mx, "M1") : !mul(Andes45VLEN_DLEN_RATIO, 1),
    !eq(mx, "M2") : !mul(Andes45VLEN_DLEN_RATIO, 2),
    !eq(mx, "M4") : !mul(Andes45VLEN_DLEN_RATIO, 4),
    !eq(mx, "M8") : !mul(Andes45VLEN_DLEN_RATIO, 8),
    !eq(mx, "MF2") : !mul(Andes45VLEN_DLEN_RATIO, 1),
    !eq(mx, "MF4") : !mul(Andes45VLEN_DLEN_RATIO, 1),
    !eq(mx, "MF8") : !mul(Andes45VLEN_DLEN_RATIO, 1)
  );
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 98-115: TableGen class Andes45GetCyclesWidening<string / TableGen 类 Andes45GetCyclesWidening<string
```tablegen

// (VLEN/DLEN)*LMUL*2, if LMUL >= 1,
// (VLEN != DLEN) ? 4 : 1, if LMUL < 1.
class Andes45GetCyclesWidening<string mx> {
  int c = !cond(
    !eq(mx, "M1") : !mul(Andes45VLEN_DLEN_RATIO, 2),
    !eq(mx, "M2") : !mul(Andes45VLEN_DLEN_RATIO, 4),
    !eq(mx, "M4") : !mul(Andes45VLEN_DLEN_RATIO, 8),
    // FIXME: .v* and .w* are different if LMUL < 1.
    !eq(mx, "MF2") : !if(!ne(Andes45VLEN, Andes45DLEN), 4, 1),
    !eq(mx, "MF4") : !if(!ne(Andes45VLEN, Andes45DLEN), 4, 1),
    !eq(mx, "MF8") : !if(!ne(Andes45VLEN, Andes45DLEN), 4, 1),
  );
}

class Andes45GetLatencyDiv<int sew> {
  int c = !add(sew, 4);
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 116-133: TableGen class Andes45GetCyclesDiv<string / TableGen 类 Andes45GetCyclesDiv<string
```tablegen

// (VLEN/DLEN)*LMUL*SEW+(VLEN/DLEN)*LMUL*2+1
// = (VLEN/DLEN)*LMUL*(SEW+2)+1
class Andes45GetCyclesDiv<string mx, int sew> {
  defvar a = !mul(Andes45VLEN_DLEN_RATIO, !add(sew, 2));
  int b = !cond(
    !eq(mx, "M1")  : !mul(a, 1),
    !eq(mx, "M2")  : !mul(a, 2),
    !eq(mx, "M4")  : !mul(a, 4),
    !eq(mx, "M8")  : !mul(a, 8),
    !eq(mx, "MF2") : !mul(a, 1),
    !eq(mx, "MF4") : !mul(a, 1),
    !eq(mx, "MF8") : !mul(a, 1)
  );

  int c = !add(b, 1);
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 134-158: TableGen class Andes45GetCyclesNarrowing<string / TableGen 类 Andes45GetCyclesNarrowing<string
```tablegen
// (VLEN/DLEN)*LMUL*2, if LMUL >= 1,
// (VLEN != DLEN) ? : 4 : 1, if LMUL < 1.
class Andes45GetCyclesNarrowing<string mx> {
  int c = !cond(
    !eq(mx, "M1") : !mul(Andes45VLEN_DLEN_RATIO, 2),
    !eq(mx, "M2") : !mul(Andes45VLEN_DLEN_RATIO, 4),
    !eq(mx, "M4") : !mul(Andes45VLEN_DLEN_RATIO, 8),
    !eq(mx, "MF2") : !if(!ne(Andes45VLEN, Andes45DLEN), 4, 1),
    !eq(mx, "MF4") : !if(!ne(Andes45VLEN, Andes45DLEN), 4, 1),
    !eq(mx, "MF8") : !if(!ne(Andes45VLEN, Andes45DLEN), 4, 1)
  );
}

// 3, if LMUL >= 1,
// (VLEN != DLEN) ? 3 : 2, if LMUL <1.
class Andes45GetLatencyNarrowing<string mx> {
  int c = !cond(
    !eq(mx, "M1") : 3,
    !eq(mx, "M2") : 3,
    !eq(mx, "M4") : 3,
    !eq(mx, "MF2") : !if(!ne(Andes45VLEN, Andes45DLEN), 3, 2),
    !eq(mx, "MF4") : !if(!ne(Andes45VLEN, Andes45DLEN), 3, 2),
    !eq(mx, "MF8") : !if(!ne(Andes45VLEN, Andes45DLEN), 3, 2)
  );
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 159-182: TableGen class Andes45GetFDivFactor<int / TableGen 类 Andes45GetFDivFactor<int
```tablegen

class Andes45GetFDivFactor<int sew> {
  int c = !cond(
    !eq(sew, 16) : 22,
    !eq(sew, 32) : 36,
    !eq(sew, 64) : 64
  );
}

class Andes45GetFSqrtFactor<int sew> {
  int c = !cond(
    !eq(sew, 16) : 20,
    !eq(sew, 32) : 34,
    !eq(sew, 64) : 62
  );
}

// (VLEN/DLEN)*LMUL+LOG2(DLEN/64)*2+LOG2(64/SEW)
class Andes45GetReductionCycles<string mx, int sew> {
  defvar d = Andes45GetCyclesDefault<mx>.c;
  int c = !add(d,
               !add(!mul(!logtwo(!div(Andes45DLEN, 64)), 2),
                    !logtwo(!div(64, sew))));
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 183-209: TableGen class Andes45GetReductionCyclesWidening<string / TableGen 类 Andes45GetReductionCyclesWidening<string
```tablegen

// (VLEN/DLEN)*LMUL*2+LOG2(DLEN/64)*2+LOG2(64/2/SEW)
class Andes45GetReductionCyclesWidening<string mx, int sew> {
  defvar w = !mul(Andes45GetCyclesDefault<mx>.c, 2);
  int c = !add(w,
               !add(!mul(!logtwo(!div(Andes45DLEN, 64)), 2),
                    !logtwo(!div(32, sew))));
}

// (VLEN/DLEN)*LMUL+LOG2(DLEN/SEW)
class Andes45GetFReductionCycles<string mx, int sew> {
  defvar d = Andes45GetCyclesDefault<mx>.c;
  int c = !add(d, !logtwo(!div(Andes45DLEN, sew)));
}

// (VLEN*LMUL)/SEW
class Andes45GetOrderedFReductionCycles<string mx, int sew> {
  defvar b = !cond(
    !eq(mx, "M1")  : !mul(Andes45VLEN, 1),
    !eq(mx, "M2")  : !mul(Andes45VLEN, 2),
    !eq(mx, "M4")  : !mul(Andes45VLEN, 4),
    !eq(mx, "M8")  : !mul(Andes45VLEN, 8),
    !eq(mx, "MF2") : !mul(Andes45VLEN, 1),
    !eq(mx, "MF4") : !mul(Andes45VLEN, 1),
    !eq(mx, "MF8") : !mul(Andes45VLEN, 1)
  );
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 210-232: TableGen class Andes45GetFWReductionCycles<string / TableGen 类 Andes45GetFWReductionCycles<string
```tablegen
  int c = !div(b, sew);
}

// (VLEN/DLEN)*LMUL*2+LOG2(DLEN/SEW)-1
class Andes45GetFWReductionCycles<string mx, int sew> {
  defvar a = !mul(Andes45GetCyclesDefault<mx>.c, 2);
  defvar b = !add(a, !logtwo(!div(Andes45DLEN, sew)));
  int c = !sub(b, 1);
}

// (VLEN*LMUL)/SEW
class Andes45GetOrderedFWReductionCycles<string mx, int sew> {
  defvar b = !mul(Andes45GetCyclesDefault<mx>.c, Andes45DLEN);
  int c = !div(b, sew);
}

def Andes45Model : SchedMachineModel {
  let MicroOpBufferSize = 0;  // Andes45 is in-order processor
  let IssueWidth = 2;         // 2 micro-ops dispatched per cycle
  let LoadLatency = 2;
  let MispredictPenalty = 5;
  let CompleteModel = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 233-258: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen

let SchedModel = Andes45Model in {

//===----------------------------------------------------------------------===//
// Define each kind of processor resource and number available.

//===----------------------------------------------------------------------===//
// Andes 45 series CPU
//   - 2 Integer Arithmetic and Logical Units (ALU)
//   - Multiply / Divide Unit (MDU)
//   - Load Store Unit (LSU)
//   - Control and Status Register Unit (CSR)
//   - Floating Point Multiply-Accumulate Unit (FMAC)
//   - Floating Point Divide / SQRT Unit (FDIV)
//   - Floating Point Move Unit (FMV)
//   - Floating Point Misc Unit (FMISC)
//
// Andes 45 series VPU
//   - Vector Arithmetic and Logical Unit (VALU)
//   - Vector Multiply Accumulate Unit (VMAC)
//   - Vector Divide Unit (VDIV)
//   - Vector Permutation Unit (VPERMUT)
//   - Vector Mask Unit (VMASK)
//   - Vector Floating-Point Miscellaneous Unit (VFMIS)
//   - Vector Floating-Point Divide Unit (VFDIV)
//===----------------------------------------------------------------------===//
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 259-279: TableGen record Andes45ALU / TableGen 记录 Andes45ALU
```tablegen

let BufferSize = 0 in {
def Andes45ALU : ProcResource<2>;
def Andes45MDU : ProcResource<1>;
def Andes45LSU : ProcResource<1>;
def Andes45CSR : ProcResource<1>;

def Andes45FMAC  : ProcResource<1>;
def Andes45FDIV  : ProcResource<1>;
def Andes45FMV   : ProcResource<1>;
def Andes45FMISC : ProcResource<1>;

def Andes45VALU    : ProcResource<1>;
def Andes45VMAC    : ProcResource<1>;
def Andes45VFMIS   : ProcResource<1>;
def Andes45VPERMUT : ProcResource<1>;
def Andes45VDIV    : ProcResource<1>;
def Andes45VFDIV   : ProcResource<1>;
def Andes45VMASK   : ProcResource<1>;
def Andes45VLSU    : ProcResource<1>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 280-299: TableGen record definition / TableGen 记录定义
```tablegen

// Integer arithmetic and logic
def : WriteRes<WriteIALU, [Andes45ALU]>;
def : WriteRes<WriteIALU32, [Andes45ALU]>;
def : WriteRes<WriteShiftImm, [Andes45ALU]>;
def : WriteRes<WriteShiftImm32, [Andes45ALU]>;
def : WriteRes<WriteShiftReg, [Andes45ALU]>;
def : WriteRes<WriteShiftReg32, [Andes45ALU]>;

// Short forward branch
def : WriteRes<WriteSFB, [Andes45ALU]> {
  let Latency = 1;
  let NumMicroOps = 2;
}

// Branching
def : WriteRes<WriteJmp, [Andes45ALU]>;
def : WriteRes<WriteJal, [Andes45ALU]>;
def : WriteRes<WriteJalr, [Andes45ALU]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 300-317: TableGen record definition / TableGen 记录定义
```tablegen
// Integer multiplication
let Latency = 3 in {
def : WriteRes<WriteIMul, [Andes45MDU]>;
def : WriteRes<WriteIMul32, [Andes45MDU]>;
}

// Integer division
let Latency = 39, ReleaseAtCycles = [39] in {
def : WriteRes<WriteIDiv, [Andes45MDU]>;
def : WriteRes<WriteIDiv32, [Andes45MDU]>;
}

// Integer remainder
let Latency = 39, ReleaseAtCycles = [39] in {
def : WriteRes<WriteIRem, [Andes45MDU]>;
def : WriteRes<WriteIRem32, [Andes45MDU]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 318-340: TableGen record definition / TableGen 记录定义
```tablegen
// Memory
let Latency = 5 in {
def : WriteRes<WriteLDB, [Andes45LSU]>;
def : WriteRes<WriteLDH, [Andes45LSU]>;
def : WriteRes<WriteFLD16, [Andes45LSU]>;
}

let Latency = 3 in {
def : WriteRes<WriteLDW, [Andes45LSU]>;
def : WriteRes<WriteLDD, [Andes45LSU]>;
def : WriteRes<WriteFLD32, [Andes45LSU]>;
def : WriteRes<WriteFLD64, [Andes45LSU]>;
}

let Latency = 1 in {
def : WriteRes<WriteSTB, [Andes45LSU]>;
def : WriteRes<WriteSTH, [Andes45LSU]>;
def : WriteRes<WriteSTW, [Andes45LSU]>;
def : WriteRes<WriteSTD, [Andes45LSU]>;
def : WriteRes<WriteFST16, [Andes45LSU]>;
def : WriteRes<WriteFST32, [Andes45LSU]>;
def : WriteRes<WriteFST64, [Andes45LSU]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 341-366: TableGen record definition / TableGen 记录定义
```tablegen

// Atomic Memory
let Latency = 9 in {
def : WriteRes<WriteAtomicW, [Andes45LSU]>;
def : WriteRes<WriteAtomicD, [Andes45LSU]>;
def : WriteRes<WriteAtomicLDW, [Andes45LSU]>;
def : WriteRes<WriteAtomicLDD, [Andes45LSU]>;
}

let Latency = 3 in {
def : WriteRes<WriteAtomicSTW, [Andes45LSU]>;
def : WriteRes<WriteAtomicSTD, [Andes45LSU]>;
}

// FMAC
let Latency = 4 in  {
def : WriteRes<WriteFAdd16, [Andes45FMAC]>;
def : WriteRes<WriteFAdd32, [Andes45FMAC]>;
def : WriteRes<WriteFAdd64, [Andes45FMAC]>;
def : WriteRes<WriteFMul16, [Andes45FMAC]>;
def : WriteRes<WriteFMul32, [Andes45FMAC]>;
def : WriteRes<WriteFMul64, [Andes45FMAC]>;
def : WriteRes<WriteFMA16, [Andes45FMAC]>;
def : WriteRes<WriteFMA32, [Andes45FMAC]>;
def : WriteRes<WriteFMA64, [Andes45FMAC]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 367-394: TableGen record definition / TableGen 记录定义
```tablegen

// FDIV
let Latency = 12, ReleaseAtCycles = [12] in
def : WriteRes<WriteFDiv16, [Andes45FDIV]>;
let Latency = 11, ReleaseAtCycles = [11] in
def : WriteRes<WriteFSqrt16, [Andes45FDIV]>;

let Latency = 19, ReleaseAtCycles = [19] in
def : WriteRes<WriteFDiv32, [Andes45FDIV]>;
let Latency = 18, ReleaseAtCycles = [18] in
def : WriteRes<WriteFSqrt32, [Andes45FDIV]>;

let Latency = 33, ReleaseAtCycles = [33] in
def : WriteRes<WriteFDiv64, [Andes45FDIV]>;
let Latency = 32, ReleaseAtCycles = [32] in
def : WriteRes<WriteFSqrt64, [Andes45FDIV]>;

// FMV
def : WriteRes<WriteFSGNJ16, [Andes45FMV]>;
def : WriteRes<WriteFSGNJ32, [Andes45FMV]>;
def : WriteRes<WriteFSGNJ64, [Andes45FMV]>;
def : WriteRes<WriteFMovF16ToI16, [Andes45FMV]>;
def : WriteRes<WriteFMovI16ToF16, [Andes45FMV]>;
def : WriteRes<WriteFMovF32ToI32, [Andes45FMV]>;
def : WriteRes<WriteFMovI32ToF32, [Andes45FMV]>;
def : WriteRes<WriteFMovF64ToI64, [Andes45FMV]>;
def : WriteRes<WriteFMovI64ToF64, [Andes45FMV]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 395-424: TableGen record definition / TableGen 记录定义
```tablegen
// FMISC
let Latency = 2 in  {
def : WriteRes<WriteFMinMax16, [Andes45FMISC]>;
def : WriteRes<WriteFMinMax32, [Andes45FMISC]>;
def : WriteRes<WriteFMinMax64, [Andes45FMISC]>;
def : WriteRes<WriteFClass16, [Andes45FMISC]>;
def : WriteRes<WriteFClass32, [Andes45FMISC]>;
def : WriteRes<WriteFClass64, [Andes45FMISC]>;
def : WriteRes<WriteFCmp16, [Andes45FMISC]>;
def : WriteRes<WriteFCmp32, [Andes45FMISC]>;
def : WriteRes<WriteFCmp64, [Andes45FMISC]>;
def : WriteRes<WriteFCvtF16ToI32, [Andes45FMISC]>;
def : WriteRes<WriteFCvtF16ToI64, [Andes45FMISC]>;
def : WriteRes<WriteFCvtF32ToI32, [Andes45FMISC]>;
def : WriteRes<WriteFCvtF32ToI64, [Andes45FMISC]>;
def : WriteRes<WriteFCvtF64ToI32, [Andes45FMISC]>;
def : WriteRes<WriteFCvtF64ToI64, [Andes45FMISC]>;
def : WriteRes<WriteFCvtI32ToF16, [Andes45FMISC]>;
def : WriteRes<WriteFCvtI32ToF32, [Andes45FMISC]>;
def : WriteRes<WriteFCvtI32ToF64, [Andes45FMISC]>;
def : WriteRes<WriteFCvtI64ToF16, [Andes45FMISC]>;
def : WriteRes<WriteFCvtI64ToF32, [Andes45FMISC]>;
def : WriteRes<WriteFCvtI64ToF64, [Andes45FMISC]>;
def : WriteRes<WriteFCvtF16ToF32, [Andes45FMISC]>;
def : WriteRes<WriteFCvtF16ToF64, [Andes45FMISC]>;
def : WriteRes<WriteFCvtF32ToF16, [Andes45FMISC]>;
def : WriteRes<WriteFCvtF32ToF64, [Andes45FMISC]>;
def : WriteRes<WriteFCvtF64ToF16, [Andes45FMISC]>;
def : WriteRes<WriteFCvtF64ToF32, [Andes45FMISC]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 425-447: TableGen record definition / TableGen 记录定义
```tablegen

// Bitmanip
// Zba extension
def : WriteRes<WriteSHXADD, [Andes45ALU]>;
def : WriteRes<WriteSHXADD32, [Andes45ALU]>;

// Zbb extension
def : WriteRes<WriteRotateImm, [Andes45ALU]>;
def : WriteRes<WriteRotateImm32, [Andes45ALU]>;
def : WriteRes<WriteRotateReg, [Andes45ALU]>;
def : WriteRes<WriteRotateReg32, [Andes45ALU]>;
def : WriteRes<WriteREV8, [Andes45ALU]>;
def : WriteRes<WriteORCB, [Andes45ALU]>;
def : WriteRes<WriteIMinMax, [Andes45ALU]>;

let Latency = 3 in  {
def : WriteRes<WriteCLZ, [Andes45ALU]>;
def : WriteRes<WriteCLZ32, [Andes45ALU]>;
def : WriteRes<WriteCTZ, [Andes45ALU]>;
def : WriteRes<WriteCTZ32, [Andes45ALU]>;
def : WriteRes<WriteCPOP, [Andes45ALU]>;
def : WriteRes<WriteCPOP32, [Andes45ALU]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 448-483: TableGen record definition / TableGen 记录定义
```tablegen

// Zbc extension
let Latency = 3 in
def : WriteRes<WriteCLMUL, [Andes45ALU]>;

// Zbs extension
def : WriteRes<WriteSingleBit, [Andes45ALU]>;
def : WriteRes<WriteSingleBitImm, [Andes45ALU]>;
def : WriteRes<WriteBEXT, [Andes45ALU]>;
def : WriteRes<WriteBEXTI, [Andes45ALU]>;

// Others
def : WriteRes<WriteCSR, [Andes45CSR]>;
def : WriteRes<WriteNop, []>;

//===----------------------------------------------------------------------===//

// Bypass and advance
def : ReadAdvance<ReadIALU, 0>;
def : ReadAdvance<ReadIALU32, 0>;
def : ReadAdvance<ReadShiftImm, 0>;
def : ReadAdvance<ReadShiftImm32, 0>;
def : ReadAdvance<ReadShiftReg, 0>;
def : ReadAdvance<ReadShiftReg32, 0>;
def : ReadAdvance<ReadSFBJmp, 0>;
def : ReadAdvance<ReadSFBALU, 0>;
def : ReadAdvance<ReadJalr, 0>;
def : ReadAdvance<ReadJmp, 0>;
def : ReadAdvance<ReadIMul, 0>;
def : ReadAdvance<ReadIMul32, 0>;
def : ReadAdvance<ReadIDiv, 0>;
def : ReadAdvance<ReadIDiv32, 0>;
def : ReadAdvance<ReadIRem, 0>;
def : ReadAdvance<ReadIRem32, 0>;
def : ReadAdvance<ReadStoreData, 0>;
def : ReadAdvance<ReadMemBase, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 484-519: TableGen record definition / TableGen 记录定义
```tablegen
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
def : ReadAdvance<ReadFMul16, 0>;
def : ReadAdvance<ReadFMul32, 0>;
def : ReadAdvance<ReadFMul64, 0>;
def : ReadAdvance<ReadFMA16, 0>;
def : ReadAdvance<ReadFMA32, 0>;
def : ReadAdvance<ReadFMA64, 0>;
def : ReadAdvance<ReadFMA16Addend, 0>;
def : ReadAdvance<ReadFMA32Addend, 0>;
def : ReadAdvance<ReadFMA64Addend, 0>;
def : ReadAdvance<ReadFDiv16, 0>;
def : ReadAdvance<ReadFDiv32, 0>;
def : ReadAdvance<ReadFDiv64, 0>;
def : ReadAdvance<ReadFSqrt16, 0>;
def : ReadAdvance<ReadFSqrt32, 0>;
def : ReadAdvance<ReadFSqrt64, 0>;
def : ReadAdvance<ReadFSGNJ16, 0>;
def : ReadAdvance<ReadFSGNJ32, 0>;
def : ReadAdvance<ReadFSGNJ64, 0>;
def : ReadAdvance<ReadFMovF16ToI16, 0>;
def : ReadAdvance<ReadFMovI16ToF16, 0>;
def : ReadAdvance<ReadFMovF32ToI32, 0>;
def : ReadAdvance<ReadFMovI32ToF32, 0>;
def : ReadAdvance<ReadFMovF64ToI64, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 520-555: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFMovI64ToF64, 0>;
def : ReadAdvance<ReadFMinMax16, 0>;
def : ReadAdvance<ReadFMinMax32, 0>;
def : ReadAdvance<ReadFMinMax64, 0>;
def : ReadAdvance<ReadFClass16, 0>;
def : ReadAdvance<ReadFClass32, 0>;
def : ReadAdvance<ReadFClass64, 0>;
def : ReadAdvance<ReadFCmp16, 0>;
def : ReadAdvance<ReadFCmp32, 0>;
def : ReadAdvance<ReadFCmp64, 0>;
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
def : ReadAdvance<ReadFCvtF16ToF32, 0>;
def : ReadAdvance<ReadFCvtF16ToF64, 0>;
def : ReadAdvance<ReadFCvtF32ToF16, 0>;
def : ReadAdvance<ReadFCvtF32ToF64, 0>;
def : ReadAdvance<ReadFCvtF64ToF16, 0>;
def : ReadAdvance<ReadFCvtF64ToF32, 0>;
def : ReadAdvance<ReadSHXADD, 0>;
def : ReadAdvance<ReadSHXADD32, 0>;
def : ReadAdvance<ReadRotateImm, 1>;
def : ReadAdvance<ReadRotateImm32, 1>;
def : ReadAdvance<ReadRotateReg, 1>;
def : ReadAdvance<ReadRotateReg32, 1>;
def : ReadAdvance<ReadCLZ, 0>;
def : ReadAdvance<ReadCLZ32, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 556-574: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadCTZ, 0>;
def : ReadAdvance<ReadCTZ32, 0>;
def : ReadAdvance<ReadCPOP, 0>;
def : ReadAdvance<ReadCPOP32, 0>;
def : ReadAdvance<ReadREV8, 0>;
def : ReadAdvance<ReadORCB, 0>;
def : ReadAdvance<ReadIMinMax, 0>;
def : ReadAdvance<ReadCLMUL, 0>;
def : ReadAdvance<ReadSingleBit, 0>;
def : ReadAdvance<ReadSingleBitImm, 0>;
def : ReadAdvance<ReadCSR, 0>;

// RVV Scheduling

// 6. Configuration-Setting Instructions
def : WriteRes<WriteVSETVLI, [Andes45CSR]>;
def : WriteRes<WriteVSETIVLI, [Andes45CSR]>;
def : WriteRes<WriteVSETVL, [Andes45CSR]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 575-597: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
// 7. Vector Loads and Stores

// Unit-stride loads and stores

// The latency for loads is (4+VLSU_MEM_LATENCY).
// The throughput for loads and stores is (VLEN/VLSU_MEM_DW)*EMUL.
foreach mx = SchedMxList in {
  defvar Cycles = Andes45GetCyclesLoadStore<mx>.c;
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxList>.c;

  let Latency = !add(4, VLSU_MEM_LATENCY), ReleaseAtCycles = [Cycles] in {
    defm "" : LMULWriteResMX<"WriteVLDE", [Andes45VLSU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVLDFF", [Andes45VLSU], mx, IsWorstCase>;
  }
  let ReleaseAtCycles = [Cycles] in
  defm "" : LMULWriteResMX<"WriteVSTE", [Andes45VLSU], mx, IsWorstCase>;

  // Mask loads and stores
  let Latency = !add(4, VLSU_MEM_LATENCY), ReleaseAtCycles = [Cycles] in
  defm "" : LMULWriteResMX<"WriteVLDM", [Andes45VLSU], mx, IsWorstCase=!eq(mx, "M1")>;
  let ReleaseAtCycles = [Cycles] in
  defm "" : LMULWriteResMX<"WriteVSTM", [Andes45VLSU], mx, IsWorstCase=!eq(mx, "M1")>;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 598-617: Commentary and design intent / 注释与设计意图
```tablegen

// Strided loads and stores.

// Strided loads and stores operate at one element per cycles.
// We uses the SEW to compute the number of elements for throughput.
// The latency for loads is (4+VLSU_MEM_LATENCY+(DLEN/EEW)).
// The throughput for loads and stores is VL.
foreach mx = SchedMxList in {
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxList>.c;

  foreach eew = [8, 16, 32, 64] in {
    defvar Cycles = Andes45GetCyclesOnePerElement<mx, eew>.c;

    let Latency = !add(4, !add(VLSU_MEM_LATENCY, !div(Andes45DLEN, eew))),
        ReleaseAtCycles = [Cycles] in
      defm "" : LMULWriteResMX<"WriteVLDS"  # eew, [Andes45VLSU], mx, IsWorstCase>;

    let ReleaseAtCycles = [Cycles] in
      defm "" : LMULWriteResMX<"WriteVSTS"  # eew, [Andes45VLSU], mx, IsWorstCase>;
  }
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 618-636: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
}

// Indexed loads and stores

// Indexed loads and stores operate at one element per cycles.
// We uses the SEW to compute the number of elements for throughput.
// The latency for loads is (5+VLSU_MEM_LATENCY+(DLEN/EEW)).
// The throughput for loads and stores is (VL+EMUL-1).
foreach mx = SchedMxList in {
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxList>.c;

  foreach eew = [8, 16, 32, 64] in {
    defvar Cycles = Andes45GetCyclesOnePerElement<mx, eew>.c;

    let Latency = !add(5, !add(VLSU_MEM_LATENCY, !div(Andes45DLEN, eew))),
        ReleaseAtCycles = [!add(Cycles, !sub(Andes45GetLMULValue<mx>.c, 1))] in {
      defm "" : LMULWriteResMX<"WriteVLDUX" # eew, [Andes45VLSU], mx, IsWorstCase>;
      defm "" : LMULWriteResMX<"WriteVLDOX" # eew, [Andes45VLSU], mx, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 637-655: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen

    let ReleaseAtCycles = [!add(Cycles, !sub(Andes45GetLMULValue<mx>.c, 1))] in {
      defm "" : LMULWriteResMX<"WriteVSTUX" # eew, [Andes45VLSU], mx, IsWorstCase>;
      defm "" : LMULWriteResMX<"WriteVSTOX" # eew, [Andes45VLSU], mx, IsWorstCase>;
    }
  }
}

// Unit-Stride Segmented Loads and Stores

// The latency for loads is (4+VLSU_MEM_LATENCY+EMUL*NFIELDS+2)
// The throughput for loads and stores is (VLEN/VLSU_MEM_DW)*EMUL*NFIELDS.
foreach mx = SchedMxList in {
  foreach nf=2-8 in {
    foreach eew = [8, 16, 32, 64] in {
      defvar Cycles = Andes45GetCyclesLoadStore<mx>.c;
      defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxList>.c;
      defvar Size = !mul(Andes45GetLMULValue<mx>.c, nf);
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 656-679: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
      let Latency = !add(4, !add(VLSU_MEM_LATENCY, !add(Size, 2))),
          ReleaseAtCycles = [!mul(Cycles, nf)] in {
        defm "" : LMULWriteResMX<"WriteVLSEG" # nf # "e" # eew,
                                 [Andes45VLSU], mx, IsWorstCase>;
        defm "" : LMULWriteResMX<"WriteVLSEGFF" # nf # "e" # eew,
                                 [Andes45VLSU], mx, IsWorstCase>;
      }
      let ReleaseAtCycles = [!mul(Cycles, nf)] in
      defm "" : LMULWriteResMX<"WriteVSSEG" # nf # "e" # eew,
                               [Andes45VLSU], mx, IsWorstCase>;
    }
  }
}

// Strided Segmented Loads and Stores

// The latency for loads is (5+VLSU_MEM_LATENCY+(DLEN/EEW)*NFIELDS)
// The throughput for loads and stores is VL*NFIELDS.
foreach mx = SchedMxList in {
  foreach nf=2-8 in {
    foreach eew = [8, 16, 32, 64] in {
      defvar Cycles = Andes45GetCyclesOnePerElement<mx, eew>.c;
      defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxList>.c;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 680-700: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
      let Latency = !add(5, !add(VLSU_MEM_LATENCY, !mul(!div(Andes45DLEN, eew), nf))),
          ReleaseAtCycles = [!mul(Cycles, nf)] in
        defm "" : LMULWriteResMX<"WriteVLSSEG" # nf # "e" # eew,
                                 [Andes45VLSU], mx, IsWorstCase>;
      let ReleaseAtCycles = [!mul(Cycles, nf)] in
        defm "" : LMULWriteResMX<"WriteVSSSEG" # nf # "e" # eew,
                                 [Andes45VLSU], mx, IsWorstCase>;
    }
  }
}

// Indexed Segmented Loads and Stores

// The latency for loads is (6+VLSU_MEM_LATENCY+(DLEN/EEW)*NFIELDS)
// The throughput for loads and stores is (VL*NFIELDS+EMUL-1).
foreach mx = SchedMxList in {
  foreach nf=2-8 in {
    foreach eew = [8, 16, 32, 64] in {
      defvar Cycles = Andes45GetCyclesOnePerElement<mx, eew>.c;
      defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxList>.c;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 701-718: Bulk record instantiation / 批量记录实例化
```tablegen
      let Latency = !add(6, !add(VLSU_MEM_LATENCY, !mul(!div(Andes45DLEN, eew), nf))),
          ReleaseAtCycles = [!add(!mul(Cycles, nf),
                                  !sub(Andes45GetLMULValue<mx>.c, 1))] in {
        defm "" : LMULWriteResMX<"WriteVLUXSEG" # nf # "e" # eew,
                                 [Andes45VLSU], mx, IsWorstCase>;
        defm "" : LMULWriteResMX<"WriteVLOXSEG" # nf # "e" # eew,
                                 [Andes45VLSU], mx, IsWorstCase>;
      }
      let ReleaseAtCycles = [!add(!mul(Cycles, nf),
                                  !sub(Andes45GetLMULValue<mx>.c, 1))] in {
        defm "" : LMULWriteResMX<"WriteVSUXSEG" # nf # "e" # eew,
                                 [Andes45VLSU], mx, IsWorstCase>;
        defm "" : LMULWriteResMX<"WriteVSOXSEG" # nf # "e" # eew,
                                 [Andes45VLSU], mx, IsWorstCase>;
      }
    }
  }
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 719-754: Bulk record instantiation LMul / 批量记录实例化 LMul
```tablegen

// Whole register move/load/store
foreach LMul = [1, 2, 4, 8] in {
  let Latency = 6, ReleaseAtCycles = [!mul(LMul, 2)] in
    def : WriteRes<!cast<SchedWrite>("WriteVLD" # LMul # "R"), [Andes45VLSU]>;
  let ReleaseAtCycles = [!mul(LMul, 2)] in
    def : WriteRes<!cast<SchedWrite>("WriteVST" # LMul # "R"), [Andes45VLSU]>;

  def : WriteRes<!cast<SchedWrite>("WriteVMov" # LMul # "V"), [Andes45VPERMUT]>;
}

// 11. Vector Integer Arithmetic Instructions
foreach mx = SchedMxList in {
  defvar Cycles = Andes45GetCyclesDefault<mx>.c;
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxList>.c;

  let Latency = 2, ReleaseAtCycles = [Cycles] in {
    defm "" : LMULWriteResMX<"WriteVIALUV", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIALUX", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIALUI", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICALUV", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICALUX", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICALUI", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICALUMV", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICALUMX", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICALUMI", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMinMaxV", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMinMaxX", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMergeV", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMergeX", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMergeI", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMovV", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMovX", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMovI", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVShiftV", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVShiftX", [Andes45VALU], mx, IsWorstCase>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 755-773: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
    defm "" : LMULWriteResMX<"WriteVShiftI", [Andes45VALU], mx, IsWorstCase>;
  }

  let Latency = 2, ReleaseAtCycles = [Cycles] in
    defm "" : LMULWriteResMX<"WriteVExtV", [Andes45VPERMUT], mx, IsWorstCase>;

  // Mask results can't chain.
  let Latency = !add(Cycles, 2),
      ReleaseAtCycles = [!add(Cycles, !ne(Andes45VLEN, Andes45DLEN))] in {
    defm "" : LMULWriteResMX<"WriteVICmpV", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICmpX", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICmpI", [Andes45VALU], mx, IsWorstCase>;
  }
}

foreach mx = SchedMxList in {
  defvar Cycles = Andes45GetCyclesDefault<mx>.c;
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxList>.c;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 774-791: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  let Latency = 4, ReleaseAtCycles = [Cycles] in {
    defm "" : LMULWriteResMX<"WriteVIMulV", [Andes45VMAC], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMulX", [Andes45VMAC], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMulAddV", [Andes45VMAC], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMulAddX", [Andes45VMAC], mx, IsWorstCase>;
  }
}

// Widening
foreach mx = SchedMxListW in {
  defvar Cycles = Andes45GetCyclesWidening<mx>.c;
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxListW>.c;

  let Latency = 2, ReleaseAtCycles = [Cycles] in {
    defm "" : LMULWriteResMX<"WriteVIWALUV", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIWALUX", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIWALUI", [Andes45VALU], mx, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 792-811: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen

  let Latency = 4, ReleaseAtCycles = [Cycles] in {
    defm "" : LMULWriteResMX<"WriteVIWMulV", [Andes45VMAC], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIWMulX", [Andes45VMAC], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIWMulAddV", [Andes45VMAC], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIWMulAddX", [Andes45VMAC], mx, IsWorstCase>;
  }
}

// Vector Integer Division and Remainder
foreach mx = SchedMxList in {
  foreach sew = SchedSEWSet<mx>.val in {
    defvar Latency = Andes45GetLatencyDiv<sew>.c;
    defvar Cycles = Andes45GetCyclesDiv<mx, sew>.c;
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxList>.c;

    let Latency = Latency, ReleaseAtCycles = [Cycles] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVIDivV", [Andes45VDIV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVIDivX", [Andes45VDIV], mx, sew, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 812-832: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  }
}

// Narrowing Shift
foreach mx = SchedMxListW in {
  defvar Latency = Andes45GetLatencyNarrowing<mx>.c;
  defvar Cycles = Andes45GetCyclesNarrowing<mx>.c;
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxListW>.c;

  let Latency = Latency, ReleaseAtCycles = [Cycles] in {
    defm "" : LMULWriteResMX<"WriteVNShiftV", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVNShiftX", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVNShiftI", [Andes45VALU], mx, IsWorstCase>;
  }
}

// 12. Vector Fixed-Point Arithmetic Instructions
foreach mx = SchedMxList in {
  defvar Cycles = Andes45GetCyclesDefault<mx>.c;
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxList>.c;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 833-855: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  let Latency = 2, ReleaseAtCycles = [Cycles] in {
    defm "" : LMULWriteResMX<"WriteVSALUV", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSALUX", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSALUI", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVAALUV", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVAALUX", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSShiftV", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSShiftX", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSShiftI", [Andes45VALU], mx, IsWorstCase>;
  }

  let Latency = 4, ReleaseAtCycles = [Cycles] in {
    defm "" : LMULWriteResMX<"WriteVSMulV", [Andes45VMAC], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSMulX", [Andes45VMAC], mx, IsWorstCase>;
  }
}

// Narrowing
foreach mx = SchedMxListW in {
  defvar Latency = Andes45GetLatencyNarrowing<mx>.c;
  defvar Cycles = Andes45GetCyclesNarrowing<mx>.c;
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxListW>.c;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 856-879: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  let Latency = Latency, ReleaseAtCycles = [Cycles] in {
    defm "" : LMULWriteResMX<"WriteVNClipV", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVNClipX", [Andes45VALU], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVNClipI", [Andes45VALU], mx, IsWorstCase>;
  }
}

// 13. Vector Floating-Point Instructions
foreach mx = SchedMxListF in {
  defvar Cycles = Andes45GetCyclesDefault<mx>.c;

  foreach sew = SchedSEWSet<mx, isF=1>.val in {
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxListF, isF=1>.c;

    // The latency for the CPU configured as 'FP32' is 5, and as 'FP32+FP64' is 6.
    // In most cases, CPU would be configured as "FP32+FP64".
    let Latency = 6, ReleaseAtCycles = [Cycles] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVFALUV", [Andes45VMAC], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFALUF", [Andes45VMAC], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFMulV", [Andes45VMAC], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFMulF", [Andes45VMAC], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFMulAddV", [Andes45VMAC], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFMulAddF", [Andes45VMAC], mx, sew, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 880-897: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  }
}

foreach mx = SchedMxListF in {
  defvar Cycles = Andes45GetCyclesDefault<mx>.c;

  foreach sew = SchedSEWSet<mx, isF=1>.val in {
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxListF, isF=1>.c;

    let Latency = 7, ReleaseAtCycles = [Cycles] in
      defm "" : LMULSEWWriteResMXSEW<"WriteVFRecpV", [Andes45VFDIV], mx, sew, IsWorstCase>;

    let Latency = 2, ReleaseAtCycles = [Cycles] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVFSgnjV", [Andes45VFMIS], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFSgnjF", [Andes45VFMIS], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFMinMaxV", [Andes45VFMIS], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFMinMaxF", [Andes45VFMIS], mx, sew, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 898-919: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen

    let Latency = 3, ReleaseAtCycles = [Cycles] in
      defm "" : LMULSEWWriteResMXSEW<"WriteVFCvtIToFV", [Andes45VFMIS], mx, sew, IsWorstCase>;
  }
}

foreach mx = SchedMxList in {
  defvar Cycles = Andes45GetCyclesDefault<mx>.c;
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxList>.c;

  // Mask results can't chain.
  let Latency = !add(Cycles, 2),
      ReleaseAtCycles = [!add(Cycles, !ne(Andes45VLEN, Andes45DLEN))] in {
    defm "" : LMULWriteResMX<"WriteVFCmpV", [Andes45VFMIS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVFCmpF", [Andes45VFMIS], mx, IsWorstCase>;
  }

  let Latency = 2, ReleaseAtCycles = [Cycles] in {
    defm "" : LMULWriteResMX<"WriteVFClassV", [Andes45VFMIS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVFMergeV", [Andes45VFMIS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVFMovV", [Andes45VFMIS], mx, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 920-940: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen

  let Latency = 3, ReleaseAtCycles = [Cycles] in
    defm "" : LMULWriteResMX<"WriteVFCvtFToIV", [Andes45VFMIS], mx, IsWorstCase>;
}

// Widening
foreach mx = SchedMxListW in {
  defvar Cycles = Andes45GetCyclesWidening<mx>.c;

  foreach sew = SchedSEWSet<mx, isF=0, isWidening=1>.val in {
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxListW>.c;

    let Latency = 3, ReleaseAtCycles = [Cycles] in
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWCvtIToFV", [Andes45VFMIS], mx, sew, IsWorstCase>;
  }
}

foreach mx = SchedMxListFW in {
  defvar Cycles = Andes45GetCyclesWidening<mx>.c;
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxListFW>.c;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 941-961: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  let Latency = 3, ReleaseAtCycles = [Cycles] in
    defm "" : LMULWriteResMX<"WriteVFWCvtFToIV", [Andes45VFMIS], mx, IsWorstCase>;
}

foreach mx = SchedMxListFW in {
  defvar Cycles = Andes45GetCyclesWidening<mx>.c;

  foreach sew = SchedSEWSet<mx, isF=1, isWidening=1>.val in {
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxListFW, isF=1>.c;

    // The latency for the CPU configured as 'FP32' is 5, and as 'FP32+FP64' is 6.
    // In most cases, CPU would be configured as "FP32+FP64".
    let Latency = 6, ReleaseAtCycles = [Cycles] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWALUV", [Andes45VMAC], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWALUF", [Andes45VMAC], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWMulV", [Andes45VMAC], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWMulF", [Andes45VMAC], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWMulAddV", [Andes45VMAC], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWMulAddF", [Andes45VMAC], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWCvtFToFV", [Andes45VFMIS], mx, sew, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 962-981: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  }
}

// Narrowing
foreach mx = SchedMxListW in {
  defvar Latency = !add(Andes45GetLatencyNarrowing<mx>.c, 1);
  defvar Cycles = Andes45GetCyclesNarrowing<mx>.c;
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxListW>.c;

  let Latency = Latency, ReleaseAtCycles = [Cycles] in
    defm "" : LMULWriteResMX<"WriteVFNCvtFToIV", [Andes45VFMIS], mx, IsWorstCase>;
}

foreach mx = SchedMxListFW in {
  defvar Latency = !add(Andes45GetLatencyNarrowing<mx>.c, 1);
  defvar Cycles = Andes45GetCyclesNarrowing<mx>.c;

  foreach sew = SchedSEWSet<mx, isF=1, isWidening=1>.val in {
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxListFW, isF=1>.c;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 982-999: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
    let Latency = Latency, ReleaseAtCycles = [Cycles] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVFNCvtIToFV", [Andes45VFMIS], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFNCvtFToFV", [Andes45VFMIS], mx, sew, IsWorstCase>;
    }
  }
}

// Vector Floating-Point Division and Square Root
foreach mx = SchedMxListF in {
  foreach sew = SchedSEWSet<mx, 1>.val in {
    defvar Factor = Andes45GetFDivFactor<sew>.c;
    defvar Cycles = !mul(Andes45GetCyclesDefault<mx>.c, Factor);
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxListF, 1>.c;

    let Latency = !add(2, Factor), ReleaseAtCycles = [Cycles] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVFDivV", [Andes45VFDIV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFDivF", [Andes45VFDIV], mx, sew, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1000-1019: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  }
}

foreach mx = SchedMxListF in {
  foreach sew = SchedSEWSet<mx, 1>.val in {
    defvar Factor = Andes45GetFSqrtFactor<sew>.c;
    defvar Cycles = !mul(Andes45GetCyclesDefault<mx>.c, Factor);
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxListF, 1>.c;

    let Latency = !add(2, Factor), ReleaseAtCycles = [Cycles] in
      defm "" : LMULSEWWriteResMXSEW<"WriteVFSqrtV", [Andes45VFDIV], mx, sew, IsWorstCase>;
  }
}

// 14. Vector Reduction Operations
foreach mx = SchedMxList in {
  foreach sew = SchedSEWSet<mx>.val in {
    defvar Cycles = Andes45GetReductionCycles<mx, sew>.c;
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxList>.c;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1020-1037: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
    let Latency = !add(Cycles, 1),
        ReleaseAtCycles = [!add(Cycles, !ne(Andes45VLEN, Andes45DLEN))] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVIRedV_From", [Andes45VALU], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVIRedMinMaxV_From", [Andes45VALU], mx, sew, IsWorstCase>;
    }
  }
}

foreach mx = SchedMxListWRed in {
  foreach sew = SchedSEWSet<mx, 0, 1>.val in {
    defvar Cycles = Andes45GetReductionCyclesWidening<mx, sew>.c;
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxListWRed>.c;

    let Latency = !add(Cycles, 1),
        ReleaseAtCycles = [!add(Cycles, !ne(Andes45VLEN, Andes45DLEN))] in
      defm "" : LMULSEWWriteResMXSEW<"WriteVIWRedV_From", [Andes45VALU], mx, sew, IsWorstCase>;
  }
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1038-1056: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen

foreach mx = SchedMxListF in {
  foreach sew = SchedSEWSet<mx, 1>.val in {
    defvar Cycles = Andes45GetFReductionCycles<mx, sew>.c;
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxListF, 1>.c;

        // 4*vfredusum-micro-ops+2
    let Latency = !add(!mul(4, Cycles), 2),
        // 1+4*(vfredusum-micro-ops-1)+(VLEN!=DLEN)
        ReleaseAtCycles = [!add(1, !add(!mul(4, !sub(Cycles, 1)), !ne(Andes45VLEN, Andes45DLEN)))] in
      defm "" : LMULSEWWriteResMXSEW<"WriteVFRedV_From", [Andes45VMAC], mx, sew, IsWorstCase>;
  }
}

foreach mx = SchedMxListF in {
  foreach sew = SchedSEWSet<mx, 1>.val in {
    defvar Cycles = Andes45GetOrderedFReductionCycles<mx, sew>.c;
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxListF, 1>.c;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1057-1074: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
        // 4*vfredosum-micro-ops+2
    let Latency = !add(!mul(4, Cycles), 2),
        // 1+4*(vfredosum-micro-ops-1)+(VLEN!=DLEN)
        ReleaseAtCycles = [!add(1, !add(!mul(4, !sub(Cycles, 1)), !ne(Andes45VLEN, Andes45DLEN)))] in
      defm "" : LMULSEWWriteResMXSEW<"WriteVFRedOV_From", [Andes45VMAC], mx, sew, IsWorstCase>;
  }
}

foreach mx = SchedMxListF in {
  foreach sew = SchedSEWSet<mx, 1>.val in {
    defvar Cycles = Andes45GetReductionCycles<mx, sew>.c;
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxListF, 1>.c;

    let Latency = !add(Cycles, 1),
        ReleaseAtCycles = [!add(Cycles, !ne(Andes45VLEN, Andes45DLEN))] in
      defm "" : LMULSEWWriteResMXSEW<"WriteVFRedMinMaxV_From", [Andes45VFMIS], mx, sew, IsWorstCase>;
  }
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1075-1093: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen

foreach mx = SchedMxListFWRed in {
  foreach sew = SchedSEWSet<mx, 1, 1>.val in {
    defvar Cycles = Andes45GetFWReductionCycles<mx, sew>.c;
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxListFWRed, 1>.c;

        // 4*vfwredusum-micro-ops+2
    let Latency = !add(!mul(4, Cycles), 2),
        // 1+4*(vfwredusum-micro-ops-1)+(VLEN!=DLEN)
        ReleaseAtCycles = [!add(1, !add(!mul(4, !sub(Cycles, 1)), !ne(Andes45VLEN, Andes45DLEN)))] in
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWRedV_From", [Andes45VMAC], mx, sew, IsWorstCase>;
  }
}

foreach mx = SchedMxListFWRed in {
  foreach sew = SchedSEWSet<mx, 1, 1>.val in {
    defvar Cycles = Andes45GetOrderedFWReductionCycles<mx, sew>.c;
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxListFWRed, 1>.c;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1094-1115: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
        // 4*vfwredosum-micro-ops+2
    let Latency = !add(!mul(4, Cycles), 2),
        // 1+4*(vfwredosum-micro-ops-1)+(VLEN != DLEN)
        ReleaseAtCycles = [!add(1, !add(!mul(4, !sub(Cycles, 1)), !ne(Andes45VLEN, Andes45DLEN)))] in
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWRedOV_From", [Andes45VMAC], mx, sew, IsWorstCase>;
  }
}

// 15. Vector Mask Instructions
foreach mx = SchedMxList in {
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxList>.c;

  let Latency = 3, ReleaseAtCycles = [Andes45VLEN_DLEN_RATIO] in {
    defm "" : LMULWriteResMX<"WriteVMALUV", [Andes45VMASK], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVMSFSV", [Andes45VMASK], mx, IsWorstCase>;
  }

  let Latency = !add(3, !ne(Andes45VLEN, Andes45DLEN)),
      ReleaseAtCycles = [Andes45VLEN_DLEN_RATIO] in {
    defm "" : LMULWriteResMX<"WriteVMPopV", [Andes45VMASK], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVMFFSV", [Andes45VMASK], mx, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1116-1134: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen

}
// TODO: viota and vid have different latency and throughput if VLEN/DLEN=2.
foreach mx = SchedMxList in {
  defvar Cycles = Andes45GetCyclesDefault<mx>.c;
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxList>.c;
  let Latency = 4, ReleaseAtCycles = [Cycles] in {
    defm "" : LMULWriteResMX<"WriteVIotaV", [Andes45VMASK], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIdxV", [Andes45VMASK], mx, IsWorstCase>;
  }
}

// 16. Vector Permutation Instructions
// TODO: Use Andes45GetCyclesDefault to get cycles since we are unable to model
// the Latency and ReleaseAtCycles accurately now.
foreach mx = SchedMxList in {
  defvar Cycles = Andes45GetCyclesDefault<mx>.c;
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxList>.c;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1135-1154: Bulk record instantiation / 批量记录实例化
```tablegen
  let Latency = Cycles, ReleaseAtCycles = [Cycles] in {
    defm "" : LMULWriteResMX<"WriteVSlideI", [Andes45VPERMUT], mx, IsWorstCase>;

    defm "" : LMULWriteResMX<"WriteVISlide1X", [Andes45VPERMUT], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVFSlide1F", [Andes45VPERMUT], mx, IsWorstCase>;

    defm "" : LMULWriteResMX<"WriteVSlideUpX", [Andes45VPERMUT], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSlideDownX", [Andes45VPERMUT], mx, IsWorstCase>;
  }
}

let Latency = 2, ReleaseAtCycles = [Andes45VLEN_DLEN_RATIO] in {
  def : WriteRes<WriteVMovSX, [Andes45VPERMUT]>;
  def : WriteRes<WriteVMovSF, [Andes45VPERMUT]>;
}

let Latency = 3, ReleaseAtCycles = [Andes45VLEN_DLEN_RATIO] in {
  def : WriteRes<WriteVMovXS, [Andes45VPERMUT]>;
  def : WriteRes<WriteVMovFS, [Andes45VPERMUT]>;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1155-1176: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen

// Gather and Compress
// TODO: Use Andes45GetCyclesDefault to get cycles since we are unable to model
// the Latency and ReleaseAtCycles accurately now.
foreach mx = SchedMxList in {
  foreach sew = SchedSEWSet<mx>.val in {
    defvar Cycles = Andes45GetCyclesDefault<mx>.c;
    defvar IsWorstCase = Andes45IsWorstCaseMXSEW<mx, sew, SchedMxList>.c;
    let Latency = Cycles, ReleaseAtCycles = [Cycles] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVRGatherVV", [Andes45VPERMUT], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVRGatherEI16VV", [Andes45VPERMUT], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVCompressV", [Andes45VPERMUT], mx, sew, IsWorstCase>;
    }
  }
}

// TODO: Use Andes45GetCyclesDefault to get cycles since we are unable to model
// the Latency and ReleaseAtCycles accurately now.
foreach mx = SchedMxList in {
  defvar Cycles = Andes45GetCyclesDefault<mx>.c;
  defvar IsWorstCase = Andes45IsWorstCaseMX<mx, SchedMxList>.c;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1177-1212: Bulk record instantiation / 批量记录实例化
```tablegen
  let Latency = Cycles, ReleaseAtCycles = [Cycles] in {
    defm "" : LMULWriteResMX<"WriteVRGatherVX", [Andes45VPERMUT], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVRGatherVI", [Andes45VPERMUT], mx, IsWorstCase>;
  }
}

// Others
def : WriteRes<WriteRdVLENB, [Andes45CSR]>;

// 6. Configuration-Setting Instructions
def : ReadAdvance<ReadVSETVLI, 0>;
def : ReadAdvance<ReadVSETVL, 0>;

// 7. Vector Loads and Stores
def : ReadAdvance<ReadVLDX, 0>;
def : ReadAdvance<ReadVSTX, 0>;
defm "" : LMULReadAdvance<"ReadVSTEV", 0>;
defm "" : LMULReadAdvance<"ReadVSTM", 0>;
def : ReadAdvance<ReadVLDSX, 0>;
def : ReadAdvance<ReadVSTSX, 0>;
defm "" : LMULReadAdvance<"ReadVSTS8V", 0>;
defm "" : LMULReadAdvance<"ReadVSTS16V", 0>;
defm "" : LMULReadAdvance<"ReadVSTS32V", 0>;
defm "" : LMULReadAdvance<"ReadVSTS64V", 0>;
defm "" : LMULReadAdvance<"ReadVLDUXV", 0>;
defm "" : LMULReadAdvance<"ReadVLDOXV", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX8", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX16", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX32", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX64", 0>;
defm "" : LMULReadAdvance<"ReadVSTUXV", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX8V", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX16V", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX32V", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX64V", 0>;
defm "" : LMULReadAdvance<"ReadVSTOX8", 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1213-1248: Bulk record instantiation / 批量记录实例化
```tablegen
defm "" : LMULReadAdvance<"ReadVSTOX16", 0>;
defm "" : LMULReadAdvance<"ReadVSTOX32", 0>;
defm "" : LMULReadAdvance<"ReadVSTOX64", 0>;
defm "" : LMULReadAdvance<"ReadVSTOXV", 0>;
defm "" : LMULReadAdvance<"ReadVSTOX8V", 0>;
defm "" : LMULReadAdvance<"ReadVSTOX16V", 0>;
defm "" : LMULReadAdvance<"ReadVSTOX32V", 0>;
defm "" : LMULReadAdvance<"ReadVSTOX64V", 0>;
// LMUL Aware
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
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1249-1269: Bulk record instantiation / 批量记录实例化
```tablegen
defm : LMULReadAdvance<"ReadVIMulAddV", 0>;
defm : LMULReadAdvance<"ReadVIMulAddX", 0>;
defm : LMULReadAdvanceW<"ReadVIWMulAddV", 0>;
defm : LMULReadAdvanceW<"ReadVIWMulAddX", 0>;
defm : LMULReadAdvance<"ReadVIMergeV", 0>;
defm : LMULReadAdvance<"ReadVIMergeX", 0>;
defm : LMULReadAdvance<"ReadVIMovV", 0>;
defm : LMULReadAdvance<"ReadVIMovX", 0>;

// 12. Vector Fixed-Point Arithmetic Instructions
defm "" : LMULReadAdvance<"ReadVSALUV", 0>;
defm "" : LMULReadAdvance<"ReadVSALUX", 0>;
defm "" : LMULReadAdvance<"ReadVAALUV", 0>;
defm "" : LMULReadAdvance<"ReadVAALUX", 0>;
defm "" : LMULReadAdvance<"ReadVSMulV", 0>;
defm "" : LMULReadAdvance<"ReadVSMulX", 0>;
defm "" : LMULReadAdvance<"ReadVSShiftV", 0>;
defm "" : LMULReadAdvance<"ReadVSShiftX", 0>;
defm "" : LMULReadAdvanceW<"ReadVNClipV", 0>;
defm "" : LMULReadAdvanceW<"ReadVNClipX", 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1270-1305: Bulk record instantiation / 批量记录实例化
```tablegen
// 13. Vector Floating-Point Instructions
defm "" : LMULSEWReadAdvanceF<"ReadVFALUV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFALUF", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFWALUV", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFWALUF", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFMulV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFMulF", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFDivV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFDivF", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFWMulV", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFWMulF", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFMulAddV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFMulAddF", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFWMulAddV", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFWMulAddF", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFSqrtV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFRecpV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFMinMaxV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFMinMaxF", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFSgnjV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFSgnjF", 0>;
defm "" : LMULReadAdvance<"ReadVFCmpV", 0>;
defm "" : LMULReadAdvance<"ReadVFCmpF", 0>;
defm "" : LMULReadAdvance<"ReadVFClassV", 0>;
defm "" : LMULReadAdvance<"ReadVFMergeV", 0>;
defm "" : LMULReadAdvance<"ReadVFMergeF", 0>;
defm "" : LMULReadAdvance<"ReadVFMovF", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFCvtIToFV", 0>;
defm "" : LMULReadAdvance<"ReadVFCvtFToIV", 0>;
defm "" : LMULSEWReadAdvanceW<"ReadVFWCvtIToFV", 0>;
defm "" : LMULReadAdvanceFW<"ReadVFWCvtFToIV", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFWCvtFToFV", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFNCvtIToFV", 0>;
defm "" : LMULReadAdvanceW<"ReadVFNCvtFToIV", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFNCvtFToFV", 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1306-1326: Bulk record instantiation / 批量记录实例化
```tablegen
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

// 15. Vector Mask Instructions
defm "" : LMULReadAdvance<"ReadVMALUV", 0>;
defm "" : LMULReadAdvance<"ReadVMPopV", 0>;
defm "" : LMULReadAdvance<"ReadVMFFSV", 0>;
defm "" : LMULReadAdvance<"ReadVMSFSV", 0>;
defm "" : LMULReadAdvance<"ReadVIotaV", 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1327-1351: Bulk record instantiation / 批量记录实例化
```tablegen
// 16. Vector Permutation Instructions
def : ReadAdvance<ReadVMovXS, 0>;
def : ReadAdvance<ReadVMovSX_V, 0>;
def : ReadAdvance<ReadVMovSX_X, 0>;
def : ReadAdvance<ReadVMovFS, 0>;
def : ReadAdvance<ReadVMovSF_V, 0>;
def : ReadAdvance<ReadVMovSF_F, 0>;
defm "" : LMULReadAdvance<"ReadVISlideV", 0>;
defm "" : LMULReadAdvance<"ReadVISlideX", 0>;
defm "" : LMULReadAdvance<"ReadVFSlideV", 0>;
defm "" : LMULReadAdvance<"ReadVFSlideF", 0>;
defm "" : LMULSEWReadAdvance<"ReadVRGatherVV_data", 0>;
defm "" : LMULSEWReadAdvance<"ReadVRGatherVV_index", 0>;
defm "" : LMULSEWReadAdvance<"ReadVRGatherEI16VV_data", 0>;
defm "" : LMULSEWReadAdvance<"ReadVRGatherEI16VV_index", 0>;
defm "" : LMULReadAdvance<"ReadVRGatherVX_data", 0>;
defm "" : LMULReadAdvance<"ReadVRGatherVX_index", 0>;
defm "" : LMULReadAdvance<"ReadVRGatherVI_data", 0>;
defm "" : LMULSEWReadAdvance<"ReadVCompressV", 0>;
// LMUL Aware
def : ReadAdvance<ReadVMov1V, 0>;
def : ReadAdvance<ReadVMov2V, 0>;
def : ReadAdvance<ReadVMov4V, 0>;
def : ReadAdvance<ReadVMov8V, 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1352-1370: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
// Others
def : ReadAdvance<ReadVMask, 0>;
def : ReadAdvance<ReadVPassthru_WorstCase, 0>;
foreach mx = SchedMxList in {
  def : ReadAdvance<!cast<SchedRead>("ReadVPassthru_" # mx), 0>;
  foreach sew = SchedSEWSet<mx>.val in
    def : ReadAdvance<!cast<SchedRead>("ReadVPassthru_" # mx  # "_E" # sew), 0>;
}

//===----------------------------------------------------------------------===//
// Unsupported extensions
defm : UnsupportedSchedQ;
defm : UnsupportedSchedZabha;
defm : UnsupportedSchedZbkb;
defm : UnsupportedSchedZbkx;
defm : UnsupportedSchedZfa;
defm : UnsupportedSchedZvk;
defm : UnsupportedSchedXsf;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

## Key Concepts / 关键概念
- **Scheduling models** / **调度模型**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
