# plugin.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/platforms/plugin.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Overview Allows hardware vendors and developers to extend SGLang **without modifying the main repository code**. The framework provides two plugin types, both discovered via Python's standard setuptools entry_points: Principles. / 该文档围绕 SGLang Plugin System 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This section provides a comparison table for Overview, covering columns such as Plugin Type, Entry Point Group, Purpose and examples such as **Hardware Platform Plugin**, **General Plugin**.
**CN:** 本节围绕 Overview 展开，概述了 Plugin, Allows, Inject, Purpose 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Principles
**EN:** **Non-intrusive**: Existing CUDA/ROCm/NPU/XPU code remains unchanged. OOT code paths are added alongside existing hardware-specific logic.
**CN:** 本节围绕 Principles 展开，概述了 code, SGLANG_PLUGINS, SGLANG_PLATFORM, OOT 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Current Scope & Future Direction
**EN:** The plugin system currently targets **out-of-tree (OOT) hardware platforms** — enabling new devices to integrate with SGLang without any changes to the main repository.
**CN:** 本节围绕 Current Scope & Future Direction 展开，概述了 hardware, plugin, OOT, NPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Platform Hierarchy
**EN:** This content focuses on Platform Hierarchy and highlights DeviceMixin, SRTPlatform, OOT, operations.
**CN:** 本节围绕 Platform Hierarchy 展开，概述了 DeviceMixin, SRTPlatform, OOT, operations 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Platform Discovery (current_platform)
**EN:** current_platform is a **lazy singleton** in sglang.srt.platforms.
**CN:** 本节围绕 Platform Discovery (current_platform) 展开，概述了 RuntimeError, SGLANG_PLATFORM, activate, activated 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Plugin Loading Flow
**EN:** This section provides a comparison table for Plugin Loading Flow, covering columns such as Call Site, Process, Timing and examples such as cli/serve.py serve(), launch_server.py __main__, engine.py _launch_subprocesses(), scheduler.py run_scheduler_process().
**CN:** 本节围绕 Plugin Loading Flow 展开，概述了 Before, Main, load_plugins, prepare_server_args 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Description
**EN:** A hardware platform plugin registers an SRTPlatform subclass that tells SGLang how to interact with a specific hardware backend.
**CN:** 本节围绕 Description 展开，概述了 SRTPlatform, hardware, tells, plugin 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quick Start
**EN:** Create a minimal package:** `` my_platform_plugin/ ├── pyproject.toml └── my_platform_plugin/ ├── __init__.py # activate() function ├── device.py # MyDeviceMixin └── platform.py # MySRTPlatform ` **2.
**CN:** 本节围绕 Quick Start 展开，概述了 def, self, MyDeviceMixin, MySRTPlatform 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Platform Interface Reference
**EN:** This section provides a comparison table for Platform Interface Reference, covering columns such as Method, Default, Description and examples such as is_cuda(), is_rocm(), is_npu(), is_cpu().
**CN:** 本节围绕 Platform Interface Reference 展开，概述了 NotImplementedError, Planned, Whether, raise 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Environment Variables
**EN:** This section provides a comparison table for Environment Variables, covering columns such as Variable, Description and examples such as SGLANG_PLATFORM, SGLANG_PLUGINS.
**CN:** 本节围绕 Environment Variables 展开，概述了 plugin, plugins, sglang.srt.plugins, general 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Description
**EN:** General function plugins inject behavior into sglang **without requiring a custom platform**.
**CN:** 本节围绕 Description 展开，概述了 Add, function, Modify, General 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quick Start
**EN:** Create a minimal package:** `` my_general_plugin/ ├── pyproject.toml └── my_general_plugin/ └── __init__.py # register() function ` **2.
**CN:** 本节围绕 Quick Start 展开，概述了 Scheduler, register, my_general_plugin/, def 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Hook Types
**EN:** This section provides a comparison table for Hook Types, covering columns such as Hook Type, Signature, Description and examples such as **BEFORE**, **AFTER**, **AROUND**, **REPLACE**.
**CN:** 本节围绕 Hook Types 展开，概述了 args, class, kwargs, AFTER 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Registration API
**EN:** This content focuses on Registration API and highlights API, args, kwargs, HookType.
**CN:** 本节围绕 Registration API 展开，概述了 API, args, kwargs, HookType 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Hook Target Resolution
**EN:** Target paths use fully-qualified dotted notation. Both formats are supported: - **Dotted**: sglang.srt.managers.scheduler.Scheduler.__init__ - **Entry-points style**: sglang.srt.managers.scheduler:Scheduler.__init__ (colon treated as dot)
**CN:** 本节围绕 Hook Target Resolution 展开，概述了 Both, Target, Dotted, Entry-points 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Common Hook Targets
**EN:** This section provides a comparison table for Common Hook Targets, covering columns such as Target, Description and examples such as sglang.srt.server_args.ServerArgs.add_cli_args, sglang.srt.server_args.ServerArgs.__post_init__, sglang.srt.server_args.ServerArgs.check_server_args, sglang.srt.managers.scheduler.Scheduler.__init__.
**CN:** 本节围绕 Common Hook Targets 展开，概述了 Custom, Add, CLI, state 等要点，并说明相关配置、流程、示例或限制条件。

### Section: File Reference
**EN:** This section provides a comparison table for File Reference, covering columns such as File, Description and examples such as sglang/srt/platforms/device_mixin.py, sglang/srt/platforms/interface.py, sglang/srt/platforms/__init__.py, sglang/srt/plugins/__init__.py.
**CN:** 本节围绕 File Reference 展开，概述了 DeviceMixin, HookType, SRTPlatform, PlatformEnum 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** NotImplementedError / **CN:** NotImplementedError
- **EN:** Planned / **CN:** Planned
- **EN:** SRTPlatform / **CN:** SRTPlatform
- **EN:** DeviceMixin / **CN:** DeviceMixin
- **EN:** Description / **CN:** Description
- **EN:** class / **CN:** class
- **EN:** device / **CN:** device
- **EN:** platform / **CN:** platform

## Dependencies / 依赖关系
- `cli/serve.py`
- `launch_server.py`
- `engine.py`
- `scheduler.py`
- `__init__.py`
- `device.py`
- `platform.py`
- `sglang/srt/platforms/device_mixin.py`
- `sglang/srt/platforms/interface.py`
- `sglang/srt/platforms/__init__.py`
- `sglang/srt/plugins/__init__.py`
- `sglang/srt/plugins/hook_registry.py`
