# QAIC execution provider

The QAIC Execution Provider for ONNX Runtime enables hardware accelerated execution on Qualcomm AIC100 chipset. It leverages AIC compiler and runtime API packaged in apps, platform SDKs.


## Installation Pre-requisites

#### System requirements

 - OS: Ubuntu 18.04+
 - Packages: gcc/g++ 9, git, cmake 3.18+, python 3.6+
 - Python requirements: numpy, wheel, flatbuffers-2.0 yaml

#### Download and Install

 - Apps SDK
 - Platform SDK (optional if intending to execute on simulator)

## Build

Set the environment

``` bash
 export QAIC_APPS="/opt/qti-aic/examples/apps"
 export QAIC_LIB="/opt/qti-aic/dev/lib/x86_64/libQAic.so"
 export QAIC_COMPILER_LIB="/opt/qti-aic/dev/lib/x86_64/libQAicCompiler.so"
```

`build_onnxrt_qaic.sh` is a wrapper script for onnxruntime's `build.sh`. 

It clones version `1.10.0` of onnxruntime into the folder onnxruntime_qaic, applies a patch comprising changes to support QAIC EP and builds it. Run the script with default options as follows.

```bash
bash /opt/qti-aic/integrations/qaic-onnxrt/build_onnxrt_qaic.sh
```

## Configuration Options

| option | type | description  |
|--------|:-------:|:------|
| config    | str   | [Required] Path to the model-settings YAML file. Contains AIC configuration parameters used by QAic execution provider of ONNX Runtime. The configuration for best performance and accuracy can be generated using model configurator tool.  |
| aic_device_id    | int   | [Optional] AIC device ID, auto-picked when not configured |

### Parameters supported in model settings yaml file

| Option |	Description |	Default |	Relevance
|--------|:-------|:------:|------|
||Runtime parameters|
| aic-binary-dir |	Absolute path or relative path ( wrt model settings file parent directory) to dir with programqpc.bin	| ""	| Required to skip compilation. |
| device-id	| AIC device ID	| 0	| Optional
| set-size	| Set Size for inference loop execution	| 10 |	Optional |
| aic-num-of-activations |	Number of activations |	1 |	Optional |
|| qaicRegisterCustomOp - Compiler C API ||
| register-custom-op |	Register custom op using this configuration file | | Required if model has AIC custom ops; vector of string |
| | Graph Config - Compiler API | |
|aic-depth-first-mem	| Sets DFS memory size	| Set by compiler |	Optional. Used in compilation with aic-enable-depth-first |
| aic-enable-depth-first	| Enables DFS with default memory size; "True", "False" |	Set by compiler |	Optional. Used in compilation. |
| aic-num-cores	| Number of aic cores to be used for inference on |	1 |	Optional. Used in compilation. |
|allocator-dealloc-delay |	Option to increase buffer lifetime 0 - 10, e.g 1 | Set by compiler	|Optional. Used in compilation.|
|batchsize |	Sets the number of batches to be used for execution |	1 |	Optional. Used in compilation.|
|convert-to-fp16| Run all floating-point in fp16; "True", "False" |	"False"	| Optional. Used in compilation.|
|enable-channelwise |	Enable channelwise quantization of Convolution op; "True", "False" |	Set by compiler	| Optional. Used in compilation with pgq-profile.|
|enable-rowwise	| Enable rowwise quantization of FullyConnected and SparseLengthsSum ops; "True", "False"	| Set by compiler |	Optional. Used in compilation with pgq-profile.|
|execute-nodes-in-fp16	| Run all insances of the operators in this list with FP16; "True", "False"	| Set by compiler |	Optional. Used in compilation with pgq-profile for mixed precision.|
|hwVersion |	HW version of AI| 	QAIC_HW_V2_0	| Cannot be configured, set to QAIC_HW_V2_0.|
|keep-original-precision-for-nodes |	Run operators in this list with original precision at generation |	| Optional. Used in compilation with pgq-profile for mixed precision.|
|mos |	Effort level to reduce the on-chip memory; eg: "1"	| Set by compiler	| Optional. Used in compilation.|
|multicast-weights	| Reduce DDR bandwidth by loading weights used on multiple-cores only once and multicasting to other cores| | |	
|ols |	Factor to increasing splitting of network for parallelism |	Set by compiler	| Optional. Used in compilation. |
|quantization-calibration	| Specify quantization calibration -"None", "KLMinimization", "Percentile", "MSE", "SQNR", "KLMinimizationV2" |	"None"	| Optional. Used in compilation with pgq-profile.|
|quantization-schema-activations |	Specify quantization schema - "asymmetric", "symmetric", "symmetric_with_uint8", "symmetric_with_power2_scale" |	"symmetric_with_uint8"	| Optional. Used in compilation with pgq-profile.|
|quantization-schema-constants	| Specify quantization schema -"asymmetric", "symmetric", "symmetric_with_uint8", "symmetric_with_power2_scale" |	"symmetric_with_uint8"	| Optional. Used in compilation with pgq-profile.|
|size-split-granularity	| To set max tile size, KiB between 512 - 2048, e.g 1024 |	Set by compiler |	Optional. Used in compilation.|
|aic-hw	 | To set the target to QAIC_SIM or QAIC_HW; "True", "False"| "True" |	Optional. |
|| Model Params - Compiler API|
|model-path	|Path to model file	| | Required. Used in compilation, OnnxRT framework.|
|onnx-define-symbol |	Define an onnx symbol with its value. pairs of onnx symbol key,value separated by space. | | Required. Used in compilation, OnnxRT framework.|
|external-quantization |	Path to load the externally generated quantization profile || Optional|
|node-precision-info	| Path to load model loader precision file for setting node instances to FP16 or FP32 | |Optional. Used in compilation with pgq-profile for mixed precision. |
| | Common | |
|relative-path	| aic-binary-dir absolute path will be constructed using base-path of model-settings file; "True", "False" |"False"	| Optional. Set to true, to allow relative-path for aic-binary-dir.|

