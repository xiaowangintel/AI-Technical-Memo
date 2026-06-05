# AArch64Schedule.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64Schedule.td`
- **Repository**: llvm/llvm-project
- **Purpose**: AArch64 Scheduler Definitions. It is written in the TableGen DSL. / 该文件实现 AArch64 后端中的调度模型。 它使用 TableGen DSL 描述目标规则。
- **Note**: Written in the TableGen DSL / 使用 TableGen DSL 编写
## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Documented TableGen section
```tablegen
//==-- AArch64Schedule.td - AArch64 Scheduling Definitions -*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

// Define TII for use in SchedVariant Predicates.
// const MachineInstr *MI and const TargetSchedModel *SchedModel
// are defined by default.
def : PredicateProlog<[{
  const AArch64InstrInfo *TII =
    static_cast<const AArch64InstrInfo*>(SchedModel->getInstrInfo());
  (void)TII;
}]>;
```
**EN:** This comment block marks a new TableGen section and explains the intent of the records that follow.  
**CN:** 该注释块标记了新的 TableGen 章节，并说明后续记录的设计意图。
### Lines 17-33: TableGen definition WriteImm
```tablegen

// AArch64 Scheduler Definitions

def WriteImm       : SchedWrite; // MOVN, MOVZ
// TODO: Provide variants for MOV32/64imm Pseudos that dynamically
// select the correct sequence of WriteImms.

def WriteI         : SchedWrite; // ALU
def WriteISReg     : SchedWrite; // ALU of Shifted-Reg
def WriteIEReg     : SchedWrite; // ALU of Extended-Reg
def ReadI          : SchedRead;  // ALU
def ReadISReg      : SchedRead;  // ALU of Shifted-Reg
def ReadIEReg      : SchedRead;  // ALU of Extended-Reg
def WriteExtr      : SchedWrite; // EXTR shifts a reg pair
def ReadExtrHi     : SchedRead;  // Read the high reg of the EXTR pair
def WriteIS        : SchedWrite; // Shift/Scale
def WriteID32      : SchedWrite; // 32-bit Divide
```
**EN:** This definition materializes WriteImm as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 WriteImm 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 34-46: TableGen definition WriteID64
```tablegen
def WriteID64      : SchedWrite; // 64-bit Divide
def ReadID         : SchedRead;  // 32/64-bit Divide
def WriteIM32      : SchedWrite; // 32-bit Multiply
def WriteIM64      : SchedWrite; // 64-bit Multiply
def ReadIM         : SchedRead;  // 32/64-bit Multiply
def ReadIMA        : SchedRead;  // 32/64-bit Multiply Accumulate
def WriteBr        : SchedWrite; // Branch
def WriteBrReg     : SchedWrite; // Indirect Branch

def WriteLD        : SchedWrite; // Load from base addr plus immediate offset
def WriteST        : SchedWrite; // Store to base addr plus immediate offset
def WriteSTP       : SchedWrite; // Store a register pair.
def WriteAdr       : SchedWrite; // Address pre/post increment.
```
**EN:** This definition materializes WriteID64 as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 WriteID64 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 47-63: TableGen definition WriteLDIdx
```tablegen

def WriteLDIdx : SchedWrite; // Load from a register index (maybe scaled).
def WriteSTIdx : SchedWrite; // Store to a register index (maybe scaled).
def ReadST     : SchedRead;  // Read the stored value.
def ReadAdrBase : SchedRead; // Read the base resister of a reg-offset LD/ST.

// Serialized two-level address load.
// EXAMPLE: LOADGot
def WriteLDAdr : WriteSequence<[WriteAdr, WriteLD]>;

// Serialized two-level address lookup.
// EXAMPLE: MOVaddr...
def WriteAdrAdr : WriteSequence<[WriteAdr, WriteAdr]>;

// The second register of a load-pair.
// LDP,LDPSW,LDNP,LDXP,LDAXP
def WriteLDHi : SchedWrite;
```
**EN:** This definition materializes WriteLDIdx as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 WriteLDIdx 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 64-78: TableGen definition WriteSTX
```tablegen

// Store-exclusive is a store followed by a dependent load.
def WriteSTX : WriteSequence<[WriteST, WriteLD]>;

def WriteSys     : SchedWrite; // Long, variable latency system ops.
def WriteBarrier : SchedWrite; // Memory barrier.
def WriteHint    : SchedWrite; // Hint instruction.

def WriteF       : SchedWrite; // General floating-point ops.
def WriteFCmp    : SchedWrite; // Floating-point compare.
def WriteFCvt    : SchedWrite; // Float conversion.
def WriteFCopy   : SchedWrite; // Float-int register copy.
def WriteFImm    : SchedWrite; // Floating-point immediate.
def WriteFMul    : SchedWrite; // Floating-point multiply.
def WriteFDiv    : SchedWrite; // Floating-point division.
```
**EN:** This definition materializes WriteSTX as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 WriteSTX 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 79-92: TableGen definition WriteVd
```tablegen

def WriteVd  : SchedWrite; // 64bit Vector D ops.
def WriteVq  : SchedWrite; // 128bit Vector Q ops.
def WriteVLD : SchedWrite; // Vector loads.
def WriteVST : SchedWrite; // Vector stores.

def WriteAtomic : SchedWrite; // Atomic memory operations (CAS, Swap, LDOP)

// Read the unwritten lanes of the VLD's destination registers.
def ReadVLD : SchedRead;

// Sequential vector load and shuffle.
def WriteVLDShuffle     : WriteSequence<[WriteVLD, WriteVq]>;
def WriteVLDPairShuffle : WriteSequence<[WriteVLD, WriteVq, WriteVq]>;
```
**EN:** This definition materializes WriteVd as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 WriteVd 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 93-100: TableGen definition WriteVSTShuffle
```tablegen

// Store a shuffled vector.
def WriteVSTShuffle : WriteSequence<[WriteVq, WriteVST]>;
def WriteVSTPairShuffle : WriteSequence<[WriteVq, WriteVq, WriteVST]>;

// SIMD MAC forwarding reads
def ReadVMAC       : SchedRead;  // SIMD MAC operand read
def ReadVMACAccum  : SchedRead;  // SIMD MAC accumulator/destination read
```
**EN:** This definition materializes WriteVSTShuffle as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 WriteVSTShuffle 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Per-core scheduling itineraries and latency models **CN:** 面向具体内核的调度行程与时延模型
- **EN:** Declarative backend description in TableGen DSL **CN:** 使用 TableGen DSL 的声明式后端描述
## Dependencies / 依赖关系
- **EN:** Consumed by LLVM TableGen to produce generated matchers, encoders, register info, or scheduling tables. **CN:** 该文件会被 LLVM TableGen 消费，用于生成匹配器、编码器、寄存器信息或调度表。
- **EN:** Closely connected with neighboring AArch64 backend components responsible for scheduling models. **CN:** 与周边负责调度模型的 AArch64 后端组件紧密协作。
