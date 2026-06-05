# convert_otel_2_perfetto.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/convert_otel_2_perfetto.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `convert_otel_2_perfetto` workflow in SGLang. It mainly handles CI orchestration, build automation. / 该Python 模块用于支撑 SGLang 中的 `convert_otel_2_perfetto` 流程，主要负责CI 编排、构建自动化。它属于 `scripts` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import bisect
import json
import time
from collections import defaultdict
from pathlib import Path
from typing import Any, Dict, Iterable, List, Tuple
```
**EN:** This block loads argparse, bisect, json, time, collections, pathlib, typing. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, bisect, json, time, collections, pathlib, typing。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 9-12: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
parser = argparse.ArgumentParser(
    description="Convert SGLang OTEL trace files to Perfetto format.",
    formatter_class=argparse.ArgumentDefaultsHelpFormatter,
)
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 13-20: Top-level expr logic / 顶层 expr 逻辑
```python
parser.add_argument(
    "-i",
    "--input",
    dest="input_file",
    required=True,
    type=str,
    help="Path to the input OTEL trace file (JSON or JSONL format).",
)
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 21-28: Top-level expr logic / 顶层 expr 逻辑
```python
parser.add_argument(
    "-o",
    "--output",
    dest="output_file",
    type=str,
    default="sglang_trace_perfetto.json",
    help="Path to the output Perfetto JSON file.",
)
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 29-31: Top-level expr logic / 顶层 expr 逻辑
```python
parser.add_argument(
    "-f", "--torch-file", dest="torch_file", help="specify torch profile file"
)
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 33-35: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
args = parser.parse_args()

perfetto_data = None
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 36-41: Top-level if logic / 顶层 if 逻辑
```python
if args.torch_file:
    with open(args.torch_file, "r", encoding="utf-8") as file:
        perfetto_data = json.load(file)
        baseline = perfetto_data["baseTimeNanoseconds"]
else:
    baseline = 0
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 44-48: Defines the `id_generator` routine / 定义 `id_generator` 例程
```python
def id_generator():
    i = 0
    while True:
        yield i
        i += 1
```
**EN:** This block defines `id_generator`. It takes no explicit parameters at the top level. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `id_generator`。它在顶层定义中不接收显式参数。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 51-51: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
relation_id_gen = id_generator()
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 54-73: Declares the `SpanLayoutContainer` class / 声明 `SpanLayoutContainer` 类
```python
class SpanLayoutContainer:
    def __init__(self):
        self.intervals = []

    def check_overlap(self, start, end):
        idx = bisect.bisect_left(self.intervals, (start, float("-inf")))

        if idx > 0:
            prev_start, prev_end = self.intervals[idx - 1]
            if prev_end > start:
                return True

        if idx < len(self.intervals):
            next_start, next_end = self.intervals[idx]
            if next_start < end:
                return True
        return False

    def insert_span(self, start, end):
        bisect.insort_left(self.intervals, (start, end))
```
**EN:** This block declares the `SpanLayoutContainer` class. The class groups 3 method(s), including __init__, check_overlap, insert_span.
**CN:** 该代码块声明了 `SpanLayoutContainer` 类。该类集中定义了 3 个方法，包括 __init__, check_overlap, insert_span。

### Lines 76-82: Defines the `new_metadata_level1` routine / 定义 `new_metadata_level1` 例程
```python
def new_metadata_level1(name: str, pid):
    return {
        "name": "process_name",
        "ph": "M",
        "pid": pid,
        "args": {"name": name},
    }
```
**EN:** This block defines `new_metadata_level1`. It accepts 2 parameter(s): name, pid. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `new_metadata_level1`。它接收 2 个参数：name, pid。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 85-92: Defines the `new_metadata_level2` routine / 定义 `new_metadata_level2` 例程
```python
def new_metadata_level2(name: str, pid, slot_seq):
    return {
        "name": "thread_name",
        "ph": "M",
        "pid": pid,
        "tid": slot_seq,
        "args": {"name": name},
    }
