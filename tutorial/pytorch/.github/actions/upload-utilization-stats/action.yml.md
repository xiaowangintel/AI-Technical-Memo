# action.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/actions/upload-utilization-stats/action.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a reusable composite GitHub Action, including declared inputs, step composition, and shell execution.
- **用途 (CN)**: 定义可复用的复合型 GitHub Action，包括输入声明、步骤组合以及 shell 执行逻辑。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````yaml
name: upload-utilization-stats

description: Upload utilization stats to artifacts.

inputs:
    workflow_run_id:
      type: string
      description: 'workflow (run) id of the workflow the test is running'
      required: True
    workflow_attempt:
      type: string
      description: 'the workflow (run) attempt'
      required: True
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 14-25 / 第 14-25 行

````yaml
    workflow_name:
      description: 'name of the workflow'
      type: string
      required: True
    job_id:
      type: string
      description: 'the job (run) id for the test'
      required: True
    job_name:
      type: string
      description: 'the job name of the test'
      required: True
````

- EN: This section describes repository automation behavior for `.github/actions/upload-utilization-stats/action.yml`.
- CN: 该部分描述 `.github/actions/upload-utilization-stats/action.yml` 的仓库自动化行为。

### Lines 26-36 / 第 26-36 行

````yaml
    local_path:
      type: string
      description: 'the local path to the utilization stats file'
      required: False
      default: ''
    artifact_prefix:
      type: string
      description: |
          'the prefix of the raw utilization data, for data stored in zip file, this is the prefix of the parent zip file'
      default: ""
      required: False
````

- EN: This section describes repository automation behavior for `.github/actions/upload-utilization-stats/action.yml`.
- CN: 该部分描述 `.github/actions/upload-utilization-stats/action.yml` 的仓库自动化行为。

### Lines 38-45 / 第 38-45 行

````yaml
runs:
  using: composite
  steps:
    - name: Setup uv
      if: ${{ !env.UV_PYTHON }}
      uses: pytorch/test-infra/.github/actions/setup-uv@main
      with:
        python-version: "3.12"
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-uv@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 47-57 / 第 47-57 行

````yaml
    - name: Print Inputs
      shell: bash
      run: |
        echo "workflow_id: ${{inputs.workflow_run_id}}"
        echo "workflow_attempt: ${{inputs.workflow_attempt}}"
        echo "workflow_Name: ${{inputs.workflow_name}}"
        echo "job_id: ${{inputs.job_id}}"
        echo "job_name:  ${{inputs.job_name}}"
        echo "artifact_prefix: ${{inputs.artifact_prefix}}"
    - uses: nick-fields/retry@v3.0.0
      name: Setup dependencies
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 58-66 / 第 58-66 行

````yaml
      with:
        shell: bash
        timeout_minutes: 5
        max_attempts: 5
        retry_wait_seconds: 30
        command: |
          set -eu
          # Pre-fetch dependencies to cache them for later uv run calls
          uv run --no-project --with python-dateutil==2.8.2 --with boto3==1.35.42 --with pandas==2.1.3 --with dataclasses_json==0.6.7 --with requests==2.32.3 python -c "print('Dependencies ready')"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 67-77 / 第 67-77 行

````yaml
    - name: Upload utilizatoin stats to s3
      shell: bash
      run: |
        uv run --no-project --with python-dateutil==2.8.2 --with boto3==1.35.42 --with pandas==2.1.3 --with dataclasses_json==0.6.7 --with requests==2.32.3 python -m tools.stats.upload_utilization_stats.upload_utilization_stats \
          --workflow-run-id "${{inputs.workflow_run_id}}" \
          --workflow-name "${{inputs.workflow_name}}" \
          --workflow-run-attempt "${{inputs.workflow_attempt}}" \
          --job-id "${{inputs.job_id}}" \
          --job-name "${{inputs.job_name}}" \
          --local-path "${{inputs.local_path}}" \
          --artifact-prefix "${{inputs.artifact_prefix}}"
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。


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
- Reusable actions / 复用 Action: `pytorch/test-infra/.github/actions/setup-uv@main`
- Inline commands / 内联命令: `'the`, `echo`, `set`, `uv`, `workflow-run-id`, `workflow-name`, `workflow-run-attempt`, `job-id`, ...
- Environment variables / 环境变量: `UV_PYTHON`
- Named jobs or sections / 命名作业或章节: `inputs`, `workflow_run_id`, `workflow_attempt`, `workflow_name`, `job_id`, `job_name`, `local_path`, `artifact_prefix`, `runs`, `steps`
