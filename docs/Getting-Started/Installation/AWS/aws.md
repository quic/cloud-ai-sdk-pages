# Getting started on AWS instances 
Qualcomm Cloud AI 100 inference accelerator instances are now available on the AWS Cloud. 

## DL2q  
DL2q is an accelerated computing instance powered by Cloud AI 100 Standard accelerator cards that can be used to cost-efficiently deploy deep learning (DL) inference workloads. 

Details regarding the DL2q instance can be found [here](https://aws.amazon.com/ec2/instance-types/dl2q/). 

The latest AMI from Qualcomm is `“Deep Learning Base Qualcomm AMI (Amazon Linux 2) 1.12.0.88P 1.14.2.0A SDK” ami-0d0b22ddccd54aec0`. Email your 12-digit AWS account number along with the intended use case (CV, NLP, GenAI etc) to `qualcomm_dl2q_ami@qti.qualcomm.com`. Qualcomm will provide access the latest AMI. 

We recommend users to attach 500GB of EBS GP3 storage to the instance to start with (especially for running LLMs).  

???+ note 
      Users of DL2q instance may need to request AWS to increase the vCPU limit to be able to spin up the first DL2q instance from an AWS account. 

SSH to the instance and add user to the `qaic` group to be able to run Cloud AI SDK tools without superuser (sudo) privileges. 
    ```
    sudo usermod -aG qaic $USER
    ```
Close and re-open SSH session for the change to take effect. 

For CV, NLP and diffusion models, users should clone [Cloud AI 100 GitHub repository](https://github.com/quic/cloud-ai-sdk) containing model recipes, code samples and toolchain tutorials on the instance to get started. 

For LLMs, users should clone [QEfficient Transformers Library](https://github.com/quic/efficient-transformers) on the instance to get started. 

## Running your first LLM on DL2Q 
[QEfficient Transformers Library](https://github.com/quic/efficient-transformers) is the easiest way to get your first LLM running on your DL2q instance. 

Install the QEfficient library on your DL2q instance
      ```
      # Login to the Cloud AI 100 Server.
      ssh -X username@hostname

      python3.8 -m venv qeff_env
      source qeff_env/bin/activate

      # Clone the QEfficient Repo (use the 1.14 branch to match the SDK on the AMI)
      git clone -b release/v1.14 --single-branch https://github.com/quic/efficient-transformers.git

      # Install the qefficient-library in Host machine (Used for CLI APIs) (Until we have docker integrated in Apps SDK)
      pip install -e .
      ```
???+ note 
      Some models (Llama, Mistral etc.) require custom ops which will need to be compiled prior to model compilation. Refer [here](https://github.com/quic/efficient-transformers/blob/main/QEfficient/customop/README.md) for instructions on compiling the custom ops. Requires CMake 3+.
      
Run your first LLM
      ```
      python -m QEfficient.cloud.infer --model_name gpt2 --batch_size 1 --prompt_len 32 --ctx_len 128 --mxfp6 --num_cores 16 --device_group [0] --prompt "My name is" --mos 1 --aic_enable_depth_first
      ```


