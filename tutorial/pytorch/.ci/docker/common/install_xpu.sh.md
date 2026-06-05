# install_xpu.sh — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `.ci/docker/common/install_xpu.sh`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Automates repository maintenance, build, test, packaging, or release tasks through shell commands. The opening comment frames the file as: "!/bin/bash."
- **Purpose (CN)**: 通过 shell 命令自动化执行仓库维护、构建、测试、打包或发布任务。 开头注释将该文件概括为：“!/bin/bash”。

## Content Analysis / 内容分析

### Lines 1-8 / 第 1-8 行

```bash
#!/bin/bash
set -xe
# Script used in CI and CD pipeline

# Intel® software for general purpose GPU capabilities.
# Refer to https://www.intel.com/content/www/us/en/developer/articles/tool/pytorch-prerequisites-for-intel-gpus.html

# Users should update to the latest version as it becomes available
```

- **EN:** This chunk introduces sections such as !/bin/bash, Script used in CI and CD pipeline, Intel® software for general purpose GPU capabilities., Refer to https://www.intel.com/content/www/us/en/developer/articles/tool/pytorch-prerequisites-for-intel-gpus.html, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 !/bin/bash、Script used in CI and CD pipeline、Intel® software for general purpose GPU capabilities.、Refer to https://www.intel.com/content/www/us/en/developer/articles/tool/pytorch-prerequisites-for-intel-gpus.html 等标题组织周边说明或配置。
- **EN:** Environment variables such as GPU communicate required tool locations or behavioral switches.
- **CN:** GPU 等环境变量用于说明所需工具位置或行为开关。

### Lines 9-16 / 第 9-16 行

```bash

function install_ubuntu() {
    . /etc/os-release
    if [[ ! " jammy noble " =~ " ${VERSION_CODENAME} " ]]; then
        echo "Ubuntu version ${VERSION_CODENAME} not supported"
        exit
    fi

```

- **EN:** The script defines shell helpers such as install_ubuntu to structure repeated tasks.
- **CN:** 脚本定义了 install_ubuntu 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as ., exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 .、exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as VERSION_CODENAME communicate required tool locations or behavioral switches.
- **CN:** VERSION_CODENAME 等环境变量用于说明所需工具位置或行为开关。

### Lines 17-28 / 第 17-28 行

```bash
    apt-get update -y
    apt-get install -y gpg-agent wget

    if [[ "${XPU_DRIVER_TYPE,,}" == "client" ]]; then
        apt-get install -y software-properties-common
        add-apt-repository -y ppa:kobuk-team/intel-graphics
        apt-get install -y \
            libze-intel-gpu1 libze1 intel-metrics-discovery intel-opencl-icd clinfo intel-gsc \
            intel-media-va-driver-non-free libmfx-gen1 libvpl2 libvpl-tools libva-glx2 va-driver-all vainfo \
            libze-dev intel-ocloc xpu-smi
    else
        # To add the online network package repository for the GPU Driver
```

- **EN:** This chunk introduces sections such as To add the online network package repository for the GPU Driver, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 To add the online network package repository for the GPU Driver 等标题组织周边说明或配置。
- **EN:** It invokes commands such as apt-get, add-apt-repository, libze-intel-gpu1, intel-media-va-driver-non-free, libze-dev, showing the operational steps the workflow performs.
- **CN:** 它调用了 apt-get、add-apt-repository、libze-intel-gpu1、intel-media-va-driver-non-free、libze-dev 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as XPU_DRIVER_TYPE, GPU communicate required tool locations or behavioral switches.
- **CN:** XPU_DRIVER_TYPE、GPU 等环境变量用于说明所需工具位置或行为开关。

### Lines 29-37 / 第 29-37 行

