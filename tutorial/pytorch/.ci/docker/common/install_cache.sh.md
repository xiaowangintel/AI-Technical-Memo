# install_cache.sh — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `.ci/docker/common/install_cache.sh`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Automates repository maintenance, build, test, packaging, or release tasks through shell commands. The opening comment frames the file as: "!/bin/bash."
- **Purpose (CN)**: 通过 shell 命令自动化执行仓库维护、构建、测试、打包或发布任务。 开头注释将该文件概括为：“!/bin/bash”。

## Content Analysis / 内容分析

### Lines 1-16 / 第 1-16 行

```bash
#!/bin/bash

set -ex

install_ubuntu() {
  echo "Installing pkg-config and libssl-dev"
  apt-get update && apt-get install -y pkg-config libssl-dev curl
  echo "Installing rust"
  curl https://sh.rustup.rs -sSf | sh -s -- -y
  echo "Checking out sccache repo"
  git clone https://github.com/mozilla/sccache -b v0.13.0
  cd sccache
  echo "Patch dist build on aarch64"
  sed -i '/all(target_os = "linux", target_arch = "x86_64"),/{ p; s/x86_64/aarch64/; }' src/bin/sccache-dist/main.rs
  echo "Building sccache"
  . "$HOME/.cargo/env" && cargo build --release --features="dist-client dist-server"
```

- **EN:** This chunk introduces sections such as !/bin/bash, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 !/bin/bash 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as install_ubuntu to structure repeated tasks.
- **CN:** 脚本定义了 install_ubuntu 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as install_ubuntu, apt-get, curl, git, cd, sed, showing the operational steps the workflow performs.
- **CN:** 它调用了 install_ubuntu、apt-get、curl、git、cd、sed 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as HOME communicate required tool locations or behavioral switches.
- **CN:** HOME 等环境变量用于说明所需工具位置或行为开关。

### Lines 17-25 / 第 17-25 行

```bash
  cp target/release/sccache /opt/cache/bin
  cp target/release/sccache-dist /opt/cache/bin
  echo "Cleaning up"
  cd ..
  rm -rf sccache
  rustup self uninstall -y
  apt-get remove -y pkg-config libssl-dev
  apt-get autoclean && apt-get clean

```

- **EN:** It invokes commands such as cp, cd, rm, rustup, apt-get, showing the operational steps the workflow performs.
- **CN:** 它调用了 cp、cd、rm、rustup、apt-get 等命令，展示该工作流执行的操作步骤。

### Lines 26-35 / 第 26-35 行

```bash
  echo "Downloading old sccache binary from S3 repo for PCH builds"
  curl --retry 3 https://s3.amazonaws.com/ossci-linux/sccache -o /opt/cache/bin/sccache-0.2.14a
  chmod 755 /opt/cache/bin/sccache-0.2.14a
}

install_binary() {
  echo "Downloading sccache binary from S3 repo"
  curl --retry 3 https://s3.amazonaws.com/ossci-linux/sccache -o /opt/cache/bin/sccache
}

```

- **EN:** The script defines shell helpers such as install_binary to structure repeated tasks.
- **CN:** 脚本定义了 install_binary 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as curl, chmod, install_binary, showing the operational steps the workflow performs.
- **CN:** 它调用了 curl、chmod、install_binary 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as PCH communicate required tool locations or behavioral switches.
- **CN:** PCH 等环境变量用于说明所需工具位置或行为开关。

### Lines 36-44 / 第 36-44 行

```bash
mkdir -p /opt/cache/bin
mkdir -p /opt/cache/lib
sed -e 's|PATH="\(.*\)"|PATH="/opt/cache/bin:\1"|g' -i /etc/environment
export PATH="/opt/cache/bin:$PATH"

# Setup compiler cache
install_ubuntu
chmod a+x /opt/cache/bin/sccache

```

- **EN:** This chunk introduces sections such as Setup compiler cache, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Setup compiler cache 等标题组织周边说明或配置。
- **EN:** It invokes commands such as mkdir, sed, install_ubuntu, chmod, showing the operational steps the workflow performs.
- **CN:** 它调用了 mkdir、sed、install_ubuntu、chmod 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as PATH communicate required tool locations or behavioral switches.
- **CN:** PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 45-52 / 第 45-52 行