```
**EN:** This block defines `new_metadata_level2`. It accepts 3 parameter(s): name, pid, slot_seq. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `new_metadata_level2`。它接收 3 个参数：name, pid, slot_seq。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 95-120: Defines the `__find_line` routine / 定义 `__find_line` 例程
```python
def __find_line(graph, trans_graph_status, slot_meta_data, pid, start, end):
    if pid in trans_graph_status:
        line = trans_graph_status[pid]
        if start == end:
            return line
        # check conflict
        if not graph[pid][line].check_overlap(start, end):
            return line

    if pid not in graph:
        line = 1
        graph[pid] = {line: SpanLayoutContainer()}
        trans_graph_status[pid] = line
        slot_meta_data.append(new_metadata_level2("slot", pid, line))
        return line

    for line in graph[pid]:
        if not graph[pid][line].check_overlap(start, end):
            trans_graph_status[pid] = line
            return line

    new_line = len(graph[pid]) + 1
    graph[pid][new_line] = SpanLayoutContainer()
    trans_graph_status[pid] = new_line
    slot_meta_data.append(new_metadata_level2("slot", pid, new_line))
    return new_line
```
**EN:** This block defines `__find_line`. It accepts 6 parameter(s): graph, trans_graph_status, slot_meta_data, pid, start, end. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `__find_line`。它接收 6 个参数：graph, trans_graph_status, slot_meta_data, pid, start, end。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 123-123: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
OtelSpan = Dict[str, Any]
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 126-135: Defines the `load_otel_data` routine / 定义 `load_otel_data` 例程
```python
def load_otel_data(path: str | Path):
    p = Path(path)
    with p.open("rt", encoding="utf-8") as f:
        first = f.read(1)
        f.seek(0)
        if first == "[":
            data = json.load(f)  # JSON array
        else:
            data = [json.loads(line) for line in f if line.strip()]  # JSONL
    return data
```
**EN:** This block defines `load_otel_data`. It accepts 1 parameter(s): path. Internally it touches filesystem paths, loads structured data, reads or writes files.
**CN:** 该代码块定义了 `load_otel_data`。它接收 1 个参数：path。内部会处理文件系统路径、加载结构化数据、读写文件。

### Lines 138-169: Defines the `extract_all_otel_spans` routine / 定义 `extract_all_otel_spans` 例程
```python
def extract_all_otel_spans(otel_data):
    engine_otel_spans = []
    smg_otel_spans = []
    for line_data in otel_data:
        for resource_spans in line_data["resourceSpans"]:
            # filter: only keep spans which service.name is 'sglang' or 'smg'
            service_name = ""
            for attr in resource_spans["resource"]["attributes"]:
                if attr["key"] == "service.name":
                    service_name = attr["value"]["stringValue"]

            if service_name == "sglang":
                spans_ref = engine_otel_spans
            elif service_name == "smg":
                spans_ref = smg_otel_spans
            else:
                continue

            for scope_spans in resource_spans["scopeSpans"]:
                for span in scope_spans["spans"]:
                    if "attributes" in span:
                        attributes_dict = {
                            attr.get("key"): next(
                                iter(attr.get("value", {}).values()), None
                            )
                            for attr in span["attributes"]
                        }
                        span["attributes"] = attributes_dict
                    else:
                        span["attributes"] = {}
                    spans_ref.append(span)
    return engine_otel_spans, smg_otel_spans
```
**EN:** This block defines `extract_all_otel_spans`. It accepts 1 parameter(s): otel_data. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `extract_all_otel_spans`。它接收 1 个参数：otel_data。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 172-195: Defines the `build_otel_span_tree` routine / 定义 `build_otel_span_tree` 例程
```python
def build_otel_span_tree(otel_spans):
    span_id_map = {span["spanId"]: span for span in otel_spans}
    for span in otel_spans:
        span["child"] = []

    root_spans = []

    for span in otel_spans:
        parent_span_id = span.get("parentSpanId", "")
        if span.get("attributes", {}).get("module") == "sglang::request":
            root_spans.append(span)
        elif parent_span_id in span_id_map:
            parent_span = span_id_map[parent_span_id]
            parent_span["child"].append(span)

        link_spans = []
        if "links" in span:
            for link in span["links"]:
                link_span = span_id_map.get(link["spanId"])
                if link_span:
                    link_spans.append(link_span)
            span["links"] = link_spans

    return root_spans
