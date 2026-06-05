# scope_id.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Proton/scope_id.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises scope-id behavior in Triton's Proton pipeline coverage. **CN:** 该MLIR 测试用于覆盖 Triton Proton 流水线中的scope-id 行为相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt --split-input-file --test-print-scope-id-allocation -verify-diagnostics=only-expected -o /dev/null %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: none; diagnostics annotations: error×6, remark×97. **CN:** FileCheck 标记为 none；诊断标注为 error×6, remark×97。
- **Validation / 验证内容:** **EN:** The file mainly validates that scope-id behavior rejects invalid inputs or reports the expected diagnostics. **CN:** 该文件主要验证 scope-id 行为 在遇到非法输入时会拒绝它们，或给出预期的诊断信息。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt --split-input-file --test-print-scope-id-allocation -verify-diagnostics=only-expected -o /dev/null %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt --split-input-file --test-print-scope-id-allocation -verify-diagnostics=only-expected -o /dev/null %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt --split-input-file --test-print-scope-id-allocation -verify-diagnostics=only-expected -o /dev/null %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-13
```mlir
module {
  // expected-remark @below {{one_scope}}
  tt.func @one_scope() {
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "name0"
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "name0"
    tt.return
  }
```
**EN:** This block defines `one_scope` and exercises operations such as `tt.func`, `proton.record`, `tt.return`. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `one_scope`，并覆盖 如 `tt.func`、`proton.record`、`tt.return` 这样的操作。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Lines 15-30
```mlir
  // expected-remark @below {{two_scopes}}
  tt.func @two_scopes() {
    // expected-remark @below {{scope id = 1}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "name0"
    // expected-remark @below {{scope id = 1}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "name0"
    // expected-remark @below {{scope id = 2}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "name1"
    // expected-remark @below {{scope id = 2}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "name1"
    tt.return
  }
```
**EN:** This block defines `two_scopes` and exercises operations such as `tt.func`, `proton.record`, `tt.return`. Expected-diagnostic annotations (remark×9) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `two_scopes`，并覆盖 如 `tt.func`、`proton.record`、`tt.return` 这样的操作。 期望诊断标注（remark×9）用于捕获 pass 应当发出的分析备注/说明。

### Lines 32-47
```mlir
  // expected-remark @below {{two_scopes_overlap}}
  tt.func @two_scopes_overlap() {
    // expected-remark @below {{scope id = 3}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "name0"
    // expected-remark @below {{scope id = 4}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "name1"
    // expected-remark @below {{scope id = 3}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "name0"
    // expected-remark @below {{scope id = 4}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "name1"
    tt.return
  }
```
**EN:** This block defines `two_scopes_overlap` and exercises operations such as `tt.func`, `proton.record`, `tt.return`. Expected-diagnostic annotations (remark×9) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `two_scopes_overlap`，并覆盖 如 `tt.func`、`proton.record`、`tt.return` 这样的操作。 期望诊断标注（remark×9）用于捕获 pass 应当发出的分析备注/说明。

### Lines 49-65
```mlir
  // expected-remark @below {{nested_scopes}}
  tt.func @nested_scopes() {
    // expected-remark @below {{scope id = 5}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "name0"
    // expected-remark @below {{scope id = 6}}
    // expected-remark @below {{scope parent id = 5}}
    proton.record start "name1"
    // expected-remark @below {{scope id = 6}}
    // expected-remark @below {{scope parent id = 5}}
    proton.record end "name1"
    // expected-remark @below {{scope id = 5}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "name0"
    tt.return
  }
}
```
**EN:** This block defines `nested_scopes` and exercises operations such as `tt.func`, `proton.record`, `tt.return`. Expected-diagnostic annotations (remark×9) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `nested_scopes`，并覆盖 如 `tt.func`、`proton.record`、`tt.return` 这样的操作。 期望诊断标注（remark×9）用于捕获 pass 应当发出的分析备注/说明。

