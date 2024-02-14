# How to Install NVIDIA cuDNN on Ubuntu 22.04

## Introduction

This guide shows how to install the CUDA Deep Learning Neural Network (cuDNN) package on Ubuntu 22.04. 

### What is cuDNN 

cuDNN is an NVIDIA library to enable GPU-based computations for deep neural networks. cuDNN allows developers to directly invoke functions for training and running inference on neural networks without having to write the base functions to do so. Neural networks are the building blocks of most modern deep learning applications, such as generative AI models.

### Prerequisites

Before starting the cuDNN installation process, you need to: 

 * [Generate an SSH key-pair](https://www.vultr.com/docs/how-do-i-generate-ssh-keys/) and [add it to your Vultr account](https://www.vultr.com/docs/deploy-a-new-server-with-an-ssh-key/#Add_an_SSH_key_to_your_Vultr_Account), if you haven't done so.
 * Deploy a new Vultr VPS or dedicated server with at least 2 GB GPU RAM and running Ubuntu 22.04 LTS
 * [Connect to the server over SSH](https://www.vultr.com/docs/connect-to-a-server-using-an-ssh-key/)

On a new server instance, you log in as the root user. Using root accounts for normal usage is inadvisable. So, 

 * [Create a new user with sudo rights](https://www.vultr.com/docs/how-to-use-sudo-on-a-vultr-cloud-server/#Create_a_Sudo_User_on_Debian___Ubuntu).
 * [Log in to the server over SSH as the non-root user](https://www.vultr.com/docs/using-your-ssh-key-to-login-to-non-root-users/).
 * This guide assumes you use the server as the user `pythonuser`.

You can install cuDNN in two ways:

 * natively - directly on the Operating System (OS)
 * using Conda. 

This guide shows both these methods. To install cuDNN using Conda:

 * either [install Anaconda](google.com) or
 * [install Miniconda](google.com) on the system
 * you also need to be familiar with [basic Conda operations](google.com).

cuDNN is only available for download from the NVIDIA website. To download software from NVIDIA, you need:

 * an active NVIDIA Developer account. [Create an account](https://developer.nvidia.com/login), if you don't already have one. 

You download the cuDNN installer onto your local machine and then copy it to the remote server. So, you also need:

 * to know [how to use SCP](https://www.vultr.com/docs/securely-transfer-files-over-the-private-network-using-scp-or-rsync/) to securely copy files over the network.

To use cuDNN, you need the CUDA Toolkit. [Install the right version of the CUDA Toolkit](google.com):

 * Install CUDA Toolkit `12.0.1` natively, or
 * Install CUDA Toolkit `11.8.0` using Conda

If you install cuDNN via Conda, it is advisable to also install the CUDA Toolkit using Conda. Similarly, to install cuDNN natively, install the CUDA Toolkit natively. This helps to keep the system consistent and simpler to manage.

### Test Server Configuration

Different versions of cuDNN depend on specific versions of dependency software and system configuration. The example commands used in this guide assume a test server with the following configuration:

 * GCC Version `11.3.0` 
 * NVIDIA graphics driver version `525.125.06`
 * Linux kernel version `5.15.0-75`

Given the configuration above, this guide shows how to:

 * install cuDNN `8.9.4` for CUDA `12.x` natively
 * install cuDNN `8.9.2` for CUDA `11.x` using Conda

To know how to check your system configuration and compatibility for other cuDNN versions, follow the steps in the section *Check System Compatibility for Other cuDNN Versions*. That section also explains how to choose the right version of cuDNN.

## Install Natively 

This section shows how to install cuDNN `8.9.4` for CUDA `12.x` natively.

### Install CUDA Toolkit

Before installing cuDNN, follow [the CUDA Toolkit installation guide](google.com) to install CUDA `12.0.1`. The following sections assume you have done so.

### Download the cuDNN Installer

To install cuDNN natively, you have two options. You can use either:

 * the OS-independent tar file, or
 * the deb file specifically for Ubuntu. 

The package manager deb file also contains a few dependencies that cuDNN needs to run on Ubuntu. However, there is a risk that the dependencies overwrite the files already installed on the system. This can make the system unstable. Thus, this guide recommends using the OS-independent tar file to install cuDNN.

On your local system, visit the [cuDNN download page](https://developer.nvidia.com/rdp/cudnn-download) to download the tar file for Linux x86_64. This is the installer for all 64-bit Linux systems, including Ubuntu 22.04. 

 * Log in with your NVIDIA developer account, or create a new account
 * Click the link that reads *Download cuDNN v8.9.4 (August 8th 2023), for CUDA 12.x*. 
 * It opens an accordion panel with links to installers for different Operating Systems. 
 * Click the link that reads *Local Installer for Linux x86_64 (Tar)*. 
 * The tar installer file downloads on your local machine. 

For cuDNN `8.9.4` for CUDA `12.x`, the filename of the tar file is:

    cudnn-linux-x86_64-8.9.4.25_cuda12-archive.tar.xz

Use SCP and copy the installer file to the server. Run this command on the local machine:

    $ scp cudnn-linux-x86_64-8.9.4.25_cuda12-archive.tar.xz pythonuser@192.168.1.1:/home/pythonuser/

### Install cuDNN

To install cuDNN, extract and move the cuDNN library files and executables into the appropriate CUDA subdirectories. In essence, you install cuDNN as an extension to CUDA itself. This guide assumes that CUDA Toolkit binaries are available in the `/usr/local/cuda/` directory.

Extract the tar file on the server:

    $ tar -xf cudnn-linux-x86_64-8.9.4.25_cuda12-archive.tar.xz 

Copy the cuDNN header files into the CUDA `include` directory:

    # cp cudnn-*-archive/include/cudnn*.h /usr/local/cuda/include/

Copy the cuDNN library files into the CUDA library:

    # cp -P cudnn-*-archive/lib/libcudnn* /usr/local/cuda/lib64/

Change the permissions of the library files so that all users have read access:

    # chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*

If you installed cuDNN following the above steps, skip the Conda installation section below and skip ahead to the section "Verify the Installation".

## Install with Conda

This section shows how to install cuDNN `8.9.2.26` for CUDA `11.x` using Conda. As of September 2023, this is the latest available cuDNN version for Conda. The section *Check System Compatibility for Other cuDNN Versions* discusses how to test compatibility and install other cuDNN versions.

### Install CUDA Toolkit

To install cuDNN `8.9.2` for CUDA 11.x, you also need:

 * CUDA Toolkit version 11.x.

The following command installs CUDA Toolkit `11.8.0` via Conda:

    $ conda install -c "nvidia/label/cuda-11.8.0" cuda

 > Ensure to follow the [Post-install steps](google.com) to activate the CUDA Toolkit on the system. If you do not do this, the tests will fail.

### Install cuDNN

To install a specific version, for example, `x.y.z.w` from a particular channel, use the command `conda install cudnn=="x.y.z.w" -c channel-name`. The examples below are based on installing cuDNN version `8.9.2.26` from the default channel.

    $ conda install cudnn=="8.9.2.26" -c default

## Verify the installation

It is necessary to test if the cuDNN installation is successful. Run a sample program packaged by NVIDIA specifically for this purpose. The sample programs are available deep inside the deb file for Ubuntu.

On your local machine: 

 * Visit the [cuDNN download page](https://developer.nvidia.com/rdp/cudnn-download) to download the local installer for Ubuntu. 
 * Click the link that reads *Local installer for Ubuntu22.04 x86_64 (Deb)*
 * The deb installer file downloads on your local machine. 

For cuDNN `8.9.4`, the filename of the deb file is:

    cudnn-local-repo-ubuntu2204-8.9.4.25_1.0-1_amd64.deb

Use SCP on the local machine and copy the deb installer to the server:

    $ scp cudnn-local-repo-ubuntu2204-8.9.4.25_1.0-1_amd64.deb pythonuser@192.168.1.1:/home/pythonuser/

On the server, create a new directory:

    $ mkdir deb

Move the deb installer file into this directory and `cd` into it:

    $ mv cudnn-local-repo-ubuntu2204-8.9.4.25_1.0-1_amd64.deb deb/

    $ cd deb

Extract the contents of the deb file using the `ar` program:

    $ ar x cudnn-local-repo-ubuntu2204-8.9.4.25_1.0-1_amd64.deb

This extracts the contents of the deb file and creates a few new files and folders:

    $ ls

Notice a new file called `data.tar.xz`. Extract its contents:

    $ tar -xf data.tar.xz

This creates 3 new folders: `etc`, `usr`, and `var`. Step into `cudnn-local-repo-ubuntu2204-8.9.4.25` in the `var` directory:

    $ cd var/cudnn-local-repo-ubuntu2204-8.9.4.25/

In this folder is another deb file with the sample programs. Extract its contents:

    $ ar x libcudnn8-samples_8.9.4.25-1+cuda12.2_amd64.deb

The extracted contents include a tar archive `data.tar.xz`. Extract this tar file:

    $ tar -xf data.tar.xz 

This extracts a `usr` directory that contains source code, including the sample programs. Step into the sample programs directory:

    $ cd usr/src/cudnn_samples_v8/

This has a few different sample programs. You need the `mnistCUDNN` program. Step into that directory:

    $ cd mnistCUDNN

The MNIST program is for image recognition. So, it depends on a few image-processing libraries. Install these dependencies:

    # apt-get install libfreeimage3 libfreeimage-dev

Clean any previous build artifacts:

    $ make clean

Compile the program:

    $ make

If the program compiles successfully, the output resembles:

    CUDA_VERSION is 12000
    Linking agains cublasLt = true
    TARGET ARCH: x86_64
    .
    .
    /usr/local/cuda/bin/nvcc -I/usr/local/cuda/include -I/usr/local/cuda/include 
    -IFreeImage/include -ccbin g++ -m64 -gencode 
    arch=compute_50,code=sm_50 -gencode 
    .
    .
    arch=compute_90,code=compute_90 -o fp16_dev.o -c fp16_dev.cu
    .
    g++ -I/usr/local/cuda/include -I/usr/local/cuda/include -IFreeImage/include   -o mnistCUDNN.o -c mnistCUDNN.cpp
    /usr/local/cuda/bin/nvcc -ccbin g++ -m64 -gencode arch=compute_50,code=sm_50 -gencode 
    .
    arch=compute_90,code=compute_90 -o mnistCUDNN fp16_dev.o fp16_emu.o mnistCUDNN.o -I/usr/local/cuda/include -I/usr/local/cuda/include -IFreeImage/include -L/usr/local/cuda/lib64 -L/usr/local/cuda/lib64 -L/usr/local/cuda/lib64 -lcublasLt -LFreeImage/lib/linux/x86_64 -LFreeImage/lib/linux -lcudart -lcublas -lcudnn -lfreeimage -lstdc++ -lm

If there are no compilation errors, there is an executable `mnistCUDNN` in this directory. Run it:

    $ ./mnistCUDNN

The output resembles:

    Executing: mnistCUDNN
    .
    Testing single precision
    .
    Loading binary file data/conv1.bin
    .
    Testing cudnnGetConvolutionForwardAlgorithm_v7 ...
    .
    ^^^^ CUDNN_STATUS_SUCCESS for Algo 5: -1.000000 time requiring 178432 memory
    .
    Testing cudnnFindConvolutionForwardAlgorithm ...
    ^^^^ CUDNN_STATUS_SUCCESS for Algo 1: 0.045248 time requiring 0 memory
    .
    ^^^^ CUDNN_STATUS_SUCCESS for Algo 4: 5.640960 time requiring 184784 memory
    .
    Test passed!

cuDNN is now installed on the system.

## Check System Compatibility for Other cuDNN Versions

This section discusses how to check the dependency packages available on your system. It then explains how to choose the right version of cuDNN depending on the software's dependencies and the configuration of your system.

### Check NVIDIA Drivers 

NVIDIA GPU Drivers are essential for the system to access the GPU. On Vultr Cloud GPU instances, these drivers are pre-installed during system initialization. Before installing NVIDIA cuDNN, check if the NVIDIA GPU drivers are properly installed:

    $ nvidia-smi

NVIDIA SMI stands for NVIDIA System Management Interface. The above command displays information about the GPU, similar to the example shown below: 

    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 525.125.06   Driver Version: 525.125.06   CUDA Version: 12.0     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |                               |                      |               MIG M. |
    |===============================+======================+======================|
    |   0  NVIDIA A40-1Q       On   | 00000000:06:00.0 Off |                    0 |
    | N/A   N/A    P8    N/A /  N/A |      0MiB /  1024MiB |      0%      Default |
    |                               |                      |             Disabled |
    +-------------------------------+----------------------+----------------------+

The example output above shows: 

 * the version of  NVIDIA Accelerated Graphics Driver Version is `525.125.06` on the test server.

You can only install a cuDNN version for which the minimum required NVIDIA driver version is lower than the one available on your system. 

### Check CUDA Toolkit 

In practice, applications that use cuDNN also need the CUDA Toolkit to work. Check if the CUDA compiler is properly installed:

    $ nvcc --version

The output of the above command should resemble:

    nvcc: NVIDIA (R) Cuda compiler driver 
    Copyright (c) 2005-2023 NVIDIA Corporation 
    Built on Tue_Jul_11_02:20:44_PDT_2023 
    Cuda compilation tools, release 12.0, V12.0.140
    Build cuda_12.0.r12.0/compiler.32267302_0

Based on the example output above, the test system has CUDA `12.x.`.

### Check the Kernel Version

cuDNN also needs a relatively recent kernel to be installed. Your system must have a kernel that is more recent than the minimum kernel version required by the cuDNN version you want. Check the kernel version installed on your system:

    $ uname -r

The output of the above command resembles the example below:

    5.15.0-75-generic 

This implies the kernel installed on the test system is version `5.15.0-75`. 

### Check the GCC Version

The GNU Compiler Collection (GCC) is necessary to compile application programs. Your system must have a higher version of GCC than required by cuDNN. 

Check the installed version of GCC:

    $ gcc --version

The output of the above command resembles:

    gcc (Ubuntu 11.3.0-1ubuntu1~22.04.1) 11.3.0 
    Copyright (C) 2021 Free Software Foundation, Inc. 
    This is free software; see the source for copying conditions.  
    There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

The example above shows that GCC version `11.3.0` is installed on the test server.

### Choose the Right Version of cuDNN

Decide the appropriate cuDNN version for your system based on the:

 * kernel version of the Operating System on the server
 * GCC version available on the system
 * NVIDIA drivers version installed on the system
 * requirements of the application programs you want to run. 

The next two sections show the compatibility checks for both the cuDNN versions used in this guide. 

#### cuDNN `8.9.4`

As of September 2023, the latest cuDNN version available natively is `8.9.4`. For the latest cuDNN release, the default [cuDNN Support Matrix](https://docs.nvidia.com/deeplearning/cudnn/support-matrix/index.html) page shows the CUDA Toolkit and NVIDIA Driver versions that cuDNN is compatible with. The [first section of the Support Matrix](https://docs.nvidia.com/deeplearning/cudnn/support-matrix/index.html#cudnn-cuda-hardware-versions) lists the GPU, CUDA Toolkit, and CUDA Driver requirements. 

Notice, from the above support matrix: 

 * cuDNN `8.9.4` for CUDA 12.x works with CUDA Toolkit versions `12.2`, `12.1`, and `12.0`. 
 * cuDNN `8.9.4` for CUDA 12.x needs NVIDIA Drivers more recent than `525.60.13`. As seen in the Drivers Check section earlier, the test server has NVIDIA Driver version `525.125.06`. 

The [second section of the Support Matrix](https://docs.nvidia.com/deeplearning/cudnn/support-matrix/index.html#os-versions) lists the architecture, kernel, and GCC requirements. Notice, from this support matrix: 

 * cuDNN `8.9.4` needs a minimum kernel version of `5.15.0` on Ubuntu `22.04`. The kernel version on the test system is `5.15.0-75`, which is higher than the minimum required version.
 * cuDNN `8.9.4` needs a minimum GCC version of `11.2.0` on Ubuntu `22.04`. The GCC version on the system is `11.3.0`, which is also higher than the minimum required version. 

Hence, the test system has the right kernel, GCC, and NVIDIA drivers to support cuDNN `8.9.4`. 

#### cuDNN `8.9.2`

As of September 2023, the latest cuDNN version for Conda is `8.9.2` for CUDA 11.x. In general, Conda has somewhat older releases.

For older cuDNN releases, [the archives page](https://docs.nvidia.com/deeplearning/cudnn/archives/index.html) has links to the documentation. Open the *NVIDIA cuDNN Support Matrix* page for the specific cuDNN version you are interested in. 

 * On the [cuDNN documentation archives page](https://docs.nvidia.com/deeplearning/cudnn/archives/index.html), locate the section for cuDNN version `8.9.2`. It is titled *cuDNN Release 8.9.2 Documentation*. 
 * Visit the [cuDNN Release 8.9.2 Documentation](https://docs.nvidia.com/deeplearning/cudnn/archives/cudnn-892/support-matrix/index.html) for this version. 

The cuDNN version available on the default Conda channel is compiled against CUDA 11.0. So, in the Support Matrix, check the requirements for *cuDNN 8.9.2 for CUDA 11.x*. 

Notice, from the above support matrix: 

cuDNN `8.9.2` for CUDA 11.x needs:

 * Kernel `5.15.0` on Ubuntu 22.04. The test system has `5.15.0-75`
 * GCC `11.2.0` on Ubuntu 22.04. The test system has `11.3.0`.
 * NVIDIA Driver version `450.80.02`. The test system has `525.125.06`.

As indicated in the section *Test Server Configuration*, the test server meets the above requirements.

The process for checking other cuDNN releases on other CUDA versions is similar. Follow the same instructions to get the links and filenames. 

### Check Available Versions

This section shows how to check which cuDNN versions are available to download and install.

#### Check Versions to Install Natively

 * The [cuDNN download page](https://developer.nvidia.com/rdp/cudnn-download) has the latest cuDNN version available to download. 
 * To check older cuDNN versions, visit [the archives page](https://developer.nvidia.com/rdp/cudnn-archive). On the archives page, find and follow the link to the version you need to download.

#### Check Versions for Conda

For Conda, check the cuDNN versions available on the `conda-forge`, `nvidia`, `anaconda`, and default channels:

    $ conda search cudnn -c conda-forge -c nvidia -c anaconda  -c default

As of September 2023: 

 * Not all versions are available on all channels. 
 * The `conda-forge` channel has a larger collection of cuDNN versions compared to other channels.
 * However, as of September 2023, the default repository `pkgs/main` has the latest version, `8.9.2.26` available for Conda.

The output of the above command resembles the example below:

    # Name                       Version           Build  Channel
    cudnn                          7.0.5       cuda8.0_0  pkgs/main
    .
    .
    cudnn                          7.6.5       cuda9.2_0  pkgs/main
    cudnn                          8.2.1      cuda11.3_0  pkgs/main
    cudnn                       8.9.2.26        cuda11_0  pkgs/main

In the above output, the `Build` column shows that the version of CUDA that each cuDNN release was compiled against. Notice that version `8.9.2.26` was compiled against CUDA `11.0`.

## Conclusion

This guide showed how to install cuDNN on Ubuntu 22.04 servers - both natively and using Conda. It discussed how to check your system configuration and decide the appropriate version of cuDNN for your needs.

To learn further details about installing cuDNN, visit NVIDIA's [cuDNN Installation Guide](https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html). To develop applications that depend on cuDNN, use the cuDNN API. The [cuDNN API Reference](https://docs.nvidia.com/deeplearning/cudnn/api/index.html) discusses the functions that are available in cuDNN, such as routines for training and inference using neural networks.

