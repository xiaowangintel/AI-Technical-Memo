# test.sh — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `.ci/caffe2/test.sh`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Automates repository maintenance, build, test, packaging, or release tasks through shell commands. Performance measurement or benchmark orchestration is part of its intent. The opening comment frames the file as: "!/bin/bash."
- **Purpose (CN)**: 通过 shell 命令自动化执行仓库维护、构建、测试、打包或发布任务。 性能测量或基准编排是其意图的一部分。 开头注释将该文件概括为：“!/bin/bash”。

## Content Analysis / 内容分析

### Lines 1-10 / 第 1-10 行

```bash
#!/bin/bash

# shellcheck source=./common.sh
source "$(dirname "${BASH_SOURCE[0]}")/common.sh"

if [[ ${BUILD_ENVIRONMENT} == *onnx* ]]; then
  pip install click mock tabulate networkx==2.0
  pip -q install "file:///var/lib/jenkins/workspace/third_party/onnx#egg=onnx"
fi

```

- **EN:** This chunk introduces sections such as !/bin/bash, shellcheck source=./common.sh, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 !/bin/bash、shellcheck source=./common.sh 等标题组织周边说明或配置。
- **EN:** It invokes commands such as source, pip, showing the operational steps the workflow performs.
- **CN:** 它调用了 source、pip 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BASH_SOURCE, BUILD_ENVIRONMENT communicate required tool locations or behavioral switches.
- **CN:** BASH_SOURCE、BUILD_ENVIRONMENT 等环境变量用于说明所需工具位置或行为开关。

### Lines 11-18 / 第 11-18 行

```bash
# Skip tests in environments where they are not built/applicable
if [[ "${BUILD_ENVIRONMENT}" == *-android* ]]; then
  echo 'Skipping tests'
  exit 0
fi
# These additional packages are needed for circleci ROCm builds.
if [[ $BUILD_ENVIRONMENT == *rocm* ]]; then
    # Need networkx 2.0 because bellmand_ford was moved in 2.1 . Scikit-image by
```

- **EN:** This chunk introduces sections such as Skip tests in environments where they are not built/applicable, These additional packages are needed for circleci ROCm builds., Need networkx 2.0 because bellmand_ford was moved in 2.1 . Scikit-image by, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Skip tests in environments where they are not built/applicable、These additional packages are needed for circleci ROCm builds.、Need networkx 2.0 because bellmand_ford was moved in 2.1 . Scikit-image by 等标题组织周边说明或配置。
- **EN:** It invokes commands such as exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT 等环境变量用于说明所需工具位置或行为开关。

### Lines 19-26 / 第 19-26 行

```bash
    # defaults installs the most recent networkx version, so we install this lower
    # version explicitly before scikit-image pulls it in as a dependency
    pip install networkx==2.0
    # click - onnx
    pip install --progress-bar off click protobuf tabulate virtualenv mock typing-extensions
fi

# Find where cpp tests and Caffe2 itself are installed
```

- **EN:** This chunk introduces sections such as defaults installs the most recent networkx version, so we install this lower, version explicitly before scikit-image pulls it in as a dependency, click - onnx, Find where cpp tests and Caffe2 itself are installed, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 defaults installs the most recent networkx version, so we install this lower、version explicitly before scikit-image pulls it in as a dependency、click - onnx、Find where cpp tests and Caffe2 itself are installed 等标题组织周边说明或配置。
- **EN:** It invokes commands such as pip, showing the operational steps the workflow performs.
- **CN:** 它调用了 pip 等命令，展示该工作流执行的操作步骤。

### Lines 27-34 / 第 27-34 行

```bash
if [[ "$BUILD_ENVIRONMENT" == *cmake* ]]; then
  # For cmake only build we install everything into /usr/local
  cpp_test_dir="$INSTALL_PREFIX/cpp_test"
  ld_library_path="$INSTALL_PREFIX/lib"
else
  # For Python builds we install into python
  # cd to /usr first so the python import doesn't get confused by any 'caffe2'
  # directory in cwd
```