```
**EN:** This block defines `build_otel_span_tree`. It accepts 1 parameter(s): otel_spans. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `build_otel_span_tree`。它接收 1 个参数：otel_spans。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 198-226: Defines the `__convert_to_perfetto_span` routine / 定义 `__convert_to_perfetto_span` 例程
```python
def __convert_to_perfetto_span(span, rid, bootstrap_room, pid, host_id):
    if bootstrap_room:
        span["attributes"]["bootstrap_room"] = bootstrap_room
    if rid:
        span["attributes"]["rid"] = rid
    if host_id:
        span["host_id"] = host_id
    span["pid"] = pid

    span["startTimeUnixNano"] = int(span["startTimeUnixNano"])
    span["endTimeUnixNano"] = int(span["endTimeUnixNano"]) - 1000
    ts = span["startTimeUnixNano"]
    dur = span["endTimeUnixNano"] - ts

    perfetto_span = {
        "ph": "X",
        "name": span.get("name", "unknown"),
        "cat": "sglang",
        "ts": (ts - baseline) / 1000.0,
        "dur": dur / 1000.0,
        "pid": pid,
        "tid": 0,
        "args": span["attributes"],
    }

    span["perfetto_span"] = perfetto_span

    for child_span in span["child"]:
        __convert_to_perfetto_span(child_span, rid, bootstrap_room, pid, host_id)
```
**EN:** This block defines `__convert_to_perfetto_span`. It accepts 5 parameter(s): span, rid, bootstrap_room, pid, host_id. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `__convert_to_perfetto_span`。它接收 5 个参数：span, rid, bootstrap_room, pid, host_id。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 229-258: Defines the `generate_perfetto_span` routine / 定义 `generate_perfetto_span` 例程
```python
def generate_perfetto_span(engine_root_spans, smg_otel_spans, thread_meta_data):
    for root_span in engine_root_spans:
        root_span["spans"] = []

        rid = root_span["attributes"]["rid"]
        bootstrap_room = root_span["attributes"].get("bootstrap_room", "")

        for thread_span in root_span["child"]:
            pid = int(thread_span["attributes"]["pid"])
            host_id = thread_span["attributes"]["host_id"]
            thread_name = f'{thread_span["attributes"]["host_id"][:8]}:{thread_span["attributes"]["thread_label"]}'
            if "pp_rank" in thread_span["attributes"]:
                thread_name += f"-PP{thread_span['attributes']['pp_rank']}"
            if "dp_rank" in thread_span["attributes"]:
                thread_name += f"-DP{thread_span['attributes']['dp_rank']}"
            if "tp_rank" in thread_span["attributes"]:
                thread_name += f"-TP{thread_span['attributes']['tp_rank']}"

            if pid not in thread_meta_data:
                thread_meta_data[pid] = new_metadata_level1(thread_name, pid)

            for span in thread_span["child"]:
                __convert_to_perfetto_span(span, rid, bootstrap_room, pid, host_id)
                root_span["spans"].append(span)

    smg_pid = "smg"
    thread_meta_data[smg_pid] = new_metadata_level1("smg", smg_pid)
    for span in smg_otel_spans:
        span["pid"] = smg_pid
        __convert_to_perfetto_span(span, None, None, smg_pid, None)
```
**EN:** This block defines `generate_perfetto_span`. It accepts 3 parameter(s): engine_root_spans, smg_otel_spans, thread_meta_data. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `generate_perfetto_span`。它接收 3 个参数：engine_root_spans, smg_otel_spans, thread_meta_data。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 261-265: Defines the `__set_span_tid` routine / 定义 `__set_span_tid` 例程
```python
def __set_span_tid(span, line):
    span["perfetto_span"]["tid"] = line

    for child_span in span["child"]:
        __set_span_tid(child_span, line)
