# update_expected.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/ci_expected_accuracy/update_expected.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: """
 2: Update committed CSV files used as reference points by dynamo/inductor CI.
 3: 
 4: Currently only cares about graph breaks, so only saves those columns.
 5: 
 6: Hardcodes a list of job names and artifacts per job, but builds the lookup
 7: by querying github sha and finding associated github actions workflow ID and CI jobs,
 8: downloading artifact zips, extracting CSVs and filtering them.
 9: 
10: Usage:
11: 
12: python benchmarks/dynamo/ci_expected_accuracy.py <sha of pytorch commit that has completed inductor benchmark jobs>
13: 
14: Known limitations:
15: - doesn't handle 'retry' jobs in CI, if the same job has multiple run attempts, it may pick the wrong one
16: """
17: 
18: import argparse
19: import json
20: import os
21: import subprocess
22: import sys
23: import urllib
24: from concurrent.futures import as_completed, ThreadPoolExecutor
25: from io import BytesIO
26: from itertools import product
27: from pathlib import Path
28: from urllib.request import urlopen
29: from zipfile import ZipFile
30: 
31: import pandas as pd
32: import requests
33: 
34: 
35: """
36: WITH job as (
37:     SELECT
38:         job.created_at as time,
39:         job.name as job_name,
40:         workflow.name as workflow_name,
41:         job.id as id,
42:         job.run_attempt as run_attempt,
43:         workflow.id as workflow_id
44:     FROM
45:         default.workflow_job job final
46:         INNER JOIN default.workflow_run workflow final on workflow.id = job.run_id
47:     WHERE
48:         job.name != 'ciflow_should_run'
49:         AND job.name != 'generate-test-matrix'
50:         -- Filter out workflow_run-triggered jobs, which have nothing to do with the SHA
51:         AND workflow.event != 'workflow_run'
52:         -- Filter out repository_dispatch-triggered jobs, which have nothing to do with the SHA
53:         AND workflow.event != 'repository_dispatch'
54:         AND workflow.head_sha = {sha: String}
55:         AND job.head_sha = {sha: String}
56:         AND workflow.repository.'full_name' = {repo: String}
57: )
58: SELECT
59:     workflow_name as workflowName,
60:     job_name as jobName,
````
- EN: Handles module imports such as `argparse`, `json`, `os`, `subprocess`.
- CN: 处理模块导入，例如 `argparse`, `json`, `os`, `subprocess`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61:     CAST(id as String) as id,
 62:     run_attempt as runAttempt,
 63:     CAST(workflow_id as String) as workflowId,
 64:     time
 65: from
 66:     job
 67: ORDER BY
 68:     workflowName, jobName
 69: """
 70: ARTIFACTS_QUERY_URL = (
 71:     "https://console-api.clickhouse.cloud/.api/query-endpoints/"  # @lint-ignore
 72:     "c1cdfadc-6bb2-4a91-bbf9-3d19e1981cd4/run?format=JSON"
 73: )
 74: CSV_LINTER = str(
 75:     Path(__file__).absolute().parents[3]
 76:     / "tools/linter/adapters/no_merge_conflict_csv_linter.py"
 77: )
 78: 
 79: 
 80: def query_job_sha(repo, sha):
 81:     params = {
 82:         "queryVariables": {"sha": sha, "repo": repo},
 83:     }
 84:     # If you are a Meta employee, go to P1679979893 to get the id and secret.
 85:     # Otherwise, ask a Meta employee give you the id and secret.
 86:     try:
 87:         KEY_ID = os.environ["CH_KEY_ID"]
 88:         KEY_SECRET = os.environ["CH_KEY_SECRET"]
 89:     except KeyError as e:
 90:         raise RuntimeError(
 91:             "CH_KEY_ID and CH_KEY_SECRET environment variables must be set. "
 92:             "If you are a Meta employee, go to P1679979893 to get the id and secret. "
 93:             "Otherwise, ask a Meta employee to give you the id and secret."
 94:         ) from e
 95: 
 96:     r = requests.post(
 97:         url=ARTIFACTS_QUERY_URL,
 98:         data=json.dumps(params),
 99:         headers={"Content-Type": "application/json"},
100:         auth=(KEY_ID, KEY_SECRET),
101:     )
102:     return r.json()["data"]
103: 
104: 
105: def parse_job_name(job_str):
106:     return (part.strip() for part in job_str.split("/"))
107: 
108: 
109: def parse_test_str(test_str):
110:     return (part.strip() for part in test_str[6:].strip(")").split(","))
111: 
112: 
113: S3_BASE_URL = "https://gha-artifacts.s3.amazonaws.com"
114: 
115: 
116: def get_artifacts_urls(results, suites, is_rocm=False):
117:     urls = {}
118:     # Sort by time (oldest first) to prefer earlier completed workflow runs
119:     # over potentially still-running newer ones
120:     sorted_results = sorted(results, key=lambda x: x.get("time", ""))
````
- EN: Implements callable logic such as `query_job_sha`, `parse_job_name`, `parse_test_str`, `get_artifacts_urls`.
- CN: 实现可调用逻辑，例如 `query_job_sha`, `parse_job_name`, `parse_test_str`, `get_artifacts_urls`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:     for r in sorted_results:
122:         if (
123:             r["workflowName"] in ("inductor", "inductor-periodic")
124:             and "test" in r["jobName"]
125:             and "build" not in r["jobName"]
126:             and "runner-determinator" not in r["jobName"]
127:             and "unit-test" not in r["jobName"]
128:         ):
129:             # Filter out CUDA-13 jobs so it won't override CUDA-12 results.
130:             # The result files should be shared between CUDA-12 and CUDA-13, but
131:             # CUDA-13 skips more tests at the moment.
132:             if "cuda13" in r["jobName"]:
133:                 continue
134: 
135:             # Filter based on whether this is a ROCm or CUDA job
136:             job_is_rocm = "rocm" in r["jobName"].lower()
137:             if job_is_rocm != is_rocm:
138:                 continue
139: 
140:             *_, test_str = parse_job_name(r["jobName"])
141:             suite, shard_id, num_shards, machine, *_ = parse_test_str(test_str)
142:             workflowId = r["workflowId"]
143:             id = r["id"]
144:             runAttempt = r["runAttempt"]
145: 
146:             if suite in suites:
147:                 artifact_filename = f"test-reports-test-{suite}-{shard_id}-{num_shards}-{machine}_{id}.zip"
148:                 s3_url = f"{S3_BASE_URL}/{repo}/{workflowId}/{runAttempt}/artifact/{artifact_filename}"
149:                 # Collect all candidate URLs per (suite, shard), ordered oldest first
150:                 key = (suite, int(shard_id))
151:                 if key not in urls:
152:                     urls[key] = []
153:                 if s3_url not in urls[key]:
154:                     urls[key].append(s3_url)
155:     return urls
156: 
157: 
158: def normalize_suite_filename(suite_name):
159:     strs = suite_name.split("_")
160:     subsuite = strs[-1]
161:     if "timm" in subsuite:
162:         subsuite = subsuite.replace("timm", "timm_models")
163: 
164:     return subsuite
165: 
166: 
167: def download_single_artifact(suite, shard, url_candidates):
168:     """Download a single artifact, trying each URL candidate until one succeeds.
169: 
170:     Returns a tuple of (suite, shard, result_dict) where result_dict maps
171:     (suite, phase) -> DataFrame, or None if download failed.
172:     """
173:     subsuite = normalize_suite_filename(suite)
174:     for url in url_candidates:
175:         try:
176:             resp = urlopen(url)
177:             artifact = ZipFile(BytesIO(resp.read()))
178:             result = {}
179:             for phase in ("training", "inference"):
180:                 # Try both paths - CUDA uses test/test-reports/, ROCm uses test-reports/
````
- EN: Implements callable logic such as `normalize_suite_filename`, `download_single_artifact`.
- CN: 实现可调用逻辑，例如 `normalize_suite_filename`, `download_single_artifact`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:                 possible_names = [
182:                     f"test/test-reports/{phase}_{subsuite}.csv",
183:                     f"test-reports/{phase}_{subsuite}.csv",
184:                 ]
185:                 found = False
186:                 for name in possible_names:
187:                     try:
188:                         df = pd.read_csv(artifact.open(name))
189:                         df["graph_breaks"] = df["graph_breaks"].fillna(0).astype(int)
190:                         result[(suite, phase)] = df
191:                         found = True
192:                         break
193:                     except KeyError:
194:                         continue
195:                 if not found and phase == "inference":
196:                     # No warning for training, since it's expected to be missing for some tests
197:                     print(
198:                         f"Warning: Unable to find {phase}_{subsuite}.csv in artifacts file from {url}, continuing"
199:                     )
200:             return (suite, shard, result)
201:         except urllib.error.HTTPError:
202:             continue  # Try next candidate URL
203:     return (suite, shard, None)
204: 
205: 
206: def download_artifacts_and_extract_csvs(urls):
207:     dataframes = {}
208:     with ThreadPoolExecutor(max_workers=16) as executor:
209:         futures = {
210:             executor.submit(download_single_artifact, suite, shard, url_candidates): (
211:                 suite,
212:                 shard,
213:                 url_candidates,
214:             )
215:             for (suite, shard), url_candidates in urls.items()
216:         }
217:         for future in as_completed(futures):
218:             suite, shard, url_candidates = futures[future]
219:             suite_result, shard_result, result = future.result()
220:             if result is None:
221:                 print(
222:                     f"Unable to download any artifact for {suite} shard {shard}, tried {len(url_candidates)} URLs"
223:                 )
224:             else:
225:                 for (s, phase), df in result.items():
226:                     prev_df = dataframes.get((s, phase), None)
227:                     dataframes[(s, phase)] = (
228:                         pd.concat([prev_df, df]) if prev_df is not None else df
229:                     )
230: 
231:     return dataframes
232: 
233: 
234: def write_filtered_csvs(root_path, dataframes):
235:     for (suite, phase), df in dataframes.items():
236:         out_fn = os.path.join(root_path, f"{suite}_{phase}.csv")
237:         # Read existing CSV and merge with new data to preserve entries
238:         # from shards that failed to download
239:         if os.path.exists(out_fn):
240:             existing_df = pd.read_csv(out_fn)
````
- EN: Implements callable logic such as `download_artifacts_and_extract_csvs`, `write_filtered_csvs`.
- CN: 实现可调用逻辑，例如 `download_artifacts_and_extract_csvs`, `write_filtered_csvs`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:             # Use new data where available, keep old data for missing entries
242:             # Set 'name' as index for both, update existing with new, then reset
243:             existing_df = existing_df.set_index("name")
244:             df = df.set_index("name")
245:             existing_df.update(df)
246:             # Add any new entries from df that weren't in existing
247:             df = existing_df.combine_first(df).reset_index()
248:         df = df.sort_values(by="name")
249:         df.to_csv(out_fn, index=False, columns=["name", "accuracy", "graph_breaks"])
250:         apply_lints(out_fn)
251: 
252: 
253: def apply_lints(filename):
254:     patch = json.loads(subprocess.check_output([sys.executable, CSV_LINTER, filename]))
255:     if patch.get("replacement"):
256:         with open(filename) as fd:
257:             data = fd.read().replace(patch["original"], patch["replacement"])
258:         with open(filename, "w") as fd:
259:             fd.write(data)
260: 
261: 
262: if __name__ == "__main__":
263:     parser = argparse.ArgumentParser(
264:         description=__doc__, formatter_class=argparse.RawDescriptionHelpFormatter
265:     )
266: 
267:     parser.add_argument("sha")
268:     args = parser.parse_args()
269: 
270:     repo = "pytorch/pytorch"
271: 
272:     suites = {
273:         f"{a}_{b}"
274:         for a, b in product(
275:             [
276:                 "aot_eager",
277:                 "aot_inductor",
278:                 "cpu_aot_inductor",
279:                 "cpu_aot_inductor_amp_freezing",
280:                 "cpu_aot_inductor_freezing",
281:                 "cpu_inductor",
282:                 "cpu_inductor_amp_freezing",
283:                 "cpu_inductor_freezing",
284:                 "dynamic_aot_eager",
285:                 "dynamic_cpu_aot_inductor",
286:                 "dynamic_cpu_aot_inductor_amp_freezing",
287:                 "dynamic_cpu_aot_inductor_freezing",
288:                 "dynamic_cpu_inductor",
289:                 "dynamic_inductor",
290:                 "dynamo_eager",
291:                 "inductor",
292:             ],
293:             ["huggingface", "timm", "torchbench"],
294:         )
295:     }
296: 
297:     root_path = "benchmarks/dynamo/ci_expected_accuracy/"
298:     if not os.path.exists(root_path):
299:         raise AssertionError(f"cd <pytorch root> and ensure {root_path} exists")
300:     rocm_path = "benchmarks/dynamo/ci_expected_accuracy/rocm/"
````
- EN: Implements callable logic such as `apply_lints`.
- CN: 实现可调用逻辑，例如 `apply_lints`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 301-324
````python
301:     if not os.path.exists(rocm_path):
302:         raise AssertionError(f"cd <pytorch root> and ensure {rocm_path} exists")
303: 
304:     results = query_job_sha(repo, args.sha)
305: 
306:     # Get URLs for both CUDA and ROCm
307:     cuda_urls = get_artifacts_urls(results, suites, is_rocm=False)
308:     rocm_urls = get_artifacts_urls(results, suites, is_rocm=True)
309: 
310:     # Download CUDA and ROCm artifacts in parallel
311:     print("Downloading CUDA and ROCm artifacts in parallel...")
312:     with ThreadPoolExecutor(max_workers=2) as executor:
313:         cuda_future = executor.submit(download_artifacts_and_extract_csvs, cuda_urls)
314:         rocm_future = executor.submit(download_artifacts_and_extract_csvs, rocm_urls)
315:         cuda_dataframes = cuda_future.result()
316:         rocm_dataframes = rocm_future.result()
317: 
318:     print("Writing CUDA CSVs...")
319:     write_filtered_csvs(root_path, cuda_dataframes)
320: 
321:     print("Writing ROCm CSVs...")
322:     write_filtered_csvs(rocm_path, rocm_dataframes)
323: 
324:     print("Success. Now, confirm the changes to .csvs and `git add` them if satisfied.")
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `query_job_sha` / 符号 `query_job_sha`
- Symbol `parse_job_name` / 符号 `parse_job_name`
- Symbol `parse_test_str` / 符号 `parse_test_str`
- Symbol `get_artifacts_urls` / 符号 `get_artifacts_urls`

## Dependencies / 依赖关系
- Python imports: `argparse`, `json`, `os`, `subprocess`, `sys`, `urllib`, `concurrent.futures`, `io`, `itertools`, `pathlib`
- Python 导入: `argparse`, `json`, `os`, `subprocess`, `sys`, `urllib`, `concurrent.futures`, `io`, `itertools`, `pathlib`