- **EN:** This chunk introduces sections such as For cmake only build we install everything into /usr/local, For Python builds we install into python, cd to /usr first so the python import doesn't get confused by any 'caffe2', directory in cwd, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 For cmake only build we install everything into /usr/local、For Python builds we install into python、cd to /usr first so the python import doesn't get confused by any 'caffe2'、directory in cwd 等标题组织周边说明或配置。
- **EN:** Environment variables such as BUILD_ENVIRONMENT, INSTALL_PREFIX communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT、INSTALL_PREFIX 等环境变量用于说明所需工具位置或行为开关。

### Lines 35-42 / 第 35-42 行

```bash
  python_installation="$(dirname $(dirname $(cd /usr && $PYTHON -c 'import os; import caffe2; print(os.path.realpath(caffe2.__file__))')))"
  caffe2_pypath="$python_installation/caffe2"
  cpp_test_dir="$python_installation/torch/test"
  ld_library_path="$python_installation/torch/lib"
fi

################################################################################
# C++ tests #
```

- **EN:** This chunk introduces sections such as , C++ tests #, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 、C++ tests # 等标题组织周边说明或配置。
- **EN:** Environment variables such as PYTHON communicate required tool locations or behavioral switches.
- **CN:** PYTHON 等环境变量用于说明所需工具位置或行为开关。

### Lines 43-58 / 第 43-58 行

```bash
################################################################################
# Only run cpp tests in the first shard, don't run cpp tests a second time in the second shard
if [[ "${SHARD_NUMBER:-1}" == "1" ]]; then
  echo "Running C++ tests.."
  for test in $(find "$cpp_test_dir" -executable -type f); do
    case "$test" in
      # skip tests we know are hanging or bad
      */mkl_utils_test|*/aten/integer_divider_test)
        continue
        ;;
      */scalar_tensor_test|*/basic|*/native_test)
        if [[ "$BUILD_ENVIRONMENT" == *rocm* ]]; then
          continue
        else
          LD_LIBRARY_PATH="$ld_library_path" "$test"
        fi
```

- **EN:** This chunk introduces sections such as , Only run cpp tests in the first shard, don't run cpp tests a second time in the second shard, skip tests we know are hanging or bad, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 、Only run cpp tests in the first shard, don't run cpp tests a second time in the second shard、skip tests we know are hanging or bad 等标题组织周边说明或配置。
- **EN:** It invokes commands such as continue, showing the operational steps the workflow performs.
- **CN:** 它调用了 continue 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as SHARD_NUMBER, BUILD_ENVIRONMENT, LD_LIBRARY_PATH communicate required tool locations or behavioral switches.
- **CN:** SHARD_NUMBER、BUILD_ENVIRONMENT、LD_LIBRARY_PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 59-66 / 第 59-66 行

```bash
        ;;
      */*_benchmark)
        LD_LIBRARY_PATH="$ld_library_path" "$test" --benchmark_color=false
        ;;
      *)
        # Currently, we use a mixture of gtest (caffe2) and Catch2 (ATen). While
        # planning to migrate to gtest as the common PyTorch c++ test suite, we
        # currently do NOT use the xml test reporter, because Catch doesn't
```

- **EN:** This chunk introduces sections such as Currently, we use a mixture of gtest (caffe2) and Catch2 (ATen). While, planning to migrate to gtest as the common PyTorch c++ test suite, we, currently do NOT use the xml test reporter, because Catch doesn't, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Currently, we use a mixture of gtest (caffe2) and Catch2 (ATen). While、planning to migrate to gtest as the common PyTorch c++ test suite, we、currently do NOT use the xml test reporter, because Catch doesn't 等标题组织周边说明或配置。
- **EN:** Environment variables such as LD_LIBRARY_PATH, NOT communicate required tool locations or behavioral switches.
- **CN:** LD_LIBRARY_PATH、NOT 等环境变量用于说明所需工具位置或行为开关。

### Lines 67-80 / 第 67-80 行

```bash
        # support multiple reporters
        # c.f. https://github.com/catchorg/Catch2/blob/master/docs/release-notes.md#223
        # which means that enabling XML output means you lose useful stdout
        # output for Jenkins.  It's more important to have useful console
        # output than it is to have XML output for Jenkins.
        # Note: in the future, if we want to use xml test reporter once we switch
        # to all gtest, one can simply do:
        LD_LIBRARY_PATH="$ld_library_path" \
            "$test" --gtest_output=xml:"$gtest_reports_dir/$(basename $test).xml"
        ;;
    esac
  done
fi

```