```bash
function write_sccache_stub() {
  # Unset LD_PRELOAD for ps because of asan + ps issues
  # https://gcc.gnu.org/bugzilla/show_bug.cgi?id=90589
  if [ $1 == "gcc" ]; then
    # Do not call sccache recursively when dumping preprocessor argument
    # For some reason it's very important for the first cached nvcc invocation
    cat >"/opt/cache/bin/$1" <<EOF
#!/bin/sh
```

- **EN:** This chunk introduces sections such as Unset LD_PRELOAD for ps because of asan + ps issues, https://gcc.gnu.org/bugzilla/show_bug.cgi?id=90589, Do not call sccache recursively when dumping preprocessor argument, For some reason it's very important for the first cached nvcc invocation, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Unset LD_PRELOAD for ps because of asan + ps issues、https://gcc.gnu.org/bugzilla/show_bug.cgi?id=90589、Do not call sccache recursively when dumping preprocessor argument、For some reason it's very important for the first cached nvcc invocation 等标题组织周边说明或配置。
- **EN:** The script defines shell helpers such as write_sccache_stub to structure repeated tasks.
- **CN:** 脚本定义了 write_sccache_stub 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as cat, showing the operational steps the workflow performs.
- **CN:** 它调用了 cat 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as LD_PRELOAD, EOF communicate required tool locations or behavioral switches.
- **CN:** LD_PRELOAD、EOF 等环境变量用于说明所需工具位置或行为开关。

### Lines 53-60 / 第 53-60 行

```bash

# sccache does not support -E flag, so we need to call the original compiler directly in order to avoid calling this wrapper recursively
for arg in "\$@"; do
  if [ "\$arg" = "-E" ]; then
    exec $(which $1) "\$@"
  fi
done

```

- **EN:** This chunk introduces sections such as sccache does not support -E flag, so we need to call the original compiler directly in order to avoid calling this wrapper recursively, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 sccache does not support -E flag, so we need to call the original compiler directly in order to avoid calling this wrapper recursively 等标题组织周边说明或配置。
- **EN:** It invokes commands such as exec, showing the operational steps the workflow performs.
- **CN:** 它调用了 exec 等命令，展示该工作流执行的操作步骤。

### Lines 61-69 / 第 61-69 行

```bash
if [ \$(env -u LD_PRELOAD ps -p \$PPID -o comm=) != sccache ]; then
  exec sccache $(which $1) "\$@"
else
  exec $(which $1) "\$@"
fi
EOF
  else
    cat >"/opt/cache/bin/$1" <<EOF
#!/bin/sh
```

- **EN:** This chunk introduces sections such as !/bin/sh, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 !/bin/sh 等标题组织周边说明或配置。
- **EN:** It invokes commands such as exec, EOF, cat, showing the operational steps the workflow performs.
- **CN:** 它调用了 exec、EOF、cat 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as LD_PRELOAD, PPID, EOF communicate required tool locations or behavioral switches.
- **CN:** LD_PRELOAD、PPID、EOF 等环境变量用于说明所需工具位置或行为开关。

### Lines 70-80 / 第 70-80 行

```bash

if [ \$(env -u LD_PRELOAD ps -p \$PPID -o comm=) != sccache ]; then
  exec sccache $(which $1) "\$@"
else
  exec $(which $1) "\$@"
fi
EOF
  fi
  chmod a+x "/opt/cache/bin/$1"
}

```

- **EN:** It invokes commands such as exec, EOF, chmod, showing the operational steps the workflow performs.
- **CN:** 它调用了 exec、EOF、chmod 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as LD_PRELOAD, PPID, EOF communicate required tool locations or behavioral switches.
- **CN:** LD_PRELOAD、PPID、EOF 等环境变量用于说明所需工具位置或行为开关。

### Lines 81-90 / 第 81-90 行

```bash
# Skip all sccache wrapping for theRock nightly: sccache PATH wrappers
# intercept assembly (.s) compilation and fail because the assembler does not
# produce the .d dependency file that sccache expects.
if [ "$ROCM_VERSION" != "nightly" ]; then
  write_sccache_stub cc
  write_sccache_stub c++
  write_sccache_stub gcc
  write_sccache_stub g++
fi

```

