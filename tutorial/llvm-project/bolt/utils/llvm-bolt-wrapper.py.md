# llvm-bolt-wrapper.py — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/utils/llvm-bolt-wrapper.py`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: !/usr/bin/env python3. It also sits in code that belongs to the BOLT post-link optimizer subsystem. / 该文件属于 BOLT 链接后优化器子系统。 源码头部说明其职责是：!/usr/bin/env python3。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14

```python
#!/usr/bin/env python3
import argparse
import subprocess
from typing import *
import tempfile
import copy
import os
import shutil
import sys
import re
import configparser
from types import SimpleNamespace
from textwrap import dedent
```

- EN: Imports Python modules such as `argparse`, `subprocess`, `from typing import *`, `tempfile`, and 8 more to support scripting logic. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `argparse`, `subprocess`, `from typing import *`, `tempfile`, `copy`, `os`.
- CN: 这里导入 Python 模块，例如 `argparse`, `subprocess`, `from typing import *`, `tempfile`, and 8 more，以支撑脚本逻辑。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `argparse`, `subprocess`, `from typing import *`, `tempfile`, `copy`, `os`。

### Lines 15-27

```python
# USAGE:
# 0. Prepare two BOLT build versions: base and compare.
# 1. Create the config by invoking this script with required options.
#    Save the config as `llvm-bolt-wrapper.ini` next to the script or
#    in the testing directory.
# In the base BOLT build directory:
# 2. Rename `llvm-bolt` to `llvm-bolt.real`
# 3. Create a symlink from this script to `llvm-bolt`
# 4. Create `llvm-bolt-wrapper.ini` and fill it using the example below.
#
# This script will compare binaries produced by base and compare BOLT, and
# report elapsed processing time and max RSS.
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 28-43

```python
# read options from config file llvm-bolt-wrapper.ini in script CWD
#
# [config]
# # mandatory
# base_bolt = /full/path/to/llvm-bolt.real
# cmp_bolt = /full/path/to/other/llvm-bolt
# # optional, default to False
# verbose
# keep_tmp
# no_minimize
# run_sequentially
# compare_output
# skip_binary_cmp
# # optional, defaults to timing.log in CWD
# timing_file = timing1.log
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 44-52

```python

def read_cfg():
    src_dir = os.path.dirname(os.path.abspath(__file__))
    cfg = configparser.ConfigParser(allow_no_value=True)
    cfgs = cfg.read("llvm-bolt-wrapper.ini")
    if not cfgs:
        cfgs = cfg.read(os.path.join(src_dir, "llvm-bolt-wrapper.ini"))
    assert cfgs, f"llvm-bolt-wrapper.ini is not found in {os.getcwd()}"
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 53-62

```python
    def get_cfg(key):
        # if key is not present in config, assume False
        if key not in cfg["config"]:
            return False
        # if key is present, but has no value, assume True
        if not cfg["config"][key]:
            return True
        # if key has associated value, interpret the value
        return cfg["config"].getboolean(key)
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 63-79

```python
    d = {
        # BOLT binary locations
        "BASE_BOLT": cfg["config"]["base_bolt"],
        "CMP_BOLT": cfg["config"]["cmp_bolt"],
        # optional
        "VERBOSE": get_cfg("verbose"),
        "KEEP_TMP": get_cfg("keep_tmp"),
        "NO_MINIMIZE": get_cfg("no_minimize"),
        "RUN_SEQUENTIALLY": get_cfg("run_sequentially"),
        "COMPARE_OUTPUT": get_cfg("compare_output"),
        "SKIP_BINARY_CMP": get_cfg("skip_binary_cmp"),
        "TIMING_FILE": cfg["config"].get("timing_file", "timing.log"),
    }
    if d["VERBOSE"]:
        print(f"Using config {os.path.abspath(cfgs[0])}")
    return SimpleNamespace(**d)
```

- EN: Declares or implements routines including `get_cfg`, `print`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `get_cfg`, `print`.
- CN: 这里声明或实现函数，例如 `get_cfg`, `print`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `get_cfg`, `print`。

