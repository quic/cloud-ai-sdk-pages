# Triton Inference Server

Triton Inference Server is an open source inference serving software that
streamlines AI inferencing. 

Cloud AI SDK enables two backends for inference execution workflow.  These backends, once
used on a host with AI 100 cards, will detect the AI 100 cards during initialization
and route the inferencing call (requested to Triton server) to the hardware.

- onnxruntime_onnx as platform with QAic EP(execution provider) for deploying ONNX graphs.
- QAic as a customized C++ backend for deploying compiled binaries optimized for AIC.

![image](../../../images/server_frontends.png) 


## Creating an AI 100 backend enabled Triton Docker image using AI 100 development kits

In order to add customized backends (to process inferencing on AI 100 hardware) into a vanilla Triton server image,
we need to run a few scripts by passing sdk_path as parameters.
`docker-build.sh` script will generate a Docker image as output.This script is a part of Cloud AI Aps SDK contents and can be run after unzipping it.

![image](../../../images/docker_workflow.png)


```bash

sample> cd </path/to/app-sdk>/tools/docker-build

sample> python3 build_image.py --tag 1.16.1.29-triton --log_level 2 --user_specification_file /opt/qti-aic/tools/docker-build-gen2/sample_user_specs/user_image_spec_triton_model_repo.json --apps-sdk /apps/sdk/path --platform-sdk /platform/sdk/path
 
```
The above command may take 15-20 minutes to complete and generate incremental images for Triton Docker image in local Docker repository.

```bash

sample> docker image ls
REPOSITORY                                                                                                                      TAG                 IMAGE ID       CREATED         SIZE
qaic-x86_64-triton-release-py38-qaic_platform-qaic_apps-pybase-onnxruntime-triton-pytools-triton_model_repo               1.16.1.29-triton          a0968cf3711b   3 days ago      28.2GB
qaic-x86_64-triton-release-py38-qaic_platform-qaic_apps-pybase-onnxruntime-triton-pytools                                 1.16.1.29-triton          038dc80fd8e4   3 days ago      27.1GB
qaic-x86_64-triton-release-py38-qaic_platform-qaic_apps-pybase-onnxruntime-triton                                         1.16.1.29-triton          760fb9dc5314   3 days ago      24GB
qaic-x86_64-triton-release-py38-qaic_platform-qaic_apps-pybase-onnxruntime                                                1.16.1.29-triton          a47266156b7f   3 days ago      23.9GB
qaic-x86_64-triton-release-py38-qaic_platform-qaic_apps-pybase                                                            1.16.1.29-triton          d620a1bdb6b6   3 days ago      20.1GB
qaic-x86_64-triton-release-py38-qaic_platform-qaic_apps                                                                   1.16.1.29-triton          8c87eb44f2db   3 days ago      15.3GB
qaic-x86_64-triton-release-py38-qaic_platform                                                                             1.16.1.29-triton          e3ba2ce282c1   3 days ago      14.7GB
qaic-x86_64-triton-release-py38                                                                                           1.16.1.29-triton          73b225d7e358   3 days ago      14.2GB
qaic-x86_64-triton-release                                                                                                1.16.1.29-triton          914fa376e865   3 days ago      14.1GB
qaic-x86_64-triton                                                                                                        1.16.1.29-triton          2090680d4d59   3 days ago      14.1GB

```
Docker can be launched using docker `run` command passing the desired image name.
Please note the shared memory argument `--shm-size` for supporting ensembles and python backends.

```bash

sample> docker run -it --rm --privileged --shm-size=4g --ipc=host --net=host <triton-docker-image-name>:<tag> /bin/bash

sample> docker ps
CONTAINER ID   IMAGE                                                                                                            COMMAND                  CREATED      STATUS      PORTS     NAMES
b88d5eb98187   qaic-x86_64-triton-release-py38-qaic_platform-qaic_apps-pybase-onnxruntime-triton-pytools-triton_model_repo      "/opt/tritonserver/n…"   2 days ago   Up 2 days             thirsty_beaver

```

## Creating a model repository and configuration file

