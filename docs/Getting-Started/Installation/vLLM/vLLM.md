# vLLM
This guide demonstrates how to add AI 100 backend support to the [vLLM](https://github.com/vllm-project/vllm) open-source library, which simplifies the creation of OpenAI-compatible web endpoints and provides features like continuous batching and other optimizations for LLM inference and serving.

## Installation  
### Docker container with vLLM support
Refer to this page for [prerequisites](../Docker/Docker.md#setup-and-system-pre-requisistes) prior to building the docker image that includes the vLLM installation

Build the docker image which includes the vLLM installation using the build_image.py script. 

```
cd </path/to/app-sdk>/common/tools/docker-build/

python3 build_image.py --user_specification_file ./sample_user_specs/user_image_spec_vllm.json --apps_sdk path_to_apps_sdk_zip_file --platform_sdk path_to_platform_sdk_zip_file --tag 1.18.2.0-vllm
```

This should create a docker image with vLLM installed. 

```
ubuntu@host:~# docker image ls
REPOSITORY                                                                      TAG                           IMAGE ID       CREATED         SIZE
qaic-x86_64-ubuntu20-py38-release-qaic_platform-qaic_apps-pybase-pytools-vllm   1.18.2.0                      3e4811ba18ae   3 hours ago     7.05GB

```

Once the docker image is built, refer to instructions [here](../Docker/Docker.md#command) to launch the container and map the QID devices to the container.

After the container is launched, activate the virtual environment and run a sample inference using the example script provided.

```
source /opt/vllm-env/bin/activate

cd /opt/qti-aic/integrations/vllm/
python examples/offline_inference_qaic.py
```

### Installing from source

vLLM with qaic backend support can be installed by applying a patch on top of the open source vLLM repo.

```
# Add user to qaic group to access Cloud AI devices without root
sudo usermod -aG qaic $USER
newgrp qaic

# Create a python virtual enviornment
python3.8 -m venv qaic-vllm-venv

source qaic-vllm-venv/bin/activate

# Install the current release version of QEfficient (vLL/M with qaic support requires QEfficient for model exporting and compilation)
pip install -U pip

pip install git+https://github.com/quic/efficient-transformers@release/v1.18

# Clone the vLLM repo, and apply the patch for qaic backend support
git clone https://github.com/vllm-project/vllm.git

cd vllm

git checkout v0.6.0

git apply /opt/qti-aic/integrations/vllm/qaic_vllm.patch

# Set environment variables and install
export VLLM_TARGET_DEVICE="qaic"

pip install -e .

# Run a sample inference
python examples/offline_inference_qaic.py
```

## Server Endpoints 
vLLM provides capabilities to start a FastAPI server to run LLM inference. Here is an example to use qaic backend (i.e. use the AI 100 cards for inference).

```
# Need to increase max open files to serve multiple requests
ulimit -n 1048576

# Start the server
python3 -m vllm.entrypoints.api_server --host 127.0.0.1 --port 8000 --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 --max-model-len 256 --max-num-seq 16 --max-seq_len-to-capture 128 --device qaic --block-size 32 --quantization mxfp6 --kv-cache-dtype mxint8

# Client request
python3 examples/api_client.py --host 127.0.0.1 --port 8000 --prompt "My name is" --stream
```

Similarly, an OpenAI compatible server can be invoked as follows

```
# Need to increase max open files to serve multiple requests
ulimit -n 1048576

# Start the server
python3 -m vllm.entrypoints.openai.api_server --host 127.0.0.1 --port 8000 --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 --max-model-len 256 --max-num-seq 16 --max-seq_len-to-capture 128 --device qaic --block-size 32 --quantization mxfp6 --kv-cache-dtype mxint8

# Client request
python3 examples/openai_chat_completion_client.py
```

## Benchmarking
vLLM provides benchmarking scripts to measure serving, latency and throughput performance. Here's an example for serving performance. First, start an OpenAI compatible endpoint using the steps in the previous section.

Download the dataset:
```
wget https://huggingface.co/datasets/anon8231489123/ShareGPT_Vicuna_unfiltered/resolve/main/ShareGPT_V3_unfiltered_cleaned_split.json
```

Start benchmarking the OpenAI endpoint
```
# Start benchmarking
python3 benchmarks/benchmark_serving.py --backend openai --base-url http://127.0.0.1:8000 --dataset-name=sharegpt --dataset-path=./ShareGPT_V3_unfiltered_cleaned_split.json --sharegpt-max-input-len 128 --sharegpt-max-model-len 256 --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 --seed 12345
```