## Usage

### Python

Here are few basic commands you can use with ONNX Runtime and QAIC.

#### Load a model

```python
import onnxruntime as ort
provider_options = []  
qaic_provider_options = {} 
qaic_provider_options['config'] = '/path/to/yaml/file' 
qaic_provider_options['device_id'] = aic_device_id 
provider_options.append(qaic_provider_options) 
session=onnxruntime.InferenceSession('/path/to/onnx/model', sess_options,                                                
                                           providers = providers, provider_options = providers_options)
```

This will bind your model to AIC100 chip, with qaic exectuion provider.

#### Perform Inference 


```python
# Perform inference using OnnxRuntime
results = sess.run(None, {'input_name': input_data})
```

In the above code replace `'input_name'` with name of model input node and input_data with the actual input data.


### C++

#### Load a Model

```cpp
#include <onnxruntime_cxx_api.h>
#include <qaic_provider_factory.h>


// Set environment as required
Ort::Env env(ORT_LOGGING_LEVEL_ERROR, "test");
// Initialize session options, create session
Ort::SessionOptions session_options;
session_options.SetIntraOpNumThreads(1);
session_options.SetGraphOptimizationLevel(
    GraphOptimizationLevel::0);
auto s = OrtSessionOptionsAppendExecutionProvider_QAic(
        session_options, "/path/to/yaml/file", aic_device_id);

Ort::Session session(env, "/path/to/onnx/model", session_options);
```


#### Perform inference

```cpp
// Run the model

auto output_tensors = session.Run(Ort::RunOptions{nullptr},
input_names.data(), &input_tensor, 1, output_names.data(), 1);
```

In the above code, replace `"/path/to/onnx/model/"` to the path for your onnx file. Also ensure data and shape of your input tensor match the requirements of your model.

## End-to-end examples

End to end examples (cpp and python) for resnet50 are available at - `/opt/qti-aic/integrations/qaic_onnxrt/tests/`.

