# ascend_npu_environment_variables.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/platforms/ascend/ascend_npu_environment_variables.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang supports various environment variables related to Ascend NPU that can be used to configure its runtime behavior. This document provides a list of commonly used environment variables and aims to stay updated over time. / 该文档围绕 Environment Variables 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang supports various environment variables related to Ascend NPU that can be used to configure its runtime behavior. This document provides a list of commonly used environment variables and aims to stay updated over time.
**CN:** 本节围绕 Overview 展开，概述了 variables, environment, NPU, list 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Directly Used in SGLang
**EN:** This section provides a comparison table for Directly Used in SGLang, covering columns such as Environment Variable, Description, Default Value and examples such as SGLANG_NPU_USE_MLAPO, SGLANG_USE_FIA_NZ, SGLANG_NPU_USE_MULTI_STREAM, SGLANG_NPU_DISABLE_ACL_FORMAT_WEIGHT.
**CN:** 本节围绕 Directly Used in SGLang 展开，概述了 SGLANG_USE_FIA_NZ, SGLANG_NPU_USE_MLAPO, br/, Enable 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Used in DeepEP Ascend
**EN:** This section provides a comparison table for Used in DeepEP Ascend, covering columns such as Environment Variable, Description, Default Value and examples such as DEEPEP_NORMAL_LONG_SEQ_PER_ROUND_TOKENS, DEEPEP_NORMAL_LONG_SEQ_ROUND, DEEPEP_NORMAL_COMBINE_ENABLE_LONG_SEQ, MOE_ENABLE_TOPK_NEG_ONE.
**CN:** 本节围绕 Used in DeepEP Ascend 展开，概述了 Enable, br/, Indicates, stage 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Others
**EN:** This section provides a comparison table for Others, covering columns such as Environment Variable, Description, Default Value and examples such as TASK_QUEUE_ENABLE, INF_NAN_MODE_ENABLE, STREAMS_PER_DEVICE, PYTORCH_NPU_ALLOC_CONF.
**CN:** 本节围绕 Others 展开，概述了 Detail, Controls, Configures, br/ 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Detail / **CN:** Detail
- **EN:** br/ / **CN:** br/
- **EN:** Enable / **CN:** Enable
- **EN:** Controls / **CN:** Controls
- **EN:** Configures / **CN:** Configures
- **EN:** Description / **CN:** Description
- **EN:** SGLANG_USE_FIA_NZ / **CN:** SGLANG_USE_FIA_NZ
- **EN:** SGLANG_NPU_USE_MLAPO / **CN:** SGLANG_NPU_USE_MLAPO

## Dependencies / 依赖关系
- `//www.hiascend.com/document/detail/zh/Pytorch/730/comref/Envvariables/docs/zh/environment_variable_reference/TASK_QUEUE_ENABLE.md`