### Lines 80-89

```python

# perf2bolt mode
PERF2BOLT_MODE = ["-aggregate-only", "-ignore-build-id", "-show-density"]

# boltdiff mode
BOLTDIFF_MODE = ["-diff-only", "-o", "/dev/null"]

# options to suppress binary differences as much as possible
MINIMIZE_DIFFS = ["-bolt-info=0"]
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 90-104

```python
# bolt output options that need to be intercepted
BOLT_OUTPUT_OPTS = {
    "-o": "BOLT output binary",
    "-w": "BOLT recorded profile",
}

# regex patterns to exclude the line from log comparison
SKIP_MATCH = [
    "BOLT-INFO: BOLT version",
    r"^Args: ",
    r"^BOLT-DEBUG:",
    r"BOLT-INFO:.*data.*output data",
    "WARNING: reading perf data directly",
]
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 105-122

```python

def run_cmd(cmd, out_f, cfg):
    if cfg.VERBOSE:
        print(" ".join(cmd))
    return subprocess.Popen(cmd, stdout=out_f, stderr=subprocess.STDOUT)


def run_bolt(bolt_path, bolt_args, out_f, cfg):
    p2b = os.path.basename(sys.argv[0]) == "perf2bolt"  # perf2bolt mode
    bd = os.path.basename(sys.argv[0]) == "llvm-boltdiff"  # boltdiff mode
    cmd = ["/usr/bin/time", "-f", "%e %M", bolt_path] + bolt_args
    if p2b:
        # -ignore-build-id can occur at most once, hence remove it from cmd
        if "-ignore-build-id" in cmd:
            cmd.remove("-ignore-build-id")
        cmd += PERF2BOLT_MODE
    elif bd:
        cmd += BOLTDIFF_MODE
```

- EN: Declares or implements routines including `print`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `print`.
- CN: 这里声明或实现函数，例如 `print`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `print`。

### Lines 123-137

```python
    elif not cfg.NO_MINIMIZE:
        cmd += MINIMIZE_DIFFS
    return run_cmd(cmd, out_f, cfg)


def prepend_dash(args: Mapping[AnyStr, AnyStr]) -> Sequence[AnyStr]:
    """
    Accepts parsed arguments and returns flat list with dash prepended to
    the option.
    Example: Namespace(o='test.tmp') -> ['-o', 'test.tmp']
    """
    dashed = [("-" + key, value) for (key, value) in args.items()]
    flattened = list(sum(dashed, ()))
    return flattened
```

- EN: Declares or implements routines including `Namespace`, `list`. Notable symbols here include `Namespace`, `list`.
- CN: 这里声明或实现函数，例如 `Namespace`, `list`。这里较值得关注的符号包括 `Namespace`, `list`。

### Lines 138-152

```python

def replace_cmp_path(tmp: AnyStr, args: Mapping[AnyStr, AnyStr]) -> Sequence[AnyStr]:
    """
    Keeps file names, but replaces the path to a temp folder.
    Example: Namespace(o='abc/test.tmp') -> Namespace(o='/tmp/tmpf9un/test.tmp')
    Except preserve /dev/null.
    """
    replace_path = (
        lambda x: os.path.join(tmp, os.path.basename(x))
        if x != "/dev/null"
        else "/dev/null"
    )
    new_args = {key: replace_path(value) for key, value in args.items()}
    return prepend_dash(new_args)
```

- EN: Declares or implements routines including `Namespace`, `replace_path`. Notable symbols here include `Namespace`, `replace_path`.
- CN: 这里声明或实现函数，例如 `Namespace`, `replace_path`。这里较值得关注的符号包括 `Namespace`, `replace_path`。

### Lines 153-160

```python

def preprocess_args(args: argparse.Namespace) -> Mapping[AnyStr, AnyStr]:
    """
    Drop options that weren't parsed (e.g. -w), convert to a dict
    """
    return {key: value for key, value in vars(args).items() if value}
```

