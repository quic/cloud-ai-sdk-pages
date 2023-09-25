# Inference Profiling 
Cloud AI supports both **system** and **device** level profiling to aid developers to identify performance bottlenecks. System profiling can be performed without any changes to the QPC while device profiling requires the model to be recompiled. 

## System Level Profiling 

System level profiling includes the breakdown of the inference time between Application, linux runtime, kernel mode driver and device processing. This can provide insights into the inference time spent on the host vs device per inference. Developers can optimize their application or the model with this information. 

Refer to Profiler notebooks in the tutorials section for the complete system level profiling workflow for a model using the qaic-runner CLI. 

Profiling using Python APIs is also supported. Refer to this [page](../../../Python-API/qaic/qaic.md#api-list-function-variables-for-session-object).

## Device Level Profiling 
Device level profiling is aimed at advanced developers looking to identify bottlenecks in inference execution on the device. This requires a good understanding of the AI core and SoC architecture. There are 3 key features that would be interesting to developers -

  - Memory metrics - This provides a compiler estimate of the usage of on-board DDR vs VTCM (Vector tightly coupled memory) for a model.
  - Summary view - This provides a histogram of the operations, total time taken by every operation, where the operands are stored (DDR vs VTCM), effective usage of the individual IP blocks in the AI cores etc. This feature is for debug only as it  may impact performance based on the size of the model. 
  - Timeline view - This provides a timeline view of all the operations executing across all IP blocks from start of an inference till the end. This feature is primarily used to zoom into the operations to understand bottlenecks. This feature is for debug only as it will impact performance. 

Refer to Profiler Jupyter [notebook](https://github.com/quic/cloud-ai-sdk/tree/1.10/tutorials/NLP/Profiler-Intermediate) for the complete device level profiling workflow for a model using the qaic-exec and qaic-runner CLI. 

