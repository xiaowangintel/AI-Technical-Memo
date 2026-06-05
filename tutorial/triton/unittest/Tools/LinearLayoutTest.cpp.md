# LinearLayoutTest.cpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `unittest/Tools/LinearLayoutTest.cpp`
- **Purpose / 目的:** Exercises the `LinearLayout` utility layer: construction, multiplication/composition, reshaping, inversion, quotient/division helpers, ordering utilities, and value-permutation helpers used by Triton layout lowering. / 覆盖 `LinearLayout` 工具层的核心行为：构造、乘法与组合、reshape、求逆、商与除法辅助函数、顺序工具，以及 Triton 布局降级使用的值重排辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-33
**EN:** Includes `LinearLayout` and `LayoutUtils`, defines `operator<<` for `StringAttr` so gtest output is readable, aliases common gmock matchers, and builds the `LinearLayoutTest` fixture with an `MLIRContext` plus helper `S()` for creating dimension names.

**CN:** 引入 `LinearLayout` 与 `LayoutUtils`，为 `StringAttr` 定义 `operator<<` 以便 gtest 输出可读，设置常用 gmock 别名，并建立 `LinearLayoutTest` 测试夹具：内部持有 `MLIRContext`，通过 `S()` 快速创建维度名。

### Lines 34-142
**EN:** Validates the basic algebra of `LinearLayout`. `Empty`, `Identity1D`, `Identity1DSize1`, and `Zeros1D` check canonical constructors. `Multiply*`, `TimesEquals`, and `GetOutDimSizeLog2` confirm that multiplying layouts concatenates/merges bases correctly for identical, disjoint, overlapping, degenerate, and zero layouts, while preserving discovered input/output dimension metadata.

**CN:** 验证 `LinearLayout` 的基础代数语义。`Empty`、`Identity1D`、`Identity1DSize1`、`Zeros1D` 检查标准构造器；`Multiply*`、`TimesEquals`、`GetOutDimSizeLog2` 则确认布局相乘时，针对相同、互不相交、重叠、退化和全零布局，base 向量会被正确拼接或合并，同时保留输入/输出维度信息。

### Lines 143-218
**EN:** Covers dimension reordering and direct evaluation. `TransposeOuts`/`TransposeIns` reorder basis coordinates without changing the represented mapping; degenerate variants ensure size-1 dimensions survive. `EmptyToString` guards printable formatting. `Apply` evaluates a non-surjective 2D mapping on concrete inputs. `ConstructLargeLayout` is effectively a smoke/perf test for building many power-of-two bases.

**CN:** 覆盖维度重排和直接求值。`TransposeOuts`/`TransposeIns` 在不改变映射语义的前提下重排 basis 坐标，退化版本保证 size-1 维度也能正确处理。`EmptyToString` 防止打印格式崩溃；`Apply` 在具体输入上验证一个非满射 2D 映射；`ConstructLargeLayout` 更像构造大量 2 的幂基向量时的烟雾/性能测试。

### Lines 219-370
**EN:** Tests structural transforms. `Compose` and `Compose4D` verify matrix-style composition, including non-surjective outputs and dropped zero rows. `ReshapeIns`/`ReshapeOuts` and `FlattenIns`/`FlattenOuts` move basis bits across dimension boundaries while handling degenerate input/output factors. These cases show that `LinearLayout` treats dimensions as factorizations of bit positions rather than fixed physical axes.

**CN:** 测试结构性变换。`Compose` 与 `Compose4D` 验证类似矩阵复合的行为，包括非满射输出以及全零行被保留/折叠的情况。`ReshapeIns`/`ReshapeOuts` 和 `FlattenIns`/`FlattenOuts` 则把 basis 位在不同维度边界之间重新分配，同时正确处理退化输入或输出因子。这说明 `LinearLayout` 把维度看作“位分解方式”，而不是固定物理轴。

### Lines 371-605
**EN:** Focuses on inversion and conversion composition. The tests move from simple permutations to larger real-world register/shared-memory layouts, then to pseudo-inverse behavior for non-injective or broadcasted layouts. `InvertAndCompose_*` checks that `A.invertAndCompose(B)` builds a layout `C` such that `C.compose(B)` recreates `A` (sometimes only up to out-dim sizes). The long `IdentityInDim` case uses 9 output dims and verifies equality by concrete application instead of simple structural comparison.

**CN:** 聚焦求逆和转换复合。测试从简单置换逐步扩展到更真实的寄存器/共享内存布局，再到非单射或带广播布局的伪逆行为。`InvertAndCompose_*` 核心在于验证 `A.invertAndCompose(B)` 生成的布局 `C` 满足 `C.compose(B)` 能重建 `A`（有时只需忽略输出维度大小）。较长的 `IdentityInDim` 用 9 个输出维度和实际求值来验证，而不是只做结构相等比较。

### Lines 606-717
**EN:** Exercises layout introspection helpers. `NumConsecutiveInOut` measures how many low-order coordinates remain consecutively addressable. `EqualsChecksOutDimSizes` distinguishes exact equality from equality ignoring explicit output sizes. `Sublayout` and `sublayoutIsZero` project onto chosen input/output subsets and detect whether the projected mapping is all-zero. `FreeVariableMasks` computes which input bits are unconstrained by the output mapping.

