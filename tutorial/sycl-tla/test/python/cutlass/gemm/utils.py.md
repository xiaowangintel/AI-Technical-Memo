# utils.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/python/cutlass/gemm/utils.py`
- **Purpose (EN):** Supplies shared layout aliases, procedural test-name generation, and the dynamic GEMM test factory used by the architecture-specific suites.
- **用途 (CN):** 提供架构测试套件共享的布局别名、过程化测试命名逻辑，以及动态 GEMM 测试工厂函数。

## Line-by-Line Analysis / 逐行分析

### Lines 33-56 / 第 33-56 行

```python
from cutlass_library import SubstituteTemplate
from cutlass_library.arch_constants import (
    INTEL_XE_ARCH_MIN, 
    INTEL_XE_ARCH_MAX, 
    INTEL_XE12, 
    INTEL_XE20, 
    INTEL_XE35,
    is_intel_xe_arch
)

import cutlass_cppgen
from cutlass_library import (
    DataTypeNames,
    EpilogueScheduleSuffixes,
    KernelScheduleSuffixes,
    LayoutType,
    OpcodeClassNames,
    ShortDataTypeNames,
    ShortLayoutTypeNames
)
from cutlass_cppgen.backend import library

from gemm_testbed import test_all_gemm

```

**EN:** The file begins by importing naming/template helpers, Intel Xe architecture constants, CUTLASS datatype/layout naming tables, the backend library package, and the shared `test_all_gemm` harness.
This mix of imports shows that `utils.py` is the glue layer between high-level declarative test registration and the lower-level runtime testbed.

**CN:** 文件开头导入了命名模板工具、Intel Xe 架构常量、CUTLASS 的数据类型/布局命名字典、后端库包，以及共享的 `test_all_gemm` 测试床。
这种导入组合说明 `utils.py` 扮演的是“高层声明式测试注册”和“底层运行时测试床”之间的胶水层角色。

### Lines 58-79 / 第 58-79 行

```python
class Layout:
    """
    Utility class to map transpose and non-transpose terminology to row- and column-major terminology
    """

    T = LayoutType.RowMajor
    N = LayoutType.ColumnMajor


class LayoutCombination:
    """
    Utility class defining all combinations of row- and column-major layouts for operands to a GEMMs
    """

    NNN = (Layout.N, Layout.N, Layout.N)
    NNT = (Layout.N, Layout.N, Layout.T)
    NTN = (Layout.N, Layout.T, Layout.N)
    NTT = (Layout.N, Layout.T, Layout.T)
    TNN = (Layout.T, Layout.N, Layout.N)
    TNT = (Layout.T, Layout.N, Layout.T)
    TTN = (Layout.T, Layout.T, Layout.N)
    TTT = (Layout.T, Layout.T, Layout.T)
```

**EN:** `Layout` is a tiny adapter class that maps the GEMM notation used in these tests (`T` and `N`) onto CUTLASS row-major and column-major layout enums.
`LayoutCombination` then enumerates all eight three-operand combinations for A, B, and C/D layouts.
These symbolic tuples let the suite files express cases like `TNT` or `TNN` concisely without rewriting enum triplets every time.

**CN:** `Layout` 是一个很小的适配器类，用来把本测试目录常用的 GEMM 记号（`T` 与 `N`）映射到 CUTLASS 的行主/列主布局枚举。
`LayoutCombination` 在其基础上列举了 A、B、C/D 三个操作数布局的全部八种组合。
因此像 `TNT`、`TNN` 这样的符号元组就可以直接在测试文件中复用，而不必反复手写三元枚举值。

### Lines 82-128 / 第 82-128 行

