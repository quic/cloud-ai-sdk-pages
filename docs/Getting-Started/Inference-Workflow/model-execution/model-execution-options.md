# Model Execution

There are 4 ways to call inference on Qualcomm Cloud AI 100 device.

## 1. CLI API

`qaic-runner` is a CLI (command line inferface) tool designed to run inference on the device using precompiled binary (also called QPC (Qaic Program Container) binary).
It provides various options and functionalities to facilitate inference and performance/benchmarking analysis. In this document, we will discuss the different options available in `qaic-runner` tool, their default values and provide examples for their usage.

### Options and Default values

| <div style="width:250px">Parameter</div>  | Description                                                                                             | Default  |
| ------------------------------ | ------------------------------------------------------------------------------------------------------- | -------- |
| `-d, --aic-device-id <id>	`    | Specify AIC device ID.	                                                                               | 0        |  
| `-D, --dev-list <qid>[:<qid>]	`| Map of device IDs for a multi-device network.	                                                       | 0[:1]    |  
| `-d, --aic-device-id <id>            ` |  AIC device ID, default Auto-pick   | 0 |
| `-D, --aic-device-map <qid>[:<qid>]  ` |  Map of Device IDs for multi-device network,     |  0[:1] |
| `-t, --test-data <path>              ` |  Location of program binaries   | |
| `-i, --input-file <path>             ` |  Input filename from which to load input data. Specify multiple times for each input file.     If no -i is given, look for available inputs in bindings.json at the -t directory. If bindings.json is not available, random input will be generated.   |  |
| `-n, --num-iter <num>                ` |  Number of iterations,    | 40 |
| `--time <t>                          ` |  Duration (in seconds) for which to submit inferences   |  |
| `-l, --live-reporting                ` |  Enable Live reporting periodic at 1 sec interval    | off |
| `-r, --live-reporting-period         ` |  Set Live Reporting Period in Ms    | 1000 |
| `-s  --stats                         ` |  Enable Live Profiling Stats reporting periodically at 1 sec interval   |  |
| `-a, --aic-num-of-activations <num>  ` |  Number of activations   | 1 |
| `--aic-profiling-start-iter <num>    ` |  Profiling Start Iteration   | 0 |
| `--aic-profiling-start-delay <num>   ` |  Profiling Start delay (in milliseconds). Profiling will start after given delay period has elapsed   |  |
| `--aic-profiling-num-samples <num>   ` |  Profiling Num Samples to save to file   | 1 |
| `--aic-profiling-format <level>      ` |  Deprecated | DEF |
| `--aic-profiling-type <type>         ` |  Profiling Type, `'stats'\|'trace'\|'latency'` for legacy profiling and `'trace_stream' \| 'latency_stream'` for stream profiling. Set multiple times for multiple formats   | none |
| `--aic-profiling-duration <num>      ` |  Profiling duration to run profiling for (in ms). After starting profiling, it will stop at the expiry of profiling duration   |  |
| `--aic-profiling-sampling-rate <num> ` |  Profiling sampling rate [`full/half/fourth/eighth/sixteenth`]. Programs will generate profiling samples at the requested rate.  To profile all samples select full, for every second sample select half and so on   | full |
| `--aic-profiling-reporting-rate <num>` |  Profiling report generation rate (in ms) [`500/1000/2000/4000`]. Profiling report will be generated at every requested interval for profiling duration   | 500 |
| `--aic-profiling-out-dir <path>      ` |  Location to save files, dir should exist and be writable   | '.' |
| `--write-output-start-iter <num>     ` |  Write outputs start iteration   | 0 |
| `--write-output-num-samples <num>    ` |  Number of outputs to write   | 1 |
| `--write-output-dir <path>           ` |  Location to save output files, dir should exist and be writable   | '.' |
| `--aic-lib-path DEPRECATED           ` |  Deprecated, please set env variable QAIC_LIB to the full path of the custom library, by default loads libQAic.so from install location   |  |
| `--aic-batch-input-directory         ` |  Batch mode: process all files from input directory. Only the networks with single input file are currently supported   | DEF |
| `--aic-batch-input-file-list         ` |  Batch mode: Specify an input file containing comma-separated absolute path for buffers. Each line is 1 inference and must have number and size of buffers required by program   | DEF |
| `--aic-batch-max-memory <mb>         ` |  Batch mode: Limit memory usage when loading files, provide parameter in Mb   | 1024 |
| `--submit-timeout <num>              ` |  Time to wait for an inference request completion on kernel. default 0 ms. When 0, kernel defaults to 5000ms   |  |
| `--submit-retry-count <num>          ` |  Number of wait-call retries when an inference request times out.    | 5 |
| `--unbound-random                    ` |  When populating random values in buffer, do not consider input buffer format and fill each byte with random input between 0 to 255. This can result in unexpected behavior from certain network.    |  |
| `--dump-input-buffers                ` |  Dump input buffers used in benchmarking mode   |  |
| `-S, --set-size <num>                ` |  Set Size for inference loop execution, min:1   | 10 |
| `-T, --aic-threads-per-queue         ` |  Number of threads per queue   | 4 |
| `--auto-batch-input                  ` |  Automatically batch inputs to meet batchsize requirements of network. Inputs should be for Batch size 1   | 1 |
| `-p, --pre-post-processing           ` |  Pre-post processing [`on\|off`]   | on |
| `-v, --verbose                       ` |  Verbose log from program   |  |
| `-h, --help                          ` |  help | |

