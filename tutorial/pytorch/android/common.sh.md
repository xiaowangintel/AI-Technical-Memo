# common.sh — Documentation Analysis / 文档分析

## Source / 来源
- File: `android/common.sh`
- Repository: `pytorch`
- Purpose (EN): Provides shared shell automation for Android build workflows.
- 用途 (CN): 为 Android 构建流程提供共享的 Shell 自动化。

## Content Analysis / 内容分析
- EN: This file contains 74 lines and is treated as documentation/configuration for the surrounding PyTorch component.
- CN: 该文件共 74 行，被视为周边 PyTorch 组件的文档/配置内容。
- EN: The script defines reusable shell functions including `retry`, `check_android_sdk`, `check_gradle`, `parse_abis_list`, `build_android`.
- CN: 该脚本定义了可复用的 Shell 函数，包括 `retry`, `check_android_sdk`, `check_gradle`, `parse_abis_list`, `build_android`。
- EN: It relies on environment variables such as `ABIS_LIST`, `ANDROID_BUILD_ROOT`, `ANDROID_HOME`, `BUILD_ROOT`, `CUSTOM_ABIS_LIST`, `GRADLE_PATH`, `INCLUDE_DIR`, `LIB_DIR` to control behavior.
- CN: 它依赖 `ABIS_LIST`, `ANDROID_BUILD_ROOT`, `ANDROID_HOME`, `BUILD_ROOT`, `CUSTOM_ABIS_LIST`, `GRADLE_PATH`, `INCLUDE_DIR`, `LIB_DIR` 等环境变量来控制行为。
- EN: Overall, the content supports this purpose: Provides shared shell automation for Android build workflows.
- CN: 总体来看，这些内容服务于如下目标：为 Android 构建流程提供共享的 Shell 自动化。

## Key Concepts / 关键概念
- Android integration / Android 集成
- Function `retry` / 函数 `retry`
- Function `check_android_sdk` / 函数 `check_android_sdk`
- Function `check_gradle` / 函数 `check_gradle`
- Function `parse_abis_list` / 函数 `parse_abis_list`

## Dependencies / 依赖关系
- Shell functions: `retry`, `check_android_sdk`, `check_gradle`, `parse_abis_list`, `build_android`
- Shell 函数: `retry`, `check_android_sdk`, `check_gradle`, `parse_abis_list`, `build_android`
- Environment variables: `ABIS_LIST`, `ANDROID_BUILD_ROOT`, `ANDROID_HOME`, `BUILD_ROOT`, `CUSTOM_ABIS_LIST`, `GRADLE_PATH`, `INCLUDE_DIR`, `LIB_DIR`, `PYTORCH_ANDROID_DIR`, `PYTORCH_DIR`
- 环境变量: `ABIS_LIST`, `ANDROID_BUILD_ROOT`, `ANDROID_HOME`, `BUILD_ROOT`, `CUSTOM_ABIS_LIST`, `GRADLE_PATH`, `INCLUDE_DIR`, `LIB_DIR`, `PYTORCH_ANDROID_DIR`, `PYTORCH_DIR`
