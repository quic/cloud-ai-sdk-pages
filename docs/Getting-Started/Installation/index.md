# Introduction  
Developers can access Qualcomm Cloud AI accelerators on Cloud or by purchasing servers equipped with Qualcomm Cloud AI accelerators. 

### Cloud Instances 
Cloud AI 100 cards are now available at 2 Cloud service providers - Amazon Web Services (AWS) and Cirrascale Cloud Services. The AI 100 accelerator SKUs and instance configurations offered at these providers can vary. 

- Refer to [**Getting Started on AWS**](AWS/aws.md) for more information on the instances available at AWS.  
- [**Cirrascale Cloud Services**](https://cirrascale.com/solutions-qualcomm-cloud-ai100.php) have multiple configurations (from 1 to 8 Pro AI 100 accelerators per instance) for the user to choose from. 

???+ note 
      **Developers using cloud instances can skip the rest of the installation section. Click here to go to the next section, [Inference Workflow](../Inference-Workflow/index.md)**

### On-Premise Servers
Developers with on-premise servers need to work with system administators to ensure Cloud AI SDKs are installed and verified properly. It is recommended for developers and system admins to go through the installation section in its entirety. 

## Installation 

The Platform SDK (x86-64 and ARM64) and Apps SDK (x86-64 only) are targeted for Linux-based platforms. The SDKs can be installed natively on Linux operating systems. Container and orchestration are also supported through Docker and Kubernetes. Virtual machines, including KVM, ESXi, and Hyper-V, are also supported. This section covers:

  - Installation of the SDKs across multiple Linux distributions
  - Building a docker image with the SDKs and third-party packages for a seamless execution of QAic inference tools/workflow
  - Setting up KVM, ESXi, and Hyper-V, and installation of SDKs

### Compilation and Execution modes 
Apps and Platform SDKs enable just-in-time(JIT) or ahead-of-time(AOT) compilation and execution on x86-64 platforms while only AOT compilation/execution is supported on ARM64. 

In JIT mode, compilation and execution are tightly coupled and require Apps and Platform SDKs to be installed on the same system/VM.

In AOT mode, compilation and execution are decoupled. Networks can be compiled ahead-of-time on x86-64 (with Apps SDK only) and the compiled networks can be deployed  on x86-64 or ARM64 with Platform SDK.

Both JIT and AOT are supported on x86-64 when Apps and Platform SDK are installed on the same server/VM. 

### Supported Operating Systems, Hypervisors, and Platforms 
The Cloud AI Platform SDK is compatible with the following operating systems (OS) and platforms.

#### Operating Systems

| **Operating systems**        | **Kernel**                          | **x86-64** | **ARM64** |
| ---------------------------- | ----------------------------------- | ---------- | -------------------------- |
| CentOS Linux 7               | Linux Kernel 5.4.1                  | ✔       | ✗                          |
| CentOS Linux 8               | Linux Kernel 4.19                   | ✗       | ✔                          |
| Ubuntu 18.04                 | Linux Kernel 5.4.1 / Default Kernel | ✔       | ✔ (Note1)                        |
| Ubuntu 20.04                 | Default Kernel                      | ✔       | ✗                          |
| Ubuntu 22.04                 | Default Kernel                      | ✔       | ✗                          |
| Red Hat Enterprise Linux 7.9 | Default Kernel                      | ✔       | ✗                          |
| Red Hat Enterprise Linux 8.3 | Default Kernel                      | ✔       | ✗                          |
| Red Hat Enterprise Linux 8.4 | Default Kernel                      | ✔       | ✗                          |
| Red Hat Enterprise Linux 8.6 | Default Kernel                      | ✔       | ✗                          |
| Red Hat Enterprise Linux 8.7 | Default Kernel                      | ✔       | ✗                          |
| Red Hat Enterprise Linux 9.0 | Default Kernel                      | ✔       | ✗                          |
| Red Hat Enterprise Linux 9.1 | Default Kernel                      | ✔       | ✗                          |
| `Note1`: Supported on certain ARM64-based Qualcomm platforms. |
| `Note2`: Arm is a trademark of Arm Limited (or its subsidiaries) in the US and/or elsewhere. |
| `Note3`: Apps SDK is available only for x86-64 platforms. |

#### Hypervisors
Cloud AI only supports PCIe passthrough to a virtual machine. This means that the virtual machine completely owns the Cloud AI device. A single Cloud AI device cannot be shared between virtual machines or between a virtual machine and the native host. 

| **Hypervisor**                                                                                                                                           | **x86-64** | **ARM64** |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- | ------- |
| KVM                                                                                                                                                      | ✔       | ✗       |
| Hyper-V                                                                                                                                                  | ✔       | ✗       |
| ESXi                                                                                                                                                     | ✔       | ✗       |
| `Note` Cloud AI SDKs are not required to be installed on the hypervisor. All the Cloud AI software/SDKs are installed on the guest VMs. |