- **EN:** This chunk introduces sections such as Skip all sccache wrapping for theRock nightly: sccache PATH wrappers, intercept assembly (.s) compilation and fail because the assembler does not, produce the .d dependency file that sccache expects., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Skip all sccache wrapping for theRock nightly: sccache PATH wrappers、intercept assembly (.s) compilation and fail because the assembler does not、produce the .d dependency file that sccache expects. 等标题组织周边说明或配置。
- **EN:** It invokes commands such as write_sccache_stub, showing the operational steps the workflow performs.
- **CN:** 它调用了 write_sccache_stub 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as PATH, ROCM_VERSION communicate required tool locations or behavioral switches.
- **CN:** PATH、ROCM_VERSION 等环境变量用于说明所需工具位置或行为开关。

### Lines 91-98 / 第 91-98 行

```bash
# NOTE: See specific ROCM_VERSION case below.
if [ "x$ROCM_VERSION" = x ]; then
  write_sccache_stub clang
  write_sccache_stub clang++
fi

if [ -n "$CUDA_VERSION" ]; then
  # TODO: This is a workaround for the fact that PyTorch's FindCUDA
```

- **EN:** This chunk introduces sections such as NOTE: See specific ROCM_VERSION case below., TODO: This is a workaround for the fact that PyTorch's FindCUDA, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 NOTE: See specific ROCM_VERSION case below.、TODO: This is a workaround for the fact that PyTorch's FindCUDA 等标题组织周边说明或配置。
- **EN:** It invokes commands such as write_sccache_stub, showing the operational steps the workflow performs.
- **CN:** 它调用了 write_sccache_stub 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as NOTE, ROCM_VERSION, CUDA_VERSION, TODO communicate required tool locations or behavioral switches.
- **CN:** NOTE、ROCM_VERSION、CUDA_VERSION、TODO 等环境变量用于说明所需工具位置或行为开关。

### Lines 99-107 / 第 99-107 行

```bash
  # implementation cannot find nvcc if it is setup this way, because it
  # appears to search for the nvcc in PATH, and use its path to infer
  # where CUDA is installed.  Instead, we install an nvcc symlink outside
  # of the PATH, and set CUDA_NVCC_EXECUTABLE so that we make use of it.

  write_sccache_stub nvcc
  mv /opt/cache/bin/nvcc /opt/cache/lib/
fi

```

- **EN:** This chunk introduces sections such as implementation cannot find nvcc if it is setup this way, because it, appears to search for the nvcc in PATH, and use its path to infer, where CUDA is installed.  Instead, we install an nvcc symlink outside, of the PATH, and set CUDA_NVCC_EXECUTABLE so that we make use of it., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 implementation cannot find nvcc if it is setup this way, because it、appears to search for the nvcc in PATH, and use its path to infer、where CUDA is installed.  Instead, we install an nvcc symlink outside、of the PATH, and set CUDA_NVCC_EXECUTABLE so that we make use of it. 等标题组织周边说明或配置。
- **EN:** It invokes commands such as write_sccache_stub, mv, showing the operational steps the workflow performs.
- **CN:** 它调用了 write_sccache_stub、mv 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as PATH, CUDA, CUDA_NVCC_EXECUTABLE communicate required tool locations or behavioral switches.
- **CN:** PATH、CUDA、CUDA_NVCC_EXECUTABLE 等环境变量用于说明所需工具位置或行为开关。

### Lines 108-115 / 第 108-115 行

```bash
if [ -n "$ROCM_VERSION" ]; then
  # Skip sccache wrapping for theRock nightly - sccache has issues parsing
  # theRock's complex include paths and causes hipconfig to fail
  if [ "$ROCM_VERSION" = "nightly" ]; then
    echo "Skipping sccache wrapping for theRock nightly ROCm"
  else
    source /etc/rocm_env.sh

```

- **EN:** This chunk introduces sections such as Skip sccache wrapping for theRock nightly - sccache has issues parsing, theRock's complex include paths and causes hipconfig to fail, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Skip sccache wrapping for theRock nightly - sccache has issues parsing、theRock's complex include paths and causes hipconfig to fail 等标题组织周边说明或配置。
- **EN:** It invokes commands such as source, showing the operational steps the workflow performs.
- **CN:** 它调用了 source 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as ROCM_VERSION communicate required tool locations or behavioral switches.
- **CN:** ROCM_VERSION 等环境变量用于说明所需工具位置或行为开关。

### Lines 116-123 / 第 116-123 行

