# sve_emblookup_codegen.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `caffe2/perfkernels/sve_emblookup_codegen.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CPU performance kernels and code generators, especially for embedding lookup and vectorized paths.
- **Purpose (CN)**: 实现 CPU 性能内核与代码生成器，尤其关注 embedding lookup 与向量化路径。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
# mypy: allow-untyped-defs
import argparse
import sys


# Unroll loops when block_size is a multiple of vector length.
def unroll(num_unrolls, IndexType, InType, OutType):
    def compute_output(num_unrolls, InType, is_main):
        code = []

        pred = "svAll" if is_main else "pg"
        if InType == "float":
            for i in range(num_unrolls):
                code.append(f"        output = svmla_x({pred}, output, svld1(svAll, &ip{i}[k]), wgt{i});")
        elif InType == "at::Half":
            for i in range(num_unrolls):
                code.append(f"        auto input{i} = svcvt_f32_x({pred}, svreinterpret_f16(\n"
                f"          svld1uh_u32({pred}, reinterpret_cast<const uint16_t*>(&ip{i}[k]))));")
```
- **EN**: The import section wires together standard-library modules such as argparse, sys for the logic below. This chunk defines `compute_output`, which implements one step in low-level runtime or performance support code. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 导入区把标准库模块，如 argparse、sys组织在一起，供下方逻辑使用。 这一段定义了 `compute_output`，其作用是实现底层运行时或性能支持代码中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 19-34
```python
            for i in range(num_unrolls):
                code.append(f"        output = svmla_x({pred}, output, input{i}, wgt{i});")
        elif InType == "at::BFloat16":
            for i in range(num_unrolls):
                code.append(f"        auto input{i} = svreinterpret_f32(svlsl_x({pred},\n"
                f"          svld1uh_u32({pred}, reinterpret_cast<const uint16_t*>(&ip{i}[k])), 16));")
            for i in range(num_unrolls):
                code.append(f"        output = svmla_x({pred}, output, input{i}, wgt{i});")
        elif InType == "uint8_t":
            code.append(f"        output = svadd_x({pred}, output, bio);")
            for i in range(num_unrolls):
                code.append(f"        auto input{i} = svcvt_f32_x({pred}, svld1ub_u32({pred}, &ip{i}[k]));")
            for i in range(num_unrolls):
                code.append(f"        output = svmla_x({pred}, output, input{i}, wgt{i});")
        else:
            raise ValueError(f'Unknown datatype "{InType}"')
```
- **EN**: This chunk continues `compute_output` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `compute_output`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 36-47
```python
        return code

    code = []

    if num_unrolls == 1:
        code.append("    // tail loop")
        code.append("    if (j < end_offset) {")
    else:
        code.append(f"    // unrolling {num_unrolls} times")
        code.append(f"    while (j + {num_unrolls - 1} < end_offset) {{")
    for i in range(num_unrolls):
        code.append(f"      const auto idx{i} = indices[pos + {i}];")
```
- **EN**: Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. This chunk continues `compute_output` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 这一段延续了 `compute_output`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 49-63
```python
    # check indices
    for i in range(num_unrolls):
        code.append(
            f"      if (idx{i} < 0 || idx{i} >= data_size) {{\n"
            + "        return false;\n"
            + "      }"
        )

    if InType == "uint8_t":
        for i in range(num_unrolls):
            code.append(f"      {OutType} wgt{i} = 1.f;")
        code.append(f"      {OutType} bio = 0.f;")
    else:
        for i in range(num_unrolls):
            code.append(f"      {OutType} wgt{i} = 1.f;")
```
- **EN**: Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. This chunk continues `compute_output` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 这一段延续了 `compute_output`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 65-77
```python
    code.append("      if (weights) {")
    for i in range(num_unrolls):
        code.append(f"        wgt{i} = weights[IS_WEIGHT_POSITIONAL ? (j + {i} - start_offset) : pos + {i}];")
    code.append("      }")
    if InType == "uint8_t":
        code.append("      if (scale_bias) {")
        for i in range(num_unrolls):
            code.append(f"        bio += wgt{i} * scale_bias[2 * idx{i} + 1];")
            code.append(f"        wgt{i} = wgt{i} * scale_bias[2 * idx{i}];")
        code.append("      }")

    for i in range(num_unrolls):
        code.append(f"      const {InType}* const ip{i} = &input[idx{i} * block_size];")
