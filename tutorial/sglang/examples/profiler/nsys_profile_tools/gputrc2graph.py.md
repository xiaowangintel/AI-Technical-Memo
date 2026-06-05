# gputrc2graph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/profiler/nsys_profile_tools/gputrc2graph.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This generates gpu kernel analysis output from nsys rep. Will call nsys stats -r cuda_gpu_kern_trace, get non-overlapped gpu cycles, then generate csv and html output for analysis / 该文件的顶部说明概述了此示例的目标与使用场景。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Module overview
````python
"""
This generates gpu kernel analysis output from nsys rep. Will call nsys
stats  -r cuda_gpu_kern_trace, get non-overlapped gpu cycles, then generate
csv and html output for analysis
"""
````
**EN:** The opening docstring summarizes the goal of the file and provides high-level context for the code that follows.
**CN:** 开头的文档字符串概括了文件目标，并为后续代码提供高层背景。

### Lines 7-12: Import dependencies and runtime symbols
````python
import argparse
import logging
import os
import shlex

import regex as re
````
**EN:** The module loads data-processing libraries used to transform profiling traces into reports.
**CN:** 模块在这里引入了数据处理库，用于把性能追踪结果转换成分析报告。

### Lines 14-14: Initialize logging state
````python
logger = logging.getLogger(__name__)
````
**EN:** The module creates reusable logger objects or other top-level helpers before entering the main workflow.
**CN:** 模块会在主流程开始前创建可复用的日志对象或其他顶层辅助对象。

### Lines 17-31: Load engine model
````python
# helper data class for annotating kernels
def load_engine_model():
    """returns engine_model built from all json files in the current dir"""
    import glob
    import json

    engine_model = {}

    json_files = glob.glob(os.path.join(os.path.dirname(__file__) or ".", "*.json"))
    for fname in json_files:
        with open(fname, encoding="utf-8") as f:
            file_engine_model = json.load(f)
        for engine, models in file_engine_model.items():
            engine_model.setdefault(engine, {}).update(models)
    return engine_model
````
**EN:** This function encapsulates the “Load engine model” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Load engine model”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 34-34: Define GPUTrace2Graph analysis class
````python
class GPUTrace2Graph:
````
**EN:** The class packages related profiling utilities into a reusable object instead of scattering them across free functions.
**CN:** 该类把相关的性能分析工具封装到一个可复用对象中，而不是分散在多个自由函数里。

### Lines 35-37: GPUTrace2Graph documentation
````python
    """
    Parses output of nsys report, generates csv and bar chart output
    """
````
**EN:** The docstring explains the responsibility of this class or class-local component before the implementation details begin.
**CN:** 该文档字符串在实现细节开始前先说明了这个类或类内组件的职责。

### Lines 39-42: Method: Initialize class state
````python
    def __init__(self):
        import pandas as pd  # avoid importing till needed

        self.pd = pd
````
**EN:** This method initializes lightweight dependencies and stores the object state needed by later methods.
**CN:** 该方法会初始化轻量级依赖，并保存后续方法需要使用的对象状态。

### Lines 44-64: Method: Transform trace data
````python
    # helper functions for generating trace->summary csvs
    def gen_nonoverlapped_sum_from_gputrace(self, in_file, out_file):
        logger.info("loading %s", in_file)
        df = self.pd.read_csv(
            in_file, usecols=["Start (ns)", "Duration (ns)", "Device", "Strm", "Name"]
        )
        df["End (ns)"] = df["Start (ns)"] + df["Duration (ns)"]
        df = self.sum_non_overlapping_intervals(df)
        # get ready to print table with elapsed times per kernel
        df["Instances"] = 1
        df_sum = df.groupby("Name", as_index=False).agg(
            {"Elapsed Time (ns)": "sum", "Duration (ns)": "sum", "Instances": "size"}
        )

        # generate csv
        df_sum["Total Time (sec)"] = df_sum["Duration (ns)"] / 1e9
        df_sum["Elapsed Time (sec)"] = df_sum["Elapsed Time (ns)"] / 1e9
        df_sum = df_sum.sort_values(by="Elapsed Time (sec)", ascending=False)
        df_sum[["Elapsed Time (sec)", "Total Time (sec)", "Instances", "Name"]].to_csv(
            out_file, index=False
        )
