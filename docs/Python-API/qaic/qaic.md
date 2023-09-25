# qaic package
------ 

`qaic` - qaic package provides a way to run inference on Qualcomm Cloud AI 100 card.

### Description
------ 

A user can create a session object by passing in 

1. with a `.onnx` file.

2. with a precompiled qpc as model_path, in case a user already has compiled qpc. Full path to qpc.bin should be passed while using precompiled binary.

???+ note
    QPC : Qualcomm Program Container

#### Example on how to run an inference

```python title="option 1 : Compiles the onnx file to generate QPC and sets up session for inference"
import qaic
import numpy as np
sess = qaic.Session('/path/to/model/model.onnx') 
input_dict = {'input_name': input_data}
output = sess.run(input_dict)
```

```python title="option 2 : Uses generate QPC and sets up session for inference"
import qaic
import numpy as np
sess = qaic.Session('/path/to/model/qpc.bin') # option 2 : Session uses compiled QPC file to 
input_dict = {'input_name': input_data}
output = sess.run(input_dict)
```

#### Example for benchmarking

```py
import qaic
sess = qaic.Session(model_path='/path/to/model', backend='aic', options_path = '/path/to/yaml') # model_path can be either onnx or precompiled qpc
inf_completed, inf_rate, inf_time, batch_size = sess.run_benchmark()

```

### Limitations

- Currently only QAic backend is supported. We plan support for QNN backend in future releases. 
- APIs are compatible with only Python 3.8 
- These APIs are supported only on x86-64 platforms 

## class Session
------ 

Session is the entry point of these APIs. Session is a factory method which user needs to call to create an instance of session.
A model is compiled by default when creating a session.


``` python
Session(model_path, **kwargs)
```