- **EN:** This chunk introduces sections such as support multiple reporters, c.f. https://github.com/catchorg/Catch2/blob/master/docs/release-notes.md#223, which means that enabling XML output means you lose useful stdout, output for Jenkins.  It's more important to have useful console, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 support multiple reporters、c.f. https://github.com/catchorg/Catch2/blob/master/docs/release-notes.md#223、which means that enabling XML output means you lose useful stdout、output for Jenkins.  It's more important to have useful console 等标题组织周边说明或配置。
- **EN:** Environment variables such as XML, LD_LIBRARY_PATH communicate required tool locations or behavioral switches.
- **CN:** XML、LD_LIBRARY_PATH 等环境变量用于说明所需工具位置或行为开关。

### Lines 81-88 / 第 81-88 行

```bash
################################################################################
# Python tests #
################################################################################
if [[ "$BUILD_ENVIRONMENT" == *cmake* ]]; then
  exit 0
fi

# If pip is installed as root, we must use sudo.
```

- **EN:** This chunk introduces sections such as , Python tests #, , If pip is installed as root, we must use sudo., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 、Python tests #、、If pip is installed as root, we must use sudo. 等标题组织周边说明或配置。
- **EN:** It invokes commands such as exit, showing the operational steps the workflow performs.
- **CN:** 它调用了 exit 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT 等环境变量用于说明所需工具位置或行为开关。

### Lines 89-96 / 第 89-96 行

```bash
# CircleCI docker images could install conda as jenkins user, or use the OS's python package.
PIP=$(which pip)
PIP_USER=$(stat --format '%U' $PIP)
CURRENT_USER=$(id -u -n)
if [[ "$PIP_USER" = root && "$CURRENT_USER" != root ]]; then
  MAYBE_SUDO=sudo
fi

```

- **EN:** This chunk introduces sections such as CircleCI docker images could install conda as jenkins user, or use the OS's python package., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 CircleCI docker images could install conda as jenkins user, or use the OS's python package. 等标题组织周边说明或配置。
- **EN:** Environment variables such as PIP, PIP_USER, CURRENT_USER, MAYBE_SUDO communicate required tool locations or behavioral switches.
- **CN:** PIP、PIP_USER、CURRENT_USER、MAYBE_SUDO 等环境变量用于说明所需工具位置或行为开关。

### Lines 97-107 / 第 97-107 行

```bash
# Uninstall pre-installed hypothesis and coverage to use an older version as newer
# versions remove the timeout parameter from settings which ideep/conv_transpose_test.py uses
$MAYBE_SUDO pip -q uninstall -y hypothesis
$MAYBE_SUDO pip -q uninstall -y coverage

# "pip install hypothesis==3.44.6" from official server is unreliable on
# CircleCI, so we host a copy on S3 instead
$MAYBE_SUDO pip -q install attrs==18.1.0 -f https://s3.amazonaws.com/ossci-linux/wheels/attrs-18.1.0-py2.py3-none-any.whl
$MAYBE_SUDO pip -q install coverage==4.5.1 -f https://s3.amazonaws.com/ossci-linux/wheels/coverage-4.5.1-cp36-cp36m-macosx_10_12_x86_64.whl
$MAYBE_SUDO pip -q install hypothesis==3.44.6 -f https://s3.amazonaws.com/ossci-linux/wheels/hypothesis-3.44.6-py3-none-any.whl

```

- **EN:** This chunk introduces sections such as Uninstall pre-installed hypothesis and coverage to use an older version as newer, versions remove the timeout parameter from settings which ideep/conv_transpose_test.py uses, "pip install hypothesis==3.44.6" from official server is unreliable on, CircleCI, so we host a copy on S3 instead, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Uninstall pre-installed hypothesis and coverage to use an older version as newer、versions remove the timeout parameter from settings which ideep/conv_transpose_test.py uses、"pip install hypothesis==3.44.6" from official server is unreliable on、CircleCI, so we host a copy on S3 instead 等标题组织周边说明或配置。
- **EN:** Environment variables such as MAYBE_SUDO communicate required tool locations or behavioral switches.
- **CN:** MAYBE_SUDO 等环境变量用于说明所需工具位置或行为开关。

