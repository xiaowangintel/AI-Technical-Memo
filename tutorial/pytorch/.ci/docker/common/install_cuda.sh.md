# install_cuda.sh — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `.ci/docker/common/install_cuda.sh`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Automates repository maintenance, build, test, packaging, or release tasks through shell commands. The opening comment frames the file as: "!/bin/bash."
- **Purpose (CN)**: 通过 shell 命令自动化执行仓库维护、构建、测试、打包或发布任务。 开头注释将该文件概括为：“!/bin/bash”。

## Content Analysis / 内容分析

### Lines 1-12 / 第 1-12 行

```bash
#!/bin/bash

set -ex

arch_path=''
targetarch=${TARGETARCH:-$(uname -m)}
if [ ${targetarch} = 'amd64' ] || [ "${targetarch}" = 'x86_64' ]; then
  arch_path='x86_64'
else
  arch_path='sbsa'
fi

```

- **EN:** This chunk introduces sections such as !/bin/bash, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 !/bin/bash 等标题组织周边说明或配置。
- **EN:** Environment variables such as TARGETARCH communicate required tool locations or behavioral switches.
- **CN:** TARGETARCH 等环境变量用于说明所需工具位置或行为开关。

### Lines 13-30 / 第 13-30 行

```bash
NVSHMEM_VERSION=3.4.5

function install_cuda {
  version=$1
  runfile=$2
  major_minor=${version%.*}
  rm -rf /usr/local/cuda-${major_minor} /usr/local/cuda
  if [[ ${arch_path} == 'sbsa' ]]; then
      runfile="${runfile}_sbsa"
  fi
  runfile="${runfile}.run"
  wget -q https://developer.download.nvidia.com/compute/cuda/${version}/local_installers/${runfile} -O ${runfile}
  chmod +x ${runfile}
  ./${runfile} --toolkit --silent
  rm -f ${runfile}
  rm -f /usr/local/cuda && ln -s /usr/local/cuda-${major_minor} /usr/local/cuda
}

```

- **EN:** It invokes commands such as rm, wget, chmod, ./, showing the operational steps the workflow performs.
- **CN:** 它调用了 rm、wget、chmod、./ 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as NVSHMEM_VERSION communicate required tool locations or behavioral switches.
- **CN:** NVSHMEM_VERSION 等环境变量用于说明所需工具位置或行为开关。

### Lines 31-44 / 第 31-44 行

```bash
function install_cudnn {
  cuda_major_version=$1
  cudnn_version=$2
  mkdir tmp_cudnn && cd tmp_cudnn
  # cuDNN license: https://developer.nvidia.com/cudnn/license_agreement
  filepath="cudnn-linux-${arch_path}-${cudnn_version}_cuda${cuda_major_version}-archive"
  wget -q https://developer.download.nvidia.com/compute/cudnn/redist/cudnn/linux-${arch_path}/${filepath}.tar.xz
  tar xf ${filepath}.tar.xz
  cp -a ${filepath}/include/* /usr/local/cuda/include/
  cp -a ${filepath}/lib/* /usr/local/cuda/lib64/
  cd ..
  rm -rf tmp_cudnn
}

```

- **EN:** This chunk introduces sections such as cuDNN license: https://developer.nvidia.com/cudnn/license_agreement, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 cuDNN license: https://developer.nvidia.com/cudnn/license_agreement 等标题组织周边说明或配置。
- **EN:** It invokes commands such as mkdir, wget, tar, cp, cd, rm, showing the operational steps the workflow performs.
- **CN:** 它调用了 mkdir、wget、tar、cp、cd、rm 等命令，展示该工作流执行的操作步骤。

### Lines 45-60 / 第 45-60 行

```bash
function install_nvshmem {
  cuda_major_version=$1      # e.g. "12"
  nvshmem_version=$2         # e.g. "3.3.9"

  case "${arch_path}" in
    sbsa)
      dl_arch="aarch64"
      ;;
    x86_64)
      dl_arch="x64"
      ;;
    *)
      dl_arch="${arch}"
      ;;
  esac

```

- **EN:** It invokes commands such as sbsa, x86_64, showing the operational steps the workflow performs.
- **CN:** 它调用了 sbsa、x86_64 等命令，展示该工作流执行的操作步骤。

### Lines 61-75 / 第 61-75 行