Session creates a session object based on the qpc provided.
[#]: <> (These comments will only be visible in markdown raw file. it will not appear on the rendered webpage.)
[#]: <> (Session defenition is edited for 1.9. Ability to compile is masked from the user since it doesn't compile when import torch is used.)
[#]: <> (provided in args. By default the backend is `qaic`, hence session object with qaic backend will be created by default.)
[#]: <> (Session compiles the model by default. If a user doesn't want to compile the model, user has to pass 'compile=False' as argument to Session.)

### Parameters

| Parameter      | Type | Description                          |
| ----------- | ------ | ------------------------------ |
| `model_path`       |`str`  | path to .onnx file or .bin file i.e. the compiled model QPC |
| `**kwargs`      |   | refer to the keyword arguments listed below |

| Keyword Arguments      | Type | Description                          |
| ----------- | ------ | ------------------------------ |
| `dev_id`       |`int`  | Device on which to run the inference. Default is 0 |
| `num_activations`      | `int`  | Number of instances on network to be activated. |
| `set_size`      | `int`  | Number of ExecObj to be created. |
| `mos`      | `int`  | Effort level to reduce the on-chip memory. |
| `ols` | `int` | 	Factor to increasing splitting of network for parallelism | 
| `aic_num_cores` | `int` |	Number of aic cores to be used for inference |
| `convert_to_fp16 ` | `bool` |	Run all floating-point in fp16 |
| `onnx_define_symbol` | `(list[tuple(str, int)])`	| Define an onnx symbol with its value |
| `output_dir ` | `str` | 	Stores model binaries at directory location provided |
| `output_node_names` | `(list[str])`	| Output node names should be in the order as present in model file. This option is mandatory for TF models |
| `model_inputs list` | `dict` | `Provide input node name with its data type and shape. Dict must contain keys 'input_name', 'input_type','input_shape'.  This is mandatory for pytorch models | 
| `allocator_dealloac_delay` | `int` |	Option to increase the lifetime of buffers to reduce false dependencies | 
| `size_split_granularity` | `int` |	Option to specify a maximum tile size target for operations that may be too large to execute out of fast memory. Tile size in KiB between 512 - 2048 | 
| `vtcm_working_set_limit_ratio ` | `float` |	Option to Specify the maximum ratio amount of fast memory to DDR any single instruction is allowed use of all available value between {0 - 1} | 
| `execute_nodes_in_fp16` | `(list[str])` |	Run all insances of the operators in this list with FP16 |
| `node_precision_info_file ` | `str` |	Load model loader precision file which contains first output name of operator instances required to be executed in FP16 or FP32. |
| `keep_original_precision_for_nodes` | `(list[str])`	| Run all insances of the operators in this list with original precision during generation of quantized precision model even if the operator is supported in Int8 precision |
| `custom_io_list_file ` | `str` |	Custom I/O config file in yaml format containing layout, precision scale and offset for each input and output of the model. |
| `dump_custom_io_config_template_file ` | `str` |	Dumps the yaml template for Custom I/O configuration |
| `external_quantization_file ` | `str` |	Load the externally generated quantization profile | 
| `quantization_schema_activations ` | `str` |	Specify which quantization schema to use for activations. Valid options: asymmetric, symmetric, symmetric_with_uint8 (default), symmetric_with_power2_scale | 
| `quantization_schema_constants ` | `str` |	Specify which quantization schema to use for constants. Valid options: asymmetric, symmetric, symmetric_with_uint8 | `(default), symmetric_with_power2_scale |
| `quantization_calibration ` | `str` |	Specify which quantization calibration to use Default is None (MinMax calibration is applied). Valid options: None (default), KLMinimization, KLMinimizationV2, Percentile, MSE and SQNR. |
| `percentile_calibration_value ` | `float` |	Specify the percentile value to be used with Percentile calibration method. The specified float value must lie within 90 and 100, default: 100. |
| `num_histogram_bins` | `int` |	Sets the num of histogram bins that will be used in profiling every node. Default value is 512 |
| `quantization_precision ` | `str` |	Specify which quantization precision to use. Int8(default) is only supported precision for now. |
| `quantization_precision_bias ` | `str` |	Specify which quantization precision to use. Value options: Int8, Int32 (default) |
| `enable_rowwise ` | `bool` |	Enable rowwise quantization of FullyConnected and SparseLengthsSum ops. |
| `enable_channelwise` | `bool` |	Enable channelwise quantization of Convolution op. |
| `dump_profile` | `str` |	Perform quantization profiling for a given graph and dump result to the file. Compilation will be done after dumping profile unlike qaic-exec |
| `load_profile` | `str` |	Load quantization profile file and quantize the graph. The profile file to be loaded is the one which is dumped through option -dump-profile. |
| `convert_to_quantize` | `bool` |	If -load-profile option is not provided then input data is profiled and run in quantized mode. Default is off. Also set-quantization-* options as per requirement. Do not use this option along with -dump-profile or -load-profile. |
| `load_embedding_tables` | `str` |	Load embedding tables from this zip file for DLRM and RecSys models. |
| `dump_embedding_tables` | `str` |	Extract embedding tables from pytorch model and dump them in the zip file specified. |
| `mdp_load_partition_config` | `str` |	Load config file for partitioning a graph across devices. |
| `mdp_dump_partition_config` | `str` |	Dump config file for partitioning a graph across devices. |
| `host_preproc` | `bool` |	Enable all pre-/post-processing on host |
| `aic_preproc` | `bool` |	Disable all pre-/post-processing on host. Operations are performed on AI 100 instead. |
| `aic_enable_depth_first` | `bool` |	Enables DFS with default memory size |
| `aic_depth_first_mem` | `int` |	Sets DFS memory size. number must be choosen from [8,32] |
| `stats_batchsize` | `int` |	This option is used to normalize performance statistics to be per inference |
| `always_expand_onnx_functions` | `bool` |	This option forces the expansion ONNX functions.  |
| `enable_debug` | `bool` |	Enables debug mode during model compilation |
| `time_passes` | `bool` |	Enables printing of compile-time statistics |
| `io_crc` | `bool` |	Enables CRC check for inputs and outputs of the network. |
| `io_crc_stride` | `int` |	Specifies size of stride to calculate CRC in the stride section |
| `sdp_cluster_sizes` | `(list[int])` |	Enables single device partitioning and sets the cluster configuration |
| `profiling_threads` | `int` |	This option is used to assign the number of threads to use for for quantization profile generation |
| `compile_threads` | `int` |	Sets the number of parallel threads used for compilation.
| `use_producer_dma` | `bool` |	Initiate NSP DMAs from the thread that produces data being transferred |
| `aic_perf_warnings` | `bool` |	Print performance warning messages |
| `aic_perf_metrics` | `bool` |	Print compiler performance metrics |
| `aic_pmu_recipe` | `str` |	Enable the PMU selection based on built-in recipe: AxiRd, AxiWr, AxiRdWr, KernelUtil, HmxMacs |
| `aic_pmu_events` | `str` |	Track events in NSP cores. Up to 8 events are supported |
| `dynamic_shape_input` | `(list[str])`	| Inform the compiler which inputs should be treated as having dynamic shape |
| `multicast_weights` | `bool` |	Reduce DDR bandwidth by loading weights used on multiple-cores only once and multicasting to other cores. |
| `ddr_stats` | `bool` |	Used to collect DDR traffic details at per core level. |
| `combine_inputs` | `bool` |	When enabled combines inputs into fewer buffers for  transfer to device. |
| `combine_outputs` | `bool` |	When enabled combines outputs into a single buffer for transfer to host. |
| `enable_metrics ` | `bool` |	Set value to True if you are interested in getting performance metrics for inference runs on a session. (Can not be used if enable_profiling is set to True.) |
| `enable_profiling ` | `bool` |	Set value to True if you want to profile the inferences and get performance metrics for inference runs on a session. (Can not be used if enable_metrics is set to True.) |

### Returns

Session object.

### Example

```python title="using options_path yaml file"
sess = qaic.Session('/path/to/model', options_path = '/path/to/options.yaml') 
input_dict = {'input_name': input_data}
output = sess.run(input_dict)
```

```yaml title="sample contents of yaml file"
aic_num_cores: 4
num_activations: 1
convert_to_fp16: true
onnx_define_symbol:
  batch: 1
output_dir: './resnet_qpc'
```

```python title="using keyword args"
    sess = qaic.Session('/path/to/model_qpc/*.bin', num_activations=4, set_size=10)
    input_dict = {'input_name': input_data}
    output = sess.run(input_dict)
```

### API List (Function variables for session object)

Session class has following methods.

#### backend_options()	

**Returns**

A dict of options that can be configured after creating session	

```py title='Usage example'
backend_options_dict = session.backend_options()
```

#### get_metrics()	

**Returns**

A dictionary containing the following metrics:

```
- num_of_inferences (int): The number of inferences.
- min_latency (float): The minimum inference time.
- max_latency (float): The maximum inference time.
- P25 (float): The 25th percentile latency.
- P50 (float): The 50th percentile latency (median).
- P75 (float): The 75th percentile latency.
- P90 (float): The 90th percentile latency.
- P99 (float): The 99th percentile latency.
- P999 (float): The 99.9th percentile latency.
- total_inference_time (float): The sum of individual insference times.
- avg_latency (float): The average latency.
```

```py title='Usage example'
metrics_dict = session.get_metrics()
```

#### model_input_shape_dict()	

**Returns**

A dict with input_name as key and input_shape, input_type as values 	

```py title='Usage example'
input_shape_dict = session.model_input_shape_dict()	
```

#### model_output_shape_dict()	

**Returns**

A dict with output_name as key and output_shape, output_type as values 	

```py title='Usage example'
output_shape_dict = session.model_output_shape_dict()
```

#### print_metrics()

**Returns**

`None`

```py title='Usage example'
session.print_metrics()	
```
???+ note

    This method assumes that either the 'enable_profiling' or 'enable_metrics' attribute is set to True.

Sample Output:

```
Number of inferences utilized for calculation are 999
Minimum latency observed 0.0009578340000000001 s
Maximum latency observed 0.002209001 s
Average latency / inference time observed is 0.0012380756316316324 s
P25 / 25% of inferences observed latency less than 0.001095435 s
P50 / 50% of inferences observed latency less than 0.0012522870000000001 s
P75 / 75% of inferences observed latency less than 0.001299786 s
P90 / 90% of inferences observed latency less than 0.002209001 s
P99 / 99% of inferences observed latency less than 0.0016082370000000002 s
Sum of all the inference times 1.2368375560000007 s
Average latency / inference time observed is 0.0012380756316316324 s
```

#### print_profile_data	

**Returns**

 `None`

```py title='Usage example'
session.print_profile_data(n)	
```

Print profiling data for the first n iterations

???+ note
    This function only works when 'enable_profiling' is set to True for the Session.

- This method assumes that the 'enable_profiling' attribute is set to True, and the 'profiling_results' attribute contains the profiling data for each iteration.
- The method prints the profiling data in a tabular format, including the file, line, function, number of calls, function time (seconds), and total time (seconds) for each function.


Sample Output:

```
|  File-Line-Function  | |  num calls  | |  func time  | |  tot time  |

('~', 0, "<method 'astype' of 'numpy.ndarray' objects>") 1 0.000149101 0.000149101

('~', 0, '<built-in method numpy.empty>') 1 2.38e-06 2.38e-06

('~', 0, '<built-in method numpy.frombuffer>') 1 4.22e-06 4.22e-06

```

#### reset()	

**Returns**

 `None`	

```py title='Usage example'
session.reset()	
```

Releases all the device resources acquired by session 

#### setup()	

**Returns**

 `None`	

```py title='Usage example'
session.setup()	
```

Loads the network to the device.

Network is usually loaded during first call of run. If this is called before that, network will be already loaded when first run is called.

#### run(input_dict)	

**Returns**

A dict with output_name and output_value of inference	

```py title='Usage example'
output = session.run(input_dict)	
```

input_dict should have input_name as key and value should be a numpy array


#### run_benchmark()	

**Returns**

inf_completed: Total number of inferences run
inf_rate: Inf/Sec of the model
inf_time: total time taken to run inferences
batch_size: Batch Size used by model

```py title='Usage example'
inf_completed, inf_rate, inf_time, batch_size = session.run_benchmark()	
```


It accepts following args:

num_inferences: num of inferences to run in benchmarking. Default 40
inf_time: duration for which inference is to be run in seconds. Default None
input_dict: Input to be used in inference. Default random

???+ note
    num_inferences and time cannot be used together.

This API uses C++ benchmarking APIs and doesn't take into account python overheads


#### update_backend_options(**kwargs)	

**Returns**

 `None`	

```py title='Usage example'
session.update_backend_options(num_activations = 2) 	
```

Update option specified in kwargs

For example:

 `num_activation`, `dev_id`, `set_size` can be configured with this API.