```python
def get_name(
    layouts,
    alignments,
    element_output,
    element_accumulator,
    element_epilogue,
    cluster_shape,
    threadblock_shape,
    stages,
    element_a,
    element_b,
    element_c,
    arch,
    opclass,
    kernel_schedule=None,
    epilogue_schedule=None,
    suffix="",
):
    """
    Generates a procedural name for a test case.

    :param layouts: indexable container of layouts of A, B, and C operands
    :param alignments: indexable container of alignments of A, B, and C operands
    :param element_output: data type of the output element
    :param element_accumulator: data type used in accumulation
    :param element_epilogue: data type used in computing the epilogue
    :param cluster_shape: indexable container of dimensions of threadblock cluster to be launched
    :param threadblock_shape: indexable container of dimensions of threadblock tiles
    :param stages: number of pipeline stages to use in the kernel
    :type stages: int
    :param element_a: data type of operand A
    :param element_b: data type of operand B
    :param element_c: data type of operand C
    :param arch: compute capability of kernel being generated
    :type arch: int
    :param opclass: class of operation being performed (e.g., SIMT, Tensor Core)
    :type opclass: cutlass_cppgen.OpcodeClass
    :param kernel_schedule: kernel_schedule type
    :type kernel_schedule: cutlass_cppgen.KernelScheduleType
    :param epilogue_schedule: epilogue_schedule type
    :type epilogue_schedule: cutlass_cppgen.EpilogueScheduleType
    :param suffix: additional string to add to the suffix of the name
    :type suffix: str

    :return: str
    """
    name_format = "test_${arch}_Device_Gemm_${eA}${lA}_${eB}${lB}_${eC}${lC}_${opclass}_${acc}_${tbM}x${tbN}x${tbK}_${cM}x${cN}x${cK}_${stages}_align${aA}-${aB}-${aC}${k}${e}${suffix}"
```

**EN:** `get_name(...)` centralizes how generated unittest methods are named.
Its signature accepts layout, alignment, datatype, architecture, schedule, and shape information, and the docstring documents each parameter carefully because the function is a public helper for many test files.
The `name_format` template encodes the entire configuration into a procedural string such as architecture, operand datatypes/layouts, opcode class, accumulator type, tile sizes, cluster shape, stage count, and alignments.

**CN:** `get_name(...)` 统一管理动态生成的 unittest 方法应如何命名。
它的参数签名包含布局、对齐、数据类型、架构、调度方式和 tile 形状等信息；由于这是多个测试文件共用的公开辅助函数，文档字符串对每个参数都做了较详细说明。
`name_format` 模板会把整个配置编码成过程化字符串，包括架构、操作数类型/布局、opcode 类、累加器类型、tile 尺寸、cluster 形状、阶段数以及对齐信息。

### Lines 130-162 / 第 130-162 行

```python
    # Map Intel Xe architectures to names
    if is_intel_xe_arch(arch):
        arch_name = f"Xe{str(arch)}"  # Generic Xe naming
    else:
        arch_name = f"SM{str(arch)}"  # NVIDIA SM naming
    
    return SubstituteTemplate(
        name_format,
        {
            "arch": arch_name,
            "eA": DataTypeNames[element_a],
            "eB": DataTypeNames[element_b],
            "eC": DataTypeNames[element_c],
            "lA": ShortLayoutTypeNames[layouts[0]],
            "lB": ShortLayoutTypeNames[layouts[1]],
            "lC": ShortLayoutTypeNames[layouts[2]],
            "opclass": OpcodeClassNames[opclass],
            "acc": DataTypeNames[element_accumulator],
            "cM": str(cluster_shape[0]),
            "cN": str(cluster_shape[1]),
            "cK": str(cluster_shape[2]),
            "tbM": str(threadblock_shape[0]),
            "tbN": str(threadblock_shape[1]),
            "tbK": str(threadblock_shape[2]),
            "stages": str(stages) if stages is not None else "auto",
            "aA": str(alignments[0]),
            "aB": str(alignments[1]),
            "aC": str(alignments[2]),
            "k": "" if kernel_schedule is None else KernelScheduleSuffixes[kernel_schedule],
            "e": "" if epilogue_schedule is None else EpilogueScheduleSuffixes[epilogue_schedule],
            "suffix": "" if suffix is None else suffix,
        },
    )
```

