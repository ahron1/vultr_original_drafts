# How to Install TensorFlow on Ubuntu 22.04

## Introduction

TensorFlow is an open-source software library. It includes implementations of many common algorithms used for machine learning in general and deep neural networks in particular. Hence, it is commonly used for Deep Learning applications. You need TensorFlow to develop and deploy many popular Deep Learning models, such as Stable Diffusion, Large Language Models, and the like.

This guide shows how to install TensorFlow on Ubuntu 22.04 systems. It shows how to install it for:

 * Systems with a GPU
 * Systems without a GPU

In both cases, the guide covers:

 * How to install natively as a Python package
 * How to install within a Conda environment

### Prerequisites

To follow the steps in this guide, you need to:

 * [Generate a public-private pair of SSH keys](https://www.vultr.com/docs/how-do-i-generate-ssh-keys/) and [add the public key to your Vultr account](https://www.vultr.com/docs/deploy-a-new-server-with-an-ssh-key/#Add_an_SSH_key_to_your_Vultr_Account).
 * Deploy a new Vultr instance running Ubuntu 22.04 LTS 
 * [Connect to the new instance over SSH](https://www.vultr.com/docs/connect-to-a-server-using-an-ssh-key/)

 * [Create a regular user that has sudo rights](https://www.vultr.com/docs/how-to-use-sudo-on-a-vultr-cloud-server/#Create_a_Sudo_User_on_Debian___Ubuntu).
 * [Use SSH to log in as the non-root user into the instance over SSH](https://www.vultr.com/docs/using-your-ssh-key-to-login-to-non-root-users/).

To install TensorFlow using Conda, you also need to:

 * either [install Anaconda](google.com) or
 * [install Miniconda](google.com) on the server
 * be comfortable with [using Conda](google.com).

## Install TensorFlow on Systems with a GPU

### Install Natively as a Python Package 

This section shows how to install TensorFlow `2.13` natively, as a pip package. TensorFlow `2.13` depends on:

 * CUDA Toolkit `11.8` 
 * cuDNN `8.6.0`. 

If you install TensorFlow natively, it is advisable to:

 * Install the dependencies natively. This helps to keep the system consistent. 
 * Install the recommended version of the dependencies. This helps avoid potential compatibility issues. Dependencies are usually backward compatible, but not always.

#### Install CUDA Toolkit 11.8

The *Install Natively* section in the [CUDA Toolkit installation guide](google.com) explains in detail how to install any version of the CUDA Toolkit. 
Download the installer runfile for CUDA Toolkit `11.8`:

    $ wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda_11.8.0_520.61.05_linux.run

Run the installer:

    # sh cuda_11.8.0_520.61.05_linux.run

 * Install only the CUDA Toolkit using the above installer.
 * Do **not** install the drivers using the above installer. Doing so can corrupt the system. Vultr instances come with the right drivers pre-installed.
 * Ensure to [configure the system, as described in the CUDA Toolkit guide](google.com). If you do not do this, the system does not recognize the newly installed CUDA dependencies.

#### Install cuDNN 8.6.0 for CUDA 11.x

The [cuDNN archives page](https://developer.nvidia.com/rdp/cudnn-archive) has links to download older cuDNN versions. Download the Linux installer `tar` file for cuDNN `v8.6.0` for CUDA `11.x`: 

 * On the archives page, click the link that reads *Download cuDNN v8.6.0 (October 3rd, 2022), for CUDA 11.x*. 
 * In the accordion that opens, click the link that reads *Download cuDNN v8.6.0 (October 3rd, 2022), for CUDA 11.x* to download the installer. 
 * You need an NVIDIA developer account to download installers.

Follow the *Install Natively* section in the [cuDNN Installation Guide](google.com) to learn how to install cuDNN 8.6.0 for CUDA 11.x.

#### Install TensorFlow

This section assumes that you have successfully installed the CUDA Toolkit and cuDNN.

Install TensorFlow `2.13` using pip:

    $ python3 -m pip install tensorflow==2.13.*

After installing TensorFlow, jump to the section *Test the Installation* to check if the installed software works as expected.

### Install in a Conda Environment

Conda has two separate packages for TensorFlow:

 * `tensorflow-gpu` is for systems with a GPU. Conda also automatically installs the CUDA and cuDNN dependencies. 
 * `tensorflow` is the package for CPU-only systems. 

As of September 2023, compared to the default channel, the `conda-forge` channel has more recent versions of TensorFlow. Install TensorFlow `2.12.1` from the `conda-forge` channel:

    $ conda install tensorflow-gpu=2.12.1 -c conda-forge -y

Conda installs TensorFlow `2.12.1` along with CUDA Toolkit `11.8` and cuDNN `8.8`. 

 > To work with GPUs, TensorFlow needs the CUDA and cuDNN packages. The Conda installation process installs the CUDA Toolkit and cuDNN dependencies. However, these are **not** the same as the full-fledged CUDA Toolkit and cuDNN packages you would normally install. For example, the `cudatoolkit` package, which Conda installs as a TensorFlow dependency, does not enable you to use the `nvcc` compiler. If your use case needs these packages, explicitly [install the CUDA Toolkit](google.com) and [cuDNN](google.com) within the Conda environment.

## Install TensorFlow on CPU-only systems

### Install Natively as a Python Package

To natively install TensorFlow using pip on CPU-only systems, install `tensorflow` without any of the GPU-specific dependencies, such as CUDA Toolkit and cuDNN. 

    $ python3 -m pip install tensorflow==2.13.*

### Install in a Conda Environment

As mentioned earlier, Conda offers two TensorFlow packages - `tensorflow-gpu` and `tensorflow`. The Conda `tensorflow` package is CPU-only. Install it:

    $ conda install tensorflow=2.12.0 -y

## Test the Installation

### Test on Systems with a GPU

If TensorFlow is properly installed, it can access the GPUs. The command below imports the TensorFlow package in Python and uses it to print the list of GPU devices:

    $ python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"

The output resembles:

    [PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]

Verify that it can perform a tensor-based operation using random numbers:

    $ python3 -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([100, 100])))"

The output resembles:

    2023-09-06 10:18:24.938874: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library libcudart.s
    .
    .
    2023-09-06 10:18:27.066352: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1406] 
    Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 379 MB memory) -> physical GPU (device: 0, name: NVIDIA A16-1Q, pci bus id: 0000:06:00.0, compute capability: 8.6)

    tf.Tensor(-79.17527, shape=(), dtype=float32)


The last line in the sample output above is the output of the tensor computation. 

### Test on CPU-only Systems 

If you try to get the list of GPU devices on a system without a GPU, you should get an empty list. Verify this:

    $ python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"

The output is:

    []

The Empty array means there are no GPU devices available to TensorFlow.

Verify that TensorFlow can perform a basic tensor-based computation:

    $ python3 -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([100, 100])))"