```bash
        wget -qO - https://repositories.intel.com/gpu/intel-graphics.key \
            | gpg --yes --dearmor --output /usr/share/keyrings/intel-graphics.gpg
        echo "deb [arch=amd64 signed-by=/usr/share/keyrings/intel-graphics.gpg] \
            https://repositories.intel.com/gpu/ubuntu ${VERSION_CODENAME}${XPU_DRIVER_VERSION} unified" \
            | tee /etc/apt/sources.list.d/intel-gpu-${VERSION_CODENAME}.list

        # Update the packages list and repository index
        apt-get update

```

- **EN:** This chunk introduces sections such as Update the packages list and repository index, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Update the packages list and repository index 等标题组织周边说明或配置。
- **EN:** It invokes commands such as wget, https, apt-get, showing the operational steps the workflow performs.
- **CN:** 它调用了 wget、https、apt-get 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as VERSION_CODENAME, XPU_DRIVER_VERSION communicate required tool locations or behavioral switches.
- **CN:** VERSION_CODENAME、XPU_DRIVER_VERSION 等环境变量用于说明所需工具位置或行为开关。

### Lines 38-53 / 第 38-53 行

```bash
        # The xpu-smi packages
        apt-get install -y flex bison xpu-smi

        # Compute and Media Runtimes
        if [[ " ${VERSION_CODENAME} " =~ " noble " ]]; then
            apt-get install -y \
                intel-opencl-icd libze-intel-gpu1 libze1 \
                intel-media-va-driver-non-free libmfx-gen1 libvpl2 \
                libegl-mesa0 libegl1-mesa-dev libgbm1 libgl1-mesa-dev libgl1-mesa-dri \
                libglapi-mesa libgles2-mesa-dev libglx-mesa0 libigdgmm12 libxatracker2 mesa-va-drivers \
                mesa-vdpau-drivers mesa-vulkan-drivers va-driver-all vainfo hwinfo clinfo intel-ocloc
        else # jammy
            apt-get install -y \
                intel-opencl-icd libze-intel-gpu1 libze1 \
                intel-media-va-driver-non-free libmfx-gen1 libvpl2 \
                libegl-mesa0 libegl1-mesa libegl1-mesa-dev libgbm1 libgl1-mesa-dev libgl1-mesa-dri \
```

- **EN:** This chunk introduces sections such as The xpu-smi packages, Compute and Media Runtimes, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 The xpu-smi packages、Compute and Media Runtimes 等标题组织周边说明或配置。
- **EN:** It invokes commands such as apt-get, intel-opencl-icd, intel-media-va-driver-non-free, libegl-mesa0, libglapi-mesa, mesa-vdpau-drivers, showing the operational steps the workflow performs.
- **CN:** 它调用了 apt-get、intel-opencl-icd、intel-media-va-driver-non-free、libegl-mesa0、libglapi-mesa、mesa-vdpau-drivers 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as VERSION_CODENAME communicate required tool locations or behavioral switches.
- **CN:** VERSION_CODENAME 等环境变量用于说明所需工具位置或行为开关。

### Lines 54-61 / 第 54-61 行

```bash
                libglapi-mesa libglx-mesa0 libigdgmm12 libxatracker2 mesa-va-drivers \
                mesa-vdpau-drivers mesa-vulkan-drivers va-driver-all vainfo hwinfo clinfo intel-ocloc
        fi
        # Development Packages
        apt-get install -y libigc-dev intel-igc-cm libigdfcl-dev libigfxcmrt-dev libze-dev
    fi

    # Cleanup
```

- **EN:** This chunk introduces sections such as Development Packages, Cleanup, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Development Packages、Cleanup 等标题组织周边说明或配置。
- **EN:** It invokes commands such as libglapi-mesa, mesa-vdpau-drivers, apt-get, showing the operational steps the workflow performs.
- **CN:** 它调用了 libglapi-mesa、mesa-vdpau-drivers、apt-get 等命令，展示该工作流执行的操作步骤。

### Lines 62-72 / 第 62-72 行