```
**EN:** This block defines `__set_span_tid`. It accepts 2 parameter(s): span, line. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `__set_span_tid`。它接收 2 个参数：span, line。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 268-308: Defines the `generate_perfetto_span_layout` routine / 定义 `generate_perfetto_span_layout` 例程
```python
def generate_perfetto_span_layout(engine_root_spans, smg_otel_spans, slot_meta_data):
    for root_span in engine_root_spans:
        root_span["spans"] = sorted(
            root_span["spans"], key=lambda x: int(x["startTimeUnixNano"])
        )

    engine_root_spans = sorted(
        engine_root_spans, key=lambda x: int(x["spans"][0]["startTimeUnixNano"])
    )
    graph = {}
    for root_span in engine_root_spans:
        req_thread_status = {}
        for span in root_span["spans"]:
            line = __find_line(
                graph,
                req_thread_status,
                slot_meta_data,
                span["perfetto_span"]["pid"],
                span["startTimeUnixNano"],
                span["endTimeUnixNano"],
            )
            graph[span["perfetto_span"]["pid"]][line].insert_span(
                span["startTimeUnixNano"], span["endTimeUnixNano"]
            )
            __set_span_tid(span, line)

    smg_otel_spans = sorted(smg_otel_spans, key=lambda x: int(x["startTimeUnixNano"]))
    req_thread_status = {}
    for span in smg_otel_spans:
        line = __find_line(
            graph,
            req_thread_status,
            slot_meta_data,
            span["perfetto_span"]["pid"],
            span["startTimeUnixNano"],
            span["endTimeUnixNano"],
        )
        graph[span["perfetto_span"]["pid"]][line].insert_span(
            span["startTimeUnixNano"], span["endTimeUnixNano"]
        )
        span["perfetto_span"]["tid"] = line
```
**EN:** This block defines `generate_perfetto_span_layout`. It accepts 3 parameter(s): engine_root_spans, smg_otel_spans, slot_meta_data. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `generate_perfetto_span_layout`。它接收 3 个参数：engine_root_spans, smg_otel_spans, slot_meta_data。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 311-332: Defines the `__convert_to_perfetto_events` routine / 定义 `__convert_to_perfetto_events` 例程
```python
def __convert_to_perfetto_events(span):
    span["perfetto_events"] = []
    if "events" in span:
        for event in span["events"]:
            attributes_dict = {
                attr.get("key"): next(iter(attr.get("value", {}).values()), None)
                for attr in event["attributes"]
            }
            perfetto_event = {
                "ph": "i",
                "cat": "sglang",
                "ts": (int(event["timeUnixNano"]) - baseline) / 1000.0,
                "pid": span["perfetto_span"]["pid"],
                "tid": span["perfetto_span"]["tid"],
                "name": event.get("name", "unknown"),
                "args": attributes_dict,
            }

            span["perfetto_events"].append(perfetto_event)

    for child_span in span["child"]:
        __convert_to_perfetto_events(child_span)
```
**EN:** This block defines `__convert_to_perfetto_events`. It accepts 1 parameter(s): span. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `__convert_to_perfetto_events`。它接收 1 个参数：span。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 335-342: Defines the `generate_perfetto_events` routine / 定义 `generate_perfetto_events` 例程
```python
def generate_perfetto_events(engine_root_spans, smg_otel_spans):
    spans = [span for root_span in engine_root_spans for span in root_span["spans"]]

    for span in spans:
        __convert_to_perfetto_events(span)

    for span in smg_otel_spans:
        __convert_to_perfetto_events(span)
```
**EN:** This block defines `generate_perfetto_events`. It accepts 2 parameter(s): engine_root_spans, smg_otel_spans. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `generate_perfetto_events`。它接收 2 个参数：engine_root_spans, smg_otel_spans。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 345-392: Defines the `generate_perfetto_links` routine / 定义 `generate_perfetto_links` 例程
```python
def generate_perfetto_links(engine_root_spans, smg_otel_spans):
    # build link between engine span and smg span
    span_id_map = {span["spanId"]: span for span in smg_otel_spans}

    for root_span in engine_root_spans:
        if "parentSpanId" in root_span and root_span["parentSpanId"] in span_id_map:
            parent_span = span_id_map[root_span["parentSpanId"]]
            root_span["spans"][0]["links"] = [parent_span]

        for span in root_span["spans"]:
            span["perfetto_links"] = []

            if "links" in span:
                for link_span in span["links"]:
                    try:
                        link_perfetto_span = link_span["perfetto_span"]
                    except (KeyError, AttributeError):
                        continue

                    if "correlation" in link_perfetto_span["args"]:
                        id = link_perfetto_span["args"]["correlation"]
                    else:
                        id = next(relation_id_gen)
                        link_perfetto_span["args"]["correlation"] = id

                    perfetto_start_node = {
                        "ph": "s",
                        "id": id,
                        "pid": link_perfetto_span["pid"],
                        "tid": link_perfetto_span["tid"],
                        "ts": link_perfetto_span["ts"],
                        "cat": "ac2g",
                        "name": "ac2g",
                    }

                    perfetto_end_node = {
                        "ph": "f",
                        "id": id,
                        "pid": span["perfetto_span"]["pid"],
                        "tid": span["perfetto_span"]["tid"],
                        "ts": span["perfetto_span"]["ts"],
                        "cat": "ac2g",
                        "name": "ac2g",
                        "bp": "e",
                    }

                    span["perfetto_links"].append(perfetto_start_node)
                    span["perfetto_links"].append(perfetto_end_node)
