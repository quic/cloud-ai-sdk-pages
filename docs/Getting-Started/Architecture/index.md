# Architecture 

## Cloud AI Platforms
Cloud AI Platforms/cards contain one or more Cloud AI 100 SoCs per card and are designed to operate at a specific TDP. The cards interface to the host via PCIe and to the BMC via SMBus/I2C.  

Cloud AI 100 [Product brief](https://www.qualcomm.com/content/dam/qcomm-martech/dm-assets/documents/Prod-Brief-QCOM-Cloud-AI-100.pdf) provides details on the form factors and SKUs. 

## Cloud AI 100 SoC
Cloud AI100 SOC’s multi-core architecture, shown in figure below, is purpose-built for deep learning inference in the Cloud. 
![Cloud AI 100 SoC Architecture](../../images/SoC.png)

The SOC is composed of 16 seventh-generation AI cores delivering 400+ Int8 TOPs and 200+ FP16 TOPs of compute performance, with 144 MB of on-chip memory for data storage. 

The on-chip memory sub-system is connected to an external LPDDR4X memory subsystem, comprising of 4 channels of 64b width(4 x 64b), providing 136 GB/s of memory bandwidth and up to 32 GB of memory capacity. 

The SoC also provides 8 lanes of PCIe Gen4 IO interfaces (PCIe-gen4 x 8) to connect to the host CPU complex and other peripherals. 

The AI cores and all internal subsystems are connected by three advanced NoCs that deliver 186 GB/s of data bandwidth and support multicast and AI core synchronization. The Compute NoC connects the AI cores and PCIe, the Memory NoC connects the AI cores to the DDR memory, and the Configuration NoC is used for boot and hardware configuration. 

The SoC is also equipped with a sophisticated power management system, optimizing both transient and peak power consumption. The SOC also implements thermal detection and control. 

The SOC implements several cloud-readiness security features, including ECC, secure boot and DDR memory zero-out on reset.

## AI Core 

The seventh-generation AI Core, leveraging over a decade of Qualcomm AI research, is shown in Figure below. The AI Core implements the architecture principle of separation-of-concerns, with three types of compute units for tensor, vector and scalar operations. 

![AI Core](../../images/AI_Core.png) 

The tensor unit implements two 2D MAC arrays—8K for 8b integer, and 4K for 16b floating point, with 125+ instructions conducive for linear algebra, providing throughput of 8192/4096 MAC operations per clock for 8/16b integer/floating point operations respectively.  

The vector unit implements over 700+ rich instructions for AI, content verification, and image processing supporting 8/16b integer and 16/32b precision floating point, providing throughput of 512/256 MAC operations per clock for 8/16b integer/floating point operations respectively. 

The scalar processor is a 4-way VLIW (very large instruction word) machine supporting six hardware threads, each with a local scalar register file, instruction and data caches, and support for 8/16b integer and 16/32b floating point operations—a rich instruction set of over 1800 instructions that provide flexibility in compute operations. 

An 8 MB Vector Tightly Coupled Memory (VTCM) provides scratch-pad data storage for both the vector unit as well as the tensor unit. The 1 MB L2 cache in the core is shared by all three compute units (scalar, vector, and tensor), and it implements hardware for the intelligent prefetching of data and for advanced DMA operations. 