**EN:** The function distinguishes Intel Xe from NVIDIA architectures: Xe uses a `Xe{arch}` prefix, while NVIDIA uses `SM{arch}`.
`SubstituteTemplate(...)` then fills the template from CUTLASS naming tables such as `DataTypeNames`, `ShortLayoutTypeNames`, `OpcodeClassNames`, and the schedule suffix maps.
If stages or optional suffixes are absent, the function substitutes user-friendly defaults like `auto` or the empty string.

**CN:** 该函数会区分 Intel Xe 与 NVIDIA 架构：Xe 采用 `Xe{arch}` 前缀，NVIDIA 则使用 `SM{arch}`。
随后通过 `SubstituteTemplate(...)` 把 `DataTypeNames`、`ShortLayoutTypeNames`、`OpcodeClassNames` 以及 schedule 后缀映射等 CUTLASS 命名字典填充进模板。
如果 `stages` 或可选后缀为空，则会自动替换成 `auto` 或空字符串等更适合阅读的默认值。

### Lines 165-237 / 第 165-237 行

```python
def add_test_gemm(
    cls=None,
    cc=None,
    element=None,
    layouts=None,
    alignments=None,
    element_output=None,
    element_accumulator=None,
    cluster_shape=None,
    threadblock_shape=None,
    warp_count=None,
    stages=None,
    opclass=None,
    swizzle=None,
    kernel_schedule=None,
    epilogue_schedule=None,
    compilation_modes=['nvcc', 'nvrtc'],
    element_A=None,
    element_B=None,
    element_C=None):
    """
    Create test-running functions with the given specification and set it as a method of ``cls``.

    :param cls: class to which the generated method will be added
    :type cls: type
    :param cc: compute capability to compile for
    :type cc: int
    :param element: data type of A and B operands
    :type element: cutlass_cppgen.DataType.f16
    :param layouts: layouts of A, B, and C operands
    :type layouts: list or tuple
    :param alignments: alingments of A, B, and C operands
    :type alignments: list or tuple
    :param element_output: data type of the output element
    :type element_output: cutlass_cppgen.DataType
    :param element_accumulator: data type used in accumulation
    :type element_accumulator: cutlass_cppgen.DataType
    :param cluster_shape: dimensions of clusters
    :type cluster_shape: list or tuple
    :param threadblock_shape: dimensions of threadblock tiles
    :type threadblock_shape: list or tuple
    :param warp_count: warps to be launched per threadblock dimension
    :type warp_count: list or tuple
    :param stages: number of pipeline stages to use in the kernel
    :type stages: int
    :param opclass: class of operation being performed (e.g., SIMT, Tensor Core)
    :type opclass: cutlass_cppgen.OpcodeClass
    :param swizzle: threadblock swizzling functor
    :param kernel_schedule: kernel schedule to use
    :type kernel_schedule: cutlass_cppgen.KernelScheduleType
    :param epilogue_schedule: epilogue schedule to use
    :type epilogue_schedule: cutlass_cppgen.EpilogueScheduleType
    :param compilation_modes: list of compilers to used in testing the kernel (options: 'nvrtc', 'nvcc')
    :type compilation_modes: list,
    :param element_A: data type of operand A. If set, overrides ``element``
    :type element_A: cutlass_cppgen.DataType
    :param element_B: data type of operand B. If set, overrides ``element``
    :type element_B: cutlass_cppgen.DataType
    :param element_C: data type of operand C. If set, overrides ``element``
    :type element_C: cutlass_cppgen.DataType
    """

    if element_A is None:
        element_A = element
    if element_B is None:
        element_B = element
    if element_C is None:
        element_C = element
    if element_output is None:
        element_output = element
    if element_accumulator is None:
        element_accumulator = element

```

