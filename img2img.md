## 1. Introduction

Stable Diffusion is an open source deep learning model that generates new images given an input text prompt. It can also be used to modify images: the model can generate a new image from an input image and a text prompt.

This guide shows how to set up a system to generate new images based on an input image and a text prompt, using pre-trained Stable Diffusion models. It does not cover how to train models from data.

## 2. Compatibility

### 2.1. Hardware

The steps described here have been tested on a Vultr Cloud GPU instance with a 10 GB Video-RAM (V-RAM) share on an NVIDIA A100 GPU.

While it is possible to run Stable Diffusion on CPU-only machines, the processing times can be orders of magnitude higher in the absence of a GPU.

### 2.2. Base System

The system described here was set up on a VM running Debian 11. Operating System based commands are compatible with recent versions of Ubuntu. Python 3 commands are independent of the Operating System.

### 2.3. Version Numbers

Given the pace of change in the field of deep learning, software updates can sometimes break compatibility. Wherever necessary, the installation commands specify the version numbers of the software used. Using newer or older versions of the same software may lead to conflicts between dependencies.

## 3. System Setup

### 3.1. Install sshfs

After generating images (in Section 4), you will need a way to view them or to copy them to the local machine. `sshfs` can mount a remote directory on the local filesystem. You can then use the command line or the GUI to browse the remote directory mounted locally. You can also copy remote files to the local filesystem.

Check on the local machine if `sshfs` is installed:

    $ sshfs --version

If it isn't installed, install it:

    # apt install sshfs

Create a mountpoint on the local machine:

    $ mkdir ~/sshfs_mountpoint

Mount the server's home directory on this mountpoint:

    $ sshfs -o follow_symlinks username@remote.server.ip.address:/home/username ~/sshfs_mountpoint

### 3.2. Get Access Codes

Before being able to use pre-trained Stable Diffusion models, you need to be able to access them. These models are available from Hugging Face - a data science community and platform (hub) for sharing machine learning tools and models. To use any of the pre-trained models, you need an access token. As of November 2022, it is free to join the hub and create access tokens.

