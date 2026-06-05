# setup_github_runner.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/developer_guide/setup_github_runner.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Add a Runner Step 1: Start a docker container. **You can mount a folder for the shared huggingface model weights cache. / 该文档围绕 Set Up Self-Hosted Runners for GitHub Actions 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Step 1: Start a docker container.
**EN:** **You can mount a folder for the shared huggingface model weights cache. ** The command below uses /tmp/huggingface as an example.
**CN:** 本节围绕 Step 1: Start a docker container. 展开，概述了 device, --device, AMD, docker 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 2: Configure the runner by config.sh
**EN:** Run these commands inside the container. The labels can be edited later in Github Settings.
**CN:** 本节围绕 Step 2: Configure the runner by config.sh 展开，概述了 Run, apt, pip, e.g 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 3: Run the runner by run.sh
**EN:** Set up environment variables `` export HF_HOME=/hf_home export SGLANG_IS_IN_CI=true export HF_TOKEN=hf_xxx export OPENAI_API_KEY=sk-xxx export CUDA_VISIBLE_DEVICES=0 ` - Run it forever ` while true; do ./run.sh; echo "Restarting..."; sleep 2; done ``
**CN:** 本节围绕 Step 3: Run the runner by run.sh 展开，概述了 export, Run, HF_HOME, HF_TOKEN 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Run / **CN:** Run
- **EN:** Step / **CN:** Step
- **EN:** export / **CN:** export
- **EN:** docker / **CN:** docker
- **EN:** device / **CN:** device
- **EN:** --device / **CN:** --device
- **EN:** AMD / **CN:** AMD
- **EN:** run.sh / **CN:** run.sh

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
