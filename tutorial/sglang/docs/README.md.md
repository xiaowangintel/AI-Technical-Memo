# README.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/README.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This is the documentation website for the SGLang project (https://github.com/sgl-project/sglang). We recommend new contributors start from writing documentation, which helps you quickly understand SGLang codebase. / 该文档围绕 SGLang Documentation 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This is the documentation website for the SGLang project (https://github.com/sgl-project/sglang). We recommend new contributors start from writing documentation, which helps you quickly understand SGLang codebase.
**CN:** 本节围绕 Overview 展开，概述了 Most, docs/, new, start 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Install Dependency
**EN:** **Linux:** ``bash apt-get update && apt-get install -y pandoc parallel retry pip install -r requirements.txt ` **macOS:** `bash brew install pandoc parallel retry pip install -r requirements.txt ``
**CN:** 本节围绕 Install Dependency 展开，概述了 install, pip, Linux, retry 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Update Documentation
**EN:** Update your Jupyter notebooks in the appropriate subdirectories under docs/. If you add new files, remember to update index.rst (or relevant .rst files) accordingly.
**CN:** 本节围绕 Update Documentation 展开，概述了 make, files, serve, checks 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Documentation Style Guidelines
**EN:** For common functionalities, we prefer **Jupyter Notebooks** over Markdown so that all examples can be executed and validated by our docs CI pipeline. For complex features (e.g., distributed serving), Markdown is preferred.
**CN:** 本节围绕 Documentation Style Guidelines 展开，概述了 e.g, time, server, reduce 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Documentation Build, Deployment, and CI
**EN:** The SGLang documentation pipeline is based on **Sphinx** and supports rendering Jupyter notebooks (.ipynb) into HTML/Markdown for web display. Detailed logits can be found in the Makefile.
**CN:** 本节围绕 Documentation Build, 部署, and CI 展开，概述了 Makefile, ipynb, Sphinx, Jupyter 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Notebook Execution (make compile)
**EN:** This content focuses on Notebook Execution (make compile) and highlights Executes, notebooks, ipynb, docs/.
**CN:** 本节围绕 Notebook Execution (make compile) 展开，概述了 Executes, notebooks, ipynb, docs/ 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Web Rendering (make html)
**EN:** After compilation, Sphinx builds the website: * Reads Markdown, reStructuredText, and Jupyter notebooks * Renders them into HTML pages * Outputs the website into: `` docs/_build/html/ `` This directory is the source for online documentation hosting.
**CN:** 本节围绕 Web Rendering (make html) 展开，概述了 After, Sphinx, Jupyter, Renders 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Markdown Export (make markdown)
**EN:** This content focuses on Markdown Export (make markdown) and highlights Does, make, Scans, target.
**CN:** 本节围绕 Markdown Export (make markdown) 展开，概述了 Does, make, Scans, target 等要点，并说明相关配置、流程、示例或限制条件。

### Section: CI Execution
**EN:** In other words, sgl-project.io is push-only. All the changes of SGLang docs should be made directly in SGLang main repo, then push to the sgl-project.io.
**CN:** 本节围绕 CI Execution 展开，概述了 make, sgl-project.io, Then, results 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** make / **CN:** make
- **EN:** Jupyter / **CN:** Jupyter
- **EN:** notebooks / **CN:** notebooks
- **EN:** files / **CN:** files
- **EN:** time / **CN:** time
- **EN:** docs/ / **CN:** docs/
- **EN:** ipynb / **CN:** ipynb
- **EN:** Makefile / **CN:** Makefile

## Dependencies / 依赖关系
- `index.rst`
- `../get_started/install.md`
- `./Makefile`
- `docs/_build/html/markdown/advanced_features/lora.md`
- `//github.com/sgl-project/sglang/blob/main/.github/workflows/release-docs.yml`