### Usage Examples   

#### 1. Running inference with Random inputs

Example:
```shell
sudo /opt/qti-aic/exec/qaic-runner -t /path/to/qpc  -a 3 -n 5000 -d 0 -v
```

- In this example we are using a precompiled binary that is already generated. 
    - 3 activations - activations here refers to number of instances of network you want to run on the device. In this case 3 copies of the network can run parallely on the device. 
    - Lets assume each network was compiled with 4 cores (`sudo /opt/qti-aic/tools/qaic-qpc validate -i /path/to/qpc/programqpc.bin` look for **Number of NSP required** value in output of this command.)
        - Make sure the device you are using has at least 12 i.e. 3x4 cores Free.
    - Since input is not provided we are feeding a randomly generated input (with appropriate dimensions and type inferred from qpc) to the device. 
    - This single randomly generated input is used for 5000 inferences.
    - on device ID `0`. Check your device ID using `/opt/qti-aic/tools/qaic-util -q` CLI tool, look for QID value in it.
    - verbose log is enabled with `-v` option.
- Tool in this configuration can be used to measure performance. 


#### 2. Running inference on a set of inputs

Before running inference, it is necessary to convert the inputs to the appropriate format based on input size and type. 
Look at this Jupyter notebook [example](https://github.com/quic/cloud-ai-sdk/blob/1.10/tutorials/NLP/Model-Onboarding-Beginner)

#### 3. Generating dumps for latency capture

- `aic-profiling-format latency`
- `aic-profiling-out-dir` : output directory for latency capture (needs to exist before this command is run)
- `aic-profiling-start-iter` : Set this value high enough to start capturing samples after device warmup 
- `aic-profiling-num-samples` : # of samples to be captured. Can be set greater than # of inferences

```shell title="Example command to generate latency stats"
!/opt/qti-aic/exec/qaic-runner -t ./BERT_LARGE -a 8 -S 1 -d 0 \ #-i inputFiles/input.raw \
--aic-profiling-format latency --aic-profiling-out-dir ./BERT_LARGE_STATS \
--aic-profiling-start-iter 100 --aic-profiling-num-samples 99999 --time 20 
```
Look at this Jupyter notebook [example](https://github.com/quic/cloud-ai-sdk/tree/1.10/tutorials/NLP/Profiler-Intermediate)

### Conclusion

`qaic-runner` CLI tool is primarily intended for performance testing purposes. For actual inference tasks, it is recommended to utilize the Python or C++ APIs, depending on your preferred technology stack. 

## 2. Use Python APIs

Refer to [Python API](../../../Python-API/qaic/qaic.md)

## 3. Use C++ APIs

Refer to [C++ API](../../../Cpp-API/example.md)

## 4. Use Onnxrt

Using [Qualcomm Cloud AI 100 as execution provider in onnxrt](../../../ONNXRT%20QAIC/onnxruntime.md).