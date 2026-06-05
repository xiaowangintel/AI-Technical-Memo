# msprobe_debugging_guide.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/developer_guide/msprobe_debugging_guide.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Introduction to MSProbe MSProbe is a debugging tool for AI models that diagnoses accuracy anomalies and numerical errors during model training and inference. / 该文档围绕 MSProbe Debugging Guide 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Introduction to MSProbe
**EN:** MSProbe is a debugging tool for AI models that diagnoses accuracy anomalies and numerical errors during model training and inference.
**CN:** 本节围绕 Introduction to MSProbe 展开，概述了 errors, MSProbe, accuracy, numerical 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Background Concepts: MSProbe Dumping Levels
**EN:** MSProbe supports three accuracy levels for data dumping, each for different debugging needs: - **L0**: Dumps tensors/statistics at the **module level** and generates construct.json (for network structure reconstruction in visualization).
**CN:** 本节围绕 Background Concepts: MSProbe Dumping Levels 展开，概述了 Dumps, API, level, MSProbe 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prerequisites: Install MSProbe
**EN:** Install MSProbe with pip: ``shell pip install mindstudio-probe --pre ``
**CN:** 本节围绕 Prerequisites: Install MSProbe 展开，概述了 pip, pre, --pre, shell 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Key Configuration Parameters
**EN:** This section provides a comparison table for Key Configuration Parameters, covering columns such as Field, Description, Required and examples such as task, dump_path, rank, step.
**CN:** 本节围绕 Key 配置 Parameters 展开，概述了 list, level, dump, br/ 等要点，并说明相关配置、流程、示例或限制条件。

### Section: End-to-End Examples
**EN:** MSProbe’s full debugging workflow follows **Enable → Collect Data → Visualize → Analyze Root Cause**. Below is a common E2E example for SGLang-based model inference debugging.
**CN:** 本节围绕 End-to-End Examples 展开，概述了 E2E, Below, Enable, MSProbe 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example : Advanced Debugging with Custom Configuration
**EN:** Suitable for targeted debugging (e.g., only collect statistics data for specific ranks/steps, enable mix level for graph reconstruction + numerical comparison) and root cause analysis via **problem vs. benchmark comparison**.
**CN:** 本节围绕 Example : Advanced Debugging with Custom 配置 展开，概述了 Step, data, Problem, dump 等要点，并说明相关配置、流程、示例或限制条件。

### Section: No Dump Files Generated
**EN:** To confirm if MSProbe is installed, use pip show mindstudio_probe to troubleshoot. If it is installed, the MSProbe version information will be printed.
**CN:** 本节围绕 No Dump Files Generated 展开，概述了 MSProbe, installed, pip, Confirm 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Dump Files Are Too Large (Excessive Data)
**EN:** Start with task: "statistics" instead of "tensor" to collect only tensor statistics (avoids raw tensor dump); 2. Narrow the dump range with the scope field (specify start/end module/API); 3.
**CN:** 本节围绕 Dump Files Are Too Large (Excessive Data) 展开，概述了 dump, tensor, list, rank 等要点，并说明相关配置、流程、示例或限制条件。

### Section: TensorBoard Visualization Fails
**EN:** Confirm construct.json is not empty (requires level: L0 or mix – L1 does not generate graph files); 2. Check that the -tp (problem dump) and -gp (benchmark dump) paths point to **valid rank/step subdirectories** ( e.g., srep0/rank0); 3.
**CN:** 本节围绕 TensorBoard Visualization Fails 展开，概述了 mix, dump, Check, files 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Numerical Comparison Shows No Divergence But Model Accuracy Is Low
**EN:** Expand the dump step range (check more token iterations for late-stage divergence); 2. Switch to task: "tensor" (statistics may mask subtle numerical differences in raw tensor data); 3.
**CN:** 本节围绕 Numerical Comparison Shows No Divergence But 模型 Accuracy Is Low 展开，概述了 may, step, Expand, Switch 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Dump directory description
**EN:** ``text ├── problem_dump or bench_dump │ ├── step0 │ │ ├── rank0 │ │ │ ├── dump_tensor_data │ │ │ │ ├── Tensor.permute.1.forward.pt │ │ │ │ ├── Functional.linear.5.backward.output.pt # Format: . │ │ │ │ │ # arg_index is the nth input or output of the API.
**CN:** 本节围绕 Dump directory description 展开，概述了 Module, data, dump.json, API 等要点，并说明相关配置、流程、示例或限制条件。

### Section: dump.json file description
**EN:** #### L0 level An L0 dump.json contains forward/backward I/O for modules together with parameters and parameter gradients.
**CN:** 本节围绕 dump.json file description 展开，概述了 Max, Min, Mean, Norm 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** data / **CN:** data
- **EN:** dump / **CN:** dump
- **EN:** level / **CN:** level
- **EN:** API / **CN:** API
- **EN:** Max / **CN:** Max
- **EN:** Min / **CN:** Min
- **EN:** Mean / **CN:** Mean
- **EN:** Norm / **CN:** Norm

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
