# vLLM
vLLM library can be used with a AIC100 backend. This brings continuous batching support, along with other features supported in vLLM.

## Installation  
### Docker container with vLLM support
Please refer to this page for [prerequisites](https://github.qualcomm.com/pages/qranium/cloud-ai-mkdocs/latest/Getting-Started/Installation/Docker/Docker/#setup-and-system-pre-requisistes) prior to building the docker image that includes the vLLM installation

Build the docker image which includes the vLLM installation using the build_image.py script. 

```
cd </path/to/app-sdk>/tools/docker-build/

python3 build_image.py --user_specification_file ./sample_user_specs/user_image_spec_vllm.json --apps_sdk path_to_apps_sdk_zip_file --platform_sdk path_to_platform_sdk_zip_file --tag 1.17.1.8-vllm
```

This should create a docker image with vLLM installed. 

```
ubuntu@host:~# docker image ls
REPOSITORY                                                                      TAG                           IMAGE ID       CREATED         SIZE
qaic-x86_64-ubuntu20-py38-release-qaic_platform-qaic_apps-pybase-pytools-vllm   1.17.1.8                      3e4811ba18ae   3 hours ago     7.05GB

```

Once the docker image is built, please see instructions [here](https://github.qualcomm.com/pages/qranium/cloud-ai-mkdocs/latest/Getting-Started/Installation/Docker/Docker/#command) to launch the container and map the QID devices to the container.

After the container is launced, activate the virtual environment and run a sample inference using the example script provided.

```
source /opt/vllm-env/bin/activate

cd /opt/qti-aic/integrations/vllm/
python examples/offline_inference_qaic.py
```

### Installing from source

vLLM with qaic backend support can be installed by applying a patch on top of the open source vLLM repo

```
# Add user to qaic group to access Cloud AI devices without root
sudo usermod -aG qaic $USER
newgrp qaic

# Create a python virtual enviornment
python3.8 -m venv qaic-vllm-venv

source qaic-vllm-venv/bin/activate

# Install the current release version of QEfficient (vLLLM with qaic support requires QEfficient for model exporting and compilation)
pip install -U pip

pip install git+https://github.com/quic/efficient-transformers@release/v1.17
pip install outlines==0.0.32
pip install ray

# Clone the vLLM repo, and apply the patch for qaic backend support
git clone https://github.com/vllm-project/vllm.git

cd vllm

git checkout bc8ad68455ce41ba672764f4a53df5a87d1dbe99

git apply /opt/qti-aic/integrations/vllm/qaic_vllm.patch

# Set environment variables and install
export VLLM_BUILD_WITH_QAIC=True

pip install -e .

# Use older FastAPI version to avoid pydantic error with OpenAI endpoints
pip install fastapi==0.112.2

# Run a sample inference
python examples/offline_inference_qaic.py
```

## Server Endpoints 
vLLM provides capabilities to start a FastAPI server to run LLM inference. Here is an example to use qaic backend (i.e. use the AI100 cards for inference). Please replace the host name and port number

```
# Start the server
python3 -m vllm.entrypoints.api_server --host <host_name> --port <port_num> --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 --max-model-len 256 --max-num-seq 4 --max-seq_len-to-capture 128 --device qaic --block-size 32 --quantization mxfp6 --kv-cache-dtype mxint8


# Client request
python3 vllm/examples/api_client.py --host <host_name> --port <port_num> --prompt "My name is" --stream
```

Similarly, an OpenAI compatible server can be invoked as follows

```
python3 -m vllm.entrypoints.openai.api_server --host <host_name> --port <port_num> --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 --max-model-len 256 --max-num-seq 4 --max-seq_len-to-capture 128 --device qaic --block-size 32 --quantization mxfp6 --kv-cache-dtype mxint8
```

## Benchmarking
vLLM provides benchmarking scripts to measure serving, latency and throughput performance. Here's an example for serving performance. First, start an OpenAI compatible endpoint using the steps in the previous section.  Please replace the host name and port number.

Download the dataset:
```
wget https://huggingface.co/datasets/anon8231489123/ShareGPT_Vicuna_unfiltered/resolve/main/ShareGPT_V3_unfiltered_cleaned_split.json
```

Start benchmarking:
```
python3 ./vllm/benchmarks/benchmark_serving.py --backend openai --base-url http://<host>:<port> --dataset-name=sharegpt --dataset-path=./ShareGPT_V3_unfiltered_cleaned_split.json --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 --seed 12345
```
