# How to Install PyTorch on Ubuntu 22.04

## Introduction

PyTorch is a fully-featured machine learning framework. It includes a library of tools used to build machine-learning models. It is commonly used because it is written in Python and hence accessible to Python programmers. As a platform, it is sometimes considered an alternative to TensorFlow.

The instructions in this guide install the latest stable release of PyTorch. 

To install previous PyTorch releases, find the sample commands from the [PyTorch documentation page](https://pytorch.org/get-started/previous-versions/).

### Prerequisites

To follow this guide and install PyTorch, you need to:

 * [Generate SSH keys](https://www.vultr.com/docs/how-do-i-generate-ssh-keys/) and [add the keys to your account](https://www.vultr.com/docs/deploy-a-new-server-with-an-ssh-key/#Add_an_SSH_key_to_your_Vultr_Account)
 * Deploy a Vultr instance running Ubuntu 22.04 LTS   
 * [Connect to the server over SSH](https://www.vultr.com/docs/connect-to-a-server-using-an-ssh-key/)

 * [Create a new user with sudo rights](https://www.vultr.com/docs/how-to-use-sudo-on-a-vultr-cloud-server/#Create_a_Sudo_User_on_Debian___Ubuntu). 
 * [Log into the server over SSH as the non-root user](https://www.vultr.com/docs/using-your-ssh-key-to-login-to-non-root-users/)

### Installation Methods

You can install PyTorch on:

 * Systems with a GPU
 * CPU-only systems

In both cases, you can install it using two methods:
 
 * Install it natively
 * Install in a Conda environment

This guide shows both of these methods for both kinds of systems. To install PyTorch using Conda, you also need to:

 * [install Anaconda](google.com) or
 * [install Miniconda](google.com) on the server
 * be comfortable with [using Conda](google.com).

Vultr instances running Ubuntu 22.04 are pre-installed with Python3 and pip3. 

## Install on Systems with a GPU

To install PyTorch, you also need to install its supporting packages - `torchvision` and `torchaudio`. These packages contain specific libraries for processing image and audio files. 

### Install Natively

To take advantage of the GPU, install the version of PyTorch that depends on CUDA. As of September 2023, CUDA `11.8` is the latest CUDA that PyTorch is built on. Pass to `pip3` the `--index-url` option with the value `https://download.pytorch.org/whl/cu118`. The `--index-url` option allows to specify a Base URL for the Python Package Index to use. pip uses the Base URL to download packages.

    $ pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

### Install in a Conda Environment

PyTorch packages for Conda are available from the `pytorch` and the `nvidia` channels. For installing using Conda in a GPU-environment, you also need to install the `pytorch-cuda` package.

    $ conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia

## Install on CPU-only Systems

To install PyTorch on CPU-only systems, install the same packages as before - `torch`, `torchvision`, and `torchaudio`.

### Install Natively

To install PyTorch using pip on CPU-only systems, set the `index-url` to `https://download.pytorch.org/whl/cpu`. This downloads the CPU-specific packages.

    $ pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu

### Install in a Conda Environment

To install PyTorch using Conda on a CPU-only system, install also install the `cpuonly` package from the `pytorch` Conda channel. Because this is a CPU-only environment, do not use packages from the `nvidia` channel.

    $ conda install pytorch torchvision torchaudio cpuonly -c pytorch

## Test the Installation

To check if PyTorch is properly installed, run a test operation. 

Start a Python terminal:

    $ python3

Type the commands below at the Python prompt. In the Python terminal, import `torch`:

    >>> import torch

Declare a random tensor:

    >>> x = torch.rand(1)

Print it:

    >>> print(x)

The output resembles:

    tensor([0.4169])

### Check GPU Access

Check also if PyTorch has access to the GPU. After importing torch in a Python shell (as shown above), test the following command:

    >>> torch.cuda.is_available()

On systems with a GPU, the output is:

    True

On systems without a GPU, the output is:

    False

## Conclusion

This guide shows how to install PyTorch on Ubuntu 22.04. The instructions cover both systems with and without GPUs. In both cases, the guide shows how to install PyTorch natively and in a Conda environment.

The [official PyTorch installation documentation](https://pytorch.org/get-started/locally/) discusses the installation methods for different scenarios.

### More Information

As of September 2023, the latest stable PyTorch release is version `2.0.1`.

The [PyTorch documentation](https://pytorch.org/get-started/previous-versions/) shows the sample commands to install older PyTorch versions, both natively and using Conda.

