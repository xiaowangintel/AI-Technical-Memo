# action.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/actions/upload-test-artifacts/action.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a reusable composite GitHub Action, including declared inputs, step composition, and shell execution.
- **用途 (CN)**: 定义可复用的复合型 GitHub Action，包括输入声明、步骤组合以及 shell 执行逻辑。

## Content Analysis / 内容分析
### Lines 1-17 / 第 1-17 行

````yaml
name: Upload test artifacts

description: |
  Upload various artifacts produced by our testing process.
  Attempts to upload to S3 first, and falls back to GitHub Actions artifacts if S3 upload fails.

inputs:
  use-gha:
    description: |
      Deprecated: This input is no longer used. The action now automatically
      attempts S3 upload first and falls back to GHA if S3 fails.
    required: false
  file-suffix:
    description: |
      Suffix to add to the filename of the artifacts. This should include the
      workflow job id, see [Job id in artifacts].
    required: true
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 18-31 / 第 18-31 行

````yaml
  s3-bucket:
    description: S3 bucket to upload artifacts to
    required: false
    default: "gha-artifacts"

runs:
  using: composite
  steps:
    # Always set up uv and zip files first (needed for S3, reusable for GHA fallback)
    - name: Setup uv
      if: ${{ runner.os != 'Windows' && !env.UV_PYTHON }}
      uses: pytorch/test-infra/.github/actions/setup-uv@main
      with:
        python-version: "3.12"
````

- EN: This section reuses actions and step building blocks such as `pytorch/test-infra/.github/actions/setup-uv@main`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 33-49 / 第 33-49 行

````yaml
    - name: Zip artifacts for upload
      if: runner.os != 'Windows'
      shell: bash
      env:
        FILE_SUFFIX: ${{ inputs.file-suffix }}
      run: |
        set -euo pipefail
        rm -f test-jsons-*.zip test-reports-*.zip logs-*.zip debug-*.zip profiler-traces-*.zip tlparse-*.zip
        ZIP_CMD='
        import sys, zipfile, os
        from pathlib import Path
        d = Path(sys.argv[1])
        o = Path(sys.argv[2])
        p = sys.argv[3:]
        with zipfile.ZipFile(o, "w", zipfile.ZIP_DEFLATED) as z:
            for pat in p:
                for f in d.glob(pat):
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 50-67 / 第 50-67 行

````yaml
                    if f.is_file():
                        z.write(f, os.path.relpath(f, d))
        '
        uv run --no-project python -c "$ZIP_CMD" \
          test/test-reports "test-jsons-${FILE_SUFFIX}.zip" '**/*.json'
        uv run --no-project python -c "$ZIP_CMD" \
          . "test-reports-${FILE_SUFFIX}.zip" 'test/test-reports/**/*.xml' 'test/test-reports/**/*.csv'
        uv run --no-project python -c "$ZIP_CMD" \
          . "logs-${FILE_SUFFIX}.zip" 'usage_log.txt' 'test/test-reports/**/*.log'
        if [ -d 'test/debug' ]; then
          uv run --no-project python -c "$ZIP_CMD" \
            test/debug "debug-${FILE_SUFFIX}.zip" '**/*'
        fi
        if [ -d 'test/test-reports/profiler_traces' ]; then
          uv run --no-project python -c "$ZIP_CMD" \
            test/test-reports/profiler_traces "profiler-traces-${FILE_SUFFIX}.zip" '**/*'
        fi
        if [ -d 'test/test-reports/tlparse_output' ]; then
````

- EN: This section describes repository automation behavior for `.github/actions/upload-test-artifacts/action.yml`.
- CN: 该部分描述 `.github/actions/upload-test-artifacts/action.yml` 的仓库自动化行为。

### Lines 68-84 / 第 68-84 行

````yaml
          uv run --no-project python -c "$ZIP_CMD" \
            test/test-reports/tlparse_output "tlparse-${FILE_SUFFIX}.zip" '**/*'
        fi

    # Windows zip
    - name: Zip JSONs for upload
      if: runner.os == 'Windows'
      shell: powershell
      env:
        FILE_SUFFIX: ${{ inputs.file-suffix }}
      run: |
        # -ir => recursive include all files in pattern
        7z a "test-jsons-$Env:FILE_SUFFIX.zip" -ir'!test\test-reports\*.json'

    - name: Zip test reports for upload
      if: runner.os == 'Windows'
      shell: powershell
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 85-101 / 第 85-101 行