**EN:** `add_test_gemm(...)` is the real factory used by the suite files.
Its long signature exposes every major kernel knob: architecture, datatypes, layouts, alignments, tile shapes, warp counts, opcode class, swizzle, schedules, compilation modes, and per-operand datatype overrides.
The first body block normalizes defaults: if `element_A/B/C`, `element_output`, or `element_accumulator` are omitted, they fall back to the shared `element` type.
The outer `for compilation_mode in compilation_modes:` loop means one declarative registration can materialize multiple unittest methods, one per compiler backend.

**CN:** `add_test_gemm(...)` 是各套件文件真正依赖的测试工厂函数。
它通过很长的参数列表暴露了几乎所有主要内核旋钮：架构、数据类型、布局、对齐、tile 形状、warp 数、opcode 类、swizzle、调度方式、编译模式，以及按操作数覆盖的数据类型。
函数体前半段先做默认值归一化：如果没有单独给出 `element_A/B/C`、`element_output` 或 `element_accumulator`，它们都会回退到公共的 `element` 类型。
外层 `for compilation_mode in compilation_modes:` 循环意味着一次声明式注册可以生成多个 unittest 方法——每个编译后端一个方法。

### Lines 238-274 / 第 238-274 行

```python
    for compilation_mode in compilation_modes:
        def run(self):
            """
            Dynamically-generated function that constructs a GEMM operation and verifies it against
            multiple test cases.
            """

            layout_A, layout_B, layout_C = layouts
            alignment_A, alignment_B, alignment_C = alignments

            plan = cutlass_cppgen.op.Gemm(element_A=element_A, element_B=element_B,
                                element_C=element_C, element_D=element_output,
                                layout_A=layout_A, layout_B=layout_B, layout_C=layout_C,
                                element_accumulator=element_accumulator,
                                kernel_cc=cc)

            plan.opclass = opclass
            if swizzle is not None:
                plan.swizzling_functor = swizzle

            td = plan.tile_descriptions()[0]

            if warp_count is not None:
                td.warp_count = warp_count
            td.threadblock_shape = threadblock_shape
            td.stages = stages
            td.cluster_shape = cluster_shape
            
            # For Intel Xe architectures, ensure we use auto schedules and default tile scheduler
            if is_intel_xe_arch(cc):
                td.kernel_schedule = cutlass_cppgen.KernelScheduleType.ScheduleAuto
                td.epilogue_schedule = cutlass_cppgen.EpilogueScheduleType.ScheduleAuto
                td.tile_scheduler = cutlass_cppgen.TileSchedulerType.Default
            
            op = plan.construct(tile_description=td, alignment_A=alignment_A, alignment_B=alignment_B, alignment_C=alignment_C)
            self.assertTrue(test_all_gemm(op, 'universal', compilation_mode=compilation_mode))

```

**EN:** Inside that loop, `run(self)` becomes the dynamically attached unittest method.
It unpacks layout and alignment tuples, builds a `cutlass_cppgen.op.Gemm` plan with the requested operand and accumulator types, then overrides execution attributes such as opcode class and swizzling functor.
The method edits the first returned tile description in place: optional warp count, explicit threadblock shape, stage count, and cluster shape all come from the caller.
For Intel Xe architectures, it forcibly switches to automatic kernel/epilogue schedules and the default tile scheduler, which is a cross-architecture compatibility policy embedded directly in the helper.
Finally, it constructs the concrete operation and asserts that `test_all_gemm(...)` returns true for the `universal` testcase under the selected compilation mode.