- EN: Declares or implements routines including `parsed`. Notable symbols here include `parsed`.
- CN: 这里声明或实现函数，例如 `parsed`。这里较值得关注的符号包括 `parsed`。

### Lines 161-170

```python
def write_to(txt, filename, mode="w"):
    with open(filename, mode) as f:
        f.write(txt)


def wait(proc, fdesc):
    proc.wait()
    fdesc.close()
    return open(fdesc.name)
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 171-188

```python

def compare_logs(main, cmp, skip_begin=0, skip_end=0, str_input=True):
    """
    Compares logs but allows for certain lines to be excluded from comparison.
    If str_input is True (default), the input it assumed to be a string,
    which is split into lines. Otherwise the input is assumed to be a file.
    Returns None on success, mismatch otherwise.
    """
    main_inp = main.splitlines() if str_input else main.readlines()
    cmp_inp = cmp.splitlines() if str_input else cmp.readlines()
    # rewind logs after consumption
    if not str_input:
        main.seek(0)
        cmp.seek(0)
    for lhs, rhs in list(zip(main_inp, cmp_inp))[skip_begin : -skip_end or None]:
        if lhs != rhs:
            # check skip patterns
            for skip in SKIP_MATCH:
```

- EN: Declares or implements routines including `True`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `True`.
- CN: 这里声明或实现函数，例如 `True`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `True`。

### Lines 189-196

```python
                # both lines must contain the pattern
                if re.search(skip, lhs) and re.search(skip, rhs):
                    break
            # otherwise return mismatching lines
            else:
                return (lhs, rhs)
    return None
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 197-214

```python

def fmt_cmp(cmp_tuple):
    if not cmp_tuple:
        return ""
    return f"main:\n{cmp_tuple[0]}\ncmp:\n{cmp_tuple[1]}\n"


def compare_with(lhs, rhs, cmd, skip_begin=0, skip_end=0):
    """
    Runs cmd on both lhs and rhs and compares stdout.
    Returns tuple (mismatch, lhs_stdout):
        - if stdout matches between two files, mismatch is None,
        - otherwise mismatch is a tuple of mismatching lines.
    """
    run = lambda binary: subprocess.run(
        cmd.split() + [binary], text=True, check=True, capture_output=True
    ).stdout
    run_lhs = run(lhs)
```

- EN: Declares or implements routines including `run`. Notable symbols here include `run`.
- CN: 这里声明或实现函数，例如 `run`。这里较值得关注的符号包括 `run`。

### Lines 215-227

```python
    run_rhs = run(rhs)
    cmp = compare_logs(run_lhs, run_rhs, skip_begin, skip_end)
    return cmp, run_lhs


def parse_cmp_offset(cmp_out):
    """
    Extracts byte number from cmp output:
    file1 file2 differ: byte X, line Y
    """
    # NOTE: cmp counts bytes starting from 1!
    return int(re.search(r"byte (\d+),", cmp_out).groups()[0]) - 1
```

- EN: Declares or implements routines including `run`, `compare_logs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `run`, `compare_logs`.
- CN: 这里声明或实现函数，例如 `run`, `compare_logs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `run`, `compare_logs`。

### Lines 228-237

```python

def report_real_time(binary, main_err, cmp_err, cfg):
    """
    Extracts real time from stderr and appends it to TIMING FILE it as csv:
    "output binary; base bolt; cmp bolt"
    """

    def get_real_from_stderr(logline):
        return "; ".join(logline.split())
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 238-248

```python
    for line in main_err:
        pass
    main = get_real_from_stderr(line)
    for line in cmp_err:
        pass
    cmp = get_real_from_stderr(line)
    write_to(f"{binary}; {main}; {cmp}\n", cfg.TIMING_FILE, "a")
    # rewind logs after consumption
    main_err.seek(0)
    cmp_err.seek(0)