```
**EN:** This block defines `generate_perfetto_links`. It accepts 2 parameter(s): engine_root_spans, smg_otel_spans. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `generate_perfetto_links`。它接收 2 个参数：engine_root_spans, smg_otel_spans。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 395-406: Defines the `__gather_one_span` routine / 定义 `__gather_one_span` 例程
```python
def __gather_one_span(span):
    elems = []
    elems.append(span["perfetto_span"])
    if "perfetto_events" in span:
        elems.extend(span["perfetto_events"])
    if "perfetto_links" in span:
        elems.extend(span["perfetto_links"])

    for child_span in span["child"]:
        elems.extend(__gather_one_span(child_span))

    return elems
```
**EN:** This block defines `__gather_one_span`. It accepts 1 parameter(s): span. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `__gather_one_span`。它接收 1 个参数：span。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 409-423: Defines the `gather_all_perfetto_elems` routine / 定义 `gather_all_perfetto_elems` 例程
```python
def gather_all_perfetto_elems(
    engine_root_spans, smg_otel_spans, thread_meta_data, slot_meta_data
):
    elems = []
    elems.extend(thread_meta_data.values())
    elems.extend(slot_meta_data)
    for root_span in engine_root_spans:
        for span in root_span["spans"]:
            elems.extend(__gather_one_span(span))

    for span in smg_otel_spans:
        elems.append(span["perfetto_span"])
        elems.extend(span["perfetto_events"])

    return elems
```
**EN:** This block defines `gather_all_perfetto_elems`. It accepts 4 parameter(s): engine_root_spans, smg_otel_spans, thread_meta_data, slot_meta_data. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `gather_all_perfetto_elems`。它接收 4 个参数：engine_root_spans, smg_otel_spans, thread_meta_data, slot_meta_data。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 426-441: Defines the `write_json` routine / 定义 `write_json` 例程
```python
def write_json(perfetto_elems):
    global perfetto_data

    if args.torch_file:
        perfetto_data["traceEvents"].extend(perfetto_elems)
        filered_data = [
            item
            for item in perfetto_data["traceEvents"]
            if item.get("cat") != "gpu_user_annotation"
        ]
        perfetto_data["traceEvents"] = filered_data
    else:
        perfetto_data = perfetto_elems

    with open(args.output_file, "w", encoding="utf-8") as file:
        json.dump(perfetto_data, file, ensure_ascii=False, indent=4)
```
**EN:** This block defines `write_json`. It accepts 1 parameter(s): perfetto_elems. Internally it reads or writes files.
**CN:** 该代码块定义了 `write_json`。它接收 1 个参数：perfetto_elems。内部会读写文件。

### Lines 444-463: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    start_time = time.time()
    otel_data = load_otel_data(args.input_file)
    engine_otel_spans, smg_otel_spans = extract_all_otel_spans(otel_data)
    engine_root_spans = build_otel_span_tree(engine_otel_spans)
    thread_meta_data = {}
    generate_perfetto_span(engine_root_spans, smg_otel_spans, thread_meta_data)
    slot_meta_data = []
    generate_perfetto_span_layout(engine_root_spans, smg_otel_spans, slot_meta_data)
    generate_perfetto_events(engine_root_spans, smg_otel_spans)
    generate_perfetto_links(engine_root_spans, smg_otel_spans)
    perfetto_elems = gather_all_perfetto_elems(
        engine_root_spans, smg_otel_spans, thread_meta_data, slot_meta_data
    )
    write_json(perfetto_elems)
    end_time = time.time()
    execution_time = end_time - start_time
    print(f"\nConversion finished successfully!")
    print(f"Output written to: {args.output_file}")
    print(f"Execution time: {execution_time * 1000:.4f} ms")
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会输出状态信息。

### Lines 466-467: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理
- **Structured data handling** / 结构化数据处理

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `bisect`, `collections`, `json`, `pathlib`, `time`, `typing`