The model configuration file specifies the execution properties of a model.
It indicates input/output structure, backend, batchsize, parameters, etc.  User needs to follow Triton's [model repository](https://github.com/triton-inference-server/server/blob/main/docs/user_guide/model_repository.md) and [model configuration](https://github.com/triton-inference-server/server/blob/main/docs/user_guide/model_configuration.md) rules while defining a config file.

### Model configuration - onnxruntime
For onnxruntime configuration, platform should be set to `onnxruntime_onnx`.
The `use_qaic` parameter should be passed and set to true.
#### AI 100 specific parameters
Parameters are user-provided key-value pairs which Triton will pass to backend runtime environment as variables and can be used in the backend processing logic.

- config : path for configuration file containing compiler options.
- device_id : id of AI 100 device on which inference is targeted. (not mandatory as the server auto picks the available device)
- use_qaic : flag to indicate to use qaic execution provider.
- share_session : flag to enable the use of single session of runtime object across model instances.

sample example of a `config.pbtxt`

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
For qaic backend configuration, the `backend` parameter should be set to `qaic`.

#### AI 100 specific parameters
Parameters are user-provided key-value pairs which Triton will pass to backend runtime environment as variables and can be used in processing logic of backend.

Parameters are user-provided key-value pairs which Triton will pass to backend runtime environment as variables and can be used in processing logic of backend.

- qpc_path : path for compiled binary of model.(programqpc.bin) (if not provided the server searches for QPC in the model folder)
- device_id : id of AI 100 device on which inference is targeted. device is set 0 (not mandatory as the server auto picks the available device)
- set_size : size of inference queue for runtime,default is set to 20
- no_of_activations : flag to enable multiple activations of a model’s network,default is set to 1

sample example of a `config.pbtxt`

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
To launch Triton server, execute the `tritonserver` binary within Triton Docker with the model repository path.

```bash
/opt/tritonserver/bin/tritonserver --model-repository=</path/to/repository>
```

![image](../../../images/triton_launch_within_container.png)


## Supported Features

- Model Ensemble
- Dynamic Batching
- Auto device-picker
- Support for ARM64
- Support for auto complete configuration
- LLM support for LlamaForCausalLM, AutoModelForCausalLM categories.

### Triton Config_generation tool
Model configuration file `config.pbtxt` is required for each model to run on the Triton server. The `triton_config_generator.py` tool helps to generate a minimal model configuration file if the `programqpc.bin` or `model.onnx` file is provided. The script can be found in "/opt/qti-aic/integrations/triton/release-artifacts/config-generation-script" path inside the container.

The script takes three arguments:

-	--model_repository: Model repository for which config.pbtxt needs to be generated (QAic backend)
-	--all: Generate config.pbtxt for ONNX (used with --model-repository)
-	--model_path: QAic model or ONNX model file path for which model folder needs to be generated.

The `model_repository` argument can be passed, and the script goes through the models and generates `config.pbtxt` for models that do not contain config (the --all option needs to be passed if config needs to be generated for ONNX models) or model path can be provided to generate model folder structure with `config.pbtxt` using random model names.

## Examples
Triton example applications are released as part of the Cloud AI Apps SDK. Inside the Triton Docker container the sample model repositories are available at "/opt/qti-aic/aic-triton-model-repositories/"
- `--model-repository` option can be used to launch the models.
 
### Stable diffusion
1) If we built the Docker with Triton model repo application then the stable diffusion model repo is available at this path: "/opt/qti-aic/aic-triton-model-repositories/ensemble-stable-diffusion".
 
2) To generate a model repo inside a Triton container:

   - Run the `generate_SD_repo.py` script. The script is located at "/opt/qti-aic/integrations/triton/release-artifacts/stable-diffusion-ensemble", which will create a ensemble-stable-diffuison model repo
 
3) Start the Triton server "/opt/tritonserver/bin/tritonserver --model-repository=/path/to/ensemble-stable-diffusion" <br>
   Example: "/opt/tritonserver/bin/tritonserver --model-repository=/opt/qti-aic/aic-triton-model-repositories/ensemble-stable-diffusion"
 
4) Triton server takes about 2 minutes to start on the first go as it needs to compile QPC.
 
5) Run the client_example.py from the same container for testing purpose or client_example.py can also be copied to Triton client container and executed from there.