```bash
    apt-get autoclean && apt-get clean
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
}

function install_rhel() {
    . /etc/os-release
    if [[ ! " 8.8 8.10 9.0 9.2 9.3 " =~ " ${VERSION_ID} " ]]; then
        echo "RHEL version ${VERSION_ID} not supported"
        exit
    fi
    # Using testing channel for CD build
```

- **EN:** This chunk introduces sections such as Using testing channel for CD build, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Using testing channel for CD build 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as install_rhel to structure repeated tasks.
- **CN:** 脚本定义了 install_rhel 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as apt-get, rm, ., exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 apt-get、rm、.、exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as VERSION_ID, RHEL communicate required tool locations or behavioral switches.
- **CN:** VERSION_ID、RHEL 等环境变量用于说明所需工具位置或行为开关。

### Lines 73-81 / 第 73-81 行

```bash
    if [[ "${ID}" == "almalinux" ]]; then
        XPU_DRIVER_VERSION="/testing"
    fi

    dnf install -y 'dnf-command(config-manager)'
    # To add the online network package repository for the GPU Driver
    dnf config-manager --add-repo \
        https://repositories.intel.com/gpu/rhel/${VERSION_ID}${XPU_DRIVER_VERSION}/unified/intel-gpu-${VERSION_ID}.repo

```

- **EN:** This chunk introduces sections such as To add the online network package repository for the GPU Driver, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 To add the online network package repository for the GPU Driver 等标题组织周边说明或配置。
- **EN:** It invokes commands such as dnf, https, showing the operational steps the workflow performs.
- **CN:** 它调用了 dnf、https 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as XPU_DRIVER_VERSION, GPU, VERSION_ID communicate required tool locations or behavioral switches.
- **CN:** XPU_DRIVER_VERSION、GPU、VERSION_ID 等环境变量用于说明所需工具位置或行为开关。

### Lines 82-92 / 第 82-92 行

```bash
    # The xpu-smi packages
    dnf install -y xpu-smi
    # Compute and Media Runtimes
    dnf install --skip-broken -y \
        intel-opencl intel-media intel-mediasdk libmfxgen1 libvpl2\
        level-zero intel-level-zero-gpu mesa-dri-drivers mesa-vulkan-drivers \
        mesa-vdpau-drivers libdrm mesa-libEGL mesa-libgbm mesa-libGL \
        mesa-libxatracker libvpl-tools intel-metrics-discovery \
        intel-metrics-library intel-igc-core intel-igc-cm \
        libva libva-utils intel-gmmlib libmetee intel-gsc intel-ocloc
    # Development packages
```

- **EN:** This chunk introduces sections such as The xpu-smi packages, Compute and Media Runtimes, Development packages, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 The xpu-smi packages、Compute and Media Runtimes、Development packages 等标题组织周边说明或配置。
- **EN:** It invokes commands such as dnf, intel-opencl, level-zero, mesa-vdpau-drivers, mesa-libxatracker, intel-metrics-library, showing the operational steps the workflow performs.
- **CN:** 它调用了 dnf、intel-opencl、level-zero、mesa-vdpau-drivers、mesa-libxatracker、intel-metrics-library 等命令，展示该工作流执行的操作步骤。

### Lines 93-103 / 第 93-103 行

```bash
    dnf install -y --refresh \
        intel-igc-opencl-devel level-zero-devel intel-gsc-devel libmetee-devel \
        level-zero-devel

    # Cleanup
    dnf clean all
    rm -rf /var/cache/yum
    rm -rf /var/lib/yum/yumdb
    rm -rf /var/lib/yum/history
}

```

- **EN:** This chunk introduces sections such as Cleanup, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Cleanup 等标题组织周边说明或配置。
- **EN:** It invokes commands such as dnf, intel-igc-opencl-devel, level-zero-devel, rm, showing the operational steps the workflow performs.
- **CN:** 它调用了 dnf、intel-igc-opencl-devel、level-zero-devel、rm 等命令，展示该工作流执行的操作步骤。

### Lines 104-111 / 第 104-111 行

