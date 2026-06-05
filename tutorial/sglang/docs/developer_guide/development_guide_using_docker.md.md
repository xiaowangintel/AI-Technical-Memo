# development_guide_using_docker.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/developer_guide/development_guide_using_docker.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Setup VSCode on a Remote Host (Optional - you can skip this step if you plan to run sglang dev container locally) In the remote host, download code from Https://code.visualstudio.com/docs/?dv=linux64cli and run code tunnel in a shell. / 该文档围绕 Development Guide Using Docker 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Setup VSCode on a Remote Host
**EN:** (Optional - you can skip this step if you plan to run sglang dev container locally) 1. In the remote host, download code from Https://code.visualstudio.com/docs/?dv=linux64cli and run code tunnel in a shell.
**CN:** 本节围绕 Setup VSCode on a Remote Host 展开，概述了 code, run, VSCode, Tunnel 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Option 1. Use the default dev container automatically from VSCode
**EN:** There is a .devcontainer folder in the sglang repository root folder to allow VSCode to automatically start up within dev container. You can read more about this VSCode extension in VSCode official document Developing inside a Container.
**CN:** 本节围绕 Option 1. Use the default dev container automatically from VSCode 展开，概述了 VSCode, Container, dev, Developing 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Option 2. Start up containers manually (advanced)
**EN:** The following startup command is an example for internal development by the SGLang team. You can **modify or add directory mappings as needed**, especially for model weight downloads, to prevent repeated downloads by different Docker containers.
**CN:** 本节围绕 Option 2. Start up containers manually (advanced) 展开，概述了 host, docker, RDMA, bin/zsh 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Debug SGLang with VSCode Debugger
**EN:** (Create if not exist) open launch.json in VSCode. Add the following config and save.
**CN:** 本节围绕 Debug SGLang with VSCode Debugger 展开，概述了 VSCode, program, Add, host 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Profile
**EN:** ``bash # Change batch size, input, output and add disable-cuda-graph (for easier analysis) # e.g.
**CN:** 本节围绕 Profile 展开，概述了 input, Change, output, DeepSeek 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Evaluation
**EN:** gsm8k 8 shot python3 benchmark/gsm8k/bench_sglang.py --num-questions 2000 --parallel 2000 --num-shots 8 ``
**CN:** 本节围绕 Evaluation 展开，概述了 e.g, shot, gsm8k, python3 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** VSCode / **CN:** VSCode
- **EN:** host / **CN:** host
- **EN:** dev / **CN:** dev
- **EN:** docker / **CN:** docker
- **EN:** Container / **CN:** Container
- **EN:** code / **CN:** code
- **EN:** local / **CN:** local
- **EN:** run / **CN:** run

## Dependencies / 依赖关系
- `benchmark/gsm8k/bench_sglang.py`
