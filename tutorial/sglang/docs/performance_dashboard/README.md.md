# README.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/performance_dashboard/README.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: A web-based dashboard for visualizing SGLang nightly test performance metrics. Features **Performance Trends**: View throughput, latency, and TTFT trends over time **Model Comparison**: Compare performance across different models and configurations. / 该文档围绕 SGLang 性能 仪表盘 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** A web-based dashboard for visualizing SGLang nightly test performance metrics.
**CN:** 本节围绕 Overview 展开，概述了 test, nightly, metrics, web-based 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Features
**EN:** This content focuses on Features and highlights View, GPU, TTFT, Zoom.
**CN:** 本节围绕 Features 展开，概述了 View, GPU, TTFT, Zoom 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Option 1: Run with Local Server (Recommended)
**EN:** For live data from GitHub Actions artifacts: ``bash # Install requirements pip install requests # Run the server python server.py --fetch-on-start # Visit http://localhost:8000 ` The server provides: - Automatic fetching of metrics from GitHub - Caching to reduce API calls - /api/metrics` endpoint for the frontend
**CN:** 本节围绕 Option 1: Run with Local 服务端 (Recommended) 展开，概述了 GitHub, Run, API, Visit 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Option 2: Fetch Data Manually
**EN:** Use the fetch script to download metrics data: ``bash # Fetch last 30 days of metrics python fetch_metrics.py --output metrics_data.json # Fetch a specific run python fetch_metrics.py --run-id 21338741812 --output single_run.json # Fetch only scheduled (nightly) runs python fetch_metrics.py --scheduled-only --days 7 ``
**CN:** 本节围绕 Option 2: Fetch Data Manually 展开，概述了 Fetch, fetch_metrics.py, days, output 等要点，并说明相关配置、流程、示例或限制条件。

### Section: GitHub Token
**EN:** To download artifacts from GitHub, you need authentication: 1. **Using gh CLI** (recommended): ``bash gh auth login ` 2.
**CN:** 本节围绕 GitHub 令牌 展开，概述了 CLI, GitHub, Without, GITHUB_TOKEN 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Data Structure
**EN:** The metrics JSON has this structure: ``json ] } ] } ``
**CN:** 本节围绕 Data Structure 展开，概述了 TP8, MTP, main, model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: GitHub Pages
**EN:** The dashboard can be deployed to GitHub Pages for public access: 1. Copy the dashboard files to docs/performance_dashboard/ 2.
**CN:** 本节围绕 GitHub Pages 展开，概述了 GitHub, Copy, dashboard, docs/performance_dashboard/ 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Self-Hosted
**EN:** For a self-hosted deployment with live data: 1. Set up a server running server.py 2.
**CN:** 本节围绕 Self-Hosted 展开，概述了 SSL, data, server.py, Configure 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Metrics Explained
**EN:** This content focuses on Metrics Explained and highlights per, Input, Output, tokens.
**CN:** 本节围绕 指标 Explained 展开，概述了 per, Input, Output, tokens 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Contributing
**EN:** To add support for new metrics or visualizations: 1. Update fetch_metrics.py if data collection needs changes 2.
**CN:** 本节围绕 Contributing 展开，概述了 Update, add, new, app.js 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Troubleshooting
**EN:** This content focuses on Troubleshooting and highlights Check, API, GitHub, Try.
**CN:** 本节围绕 Troubleshooting 展开，概述了 Check, API, GitHub, Try 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** GitHub / **CN:** GitHub
- **EN:** data / **CN:** data
- **EN:** metrics / **CN:** 指标
- **EN:** Fetch / **CN:** Fetch
- **EN:** Run / **CN:** Run
- **EN:** API / **CN:** API
- **EN:** fetch_metrics.py / **CN:** fetch_指标.py
- **EN:** MTP / **CN:** MTP

## Dependencies / 依赖关系
- `server.py`
- `fetch_metrics.py`
