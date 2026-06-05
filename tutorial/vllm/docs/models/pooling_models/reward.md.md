# reward.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/pooling_models/reward.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page frames reward models as preference or quality evaluators and separates them into sequence reward, token reward, and process reward variants.
  **CN:** 本页将 reward 模型定位为偏好或质量评估器，并区分为序列 reward、token reward 与 process reward 三类。
- **EN:** Its practical focus is on choosing the right pooling task for `LLM.encode` or `/pooling`, plus special configuration needed by process reward models.
  **CN:** 它的实践重点是为 `LLM.encode` 或 `/pooling` 选择正确的 pooling task，并补充 process reward 模型所需的特殊配置。

## Key Concepts / 关键概念
- **EN:** Sequence reward models use `pooling_task="classify"`; token/process reward models use `token_classify`.
  **CN:** 序列 reward 模型使用 `pooling_task="classify"`；token/process reward 模型使用 `token_classify`。
- **EN:** Process reward models may require explicit `STEP` pooling configuration and token IDs.
  **CN:** process reward 模型可能需要显式的 `STEP` pooling 配置与特定 token ID。
- **EN:** `LLM.reward` is deprecated in favor of `LLM.encode` with the correct pooling task.
  **CN:** `LLM.reward` 已废弃，推荐改用带正确 pooling task 的 `LLM.encode`。