````
**EN:** This method transforms tabular profiling data, aggregates timing information, and emits derived analysis artifacts such as CSV or HTML files.
**CN:** 该方法会转换表格化的性能数据、聚合时间信息，并输出 CSV 或 HTML 等分析产物。

### Lines 66-104: Method: Handle non overlapping intervals
````python
    def sum_non_overlapping_intervals(self, df):
        """
        returns new sorted df with Elapsed Time (ns) column using
        vectorized operations
        """
        logger.info("sorting %s trace records by start time", str(df.shape))

        # Sort by start time and reset index
        df = df.sort_values(by="Start (ns)").reset_index(drop=True)

        # Initialize elapsed time as duration
        df["Elapsed Time (ns)"] = df["Duration (ns)"]

        # Get numpy arrays for faster operations
        starts = df["Start (ns)"].values
        ends = df["End (ns)"].values

        # Keep track of current interval end
        current_end = ends[0]
        display_units = max(1, int(len(df) / 100))
        # Update current_end for overlapping intervals
        for i in range(1, len(df)):
            if i % display_units == 0:
                print(f"processing trace: {int(i/len(df) * 100)} %", end="\r")
            if starts[i] <= current_end:
                if ends[i] > current_end:
                    # Partial overlap
                    df.iloc[i, df.columns.get_loc("Elapsed Time (ns)")] = (
                        ends[i] - current_end
                    )
                    current_end = ends[i]
                else:
                    # Complete overlap
                    df.iloc[i, df.columns.get_loc("Elapsed Time (ns)")] = 0
            else:
                # No overlap
                current_end = ends[i]

        return df
````
**EN:** This method encapsulates the “Handle non overlapping intervals” step so the overall example remains modular and easier to follow.
**CN:** 该方法把“Handle non overlapping intervals”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 106-163: Method: Transform trace data
````python
    # functions for generating html files
    def make_html(self, df, output_dir, title):
        """make html graph from df"""
        import plotly.express as px

        if df.empty:
            return
        output_name = os.path.join(output_dir, "result")
        if not title:
            title = "Model_Engine"
        x = "Model_Engine"
        y = "Elapsed Time (sec)"
        color = "Category"
        """ generate kernel mapping table  """
        # Sort Model_Engine categories by last field after underscore
        df["Model_Engine"] = self.pd.Categorical(
            df["Model_Engine"],
            sorted(df["Model_Engine"].unique(), key=lambda x: x.split("_")[-1]),
        )
        df[["Model_Engine", color, "Instances", "Name", y]].sort_values(
            by=color
        ).to_csv(f"{output_name}.csv", index=False)
        graph = px.histogram(
            df.round(2),
            x=x,
            y=y,
            title=(f"{y} for {title}"),
            color=color,
            text_auto=True,
        )
        # wrap x axis labels
        graph.update_xaxes(automargin=True)
        graph.write_html(f"{output_name}.html")
        """
            Generate data table with columns per Model_Engine into result.html
        """
        pivot_df = df.pivot_table(
            values="Elapsed Time (sec)",
            index="Category",
            columns="Model_Engine",
            aggfunc="sum",
            observed=False,
        ).round(2)
        # Add sum row at bottom
        pivot_df.loc["total_elapsed_sec"] = pivot_df.sum()
        pivot_df.fillna("").to_html("temp.html")
        with (
            open(f"{output_name}.html", "a", encoding="utf-8") as outfile,
            open("temp.html", encoding="utf-8") as infile,
        ):
            outfile.write(infile.read())
        os.remove("temp.html")

        print(
            f"Finished generating: \n"
            f" {output_name}.html for stack bar chart \n"
            f" {output_name}.csv for Kernel-Category mapping"
        )
````
**EN:** This method transforms tabular profiling data, aggregates timing information, and emits derived analysis artifacts such as CSV or HTML files.
**CN:** 该方法会转换表格化的性能数据、聚合时间信息，并输出 CSV 或 HTML 等分析产物。

### Lines 165-173: Method: Annotate GPU kernname
````python
    def anno_gpu_kernname(self, df, mapping):
        """add "Category" column"""

        def anno_gpu_kernname_helper(name):
            for kern_name, val in mapping.items():
                if re.search(kern_name, name):
                    return val

        df["Category"] = df["Name"].apply(anno_gpu_kernname_helper)