### Line 67
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 69-79
```mlir
module {
  // expected-remark @below {{inner}}
  tt.func @inner() {
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "name0"
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "name0"
    tt.return
  }
```
**EN:** This block defines `inner` and exercises operations such as `tt.func`, `proton.record`, `tt.return`. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `inner`，并覆盖 如 `tt.func`、`proton.record`、`tt.return` 这样的操作。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Lines 81-92
```mlir
  // expected-remark @below {{outer}}
  tt.func @outer() {
    // expected-remark @below {{scope id = 1}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "name0"
    tt.call @inner() : () -> ()
    // expected-remark @below {{scope id = 1}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "name0"
    tt.return
  }
}
```
**EN:** This block defines `outer`, `inner` and exercises operations such as `tt.func`, `proton.record`, `tt.call`, `tt.return`. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `outer`, `inner`，并覆盖 如 `tt.func`、`proton.record`、`tt.call`、`tt.return` 这样的操作。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Line 94
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 96-113
```mlir
module {
  // expected-remark @below {{duplicate}}
  tt.func @duplicate() {
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "name0"
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "name0"
    // expected-remark @below {{scope id = 1}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "name0"
    // expected-remark @below {{scope id = 1}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "name0"
    tt.return
  }
}
```
**EN:** This block defines `duplicate` and exercises operations such as `tt.func`, `proton.record`, `tt.return`. Expected-diagnostic annotations (remark×9) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `duplicate`，并覆盖 如 `tt.func`、`proton.record`、`tt.return` 这样的操作。 期望诊断标注（remark×9）用于捕获 pass 应当发出的分析备注/说明。

