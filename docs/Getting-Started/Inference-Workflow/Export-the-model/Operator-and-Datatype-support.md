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

The device performance is optimum for trained model weights in either `fp16` (half-precision) or `int8` formats.

If the model is originally trained in `fp32` format, it gets downconverted to `fp16` format with `-convert-to-fp16` during the compilation process. In certain scenarios, models may contain constants which are beyond the `fp16` range. In those scenarios, it is recommended to clip to `fp16` range as shown in this [notebook](https://github.com/quic/cloud-ai-sdk/blob/1.10/tutorials/NLP/Model-Onboarding-Beginner/getting-started-beginner-threading.ipynb), see the `fix_onnx_fp16` function. 

The device also offers support for mixed precision, specifically `fp16` and `int8`. While the device is technically capable of running `fp32` precision models using the scalar processor, its important to note that the performance achieved **will** be suboptimal compared to models utilizing `fp16` or `int8` precision. 

### Conversion from bf16 to fp16

If the model is trained in `bf16` (bfloat16),  Qualcomm can provide a script to identify the scaling factors to scale down the weights of the models such that intermediate activations will not overflow 'fp16'.