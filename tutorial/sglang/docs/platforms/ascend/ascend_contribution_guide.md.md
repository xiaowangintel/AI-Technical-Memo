# ascend_contribution_guide.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/platforms/ascend/ascend_contribution_guide.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Welcome to **SGLang**! We appreciate your interest in contributing. / 该文档围绕 Contribution Guide 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** Welcome to **SGLang**! We appreciate your interest in contributing.
**CN:** 本节围绕 Overview 展开，概述了 Welcome, Whether, run, bug 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prepare Environment
**EN:** Before contributing, please ensure that your environment is set up correctly. Follow the steps in the Installation Guide to install the necessary dependencies.
**CN:** 本节围绕 Prepare Environment 展开，概述了 Before, Follow, environment, ascend_npu.md 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Fork and clone the repository
**EN:** **Note**: New contributors do **not** have the write permission to push to the official SGLang repo. Please fork the repository under your GitHub account, then clone your fork locally.
**CN:** 本节围绕 Fork and clone the repository 展开，概述了 PYTHONPATH, New, Note, fork 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Format code with pre-commit
**EN:** We use pre-commit to maintain consistent code style checks.
**CN:** 本节围绕 Format code with pre-commit 展开，概述了 pre-commit, run, checks, branch 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Run and add unit tests
**EN:** If you add a new feature or fix a bug, please add corresponding unit tests to ensure coverage and prevent regression. SGLang uses Python's built-in unittest framework.
**CN:** 本节围绕 Run and add unit tests 展开，概述了 model, your_model, your_model_repo, python/sglang/test/ascend/test_ascend_utils.py 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Write documentations
**EN:** We recommend new contributors start from writing documentation, which helps you quickly understand SGLang codebase. For more details, please refer to docs/README.md.
**CN:** 本节围绕 Write documentations 展开，概述了 README.md, new, start, helps 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Test the accuracy
**EN:** If your code changes the model output, please run the accuracy tests. A quick sanity check is the few-shot GSM8K.
**CN:** 本节围绕 Test the accuracy 展开，概述了 accuracy, GSM8K, Please, test 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Benchmark the speed
**EN:** Refer to Benchmark and Profiling.
**CN:** 本节围绕 Benchmark the speed 展开，概述了 Refer, Benchmark, Profiling, developer_guide/benchmark_and_profiling.md 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Requesting a review for merge
**EN:** You can follow the pull request merge process described in MAINTAINER.md. You will need to work with the Merge Oncall, Codeowner, and other reviewers to get their approvals.
**CN:** 本节围绕 Requesting a review for merge 展开，概述了 MAINTAINER.md, Then, Codeowner, pull 等要点，并说明相关配置、流程、示例或限制条件。

### Section: How to Trigger CI Tests
**EN:** We have a lot of open PRs but limited CI machines, so only top and trusted contributors have permission to trigger CI tests. Users with permission are listed in the CI_PERMISSIONS.json For CI to run on a pull request, it must have the "run-ci" label.
**CN:** 本节围绕 How to Trigger CI Tests 展开，概述了 Reruns, trigger, permission, test 等要点，并说明相关配置、流程、示例或限制条件。

### Section: CI rate limits
**EN:** Due to CI scheduling and limited resources, higher-priority PRs may preempt running jobs. In such cases, you may need to rerun the tests.
**CN:** 本节围绕 CI rate limits 展开，概述了 workflow, may, cool-down-minutes, CI_PERMISSIONS.json 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Code style guidance
**EN:** Avoid code duplication. If the same code snippet (more than five lines) appears multiple times, extract it into a shared function.
**CN:** 本节围绕 Code style guidance 展开，概述了 code, e.g, new, Keep 等要点，并说明相关配置、流程、示例或限制条件。

### Section: How to update sgl-kernel
**EN:** Since sglang and sgl-kernel are separate Python packages, our current GitHub CI infrastructure does not support updating a kernel and using it immediately within the same pull request (PR).
**CN:** 本节围绕 How to update sgl-kernel 展开，概述了 Update, kernel, sglang-kernel, new 等要点，并说明相关配置、流程、示例或限制条件。

### Section: How to update sgl-kernel-npu
**EN:** Sgl-kernel-npu is the kernel package for Ascend NPU and is maintained in the sgl-kernel-npu repository. if you want to add a new kernel and want to use it in sglang, please follow the steps in Contribution Guide.
**CN:** 本节围绕 How to update sgl-kernel-npu 展开，概述了 want, kernel, Sgl-kernel-npu, NPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Tips for newcomers
**EN:** If you want to contribute but don’t have a specific idea in mind, pick issues labeled “good first issue” or “help wanted”. These tasks typically have lower complexity and provide an excellent introduction to the codebase.
**CN:** 本节围绕 Tips for newcomers 展开，概述了 want, good, help, These 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** code / **CN:** code
- **EN:** tests / **CN:** tests
- **EN:** new / **CN:** new
- **EN:** run / **CN:** run
- **EN:** model / **CN:** 模型
- **EN:** e.g / **CN:** e.g
- **EN:** please / **CN:** please
- **EN:** test / **CN:** test

## Dependencies / 依赖关系
- `ascend_npu.md`
- `python/sglang/test/ascend/test_ascend_utils.py`
- `../../developer_guide/benchmark_and_profiling.md`
- `scheduler.py`
- `scheduler_output_processor_mixin.py`
- `test_eagle_infer_a.py`
- `test_eagle_infer_b.py`
- `allocator_npu.py`
- `test/README.md`
- `//github.com/sgl-project/sglang/tree/main/test/README.md`
- `docs/README.md`
- `//github.com/sgl-project/sglang/tree/main/docs/README.md`