### Running the ResNet C++ sample   

Compile the Sample Resnet C++ test using build_tests.sh script. By default, test is built using libs from onnxruntime_qaic release build. To enable debugging, re-build onnxruntime_qaic project in Debug configuration and run ./build_test.sh with debug flag.  
 
``` bash
build_tests.sh [--release|--debug]    
``` 

Run the executable. The commands below set the environment and run the ResNet-50 model with the provided image on QAic or CPU backend. The program outputs the most probable prediction class index for each iteration.   

``` bash
cd build/release 
./qaic-onnxrt-resnet50-test -i <path/to/input/png/image>  
                            -m  ../../resnet50/resnet50.yaml 
```

Test options   

| Option  | Description  |
|---------|:--------------|
|-m, --model-config | [Required] Path to the model-setting yaml file |
|-i, --input-path  | [Required]  Path to the input PNG image file | 
|-b, --backend  | [Optional] Default='qaic', Specify qaic/cpu as backend | 
|-d, --device-id | [Optional]  Default=0 Specify qaic device ID  |
|-n, --num-iter | [Optional] | Default=100, Specify num iterations for the test | 

  

### Running the ResNet Python sample   

  
``` bash
Run test_resnet.py at /opt/qti-aic/integrations/qaic_onnxrt/tests/resnet50  
python test_resnet.py --model_config ./resnet50/resnet50.yaml   
                      --input_file </path/to/png/image>  
```
  
Test options

| Option  | Description  |
|---------|:--------------|
|--model_config   | [Required] Path to the model-setting yaml file |
|--input_file  | [Required]  Path to the input PNG image file | 
|--backend  | [Optional] Default='qaic', Specify qaic/cpu as backend | 
|--device_id | [Optional]  Default=0 Specify qaic device ID  |
|--num_iter | [Optional] | Default=100, Specify num iterations for the test |
 

### Running models with generic QAic EP test  

`test_qaic_ep.py` is a generic test runner for compilation, execution on AIC100.
  

Run `test_qaic_ep.py` at `/opt/qti-aic/integrations/qaic_onnxrt/tests/` -

``` bash
python test_qaic_ep.py --model_config ./resnet50/resnet50.yaml   
                           --input_file_list </path/to/input/list>   
```
  

Test options    

| Option  | Description  |
|---------|:--------------|
|--model_config   | [Required] Path to the model-setting yaml file |
|--input_file_list  | [Required]  Path of the file (.txt) containing list of batched inputs in .raw format | 
|--backend  | [Optional] Default='qaic', Specify qaic/cpu as backend | 
|--device_id | [Optional]  Default=0 Specify qaic device ID  |
|--num_iter | [Optional] | Default=100, Specify num iterations for the test |
|--max_threads | [Optional]  Default=1000 Maximum no. of threads to run inferences  |
|--log_level | [Optional] | Default=4, Onnxruntime log severity level (0-4) |

## Execution through Onnxruntime test framework

- QAic EP is enabled for execution with onnx_test_runner, onnxruntime_perf_test.

- For model directory requirements and comprehensive list of options supported, refer to [onnxruntime perf test documentation](https://github.com/microsoft/onnxruntime/blob/v1.10.0/onnxruntime/test/perftest/README.md).

- [onnx_test_runner documentation](https://github.com/microsoft/onnxruntime/blob/v1.10.0/onnxruntime/test/onnx/README.txt).

- Sample testdata can be downloaded [here](https://github.com/onnx/models/blob/main/vision/classification/resnet/model/resnet50-v1-7.tar.gz).

```bash
cd /opt/qti-aic/integrations/qaic_onnxrt/onnxruntime_qaic/build/Release

./onnxruntime_perf_test -e qaic -i 'config|/path/to/resnet50.yaml aic_device_id|0' -m times -r 1000 /path/to/model.onnx
```

```bash
./onnx_test_runner -e qaic /path/to/model/dir
```