The above command executes a tensor-based operation using random numbers.

The output resembles:

    .
    Could not find cuda drivers on your machine, GPU will not be used.
    .
    This TensorFlow binary is optimized to use available CPU instructions in performance-critical operations.
    .
    tf.Tensor(-276.19254, shape=(), dtype=float32)

The last line in the sample output above is the result of the computation. 

Sometimes, the output contains a warning that it could not find TensorRT:

    2023-09-12 10:43:13.438987: W tensorflow/compiler/tf2tensorrt/utils/py_utils.cc:38] 
    TF-TRT Warning: Could not find TensorRT

[TensorRT](https://developer.nvidia.com/tensorrt) is a package used for optimizing inference operations. It is not necessary for regular operations using TensorFlow. Installing TensorRT is beyond the scope of this guide. 

## Choose the Right Installation Method

 * Conda doesn't always have the latest version available. 
 * If the application you are deploying is released via Conda, it is advisable to also install TensorFlow using Conda. 
 * Installing natively allows you to install more recent versions. It also gives you more control but at the cost of having to manually install dependencies and configure them. 
 * It is possible to install some packages via Conda and some natively. If you do this, take care that you do not install conflicting versions of the same dependency from different sources.

## Conclusion

This guide showed how to install TensorFlow on systems with and without a GPU. In both cases, it shows how to install it natively or using Conda. 

### Further Information

 * The [official TensorFlow installation guide](https://www.tensorflow.org/install/pip#linux) has further information on various methods of installing TensorFlow. The sample commands on this page indicate the correct version of the CUDA Toolkit and cuDNN that you need to install as dependencies.
 * The [TensorFlow page on Anaconda](https://docs.anaconda.com/free/anaconda/applications/tensorflow/) discusses how to install using Conda. 
 * The [TensorFlow page of the `default` channel of Conda](https://anaconda.org/anaconda/tensorflow) has information on the different versions available on the `default` channel.
 * The [TensorFlow page of the `conda-forge` channel of Conda](https://anaconda.org/conda-forge/tensorflow) has information on the different versions available on the `conda-forge` channel.

