# Implement Stable Diffusion with Keras and TensorFlow

## 1. Introduction

The advent of machine learning, in particular, deep learning, has opened up new possibilities in computing. In recent years, this technology has been making its way from researchers to practitioners. Deep learning models can summarize articles, answer open and closed ended questions, generate text, and even generate images and videos.

In mid-late 2022, OpenAI released DALL-E 2 to the general public. DALL-E 2 is a deep learning model for generating images based on text prompts entered by the user. As of October 2022, this model is closed source. There were restrictions on the nature of images that went into the training set. There are also restrictions on the nature of prompts that a user can enter. Google's Imagen is another proprietary text-to-image model that is not available for the general public to use (as of October 2022).

### 1.1. Stable Diffusion

Stable Diffusion is an open-source deep learning model that generates images from text prompts. Stability AI, the CompVis group at LMU Munich, and Runway are its main developers. Stable Diffusion models were trained using millions of publicly available (online) high resolution images and their associated text descriptions (in the English language). The training process used a [cluster of 4000 NVIDIA A100 GPUs](https://stability.ai/blog/stable-diffusion-announcement).

The model weights, the model card, and the code, are all [open source](https://github.com/CompVis/stable-diffusion). Anyone can train and customize the model, or download and run a pre-trained model.

### 1.2. Scope

This guide shows how to implement (deploy) a pre-trained Stable Diffusion model using Keras and TensorFlow. The starting point is a vanilla Debian installation. The end goal is to use a pre-trained model to deploy your own system to generate images from text prompts. This guide does not cover how to train models from data.

## 2. Pre-requisites and Compatibility

### 2.1 First Principles

In deep learning, input data are represented as tensors. The input data is transformed into the output by many sequential layers of tensor operations.  TensorFlow and PyTorch are opensource frameworks of computational tools for building deep learning models. Keras is an easier-to-use Python frontend API at a higher level of abstraction. This makes it better suited to implement models. Keras uses TensorFlow as the backend for performing the actual computations. It can also use other backends like Theano. Keras is now integrated with TensorFlow. KerasCV is a "horizontal extension" of the core Keras API. It is a collection of building blocks specifically for computer vision problems.

Stable Diffusion was originally built on PyTorch. In September 2022, it became available directly on KerasCV.

### 2.2. Hardware

As of October 2022, the Stable Diffusion model uses about 7 GB of Video-RAM. It is recommended to run the model on a system with at least 10 GB Video-RAM. Stability AI recommends using NVIDIA chips and plans to release optimized versions for other chipsets (AMD, Apple silicon, etc.). NVIDIA's A100 GPU is oriented towards applications like AI, analytics, and deep learning; the A40 GPU is better suited for visual computing, graphics, simulations, etc.

Vultr Cloud GPU VMs virtualize the GPU and allocate a fraction of the GPU chip's total capacity to each VM. In principle, it is possible to run the models on CPU-only machines. In practice, it can take orders of magnitude more time to generate images in the absence of a GPU.

This guide has been thoroughly tested on VMs featuring a 10 GB GPU RAM share of the NVIDIA A100 GPU. Trying to run the models on a machine with insufficient GPU memory leads to errors.

**Note:** The user does not need to do anything special to take advantage of the GPU. As long as the drivers and CUDA libraries are installed, it happens automatically.

#### Practical Tip

Cloud GPU Virtual Machines are expensive. It can be cost effective to set up the model on a Virtual Machine with a small amount of GPU RAM. This fetches the right drivers and dependencies. For running the model and generating images, a snapshot of this VM can be restored onto a machine with more GPU RAM. Note that both source and target VMs must use the same GPU chipset.

### 2.3. Programming Language

Basic familiarity with Python 3 is necessary to follow the commands in the subsequent sections. This guide uses Conda to manage Python environments and PyPI `pip` for package management.

### 2.4. Operating System and Images

The base Operating System used is the Vultr Debian 11 image. The commands should be 1-to-1 compatible with recent Ubuntu releases. To use another Operating System, adapt the package installation commands. Python commands are the same across Operating Systems.

It is also possible to use the Vultr Anaconda and Miniconda images. They come with Conda preinstalled. Skip the step (Section 3.2.1) about installing `conda` if you use these images.

### 2.5. Version Numbers

Deep Learning is a rapidly evolving field. The models and the software packages are frequently updated. This sometimes leads to compatibility issues. The current version of a package can depend on an older version of another package. The latest versions of two packages might be incompatible with each other. Two required packages can each depend on different versions of a third package.

Wherever applicable, the installation commands mention the version numbers of the software used in the test setup. Using newer or older versions of the same software may lead to incompatibilities between the dependencies.

## 3. System Setup

### 3.1. Create New User

From the Vultr dashboard, deploy a new Cloud GPU instance. It is recommended to add a SSH key during deployment.

By default, you log in as the root user. It is strongly recommended to create and use a regular user account.

Log in to the new server as `root` and create an unprivileged user:

    # adduser username

Follow the on-screen instructions and complete the user creation process.

Give the new user `sudo` rights so it can do privileged actions like installing new programs:

    # usermod -aG sudo username

Copy the `.ssh` directory (containing the keys) of `root` into the home directory of the new user. This allows to SSH into the server as the unprivileged user.

    # cp -R /root/.ssh /home/username/

Change the owner of the copied directory from `root` to the new user:

    # chown -R username /home/username/.ssh

The default Debian installation on Vultr uses UFW for managing the firewall. Confirm that this is the case by checking the UFW settings:

    # ufw status verbose

The output should be equivalent to:

    Status: active
    Logging: on (low)
    Default: deny (incoming), allow (outgoing), disabled (routed)
    New profiles: skip
    
    To                         Action      From
    --                         ------      ----
    22 (v6)                    ALLOW IN    Anywhere 
    22 (v6)                    ALLOW IN    Anywhere (v6)

If the firewall is not configured, [set it up](https://www.vultr.com/docs/how-to-configure-uncomplicated-firewall-ufw-on-ubuntu-20-04/) before continuing further.

Open a new terminal window and log in as the unprivileged user:

    ssh username@ip.address

After confirming that the new user has SSH and `sudo` access, go back to the old terminal session where you are logged in as `root`. Exit that session and continue further as the regular user.

It is advisable to [properly configure SSH](https://www.vultr.com/docs/best-practices-for-ssh-on-a-production-cloud-server/) on the server, and to use either [TMUX](https://www.vultr.com/docs/how-to-install-and-use-tmux/) or [Screen](https://www.vultr.com/docs/an-introductory-guide-to-screen/) for managing remote sessions.

### 3.2. Install Required Software

This guide uses Conda for managing Python environments. Conda also makes it easier to install the required CUDA and cuDNN software. CUDA is a parallel computation platform for NVIDIA GPUs. cuDNN is a library for Deep Neural Networks optimized for CUDA.

#### 3.2.1. Install Conda

Miniconda provides a minimal installation tool for Conda. Copy the download link for the latest Miniconda release from the [Conda website](https://docs.conda.io/en/latest/miniconda.html). Download the file:

    $ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

Execute the downloaded script to install Miniconda:

    $ bash Miniconda3-latest-Linux-x86_64.sh

Read and follow the on-screen instructions to complete the installation process and to initialize Conda.

Check if Conda is installed:

    $ conda --version

Note: Skip this subsection if you use the Vultr Anaconda or Miniconda installation images.

#### 3.2.2. Set up New Environment

Ensure `conda` is upgraded to the latest version:

    $ conda upgrade conda

Create a new Conda environment:

    $ conda create --name env1 python=3.9

This creates a new environment *env1* and pre-installs Python 3.9 into it. Read and follow the on-screen instructions during the installation process.

View the list of Conda environments with `conda env list`. Activate the newly created Conda environment:

    $ conda activate env1

To deactivate the currently active environment, enter the command `conda deactivate` at the terminal.

The rest of this guide assumes that the Conda environment `env1` is active.

Upgrade `pip` to the latest version:

    $ pip install --upgrade pip

#### 3.2.3. Install GPU Packages

Install the CUDA and cuDNN packages (into the current Conda environment):

    $ conda install -c conda-forge cudatoolkit=11.2 cudnn=8.1.0

Update the system path to reflect the Conda installation paths:

    $ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CONDA_PREFIX/lib/

Create the Conda activation directory and add the system paths to it.

    $ mkdir -p $CONDA_PREFIX/etc/conda/activate.d

    $ echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CONDA_PREFIX/lib/' > $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh

This automatically loads the paths at the start of every Conda session.

#### 3.2.4. Install TensorFlow and KerasCV

The version of KerasCV on the Conda repository is outdated (as of October 2022) and doesn't include Stable Diffusion. Therefore, PyPI `pip` is used to install KerasCV and its dependencies (into the current Conda environment).

Install TensorFlow with `pip`:

    $ pip install tensorflow==2.10.0

**Note:** The PyPI repositories contain the packages `tensorflow`, `tensorflow-gpu`, and `tensorflow-cpu`. It is sufficient to install just `tensorflow` - since TensorFlow v2.0, this package supports both CPU and GPU computations.

Install TensorFlow Datasets:

    $ pip install tensorflow-datasets==4.7.0

These datasets are used by the Deep Learning libraries.

Install KerasCV:

    $ pip install keras-cv==0.3.4

Install Matplotlib to process images:

    $ pip install matplotlib==3.6.1

#### 3.2.5. Verify the Installation

Verify that the software has installed correctly and is working as expected. Open a Python terminal:

    $ python

Import tensorflow:

    >>> import tensorflow as tf

Run a CPU based TensorFlow operation:

    >>> print(tf.reduce_sum(tf.random.normal([1000, 1000])))

If the above command ran properly, it should return a tensor. The output should look like:

    tf.Tensor(-1798.1721, shape=(), dtype=float32)

Run a GPU based command:

    >>> print(tf.config.list_physical_devices('GPU'))

The above command lists the GPU devices available. The output should look like:

    [PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]

Exit the Python command-line.

### 3.3. Warning Message about TensorRT

Importing TensorFlow in Python may have shown a message like this:

    >>> import tensorflow as tf
    ...
    2022-10-23 05:01:28.981482: W tensorflow/stream_executor/platform/default/dso_loader.cc:64] Could not load dynamic library 'libnvinfer.so.7'; dlerror: libnvinfer.so.7: cannot open shared object file: No such file or directory
    2022-10-23 05:01:28.981561: W tensorflow/stream_executor/platform/default/dso_loader.cc:64] Could not load dynamic library 'libnvinfer_plugin.so.7'; dlerror: libnvinfer_plugin.so.7: cannot open shared object file: No such file or directory
    2022-10-23 05:01:28.981576: W tensorflow/compiler/tf2tensorrt/utils/py_utils.cc:38] TF-TRT Warning: Cannot dlopen some TensorRT libraries. If you would like to use Nvidia GPU with TensorRT, please make sure the missing libraries mentioned above are installed properly.
    ...

This message is a warning that the installation is missing TensorRT. TensorRT is a TensorFlow SDK specifically for deep learning inference. Deep learning models are of two broad classes - inference and prediction. Stable Diffusion is a prediction model. It does not need TensorRT to run.

### 3.4. Viewing Remote Images

Viewing images on a remote server over SSH needs a bit of effort. As the last step before generating images, create the setup to view the generated images. Use `sshfs` to mount and browse a remote directory on the local filesystem.

Check if `sshfs` is installed on the local machine:

    $ sshfs --version

Install it, if it already isn't:

    # apt install sshfs

    $ brew install sshfs

Create an empty directory on the local machine:

    $ mkdir ~/sshfs_mountpoint

In the next section, you will save the generated images on the server's home directory. Connect `sshfs` to the server and mount the home directory:

    $ sshfs -o follow_symlinks username@remote.server.ip.address:/home/username ~/sshfs_mountpoint

This allows to browse (either via command-line or the local GUI) the remote directory as if it were a local directory. View the images using a viewing program on the local machine, or copy the images to a local directory.

**Note:** Depending on the server's `sshd_config` settings, the connection may close if left unused for a period of time. When this happens, reconnect (using the above command) and refresh the GUI window.

## 4. Generate Images

Start a new Python command-line and import the necessary modules.

Matplotlib is used to save the generated images.

    >>> import matplotlib.pyplot as plt

 Import Keras:

    >>> from tensorflow import keras

Set Keras to use half precision floating point numbers (16 bits instead of the usual 32 bits):

    >>> keras.mixed_precision.set_global_policy("mixed_float16")

This helps to speed up GPU computations.

Import the KerasCV module:

    >>> import keras_cv

Stable Diffusion is a packaged as a part of KerasCV.

### 4.1. Default Settings

 Construct a new Stable Diffusion model with the default settings:

    >>> model = keras_cv.models.StableDiffusion()

This downloads the current version of the text encoder, the diffusion model, and the decoder, and constructs the model.

Call the model with a text prompt to generate an image:

    >>> image = model.text_to_image(prompt='oil painting of a vulture wearing a christmas hat')

This generates an array consisting of a single image. Save this image to the current working directory:

    >>> plt.imsave('/home/username/vulture_pic.png', image[0])

Use `sshfs` (as discussed in Section 3.4) to view the saved image.

### 4.2. Advanced Settings

Restart the Python command-line to free up memory for a new model. Import `matplotlib`, `keras`, and `keras_cv` as before.

#### 4.2.1. Model Specification - Image Resolution

By default, `keras_cv.models.StableDiffusion()` constructs a new Stable Diffusion Model that generates `512x512` images. It is possible to change this with the parameters `img_height` and `img_width`:

    >>> model = keras_cv.models.StableDiffusion(img_height = 128, img_width = 128)

Note that the resolution directly affects the GPU memory required by the model to function.

#### 4.2.2. Image Generation

The function `model.text_to_image` accepts the following parameters:

* `prompt`: the text prompt based on which images are generated
* `batch_size`: the number of images to generate at a time. The default value is 1.
* `num_steps`: the number of iterations - higher the number of iterations, better the image quality and detail. The default value is 25. The higher this value, the longer it takes to generate the image(s). This parameter mainly affects the CPU usage time.

##### Example 1

Generate a single image of low quality:

     images = model.text_to_image(
        prompt =  'oil painting of a vulture wearing a christmas hat',
        batch_size = 1,
        num_steps = 5,
     )

##### Example 2

Generate a set of 3 images of high quality:

     images = model.text_to_image(
        prompt =  'oil painting of a vulture wearing a christmas hat',
        batch_size = 3,
        num_steps = 50,
     )

The output in the above examples is an array of one or more images.

#### 4.3. Save Multiple Images

Write a simple function to save multiple generated images in an array:

    def save_images(images, file_name_with_path):
        for i in range(len(images)):
            plt.imsave(file_name_with_path + str(i+1) + '.png', images[i])

Call this function as:

     save_images(images, '/home/username/christmas_vultr_')

This saves the generated images with the filenames `christmas_vultr_1.png`, `christmas_vultr_2.png`, and so on, in the home directory, `/home/username/`. Use `sshfs` to view the saved images.

#### 4.4. Out of Memory Errors

Since Stable Diffusion relies on the GPU, heavy models cause the system to run out of memory.

An out of memory error message looks like this:

    2022-10-28 04:33:01.109327: I tensorflow/core/common_runtime/bfc_allocator.cc:1097] 1 Chunks of size 122713088 totalling 117.03MiB
    2022-10-28 04:33:01.109335: I tensorflow/core/common_runtime/bfc_allocator.cc:1097] 2 Chunks of size 151781376 totalling 289.50MiB
    2022-10-28 04:33:01.109344: I tensorflow/core/common_runtime/bfc_allocator.cc:1097] 2 Chunks of size 176947200 totalling 337.50MiB
    ...
    Memory allocation details
    ...
    2022-10-28 04:33:01.109362: I tensorflow/core/common_runtime/bfc_allocator.cc:1101] Sum Total of in-use chunks: 6.66GiB
    2022-10-28 04:33:01.109370: I tensorflow/core/common_runtime/bfc_allocator.cc:1103] total_region_allocated_bytes_: 7365066752 memory_limit_: 7365066752 available bytes: 0 curr_region_allocation_bytes_: 14730133504
    2022-10-28 04:33:01.109384: I tensorflow/core/common_runtime/bfc_allocator.cc:1109] Stats: 
    Limit:                      7365066752
    InUse:                      7151588096
    MaxInUse:                   7151588352
    NumAllocs:                        5650
    MaxAllocSize:                176947200
    Reserved:                            0
    PeakReserved:                        0
    LargestFreeBlock:                    0
    
    2022-10-28 04:33:01.109467: W tensorflow/core/common_runtime/bfc_allocator.cc:491] ****************************************************************************************************
    2022-10-28 04:33:01.113312: W tensorflow/core/framework/op_kernel.cc:1768] RESOURCE_EXHAUSTED: failed to allocate memory

    Traceback (most recent call last):
    ...
    Detailed stack trace
    ...

It is difficult to accurately predict all the parameters that cause the system to run out of memory. These are some example scenarios where out of memory errors were observed as of October 2022 on a VM with 10 GB GPU memory share of an NVIDIA A100 GPU:

* Constructing two models (by calling the `keras_cv.StableDiffusion` function twice) in the same Python session
* Attempting to generate an image with a high resolution (1024x1024) model; constructing the model itself was not a problem
* Attempting to generate a large number of images in a single pass (using a large batch size)

If you run into out of memory errors, either reduce the memory requirement of the model (for instance, by constructing the model at a lower resolution), or use a VM with more GPU RAM.

## 5. Licensing

Stability AI has released the model to the general public under a [*Creative ML OpenRAIL-M*](https://huggingface.co/spaces/CompVis/stable-diffusion-license) license. It permits commercial and non-commercial usage. The license is focused on ensuring ethical and legal usage of the model. It also mandates that the license itself be included with any distribution of the model, including to end users.