```
- **EN**: This chunk continues `compute_output` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `compute_output`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 79-96
```python
    # compute and store
    code.append("      svbool_t pg;")
    code.append("      int64_t k = 0;")
    # main loop
    code.append("      while (k + vLen - 1 < block_size) {")
    code.append("        auto output = svld1(svAll, &op[k]);")
    code.extend(compute_output(num_unrolls, InType, True))
    code.append("        svst1(svAll, &op[k], output);")
    code.append("        k += vLen;")
    code.append("      }")
    # tail loop
    code.append("      if (k < block_size) {")
    code.append("        pg = svwhilelt_b32_s64(k, block_size);")
    code.append("        auto output = svld1(pg, &op[k]);")
    code.extend(compute_output(num_unrolls, InType, False))
    code.append("        svst1(pg, &op[k], output);")
    code.append("        k += vLen;")
    code.append("      }")
```
- **EN**: This chunk continues `compute_output` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `compute_output`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 97-114
```python
    if num_unrolls == 1:
        code.append("      pos ++;")
    else:
        code.append(f"      j += {num_unrolls};")
        code.append(f"      pos += {num_unrolls};")

    code.append("    }")

    return code

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("-f", "--filename", help="file name")
    opts = parser.parse_args()
    if opts.filename:
        filename = opts.filename
    else:
        filename = "embedding_lookup_idx_sve.cc"
