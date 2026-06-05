# replay_request_dump.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/playground/replay_request_dump.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `replay_request_dump` workflow in SGLang. It mainly handles test execution, benchmark automation. / 该Python 模块用于支撑 SGLang 中的 `replay_request_dump` 流程，主要负责测试执行、基准测试自动化。它属于 `playground` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Usage:
# replay from a folder
python3 replay_request_dump.py --file-number 100 --parallel 512 --input-folder /data/lianmin/sglang_request_dump/engine-34xd1/

# replay from a single file
python3 replay_request_dump.py --parallel 512 --input-file /data/sglang_crash_dump/crash_dump_2025-06-04_20-13-18.pkl
"""
```
**EN:** Usage: # replay from a folder python3 replay_request_dump.py --file-number 100 --parallel 512 --input-folder /data/lianmin/sglang_request_dump/engine-34xd1/ # replay from a single file python3 replay_request_dump.py --parallel 512 --input-file /data/sglang_crash_dump/crash_dump_2025-06-04_20-13-18.pkl
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 10-22: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import glob
import json
import time
from concurrent.futures import ThreadPoolExecutor
from dataclasses import asdict
from datetime import datetime

import requests

from sglang.benchmark.utils import set_ulimit
from sglang.srt.utils.common import safe_pickle_load
from sglang.utils import get_exception_traceback
```
**EN:** This block loads argparse, glob, json, time, concurrent.futures, dataclasses, datetime, requests. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, glob, json, time, concurrent.futures, dataclasses, datetime, requests。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 25-28: Defines the `normalize_mm_data_item` routine / 定义 `normalize_mm_data_item` 例程
```python
def normalize_mm_data_item(item):
    if isinstance(item, dict) and "url" in item:
        return item["url"]
    return item
```
**EN:** This block defines `normalize_mm_data_item`. It accepts 1 parameter(s): item. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `normalize_mm_data_item`。它接收 1 个参数：item。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 31-43: Defines the `normalize_mm_data` routine / 定义 `normalize_mm_data` 例程
```python
def normalize_mm_data(data):
    if data is None:
        return None
    if isinstance(data, list):
        return [
            (
                [normalize_mm_data_item(item) for item in sublist]
                if isinstance(sublist, list)
                else normalize_mm_data_item(sublist)
            )
            for sublist in data
        ]
    return normalize_mm_data_item(data)
```
**EN:** This block defines `normalize_mm_data`. It accepts 1 parameter(s): data. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `normalize_mm_data`。它接收 1 个参数：data。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 46-51: Defines the `normalize_request_data` routine / 定义 `normalize_request_data` 例程
```python
def normalize_request_data(json_data):
    """Normalize multimodal fields in request data for replay compatibility."""
    for field in ["image_data", "video_data", "audio_data"]:
        if field in json_data and json_data[field] is not None:
            json_data[field] = normalize_mm_data(json_data[field])
    return json_data
```
**EN:** This block defines `normalize_request_data`. Normalize multimodal fields in request data for replay compatibility. It accepts 1 parameter(s): json_data. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `normalize_request_data`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：json_data。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 54-64: Defines the `read_records` routine / 定义 `read_records` 例程
```python
def read_records(files):
    records = []
    for f in files:
        with open(f, "rb") as fh:
            tmp = safe_pickle_load(fh)
        if isinstance(tmp, dict) and "requests" in tmp:
            records.extend(tmp["requests"])
        else:
            records.extend(tmp)

    return records
```
**EN:** This block defines `read_records`. It accepts 1 parameter(s): files. Internally it reads or writes files.
**CN:** 该代码块定义了 `read_records`。它接收 1 个参数：files。内部会读写文件。

### Lines 67-105: Defines the `run_one_request_internal` routine / 定义 `run_one_request_internal` 例程
```python
def run_one_request_internal(record):
    req, output, replay_init_time, start_time, end_time, idx = record
    time.sleep(max(0, (start_time - (time.time() - replay_init_time)) / args.speed))

    if "completion_tokens" in output.get("meta_info", {}):
        recorded_completion_tokens = output["meta_info"]["completion_tokens"]
    else:
        recorded_completion_tokens = ""

    json_data = normalize_request_data(asdict(req))
    stream = json_data["stream"]

    if args.ignore_eos:
        json_data["sampling_params"]["ignore_eos"] = True
        if recorded_completion_tokens:
            json_data["sampling_params"]["max_new_tokens"] = recorded_completion_tokens

    response = requests.post(
        f"http://{args.host}:{args.port}/generate",
        json=json_data,
        stream=stream,
    )

    if stream:
        for chunk in response.iter_lines(decode_unicode=False):
            chunk = chunk.decode("utf-8")
            if chunk and chunk.startswith("data:"):
                if chunk == "data: [DONE]":
                    break
                ret = json.loads(chunk[5:].strip("\n"))
    else:
        ret = response.json()

    prompt_tokens = ret["meta_info"]["prompt_tokens"]
    completion_tokens = ret["meta_info"]["completion_tokens"]
    print(
        f"{idx=}, {start_time=:.2f}, {prompt_tokens=}, "
        f"{completion_tokens=}, {recorded_completion_tokens=}"
    )
```
**EN:** This block defines `run_one_request_internal`. It accepts 1 parameter(s): record. Internally it loads structured data, emits status messages.
**CN:** 该代码块定义了 `run_one_request_internal`。它接收 1 个参数：record。内部会加载结构化数据、输出状态信息。

### Lines 108-117: Defines the `run_one_request` routine / 定义 `run_one_request` 例程
```python
def run_one_request(record):
    # global success_ct, error_ct

    try:
        run_one_request_internal(record)
        # success_ct += 1
    except Exception:
        # error_ct += 1
        traceback = get_exception_traceback()
        print(f"Hit an exception: {traceback}")
```
**EN:** This block defines `run_one_request`. It accepts 1 parameter(s): record. Internally it emits status messages.
**CN:** 该代码块定义了 `run_one_request`。它接收 1 个参数：record。内部会输出状态信息。

### Lines 120-135: Defines the `main` routine / 定义 `main` 例程
```python
def main(records):
    if len(records) == 0:
        return

    base_time = records[0][-2]
    base_time_str = datetime.fromtimestamp(base_time).strftime("%y-%m-%d %H:%M:%S")
    print(f"{base_time_str=}")
    replay_init_time = time.time()

    for i in range(len(records)):
        req, output, start_time, end_time = records[i]
        start_time -= base_time
        records[i] = (req, output, replay_init_time, start_time, end_time, i)

    with ThreadPoolExecutor(args.parallel) as executor:
        executor.map(run_one_request, records)
```
**EN:** This block defines `main`. It accepts 1 parameter(s): records. Internally it emits status messages.
**CN:** 该代码块定义了 `main`。它接收 1 个参数：records。内部会输出状态信息。

### Lines 138-181: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--host", type=str, default="localhost")
    parser.add_argument("--port", type=int, default=30000)
    parser.add_argument(
        "--input-folder", type=str, default=None, help="Folder containing pickle files"
    )
    parser.add_argument(
        "--input-file", type=str, default=None, help="Single pickle file to process"
    )
    parser.add_argument("--file-number", type=int, default=1)
    parser.add_argument("--req-number", type=int, default=1000000)
    parser.add_argument("--req-start", type=int, default=0)
    parser.add_argument("--parallel", type=int, default=512)
    parser.add_argument("--idx", type=int, default=None)
    parser.add_argument("--ignore-eos", action="store_true")
    parser.add_argument("--speed", type=float, default=1)
    args = parser.parse_args()

    set_ulimit()

    files = []
    if args.input_file:
        files = [args.input_file]
        if args.file_number > 1:
            print("Warning: --file-number is ignored when --input-file is provided.")
    elif args.input_folder:
        files = glob.glob(f"{args.input_folder}/*.pkl")
        files = files[: args.file_number]
    else:
        print("Error: Either --input-folder or --input-file must be provided.")
        exit(1)
    print(f"{files=}")

    records = read_records(files)
    # Sort by the receive time, before filtering
    records.sort(key=lambda x: x[-2])
    records = records[args.req_start :]
    if args.idx:
        records = [records[args.idx]]
        print(f"testing {args.idx=}")
        print(f"{records[0]}")
    print(f"{len(records)=}")
    main(records)
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It controls process exit status, parses CLI arguments, emits status messages.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会控制进程退出状态、解析命令行参数、输出状态信息。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Environment management** / 环境管理
- **Structured data handling** / 结构化数据处理
- **HTTP integration** / HTTP 集成
- **Performance benchmarking** / 性能基准测试

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `concurrent`, `dataclasses`, `datetime`, `glob`, `json`, `time`
- **Third-party modules / 第三方模块**: `requests`
- **Repository-local imports / 仓库内导入**: `sglang.benchmark.utils`, `sglang.srt.utils.common`, `sglang.utils`
