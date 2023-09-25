# Installation Checklist

First, check for [supported operating environments](../index.md#supported-operating-systems-hypervisors-and-platforms).

## Local Server Installation

1. Set up Linux server/workstation environment ([Pre-requisites](../Pre-requisites/pre-requisites.md))

    - Enable 32 message signaled interrupts (MSI) in UEFI/BIOS setup ([MSI instructions](../Pre-requisites/pre-requisites.md#message-signaled-interrupts))
    
2. Install Platform and Apps SDKs ([Cloud AI SDK](../Cloud-AI-SDK/Cloud-AI-SDK.md))

  -	**Tip:** To run Platform SDK tools without root (sudo) privilege, add yourself to the qaic group.  
    ```
    sudo usermod -aG qaic $USER
    ```
    - Log in again for the changes to take effect, or run ‘newgrp qaic’.

## Verify Card Health/Function
1. Check PCIe enumeration:
  ```
  $ lspci | grep Qualcomm
  01:00.0 Unassigned class [ff00]: Qualcomm Device a100
  ```
    - A PCIe enumeration failure may indicate an unsecure connection. Try re-inserting the card, or troubleshoot with a different PCIe card.

  - Check device nodes:
  ```
  ls /dev/mhi*
  /dev/mhi0_QAIC_DIAG  /dev/mhi0_QAIC_TIMESYNC /dev/mhi0_QAIC_QDSS /dev/mhi1_QAIC_DIAG
  ```
    - For every card, QAIC_DIAG, QAIC_TIMESYNC, QAIC_QDSS and QAIC_DIAG nodes are created. 
    - If mhi* folders do not exist, double-check the [MSI settings](../Pre-requisites/pre-requisites.md#message-signaled-interrupts) in UEFI/BIOS setup

  - Check card health and status with [qaic-util](../../System-Management/system-management.md)
  ```
  sudo /opt/qti-aic/tools/qaic-util -q | grep -e Status -e QID

      QID 0
        Status:Ready
      QID 1
        Status:Ready
      QID 2
        Status:Ready
      QID 3
        Status:Ready
  ```
    - ‘Status: Ready’ indicates card(s) is in good health. Shown here are a system with 4 Cloud AI SoCs (Each card may have one or more SoCs based on the SKU).
    - ‘Status: Error’ indicates the respective card(s) has not booted up completely. 
        - Card could still be booting. Wait for a few mins and retry above command.
        - Card error could be due to several reasons - unsupported OS/Platform, secure boot etc. 

  -	Run a test inference workload with qaic-runner CLI to sanity check HW/SW function.  Example:
  ```
  sudo /opt/qti-aic/exec/qaic-runner -t /opt/qti-aic/test-data/aic100/v2/1nsp/1nsp-conv-hmx/ -a 14 -n 5000 -d 0

  Input file: /opt/qti-aic/test-data/aic100/v2/1nsp/1nsp-conv-hmx//user_idx0_in.bin
  ---- Stats ----
  InferenceCnt 5000 TotalDuration 17598us BatchSize 1 Inf/Sec 284123.196
  ```
    - `-d` specify QID to run the workload on.
    - `-a` edit value based on no of NSPs in specified QID.
  
    A positive Inf/Sec indicates HW/SW is functioning correctly. The Inf/Sec reported depends on `-a` specified in the command. 
  

## Virtual Machine Setup

1. Configure PCIe pass-through for the supported [Hypervisor](../Hypervisors/hypervisor.md)
2. Configure [message signaled interrupts](../Pre-requisites/pre-requisites.md#message-signaled-interrupts) in UEFI/BIOS setup and [Hypervisor](../Hypervisors/hypervisor.md)
3. Install and configure [Hypervisor](../Hypervisors/hypervisor.md)
4. Configure and launch Virtual Machine instance with one of the supported [Operating Systems](../index.md#supported-operating-systems-hypervisors-and-platforms) and [Pre-requisites](../Pre-requisites/pre-requisites.md)
5. Install [Cloud AI SDKs](../Cloud-AI-SDK/Cloud-AI-SDK.md/)
6. Check [Card Health/Function](#verify-card-healthfunction)

## Docker Setup

1. Install Platform SDK on host system or virtual machine ([Cloud AI SDK](../Cloud-AI-SDK/Cloud-AI-SDK.md))
2. Build Docker image and start the container ([Docker](../Docker/Docker.md))