### Line 115
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 117-133
```mlir
module {
  // expected-remark @below {{cf_reordered}}
  tt.func @cf_reordered() {
  ^entry:
    cf.br ^start
  ^exit:
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "name0"
    tt.return
  ^start:
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "name0"
    cf.br ^exit
  }
}
```
**EN:** This block defines `cf_reordered` and exercises operations such as `tt.func`, `cf.br`, `proton.record`, `tt.return`. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `cf_reordered`，并覆盖 如 `tt.func`、`cf.br`、`proton.record`、`tt.return` 这样的操作。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Line 135
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 137-150
```mlir
module {
  // expected-remark @below {{scf_cond}}
  tt.func @scf_cond(%cond: i1) {
    scf.if %cond {
      // expected-remark @below {{scope id = 0}}
      // expected-remark @below {{scope parent id = -1}}
      proton.record start "if_only"
    }
    // expected-remark @below {{scope id = 0}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "if_only"
    tt.return
  }
}
```
**EN:** This block defines `scf_cond` and exercises control-flow joins. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `scf_cond`，并覆盖 控制流汇合。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Line 152
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 154-169
```mlir
module {
  tt.func @scf_loop() {
    %c0 = arith.constant 0 : index
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "loop"
    scf.for %i = %c0 to %c0 step %c0 {
      // expected-remark @below {{scope id = 1}}
      // expected-remark @below {{scope parent id = 0}}
      proton.record start "loop_body"
      proton.record end "loop_body"
    }
    proton.record end "loop"
    tt.return
  }
}
```
**EN:** This block defines `scf_loop` and exercises loop-carried state. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `scf_loop`，并覆盖 循环携带状态。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Line 171
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 173-190
```mlir
module {
  tt.func @scf_loop_if(%cond: i1) {
    %c0 = arith.constant 0 : index
    scf.for %i = %c0 to %c0 step %c0 {
      scf.if %cond {
        // expected-remark @below {{scope id = 0}}
        // expected-remark @below {{scope parent id = -1}}
        proton.record start "loop_if"
      }
      scf.if %cond {
        // expected-remark @below {{scope id = 0}}
        // expected-remark @below {{scope parent id = -1}}
        proton.record end "loop_if"
      }
    }
    tt.return
  }
}
```
**EN:** This block defines `scf_loop_if` and exercises loop-carried state, control-flow joins. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `scf_loop_if`，并覆盖 循环携带状态、控制流汇合。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Line 192
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 194-211
```mlir
module {
  // expected-remark @below {{cf_single_branch}}
  tt.func @cf_single_branch(%cond: i1) {
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "name0"
    cf.cond_br %cond, ^then, ^else
  ^then:  // pred: ^entry
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "name0"
    cf.br ^merge
  ^else:  // pred: ^entry
    cf.br ^merge
  ^merge:  // preds: ^then, ^else
    tt.return
  }
}
```
**EN:** This block defines `cf_single_branch` and exercises operations such as `tt.func`, `proton.record`, `cf.cond_br`, `cf.br`, `tt.return`. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `cf_single_branch`，并覆盖 如 `tt.func`、`proton.record`、`cf.cond_br`、`cf.br`、`tt.return` 这样的操作。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Line 214
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 216-246
```mlir
module {
  // expected-remark @below {{warp_specialize_balanced}}
  tt.func @warp_specialize_balanced() {
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "outer"
    ttg.warp_specialize()
    default {
      // expected-remark @below {{scope id = 1}}
      // expected-remark @below {{scope parent id = 0}}
      proton.record start "default"
      // expected-remark @below {{scope id = 1}}
      // expected-remark @below {{scope parent id = 0}}
      proton.record end "default"
      ttg.warp_yield
    }
    partition0() num_warps(1) {
      // expected-remark @below {{scope id = 2}}
      // expected-remark @below {{scope parent id = 0}}
      proton.record start "partition"
      // expected-remark @below {{scope id = 2}}
      // expected-remark @below {{scope parent id = 0}}
      proton.record end "partition"
      ttg.warp_return
    } : () -> ()
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "outer"
    tt.return
  }
}
```
**EN:** This block defines `warp_specialize_balanced` and exercises operations such as `tt.func`, `proton.record`, `ttg.warp_specialize`, `ttg.warp_yield`, `ttg.warp_return`. Expected-diagnostic annotations (remark×13) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `warp_specialize_balanced`，并覆盖 如 `tt.func`、`proton.record`、`ttg.warp_specialize`、`ttg.warp_yield`、`ttg.warp_return` 这样的操作。 期望诊断标注（remark×13）用于捕获 pass 应当发出的分析备注/说明。

### Line 248
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 250-271
```mlir
module {
  // expected-remark @below {{cf_loop_closed}}
  tt.func @cf_loop_closed() {
  ^entry:
    %c0 = arith.constant 0 : index
    cf.br ^loop(%c0 : index)
  ^exit:
    tt.return
  ^loop(%iv: index):
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "loop_body"
    %c1 = arith.constant 1 : index
    %next = arith.addi %iv, %c1 : index
    %c2 = arith.constant 2 : index
    %cond = arith.cmpi ult, %next, %c2: index
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "loop_body"
    cf.cond_br %cond, ^loop(%next : index), ^exit
  }
}
```
**EN:** This block defines `cf_loop_closed` and exercises operations such as `tt.func`, `arith.constant`, `cf.br`, `tt.return`, `proton.record`. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `cf_loop_closed`，并覆盖 如 `tt.func`、`arith.constant`、`cf.br`、`tt.return`、`proton.record` 这样的操作。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Line 273
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 275-298
```mlir
module {
  // expected-remark @below {{cf_loop_closed_two_blocks}}
  tt.func @cf_loop_closed_two_blocks() {
  ^entry:
    %c0 = arith.constant 0 : index
    cf.br ^loop(%c0 : index)
  ^exit:
    tt.return
  ^loop(%iv: index):
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record start "loop_body"
    %c1 = arith.constant 1 : index
    %next = arith.addi %iv, %c1 : index
    cf.br ^loop_body(%next : index)
  ^loop_body(%iv_next: index):
    %c2 = arith.constant 2 : index
    %cond = arith.cmpi ult, %iv_next, %c2: index
    // expected-remark @below {{scope id = 0}}
    // expected-remark @below {{scope parent id = -1}}
    proton.record end "loop_body"
    cf.cond_br %cond, ^loop(%iv_next : index), ^exit
  }
}
```
**EN:** This block defines `cf_loop_closed_two_blocks` and exercises operations such as `tt.func`, `arith.constant`, `cf.br`, `tt.return`, `proton.record`. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `cf_loop_closed_two_blocks`，并覆盖 如 `tt.func`、`arith.constant`、`cf.br`、`tt.return`、`proton.record` 这样的操作。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Line 300
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 302-308
```mlir
module {
  tt.func @cf_unclosed() {
    // expected-error @below {{The scope name 'unclosed' is not properly closed (missing end record)}}
    proton.record start "unclosed"
    tt.return
  }
}
```
**EN:** This block defines `cf_unclosed` and exercises operations such as `tt.func`, `proton.record`, `tt.return`. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 `cf_unclosed`，并覆盖 如 `tt.func`、`proton.record`、`tt.return` 这样的操作。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

