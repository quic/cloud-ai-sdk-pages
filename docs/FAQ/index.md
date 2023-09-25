# Frequently Asked Questions

## General 
??? question "What is Cloud AI 100 Accelerator?"
    Cloud AI 100 accelerators enable high performance inference on deep learning models. The accelerators are available in multiple form factors and associated SKUs. Cloud AI SDKs enable end to end workflows - from onboarding a pre-trained model to deployment of the ML inference application in production. 

## Cloud AI SDK Installation and Platform/OS Support

??? question "What operating systems and platforms are supported?"
    See [Operating System and Platform Support](../Getting-Started/Installation/index.md#supported-operating-systems-hypervisors-and-platforms)

??? question "Where do I download the SDKs?"
    Cloud AI SDK consists of a Platform and Apps SDK. Refer to [Cloud AI SDKs](../Getting-Started/index.md#cloud-ai-sdks) for more information. 

    For Platform SDK download, see [Platform SDK Download](../Getting-Started/Installation/Cloud-AI-SDK/Cloud-AI-SDK.md#platform-sdk)

    For Apps SDK download, see [Apps SDK Download](../Getting-Started/Installation/Cloud-AI-SDK/Cloud-AI-SDK.md#apps-sdk)

??? question "What environment variables need to be set to resolve toolchain errors such as libQAic.so?"
    Set the environment variables as mentioned [here](../Getting-Started/Installation/Pre-requisites/pre-requisites.md#addupdate-environment-variables)


## Deep Learning frameworks and networks 

??? question "Which deep learning frameworks for supported by Cloud AI SDKs?"
    Onnx, tensorflow, pytorch, caffe or caffe2 are supported by the [compiler](../Getting-Started/Inference-Workflow/model-compilation/Compile%20the%20Model.md). 
    `qaic-exec` can dump the operators supported across different frameworks. Onnx has the best operator support. 
    
??? question "Which deep learning neural networks are supported?"
    Cloud AI platforms supports many network categories - Computer vision, object detection, Semantic segmentation, Natural language processing, ADAS and Generative AI networks. 
    Performance information can be found in the [Qualcomm Cloud AI 100 page](https://www.qualcomm.com/products/technology/processors/cloud-artificial-intelligence) 
    Model recipes can be found in the `cloud-ai-sdk` github. 

??? question "I have a neural network that I would like to run on Cloud AI platforms. How do I go about it?"
    There are 3 steps run an inference on Cloud AI platforms. 

    1. Export the model in ONNX format (preferred due to operator support) and prepare the model 
    2. Compile the model to generate a QPC (Qaic Program Container)
    3. Execute, integrate and deploy into production pipeline

    The [quick start guide](../Getting-Started/Quick-Start-Guide/index.md) provides a quick overview of the steps involved in running inference using a vision transformer model as an example. 

    Refer to [Inference Workflow](../Getting-Started/Inference-Workflow/index.md) for detailed information how to onboard and run inference on Cloud AI platforms. 

    Users can also refer to the [model recipes](https://github.com/quic/cloud-ai-sdk) that provide the best performance for several networks across several categories. 

    [Tutorials](https://github.com/quic/cloud-ai-sdk) are another resource that walks through the workflows to onboard models, tune for best performance, profile inferences etc. 

    
## System Management 
??? question "Which utility/tool is used to query health, telemetry etc of all Cloud AI cards in the server?"
    Use the [qaic-util](../Getting-Started/System-Management/system-management.md) CLI tool to query health, telemetry etc of Cloud AI cards in the server. 

??? question "The Cloud AI device shows `Status:Error`. How do i fix it?"
    `Status: Error` could be due to one of the following:

    - indicates the respective card(s) has not booted up completely 
    - user has not used `sudo` prefix if user has not been added to `qaic` group. `sudo /opt/qti-aic/tools/qaic-util`
    - unsupported OS/platforms, secure boot etc

    Users can try to issue an [soc_reset](../Getting-Started/System-Management/system-management.md#soc-reset) to see if the device recovers. 