```bash
  tmpdir="tmp_nvshmem"
  mkdir -p "${tmpdir}" && cd "${tmpdir}"

  # nvSHMEM license: https://docs.nvidia.com/nvshmem/api/sla.html
  # This pattern is a lie as it is not consistent across versions, for 3.3.9 it was cuda_ver-arch-nvshhem-ver
  filename="libnvshmem-linux-${arch_path}-${nvshmem_version}_cuda${cuda_major_version}-archive"
  suffix=".tar.xz"
  url="https://developer.download.nvidia.com/compute/nvshmem/redist/libnvshmem/linux-${arch_path}/${filename}${suffix}"

  # download, unpack, install
  wget -q "${url}"
  tar xf "${filename}${suffix}"
  cp -a "${filename}/include/"* /usr/local/cuda/include/
  cp -a "${filename}/lib/"*     /usr/local/cuda/lib64/

```

- **EN:** This chunk introduces sections such as nvSHMEM license: https://docs.nvidia.com/nvshmem/api/sla.html, This pattern is a lie as it is not consistent across versions, for 3.3.9 it was cuda_ver-arch-nvshhem-ver, download, unpack, install, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 nvSHMEM license: https://docs.nvidia.com/nvshmem/api/sla.html、This pattern is a lie as it is not consistent across versions, for 3.3.9 it was cuda_ver-arch-nvshhem-ver、download, unpack, install 等标题组织周边说明或配置。
- **EN:** It invokes commands such as mkdir, wget, tar, cp, showing the operational steps the workflow performs.
- **CN:** 它调用了 mkdir、wget、tar、cp 等命令，展示该工作流执行的操作步骤。

### Lines 76-88 / 第 76-88 行

```bash
  # cleanup
  cd ..
  rm -rf "${tmpdir}"

  echo "nvSHMEM ${nvshmem_version} for CUDA ${cuda_major_version} (${arch_path}) installed."
}

function install_124 {
  CUDNN_VERSION=9.1.0.70
  CUSPARSELT_VERSION=0.6.2.3
  echo "Installing CUDA 12.4.1 and cuDNN ${CUDNN_VERSION} and NCCL and cuSparseLt-${CUSPARSELT_VERSION}"
  install_cuda 12.4.1 cuda_12.4.1_550.54.15_linux

```

- **EN:** This chunk introduces sections such as cleanup, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 cleanup 等标题组织周边说明或配置。
- **EN:** It invokes commands such as cd, rm, install_cuda, showing the operational steps the workflow performs.
- **CN:** 它调用了 cd、rm、install_cuda 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUDA, CUDNN_VERSION, CUSPARSELT_VERSION, NCCL communicate required tool locations or behavioral switches.
- **CN:** CUDA、CUDNN_VERSION、CUSPARSELT_VERSION、NCCL 等环境变量用于说明所需工具位置或行为开关。

### Lines 89-103 / 第 89-103 行

```bash
  install_cudnn 12 $CUDNN_VERSION

  CUDA_VERSION=12.4 bash install_nccl.sh

  CUDA_VERSION=12.4 bash install_cusparselt.sh $CUSPARSELT_VERSION

  ldconfig
}

function install_126 {
  CUDNN_VERSION=9.10.2.21
  CUSPARSELT_VERSION=0.7.1.0
  echo "Installing CUDA 12.6.3 and cuDNN ${CUDNN_VERSION} and NVSHMEM and NCCL and cuSparseLt-${CUSPARSELT_VERSION}"
  install_cuda 12.6.3 cuda_12.6.3_560.35.05_linux

```

- **EN:** It invokes commands such as install_cudnn, ldconfig, install_cuda, showing the operational steps the workflow performs.
- **CN:** 它调用了 install_cudnn、ldconfig、install_cuda 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUDNN_VERSION, CUDA_VERSION, CUSPARSELT_VERSION, CUDA, NVSHMEM, NCCL communicate required tool locations or behavioral switches.
- **CN:** CUDNN_VERSION、CUDA_VERSION、CUSPARSELT_VERSION、CUDA、NVSHMEM、NCCL 等环境变量用于说明所需工具位置或行为开关。

### Lines 104-119 / 第 104-119 行

```bash
  install_cudnn 12 $CUDNN_VERSION

  install_nvshmem 12 $NVSHMEM_VERSION

  CUDA_VERSION=12.6 bash install_nccl.sh

  CUDA_VERSION=12.6 bash install_cusparselt.sh $CUSPARSELT_VERSION

  ldconfig
}

function install_129 {
  CUDNN_VERSION=9.20.0.48
  CUSPARSELT_VERSION=0.8.1.1
  echo "Installing CUDA 12.9.1 and cuDNN ${CUDNN_VERSION} and NVSHMEM and NCCL and cuSparseLt-${CUSPARSELT_VERSION}"
  # install CUDA 12.9.1 in the same container
```