```

- EN: Declares or implements routines including `get_real_from_stderr`, `write_to`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `get_real_from_stderr`, `write_to`.
- CN: 这里声明或实现函数，例如 `get_real_from_stderr`, `write_to`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `get_real_from_stderr`, `write_to`。

### Lines 249-258

```python

def clean_exit(tmp, out, exitcode, cfg):
    # temp files are only cleaned on success
    if not cfg.KEEP_TMP:
        shutil.rmtree(tmp)

    # report stdout and stderr from the main process
    shutil.copyfileobj(out, sys.stdout)
    sys.exit(exitcode)
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 259-276

```python

def find_section(offset, readelf_hdr):
    hdr = readelf_hdr.split("\n")
    section = None
    # extract sections table (parse objdump -hw output)
    for line in hdr[5:-1]:
        cols = line.strip().split()
        # extract section offset
        file_offset = int(cols[5], 16)
        # section size
        size = int(cols[2], 16)
        if offset >= file_offset and offset < file_offset + size:
            if sys.stdout.isatty():  # terminal supports colors
                print(f"\033[1m{line}\033[0m")
            else:
                print(f">{line}")
            section = cols[1]
        else:
```

- EN: Declares or implements routines including `int`, `print`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `int`, `print`.
- CN: 这里声明或实现函数，例如 `int`, `print`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `int`, `print`。

### Lines 277-294

```python
            print(line)
    return section


def main_config_generator():
    parser = argparse.ArgumentParser()
    parser.add_argument("base_bolt", help="Full path to base llvm-bolt binary")
    parser.add_argument("cmp_bolt", help="Full path to cmp llvm-bolt binary")
    parser.add_argument(
        "--verbose",
        action="store_true",
        help="Print subprocess invocation cmdline (default False)",
    )
    parser.add_argument(
        "--keep_tmp",
        action="store_true",
        help="Preserve tmp folder on a clean exit "
        "(tmp directory is preserved on crash by default)",
```

- EN: Declares or implements routines including `print`, `cmdline`. Notable symbols here include `print`, `cmdline`.
- CN: 这里声明或实现函数，例如 `print`, `cmdline`。这里较值得关注的符号包括 `print`, `cmdline`。

### Lines 295-312

```python
    )
    parser.add_argument(
        "--no_minimize",
        action="store_true",
        help=f"Do not add `{MINIMIZE_DIFFS}` that is used "
        "by default to reduce binary differences",
    )
    parser.add_argument(
        "--run_sequentially",
        action="store_true",
        help="Run both binaries sequentially (default "
        "in parallel). Use for timing comparison",
    )
    parser.add_argument(
        "--compare_output",
        action="store_true",
        help="Compare bolt stdout/stderr (disabled by default)",
    )
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 313-321

```python
    parser.add_argument(
        "--skip_binary_cmp", action="store_true", help="Disable output comparison"
    )
    parser.add_argument(
        "--timing_file",
        help="Override path to timing log " "file (default `timing.log` in CWD)",
    )
    args = parser.parse_args()
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 322-339

```python
    print(
        dedent(
            f"""\
    [config]
    # mandatory
    base_bolt = {args.base_bolt}
    cmp_bolt = {args.cmp_bolt}"""
        )
    )
    del args.base_bolt
    del args.cmp_bolt
    d = vars(args)
    if any(d.values()):
        print("# optional")
        for key, value in d.items():
            if value:
                print(key)
```

- EN: Declares or implements routines including `vars`, `print`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `vars`, `print`.
- CN: 这里声明或实现函数，例如 `vars`, `print`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `vars`, `print`。

### Lines 340-352

```python

def main():
    cfg = read_cfg()
    # intercept output arguments
    parser = argparse.ArgumentParser(add_help=False)
    for option, help in BOLT_OUTPUT_OPTS.items():
        parser.add_argument(option, help=help)
    args, unknownargs = parser.parse_known_args()
    args = preprocess_args(args)
    cmp_args = copy.deepcopy(args)
    tmp = tempfile.mkdtemp()
    cmp_args = replace_cmp_path(tmp, cmp_args)
```

