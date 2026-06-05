# apply-release-changes.sh — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `scripts/release/apply-release-changes.sh`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Contains release automation scripts that update branches, versions, and other cut-release bookkeeping.
- **用途 (CN)**: 包含发布自动化脚本，用于更新分支、版本以及其他发版流程中的账务性步骤。

## Content Analysis / 内容分析
### Lines 1-12 / 第 1-12 行

````bash
#!/usr/bin/env bash
#
# Step 2 after branch cut is complete.
#
# Creates PR with release only changes.
#
# Prerequisite: Must be  successfully authenticated in aws fbossci account.
#
# Usage (run from root of project):
#  DRY_RUN=disabled ./scripts/release/apply-release-changes.sh
#
# RELEASE_VERSION: Version of this current release
````

- EN: This section declares the interpreter used by the script.
- CN: 该部分声明脚本使用的解释器。

### Lines 14-23 / 第 14-23 行

````bash
set -eou pipefail

GIT_TOP_DIR=$(git rev-parse --show-toplevel)
RELEASE_VERSION=${RELEASE_VERSION:-$(cut -d'.' -f1-2 "${GIT_TOP_DIR}/version.txt")}
DRY_RUN=${DRY_RUN:-enabled}

echo "Applying to workflows"
for i in .github/workflows/*.yml; do
    sed -i -e s#@main#@"release/${RELEASE_VERSION}"# $i;
done
````

- EN: This section enables strict shell options for safer execution; uses control flow to branch on environment or iterate through inputs; invokes commands such as `set`, `echo`, `sed`.
- CN: 该部分启用更严格的 shell 选项以提升执行安全性；通过控制流按环境分支或遍历输入；调用多个外部命令来完成自动化动作。

### Lines 25-36 / 第 25-36 行

````bash
echo "Applying to templates"
for i in .github/templates/*.yml.j2; do
    sed -i '/checkout_pr_head/!s#common.checkout(\([^)]*\))#common.checkout(\1, checkout_pr_head=False)#' $i;
    sed -i -e s#main#"release/${RELEASE_VERSION}"# $i;
done

echo "Applying to changes to linux binary builds"
for i in  ".github/workflows/_binary-build-linux.yml" ".github/workflows/_binary-test-linux.yml"; do
    sed -i "/github.event_name == 'pull_request'/d" $i;
done

sed -i "s#^        \.github/scripts/generate_ci_workflows.py#        RELEASE_VERSION_TAG=${RELEASE_VERSION} .github/scripts/generate_ci_workflows.py#" .github/workflows/lint.yml
````

- EN: This section uses control flow to branch on environment or iterate through inputs; invokes commands such as `echo`, `sed`.
- CN: 该部分通过控制流按环境分支或遍历输入；调用多个外部命令来完成自动化动作。

### Lines 38-49 / 第 38-49 行

````bash
# Triton wheel
echo "Triton Changes"
sed -i -e s#-\ main#"-\ release\/${RELEASE_VERSION}"# .github/workflows/build-triton-wheel.yml

# XLA related changes
echo "XLA Changes"
sed -i -e s#--quiet#-b\ r"${RELEASE_VERSION}"# .ci/pytorch/common_utils.sh
sed -i -e s#.*#r"${RELEASE_VERSION}"# .github/ci_commit_pins/xla.txt

# Strip +PTX from CUDA arch lists in release builds
echo "Stripping +PTX from CUDA arch lists"
sed -i 's/+PTX//' .ci/manywheel/build_cuda.sh
````

- EN: This section invokes commands such as `echo`, `sed`.
- CN: 该部分调用多个外部命令来完成自动化动作。

### Lines 51-63 / 第 51-63 行

````bash
# Regenerate templates
export RELEASE_VERSION_TAG=${RELEASE_VERSION}
./.github/regenerate.sh

# Pin Unstable and disabled jobs and tests
UNSTABLE_VER=$(aws s3api list-object-versions --bucket ossci-metrics --prefix unstable-jobs.json --query 'Versions[?IsLatest].[VersionId]' --output text)
DISABLED_VER=$(aws s3api list-object-versions --bucket ossci-metrics --prefix disabled-jobs.json --query 'Versions[?IsLatest].[VersionId]' --output text)
SLOW_VER=$(aws s3api list-object-versions --bucket ossci-metrics --prefix slow-tests.json --query 'Versions[?IsLatest].[VersionId]' --output text)
DISABLED_TESTS_VER=$(aws s3api list-object-versions --bucket ossci-metrics --prefix disabled-tests-condensed.json --query 'Versions[?IsLatest].[VersionId]' --output text)
sed -i -e s#unstable-jobs.json#"unstable-jobs.json?versionId=${UNSTABLE_VER}"# .github/scripts/filter_test_configs.py
sed -i -e s#disabled-jobs.json#"disabled-jobs.json?versionId=${DISABLED_VER}"# .github/scripts/filter_test_configs.py
sed -i -e s#disabled-tests-condensed.json#"disabled-tests-condensed.json?versionId=${DISABLED_TESTS_VER}"# tools/stats/import_test_stats.py
# Optional
````

- EN: This section invokes commands such as `./.github/regenerate.sh`, `sed`.
- CN: 该部分调用多个外部命令来完成自动化动作。

### Lines 64-64 / 第 64-64 行

````bash
# git commit -m "[RELEASE-ONLY CHANGES] Branch Cut for Release ${RELEASE_VERSION}"
````

- EN: This section uses control flow to branch on environment or iterate through inputs.
- CN: 该部分通过控制流按环境分支或遍历输入。

### Lines 65-65 / 第 65-65 行

````bash
# git push origin "release/${RELEASE_VERSION}"
````

- EN: This section documents shell-based automation steps.
- CN: 该部分记录基于 shell 的自动化步骤。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Configures native build inputs, optional backends, and dependency discovery at configure time.
  CN: 在配置阶段设置原生构建输入、可选后端以及依赖发现逻辑。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。

## Dependencies / 依赖关系
- Shell commands / Shell 命令: `set`, `echo`, `sed`, `./.github/regenerate.sh`
- Sourced files / source 文件: none
- Environment variables / 环境变量: `DRY_RUN`, `RELEASE_VERSION`, `GIT_TOP_DIR`, `RELEASE_VERSION_TAG`, `XLA`, `PTX`, `CUDA`, `UNSTABLE_VER`, `DISABLED_VER`, `SLOW_VER`, ...