- **EN:** This chunk introduces sections such as install CUDA 12.9.1 in the same container, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 install CUDA 12.9.1 in the same container 等标题组织周边说明或配置。
- **EN:** It invokes commands such as install_cudnn, install_nvshmem, ldconfig, showing the operational steps the workflow performs.
- **CN:** 它调用了 install_cudnn、install_nvshmem、ldconfig 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUDNN_VERSION, NVSHMEM_VERSION, CUDA_VERSION, CUSPARSELT_VERSION, CUDA, NVSHMEM communicate required tool locations or behavioral switches.
- **CN:** CUDNN_VERSION、NVSHMEM_VERSION、CUDA_VERSION、CUSPARSELT_VERSION、CUDA、NVSHMEM 等环境变量用于说明所需工具位置或行为开关。

### Lines 120-133 / 第 120-133 行

```bash
  install_cuda 12.9.1 cuda_12.9.1_575.57.08_linux

  # cuDNN license: https://developer.nvidia.com/cudnn/license_agreement
  install_cudnn 12 $CUDNN_VERSION

  install_nvshmem 12 $NVSHMEM_VERSION

  CUDA_VERSION=12.9 bash install_nccl.sh

  CUDA_VERSION=12.9 bash install_cusparselt.sh $CUSPARSELT_VERSION

  ldconfig
}

```

- **EN:** This chunk introduces sections such as cuDNN license: https://developer.nvidia.com/cudnn/license_agreement, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 cuDNN license: https://developer.nvidia.com/cudnn/license_agreement 等标题组织周边说明或配置。
- **EN:** It invokes commands such as install_cuda, install_cudnn, install_nvshmem, ldconfig, showing the operational steps the workflow performs.
- **CN:** 它调用了 install_cuda、install_cudnn、install_nvshmem、ldconfig 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUDNN_VERSION, NVSHMEM_VERSION, CUDA_VERSION, CUSPARSELT_VERSION communicate required tool locations or behavioral switches.
- **CN:** CUDNN_VERSION、NVSHMEM_VERSION、CUDA_VERSION、CUSPARSELT_VERSION 等环境变量用于说明所需工具位置或行为开关。

### Lines 134-145 / 第 134-145 行

```bash
function install_128 {
  CUDNN_VERSION=9.20.0.48
  CUSPARSELT_VERSION=0.7.1.0
  echo "Installing CUDA 12.8.1 and cuDNN ${CUDNN_VERSION} and NVSHMEM and NCCL and cuSparseLt-${CUSPARSELT_VERSION}"
  # install CUDA 12.8.1 in the same container
  install_cuda 12.8.1 cuda_12.8.1_570.124.06_linux

  # cuDNN license: https://developer.nvidia.com/cudnn/license_agreement
  install_cudnn 12 $CUDNN_VERSION

  install_nvshmem 12 $NVSHMEM_VERSION

```

- **EN:** This chunk introduces sections such as install CUDA 12.8.1 in the same container, cuDNN license: https://developer.nvidia.com/cudnn/license_agreement, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 install CUDA 12.8.1 in the same container、cuDNN license: https://developer.nvidia.com/cudnn/license_agreement 等标题组织周边说明或配置。
- **EN:** It invokes commands such as install_cuda, install_cudnn, install_nvshmem, showing the operational steps the workflow performs.
- **CN:** 它调用了 install_cuda、install_cudnn、install_nvshmem 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUDNN_VERSION, CUSPARSELT_VERSION, CUDA, NVSHMEM, NCCL, NVSHMEM_VERSION communicate required tool locations or behavioral switches.
- **CN:** CUDNN_VERSION、CUSPARSELT_VERSION、CUDA、NVSHMEM、NCCL、NVSHMEM_VERSION 等环境变量用于说明所需工具位置或行为开关。

### Lines 146-157 / 第 146-157 行

```bash
  CUDA_VERSION=12.8 bash install_nccl.sh

  CUDA_VERSION=12.8 bash install_cusparselt.sh $CUSPARSELT_VERSION

  ldconfig
}

function install_130 {
  CUDNN_VERSION=9.20.0.48
  CUSPARSELT_VERSION=0.8.1.1
  echo "Installing CUDA 13.0 and cuDNN ${CUDNN_VERSION} and NVSHMEM and NCCL and cuSparseLt-${CUSPARSELT_VERSION}"
  # install CUDA 13.0 in the same container
```

- **EN:** This chunk introduces sections such as install CUDA 13.0 in the same container, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 install CUDA 13.0 in the same container 等标题组织周边说明或配置。
- **EN:** It invokes commands such as ldconfig, showing the operational steps the workflow performs.
- **CN:** 它调用了 ldconfig 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUDA_VERSION, CUSPARSELT_VERSION, CUDNN_VERSION, CUDA, NVSHMEM, NCCL communicate required tool locations or behavioral switches.
- **CN:** CUDA_VERSION、CUSPARSELT_VERSION、CUDNN_VERSION、CUDA、NVSHMEM、NCCL 等环境变量用于说明所需工具位置或行为开关。

