# docs_migration_plan.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs_new/docs_migration_plan.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Background Migrate the new Mintlify-based documentation (currently in the standalone sgl-docs repo) into the sglang main repo under docs_new/, and point staging.docs.sglang.io to it. / 该文档围绕 SGLang Documentation Migration Plan 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Background
**EN:** Migrate the new Mintlify-based documentation (currently in the standalone sgl-docs repo) into the sglang main repo under docs_new/, and point staging.docs.sglang.io to it.
**CN:** 本节围绕 Background 展开，概述了 repo, Migrate, sgl-docs, docs_new/ 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Current State
**EN:** This section provides a comparison table for Current State, covering columns such as Item, Location, Stack and examples such as Old docs, New docs + cookbook.
**CN:** 本节围绕 Current State 展开，概述了 Old, New, Item, repo 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Phase 1: Git Subtree Merge (Local Experiment)
**EN:** > Goal: Merge sgl-docs into sglang repo's docs_new/ directory, preserving full commit history and authorship. Create a new branch (sglang remote is NOT affected) cd /path/to/sglang git checkout -b docs-new-migration # 2.
**CN:** 本节围绕 Phase 1: Git Subtree Merge (Local Experiment) 展开，概述了 sgl-docs, git, remote, docs_new/ 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Safety Guarantees
**EN:** sgl-docs original repo: **unaffected** (fetch only, no push) - sglang remote: **unaffected** (local branch, no push until ready) - Rollback: git checkout main && git branch -D docs-new-migration
**CN:** 本节围绕 Safety Guarantees 展开，概述了 git, push, branch, sgl-docs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Side Effect: Contributors
**EN:** git subtree add (without --squash) imports all original commits. Authors from sgl-docs will appear in sglang's git history and GitHub Contributors list.
**CN:** 本节围绕 Side Effect: Contributors 展开，概述了 git, Authors, --squash, sgl-docs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Phase 2: Configure Mintlify for docs_new/ (on branch)
**EN:** > Goal: Make Mintlify read from sglang repo's docs_new/ subdirectory instead of the standalone sgl-docs repo. **No need to merge to main first** — Mintlify can point to a specific branch for validation.
**CN:** 本节围绕 Phase 2: Configure Mintlify for docs_new/ (on branch) 展开，概述了 Mintlify, docs_new/, Log, Goal 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Phase 3: DNS & Custom Domain for staging.docs.sglang.io
**EN:** > Goal: Make staging.docs.sglang.io serve the new Mintlify docs. **DNS**: Add a CNAME record for staging.docs.sglang.io pointing to Mintlify's endpoint (typically cname.mintlify.dev) 2.
**CN:** 本节围绕 Phase 3: DNS & Custom Domain for staging.docs.sglang.io 展开，概述了 staging.docs.sglang.io, Mintlify, DNS, Add 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Phase 4: Ongoing Sync During Migration Period
**EN:** > During the transition, sgl-docs may still receive updates. Sync them into docs_new/ as needed.
**CN:** 本节围绕 Phase 4: Ongoing Sync During Migration Period 展开，概述了 sgl-docs, docs_new/, Sync, Pull 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Phase 5: CI/CD (Optional, Post-Migration)
**EN:** Current docs/** CI workflows will **NOT** trigger for docs_new/** changes. This is fine initially since Mintlify has its own GitHub integration for auto-deployment on push to main.
**CN:** 本节围绕 Phase 5: CI/CD (Optional, Post-Migration) 展开，概述了 Mintlify, Link, GitHub, Current 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Phase 6: Final Cutover
**EN:** This section provides a comparison table for Phase 6: Final Cutover, covering columns such as Stage, docs.sglang.io, staging.docs.sglang.io and examples such as After Phase 3, After cutover.
**CN:** 本节围绕 Phase 6: Final Cutover 展开，概述了 Mintlify, docs.sglang.io, Sphinx, Remove 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Execution Order
**EN:** This section provides a comparison table for Execution Order, covering columns such as Step, Action, Who and examples such as 1, 2, 3, 4.
**CN:** 本节围绕 Execution Order 展开，概述了 Step, Phase, Mintlify, Dev 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Mintlify / **CN:** Mintlify
- **EN:** sgl-docs / **CN:** sgl-docs
- **EN:** Phase / **CN:** Phase
- **EN:** docs_new/ / **CN:** docs_new/
- **EN:** staging.docs.sglang.io / **CN:** staging.docs.sglang.io
- **EN:** Step / **CN:** Step
- **EN:** git / **CN:** git
- **EN:** repo / **CN:** repo

## Dependencies / 依赖关系
- `execute-notebook.yml`
- `lint.yml`
- `.github/workflows/release-docs.yml`
- `.github/workflows/execute-notebook.yml`