Sign up for an account on the [Hugging Face website](https://huggingface.co) using your email address. Follow the on-screen instructions and create your account. You need to confirm your email address.

After signing in with a confirmed account, navigate to the Settings page (click on the profile picture icon at the top right of the screen and click on Settings from the drop-down options). On the [Settings page](https://huggingface.co/settings/profile), navigate to Access Tokens settings (click on Access Tokens in the vertical menu on the left). On the [Access Tokens page](https://huggingface.co/settings/tokens), use the "New token" button to create a new token. Give it a relevant name and choose "read" for the role - you only need to read the pre-trained model.

You need this access code in Section 4.

### 3.3. Set up Development Environment

Stable Diffusion sessions can be long-running. To avoid losing work in case the SSH session disconnects, use either [TMUX](https://www.vultr.com/docs/how-to-install-and-use-tmux/) or [Screen](https://www.vultr.com/docs/an-introductory-guide-to-screen/).

#### 3.3.1 Create New User

By default, on a new Vultr instance, you log in as root. It is not advisable to install Python packages as root. Before continuing further, [set up a new unprivileged user and configure SSH access for that user](https://www.vultr.com/docs/initial-secure-server-configuration-of-ubuntu-18-04/).

Further steps assume that you are logged in as a regular, unprivileged user.

#### 3.3.2 Install Conda

*Environments* help to isolate software used by different projects by installing them into separate directory trees. This guide uses Conda to manage Python environments.

Miniconda is a minimal Conda installer. Visit the  [Conda website](https://docs.conda.io/en/latest/miniconda.html) and copy the download link for the latest Miniconda release. Download the installation file:

    $ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

Run the installation file to install Miniconda:

    $ bash Miniconda3-latest-Linux-x86_64.sh

Follow all on-screen instructions during the installation process.

Check that Conda is installed:

    $ conda --version

#### 3.3.3. New Conda Environment

Upgrade `conda` to the latest version:

    $ conda upgrade conda

Create a new Conda environment:

    $ conda create --name env1 python=3.9

This creates a new environment `env1` and installs into it Python 3.9 and `pip`. Follow the on-screen instructions during the installation process.

Enter `conda env list` at the command line to view the list of Conda environments. Activate the new Conda environment:

    $ conda activate env1

To deactivate the currently active environment and get back to the regular command line, use `conda deactivate`.

All further steps assume that the Conda environment `env1` is active.

Export an environment variable containing the Conda installation path:

    $ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CONDA_PREFIX/lib/

After exporting the path, you no longer need to type out the full paths of the applications installed into the environment.

Create an activation directory within the Conda directory:

    $ mkdir -p $CONDA_PREFIX/etc/conda/activate.d

In a new file `env_vars.sh` within the activation directory, add a command to export the variable with the Conda installation path:

    $ echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CONDA_PREFIX/lib/' > $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh

This automatically loads the path at the start of every Conda session.

### 3.4. Install Python Packages - Conda

#### 3.4.1. PyTorch and CUDA

PyTorch (`torch`) is a Python library that enables tensor computations and includes tools for processing neural networks. PyTorch modules work on both GPU and CPU. On systems with GPU support, Torch takes advantage of the GPU for accelerating tensor computations.

CUDA is a "parallel computing platform and programming model" developed by NVIDIA for its own GPUs. `cudatoolkit` contains the runtime support for CUDA. As of November 2022, `cudatoolkit` is not available to install via PyPI. Installing PyTorch from PyPI and CUDA from Conda can lead to version and GPU-driver-architecture related incompatibilities. To ensure compatibility of GPU libraries, install both Pytorch and CUDA from Conda:

    $ conda install pytorch=1.12.1 cudatoolkit=11.6 -c pytorch -c conda-forge

#### 3.4.2. Quick Check

Before continuing with installing other packages, check that PyTorch has been installed with CUDA support. Note that you might need to deactivate and re-activate the Conda environment `env1`. Start a Python shell and import `torch`:

    >>> import torch

Check the CUDA version in `torch`:

    >>> torch.version.cuda

The output should be a version number, resembling `11.6`.

Check the availability of CUDA support in the PyTorch installation:

    >>> torch.cuda.is_available()

The output should be `True`.

Exit the Python shell and continue to install the remaining packages.

#### 3.4.3. Other Versions

To install other versions of PyTorch and CUDA, choose the appropriate versions based on the GPU drivers installed on your computer. You might need to do this if, for instance, the pre-installed NVIDIA drivers are updated. When this happens, you can get error messages that say the CUDA architecture is incompatible with the current PyTorch installation. You might also get errors if your computer uses a different GPU than the one this guide was tested on (NVIDIA A100).

Check the highest CUDA version that can be supported by the installed drivers:

    $ nvidia-smi

The first line of the output should look like this:

    NVIDIA-SMI 510.85.02    Driver Version: 510.85.02    CUDA Version: 11.6 

The top right corner of the output shows the highest supported CUDA version.

Check the compatibility information on the [PyTorch website](https://pytorch.org/get-started/previous-versions/) to determine suitable versions of PyTorch and CUDA for your computer. If needed, use those versions instead of the ones specified in Section 3.4.1.

### 3.5. Install Python Packages - PyPI

Upgrade to the latest version of `pip`:

    $ pip install --upgrade pip

The packages below can also be installed from Conda, but the installation process is faster with `pip`.

#### 3.5.1. Python Imaging Library

Pillow is the actively maintained (as of November 2022) fork of the Python Imaging Library, PIL, which is no longer maintained. Install Pillow:

    $ pip install --upgrade Pillow

#### 3.5.2. Diffusers

Machine Learning *pipelines* are a convenient way of bundling the entire data processing workflow to construct a model.

Diffusers provides access to pre-trained diffusion models in the form of prepackaged pipelines. It provides the tools for building and training diffusion models. Diffusers also includes a number of different core neural network models - these are used as building blocks to create new pipelines.

Since August 2022, Stable Diffusion pipelines have been available via Diffusers. Install `diffusers`:

    $ pip install --upgrade diffusers==0.7.2

#### 3.5.3. Transformers

Built by the Hugging Face team, Transformers is a library of many pre-trained machine learning models. These models are based on TensorFlow and PyTorch. Transformers models can be used as the building blocks while constructing a new pipeline. Install Transformers:

    $ pip install --upgrade transformers==4.24.0

## 4. Generate Images

### 4.1. Access Code

Open a new Python command line. Copy your Hugging Face access code and assign it to a variable:

    >>> hf_token = 'hf_copythetokenfromyourhuggingfaceaccount'

### 4.2. Import Packages

Import `torch`:

    >>> import torch

The `Image` module of Pillow is used for loading images from files and for saving new images. Import the `Image` module:

    >>> from PIL import Image

Many deep learning pipelines, including different Stable Diffusion pipelines, are available via Diffusers. The Stable Diffusion Img2Img pipeline includes all the processing steps to generate images given an input image and an input text prompt. Import the Img2Img pipeline:

    >>> from diffusers import StableDiffusionImg2ImgPipeline

### 4.3. Load the Pipeline

The `from_pretrained()` function loads all the components of a pre-trained pipeline, including model weights.

    >>> pipe = StableDiffusionImg2ImgPipeline.from_pretrained(
        "CompVis/stable-diffusion-v1-4",
        revision="fp16", 
        torch_dtype=torch.float16, 
        use_auth_token=hf_token
    )

 The parameters passed to the `from_pretrained()` function are:

* the model id of a pipeline. The function call above loads the `stable-diffusion-v1-4` model hosted under the CompVis organization's repo. The model id can also be the path to a local directory containing model weights, or a path (local or URL) to a checkpoint file.
* `revision` denotes the specific version of the model whose repo id is used. This is a git-based branch name, tag name, or commit id.
* `torch_dtype` is the Torch datatype of the tensors used for pipeline computations. `float16` is specified explicitly so that the model computations are done in 16-bit floating point numbers (instead of the default 32-bit). This helps on systems with limited memory. It is possible to let the system chose the optimal data type using `torch_dtype = "auto"`. The default setting is `torch_dtype = "full-precision"`.
* `use_auth_token` specifies an access token to download the pre-trained model from the Hugging Face repo.

By default, the `from_pretrained()` function above loads the model into RAM (i.e., the memory space that the CPU has access to). The GPU performs tensor computations much faster than the CPU. But the GPU does not have access to the RAM, it can only access its own memory. So, the model needs to be "moved" to the GPU. The `Tensor.to()` function is used to do this. Move the model to the GPU device (in this case, the GPU device is CUDA):

    >>> device = "cuda"
    >>> pipe = pipe.to(device)

### 4.4. Load the Input Image

#### 4.4.1. Load an Image from the Filesystem

If the input image is already present on the server, load it in the Python terminal:

    >>> input_image = Image.open('/path/to/image')

#### 4.4.2. Copy Local Image to Server

If the image is on the local machine, copy it onto the server using `scp` (on the local machine):

    $ scp /local/path/to/image username@server.ip.address:/remote/path/to/image

After the image has been copied onto the server, load it in the Python terminal as in the previous subsection:

    >>> input_image = Image.open('/path/to/image')

### 4.5. Resize Input Image

It is advisable to resize the input image to a width of 512 pixels.

    >>> height_orig = input_image.height
    >>> width_orig = input_image.width
    >>> aspect_ratio = width_orig / height_orig 

    >>> width_new = 512
    >>> height_new = int(width_new / aspect_ratio)

    >>> input_image = input_image.resize((width_new, height_new), 0)

Stable Diffusion models have been trained on images of 512x512 resolution. It is recommended that at least one of the dimensions of the input image be 512 pixels. Trying to generate an image of resolution greater than 512 along both axes can lead to repetition of regions of the image. Note that trying to process an image of resolution 1024x1024 on a system with a 10 GB share of V-RAM (on a A100 GPU) can lead to out-of-memory errors.

### 4.6. Text Prompt

Create an input text prompt:

    >>> prompt = "wearing a christmas hat"

### 4.7. Generate New Images

#### 4.7.1. Default Settings

Generate a single new image using the default settings on the loaded pipeline:

    >>> pipe_output = pipe(
        prompt = prompt,
        init_image = input_image,
    )

In the above function call, the parameter `prompt` is the input text prompt and `init_image` is the input image, as defined earlier.

Extract the images from the output object:

    >>> output_images = pipe_output.images

`output_images` is an array of images.

Save the image:

    >>> output_images[0].save("/path/to/image/filename.png")

View the images using `sshfs`, as described in Section 3.1.

#### 4.7.2. Advanced Settings

The image generation pipeline accepts a number of additional options. Only the parameters `prompt` and `init_image` are mandatory.

    >>> pipe_output = pipe(
        prompt = prompt,
        init_image = input_image,
        strength = 0.75,
        num_inference_steps = 100,
        guidance_scale = 7.5,
        negative_prompt = negative_prompt,
        num_images_per_prompt = 5
    )

The above call to the pipeline function uses the following parameters:

* `prompt` is the input text prompt. It is either a single text string or an array of strings. Note that using an array of strings leads to greater V-RAM requirements. In general, the more complex the prompt, the more V-RAM is needed to process it.
* `init_image` is the input image to be used as the starting point for generating the new image.
* `strength` is a value between 0 and 1. It indicates the amount of transformation that the input image can be subject to. The higher this value, the more the input image is changed. The default value is 0.8. When `strength` is set to 1, the input image plays almost no role in generating the new image.
* `num_inference_steps`: Stable Diffusion starts with random noise and repeatedly *de-noises* it to generate a meaningful picture. `num_inference_steps` denotes the number of *de-noising*  steps. The larger this value, the higher the quality of the output image(s). The default value is 50. The actual number of steps is modulated by the `strength` parameter. When the `strength` parameter has a value less than 1, the actual number of iterations is less than the number specified in `num_inference_steps`.
* `guidance_scale`: Guidance refers to the extent to which the input text prompt influences the image generation. It can be enabled by setting `guidance_scale` to a value greater than 1. The higher this value, the more closely the generated image conforms to the input text prompt. However, the more the generated image matches the input text prompt, the lower the quality of the image. The default value is 7.5.
* `negative_prompt`: If the prompt guides what the image *should be*, the negative prompt guides what it *should* ***not*** *be*. The negative prompt applies only if the guidance scale value is greater than 1.
* `num_images_per_prompt` specifies the number of images to generate. Note that trying to generate a large number of images per function call can lead to out of memory errors. If you need to generate more images, call the function repeatedly.

Extract and view the images as in the previous section.

The default values mentioned above are true as of November 2022 and might change in the future as models are updated.

#### 4.7.3. Remarks

The extent to which images generated by the model are "accurate" depends on the extent to which the model was trained on related images and their associated text tags. It also depends on the configuration of the parameters described in this section. It can take some trial and error to figure out a good set of parameter values.

#### 4.7.4. NSFW Images

Stable Diffusion has a built-in NSFW filter, and it is turned on by default. The `pipe_output` object (as in Section 4.7.1) has a list `nsfw_content_detected`. It can be accessed as `pipe_output.nsfw_content_detected`. If NSFW content is detected in an image, it is displayed as a black image. It is possible to turn off the NSFW filter. Details of how to do this are not discussed here.

## 5. Licensing

Stability AI has released the model to the general public under a [*Creative ML OpenRAIL-M*](https://huggingface.co/spaces/CompVis/stable-diffusion-license) license. It permits commercial and non-commercial usage. The license is focused on ensuring ethical and legal usage of the model. It also mandates that the license itself be included with any distribution of the model, including to end users.

## 6. Conclusion

This guide gives an overview of how to set up a Stable Diffusion system to generate new images from a given image and a text prompt. The observations made here are accurate as of November 2022. Future developments and software upgrades can render them partially inaccurate. It is common to come across bugs and inconsistencies in the model behavior across different versions. It is helpful to be able to read and understand error messages while configuring the system. The [model card page on the Hugging Face website](https://huggingface.co/CompVis/stable-diffusion-v1-4) and the [Stable Diffusion blog](https://huggingface.co/blog/stable_diffusion) have complete up-to-date details of the model.