````yaml
      env:
        FILE_SUFFIX: ${{ inputs.file-suffix }}
      run: |
        # -ir => recursive include all files in pattern
        7z a "test-reports-$Env:FILE_SUFFIX.zip" -ir'!test\test-reports\*.xml' -ir'!test\test-reports\*.csv'

    - name: Zip usage log for upload
      if: runner.os == 'Windows'
      continue-on-error: true
      shell: powershell
      env:
        FILE_SUFFIX: ${{ inputs.file-suffix }}
      run: |
        # -ir => recursive include all files in pattern
        7z a "logs-$Env:FILE_SUFFIX.zip" 'usage_log.txt' -ir'!test\test-reports\*.log'

    # ===========================================
````

- EN: This section embeds shell commands that perform the actual side effects; controls token scope or environment variables for safe execution.
- CN: 该部分内嵌执行实际副作用的 shell 命令；控制令牌权限范围或环境变量以实现安全执行。

### Lines 102-114 / 第 102-114 行

````yaml
    # S3 upload (primary path)
    # ===========================================
    - name: Store Test Downloaded JSONs on S3
      id: s3-upload-jsons
      uses: seemethere/upload-artifact-s3@v5
      continue-on-error: true
      with:
        s3-bucket: ${{ inputs.s3-bucket }}
        s3-prefix: |
          ${{ github.repository }}/${{ github.run_id }}/${{ github.run_attempt }}/artifact
        retention-days: 14
        if-no-files-found: warn
        path: test-jsons-*.zip
````

- EN: This section reuses actions and step building blocks such as `seemethere/upload-artifact-s3@v5`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 116-131 / 第 116-131 行

````yaml
    - name: Store Test Reports on S3
      id: s3-upload-reports
      uses: seemethere/upload-artifact-s3@v5
      continue-on-error: true
      with:
        s3-bucket: ${{ inputs.s3-bucket }}
        s3-prefix: |
          ${{ github.repository }}/${{ github.run_id }}/${{ github.run_attempt }}/artifact
        retention-days: 14
        if-no-files-found: warn
        path: test-reports-*.zip

    - name: Store Usage Logs on S3
      id: s3-upload-logs
      uses: seemethere/upload-artifact-s3@v5
      continue-on-error: true
````

- EN: This section reuses actions and step building blocks such as `seemethere/upload-artifact-s3@v5`, `seemethere/upload-artifact-s3@v5`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 132-143 / 第 132-143 行

````yaml
      with:
        s3-bucket: ${{ inputs.s3-bucket }}
        s3-prefix: |
          ${{ github.repository }}/${{ github.run_id }}/${{ github.run_attempt }}/artifact
        retention-days: 14
        if-no-files-found: ignore
        path: logs-*.zip

    - name: Store Debug Artifacts on S3
      id: s3-upload-debug
      uses: seemethere/upload-artifact-s3@v5
      continue-on-error: true
````

- EN: This section reuses actions and step building blocks such as `seemethere/upload-artifact-s3@v5`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 144-155 / 第 144-155 行

````yaml
      with:
        s3-bucket: ${{ inputs.s3-bucket }}
        s3-prefix: |
          ${{ github.repository }}/${{ github.run_id }}/${{ github.run_attempt }}/artifact
        retention-days: 14
        if-no-files-found: ignore
        path: debug-*.zip

    - name: Store Profiler Traces on S3
      id: s3-upload-profiler-traces
      uses: seemethere/upload-artifact-s3@v5
      continue-on-error: true
````

- EN: This section reuses actions and step building blocks such as `seemethere/upload-artifact-s3@v5`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 156-167 / 第 156-167 行

````yaml
      with:
        s3-bucket: ${{ inputs.s3-bucket }}
        s3-prefix: |
          ${{ github.repository }}/${{ github.run_id }}/${{ github.run_attempt }}/artifact
        retention-days: 14
        if-no-files-found: ignore
        path: profiler-traces-*.zip

    - name: Store TLParse Output on S3
      id: s3-upload-tlparse
      uses: seemethere/upload-artifact-s3@v5
      continue-on-error: true
````

- EN: This section reuses actions and step building blocks such as `seemethere/upload-artifact-s3@v5`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 168-185 / 第 168-185 行