- EN: Declares or implements routines including `read_cfg`, `preprocess_args`, `replace_cmp_path`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `read_cfg`, `preprocess_args`, `replace_cmp_path`.
- CN: 这里声明或实现函数，例如 `read_cfg`, `preprocess_args`, `replace_cmp_path`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `read_cfg`, `preprocess_args`, `replace_cmp_path`。

### Lines 353-367

```python
    # reconstruct output arguments: prepend dash
    args = prepend_dash(args)

    # run both BOLT binaries
    main_f = open(os.path.join(tmp, "main_bolt.stdout"), "w")
    cmp_f = open(os.path.join(tmp, "cmp_bolt.stdout"), "w")
    main_bolt = run_bolt(cfg.BASE_BOLT, unknownargs + args, main_f, cfg)
    if cfg.RUN_SEQUENTIALLY:
        main_out = wait(main_bolt, main_f)
        cmp_bolt = run_bolt(cfg.CMP_BOLT, unknownargs + cmp_args, cmp_f, cfg)
    else:
        cmp_bolt = run_bolt(cfg.CMP_BOLT, unknownargs + cmp_args, cmp_f, cfg)
        main_out = wait(main_bolt, main_f)
    cmp_out = wait(cmp_bolt, cmp_f)
```

- EN: Declares or implements routines including `prepend_dash`, `open`, `run_bolt`, `wait`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `prepend_dash`, `open`, `run_bolt`, `wait`.
- CN: 这里声明或实现函数，例如 `prepend_dash`, `open`, `run_bolt`, `wait`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `prepend_dash`, `open`, `run_bolt`, `wait`。

### Lines 368-376

```python
    # check exit code
    if main_bolt.returncode != cmp_bolt.returncode:
        print(tmp)
        exit("exitcode mismatch")

    # don't compare output upon unsuccessful exit
    if main_bolt.returncode != 0:
        cfg.SKIP_BINARY_CMP = True
```

- EN: Declares or implements routines including `print`, `exit`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `print`, `exit`.
- CN: 这里声明或实现函数，例如 `print`, `exit`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `print`, `exit`。

### Lines 377-388

```python
    # compare logs, skip_end=1 skips the line with time
    out = (
        compare_logs(main_out, cmp_out, skip_end=1, str_input=False)
        if cfg.COMPARE_OUTPUT
        else None
    )
    if out:
        print(tmp)
        print(fmt_cmp(out))
        write_to(fmt_cmp(out), os.path.join(tmp, "summary.txt"))
        exit("logs mismatch")
```

- EN: Declares or implements routines including `compare_logs`, `print`, `write_to`, `exit`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `compare_logs`, `print`, `write_to`, `exit`.
- CN: 这里声明或实现函数，例如 `compare_logs`, `print`, `write_to`, `exit`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `compare_logs`, `print`, `write_to`, `exit`。

### Lines 389-399

```python
    if os.path.basename(sys.argv[0]) == "llvm-boltdiff":  # boltdiff mode
        # no output binary to compare, so just exit
        clean_exit(tmp, main_out, main_bolt.returncode, cfg)

    # compare binaries (using cmp)
    main_binary = args[args.index("-o") + 1]
    cmp_binary = cmp_args[cmp_args.index("-o") + 1]
    if main_binary == "/dev/null":
        assert cmp_binary == "/dev/null"
        cfg.SKIP_BINARY_CMP = True
```

- EN: Declares or implements routines including `clean_exit`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `clean_exit`.
- CN: 这里声明或实现函数，例如 `clean_exit`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `clean_exit`。

### Lines 400-417