**CN:** 覆盖布局自省辅助函数。`NumConsecutiveInOut` 测量低位坐标中还能保持连续寻址的范围；`EqualsChecksOutDimSizes` 区分“完全相等”和“忽略显式输出大小后的相等”；`Sublayout`/`sublayoutIsZero` 对指定输入输出子集做投影并判断投影是否全零；`FreeVariableMasks` 计算哪些输入位没有被输出映射约束。

### Lines 718-814
**EN:** Tests quotient-style factorization. `Quotient*` confirms quotienting only succeeds when the chosen dimensions are trivial/independent in the algebraic sense. The helper `getPackedCoordtoPaddedOffset` starts a Blackwell-specific section by manually building a packed-to-padded address mapping used as a ground truth for pseudoinverse checks.

**CN:** 测试商（quotient）式分解。`Quotient*` 说明只有当被约去的维度在代数上足够“平凡/独立”时，求商才成立。随后 `getPackedCoordtoPaddedOffset` 进入 Blackwell 专题：它手工构造 packed 坐标到 padded offset 的映射，用来作为后续伪逆验证的基准答案。

### Lines 815-897
**EN:** Verifies Blackwell mixed-precision scaled-dot shared-memory addressing. The first test compares a computed pseudoinverse against a hand-derived packed-to-padded offset conversion. The swizzled variant adds XOR-style column swizzling and checks that the computed inverse matches the analytically expected swizzled address.

**CN:** 验证 Blackwell 混合精度 scaled-dot 的共享内存寻址。第一个测试把自动求出的伪逆与手工推导的 packed→padded offset 转换进行对比；第二个测试加入按行相位变化的列 swizzle，并确认求出的逆映射与解析式计算出的 swizzled 地址一致。

### Lines 898-954
**EN:** Introduces `makeList` plus a separate `SupremumTest` cluster for the `supremum` ordering utility. These tests cover identical orders, non-unique but valid merges, different lengths, empty lists, and (under assertions) an inconsistent order that must crash.

**CN:** 这里通过 `makeList` 和单独的 `SupremumTest` 测试 `supremum` 顺序合并工具。它覆盖完全相同的顺序、非唯一但合法的合并、不同长度、空列表，以及在开启断言时必须触发失败的矛盾顺序。

### Lines 955-1094
**EN:** Covers left/right layout division. `Divide_Basic`, `Divide_Simple`, and `Divide_2D` show successful recovery of missing factors when `A = B * C`. Failure and elimination cases check that division rejects extra dimensions and can still produce degenerate factors that remove redundant input or output dimensions.

**CN:** 覆盖左右除法。`Divide_Basic`、`Divide_Simple`、`Divide_2D` 展示在 `A = B * C` 时如何恢复缺失因子。失败案例与消除案例说明：若维度不匹配，除法应拒绝；但若只是冗余输入/输出维度，则仍可得到一个退化因子来完成分解。

### Lines 1095-1166
**EN:** Tests `ColumnAction`, a helper that permutes or drops basis columns. One test applies it to a `LinearLayout` and checks the reordered bases; the other applies it to a vector of dummy `mlir::Value`s, proving that the same column action induces the expected value permutation/subsampling.

**CN:** 测试 `ColumnAction`，它用于重排或丢弃 basis 列。一个测试直接作用于 `LinearLayout` 并检查 basis 的新顺序；另一个测试作用于伪造的 `mlir::Value` 数组，证明相同的列操作也会诱导出预期的值重排/抽样效果。

### Lines 1167-1209
**EN:** `invertAndCompose1` is a regression-style test for register-to-shared conversion: it builds explicit register and shared layouts and ensures the resulting conversion is trivial over the `block` dimension, i.e. block handling remains identity even if sublayout out-dim sizes differ. The file ends with the standard gtest `main`.

**CN:** `invertAndCompose1` 是一个偏回归性质的测试：它手工构造寄存器布局和共享内存布局，并验证得到的转换在 `block` 维度上是平凡的，也就是即使子布局的输出维度大小不同，block 维处理仍保持恒等。文件最后是标准的 gtest `main`。

## Key Concepts / 关键概念
- **Linear layout algebra / 线性布局代数:** layouts are represented as basis vectors from named input dimensions to named output dimensions.
- **Surjective vs. non-surjective mappings / 满射与非满射映射:** many tests intentionally keep explicit output sizes even when some outputs are unreachable.
- **Pseudo-inverse composition / 伪逆复合:** essential for converting between register, shared-memory, and packed representations.
- **Dimension factoring / 维度因子化:** reshape/flatten/divide/quotient treat dimensions as redistributions of bit significance.
- **Ordering utilities / 顺序工具:** `supremum` and `ColumnAction` support stable dimension ordering and value reindexing.

## Dependencies / 依赖关系
- `triton/Tools/LinearLayout.h`: core `LinearLayout`, division helpers, quotient, pseudoinvert, and `ColumnAction` APIs.
- `triton/Tools/LayoutUtils.h`: utilities such as `ensureLayoutNotSmallerThan` and `supremum` used in later tests.
- MLIR core (`MLIRContext`, `StringAttr`, `Value`): provides symbolic dimension names and opaque test values.
- gtest/gmock: structural equality, death tests, and matcher-based checks.
