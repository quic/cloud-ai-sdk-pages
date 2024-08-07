# Pre-requisites 

## Message Signaled Interrupts

The QAic Kernel Driver for Linux requires 32 message signaled interrupts (MSI) for best performance. QAic kernel driver does support single MSI configuration but is not recommended. On x86-based host systems, Intel VT-d or IOMMU features must be enabled in the BIOS to enable the required number of MSIs.

For host systems using Intel chipsets, ensure that Intel Virtualization (VT-d) is enabled in the BIOS.
For host systems using AMD chipsets, ensure that the IOMMU feature is enabled in the BIOS.

## Ubuntu on x86-64

- Install Ubuntu 18.04, 20.04, 22.04 or 24.04 with default kernel and minimum options
-	Install the following packages:

???+ note 
    Python 3.8 64-bit is the only supported Python version by Cloud AI SDK.
    
  ```bash
  sudo apt-get update && sudo apt-get install -y software-properties-common 
  sudo add-apt-repository ppa:ubuntu-toolchain-r/test -y
  sudo add-apt-repository ppa:deadsnakes/ppa -y
  sudo apt-get update && sudo apt-get install -y build-essential git less vim-tiny nano libpci-dev libudev-dev libatomic1 python3-pip python3-setuptools python3-wheel python3.8 python3.8-dev python3.8-venv
  sudo apt-get update && sudo apt-get install -y unzip zip wget ca-certificates sudo pciutils libglib2.0-dev libssl-dev snap snapd openssh-server pkg-config clang-format libpng-dev gpg
  sudo apt-get install -y libstdc++6
  sudo apt-get install -y libncurses5-dev
  python3.8 -m pip install --upgrade pip
  python3.8 -m pip install wheel numpy opencv-python onnx
  ```

#### Add/update environment variables:

  ```bash
  export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/opt/qti-aic/dev/lib/x86_64"
  export PATH="/usr/local/bin:${PATH}"
  export PATH="${PATH}:/opt/qti-aic/tools:/opt/qti-aic/exec:/opt/qti-aic/scripts"
  export QAIC_EXAMPLES="/opt/qti-aic/examples"
  export QAIC_APPS="/opt/qti-aic/examples/apps"
  export QAIC_LIB="/opt/qti-aic/dev/lib/x86_64/libQAic.so"
  export QAIC_COMPILER_LIB="/opt/qti-aic/dev/lib/x86_64/libQAicCompiler.so"
  ```

## CentOS 7 / RHEL on x86-64

- CentOS 7 / RHEL with default kernel installed
-	Run all commands as 'root'.
- Install the dkms package for the kernel.
    - CentOS 7 – yum -y install dkms
    - RHEL – Refer to this [link](https://docs.fedoraproject.org/en-US/epel/#How_can_I_use_these_extra_packages.3F) on how to install EPEL packages
- Install the appropriate linux-headers package for the kernel.
- Install the following packages:
  
???+ note
    Python 3.8 64-bit is the only supported Python version by Cloud AI SDK.

  ```bash
  yum update –y
  [Optional] update-ca-trust force-enable
  yum install -y epel-release centos-release-scl centos-release-scl-rh
  yum install -y devtoolset-9 automake
  yum install -y git vim-minimal nano cmake pciutils-devel rpm-build systemd-devel libudev-devel python3-pip python3-setuptools python3-wheel python3-devel rh-python38-python
  yum install -y unzip zip wget ca-certificates findutils gpg openssh-server scl-utils sudo tar which pciutils git less libatomic ncurses-devel libasan glib2-devel mesa-libGL libpng-devel

  # Add the following lines to end of /etc/bashrc:
  source /opt/rh/devtoolset-9/enable
  source /opt/rh/rh-python38/enable
  
  # Start new bash shell then:
  wget https://bootstrap.pypa.io/get-pip.py -O /tmp/get-pip.py
  python3.8 /tmp/get-pip.py
  python3.8 -m pip install wheel numpy opencv-python onnx
  ```

#### Add/update environment variables:

  ```bash
  export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/opt/qti-aic/dev/lib/x86_64"
  export PATH="/usr/local/bin:${PATH}"
  export PATH="${PATH}:/opt/qti-aic/tools:/opt/qti-aic/exec:/opt/qti-aic/scripts"
  export QAIC_EXAMPLES="/opt/qti-aic/examples"
  export QAIC_APPS="/opt/qti-aic/examples/apps"
  export QAIC_LIB="/opt/qti-aic/dev/lib/x86_64/libQAic.so"
  export QAIC_COMPILER_LIB="/opt/qti-aic/dev/lib/x86_64/libQAicCompiler.so"

  ```

