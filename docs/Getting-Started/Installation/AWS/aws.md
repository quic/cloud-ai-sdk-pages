# Getting started on AWS instances 
Qualcomm Cloud AI 100 inference accelerator instances are now available on the AWS Cloud. 

## DL2q  
DL2q is an accelerated computing instance powered by Cloud AI 100 Standard accelerator cards that can be used to cost-efficiently deploy deep learning (DL) inference workloads. 

Details regarding the DL2q instance can be found [here](https://aws.amazon.com/ec2/instance-types/dl2q/). 

To get started quickly, customers can load one of many Deep Learning Amazon Machine Images (DLAMI) which come pre-packaged with Cloud AI Platform and Apps SDK. Search `Qualcomm AMI` for a list of AMIs that can be loaded on the DL2q instance. 

We recommend users to attach 200GB of EBS GP3 storage to the instance to start with.  

???+ note 
      Users of DL2q instance may need to request AWS to increase the vCPU limit to be able to spin up the first DL2q instance from an AWS account. 

SSH to the instance and add user to the `qaic` group to be able to run Cloud AI SDK tools without superuser (sudo) privileges. 
    ```
    sudo usermod -aG qaic $USER
    ```
Close and re-open SSH session for the change to take effect. 

Users can clone the [Cloud AI 100 GitHub repository](https://github.com/quic/cloud-ai-sdk) containing model recipes, code samples and toolchain tutorials on the instance to get started. 