### Lines 108-115 / 第 108-115 行

```bash
# Collect additional tests to run (outside caffe2/python)
EXTRA_TESTS=()

# CUDA builds always include NCCL support
if [[ "$BUILD_ENVIRONMENT" == *-cuda* ]] || [[ "$BUILD_ENVIRONMENT" == *-rocm* ]]; then
  EXTRA_TESTS+=("$caffe2_pypath/contrib/nccl")
fi

```

- **EN:** This chunk introduces sections such as Collect additional tests to run (outside caffe2/python), CUDA builds always include NCCL support, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Collect additional tests to run (outside caffe2/python)、CUDA builds always include NCCL support 等标题组织周边说明或配置。
- **EN:** It invokes commands such as EXTRA_TESTS+, showing the operational steps the workflow performs.
- **CN:** 它调用了 EXTRA_TESTS+ 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as EXTRA_TESTS, CUDA, NCCL, BUILD_ENVIRONMENT communicate required tool locations or behavioral switches.
- **CN:** EXTRA_TESTS、CUDA、NCCL、BUILD_ENVIRONMENT 等环境变量用于说明所需工具位置或行为开关。

### Lines 116-123 / 第 116-123 行

```bash
rocm_ignore_test=()
if [[ $BUILD_ENVIRONMENT == *-rocm* ]]; then
  # Currently these tests are failing on ROCM platform:

  # On ROCm, RCCL (distributed) development isn't complete.
  # https://github.com/ROCmSoftwarePlatform/rccl
  rocm_ignore_test+=("--ignore $caffe2_pypath/python/data_parallel_model_test.py")

```

- **EN:** This chunk introduces sections such as Currently these tests are failing on ROCM platform:, On ROCm, RCCL (distributed) development isn't complete., https://github.com/ROCmSoftwarePlatform/rccl, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Currently these tests are failing on ROCM platform:、On ROCm, RCCL (distributed) development isn't complete.、https://github.com/ROCmSoftwarePlatform/rccl 等标题组织周边说明或配置。
- **EN:** It invokes commands such as rocm_ignore_test+, showing the operational steps the workflow performs.
- **CN:** 它调用了 rocm_ignore_test+ 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as BUILD_ENVIRONMENT, ROCM, RCCL communicate required tool locations or behavioral switches.
- **CN:** BUILD_ENVIRONMENT、ROCM、RCCL 等环境变量用于说明所需工具位置或行为开关。

### Lines 124-132 / 第 124-132 行

```bash
  # This test has been flaky in ROCm CI (but note the tests are
  # cpu-only so should be unrelated to ROCm)
  rocm_ignore_test+=("--ignore $caffe2_pypath/python/operator_test/blobs_queue_db_test.py")
  # This test is skipped on Jenkins(compiled without MKL) and otherwise known flaky
  rocm_ignore_test+=("--ignore $caffe2_pypath/python/ideep/convfusion_op_test.py")
  # This test is skipped on Jenkins(compiled without MKL) and causing segfault on Circle
  rocm_ignore_test+=("--ignore $caffe2_pypath/python/ideep/pool_op_test.py")
fi

```

- **EN:** This chunk introduces sections such as This test has been flaky in ROCm CI (but note the tests are, cpu-only so should be unrelated to ROCm), This test is skipped on Jenkins(compiled without MKL) and otherwise known flaky, This test is skipped on Jenkins(compiled without MKL) and causing segfault on Circle, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 This test has been flaky in ROCm CI (but note the tests are、cpu-only so should be unrelated to ROCm)、This test is skipped on Jenkins(compiled without MKL) and otherwise known flaky、This test is skipped on Jenkins(compiled without MKL) and causing segfault on Circle 等标题组织周边说明或配置。
- **EN:** It invokes commands such as rocm_ignore_test+, showing the operational steps the workflow performs.
- **CN:** 它调用了 rocm_ignore_test+ 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as MKL communicate required tool locations or behavioral switches.
- **CN:** MKL 等环境变量用于说明所需工具位置或行为开关。

### Lines 133-142 / 第 133-142 行