````yaml
      with:
        s3-bucket: ${{ inputs.s3-bucket }}
        s3-prefix: |
          ${{ github.repository }}/${{ github.run_id }}/${{ github.run_attempt }}/artifact
        retention-days: 14
        if-no-files-found: ignore
        path: tlparse-*.zip

    # Check if S3 upload failed (test-reports is the critical one)
    - name: Check S3 upload status
      id: check-s3
      shell: bash
      run: |
        if [[ "${{ steps.s3-upload-reports.outcome }}" == "failure" ]]; then
          echo "S3 upload failed, will fallback to GHA"
          echo "s3-failed=true" >> "$GITHUB_OUTPUT"
        else
          echo "S3 upload succeeded"
````

- EN: This section embeds shell commands that perform the actual side effects; threads configuration values into later steps.
- CN: 该部分内嵌执行实际副作用的 shell 命令；把配置值传递到后续步骤中。

### Lines 186-201 / 第 186-201 行

````yaml
          echo "s3-failed=false" >> "$GITHUB_OUTPUT"
        fi

    # ===========================================
    # GHA upload (fallback path if S3 failed)
    # ===========================================
    - name: Store Test Downloaded JSONs on Github (fallback)
      uses: actions/upload-artifact@v4
      if: steps.check-s3.outputs.s3-failed == 'true'
      continue-on-error: true
      with:
        # Add the run attempt, see [Artifact run attempt]
        name: test-jsons-runattempt${{ github.run_attempt }}-${{ inputs.file-suffix }}.zip
        retention-days: 14
        if-no-files-found: warn
        path: test/**/*.json
````

- EN: This section reuses actions and step building blocks such as `actions/upload-artifact@v4`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 203-219 / 第 203-219 行

````yaml
    - name: Store Test Reports on Github (fallback)
      uses: actions/upload-artifact@v4
      if: steps.check-s3.outputs.s3-failed == 'true'
      continue-on-error: true
      with:
        # Add the run attempt, see [Artifact run attempt]
        name: test-reports-runattempt${{ github.run_attempt }}-${{ inputs.file-suffix }}.zip
        retention-days: 14
        if-no-files-found: warn
        path: |
          test/**/*.xml
          test/**/*.csv

    - name: Store Usage Logs on Github (fallback)
      uses: actions/upload-artifact@v4
      if: steps.check-s3.outputs.s3-failed == 'true'
      continue-on-error: true
````

- EN: This section reuses actions and step building blocks such as `actions/upload-artifact@v4`, `actions/upload-artifact@v4`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 220-236 / 第 220-236 行

````yaml
      with:
        # Add the run attempt, see [Artifact run attempt]
        name: logs-runattempt${{ github.run_attempt }}-${{ inputs.file-suffix }}.zip
        retention-days: 14
        if-no-files-found: ignore
        path: |
          usage_log.txt
          test/**/*.log

    - name: Store Profiler Traces on Github
      uses: actions/upload-artifact@v4
      continue-on-error: true
      with:
        name: profiler-traces-runattempt${{ github.run_attempt }}-${{ inputs.file-suffix }}
        retention-days: 14
        if-no-files-found: ignore
        path: test/test-reports/profiler_traces/**/*
````

- EN: This section reuses actions and step building blocks such as `actions/upload-artifact@v4`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 238-240 / 第 238-240 行

````yaml
    - name: Store TLParse Output on Github
      uses: actions/upload-artifact@v4
      continue-on-error: true
````

- EN: This section reuses actions and step building blocks such as `actions/upload-artifact@v4`.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程。

### Lines 241-245 / 第 241-245 行

````yaml
      with:
        name: tlparse-runattempt${{ github.run_attempt }}-${{ inputs.file-suffix }}
        retention-days: 14
        if-no-files-found: ignore
        path: test/test-reports/tlparse_output/**/*
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `pytorch/test-infra/.github/actions/setup-uv@main`, `seemethere/upload-artifact-s3@v5`, `actions/upload-artifact@v4`
- Inline commands / 内联命令: `Upload`, `Attempts`, `attempts`, `Suffix`, `workflow`, `set`, `rm`, `import`, ...
- Environment variables / 环境变量: `GHA`, `UV_PYTHON`, `FILE_SUFFIX`, `ZIP_CMD`, `ZIP_DEFLATED`, `GITHUB_OUTPUT`
- Named jobs or sections / 命名作业或章节: `inputs`, `use-gha`, `file-suffix`, `s3-bucket`, `runs`, `steps`
