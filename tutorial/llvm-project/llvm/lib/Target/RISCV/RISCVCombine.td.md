# RISCVCombine.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVCombine.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Commentary and design intent / 注释与设计意图
```tablegen
//=- RISCVCombine.td - Define RISC-V Combine Rules -----------*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 11-20: Included TableGen fragments / 引入的 TableGen 片段
```tablegen

include "llvm/Target/GlobalISel/Combine.td"

def RISCVPreLegalizerCombiner: GICombiner<
  "RISCVPreLegalizerCombinerImpl", [all_combines]> {
}

def RISCVO0PreLegalizerCombiner: GICombiner<
  "RISCVO0PreLegalizerCombinerImpl", [optnone_combines]> {
}
```
**EN:** This TableGen block composes the file from reusable fragments so generated backend records can share common definitions.

**CN:** 该 TableGen 区段通过复用片段来组合文件，使生成的后端记录能够共享公共定义。

### Lines 21-29: TableGen record fp_zero_store_matchdata / TableGen 记录 fp_zero_store_matchdata
```tablegen

// Rule: fold store (fp +0.0) -> store (int zero [XLEN])
def fp_zero_store_matchdata : GIDefMatchData<"Register">;
def fold_fp_zero_store : GICombineRule<
  (defs root:$root, fp_zero_store_matchdata:$matchinfo),
  (match (G_STORE $src, $addr):$root,
         [{ return matchFoldFPZeroStore(*${root}, MRI, STI, ${matchinfo}); }]),
  (apply [{ applyFoldFPZeroStore(*${root}, MRI, B, STI, ${matchinfo}); }])>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 30-38: TableGen record RISCVPostLegalizerCombiner / TableGen 记录 RISCVPostLegalizerCombiner
```tablegen
// Post-legalization combines which are primarily optimizations.
// TODO: Add more combines.
def RISCVPostLegalizerCombiner
    : GICombiner<"RISCVPostLegalizerCombinerImpl",
                 [sub_to_add, combines_for_extload, redundant_and,
                  identity_combines, shift_immed_chain,
                  commute_constant_to_rhs, simplify_neg_minmax,
                  fold_fp_zero_store]> {
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- `llvm/Target/GlobalISel/Combine.td` — Directly referenced by this file. / 该文件直接引用的依赖。
