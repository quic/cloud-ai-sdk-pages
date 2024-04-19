## Operator support

In this section we will discuss the operator support available for the device across various frameworks. 

To determine the list of operators supported by the device for various frameworks, you can execute the following command

```bash title="Example command to generate operators supported for onnx"
/opt/qti-aic/exec/qaic-exec -operators-supported=onnx
```
This command generators a file `OnnxSupportedOperators.txt` which comprehensive list of ops supported. 

It is important to note that the operator support keep expanding with the release of new SDK versions.

???+ note
    ```
    -operators-supported supports only onnx, tensorflow, pytorch, caffe, and caffe2.
    ```

???+ note
    `onnx` is the preferred format to compile the model for the device. 

### Handling Unsupported Operators

In some cases, you might encounter errors related to unsupported operations while compiling the model for the device. 

For instance, certain operations like `einsum` present in the model file might not be directly supported by the device. 

In such scenarios, the [Model Preparator](Prepare-the-model.md) tool can be employed to modify the model and substitute these unsupported operations with their corresponding mathematical equivalent subgraphs.


## Datatype Support

### FP32 (Single Precision Floating Point)
Models can be executed in FP32 for use cases where accuracy is critical and computational efficiency is not a primary concern. It is essential to note that FP32 models tend to have larger sizes and will exhibit lower throughput performance. FP32 execution is supported but not recommended. Do *not* use the `-convert-to-fp16` with `qaic-exec` (compiler CLI) during compilation. 

### FP16 (Half Precision Floating Point)
FP16 strikes a balance between accuracy and efficiency, making it suitable for most deep learning workloads.

If a model is originally trained in FP32 format, it can be down-converted to FP16 during the compilation process using the `-convert-to-fp16` flag. However, certain scenarios may involve constants beyond the FP16 range. In such cases, it is recommended to clip values to the FP16 range (as demonstrated in the fix_onnx_fp16 function in the NLP tutorials in [Cloud-ai-sdk repo](https://github.com/quic/cloud-ai-sdk)).

### Shared Micro-exponents (Narrow Precision Format)

The shared micro-exponent spec is [here](https://www.opencompute.org/documents/ocp-microscaling-formats-mx-v1-0-spec-final-pdf). The Cloud AI compiler will support all of the formats over time. Currently, MXFP6 is supported by the compiler. 

Models compiled in MXFP6 format stores FP32/FP16 weights using a 6-bit format. By doing so, it significantly reduces model sizes due to the compact representation. This format is particularly beneficial for models that require high data bandwidth, such as large language models (LLMs).

AI100 stores Matmul weights in MXFP6 format while keeping the rest of the weights in FP16 format. Computation/activations on the NSP still occur in FP16.

LLMs experience up to 2x throughput with minimal accuracy loss with MXFP6 format. Within a constant memory footprint, a larger model can be supported with MXFP6. 

FP32 models can be compiled into MXFP6 format using the compiler flag `-mxfp6-matmul`. FP16 execution should use both `-convert-to-fp16` and `-mxfp6-matmul` flags for the `qaic-exec` compiler CLI. 

### UINT8 (Unsigned 8-bit Integer)
AI100 also supports executing INT8 quantized models, especially relevant for Natural Language Processing (NLP) and Computer Vision (CV) tasks.

Quantization Methods Supported:
- Quantization Schema for Weights and Activations: Both symmetric and asymmetric.
- Quantization Calibration: Options include KLMinimization, KLMinimizationV2, MSE, SQNR, and Percentile (with percentile calibration values: 99.9, 99.99, 99.999, 99.9999).

The SDKs provide tools to run a PGQ sweep, allowing you to identify the optimal quantization parameters for your specific requirements.

### BF16 (BFloat16) 

If a model is trained in BF16 (bfloat16), ensure that the weights are scaled down using an appropriate scaling factor. This prevents intermediate activations from overflowing into `fp16`. Qualcomm can provide a script to identify the scaling factors to scale down the weights of the models such that intermediate activations will not overflow `fp16`.