# action.yml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/actions/ecr-login/action.yml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Defines a reusable composite GitHub Action, including declared inputs, step composition, and shell execution.
- **用途 (CN)**: 定义可复用的复合型 GitHub Action，包括输入声明、步骤组合以及 shell 执行逻辑。

## Content Analysis / 内容分析
### Lines 1-11 / 第 1-11 行

````yaml
name: Login to ECR via OIDC

description: |
  Configures AWS credentials via OIDC and logs into Amazon ECR.
  Works uniformly on all runner types (AWS EC2, GCP, ROCm, b200, etc.)

inputs:
  aws-role-to-assume:
    description: IAM role to assume via OIDC
    required: false
    default: arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 12-23 / 第 12-23 行

````yaml
  aws-region:
    description: AWS region
    required: false
    default: us-east-1
  role-duration-seconds:
    description: Duration for the assumed role session
    required: false
    default: "18000"
  use-iam-profile-if-available:
    description: If true, will check for and use an IAM instance profile if available.
    required: false
    default: 'true'
````

- EN: This section describes repository automation behavior for `.github/actions/ecr-login/action.yml`.
- CN: 该部分描述 `.github/actions/ecr-login/action.yml` 的仓库自动化行为。

### Lines 25-38 / 第 25-38 行

````yaml
outputs:
  auth-method:
    description: Authentication method used (oidc or instance-profile)
    value: ${{ steps.check_env.outputs.has_profile == 'true' && 'instance-profile' || 'oidc' }}

runs:
  using: composite
  steps:
    - name: Check for IAM instance profile
      id: check_env
      if: inputs.use-iam-profile-if-available == 'true'
      shell: bash
      run: |
        echo "::group::Checking for IAM instance profile"
````

- EN: This section reuses actions and step building blocks such as named actions; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 39-52 / 第 39-52 行

````yaml
        echo "Checking EC2 metadata endpoint for IAM instance profile..."

        # Try IMDSv2 first (more secure, required on some instances)
        echo "Attempting IMDSv2..."
        TOKEN=$(curl -sf -X PUT "http://169.254.169.254/latest/api/token" \
          -H "X-aws-ec2-metadata-token-ttl-seconds: 60" \
          --connect-timeout 2 2>/dev/null) || TOKEN=""

        if [[ -n "$TOKEN" ]]; then
          echo "IMDSv2 token obtained, checking IAM info..."
          IAM_INFO=$(curl -sf -H "X-aws-ec2-metadata-token: $TOKEN" \
            "http://169.254.169.254/latest/meta-data/iam/info" \
            --connect-timeout 2 2>/dev/null) || IAM_INFO=""
        else
````

- EN: This section describes repository automation behavior for `.github/actions/ecr-login/action.yml`.
- CN: 该部分描述 `.github/actions/ecr-login/action.yml` 的仓库自动化行为。

### Lines 53-66 / 第 53-66 行

````yaml
          echo "IMDSv2 not available, trying IMDSv1..."
          IAM_INFO=$(curl -sf --connect-timeout 2 \
            "http://169.254.169.254/latest/meta-data/iam/info" 2>/dev/null) || IAM_INFO=""
        fi

        if [[ -n "$IAM_INFO" ]]; then
          echo "IAM instance profile detected:"
          echo "$IAM_INFO" | head -5
          echo "has_profile=true" >> "$GITHUB_OUTPUT"
        else
          echo "No IAM instance profile found (not on EC2, or no profile attached)"
          echo "has_profile=false" >> "$GITHUB_OUTPUT"
        fi
        echo "::endgroup::"
````

- EN: This section describes repository automation behavior for `.github/actions/ecr-login/action.yml`.
- CN: 该部分描述 `.github/actions/ecr-login/action.yml` 的仓库自动化行为。

### Lines 68-77 / 第 68-77 行

````yaml
    - name: Configure AWS credentials via OIDC
      id: configure-oidc
      if: inputs.use-iam-profile-if-available == 'false' || steps.check_env.outputs.has_profile == 'false'
      uses: aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722 # v4.1.0
      with:
        role-to-assume: ${{ inputs.aws-role-to-assume != '' && inputs.aws-role-to-assume || 'arn:aws:iam::308535385114:role/gha_workflow_s3_and_ecr_read_only' }}
        role-session-name: gha-ecr-login
        aws-region: ${{ inputs.aws-region }}
        role-duration-seconds: ${{ inputs.role-duration-seconds }}
        role-skip-session-tagging: true
````

- EN: This section reuses actions and step building blocks such as `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`; threads configuration values into later steps.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；把配置值传递到后续步骤中。

### Lines 79-86 / 第 79-86 行

````yaml
    - name: Log authentication method
      shell: bash
      run: |
        if [[ "${{ steps.check_env.outputs.has_profile }}" == "true" ]]; then
          echo "✓ Using IAM instance profile for authentication"
        else
          echo "✓ Using OIDC for authentication"
        fi
````

- EN: This section embeds shell commands that perform the actual side effects.
- CN: 该部分内嵌执行实际副作用的 shell 命令。

### Lines 88-99 / 第 88-99 行

````yaml
    - name: Set AWS region for instance profile auth
      if: steps.check_env.outputs.has_profile == 'true'
      shell: bash
      run: |
        echo "Setting AWS_REGION=${{ inputs.aws-region }} for instance profile auth"
        echo "AWS_REGION=${{ inputs.aws-region }}" >> "$GITHUB_ENV"
        echo "AWS_DEFAULT_REGION=${{ inputs.aws-region }}" >> "$GITHUB_ENV"

    # Primary ECR login - works with both OIDC and instance profile credentials
    - name: Log in to ECR
      id: ecr-login
      uses: aws-actions/amazon-ecr-login@062b18b96a7aff071d4dc91bc00c4c1a7945b076 # v2.0.1
````

- EN: This section reuses actions and step building blocks such as `aws-actions/amazon-ecr-login@062b18b96a7aff071d4dc91bc00c4c1a7945b076`; embeds shell commands that perform the actual side effects.
- CN: 该部分复用 Action 与步骤构件来拼装自动化流程；内嵌执行实际副作用的 shell 命令。

### Lines 100-102 / 第 100-102 行

````yaml
      with:
        registries: 308535385114  # META_AWS_ACCOUNT_ID
        aws-region: ${{ inputs.aws-region }}
````

- EN: This section threads configuration values into later steps.
- CN: 该部分把配置值传递到后续步骤中。

### Lines 104-107 / 第 104-107 行

````yaml
    - name: Verify ECR login
      shell: bash
      run: |
        echo "✓ Successfully logged into ECR (registry: 308535385114)"
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

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: `aws-actions/configure-aws-credentials@ececac1a45f3b08a01d2dd070d28d111c5fe6722`, `aws-actions/amazon-ecr-login@062b18b96a7aff071d4dc91bc00c4c1a7945b076`
- Inline commands / 内联命令: `Configures`, `Works`, `echo`, `H`, `connect-timeout`, `"http://169.254.169.254/latest/meta-data/iam/info"`
- Environment variables / 环境变量: `ECR`, `OIDC`, `AWS`, `EC2`, `GCP`, `IAM`, `TOKEN`, `PUT`, `IAM_INFO`, `GITHUB_OUTPUT`, ...
- Named jobs or sections / 命名作业或章节: `inputs`, `aws-role-to-assume`, `aws-region`, `role-duration-seconds`, `use-iam-profile-if-available`, `outputs`, `auth-method`, `runs`, `steps`