````
**EN:** This method encapsulates the “Annotate GPU kernname” step so the overall example remains modular and easier to follow.
**CN:** 该方法把“Annotate GPU kernname”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 175-181: Method: Build non-GPU row
````python
    def make_nongpu_row(self, df, nongpu_sec):
        """this will append non-gpu time entry at end of df"""
        nongpu_row = self.pd.DataFrame([df.iloc[-1]])
        nongpu_row["Category"] = nongpu_row["Name"] = "CPU(non-GPU)"
        nongpu_row["Instances"] = 1
        nongpu_row["Elapsed Time (sec)"] = nongpu_sec
        return nongpu_row
````
**EN:** This method encapsulates the “Build non-GPU row” step so the overall example remains modular and easier to follow.
**CN:** 该方法把“Build non-GPU row”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 183-187: Method: Validate valid file
````python
    def is_valid_file(self, base_file):
        """asserts if base_file is non-existent or is empty"""
        assert (
            os.path.isfile(base_file) and os.path.getsize(base_file) > 0
        ), f"{base_file} doesn't exist or is empty"
````
**EN:** This method encapsulates the “Validate valid file” step so the overall example remains modular and easier to follow.
**CN:** 该方法把“Validate valid file”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 189-201: Method: Decide gen file
````python
    def should_gen_file(self, new_file, base_file):
        """figure out if new file should be generated from base_file"""
        self.is_valid_file(base_file)
        if (
            os.path.exists(new_file)
            and (os.path.getmtime(new_file) > os.path.getmtime(base_file))
            and (os.path.getsize(base_file) > 0)
        ):
            logger.info("reusing %s", new_file)
            return False
        else:
            logger.info("generating %s", new_file)
            return True
````
**EN:** This method encapsulates the “Decide gen file” step so the overall example remains modular and easier to follow.
**CN:** 该方法把“Decide gen file”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 203-248: Method: Generate sum file
````python
    def gen_sum_file(self, file, nsys_cmd):
        """
        generates sum file from nsys trace with times per kernel and
        returns the name of the sum file
        """
        import subprocess

        file_dir = os.path.dirname(file)
        file_name = os.path.basename(file)

        if not file_dir:
            file_dir = "."
        # Walk through trace and get the total non-overlapped time
        nsys_stats_file = os.path.join(file_dir, f"{file_name}_cuda_gpu_trace.csv")
        sum_file = os.path.join(file_dir, f"{file_name}_cuda_gpu_kernel_tracesum.csv")
        if self.should_gen_file(nsys_stats_file, file):
            cmd = [
                nsys_cmd,
                "stats",
                "-r",
                "cuda_gpu_trace",
                file,
                "-o",
                f"{file_dir}/{file_name}",
            ]
            cmd_str = shlex.join(cmd)
            logger.info("+ %s", cmd_str)
            # estimate time based on calibrated 240M/min
            file_size_mb = os.path.getsize(file) / 1e6
            logger.info(
                "nsys stats for %.2f MB file expected to take %.2f min",
                file_size_mb,
                file_size_mb / 240,
            )
            try:
                subprocess.run(cmd, check=True)
            except (FileNotFoundError, subprocess.CalledProcessError) as e:
                logger.error(
                    "'%s' failed: %s. Use --nsys_cmd to specify nsys path", cmd_str, e
                )
                exit(1)
            logger.info("generating non-overalapped sum %s", sum_file)
            self.gen_nonoverlapped_sum_from_gputrace(nsys_stats_file, sum_file)
        self.is_valid_file(sum_file)
        logger.info("Finished generating %s", sum_file)
        return sum_file
