# How to Use Hugging Face Transformer Models on Vultr Cloud GPU

## Introduction

Transformers are a type of neural network and they are used for deep learning. Transformers are often referred to as foundation models. Foundation models are trained using self-supervised learning or unsupervised learning on large amounts of data. They can, therefore, be adapted (fine-tuned) to a range of different tasks. 

In particular, transformers have proved useful for Natural Language Processing (NLP) applications. Transformers which are trained on large volumes of raw text are capable of various language-related tasks, such as sentence completion, summarization, and so on. Such transformers are called Large Language Models (LLMs).

### Scope and Prerequisites

This article shows how to use Transformer models, including some recent and popular LLMs.

To follow the examples in this article, you are expected to be comfortable with basic Python programming. It is assumed you have access to a computer with a GPU chip or a [cloud server with GPU capabilities](https://www.vultr.com/products/cloud-gpu/). It is possible to run these models on a CPU-only machine. However, the processing times are often much higher in the absence of a GPU. 

The examples in this article are tested on freshly installed Debian 11 servers with NVIDIA A100 and A40 GPUs. They should work on all recent Linuxes with NVIDIA GPUs.

## Using Transformers 

Transformer models are typically trained on large datasets. The training process can take days and require a large amount of GPU RAM. This article does not cover how to train a transformer model. 

Pre-trained models, being trained on large datasets, are good general-purpose models. They perform well on many tasks but do not excel at any specific task. This article shows how to use pre-trained general-purpose transformer models. 

Pre-trained models can be further trained on specific tasks - this process is called fine-tuning. Fine-tuning is not covered in this article.

### Hugging Face

The examples in this article are based on downloading the models and model components from Hugging Face. 

[Hugging Face](https://huggingface.co) is 1) a platform for (studying and downloading) data science and machine learning tools and models, and 2) a community of data science professionals. It also has many learning materials. The "model card" page of each model describes the scope, parameters, and specifications of the model. It also contains code samples for using it. 

## Getting Started

This article assumes you start from a fresh Debian machine. This section shows how to set up the system before running the models. All the commands in this section are to be run on command prompt (terminal).

Download the Conda installer: 

    $ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

Run the installer and follow the on-screen instructions: 

    $ bash Miniconda3-latest-Linux-x86_64.sh

After the installation finishes, log out of the server and log back in to activate Conda.

Upgrade the installed version of Conda: 

    $ conda upgrade -y conda

Create a new Conda environment `env1` with Python 3.9:

    $ conda create -y --name env1 python=3.9 

Activate the new Conda environment:

    $ conda activate env1

Upgrade `pip`, the Python installer:

    $ pip install --upgrade pip

Using Conda, install the CUDA GPU packages:

    $ conda install -y -c conda-forge cudatoolkit=11.8 cudnn=8.2

Using Conda, install Pytorch and related dependencies to run it on GPUs:

    $ conda install -y -c pytorch -c nvidia pytorch pytorch-cuda=11.8 

Set the appropriate paths to initialize Conda:

    $ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CONDA_PREFIX/lib/

Create the activation directory:

    $ mkdir -p $CONDA_PREFIX/etc/conda/activate.d

Add the paths of Nvidia tools to the activation shell script (copy the below commands individually):

    $ echo 'CUDNN_PATH=$(dirname $(python -c "import nvidia.cudnn;print(nvidia.cudnn.__file__)"))' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh

    $ echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CONDA_PREFIX/lib/' > $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh

Activate Conda:

    $ source $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh

Using Pip, install [`tensorflow`](https://pypi.org/project/tensorflow/), [`transformers`](https://pypi.org/project/transformers/), [`huggingface-hub`](https://pypi.org/project/huggingface-hub/), Nvidia tools and, a few other dependencies like [`einops`](https://pypi.org/project/einops/), [`accelerate`](https://pypi.org/project/accelerate/) and, [`xformers`](https://pypi.org/project/xformers/):

    $ pip install nvidia-cudnn-cu11==8.6.0.163 tensorflow==2.12.* transformers huggingface-hub einops accelerate xformers 

Check that Python is installed with GPU support:

    $ python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"

The output of the above command should resemble:

    [PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]

This shows that the Python and Conda environments have access to the machine's GPU.

Most of the above commands are based on the documentation of [Tensorflow](https://www.tensorflow.org/install/pip), [PyTorch](https://pytorch.org/get-started/locally/), [Nvidia](https://docs.nvidia.com/deeplearning/cudnn/support-matrix/index.html) and, the different models used as examples.

Start a Python shell

    $ python

## Using the Models

Pipelines are high-level tools that package the necessary components needed to perform different predefined *tasks*. Examples of tasks include *text-generation*, *question-answering*, *sentiment-analysis*, and so on. Pipelines can be run by just specifying a *task* and letting it use the default settings (for that task) for everything else. It is also possible to custom build a pipeline by specifying the model, tokenizer, and other parameters. The examples below cover both approaches.

The examples in this section are based on text generation models/tasks. Each example also mentions the amount of GPU RAM used to run the model. More complex tasks (such as generating longer text) use more memory. Before loading the models in each example, it is recommended to close and reopen the Python shell. This helps in clearing out the old models from memory and freeing up space for new models.

Note that all the commands in this section are to be run at the Python prompt.

### Pipelines with Default Models

Pipelines can automatically choose a default model for the given task. In general, the default models tend to have low memory and storage requirements. This makes them handy as a learning tool.

Before using the pipeline module, import it:

    >>> from transformers import pipeline

The syntax for using a pipeline's default model is:

    >>> # pseudo-code: my_pipeline = pipeline(task="task_name")

`task_name` can be one of the following values:

1. *audio-classification*
1. *automatic-speech-recognition*
1. *image-classification*
1. *object-detection* 
1. *image-segmentation* 
1. *depth-estimation* 
1. *sentiment-analysis* 
1. *ner* (Named Entity Recognition)
1. *question-answering*
1. *summarization* 
1. *translation* 
1. *text-generation*
1. *fill-mask* 

The [Hugging Face Pipelines documentation](https://huggingface.co/docs/transformers/task_summary) has a full description of the different tasks.

#### Example Tasks

For example, to use a pipeline for *text-generation*, the command is:

    >>> my_text_generator = pipeline("text-generation", device=0)

By default, this task downloads OpenAI's [GPT-2 model ](https://huggingface.co/gpt2), which takes over 500 MB of storage. Loading the model into memory takes around 1.5GB of GPU RAM. Using the pipeline (shown below) to generate text can consume over 2 GB of GPU RAM.

Use this pipeline as follows:

    >>> my_text = "Vultr is a cloud service provider"
    >>> my_text_generator(my_text, max_length=200)

Note that as of June 2023, OpenAI's GPT-3 and GPT-4 models are not available to use on HuggingFace.

### Custom-built Pipelines

The previous section showed how to specify just a task and let Pipelines pick the default model for that task. It is also possible to specify a particular model to be used for the task. Before using a particular model for a specific task, verify that the model can indeed be used for that task. The *model-card* page of most models describes what it can be used for.

The syntax for declaring a pipeline to do a given task with a specific model is:

    >>> # pseudo-code: my_pipeline = pipeline(task="task_name", model="model_name")

In addition to the model, it is also possible to specify other parameters to construct the pipeline. The examples below show how to do this.

#### Falcon-7B

The Technology Innovation Institute (TII) sponsored the development of new LLMs, like Falcon, built from scratch. The Falcon model has two variants - Falcon-7B and Falcon-40B. They are both pre-trained models. So they perform reasonably well on a broad range of text-based tasks. 

Falcon-7B is based on 7 billion parameters. The model occupies around 14GB of storage. 

Loading it into memory (by creating a pipeline) with 16-bit weights consumes around 14GB of GPU RAM. Running tasks using this model has a memory footprint of over 15GB. Therefore, it is advisable to run this model on a system with over 16GB GPU RAM. Loading the model using the default 32-bit floats takes over 22 GB of GPU. Running it can take more than 25 GB GPU RAM.

To use the model, import the following packages:

    >>> from transformers import AutoTokenizer, AutoModelForCausalLM
    >>> import transformers, torch

Declare the model name with a variable:

    >>> model = "tiiuae/falcon-7b"

Initialize the tokenizer corresponding to the model:

    >>> tokenizer = AutoTokenizer.from_pretrained(model)

Declare the pipeline with 16-bit weights:

    >>> pipeline = transformers.pipeline(
        "text-generation",
        model=model,
        tokenizer=tokenizer,
        torch_dtype=torch.bfloat16,
        trust_remote_code=True,
        device_map="auto",
    )

**Note:** The above pipeline loads model weights as 16-bit floats. To load the weights in 32-bit, omit the line `torch_dtype=torch.bfloat16,` in the above code snippet. Using 32-bit weights gives somewhat better results, at the cost of using more memory.

Use the declared pipeline to generate text based on an input prompt:

    >>> sequences = pipeline("Vultr is a cloud service provider", do_sample=True)

See the contents of the generated text:

    >>> for seq in sequences:
            print(f"Result: {seq['generated_text']}")

By default, the generated text is not long. To enhance the generated text, specify a few additional parameters to the pipeline function call:

    >>> sequences = pipeline(
           "Vultr is a cloud service provider",
            max_length=200,
            do_sample=True,
            num_return_sequences=1,
        )

Check again the generated text:

    >>> for seq in sequences:
            print(f"Result: {seq['generated_text']}")

### Falcon 40B

Falcon-40B is based on 40 billion parameters. The model itself occupies around 76GB of storage. Loading it into memory consumes around 76GB of GPU RAM. A system with single A100 80GB GPU is **insufficient** to run this model (after loading it into memory). Therefore, it is advisable to run this model on a system with two A100 or A40 GPUs.

The steps for Falcon 40B are almost the same as for Falcon 7B. In the code sample in the previous section, replace the model name. Instead of `model = "tiiuae/falcon-7b"`, use `model = "tiiuae/falcon-40b"`.

### GPT-J

EleutherAI is a non-profit AI research lab that started in 2020. Their work is primarily focused on LLMs. Their landmark model, GPT Neo (last updated around mid-2021) is an open-source transformer model based on the (OpenAI's) GPT architecture. GPT-J is the successor to GPT Neo. 

GPT-J is a class of models. GPT-J-6B is a specific GPT-J model with 6 billion parameters. It is a large model. The default 32-bit variant takes around 24GB of GPU RAM. The 16-bit variant (`float16`) has a smaller memory footprint of around 12 GB. Using this variant in a pipeline uses over 12 GB of GPU RAM.

Import the required packages:

    >>> from transformers import GPTJForCausalLM, pipeline, AutoTokenizer
    >>> import torch

Specify the model parameters with 16-bit weights: 

    >>> model = GPTJForCausalLM.from_pretrained(
            "EleutherAI/gpt-j-6B",
            revision="float16",
            torch_dtype=torch.float16,
            low_cpu_mem_usage=True
        )

**Note:** To load the default 32-bit variant of GPT-J-6B, in the model specification above, omit the lines `revision="float16"` and `torch_dtype=torch.float16`.

Initialize the tokenizer:

    >>> tokenizer = AutoTokenizer.from_pretrained("EleutherAI/gpt-j-6B") 

Create the pipeline based on the model and tokenizer declared earlier:

    >>> gen_gptj = pipeline(task="text-generation",
            model=model,
            tokenizer=tokenizer,
            device=0,
            max_length=200
        )

Use the pipeline:

    >>> gen_gptj("Vultr is a cloud service provider")

This outputs the generated text-based on the input prompt.

## Hardware Considerations

On most Linux systems, by default, the model files are stored in the `~/.cache/huggingface/hub` directory. To get an idea of the disk space consumed by downloaded models, use this command at the Linux terminal:

    $ du -h -d 1 ~/.cache/huggingface/hub/

Below is an excerpt of the output of the above command from a test machine. It shows the storage requirements of some common models.

    256M    /root/.cache/huggingface/hub/models--distilbert-base-uncased-finetuned-sst-2-english
    460M    /root/.cache/huggingface/hub/models--openai-gpt
    503M    /root/.cache/huggingface/hub/models--EleutherAI--gpt-neo-125M
    2.0G    /root/.cache/huggingface/hub/models--bigscience--bloomz-1b1
    5.0G    /root/.cache/huggingface/hub/models--EleutherAI--gpt-neo-1.3B
    12G     /root/.cache/huggingface/hub/models--EleutherAI--gpt-j-6B
    14G     /root/.cache/huggingface/hub/models--tiiuae--falcon-7b
    78G     /root/.cache/huggingface/hub/models--tiiuae--falcon-40b

Before the model is used, it is (automatically) "unpacked" in memory. So the amount of memory (RAM) required to run a model is more than the storage space it consumes. Decide the amount of hardware resources based on the size of the model you want to run. Note also that the amount of GPU memory needed depends on the work the model has to do. For example, using a text-generation pipeline with the `max_length` parameter set to 200 will use less memory compared to setting the `max_length` parameter to 2000.

If the server has insufficient memory to handle the model, the process terminates with an error.

To monitor CPU and memory usage, use a tool like `htop`. 

To monitor GPU usage, use `watch` to fetch the output of `nvidia-smi` every 1 second:

    $ watch -n 1 nvidia-smi

## Conclusion

This article is a guide to start using transformer models from Hugging Face. It shows how to use some recent models. It also informs of hardware resource considerations for running large models. To use a new model, look at its "model card" page to learn to use it. To use a specific model in practice, study the details and configuration options of that model from the model documentation.

### Caveats

LLM models are undoubtedly powerful. However, they are not perfect and cannot be used blindly. It is critical to verify their output. In particular, "language models" have no notion of "facts". The prompt passed to the text-generation models in the above examples is *Vultr is a cloud service provider*. From the perspective of the model, this is equivalent to *FooBar is a cloud service provider*. Observe carefully the output text generated by the model - you will notice coherent on-topic sentences with factual errors. This is called "hallucination". It is expected that future models will address this significant shortcoming.

