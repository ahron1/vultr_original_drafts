# Build an Inference API using Hugging Face Transformers and FastAPI

## Introduction

Transformer based machine learning models are versatile and can be adapted to perform a broad range of tasks. In particular, LLMs with their language-related abilities are valuable for many use cases. You can [get started with LLMs using pre-trained models, such as GPT-J, Falcon, and the like](https://www.vultr.com/docs/how-to-use-hugging-face-transformer-models-on-vultr-cloud-gpu/). You can also [further train (fine-tune) a pre-trained model for a specific task](https://google.com).

Inference is the process of applying a model to input data to produce an output. To serve a model to users/customers over the web, build an inference API, and put the model into production. To build an inference API, you need:

* An AI model, such as an LLM (pre-trained or fine-tuned)
* A web framework for building and serving APIs
* A system that is configured to accept and serve requests over the internet

This article shows how to implement each of these steps and have a functional inference API.

### Scope and Prerequisites

This article shows how to build inference APIs for Hugging Face Transformer models. The examples are based on text generation using pre-trained GPT-J with 6 billion parameters. The example code is tested to run on a Debian server with 16 GB GPU RAM. 

It also briefly shows how to use the smaller GPT Neo 125M model. You can run this model with 1 GB GPU RAM. However, the output quality is noticeably worse when using smaller pre-trained models.

This article uses FastAPI to build the API interface. It uses [Gunicorn with Uvicorn workers](https://fastapi.tiangolo.com/deployment/server-workers/) to serve the API. Python-based frameworks such as Django are useful to build full-fledged web applications. To build only an API (not a full web app) using a Python-based framework, FastAPI is a better choice. Being a leaner and more focused tool, it is more efficient.

To follow the code samples, it is necessary to be familiar with [using Hugging Face Transformer models](https://www.vultr.com/docs/how-to-use-hugging-face-transformer-models-on-vultr-cloud-gpu/). It is also helpful to have a basic understanding of [REST APIs](https://www.vultr.com/docs/how-to-create-a-restful-api-using-python-and-fastapi-3398/) in general. In particular, an understanding of [FastAPI and Gunicorn based servers](https://www.vultr.com/docs/how-to-deploy-fastapi-applications-with-gunicorn-and-nginx-on-ubuntu-20-04/) is beneficial. To implement an API inference engine, you need to be familiar with intermediate Python programming. To connect to and test the API server, you need to be comfortable with the use of Postman, [cURL](https://www.vultr.com/docs/how-to-use-curl/), or another similar tool. To open the server to internet users, you need to know the [basics of firewalls](https://www.vultr.com/docs/how-to-configure-uncomplicated-firewall-ufw-on-ubuntu-20-04/). To edit text files, use a tool like Nano or [Vim](https://www.vultr.com/docs/getting-started-with-vim/). Use [Tmux](https://www.vultr.com/docs/how-to-install-and-use-tmux/) to run the code in one pane and simultaneously monitor the system in another pane. Tmux also preserves long running sessions in case the SSH connection gets dropped.

## Set up the System 

This section shows how to start from a fresh Debian system and set it up with the software required to run an inference API using Hugging Face transformer models. Install the tools to run the models and to serve the API. Run the commands in this section at the command prompt (terminal).

Install htop and Tmux:

    # apt install -y htop tmux

Download the Conda installer: 

    $ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

Run the installer and follow the on-screen instructions: 

    $ bash Miniconda3-latest-Linux-x86_64.sh

After the installation finishes, log out of the server and log back in. This activates Conda.

Upgrade Conda: 

    $ conda upgrade -y conda

Create a new Conda environment `env1` with Python 3.9:

    $ conda create -y --name env1 python=3.9 

Activate `env1`:

    $ conda activate env1

Upgrade `pip`:

    $ pip install --upgrade pip

Install the CUDA GPU packages using Conda:

    $ conda install -y -c conda-forge cudatoolkit=11.8 cudnn=8.2

Install Pytorch and related dependencies to run it on GPUs:

    $ conda install -y -c pytorch -c nvidia pytorch=2.0.1 pytorch-cuda=11.8 

Set the appropriate paths to initialize Conda:

    $ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CONDA_PREFIX/lib/

Create the activation directory:

    $ mkdir -p $CONDA_PREFIX/etc/conda/activate.d

Add the paths of Nvidia tools to the activation shell script (copy the commands individually):

    $ echo 'CUDNN_PATH=$(dirname $(python -c "import nvidia.cudnn;print(nvidia.cudnn.__file__)"))' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh

    $ echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CONDA_PREFIX/lib/' > $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh

Activate Conda:

    $ source $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh

Install [`tensorflow`](https://pypi.org/project/tensorflow/), [`transformers`](https://pypi.org/project/transformers/), [`huggingface-hub`](https://pypi.org/project/huggingface-hub/), Nvidia tools and, other dependencies like [`accelerate`](https://pypi.org/project/accelerate/):

    $ pip install nvidia-cudnn-cu11==8.6.0.163 tensorflow==2.12.* transformers==4.30.* huggingface-hub accelerate==0.20.3 xformers==0.0.20

Run a GPU based command in Python: 

    $ python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"

If the output of the above command resembles the line below, Python and Conda environments have access to the machine's GPU.

    [PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]

The above commands are based on the documentation of [Tensorflow](https://www.tensorflow.org/install/pip), [PyTorch](https://pytorch.org/get-started/locally/), and [Nvidia](https://docs.nvidia.com/deeplearning/cudnn/support-matrix/index.html). 

Install FastAPI and related tools:
    
    $ pip install fastapi==0.100.0 pydantic==1.10.4 "uvicorn[standard]"==0.22.0 gunicorn==20.1.0

FastAPI is the Python framework used to build the API application. Pydantic is a Python-based data validation library. It is used to implement custom data types. Uvicorn is a Python-based low-level web server for asynchronous applications. It is based on the Asynchronous Server Gateway Interface (AGSI) standard. Gunicorn is a Python based HTTP server based on the Web Server Gateway Interface (WGSI) standard - it serves the API. In principle, Uvicorn by itself can also serve the API. This article uses Gunicorn because it has better support to manage [workers](https://docs.gunicorn.org/en/latest/design.html).

## Inference API for Text Generation 

This section shows how to set up a basic API to serve a text generation model. It also introduces how to configure it for production use.

Create a new Python file `app.py`:

    $ touch app.py

Use `nano` or `vim` to edit this file and add the lines below.

Import the transformer:

    # import this transformer to run GPT J 6B
    from transformers import GPTJForCausalLM

    # import this transformer to run GPT Neo 125M
    # from transformers import GPTNeoForCausalLM 

Import torch, the tokenizer, and the pipeline packages:

    from transformers import pipeline, AutoTokenizer 
    import torch  

Import FastAPI:

    from fastapi import FastAPI 

Import `BaseModel` from the `pydantic` package to help with type checking:

    from pydantic import BaseModel 

FastAPI is an AGSI framework. Gunicorn, being a WGSI based framework, is not bundled together with AGSI workers. However, it can manage Uvicorn's AGSI workers. Import the package for Uvicorn workers:

    from uvicorn.workers import UvicornWorker

Define the tokenizer: 

    # use this tokenizer to run GPT J 6B
    tokenizer = AutoTokenizer.from_pretrained("EleutherAI/gpt-j-6b") 

    # use this tokenizer to run GPT Neo 125M
    # tokenizer = AutoTokenizer.from_pretrained("EleutherAI/gpt-neo-125m")  

Define the model:

    # use this model for GPT J 6B
    model = GPTJForCausalLM.from_pretrained("EleutherAI/gpt-j-6b", torch_dtype=torch.float16, low_cpu_mem_usage=True).cuda() 

    # use this model for GPT Neo 125M
    # model = GPTNeoForCausalLM.from_pretrained("EleutherAI/gpt-neo-125m", torch_dtype=torch.float16, low_cpu_mem_usage=True).cuda() 

Use Hugging Face pipelines to declare a text-generation pipeline using the GPT-J model defined above:
    
    generate_pipeline = pipeline(task="text-generation", model=model, device = 0, max_length=500, do_sample=True, num_return_sequences=1, tokenizer=tokenizer)

This defines a text generation pipeline. For other types of tasks (such as question answering), define the pipeline appropriately.

The next steps package this pipeline into an API endpoint. Instantiate the app:

    app = FastAPI() 

An API endpoint exposed to the internet should, at a minimum, implement type safety. Declare a class to specify the data type of the input prompt:

    class InputPrompt(BaseModel):     
        text:str  

The input can have a single string-type item with the key `text`.

Declare another class to specify the data type of the output (generated text):

    class GeneratedText(BaseModel):     
        text:str  

The output can have a single string-type item with the key `text`.

Create a POST endpoint, `generate`, for the app:

    @app.post("/generate", response_model=GeneratedText) 

The POST endpoint accepts the input user data as the body of the HTTP request. This endpoint responds with data of the type `GeneratedText` (declared earlier).

It is, in principle, possible to also use a GET endpoint. When using a GET endpoint, the input data is sent as query parameters. In practice, GET is not the right type of request for this application. Query parameters can only accept limited amounts of data. They also expose user data in the URL. It is advisable to use POST requests in production. The code samples in this article are based on POST requests.

For the POST endpoint, `generate`, declare a function, `generate_func` that processes the text entered by the user. This function accepts data of the type `InputPrompt` (declared earlier) as input. The input is in the form of a JSON object. It extracts the text field of the input and passes the string to the pipeline object `generate_pipeline` declared earlier. The output of the text generation pipeline contains a dictionary array. The generated text is the value of the key `generated_text`. Strip out the data structures and return just the generated text to the user.

    async def generate_func(prompt: InputPrompt):     
        output = generate_pipeline(prompt.text)          
        return {"text": output[0]["generated_text"]}

Save and close the file.

Run the new app with Gunicorn:

    $ gunicorn app:app --timeout 1000  --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8080

This starts up a server listening on the localhost at port 8080. It serves `app`, the FastAPI app declared in the `app.py` file. Larger models take longer to load into memory and GPU. Hence, it is necessary to supply a high value for the timeout option.

Test it using `curl` with POST data:

    $ curl -X 'POST' 'http://localhost:8080/generate' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"text": "my name is "}'

cURL sends the data as a JSON object. The option `-d` specifies the data field. The `accept` header specifies the data type the client can accept and understand. The `Content-Type` header specifies the data type of the request.

## Open the API Server to Internet Users

At this point, the server is running a functional inference API. However, external users over the internet cannot connect to it. By default, most firewalls limit incoming connections. Vultr's Debian instances come with UFW enabled. In the default configuration, the firewall only allows (SSH) connections on port 22. 

Check the status of UFW:

    # ufw status verbose numbered

To allow users to connect to the API server, open up port 8080:

    # ufw allow 8080

Connect to the inference API over the internet. From your local machine:

    $ curl -X 'POST' 'http://remote.server.ip.address:8080/generate' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"text": "my name is "}'

Using Postman, make a POST request to the endpoint http://remote.server.ip.address:8080/generate. In the request's body, specify the data to be raw JSON. Send a JSON object as the body: 

    {"text": "my name is "}
   
Press `CTRL`+`C` to stop the server. 

This section showed how to implement a basic inference API with type safety on the text generation pipeline. To run inference on other types of pipelines and models, modify the pipeline and type definitions appropriately. 

## Serve Fine-tuned Models

The process for serving fine-tuned models is the same as that for serving pre-trained models. Before serving a fine-tuned model, train and save it. [The article on fine-tuning discusses in detail how to do this](google.com). You can also download and save a fine-tuned model.

For example, instead of:

    # model = GPTJForCausalLM.from_pretrained("EleutherAI/gpt-j-6b", torch_dtype=torch.float16, low_cpu_mem_usage=True).cuda() 

Use the path to the saved fine-tuned model:

    # model = GPTNeoForCausalLM.from_pretrained("vultr/fine_tuned_gpt_neo_125", torch_dtype=torch.float16, low_cpu_mem_usage=True).cuda() 

The model path `vultr/fine_tuned_gpt_neo_125` in the above code sample is based on the examples in the [fine-tuning article](google.com).

## Multithreading

To run the server with multiple workers, invoke Gunicorn with the `--workers` option:

    $ gunicorn app:app --workers 2 --timeout 1000  --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8080

This forks the main thread into N processes. When unspecified, N defaults to 1. For each fork, an instance of the model is replicated in the GPU. Thus, if the model needs X GB GPU RAM to run, having N workers needs around X\*N GB of GPU. Each worker has its own memory space. 

After starting Gunicorn with multiple workers, check the output of `ps` to see the different forks:

    $ ps -ax | grep python

This should show several lines, each resembling:

    21895 pts/1    Dl+    1:08 /root/miniconda3/envs/env1/bin/python /root/miniconda3/envs/env1/bin/gunicorn app:app --workers 2 --timeout 1000 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8080

Each line, like the one above, is a fork of the main thread. Run `watch nvidia-smi` at the Linux terminal to monitor the GPU usage in real time. Verify that each of the forked processes occupies its own GPU memory space. Use the command `htop` (at the terminal) to monitor in real time the processes (threads) running on the system.

A common rule of thumb to decide the approximate number of workers, N, is:

    N = number of threads + 1

On VPSs, such as those from Vultr, 1 vCPU is equal to 1 thread. For modern dedicated machines that support hyperthreading, each CPU core runs 2 threads. Too few workers don't use the system's full capabilities. Having too many workers slows things down. Also consider if the amount of GPU memory is enough to run (not just load) N copies of the model. Spawning more workers than the GPU has space for gives an out-of-memory error. 

Sometimes, even with enough GPU, loading a large model with multiple workers throws warnings that workers are being terminated:

    [20153] [WARNING] Worker with pid 20154 was terminated due to signal 9

Check the output of `dmesg` (enter the command `dmesg` at the terminal):

    oom-kill:constraint=CONSTRAINT_NONE,nodemask=(null),cpuset=/,mems_allowed=0,global_oom,task_memcg=/user.slice/user-0.slice/session-3.scope,task=gunicorn,pid=20154,uid=0
    [ 4412.006670] Out of memory: Killed process 20154 (gunicorn) total-vm:38942992kB, anon-rss:30142812kB, file-rss:2304kB, shmem-rss:0kB, UID:0 pgtables:68700kB oom_score_adj:0

This implies loading the worker's objects (such as its copy of the model) led to an out of memory error and hence, the worker was terminated. In general, the system automatically spawns another process and recovers from this error. If the warnings persist, try increasing the timeout value.

Python functions can be defined using either `def` or `async def`. The example code uses `async def`. Using `async def` together with N Uvicorn workers creates N forks of the main thread. Incoming requests are distributed among these N processes. Because the function is asynchronous, it accepts new requests while the slow task (generating ML output) from a previous request is still processing. Each thread sequentially processes the requests assigned to it. Using `def` (instead of `async def`) creates a new thread (from a central thread pool) for each incoming request. Each thread runs in parallel and processes its request. Generating output from a machine learning model is a resource-intensive operation. Having too many concurrent threads, therefore, leads to resource contention and slows down the system.

To get a better understanding, test the performance (as described in the next section) of the system using both `async def` and `def`, and different numbers of workers. Consider using a smaller model, such as GPT-Neo-125m to study the performance differences. The same principles are generalizable to other models. Also refer FastAPI's documentation on [concurrency](https://fastapi.tiangolo.com/async/#concurrency-and-async-await) and [`async` functions](https://fastapi.tiangolo.com/async/#path-operation-functions).

## Performance Testing

Performance testing is a vast topic. This section only shows how to get a rough idea of how fast the API server responds to requests.

Use cURL to check how long the server takes to respond to a single HTTP POST request:

    $ curl -o /dev/null -X 'POST' -w 'Total: %{time_total}s\n' 'http://localhost:8080/generate' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"text": "my name is "}'

This command shows the total time taken to get a response from the server. To avoid clutter, it does not print the generated text on the screen. Run this command on the localhost (on the remote server). This helps in testing the response time (without the effect of network latencies) of the server. 

Test the responsiveness of the server to concurrent requests. Open a few (2-8) Tmux panes. Issue the above cURL command from each pane in quick succession. While the system is processing the requests, monitor the output of `htop` to observe the number of threads and CPU use. Note the time taken for each request to complete.

## Conclusion

This article gave an introduction to setting up an API server for running inference on Hugging Face Transformer models. It built a text generation API from scratch. It also discussed the code changes to run any pre-trained or fine-tuned model. 

Before serving an API to a wide user base, ensure to adequately address performance, security and [load balancing](https://www.vultr.com/docs/how-to-setup-load-balancing-using-nginx/) concerns. 

