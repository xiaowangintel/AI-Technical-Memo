# 51_hopper_gett.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/51_hopper_gett/51_hopper_gett.cu`  
**Purpose / 用途**: Shows how a generalized tensor-times-tensor contraction (GETT) can be expressed as a Hopper CUTLASS 3 kernel simply by replacing ordinary GEMM strides with CuTe multi-mode strides and then launching a GETT helper. / 展示如何仅通过把普通 GEMM stride 替换成 CuTe 的多模态 stride，就把 generalized tensor-times-tensor contraction（GETT）表达成一个 Hopper CUTLASS 3 内核，并通过 GETT helper 发起执行。

---

## Line-by-Line Analysis / 逐行分析

### Block 1 (Lines 1-133)
```cpp
/*! \file
  \brief Example of a GETT targeting Hopper tensor cores using the CUTLASS 3.x API.
  ...
  The modes of a tensor that participate in a GETT can be fundamentally grouped into
  four semantic categories ... M, N, K, L ...
*/

namespace example {

template<class Stride>
constexpr bool is_left_major() { ... }

template <int Rank, bool IsMajor, class Indexable>
static constexpr auto make_stride_tuple(Indexable const& t, int n, int64_t init_default = 0) { ... }
```
**EN**: The prologue gives the conceptual model for the whole file: a tensor contraction can be folded into four semantic mode groups—row (`M`), column (`N`), reduction (`K`), and batch (`L`)—so it can be handled like a rank-4 GEMM problem whose components are themselves tuples. The helper `is_left_major()` checks whether a compile-time stride tuple begins with static `1`, and `make_stride_tuple()` injects that `Int<1>` into runtime data when a mode family is declared left-major. Those two utilities are the bridge between command-line mode descriptions and the static layout assumptions required by the Hopper collective.

**CN**: 文件开头先给出了整份示例的概念模型：一个张量收缩可以折叠成四类语义模式——行模式（`M`）、列模式（`N`）、规约模式（`K`）、批模式（`L`）——于是它就能像 rank-4 GEMM 那样处理，只不过每个分量本身仍是一个 tuple。辅助函数 `is_left_major()` 用来判断某个编译期 stride tuple 是否以静态 `1` 开头；`make_stride_tuple()` 则在某个模式家族被声明为 left-major 时，把这个 `Int<1>` 注入运行时数据。它们共同搭起了“命令行模式描述”与“Hopper collective 所需的静态布局假设”之间的桥梁。

### Block 2 (Lines 137-227)
```cpp
using RowModeStridesA = cute::Stride<cute::Int<1>, int64_t, int64_t, int64_t>;
using RedModeStridesA = cute::Stride<int64_t, int64_t, int64_t>;
using BatModeStridesA = cute::Stride<int64_t, int64_t, int64_t, int64_t>;
...
using ColModeStridesB = cute::Stride<int64_t, int64_t, int64_t, int64_t>;
using RedModeStridesB = cute::Stride<cute::Int<1>, int64_t, int64_t>;
...
using StrideA = cute::Stride<RowModeStridesA, RedModeStridesA, BatModeStridesA>;
using StrideB = cute::Stride<ColModeStridesB, RedModeStridesB, BatModeStridesB>;
using StrideC = cute::Stride<RowModeStridesC, ColModeStridesC, BatModeStridesC>;
```
**EN**: This section hard-wires one precise GETT contraction family. `A` is major in its first row mode (`M0`) because `RowModeStridesA` begins with `Int<1>`. `B` is major in its first reduction mode (`K0`) because `RedModeStridesB` begins with `Int<1>`. The output strides are fully dynamic. Nesting these per-family strides into `StrideA/B/C` is the key GETT trick: each operand still looks like a GEMM operand to CUTLASS, but each of the GEMM dimensions now internally contains multiple tensor modes.

**CN**: 这一段把示例锁定到一个非常具体的 GETT 收缩家族。由于 `RowModeStridesA` 以 `Int<1>` 开头，`A` 在第一个行模式（`M0`）上是主序；由于 `RedModeStridesB` 以 `Int<1>` 开头，`B` 在第一个规约模式（`K0`）上是主序；输出张量的 stride 则完全动态。把这些“按模式家族划分的 stride”再嵌套成 `StrideA/B/C`，正是 GETT 的核心技巧：对 CUTLASS 来说，每个操作数仍然像 GEMM 操作数；但在 GEMM 的每个维度内部，实际上封装了多个张量模式。

