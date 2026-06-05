# nightly_builds.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/contributing/ci/nightly_builds.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Describes how vLLM publishes per-commit and release wheel indices at `wheels.vllm.ai`, including backend and architecture variants built in the release pipeline.
- **CN:** 本文说明 vLLM 如何在 `wheels.vllm.ai` 发布按提交和按版本组织的 wheel 索引，并介绍发布流水线中的后端与架构变体。
- **EN:** It also explains index generation, S3 and CloudFront behavior, variant naming, and how `setup.py` uses `metadata.json` to fetch compatible precompiled binaries.
- **CN:** 文档还解释了索引生成、S3 与 CloudFront 的配合方式、变体命名规则，以及 `setup.py` 如何借助 `metadata.json` 获取兼容的预编译二进制。

## Key Concepts / 关键概念
- **EN:** Wheel variants — Wheels are organized by backend and platform, with a default variant in the root and named variants in subdirectories.
- **CN:** Wheel 变体 — wheel 按后端与平台组织，默认变体放在根目录，其余命名变体放在子目录中。
- **EN:** `metadata.json` — The generated metadata file gives machine-readable wheel information used during precompiled installation.
- **CN:** `metadata.json` — 该元数据文件提供机器可读的 wheel 信息，供预编译安装流程使用。
- **EN:** S3 + CloudFront — S3 stores the files while CloudFront rewrites directory-like URLs into PyPI-compatible index paths.
- **CN:** S3 与 CloudFront — S3 负责存储文件，CloudFront 负责把目录式 URL 改写为兼容 PyPI 的索引路径。
- **EN:** Base commit lookup — Precompiled wheel selection is tied to the branch merge-base with upstream `main` to improve compatibility.
- **CN:** 基线提交选择 — 预编译 wheel 的选择会基于当前分支与上游 `main` 的 merge-base，以提高兼容性。