### Lines 158-171 / 第 158-171 行

```bash
  install_cuda 13.0.2 cuda_13.0.2_580.95.05_linux

  # cuDNN license: https://developer.nvidia.com/cudnn/license_agreement
  install_cudnn 13 $CUDNN_VERSION

  install_nvshmem 13 $NVSHMEM_VERSION

  CUDA_VERSION=13.0 bash install_nccl.sh

  CUDA_VERSION=13.0 bash install_cusparselt.sh $CUSPARSELT_VERSION

  ldconfig
}

```

- **EN:** This chunk introduces sections such as cuDNN license: https://developer.nvidia.com/cudnn/license_agreement, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 cuDNN license: https://developer.nvidia.com/cudnn/license_agreement 等标题组织周边说明或配置。
- **EN:** It invokes commands such as install_cuda, install_cudnn, install_nvshmem, ldconfig, showing the operational steps the workflow performs.
- **CN:** 它调用了 install_cuda、install_cudnn、install_nvshmem、ldconfig 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUDNN_VERSION, NVSHMEM_VERSION, CUDA_VERSION, CUSPARSELT_VERSION communicate required tool locations or behavioral switches.
- **CN:** CUDNN_VERSION、NVSHMEM_VERSION、CUDA_VERSION、CUSPARSELT_VERSION 等环境变量用于说明所需工具位置或行为开关。

### Lines 172-183 / 第 172-183 行

```bash
function install_132 {
  CUDNN_VERSION=9.20.0.48
  CUSPARSELT_VERSION=0.8.1.1
  echo "Installing CUDA 13.2 and cuDNN ${CUDNN_VERSION} and NVSHMEM and NCCL and cuSparseLt-${CUSPARSELT_VERSION}"
  # install CUDA 13.2 in the same container
  install_cuda 13.2.1 cuda_13.2.1_595.58.03_linux

  # cuDNN license: https://developer.nvidia.com/cudnn/license_agreement
  install_cudnn 13 $CUDNN_VERSION

  install_nvshmem 13 $NVSHMEM_VERSION

```

- **EN:** This chunk introduces sections such as install CUDA 13.2 in the same container, cuDNN license: https://developer.nvidia.com/cudnn/license_agreement, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 install CUDA 13.2 in the same container、cuDNN license: https://developer.nvidia.com/cudnn/license_agreement 等标题组织周边说明或配置。
- **EN:** It invokes commands such as install_cuda, install_cudnn, install_nvshmem, showing the operational steps the workflow performs.
- **CN:** 它调用了 install_cuda、install_cudnn、install_nvshmem 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUDNN_VERSION, CUSPARSELT_VERSION, CUDA, NVSHMEM, NCCL, NVSHMEM_VERSION communicate required tool locations or behavioral switches.
- **CN:** CUDNN_VERSION、CUSPARSELT_VERSION、CUDA、NVSHMEM、NCCL、NVSHMEM_VERSION 等环境变量用于说明所需工具位置或行为开关。

### Lines 184-195 / 第 184-195 行

```bash
  CUDA_VERSION=13.2 bash install_nccl.sh

  CUDA_VERSION=13.2 bash install_cusparselt.sh $CUSPARSELT_VERSION

  ldconfig
}

# idiomatic parameter and option handling in sh
while test $# -gt 0
do
    case "$1" in
    12.4) install_124;
```

- **EN:** This chunk introduces sections such as idiomatic parameter and option handling in sh, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 idiomatic parameter and option handling in sh 等标题组织周边说明或配置。
- **EN:** It invokes commands such as ldconfig, 12.4, showing the operational steps the workflow performs.
- **CN:** 它调用了 ldconfig、12.4 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as CUDA_VERSION, CUSPARSELT_VERSION communicate required tool locations or behavioral switches.
- **CN:** CUDA_VERSION、CUSPARSELT_VERSION 等环境变量用于说明所需工具位置或行为开关。

### Lines 196-211 / 第 196-211 行

```bash
        ;;
    12.6|12.6.*) install_126;
        ;;
    12.8|12.8.*) install_128;
        ;;
    12.9|12.9.*) install_129;
        ;;
    13.0|13.0.*) install_130;
        ;;
    13.2|13.2.*) install_132;
        ;;
    *) echo "bad argument $1"; exit 1
        ;;
    esac
    shift
done
```

- **EN:** It invokes commands such as 12.6, 12.8, 12.9, 13.0, 13.2, shift, showing the operational steps the workflow performs.
- **CN:** 它调用了 12.6、12.8、12.9、13.0、13.2、shift 等命令，展示该工作流执行的操作步骤。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑

## Dependencies / 依赖关系

- `bash`
- `wget`
- `tar`