### Block 3 (Lines 199-277)
```cpp
auto& m = parsed_args.M;  ... int rank_m = int(m.size());
auto& n = parsed_args.N;  ... int rank_n = int(n.size());
auto& k = parsed_args.K;  ... int rank_k = int(k.size());
auto& l = parsed_args.L;  ... int rank_l = int(l.size());
...
auto M   = make_int_tuple<MaxRank_M>(m.data(), rank_m, 1);
auto dAm = example::make_stride_tuple<MaxRank_M, example::is_left_major<RowModeStridesA>()>(ldAm.data(), rank_m);
...
auto problem_shape = make_shape(M, N, K, L);
StrideA stride_A   = make_stride(dAm, dAk, dAl);
```
**EN**: After parsing the command line, the file converts arbitrary user-specified mode lists into fixed-rank CuTe tuples. Missing extents default to `1`, and missing batch strides default to `0`, which matches broadcast-like semantics. The validation before tuple assembly is important: if a stride family is statically declared left-major, the first runtime leading dimension must really be `1`. Once that is confirmed, `problem_shape = make_shape(M, N, K, L)` produces the nested rank-4 GETT problem descriptor that the kernel launch expects.

**CN**: 命令行解析完成后，文件会把用户任意给出的模式列表转换成固定 rank 的 CuTe tuple。缺失的 extent 默认补成 `1`，而缺失的 batch stride 默认补成 `0`，这与广播式语义相匹配。tuple 组装前的校验非常关键：如果某个 stride 家族在静态类型上声明为 left-major，那么运行时给出的第一个 leading dimension 也必须真的等于 `1`。确认无误后，`problem_shape = make_shape(M, N, K, L)` 就得到内核 launch 所需的嵌套 rank-4 GETT 问题描述符。

### Block 4 (Lines 281-326)
```cpp
thrust::host_vector<ElementA> h_A(M_size * K_size * L_size);
...
auto status = example::gett_kernel(
  problem_shape,
  d_A.data().get(), stride_A,
  d_B.data().get(), stride_B,
  ElementAccumulator{},
  d_C.data().get(), stride_C,
  cutlass_result.data().get(), stride_D,
  alpha, beta);
```
**EN**: Once the nested shape and nested strides have been constructed, the rest looks intentionally ordinary. The code allocates dense flat storage sized by the products of all M/N/K/L submodes, fills it with small random values, copies to the device, and launches `example::gett_kernel`. The point is that GETT does not require a separate user-facing execution model here; the only difference from GEMM is the richer shape/stride metadata.

**CN**: 一旦嵌套 shape 和嵌套 stride 都构造好，后面的代码就刻意保持得“很像普通 GEMM”。它根据所有 M/N/K/L 子模式的乘积为扁平存储分配空间，用小范围随机数初始化，复制到设备上，然后调用 `example::gett_kernel`。也就是说，这里的 GETT 并不需要另一套完全不同的用户执行模型；和 GEMM 的核心差别只在于更丰富的 shape/stride 元数据。

### Block 5 (Lines 328-371)
```cpp
cutlass::reference::device::gett(
  problem_shape,
  d_A.data().get(), stride_A,
  d_B.data().get(), stride_B,
  ElementAccumulator{},
  d_C.data().get(), stride_C,
  reference_result.data().get(), stride_D,
  alpha, beta);

bool passed = cutlass::reference::device::BlockCompareEqual(...);
```
**EN**: Verification mirrors execution exactly: the same nested shape and multi-mode strides are fed into CUTLASS’s device reference GETT. A blockwise equality check is then performed. If it fails, the file prints relative-error diagnostics plus the fully assembled multi-mode strides, which is helpful because GETT bugs are often layout bugs rather than arithmetic bugs.

**CN**: 校验路径与执行路径几乎完全镜像：相同的嵌套 shape 和 multi-mode stride 会传给 CUTLASS 的设备端参考 GETT。随后再执行逐块相等性比较。若失败，程序会打印相对误差诊断以及完整拼装出来的 multi-mode stride；这很有价值，因为 GETT 的问题往往首先是布局错误，而不一定是算术错误。

---

## Key Concepts / 关键概念

- **Tensor contraction as GEMM-shaped metadata / 用 GEMM 形状元数据表达张量收缩**: GETT keeps the GEMM execution skeleton but enriches each MNKL dimension into a tuple of modes. / GETT 保留 GEMM 的执行骨架，但把每个 MNKL 维扩展成模式 tuple。
- **Static major-mode encoding / 静态主序编码**: A compile-time `Int<1>` inside a stride tuple declares which mode family is major. / stride tuple 中的编译期 `Int<1>` 用来声明哪个模式家族是主序。
- **Runtime parsing to fixed-rank tuples / 运行时输入转固定 rank tuple**: The command line remains flexible while the kernel still gets statically structured layouts. / 命令行输入保持灵活，但内核仍能拿到静态结构化布局。

## Dependencies / 依赖项

- `gett_kernel.cuh` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `thrust/host_vector.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `thrust/device_vector.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/cutlass.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/gemm/device/gemm_universal_adapter.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/gemm/kernel/gemm_universal.hpp` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/gemm/collective/collective_builder.hpp` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/epilogue/collective/default_epilogue.hpp` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/util/gett_commandline.hpp` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/util/reference/device/gett.hpp` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/util/reference/device/tensor_compare.h` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
- `cutlass/util/print_error.hpp` — Included header used by this example or helper. / 此示例或辅助实现使用的头文件。
