# README.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/release_lookup/README.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This tool allows users to find the earliest release that contains a specific PR or commit. It runs entirely in the browser using a static JSON index generated from the git history. / 该文档围绕 SGLang 版本查询 Tool 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This tool allows users to find the earliest release that contains a specific PR or commit. It runs entirely in the browser using a static JSON index generated from the git history.
**CN:** 本节围绕 Overview 展开，概述了 git, tool, find, runs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Usage
**EN:** **Generate the Index**: Run the Python script to generate the release_index.json file from your local git repository. ``bash python3 generate_index.py --output release_index.json ` This script: - Finds all tags matching v* and gateway-v*.
**CN:** 本节围绕 Usage 展开，概述了 Open, release_index.json, Run, Tool 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Files
**EN:** index.html: The UI for the lookup tool. generate_index.py: Script to build the index.
**CN:** 本节围绕 Files 展开，概述了 Script, index.html, generate_index.py, release_index.json 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Logic
**EN:** The tool determines the "earliest release" based on the tag creation date. It traverses tags from oldest to newest.
**CN:** 本节围绕 Logic 展开，概述了 tag, release, reachable, tool 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Open / **CN:** Open
- **EN:** commit / **CN:** commit
- **EN:** release / **CN:** release
- **EN:** release_index.json / **CN:** release_index.json
- **EN:** tag / **CN:** tag
- **EN:** Tool / **CN:** Tool
- **EN:** browser / **CN:** browser
- **EN:** index.html / **CN:** index.html

## Dependencies / 依赖关系
- `generate_index.py`