### Line 310
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 312-318
```mlir
module {
  tt.func @cf_dangling_end() {
    // expected-error @below {{The scope name 'dangling' is closed without being opened}}
    proton.record end "dangling"
    tt.return
  }
}
```
**EN:** This block defines `cf_dangling_end` and exercises operations such as `tt.func`, `proton.record`, `tt.return`. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 `cf_dangling_end`，并覆盖 如 `tt.func`、`proton.record`、`tt.return` 这样的操作。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

### Line 320
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 322-336
```mlir
module {
  tt.func @cf_liveness_error(%cond: i1) {
    proton.record start "name0"
    cf.cond_br %cond, ^then, ^else
  ^then:  // pred: ^entry
    proton.record end "name0"
    cf.br ^merge
  ^else:  // pred: ^entry
    // expected-error @below {{The scope name 'name0' is not properly closed (missing start record)}}
    proton.record end "name0"
    cf.br ^merge
  ^merge:  // preds: ^then, ^else
    tt.return
  }
}
```
**EN:** This block defines `cf_liveness_error` and exercises operations such as `tt.func`, `proton.record`, `cf.cond_br`, `cf.br`, `tt.return`. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 `cf_liveness_error`，并覆盖 如 `tt.func`、`proton.record`、`cf.cond_br`、`cf.br`、`tt.return` 这样的操作。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

### Line 338
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 340-353
```mlir
module {
  tt.func @cf_branch_unclosed_dangling(%cond: i1) {
    cf.cond_br %cond, ^then, ^else
  ^then:  // pred: ^entry
    proton.record start "ghost"
    cf.br ^merge
  ^else:  // pred: ^entry
    // expected-error @below {{The scope name 'ghost' is closed without being opened}}
    proton.record end "ghost"
    cf.br ^merge
  ^merge:  // preds: ^then, ^else
    tt.return
  }
}
```
**EN:** This block defines `cf_branch_unclosed_dangling` and exercises operations such as `tt.func`, `cf.cond_br`, `proton.record`, `cf.br`, `tt.return`. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 `cf_branch_unclosed_dangling`，并覆盖 如 `tt.func`、`cf.cond_br`、`proton.record`、`cf.br`、`tt.return` 这样的操作。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

### Line 355
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 357-373
```mlir
module {
  tt.func @cf_merge_unclosed(%cond: i1) {
    cf.br ^start(%cond : i1)
  ^start(%cond_arg: i1):
    proton.record start "ghost"
    cf.cond_br %cond_arg, ^then, ^else
  ^then:  // pred: ^start
    proton.record end "ghost"
    cf.br ^merge
  ^else:  // pred: ^start
    proton.record start "ghost"
    cf.br ^merge
  ^merge:  // preds: ^then, ^else
    proton.record end "ghost"
    tt.return
  }
}
```
**EN:** This block defines `cf_merge_unclosed` and exercises operations such as `tt.func`, `cf.br`, `proton.record`, `cf.cond_br`, `tt.return`.
**CN:** 这一块定义了 `cf_merge_unclosed`，并覆盖 如 `tt.func`、`cf.br`、`proton.record`、`cf.cond_br`、`tt.return` 这样的操作。