````
**EN:** This method encapsulates the “Generate sum file” step so the overall example remains modular and easier to follow.
**CN:** 该方法把“Generate sum file”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 250-287: Method: Transform trace data
````python
    def gen_graph(self, in_file, out_dir, title, nsys_cmd, engine_model):
        """generates graph and csv file from in_file into out_dir"""
        # Initialize an empty DataFrame to store combined data
        combined_df = self.pd.DataFrame()
        for idx, (file, engine, model, total_sec) in enumerate(in_file):
            file_dir = os.path.dirname(file)
            file_name = os.path.basename(file)
            if not file_dir:
                file_dir = "."
            sum_file = self.gen_sum_file(file, nsys_cmd)
            # read kernel summary file
            df = self.pd.read_csv(sum_file)
            # annotate kernel to their categories
            assert engine_model.get(engine), f"engine {engine} unknown"
            assert engine_model[engine].get(model), f"model {model} unknown"
            # remove nsys-rep from file_name for shorter x-label
            file_name = file_name.replace(".nsys-rep", "")
            df["Model_Engine"] = f"{model}_{engine}_{file_name}_{idx}"
            self.anno_gpu_kernname(df, engine_model[engine][model])
            # patch in non-gpu time
            gpu_sec = round(df["Elapsed Time (sec)"].sum(), 1)
            total_sec = round(float(total_sec), 1)
            if total_sec < gpu_sec:
                logger.warning(
                    "Elapsed sec %.2f < GPU sec %.2f resetting Elapsed sec ",
                    total_sec,
                    gpu_sec,
                )
                total_sec = gpu_sec
            nongpu_row = self.make_nongpu_row(df, total_sec - gpu_sec)
            df = self.pd.concat([df, nongpu_row], ignore_index=True)
            combined_df = self.pd.concat([combined_df, df], ignore_index=True)
        if out_dir is None:
            out_dir = "."
        else:
            os.makedirs(out_dir, exist_ok=True)
        # generate html file
        self.make_html(combined_df, out_dir, title)
````
**EN:** This method transforms tabular profiling data, aggregates timing information, and emits derived analysis artifacts such as CSV or HTML files.
**CN:** 该方法会转换表格化的性能数据、聚合时间信息，并输出 CSV 或 HTML 等分析产物。

### Lines 290-291: Parse tuple
````python
def parse_tuple(s):
    return tuple(s.split(","))
````
**EN:** This function encapsulates the “Parse tuple” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Parse tuple”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 294-342: Parse arguments and run the workflow
````python
def main():
    logging.basicConfig(
        format=("%(asctime)s - %(levelname)s - %(message)s"), level=logging.INFO
    )
    parser = argparse.ArgumentParser(
        description=(
            "Process nsys rep and generate kernel non-overlapped cycles. \n"
            "Example:\n"
            "gputrc2graph.py --in_file d1.nsys-rep,sglang,llama,100 \n"
            "d2.nsys-rep,sglang,gpt-oss,102 "
            '--out_dir results/ --title "Model=gpt-oss SGLANG chart"'
        ),
        formatter_class=argparse.RawDescriptionHelpFormatter,
    )

    # load supported engine_model
    engine_model_supported = load_engine_model()
    # Get a string representation of supported engine/model combinations
    engine_model_supported_str = ", ".join(
        f"{engine}:[{', '.join(models.keys())}]"
        for engine, models in engine_model_supported.items()
    )
    parser.add_argument(
        "--in_file",
        type=parse_tuple,
        nargs="+",
        help=(
            "list of (nsys-rep, engine, model, elapsed_nonprofiled_sec) "
            "separated by space. Elapsed_nonprofiled_sec is runtime without "
            "profiling used to calculate non-gpu time. Specify 0 to use "
            "elapsed time from nsys-rep but that might inflate non-gpu time. "
            f"Available engine:[model] are: {engine_model_supported_str} "
            f"Example: --infile d1.nsys-rep,sglan,llama,100 "
            "d2.nsys-rep,sglang,gpt-oss,102"
        ),
        required=True,
    )
    parser.add_argument("--out_dir", help=("output dir for result.csv/html"))
    parser.add_argument("--title", help=("title for html chart"))
    parser.add_argument(
        "--nsys_cmd",
        help=("nsys cmd, e.g. /usr/bin/nsys, Default: nsys"),
        default="nsys",
    )
    args = parser.parse_args()
    gputrace = GPUTrace2Graph()
    gputrace.gen_graph(
        args.in_file, args.out_dir, args.title, args.nsys_cmd, engine_model_supported
    )
````
**EN:** This function serves as the entry workflow: it parses user options, prepares runtime objects, and dispatches the requested operation.
**CN:** 该函数充当入口工作流：它会解析用户选项、准备运行时对象，并分发到相应的操作步骤。

### Lines 345-346: Program entry point
````python
if __name__ == "__main__":
    main()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Profiler post-processing / 性能分析后处理**: Trace data is transformed into summaries and visual reports. / 追踪数据会被转换成摘要和可视化报告。

## Dependencies / 依赖关系
- **Standard library / 标准库**: argparse, glob, json, logging, os, shlex, subprocess
- **Third-party / 第三方**: pandas, plotly.express, regex