```bash
function install_sles() {
    . /etc/os-release
    VERSION_SP=${VERSION_ID//./sp}
    if [[ ! " 15sp4 15sp5 " =~ " ${VERSION_SP} " ]]; then
        echo "SLES version ${VERSION_ID} not supported"
        exit
    fi

```

- **EN:** The script defines shell helpers such as install_sles to structure repeated tasks.
- **CN:** 脚本定义了 install_sles 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as ., exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 .、exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as VERSION_SP, VERSION_ID, SLES communicate required tool locations or behavioral switches.
- **CN:** VERSION_SP、VERSION_ID、SLES 等环境变量用于说明所需工具位置或行为开关。

### Lines 112-119 / 第 112-119 行

```bash
    # To add the online network package repository for the GPU Driver
    zypper addrepo -f -r \
        https://repositories.intel.com/gpu/sles/${VERSION_SP}${XPU_DRIVER_VERSION}/unified/intel-gpu-${VERSION_SP}.repo
    rpm --import https://repositories.intel.com/gpu/intel-graphics.key

    # The xpu-smi packages
    zypper install -y lsb-release flex bison xpu-smi
    # Compute and Media Runtimes
```

- **EN:** This chunk introduces sections such as To add the online network package repository for the GPU Driver, The xpu-smi packages, Compute and Media Runtimes, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 To add the online network package repository for the GPU Driver、The xpu-smi packages、Compute and Media Runtimes 等标题组织周边说明或配置。
- **EN:** It invokes commands such as zypper, https, rpm, showing the operational steps the workflow performs.
- **CN:** 它调用了 zypper、https、rpm 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as GPU, VERSION_SP, XPU_DRIVER_VERSION communicate required tool locations or behavioral switches.
- **CN:** GPU、VERSION_SP、XPU_DRIVER_VERSION 等环境变量用于说明所需工具位置或行为开关。

### Lines 120-127 / 第 120-127 行

```bash
    zypper install -y intel-level-zero-gpu level-zero intel-gsc intel-opencl intel-ocloc \
        intel-media-driver libigfxcmrt7 libvpl2 libvpl-tools libmfxgen1 libmfx1
    # Development packages
    zypper install -y libigdfcl-devel intel-igc-cm libigfxcmrt-devel level-zero-devel
}

function install_xpu_packages() {
    # Download the Intel® software for general purpose GPU capabilities
```

- **EN:** This chunk introduces sections such as Development packages, Download the Intel® software for general purpose GPU capabilities, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Development packages、Download the Intel® software for general purpose GPU capabilities 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as install_xpu_packages to structure repeated tasks.
- **CN:** 脚本定义了 install_xpu_packages 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as zypper, intel-media-driver, showing the operational steps the workflow performs.
- **CN:** 它调用了 zypper、intel-media-driver 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as GPU communicate required tool locations or behavioral switches.
- **CN:** GPU 等环境变量用于说明所需工具位置或行为开关。

### Lines 128-135 / 第 128-135 行

```bash
    wget -qO /tmp/intel-deep-learning-essentials.sh ${XPU_PACKAGES_URL}
    chmod +x /tmp/intel-deep-learning-essentials.sh
    # Install the Intel® software for general purpose GPU capabilities
    /tmp/intel-deep-learning-essentials.sh -a --silent --eula accept
    # Cleanup
    rm -f /tmp/intel-deep-learning-essentials.sh
}

```

- **EN:** This chunk introduces sections such as Install the Intel® software for general purpose GPU capabilities, Cleanup, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Install the Intel® software for general purpose GPU capabilities、Cleanup 等标题组织周边说明或配置。
- **EN:** It invokes commands such as wget, chmod, /tmp/intel-deep-learning-essentials.sh, rm, showing the operational steps the workflow performs.
- **CN:** 它调用了 wget、chmod、/tmp/intel-deep-learning-essentials.sh、rm 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as XPU_PACKAGES_URL, GPU communicate required tool locations or behavioral switches.
- **CN:** XPU_PACKAGES_URL、GPU 等环境变量用于说明所需工具位置或行为开关。