### Line 375
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 377-392
```mlir
module {
  tt.func @cf_loop_unclosed() {
    %c0 = arith.constant 0 : index
    cf.br ^loop(%c0 : index)
  ^exit:
    tt.return
  ^loop(%iv: index):
    // expected-error @below {{The scope name 'loop' is started without being closed}}
    proton.record start "loop"
    %c1 = arith.constant 1 : index
    %next = arith.addi %iv, %c1 : index
    %c2 = arith.constant 2 : index
    %cond = arith.cmpi ult, %next, %c2: index
    cf.cond_br %cond, ^loop(%next : index), ^exit
  }
}
```
**EN:** This block defines `cf_loop_unclosed` and exercises operations such as `tt.func`, `arith.constant`, `cf.br`, `tt.return`, `proton.record`. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 `cf_loop_unclosed`，并覆盖 如 `tt.func`、`arith.constant`、`cf.br`、`tt.return`、`proton.record` 这样的操作。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

### Line 394
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 396-412
```mlir
module {
  tt.func @cf_loop_end_before_start() {
    %c0 = arith.constant 0 : index
    cf.br ^loop(%c0 : index)
  ^exit:
    tt.return
  ^loop(%iv: index):
    // expected-error @below {{The scope name 'loop' has end record that dominates its start record}}
    proton.record end "loop"
    %c1 = arith.constant 1 : index
    %next = arith.addi %iv, %c1 : index
    %c2 = arith.constant 2 : index
    %cond = arith.cmpi ult, %next, %c2: index
    proton.record start "loop"
    cf.cond_br %cond, ^loop(%next : index), ^exit
  }
}
```
**EN:** This block defines `cf_loop_end_before_start` and exercises operations such as `tt.func`, `arith.constant`, `cf.br`, `tt.return`, `proton.record`. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 `cf_loop_end_before_start`，并覆盖 如 `tt.func`、`arith.constant`、`cf.br`、`tt.return`、`proton.record` 这样的操作。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's Proton pipeline coverage and focuses on scope-id behavior.  
  **CN:** 该文件属于 Triton 的 Proton 流水线 测试覆盖，关注点是 scope-id 行为。
- **EN:** The `RUN` pipeline drives tools/passes such as `--split-input-file`, `--test-print-scope-id-allocation`, `-verify-diagnostics=only-expected`.  
  **CN:** `RUN` 流水线会驱动 `--split-input-file`, `--test-print-scope-id-allocation`, `-verify-diagnostics=only-expected` 等工具/pass。
- **EN:** The IR mixes `tt` (Triton core ops), `proton` (Proton dialect ops), `cf` (control-flow ops), `scf` (structured control flow), `arith` (scalar/tensor arithmetic), `ttg` (TritonGPU ops/layouts).  
  **CN:** 该 IR 混合使用了 `tt`（Triton 核心操作）、`proton`（Proton 方言操作）、`cf`（控制流操作）、`scf`（结构化控制流）、`arith`（标量/张量算术）、`ttg`（TritonGPU 操作/布局）。
- **EN:** Expected-diagnostic annotations make negative tests precise and reproducible.  
  **CN:** 期望诊断标注让负向测试更精确、可复现。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`.  
  **CN:** 外部测试工具：`triton-opt`。
- **EN:** IR dialect dependencies: `tt`, `proton`, `cf`, `scf`, `arith`, `ttg`.  
  **CN:** IR 方言依赖：`tt`、`proton`、`cf`、`scf`、`arith`、`ttg`。
