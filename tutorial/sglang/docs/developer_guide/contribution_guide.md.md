# contribution_guide.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/developer_guide/contribution_guide.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Welcome to **SGLang**! We appreciate your interest in contributing. / 该文档围绕 Contribution Guide 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** Welcome to **SGLang**! We appreciate your interest in contributing.
**CN:** 本节围绕 Overview 展开，概述了 Welcome, Whether, run, bug 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Fork and clone the repository
**EN:** **Note**: New contributors do **not** have the write permission to push to the official SGLang repo. Please fork the repository under your GitHub account, then clone your fork locally.
**CN:** 本节围绕 Fork and clone the repository 展开，概述了 New, Note, fork, clone 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Build from source
**EN:** Refer to Install SGLang from Source.
**CN:** 本节围绕 Build from source 展开，概述了 Refer, Source, Install, Install SGLang 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Format code with pre-commit
**EN:** We use pre-commit to maintain consistent code style checks.
**CN:** 本节围绕 Format code with pre-commit 展开，概述了 pre-commit, run, checks, branch 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Run and add unit tests
**EN:** If you add a new feature or fix a bug, please add corresponding unit tests to ensure coverage and prevent regression.
**CN:** 本节围绕 Run and add unit tests 展开，概述了 add, new, fix, bug 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Unit tests (no server required)
**EN:** Unit tests live under test/registered/unit/, organized to mirror the python/sglang/srt/ source tree. These tests validate component logic **without** launching a server or loading real model weights.
**CN:** 本节围绕 Unit tests (no 服务端 required) 展开，概述了 test/registered/unit/, Run, test, tests 等要点，并说明相关配置、流程、示例或限制条件。

### Section: E2E tests (server required)
**EN:** For tests that require launching a server, refer to test/registered/README.md for guidance on where to place your test. For detailed instructions on running tests and integrating them into CI, refer to test/README.md.
**CN:** 本节围绕 E2E tests (服务端 required) 展开，概述了 README.md, tests, refer, test/registered/README.md 等要点，并说明相关配置、流程、示例或限制条件。

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
**EN:** We have a lot of open PRs but limited CI machines, so only top and trusted contributors have permission to trigger CI tests.
**CN:** 本节围绕 How to Trigger CI Tests 展开，概述了 CI_PERMISSIONS.json, PRs, Reruns, trigger 等要点，并说明相关配置、流程、示例或限制条件。

### Section: CI rate limits
**EN:** Due to CI scheduling and limited resources, higher-priority PRs may preempt running jobs. In such cases, you may need to rerun the tests.
**CN:** 本节围绕 CI rate limits 展开，概述了 workflow, may, cooldown, cool-down-minutes 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Code style guidance
**EN:** Avoid code duplication. If the same code snippet (more than five lines) appears multiple times, extract it into a shared function.
**CN:** 本节围绕 Code style guidance 展开，概述了 code, e.g, new, Keep 等要点，并说明相关配置、流程、示例或限制条件。

### Section: How to update sgl-kernel
**EN:** Since sglang and the sglang-kernel (prior sgl-kernel) distribution are separate Python packages, our current GitHub CI infrastructure does not support updating a kernel and using it immediately within the same pull request (PR).
**CN:** 本节围绕 How to update sgl-kernel 展开，概述了 sglang-kernel, Update, kernel, new 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Tips for newcomers
**EN:** If you want to contribute but don’t have a specific idea in mind, pick issues labeled “good first issue” or “help wanted”. These tasks typically have lower complexity and provide an excellent introduction to the codebase.
**CN:** 本节围绕 Tips for newcomers 展开，概述了 want, good, help, These 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** tests / **CN:** tests
- **EN:** code / **CN:** code
- **EN:** run / **CN:** run
- **EN:** test / **CN:** test
- **EN:** new / **CN:** new
- **EN:** e.g / **CN:** e.g
- **EN:** CI_PERMISSIONS.json / **CN:** CI_PERMISSIONS.json
- **EN:** Please / **CN:** Please

## Dependencies / 依赖关系
- `../get_started/install.md`
- `../developer_guide/benchmark_and_profiling.md`
- `scheduler.py`
- `scheduler_output_processor_mixin.py`
- `test_eagle_infer_a.py`
- `test_eagle_infer_b.py`
- `allocator_ascend.py`
- `srt/mem_cache/radix_cache.py`
- `unit/mem_cache/test_radix_cache.py`
- `srt/sampling/sampling_params.py`
- `unit/sampling/test_sampling_params.py`
- `test/registered/unit/README.md`