**CN:** 在这个循环内部定义的 `run(self)` 就是最终会挂载到 unittest 类上的动态测试方法。
它先解包布局和对齐元组，再根据请求的操作数类型与累加器类型构造 `cutlass_cppgen.op.Gemm` plan，然后覆盖 opcode 类和 swizzle functor 等执行属性。
接着代码会直接修改返回的第一个 tile description：可选的 warp 数、明确的 threadblock 形状、阶段数以及 cluster 形状都由调用者提供。
对于 Intel Xe 架构，它会强制使用自动 kernel/epilogue schedule 和默认 tile scheduler，这是一条直接嵌入到辅助函数中的跨架构兼容策略。
最后函数构造出具体 operation，并断言在所选编译模式下执行 `test_all_gemm(...)` 的 `universal` 测试必须返回 true。

### Lines 275-282 / 第 275-282 行

```python
        element_epilogue = element_accumulator
        name = get_name(
            layouts=layouts, alignments=alignments, element_output=element_output, element_accumulator=element_accumulator,
            element_epilogue=element_epilogue, cluster_shape=cluster_shape, threadblock_shape=threadblock_shape,
            stages=stages, element_a=element_A, element_b=element_B, element_c=element_C, arch=cc, opclass=opclass,
            kernel_schedule=kernel_schedule, epilogue_schedule=epilogue_schedule, suffix=f'_{compilation_mode}')

        setattr(cls, name, run)
```

**EN:** After defining `run`, the helper computes a deterministic test name by calling `get_name(...)` and appending the compilation mode as a suffix.
`setattr(cls, name, run)` is the critical line that turns the closure into a discoverable unittest method on the target class.

**CN:** 定义完 `run` 之后，辅助函数会调用 `get_name(...)` 计算一个确定性的测试名，并把编译模式附加到名称后缀中。
最关键的一行是 `setattr(cls, name, run)`：它把这个闭包真正挂载为目标类上的 unittest 方法，从而能被测试框架发现。

## Key Concepts / 关键概念

- **Layout alias layer / 布局别名层:** Short symbols like `TNT` are translated into concrete CUTLASS layout tuples through `Layout` and `LayoutCombination`. / 像 `TNT` 这样的短符号通过 `Layout` 与 `LayoutCombination` 被翻译成具体的 CUTLASS 布局三元组。
- **Procedural naming / 过程化命名:** Generated test method names encode architecture, datatypes, layouts, shapes, schedules, and compiler mode. / 生成出的测试方法名会编码架构、数据类型、布局、形状、调度方式和编译模式。
- **Dynamic unittest method synthesis / 动态合成 unittest 方法:** A closure is created for each configuration and installed onto the chosen test class with `setattr`. / 每个配置都会生成一个闭包，并通过 `setattr` 安装到目标测试类上。
- **Intel Xe specialization / Intel Xe 特化处理:** The helper injects Xe-specific scheduling defaults so suite files do not have to repeat that policy. / 辅助函数内置了 Xe 专用的调度默认策略，避免各套件文件重复书写。

## Dependencies / 依赖关系

- **`cutlass_library` naming utilities / `cutlass_library` 命名工具:** Template substitution plus datatype/layout/opcode naming tables drive stable procedural test names. / 模板替换与数据类型/布局/Opcode 命名字典共同驱动稳定的过程化测试命名。
- **Architecture constants / 架构常量:** `INTEL_XE12`, `INTEL_XE20`, `INTEL_XE35`, and `is_intel_xe_arch` support cross-vendor naming and policy decisions. / `INTEL_XE12`、`INTEL_XE20`、`INTEL_XE35` 以及 `is_intel_xe_arch` 用于跨厂商命名和策略判断。
- **`cutlass_cppgen` / `cutlass_cppgen`:** Provides the GEMM plan object, schedule enums, tile scheduler enum, and backend interfaces used to build operations. / 提供构造 operation 所需的 GEMM plan、调度枚举、tile scheduler 枚举和后端接口。
- **Local `gemm_testbed` / 本地 `gemm_testbed`:** `test_all_gemm` is the execution-and-verification engine that every generated test method ultimately calls. / `test_all_gemm` 是所有动态生成测试方法最终都会调用的执行与校验引擎。
