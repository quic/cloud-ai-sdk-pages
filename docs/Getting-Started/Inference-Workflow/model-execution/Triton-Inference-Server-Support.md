# Triton Inference Server

Triton Inference Server is an open source inference serving software that
streamlines AI inferencing. 

AIC100 SDK enables two backends for inference execution workflow.  These backends, once
used on a host with AIC100 cards, will detect the AIC100 cards during initialization
and route the inferencing call (requested to Triton server) to the hardware.

- onnxruntime_onnx as platform with QAic EP(execution provider)
- QAic as a customized C++ backend

![image](../../../images/server_frontends.png) 


## Creating a AIC100 backend enabled Triton docker image using AIC100 development kits

In order to add a customized backends (to process inferencing on AIC100 hardware) into a Vanilla Triton server image,
we need to run few scripts by passing sdk_path as parameters.
"docker-build.sh" script will generate a docker image as output.This script is a part of apps-sdk contents and can be run after unzipping it.

![image](https://github.qualcomm.com/storage/user/14267/files/af341a03-c8a0-41ef-969f-0f7acee83997)


```bash

sample> cd </path/to/app-sdk>/tools/docker-build

sample> ./build_image.sh --apps-sdk </path/to/apps-sdk.zip> --platform-sdk </path/to/platform-sdk.zip> --tag <tag-name> --os triton --create-model-repo
 
```
The above command will take 15-20 minutes to complete and generate a Triton docker image in local docker repository.

```bash

sample> docker image ls
REPOSITORY                                         TAG                 IMAGE ID       CREATED         SIZE
qaic-triton-x86_64-latest                          1.11.0.46-triton    cc217d09baea   6 days ago      34.4GB

```
Docker can be launched using docker "run" command passing the desired image name.
Please note that a shared memory argument(--shm-size) to pass for supporting ensembles and python backends.

```bash

sample> docker run -it --rm --privileged --shm-size=4g --ipc=host --net=host <triton-docker-image-name> /bin/bash

sample> docker ps
CONTAINER ID   IMAGE                                                                COMMAND                  CREATED      STATUS      PORTS     NAMES
b88d5eb98187   qaic-triton-x86_64-latest                                            "/opt/tritonserver/n…"   2 days ago   Up 2 days             thirsty_beaver

```

## Creating a model repository and configuration file

The model configuration file specifies the execution properties of a model.
It indicates input/output structure, backend, batchsize, parameters, etc.  User needs to follow Triton's [model repository](https://github.com/triton-inference-server/server/blob/main/docs/user_guide/model_repository.md) and [model configuration](https://github.com/triton-inference-server/server/blob/main/docs/user_guide/model_configuration.md) rules while defining a config file.

### Model configuration - onnxruntime
For onnxruntime configuration, platform should be set to "onnxruntime_onnx".
The "use_qaic" parameter should be passed and set to true.
#### AIC100 specific parameters
Parameters are user-provided key-value pairs which Triton will pass to backend runtime environment as variables and can be used in the backend processing logic.

- config : path for configuration file containing compiler options.
- device_id : id of AIC100 device on which inference is targeted. 
- use_qaic : flag to indicate to use qaic execution provider.
- hare_session : flag to enable the use of single session of runtime object across model instances.

sample example of a config.pbtxt

```
name: "resnet_onnx"
platform: "onnxruntime_onnx"
max_batch_size : 16
default_model_filename : "aic100/model.onnx"
input [
  {
    name: "data"
    data_type: TYPE_FP32
    dims: [3, 224, 224 ]
  }
]
output [
  {
    name: "resnetv18_dense0_fwd"
    data_type: TYPE_FP32
    dims: [1000]
  }
]
parameters [
  {
    key: "config"
    value: { string_value: "1/aic100/resnet.yaml" }
  },
  {
    key: "device_id"
    value: { string_value: "0" }
  },
  {
    key: "use_qaic"
    value: { string_value: "true" }
  },
  {
    key: "share_session"
    value: { string_value: "true" }
  }
]
instance_group [
  {
    count: 2
    kind: KIND_MODEL
  }
]

```
### Model configuration - qaic backend
For qaic backend configuration, the "backend" parameter should be set to "qaic".

#### AIC100 specific parameters
Parameters are user-provided key-value pairs which Triton will pass to backend runtime environment as variables and can be used in processing logic of backend.

Parameters are user-provided key-value pairs which Triton will pass to backend runtime environment as variables and can be used in processing logic of backend.

- qpc_path : path for compiled binary of model.(programqpc.bin)
- device_id : id of AIC100 device on which inference is targeted. device is set 0
- set_size : size of inference queue for runtime,default is set to 20
- no_of_activations : flag to enable multiple activations of a model’s network,default is set to 1

sample example of a config.pbtxt

```
name: "yolov5m_qaic"
backend: "qaic"
max_batch_size : 4
default_model_filename : "aic100/model.onnx"
input [
  {
    name: "images"
    data_type: TYPE_FP32
    dims: [3, 640, 640 ]
  }
]
output [
  {
    name: "feature_map_1"
    data_type: TYPE_FP32
    dims: [3, 80, 80, 85]
  },
  {
    name: "feature_map_2"
    data_type: TYPE_FP32
    dims: [3, 40, 40, 85]
  },
  {
    name: "feature_map_3"
    data_type: TYPE_FP32
    dims: [3, 20, 20, 85]
  }
]
parameters [
  {
    key: "qpc_path"
    value: { string_value: "/path/to/qpc" }
  },
  {
    key: "device_id"
    value: { string_value: "0" }
  }
]
instance_group [
  {
    count: 2
    kind: KIND_MODEL
  }
]
```

## Launching Triton server inside container
To launch Triton server, execute the tritonserver binary within Triton docker with the model repository path.

```bash
/opt/tritonserver/bin/tritonserver --model-repository=</path/to/repository>
```

![image](https://github.qualcomm.com/storage/user/14267/files/a5bb13a1-a8fc-4a00-91da-c00e73fba795)


## Supported Features

- Model Ensemble
- Dynamic Batching
- Auto device-picker
- Support for ARM64

