```python
    # report binary timing as csv: output binary; base bolt real; cmp bolt real
    report_real_time(main_binary, main_out, cmp_out, cfg)

    if not cfg.SKIP_BINARY_CMP:
        # check if files exist
        main_exists = os.path.exists(main_binary)
        cmp_exists = os.path.exists(cmp_binary)
        if main_exists and cmp_exists:
            # proceed to comparison
            pass
        elif not main_exists and not cmp_exists:
            # both don't exist, assume it's intended, skip comparison
            clean_exit(tmp, main_out, main_bolt.returncode, cfg)
        elif main_exists:
            assert not cmp_exists
            exit(f"{cmp_binary} doesn't exist")
        else:
            assert not main_exists
```

- EN: Declares or implements routines including `report_real_time`, `clean_exit`, `exit`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `report_real_time`, `clean_exit`, `exit`.
- CN: 这里声明或实现函数，例如 `report_real_time`, `clean_exit`, `exit`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `report_real_time`, `clean_exit`, `exit`。

### Lines 418-435

```python
            exit(f"{main_binary} doesn't exist")

        cmp_proc = subprocess.run(
            ["cmp", "-b", main_binary, cmp_binary], capture_output=True, text=True
        )
        if cmp_proc.returncode:
            # check if output is an ELF file (magic bytes)
            with open(main_binary, "rb") as f:
                magic = f.read(4)
                if magic != b"\x7fELF":
                    exit("output mismatch")
            # check if ELF headers match
            mismatch, _ = compare_with(main_binary, cmp_binary, "readelf -We")
            if mismatch:
                print(fmt_cmp(mismatch))
                write_to(fmt_cmp(mismatch), os.path.join(tmp, "headers.txt"))
                exit("headers mismatch")
            # if headers match, compare sections (skip line with filename)
```

- EN: Declares or implements routines including `exit`, `compare_with`, `print`, `write_to`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `exit`, `compare_with`, `print`, `write_to`.
- CN: 这里声明或实现函数，例如 `exit`, `compare_with`, `print`, `write_to`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `exit`, `compare_with`, `print`, `write_to`。

### Lines 436-444

```python
            mismatch, hdr = compare_with(
                main_binary, cmp_binary, "objdump -hw", skip_begin=2
            )
            assert not mismatch
            # check which section has the first mismatch
            mismatch_offset = parse_cmp_offset(cmp_proc.stdout)
            section = find_section(mismatch_offset, hdr)
            exit(f"binary mismatch @{hex(mismatch_offset)} ({section})")
```

- EN: Declares or implements routines including `parse_cmp_offset`, `find_section`, `exit`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `parse_cmp_offset`, `find_section`, `exit`.
- CN: 这里声明或实现函数，例如 `parse_cmp_offset`, `find_section`, `exit`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `parse_cmp_offset`, `find_section`, `exit`。

### Lines 445-454

```python
    clean_exit(tmp, main_out, main_bolt.returncode, cfg)


if __name__ == "__main__":
    # config generator mode if the script is launched as is
    if os.path.basename(__file__) == "llvm-bolt-wrapper.py":
        main_config_generator()
    else:
        # llvm-bolt interceptor mode otherwise
        main()
```

- EN: Declares or implements routines including `clean_exit`, `main_config_generator`, `main`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `clean_exit`, `main_config_generator`, `main`.
- CN: 这里声明或实现函数，例如 `clean_exit`, `main_config_generator`, `main`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `clean_exit`, `main_config_generator`, `main`。

## Key Concepts / 关键概念

- `get_cfg`: function or method entry point / 函数或方法入口
- `print`: function or method entry point / 函数或方法入口
- `Namespace`: function or method entry point / 函数或方法入口
- `list`: function or method entry point / 函数或方法入口
- `replace_path`: function or method entry point / 函数或方法入口
- `argparse`: imported dependency / 导入的依赖
- `subprocess`: imported dependency / 导入的依赖

## Dependencies / 依赖关系

- Python imports / Python 导入: `argparse`, `subprocess`, `from typing import *`, `tempfile`, `copy`, `os`, `shutil`, `sys`, `re`, `configparser`, `from types import SimpleNamespace`, `from textwrap import dedent`
- Directory context / 目录上下文: `bolt/utils` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/utils` 下的相邻文件通常与本文件协作组成对应子系统