```bash
echo "Running Python tests.."
# locale setting is required by click package
for loc in "en_US.utf8" "C.UTF-8"; do
  if locale -a | grep "$loc" >/dev/null 2>&1; then
    export LC_ALL="$loc"
    export LANG="$loc"
    break;
  fi
done

```

- **EN:** This chunk introduces sections such as locale setting is required by click package, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 locale setting is required by click package 等标题组织周边说明或配置。
- **EN:** It invokes commands such as break, showing the operational steps the workflow performs.
- **CN:** 它调用了 break 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as UTF, LC_ALL, LANG communicate required tool locations or behavioral switches.
- **CN:** UTF、LC_ALL、LANG 等环境变量用于说明所需工具位置或行为开关。

### Lines 143-152 / 第 143-152 行

```bash
# Some Caffe2 tests fail when run using AVX512 ISA, see https://github.com/pytorch/pytorch/issues/66111
export DNNL_MAX_CPU_ISA=AVX2

# Should still run even in the absence of SHARD_NUMBER
if [[ "${SHARD_NUMBER:-1}" == "1" ]]; then
  # TODO(sdym@meta.com) remove this when the linked issue resolved.
  # py is temporary until https://github.com/Teemu/pytest-sugar/issues/241 is fixed
  pip install py==1.11.0
  pip install pytest-sugar
  # NB: Warnings are disabled because they make it harder to see what
```

- **EN:** This chunk introduces sections such as Some Caffe2 tests fail when run using AVX512 ISA, see https://github.com/pytorch/pytorch/issues/66111, Should still run even in the absence of SHARD_NUMBER, TODO(sdym@meta.com) remove this when the linked issue resolved., py is temporary until https://github.com/Teemu/pytest-sugar/issues/241 is fixed, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Some Caffe2 tests fail when run using AVX512 ISA, see https://github.com/pytorch/pytorch/issues/66111、Should still run even in the absence of SHARD_NUMBER、TODO(sdym@meta.com) remove this when the linked issue resolved.、py is temporary until https://github.com/Teemu/pytest-sugar/issues/241 is fixed 等标题组织周边说明或配置。
- **EN:** It invokes commands such as pip, showing the operational steps the workflow performs.
- **CN:** 它调用了 pip 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as AVX512, ISA, DNNL_MAX_CPU_ISA, AVX2, SHARD_NUMBER, TODO communicate required tool locations or behavioral switches.
- **CN:** AVX512、ISA、DNNL_MAX_CPU_ISA、AVX2、SHARD_NUMBER、TODO 等环境变量用于说明所需工具位置或行为开关。

### Lines 153-168 / 第 153-168 行

```bash
  # the actual erroring test is
  "$PYTHON" \
    -m pytest \
    -x \
    -v \
    --disable-warnings \
    --junit-xml="$pytest_reports_dir/result.xml" \
    --ignore "$caffe2_pypath/python/test/executor_test.py" \
    --ignore "$caffe2_pypath/python/operator_test/matmul_op_test.py" \
    --ignore "$caffe2_pypath/python/operator_test/pack_ops_test.py" \
    --ignore "$caffe2_pypath/python/mkl/mkl_sbn_speed_test.py" \
    --ignore "$caffe2_pypath/python/trt/test_pt_onnx_trt.py" \
    ${rocm_ignore_test[@]} \
    "$caffe2_pypath/python" \
    "${EXTRA_TESTS[@]}"
fi
```

- **EN:** This chunk introduces sections such as the actual erroring test is, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 the actual erroring test is 等标题组织周边说明或配置。
- **EN:** It invokes commands such as -m, -x, -v, --disable-warnings, --junit-xml, --ignore, showing the operational steps the workflow performs.
- **CN:** 它调用了 -m、-x、-v、--disable-warnings、--junit-xml、--ignore 等命令，展示该工作流执行的操作步骤。
- **EN:** Environment variables such as PYTHON, EXTRA_TESTS communicate required tool locations or behavioral switches.
- **CN:** PYTHON、EXTRA_TESTS 等环境变量用于说明所需工具位置或行为开关。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **Android integration** — 聚焦 Android 构建、打包或运行时集成细节。
- **CMake build logic** — 控制构建系统配置、特性检查或依赖发现。

## Dependencies / 依赖关系

- `"$(dirname`
- `bash`
- `cmake`
- `python`
- `make`