```bash
    # ROCm compiler is hcc or clang. However, it is commonly invoked via hipcc wrapper.
    # hipcc will call either hcc or clang using an absolute path starting with $ROCM_PATH,
    # causing the /opt/cache/bin to be skipped. We must create the sccache wrappers
    # directly under $ROCM_PATH while also preserving the original compiler names.
    # Note symlinks will chain as follows: [hcc or clang++] -> clang -> clang-??
    # Final link in symlink chain must point back to original directory.

    # Original compiler is moved one directory deeper. Wrapper replaces it.
```

- **EN:** This chunk introduces sections such as ROCm compiler is hcc or clang. However, it is commonly invoked via hipcc wrapper., hipcc will call either hcc or clang using an absolute path starting with $ROCM_PATH,, causing the /opt/cache/bin to be skipped. We must create the sccache wrappers, directly under $ROCM_PATH while also preserving the original compiler names., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ROCm compiler is hcc or clang. However, it is commonly invoked via hipcc wrapper.、hipcc will call either hcc or clang using an absolute path starting with $ROCM_PATH,、causing the /opt/cache/bin to be skipped. We must create the sccache wrappers、directly under $ROCM_PATH while also preserving the original compiler names. 等标题组织周边说明或配置。
- **EN:** Environment variables such as ROCM_PATH communicate required tool locations or behavioral switches.
- **CN:** ROCM_PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 124-133 / 第 124-133 行

```bash
    function write_sccache_stub_rocm() {
      OLDCOMP=$1
      COMPNAME=$(basename $OLDCOMP)
      TOPDIR=$(dirname $OLDCOMP)
      WRAPPED="$TOPDIR/original/$COMPNAME"
      mv "$OLDCOMP" "$WRAPPED"
      printf "#!/bin/sh\nexec sccache $WRAPPED \"\$@\"" >"$OLDCOMP"
      chmod a+x "$OLDCOMP"
    }

```

- **EN:** The script defines shell helpers such as write_sccache_stub_rocm to structure repeated tasks.
- **CN:** 脚本定义了 write_sccache_stub_rocm 等 shell 辅助函数，以组织重复任务。
- **EN:** It invokes commands such as mv, printf, chmod, showing the operational steps the workflow performs.
- **CN:** 它调用了 mv、printf、chmod 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as OLDCOMP, COMPNAME, TOPDIR, WRAPPED communicate required tool locations or behavioral switches.
- **CN:** OLDCOMP、COMPNAME、TOPDIR、WRAPPED 等环境变量用于说明所需工具位置或行为开关。

### Lines 134-149 / 第 134-149 行

```bash
    # ROCm 3.5 and beyond use llvm/bin/clang
    if [[ -e "${ROCM_PATH}/llvm/bin/clang" ]]; then
      mkdir ${ROCM_PATH}/llvm/bin/original
      write_sccache_stub_rocm ${ROCM_PATH}/llvm/bin/clang
      write_sccache_stub_rocm ${ROCM_PATH}/llvm/bin/clang++
      # Fix last link in symlink chain for traditional ROCm where clang -> clang-17
      pushd ${ROCM_PATH}/llvm/bin/original
      if [[ -L clang ]] && [[ "$(readlink clang)" == clang-* ]]; then
        ln -s ../$(readlink clang)
      fi
      popd
    else
      echo "Cannot find ROCm compiler."
      exit 1
    fi
  fi
```

- **EN:** This chunk introduces sections such as ROCm 3.5 and beyond use llvm/bin/clang, Fix last link in symlink chain for traditional ROCm where clang -> clang-17, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ROCm 3.5 and beyond use llvm/bin/clang、Fix last link in symlink chain for traditional ROCm where clang -> clang-17 等标题组织周边说明或配置。
- **EN:** It invokes commands such as mkdir, write_sccache_stub_rocm, pushd, ln, popd, exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 mkdir、write_sccache_stub_rocm、pushd、ln、popd、exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as ROCM_PATH communicate required tool locations or behavioral switches.
- **CN:** ROCM_PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 150-150 / 第 150-150 行

```bash
fi
```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **Repository bootstrap** — 与仓库状态、子模块或源码检出流程交互。
- **Representative symbols: install_ubuntu, install_binary, write_sccache_stub, write_sccache_stub_rocm** — 代表性符号：install_ubuntu、install_binary、write_sccache_stub、write_sccache_stub_rocm

## Dependencies / 依赖关系

- `"$HOME/.cargo/env"`
- `/etc/rocm_env.sh`
- `bash`
- `curl`
- `git`
- `make`