# Triton LLM
- LLM serving through Triton is enabled using `triton-qaic-backend-python`
- It supports execution of QPC binaries for Causal models, KV Cache models of LlamaForCausalLM, AutoModelForCausalLM categories.
- It supports two modes of server to client response - batch, decoupled (stream). In batch response mode, all of the generated tokens are cached and composed as a single response at the end of decode stage. In decoupled (stream) response mode, each generated token is sent to client as a separate response.
- Currently we include sample configurations for Mistral and Starcoder models.
- Sample client scripts are provided to test kv, causal models in stream-response, batch-response transaction modes.
## Instructions to launch LLM models on Triton server
### Launch Triton server container
```bash
docker run -it --shm-size=4g --rm --privileged --net=host -v /path/to/custom/models/:/path/to/custom/models/ <triton-docker-image-name>:<tag> bash
```
- qefficient virtual environment comprises compatible packages for compilation/execution of a wide range of LLMs. Activate this environment within the container.
```bash
. /opt/qeff-env/bin/activate
```

### Generating a model repository

#### Sample models

Model Folder  | Model Type | Response Type 
--- |--- |---
starcoder_15b | Causal | Batch 
starcoder_decoupled | Causal | Decoupled (Stream) 
mistral_7b | KV cache | Batch 
mistral_decoupled | KV cache | Decoupled (Stream) 

- Pass in the QPC to `generate_llm_model_repo.py` script available at **/opt/qti-aic/integrations/triton/release-artifacts/llm-models/** within Triton container.
```bash
python generate_llm_model_repo.py --model_name mistral_7b --aic_binary_dir <path/to/qpc> --python_backend_dir /opt/qti-aic/integrations/triton/backends/qaic/qaic_backend_python/
```
#### Custom models
- `generate_llm_model_repo.py` script uses a template to auto-generate config for custom models. Configure required parameters such as `use_kv_cache`, `model_name`, decoupled transaction policy through command line options to the script. Choosing `model_type` will configure `use_kv_cache` parameter. If not provided, it will be determined by loading QPC object which may take several minutes for large models. This creates a model folder for <custom_model> in  **/opt/qti-aic/integrations/triton/release-artifacts/llm-models/llm_model_dir**
```bash
python generate_llm_model_repo.py --model_name <custom_model> \
                                  --aic_binary_dir <path/to/qpc> \
                                  --hf_model_name \
                                  --model_type <causal/kv_cache> \
                                  --decoupled
```

### Launch tritonserver and load models
- Pre-requisite: Users need to get access for necessary models from Hugging Face and login with Hugging Face token using **'huggingface-cli login`** before launching the server.
- Launch the Triton server with **llm_model_dir**.
```bash
/opt/tritonserver/bin/tritonserver --model-repository=<path/to/llm_model_dir>
```

### Running the client
#### Launch client container
```bash
docker run -it --rm -v /path/to/unzipped/apps-sdk/integrations/triton/release-artifacts/llm-models/:/llm-models --net=host nvcr.io/nvidia/tritonserver:22.12-py3-sdk bash
```
- Once the server has started you can run example Triton client (client_example_kv.py/client_example_causal.py) provided to submit inference requests to loaded models.
- Decoupled model transaction policy is supported only over gRPC protocol. Therefore, decoupled models (stream response) use gRPC clients whereas batch response mode uses HTTP client as a sample.
```bash

# mistral_decoupled
python /llm-models/tests/stream-response/client_example_kv.py --prompt "My name is"
 
# mistral_decoupled (QPC compiled for batch_size=2)
python /llm-models/tests/stream-response/client_example_kv.py --prompt "My name is|Maroon bells"
 
# mistral_7b
python /llm-models/tests/batch-response/client_example_kv.py --prompt "My name is"
 
# starcoder_decoupled
python /llm-models/tests/stream-response/client_example_causal.py --prompt "Write a python program to print hello world"
 
# starcoder_15b
python /llm-models/tests/batch-response/client_example_causal.py --prompt "Write a python program to print hello world"
```

Note: For batch-response tests, the default network timeout in `client_example_kv.py`, `client_example_causal.py` is configured as 10 min (600 sec), 100 min (6000 sec) respectively.