### Lines 136-143 / 第 136-143 行

```bash
# Default use GPU driver rolling releases
XPU_DRIVER_VERSION=""
if [[ "${XPU_DRIVER_TYPE,,}" == "lts" ]]; then
    # Use GPU driver LTS releases
    XPU_DRIVER_VERSION="/lts/2523"
fi

# Default use Intel® oneAPI Deep Learning Essentials 2025.2
```

- **EN:** This chunk introduces sections such as Default use GPU driver rolling releases, Use GPU driver LTS releases, Default use Intel® oneAPI Deep Learning Essentials 2025.2, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Default use GPU driver rolling releases、Use GPU driver LTS releases、Default use Intel® oneAPI Deep Learning Essentials 2025.2 等标题组织周边说明或配置。
- **EN:** Environment variables such as GPU, XPU_DRIVER_VERSION, XPU_DRIVER_TYPE, LTS communicate required tool locations or behavioral switches.
- **CN:** GPU、XPU_DRIVER_VERSION、XPU_DRIVER_TYPE、LTS 等环境变量用于说明所需工具位置或行为开关。

### Lines 144-159 / 第 144-159 行

```bash
if [[ "$XPU_VERSION" == "2025.3" ]]; then
    XPU_PACKAGES_URL="https://registrationcenter-download.intel.com/akdlm/IRC_NAS/b3e6c1bf-a6d5-4580-8b1d-80cbfd38c8af/intel-deep-learning-essentials-2025.3.2.36_offline.sh"
else
    XPU_PACKAGES_URL="https://registrationcenter-download.intel.com/akdlm/IRC_NAS/de3686c4-d3e1-41da-bf3b-bf5908da075c/intel-deep-learning-essentials-2025.2.1.24_offline.sh"
fi

# The Driver installation depends on the base OS
ID=$(grep -oP '(?<=^ID=).+' /etc/os-release | tr -d '"')
case "$ID" in
    ubuntu)
        install_ubuntu
    ;;
    rhel|almalinux)
        install_rhel
    ;;
    sles)
```

- **EN:** This chunk introduces sections such as The Driver installation depends on the base OS, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 The Driver installation depends on the base OS 等标题组织周边说明或配置。
- **EN:** It invokes commands such as ubuntu, install_ubuntu, rhel, install_rhel, sles, showing the operational steps the workflow performs.
- **CN:** 它调用了 ubuntu、install_ubuntu、rhel、install_rhel、sles 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as XPU_VERSION, XPU_PACKAGES_URL, IRC_NAS communicate required tool locations or behavioral switches.
- **CN:** XPU_VERSION、XPU_PACKAGES_URL、IRC_NAS 等环境变量用于说明所需工具位置或行为开关。

### Lines 160-167 / 第 160-167 行

```bash
        install_sles
    ;;
    *)
        echo "Unable to determine OS..."
        exit 1
    ;;
esac

```

- **EN:** It invokes commands such as install_sles, exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 install_sles、exit 等命令，展示该工作流执行的操作步骤。

### Lines 168-169 / 第 168-169 行

```bash
# XPU support packages installation
install_xpu_packages
```

- **EN:** This chunk introduces sections such as XPU support packages installation, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 XPU support packages installation 等标题组织周边说明或配置。
- **EN:** It invokes commands such as install_xpu_packages, showing the operational steps the workflow performs.
- **CN:** 它调用了 install_xpu_packages 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as XPU communicate required tool locations or behavioral switches.
- **CN:** XPU 等环境变量用于说明所需工具位置或行为开关。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **Representative symbols: install_ubuntu, install_rhel, install_sles, install_xpu_packages** — 代表性符号：install_ubuntu、install_rhel、install_sles、install_xpu_packages

## Dependencies / 依赖关系

- `/etc/os-release`
- `bash`
- `wget`
