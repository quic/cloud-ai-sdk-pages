# Compile the Model

The AI/ML models compilation step compiles a pre-trained model defined in other formats (ONNX is preferred) into QPC (Qaic Program Container) format.
This is required, since  Cloud AI devices works on this format, to run inference.

A pretrained model can be compiled in three ways:

1. Using qaic-exec (Binary executable shipped with the Apps SDK). This CLI tool has support for all the latest compiler flags.  
2. Using High Level Python APIs 
3. Using C++ APIs 


## Before compiling model into QPC format:
1. Use ONNX format as input.
2. Try to squash the original model into a single file ONNX format.
   

## Compilation using qaic-exec
The qaic-exec (QAic executor), a CLI tool, is used to compile the model. `qaic-exec` is a sophisticated tool with a number of arguments. 

qaic-exec CLI is located at `/opt/qti-aic/exec/qaic-exec`

The tutorials and models folder (#FIXME) provides several examples of `qaic-exec` usage. 

The help/usage command provides the extensive list of arguments and their descriptions for usage.
```
/opt/qti-aic/exec/qaic-exec -h
/opt/qti-aic/exec/qaic-exec --usage
```

Few of the frequently used arguments are additionally explained:
```
-m=<path>;, -model=<path> # specifies the path of input ONNX model 

-onnx-define-symbol=<sym, value> # defines the names and values of the ONNX symbols that needed to be passed into the QPC.
  For example
 -onnx-define-symbol=sequence,10 # For single symbol
 -onnx-define-symbol=sequence,10 -onnx-define-symbol=batch_size,8 # For more than one symbol

-aic-num-cores=<numCores>
  The Cloud AI cards can have a number of Neural Signal Processing cores (a.k.a AI cores) based on the SKU. The Inferencing workload can be distributed among different cores, so that they can execute concurrently and can produce more efficient inferencing. Refer to Tune Performance section on how to set this value.

-ols= <1,2,4,8.. numCores>
  Factor to increase splitting of network operations for more fine-grained parallelism. Refer to Tune Performance section on how to set this value.

-mos= <1,2,4,8.. numCores>
  Effort level to reduce on-chip memory usage. Refer to Tune Performance section on how to set this value.

-multicast-weights
  Reduce DDR bandwidth by loading weights used on multiple-cores only once and multicasting to other cores. 

-convert-to-fp16
  mandatory flag, ensures that compiled QPC executes all floating point computations on the AIC 100 device is 16-bit precision. 

-batchsize=<numBatch>
  batchsize refers to number of number of input samples that can be passed to the model during inferencing. Ideally a careful selection of batch size can facilitate better parallel processing and hence a better throughput from the device. Tune performance section expands on batchsize selection. 

-stats-batchsize=<numBatch>
  Set this value to numBatch. Used in performance statistics reporting

-aic-binary-dir=<path>
  specifies the output QPC path.

-aic-hw
  Mandatory flag. This flag enables the QPC to be run on hardware.

-compile-only 
  Mandatory flag, allows to only compile and produce the QPC format file for the model and does not run the model with random data. Use qaic-runner CLI to execute the QPC. 

-aic-hw-version=2.0
  The version string must be passed as "2.0" which is the .

-aic-profiling-type=<stats|trace|latency>
  Used in device level profiling. Refer to Profiler notebook under tutorials. 

```

`qaic-exec` can also be used to dump the operators supported across onnx, tensorflow, pytorch, caffe or caffe2 ML frameworks.

```
/opt/qti-aic/exec/qaic-exec -operators-supported=<onnx | tensorflow | pytorch | caffe | caffe2>
```