```
- **EN**: This chunk defines `main`, which implements one step in low-level runtime or performance support code. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `main`，其作用是实现底层运行时或性能支持代码中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 116-133
```python
    options = [
        ["int32_t", "int32_t", "float", "float", "float", "float"],
        ["int64_t", "int64_t", "float", "float", "float", "float"],
        ["int32_t", "int32_t", "half", "at::Half", "float", "float"],
        ["int64_t", "int64_t", "half", "at::Half", "float", "float"],
        ["int32_t", "int32_t", "bfloat16", "at::BFloat16", "float", "float"],
        ["int64_t", "int64_t", "bfloat16", "at::BFloat16", "float", "float"],
        ["int32_t", "int32_t", "uint8_t", "uint8_t", "float", "float"],
        ["int64_t", "int64_t", "uint8_t", "uint8_t", "float", "float"],
    ]

    code = []
    # includes
    code.append("//// --------------------------")
    code.append("//// ATTENTION:")
    code.append("//// THIS CODE IS AUTOGENERATED")
    code.append(f"//// BY {' '.join(sys.argv)}")
    code.append("//// DO NOT MODIFY!!!")
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 134-147
```python
    code.append("//// --------------------------\n")

    code.append("#include <arm_sve.h>")
    code.append("#include <c10/util/BFloat16.h>")
    code.append("#include <c10/util/Half.h>")
    code.append("#include <cstdint>")
    code.append("#include <cstring>")

    code.append("namespace caffe2 {\n")
    for o in options:
        [IndexTypeName, IndexType, InTypeName, InType, OutTypeName, OutType] = o

        code.append("template <bool IS_WEIGHT_POSITIONAL>")
        fn_base = f"EmbeddingLookupIdx_{IndexTypeName}_{InTypeName}_{OutTypeName}"
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 149-165
```python
        suffix = "__sve"
        fn = "static bool " + fn_base + suffix
        code.append(fn + "(")

        args = []
        args.append("    const int64_t block_size,")
        args.append("    const int64_t output_size,")
        args.append("    const int64_t index_size,")
        args.append("    const int64_t data_size,")
        args.append("    const " + InType + "* input,")
        args.append("    const " + IndexType + "* indices,")
        args.append("    const " + IndexType + "* offsets,")
        args.append("    const float* weights,")
        args.append("    const float* scale_bias,")
        args.append("    bool normalize_by_lengths,")
        args.append("    " + OutType + "* out) {")
        code += args
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 167-184
```python
        code.append("  const svbool_t svAll = svptrue_b32();")
        code.append("  const auto vLen = static_cast<int64_t>(svcntw());")
        code.append("  int64_t pos = 0;")

        code.append("  for (int64_t i = 0; i < output_size; ++i) {")
        code.append("    " + OutType + "* const op = &out[i * block_size];")

        # initialize to 0
        code.append("    memset(op, 0, sizeof(float) * block_size);")

        # inner loop
        code.append(
            "    if (pos != offsets[i] - offsets[0]) {\n"
            + "      return false;\n"
            + "    }"
        )
        code.append(
            "    int64_t start_offset = offsets[i];\n"
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 185-202
```python
            + "    int64_t end_offset = offsets[i + 1];"
        )
        code.append("    int64_t j = start_offset;")

        code += unroll(16, IndexType, InType, OutType)
        code += unroll(8, IndexType, InType, OutType)
        code += unroll(4, IndexType, InType, OutType)
        code += unroll(2, IndexType, InType, OutType)
        code += unroll(1, IndexType, InType, OutType)

        code.append("    const int64_t length = end_offset - start_offset;\n")
        code.append("    if (normalize_by_lengths && length != 0) {")
        code.append("      const float len_inv = 1.0f / length;")
        code.append("      svbool_t pg;")
        code.append("      int64_t j = 0;")
        code.append("      while (j + vLen - 1 < block_size) {")
        code.append("        svst1(svAll, &op[j], svmul_x(svAll, svld1(svAll, &op[j]), len_inv));")
        code.append("        j += vLen;")
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 203-216
```python
        code.append("      }")
        code.append("      if (j < block_size) {")
        code.append("        pg = svwhilelt_b32_s64(j, block_size);")
        code.append("        svst1(pg, &op[j], svmul_x(pg, svld1(pg, &op[j]), len_inv));")
        code.append("      }")
        code.append("    }")

        code.append("  }")
        code.append("  return pos == index_size;")
        code.append("}")

        for is_weight_positional in ["false", "true"]:
            code.append("bool " + fn_base + "_" + is_weight_positional + suffix + "(")
            code += args
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 218-234
```python
            # Resolve the Lint warnings: Limit of 80 characters in one line.
            extra_space = "\n      "
            ret_string = (
                "  return " + fn_base + suffix + "<" + is_weight_positional + ">("
            )
            if len(ret_string) <= 80:
                code.append(ret_string)
            else:
                code.append(
                    "  return "
                    + fn_base
                    + suffix
                    + "<"
                    + extra_space
                    + is_weight_positional
                    + ">("
                )
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 236-251
```python
            code.append("      block_size,")
            code.append("      output_size,")
            code.append("      index_size,")
            code.append("      data_size,")
            code.append("      input,")
            code.append("      indices,")
            code.append("      offsets,")
            code.append("      weights,")
            code.append("      scale_bias,")
            code.append("      normalize_by_lengths,")
            code.append("      out);")
            code.append("}")

        code.append("")

    code.append("} // namespace caffe2")
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。

### Lines 253-260
```python
    with open(filename, "w") as fout:
        fout.write("\n".join(code) + "\n")

    print("Created " + filename)


if __name__ == "__main__":
    main()
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

## Key Concepts / 关键概念

- **Low-level runtime support**
  - EN: Provides foundational runtime, serialization, and CPU-performance helpers.
  - CN: 提供基础运行时、序列化与 CPU 性能辅助能力。
- **Code-generation templates**
  - EN: Uses placeholders and boilerplate patterns that are later materialized into concrete sources.
  - CN: 使用占位符与样板模式，随后会被实例化为具体源码。
- **Embedding kernels**
  - EN: Implements optimized embedding lookup kernels and related code generation.
  - CN: 实现优化的 embedding lookup 内核及其相关代码生成。
- **unroll**
  - EN: `unroll` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `unroll` 是本文件声明、导出或驱动的显著符号之一。
- **compute_output**
  - EN: `compute_output` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `compute_output` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Standard library / 标准库**: `argparse`, `sys`
- **Primary symbols / 核心符号**: `unroll`, `compute_output`, `main`
