# Installation - Cloud AI SDK
## Platform SDK 

- The Platform SDK is available at [https://www.qualcomm.com/products/technology/processors/cloud-artificial-intelligence/cloud-ai-100#Software](https://www.qualcomm.com/products/technology/processors/cloud-artificial-intelligence/cloud-ai-100#Software)
- Log in with your Qualcomm ID to access the SDK. First time users need to register. 
- The latest Platform SDKs are available under "Software Packages". 
    - The Platform SDK is available in four different packages for the different combinations of x86-64/ARM64 and deb/rpm: `x86-deb, x86-rpm, aarch64-deb, and aarch64-rpm`
- On the host machine, log in as root or use `sudo` to have the right permissions to complete installation 
- Copy the Platform SDK downloaded from the Qualcomm Portal to the host machine:
    - For networked x86-64 or ARM64 host:
        - Use scp, rsync, or samba to copy the Platform SDK zip file to the host machine
        - Log in to the host machine (ssh or local terminal)
        - Unzip the downloaded zip file to a working directory
        - `cd` to the working directory
    - For ARM64 hosts that support Google Android Debug Bridge (ADB):
        ```
        - adb push <Platform SDK zip file> /data
        - adb shell
        - cd /data
        - Unzip the downloaded zip file to a working directory
        - cd to the working directory
        ```
  
???+ note 
      Extract the downloaded zip file until `qaic-platform-sdk-<x86_64/aarch64>-<deb/rpm>-<SDK version>.zip` is available. Confirm the architecture and installation linux package format works for your setup.  
  
  The Platform SDK is composed of the following tree structure. Users will see rpm or deb based on the SDK package:
      ```
      ├── common
      │   ├── sectools 
      ├── <architecture - x86_64 or aarch64>  
      │   ├── rpm 
      │   │   ├── rpm 
      │   │   │   ├── qaic-fw-<version>.el7.x86_64.rpm 
      │   │   │   ├── qaic-kmd-<version>.el7.x86_64.rpm 
      │   │   │   └── qaic-rt-<version>.el7.x86_64.rpm   
      │   │   ├── rpm-docker 
      │   │   │   └── qaic-rt-docker-<version>.el7.x86_64.rpm  
      │   │   ├── install.sh 
      │   │   ├── Notice.txt 
      │   │   └── uninstall.sh 
      │   ├── deb
      │   │   ├── deb 
      │   │   │   ├── qaic-fw_<version>.deb 
      │   │   │   ├── qaic-kmd_<version>-devel_amd64.deb 
      │   │   │   └── qaic-rt_<version>_amd64.debm  
      │   │   ├── install.sh 
      │   │   ├── Notice.txt 
      │   │   └── uninstall.sh 
      │   ├── test_suite   
      │   │   ├── pcietool   
      └── └── └── powerstress    
      ```

  Uninstall existing Platform SDK
    ```bash
    sudo ./uninstall.sh
    sync
    ```  

  Run the install.sh script as root or with sudo to install with root permissions. Installation may take up to 30 mins depending on the number of Cloud AI cards in the server/VM. Cloud AI cards undergo resets several times during the installation. 
    ```bash
    cd <architecture>/<deb/rpm>
    # For Hybrid boot cards (PCIe CEM form factor cards), run 
    sudo ./install.sh --auto_upgrade_sbl --ecc enable
    # For VM on ESXi hypervisor, run 
    sudo ./install.sh --auto_upgrade_sbl --datapath_polling –-ecc enable

    # For Flashless boot cards, run 
    sudo ./install.sh –-ecc enable
    # For VM on ESXi hypervisor, run 
    sudo ./install.sh --datapath_polling –-ecc enable
    ```

  On successful installation of the platform SDK, the contents shown below are stored in /opt/qti-aic:
    ```
    dev  drivers  examples  exec  firmware  services  test-data  tools
    ```
  
  Check Platform SDK version using 
    ```
    cat /opt/qti-aic/versions/platform.xml
    ```
  Add user to the qaic group to allow command-line tools to run without sudo:
    ```
    sudo usermod -a -G qaic $USER
    ```

???+ warning
      Adding a user to the qaic group grants root-level privileges.  

### Verify card operation 
  Refer to [Verify Card Operation](../Checklist/checklist.md#verify-card-operation)
  
## Apps SDK 
The Apps SDK is only available for x86-64 Linux-based hosts. For ARM64-based Qualcomm platforms, models are first compiled on x86 with the Apps SDK. The compiled binary (QPC) is transferred to the ARM64 host for loading and execution by the Platform SDK on Cloud AI hardware.

- The Apps SDK is available at [https://www.qualcomm.com/products/technology/processors/cloud-artificial-intelligence/cloud-ai-100#Software](https://www.qualcomm.com/products/technology/processors/cloud-artificial-intelligence/cloud-ai-100#Software)
- Log in with your Qualcomm ID to access the SDK. First time users need to register. 
- The latest Apps SDK is available under "Software Packages". 
- Download the Apps SDK. Unzip until you see the below folder structure. 

  ```
    <Apps-SDK> 
    ├── dev  
    │   ├── hexagon_tools  
    │   ├── inc 
    │   └── lib 
    │       └── x86_64 
    │           └── apps 
    ├── examples 
    │   ├── apps 
    │   └── scripts 
    ├── exec 
    │   └── qaic-exec 
    ├── tools
    │   ├── docker-build
    │   ├── package-generator
    │   ├── qaic-pytools
    │   ├── qaic-version-util
    │   ├── rcnn-exporter
    │   └── smart-nms
    ├── install.sh  
    ├── integrations  
    │   └── qaic_onnxrt 
    │   └── triton 
    ├── Notice.txt  
    ├── scripts 
    │   └── qaic-model-configurator
    ├── versions 
    │   └── apps.xml  
    └── uninstall.sh
  ```
### Install Apps SDK 
  - Uninstall existing Apps SDK<br>
    ```sudo ./uninstall.sh```
  - Run the install.sh script as root or with sudo to install with root permissions.<br>
    ```sudo ./install.sh --enable-qaic-pytools ```
  - On successful installation of the Apps SDK, the contents are stored to the /opt/qti-aic path under the dev and exec directories:<br>
    ```dev exec integrations scripts```
  - Check the Apps SDK version with the following command <br>
    ```cat /opt/qti-aic/versions/apps.xml```
  - Apply chmod commands 
    
    ```
    sudo chmod a+x /opt/qti-aic/dev/hexagon_tools/bin/*
    sudo chmod a+x /opt/qti-aic/exec/*
    ```
  
## Activate `qaic-env`

`qaic-env` virtual environment is created during the installation of the APPS SDK. Developers are expected to activate `qaic-env` for seamless interaction with Cloud AI devices/SDKs.

- For bash shell, 
  ```
  source /opt/qti-aic/dev/python/qaic-env/bin/activate
  ```

- For csh shell, 
  ```
  source /opt/qti-aic/dev/python/qaic-env/bin/activate.csh
  ```

To deactivate the virtual environment, type `deactivate` in any directory.    
  
  





