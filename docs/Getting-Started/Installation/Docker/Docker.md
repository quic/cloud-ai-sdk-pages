# Introduction 
Docker is a product that allows users to build, test, and deploy applications through software containers. Docker for Cloud AI 100 packages the Platform SDK, Apps SDK (x86-64 only), libraries, system tools, etc., which enables the user to navigate the inference workflow seamlessly. 


???+ note 
    Docker containers require the Cloud AI device drivers to communicate with the devices. Install the Platform SDK on the host bare metal OS or VM.

The Docker scripts are in the Apps SDK in the `tools/docker-build` folder. The scripts to build a QAic Docker image are composed of the following structure.
```
  ├── build_image.sh
  ├── caffe
  │   ├── detection-output.patch
  │   ├── Dockerfile.ubuntu18
  │   ├── makefile.config.patch
  │   └── mkldnn.patch
  ├── config
  │   ├── aarch64
  │   │   ├── Dockerfile.centos8
  |   |   ├── Dockerfile.triton
  │   │   └── Dockerfile.ubuntu18
  │   ├── qaic_pytools
  │   │   └── pytools.dockerfile
  │   ├── qms_agent
  │   │   └── qms_agent.dockerfile
  │   └── x86_64
  │   |   ├── Dockerfile.centos8
  │   |   ├── Dockerfile.triton
  │   |   ├── Dockerfile.ubuntu18
  |   |   ├── Dockerfile.centos7
  |   |   ├── Dockerfile.ubuntu20
  |   |   └── Dockerfile.ubuntu22
  |   └── qinf
  |   |   └── qinf.dockerfile
  |   └── aimet
  |       └── aimet.dockerfile
  ├── README.md
```

## Build the Docker image 
Download Apps SDK and Platform SDK from Qualcomm site. Refer to [Platform SDK Download](../Cloud-AI-SDK/Cloud-AI-SDK.md#platform-sdk) and [Apps SDK Download](../Cloud-AI-SDK/Cloud-AI-SDK.md#apps-sdk). Unzip Apps SDK and the build scripts are located under `/tools/docker-build/`

```
unzip qaic-apps-1.10.0.193.zip
cd qaic-apps-1.10.0.193/tools/docker-build/
```

QAic docker build script is used to create new docker image with Cloud AI 100 Apps SDK and Cloud AI 100 Platform SDK with the supported OS.

The command to build the docker image is:

```bash
build_image.sh [ --mirror (Optional) <docker-registry-mirror-location> ] [ --apps-sdk (Optional) <apps-sdk-zip-file-path> ] [ --platform-sdk <platform-sdk-zip-file-path> ] [ --install-pkgs <install-pkgs-zip-file-path> ] [ --tag <tag  for  the  created  image> ][ --os <centos8/centos7/ubuntu18> ] [--arch (Optional) <aarch64/x86_64>] [--install-qaic-pytools] [--install-caffe] [--install-aimet] [--install-qms-agent] [--install-python-sdk]
```

Usage: 
```
apps-sdk:	specifies path to the zip file of apps sdk (Optional argument)
platform-sdk:	specifies path to the zip file of platform sdk
install-pkgs:	specifies path to the zip file of install-pkgs (Required for aarch64 based systems)
tag:		specifies the tag name to build the image with. (Default: “latest”).
os:		centos8/centos7/ubuntu18. (Default: centos8)
arch:		aarch64/x86_64. (Default: host arch)
install-qaic-pytools: Option to install qaic-pytools
install-caffe:	whether to install caffe along with pytools
```

For Example:
```bash
./build_image.sh --apps-sdk ~/qaic-apps-1.10.0.193.zip --platform-sdk ~/qaic-platform-sdk-1.10.0.193.zip --tag "1.10.0.193" --os ubuntu20 --install-qaic-pytools
```

To check the docker image created with above script: <br>
```
$ docker images
   REPOSITORY      TAG      IMAGE ID        CREATED        SIZE 
   qaic-ubuntu20-x86_64&nbsp 1.10.0.193  64a78fa17164  About a minute ago  13GB
```


## Create the container 
Create the container using the docker image created above:

```
$ docker run -dit --name <container name> --device=<AIC device that want to pass to the container> <docker image name>:<tag>
```

For example: 
```
$ docker run -dit --name qaic-ubuntu20-x86_64 --device=/dev/qaic_aic100_0 qaic-ubuntu20-x86_64:1.10.0.193
```
To pass more than one Cloud AI device, use `--device` as many times as the number of devices to be passed to the container. 

For example: 
```
$ docker run -dit --name qaic-ubuntu20-x86_64 --device=/dev/qaic_aic100_0 --device=/dev/qaic_aic100_1 qaic-ubuntu20-x86_64:1.10.0.193
```
 
Check container that got created.
```
$ docker ps

CONTAINER ID    IMAGE                            COMMAND      CREATED           STATUS            PORTS     NAMES    
ae6a9e34c519    qaic-ubuntu20-x86_64:1.10.0.193  "/bin/bash"  8 seconds ago     Up 6 seconds                qaic-ubuntu20-x86_64
```

## Execute the container 

Execute the container created in the above step using its container ID: 

```
$ docker exec -it <container ID> /bin/bash
```
For example:

```
$ docker exec -it ae6a9e34c519 /bin/bash
```
This will bring the bash shell within the container.

Query the Cloud AI devices from the container: <br>
```
$ /opt/qti-aic/tools/qaic-util -q
```

This should show only one device “QID 0” in a ready state as we only passed one device to the container.


## Build Docker for aarch64 from x86_64 
When passing an --arch option where the host os is not the same as the requested architecture, <br>
Setup the host for docker multiarch using qemu.

- Ubuntu Host

  ```
	[user@localhost qaic-docker]$ sudo apt install qemu binfmt-support qemu-user-static
	[user@localhost qaic-docker]$ docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
	// Check the setup
	[user@localhost qaic-docker]$ docker run --rm -t --platform=linux/arm64 <image name> uname -m
	The above should give the architecture as aarch64
  ```
  
- CentOS Host

  ```
	[user@localhost qaic-docker]$ sudo apt install qemu qemu-kvm 
	[user@localhost qaic-docker]$ docker run --rm --privileged multiarch/qemu-user-static --reset -p yes 
	// Check the setup 
	[user@localhost qaic-docker]$ docker run --rm -t --platform=linux/arm64 <image name> uname -m 
	The above should give the architecture as aarch64
  ```
