# create_release.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/workflows/create_release.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a GitHub Actions workflow that orchestrates CI triggers, jobs, runner selection, and automation steps.
- **用途 (CN)**: 定义 GitHub Actions 工作流，用于编排 CI 触发条件、作业、运行器选择以及自动化步骤。

## Content Analysis / 内容分析
### Lines 1-12 / 第 1-12 行

````yaml
name: Create Release

on:
  push:
    branches:
      - main
      - release/*
    tags:
      # Final Release tags look like: v1.11.0
      - v[0-9]+.[0-9]+.[0-9]+
      # Release candidate tags look like: v1.11.0-rc1
      - v[0-9]+.[0-9]+.[0-9]+-rc[0-9]+
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。

### Lines 13-22 / 第 13-22 行

````yaml
  release:
    types: [published]
  pull_request:
    paths: [.github/workflows/create_release.yml]

jobs:
  get-label-type:
    if: github.repository_owner == 'pytorch'
    name: get-label-type
    uses: pytorch/pytorch/.github/workflows/_runner-determinator.yml@main
````

- EN: This section declares workflow triggers or event bindings; lays out job topology or execution stages; reuses actions and step building blocks such as `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`.
- CN: 该部分声明工作流触发条件或事件绑定；铺排作业拓扑或执行阶段；复用 Action 与步骤构件来拼装自动化流程。

### Lines 23-35 / 第 23-35 行

````yaml
    with:
      triggering_actor: ${{ github.triggering_actor }}
      issue_owner: ${{ github.event.pull_request.user.login || github.event.issue.user.login }}
      curr_branch: ${{ github.head_ref || github.ref_name }}
      curr_ref_type: ${{ github.ref_type }}

  release:
    if: ${{ github.repository == 'pytorch/pytorch' }}
    name: Create Release
    runs-on: ubuntu-latest
    # https://github.com/softprops/action-gh-release?tab=readme-ov-file#permissions
    permissions:
      contents: write
````

- EN: This section declares workflow triggers or event bindings; threads configuration values into later steps; controls token scope or environment variables for safe execution.
- CN: 该部分声明工作流触发条件或事件绑定；把配置值传递到后续步骤中；控制令牌权限范围或环境变量以实现安全执行。

### Lines 36-47 / 第 36-47 行

````yaml
    outputs:
      pt_release_name: ${{ steps.release_name.outputs.pt_release_name }}
      pt_pep517_release_name: ${{ steps.release_name.outputs.pt_pep517_release_name }}
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
        with:
          show-progress: false
          submodules: 'recursive'
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}
      - name: Fake name for PRs
        if: ${{ github.event_name == 'pull_request' }}
        run: echo "PT_GITHUB_REF=refs/tags/pr-tag" >> "$GITHUB_ENV"
````

- EN: This section declares workflow triggers or event bindings; reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分声明工作流触发条件或事件绑定；复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 48-56 / 第 48-56 行

````yaml
      - name: Real name for non-PRs
        if: ${{ github.event_name != 'pull_request' }}
        run: echo "PT_GITHUB_REF=$GITHUB_REF" >> "$GITHUB_ENV"
      - name: Set filenames
        run: |
          tag_or_branch="${PT_GITHUB_REF#refs/tags/}"
          tag_or_branch="${tag_or_branch#refs/heads/}"
          # replace directory separators with _ in branch name
          tag_or_branch="${tag_or_branch//\//_}"
````

- EN: This section declares workflow triggers or event bindings; embeds shell commands that perform the actual side effects.
- CN: 该部分声明工作流触发条件或事件绑定；内嵌执行实际副作用的 shell 命令。

### Lines 57-70 / 第 57-70 行

````yaml
          # Set PYTORCH_BUILD_VERSION only for release/RC tags; convert to PEP 440
          if [[ "$PT_GITHUB_REF" =~ ^refs/tags/v[0-9]+\.[0-9]+\.[0-9]+(-rc[0-9]+)?$ ]]; then
            ver="${PT_GITHUB_REF#refs/tags/v}"
            export PYTORCH_BUILD_VERSION="${ver/-rc/rc}"
            export PYTORCH_BUILD_NUMBER=0
          fi
          torch_version="$(python -c 'from tools.generate_torch_version import get_torch_version; print(get_torch_version())')"
          {
            echo "PT_RELEASE_NAME=pytorch-$tag_or_branch";
            echo "PT_RELEASE_FILE=pytorch-$tag_or_branch.tar.gz";
            echo "PT_PEP517_RELEASE_FILE=torch-${torch_version}.tar.gz";
            echo "PYTORCH_BUILD_VERSION=${PYTORCH_BUILD_VERSION:-}";
            echo "PYTORCH_BUILD_NUMBER=${PYTORCH_BUILD_NUMBER:-}";
          } >> "$GITHUB_ENV"
````

- EN: This section describes repository automation behavior for `.github/workflows/create_release.yml`.
- CN: 该部分描述 `.github/workflows/create_release.yml` 的仓库自动化行为。

### Lines 71-83 / 第 71-83 行

````yaml
      - name: Checkout optional submodules
        run: python3 tools/optional_submodules.py
      - name: Copy docs requirements for inclusion
        run: |
          # Replace symlink with actual file
          rm docs/requirements.txt || true
          cp .ci/docker/requirements-docs.txt docs/requirements.txt
      - name: Create source distribution
        run: |
          # Create new folder with specified name so extracting the archive yields that
          rm -rf "/tmp/$PT_RELEASE_NAME"
          cp -r "$PWD" "/tmp/$PT_RELEASE_NAME"
          mv "/tmp/$PT_RELEASE_NAME" .
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 84-94 / 第 84-94 行

````yaml
          # Cleanup
          rm -rf "$PT_RELEASE_NAME"/.ci
          find "$PT_RELEASE_NAME" -name '.git*' -exec rm -rv {} \; || true
          # Create archive
          tar -czf "$PT_RELEASE_FILE" "$PT_RELEASE_NAME"
          echo "Created source archive $PT_RELEASE_FILE with content: $(ls -a "$PT_RELEASE_NAME")"
      - name: Create PEP 517 compatible source distribution
        run: |
          pip install build==1.2.2.post1 || exit 1
          python -m build --sdist || exit 1
          mv dist/$PT_PEP517_RELEASE_FILE . || exit 1
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 95-107 / 第 95-107 行

````yaml
      - name: Upload source distribution for release
        if: ${{ github.event_name == 'release' }}
        uses: softprops/action-gh-release@da05d552573ad5aba039eaac05058a918a7bf631 # v2.2.2
        with:
          files: |
            ${{ env.PT_RELEASE_FILE }}
            ${{ env.PT_PEP517_RELEASE_FILE }}
      - name: Upload source distribution to GHA artifacts  # for release tags
        if: ${{ github.event_name == 'push' && startsWith(github.ref, 'refs/tags/v') && contains(github.ref, 'rc') }}
        uses: actions/upload-artifact@50769540e7f4bd5e21e526ee35c689e35e0d6874 # v4.4.0
        with:
          name: ${{ env.PT_RELEASE_FILE }}
          path: ${{ env.PT_RELEASE_FILE }}
````

- EN: This section reuses actions and step building blocks such as `softprops/action-gh-release@da05d552573ad5aba039eaac05058a918a7bf631`, `actions/upload-artifact@50769540e7f4bd5e21e526ee35c689e35e0d6874`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 108-120 / 第 108-120 行

````yaml
      - name: Upload PEP 517 source distribution to GHA artifacts  # for release tags
        if: ${{ github.event_name == 'push' && startsWith(github.ref, 'refs/tags/v') && contains(github.ref, 'rc') }}
        uses: actions/upload-artifact@50769540e7f4bd5e21e526ee35c689e35e0d6874 # v4.4.0
        with:
          name: ${{ env.PT_PEP517_RELEASE_FILE }}
          path: ${{ env.PT_PEP517_RELEASE_FILE }}
      - name: Set output
        id: release_name
        run: |
          {
            echo "pt_release_name=${{ env.PT_RELEASE_FILE }}";
            echo "pt_pep517_release_name=${{ env.PT_PEP517_RELEASE_FILE }}";
          } >> "${GITHUB_OUTPUT}"
````

- EN: This section reuses actions and step building blocks such as `actions/upload-artifact@50769540e7f4bd5e21e526ee35c689e35e0d6874`; embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 122-133 / 第 122-133 行

````yaml
  upload_source_code_to_s3:
    if: ${{ github.repository == 'pytorch/pytorch' && github.event_name == 'push' && startsWith(github.ref, 'refs/tags/v') && contains(github.ref, 'rc') }}
    runs-on: "${{ needs.get-label-type.outputs.label-type }}linux.2xlarge"
    environment: sourcecode-upload
    name: Upload source code to S3 for release tags
    permissions:
      id-token: write
    needs:
      - get-label-type
      - release
    steps:
      - uses: actions/download-artifact@65a9edc5881444af0b9093a5e628f2fe47ea3b2e # v4.1.7
````

- EN: This section reuses actions and step building blocks such as named actions; controls token scope or environment variables for safe execution.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；控制令牌权限范围或环境变量以实现安全执行。

### Lines 134-144 / 第 134-144 行

````yaml
        with:
          name: ${{ needs.release.outputs.pt_release_name }}
      - uses: actions/download-artifact@65a9edc5881444af0b9093a5e628f2fe47ea3b2e # v4.1.7
        with:
          name: ${{ needs.release.outputs.pt_pep517_release_name }}
      - name: Configure AWS credentials(PyTorch account)
        uses: aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722 # v4.1.0
        with:
          role-to-assume: arn:aws:iam::749337293305:role/gha_pytorch_source_code_upload_role
          aws-region: us-east-1
      - uses: seemethere/upload-artifact-s3@baba72d0712b404f646cebe0730933554ebce96a # v5.1.0
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 145-151 / 第 145-151 行

````yaml
        with:
          s3-bucket: pytorch
          s3-prefix: source_code/test
          if-no-files-found: warn
          path: |
            ${{ needs.release.outputs.pt_release_name }}
            ${{ needs.release.outputs.pt_pep517_release_name }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 153-155 / 第 153-155 行

````yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.sha }}-${{ github.event_name }}
  cancel-in-progress: true
````

- EN: This section declares workflow triggers or event bindings.
- CN: 该部分声明工作流触发条件或事件绑定。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `pytorch/pytorch/.github/workflows/_runner-determinator.yml@main`, `softprops/action-gh-release@da05d552573ad5aba039eaac05058a918a7bf631`, `actions/upload-artifact@50769540e7f4bd5e21e526ee35c689e35e0d6874`, `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`
- Inline commands / 内联命令: `echo`, `rm`, `cp`, `mv`, `find`, `tar`, `pip`, `python`
- Environment variables / 环境变量: `PT_GITHUB_REF`, `GITHUB_ENV`, `GITHUB_REF`, `PYTORCH_BUILD_VERSION`, `PEP`, `PYTORCH_BUILD_NUMBER`, `PT_RELEASE_NAME`, `PT_RELEASE_FILE`, `PT_PEP517_RELEASE_FILE`, `PWD`, ...
- Named jobs or sections / 命名作业或章节: `on`, `push`, `branches`, `tags`, `release`, `pull_request`, `jobs`, `get-label-type`, `with`, `permissions`, ...
