# Important AI / ML Packages

Machine Learning (ML) and Artificial Intelligence (AI) are vast and complex fields. They involve the use of many software packages. These packages offer the developer a set of ready-to-use programs for specific tasks. Thus, the developer can focus on the modeling task at hand instead of writing boilerplate code and implementing complex algorithms from scratch. 

This article gives an overview of some commonly used software packages in ML and AI applications. 

## Frameworks and Libraries

### TensorFlow

[TensorFlow](https://www.vultr.com/docs/how-to-install-tensorflow-on-ubuntu-22-04-79647/) is an open-source library of tools for both machine learning and deep learning. It works on CPUs, GPUs, and TPUs. TensorFlow started as an internal Google project and Google continues to maintain it. Developers use TensorFlow for a variety of applications, such as: 

 * Traditional ML problems, like classification and regression
 * Generative AI applications, like Stable Diffusion
 * Computer Vision (CV) models, such as object recognition
 * Deploying models to production via TensorFlow Serving

The following pages discuss example applications that use TensorFlow:

 * [Implement Stable Diffusion with TensorFlow and Keras](https://www.vultr.com/docs/implement-stable-diffusion-with-keras-and-tensorflow/)
 * [Build a Discord Chatbot with TensorFlow](https://www.vultr.com/docs/build-a-discord-chatbot-with-tensorflow/)
 * [Build a Question Answering Engine Using TensorFlow](https://www.vultr.com/docs/how-to-use-bert-question-answering-in-tensorflow-with-python/)
 * [Implement an Object Detection Model using Keras and TensorFlow](https://www.vultr.com/docs/object-detection-with-tensorflow-on-a-vultr-cloud-server/)
 * [Deploy a Machine Learning Model in Production with TensorFlow Serving](https://www.vultr.com/docs/deploy-a-machine-learning-model-to-production-with-tensorflow-serving/)

### PyTorch

[PyTorch](https://www.vultr.com/docs/how-to-install-pytorch-on-ubuntu-22-04/) is an open-source ML framework. Facebook (Meta) initially led PyTorch development. Since 2022, the Linux Foundation has been responsible for PyTorch. It is based on Torch, which is an open-source ML library. It also includes many tools used to build deep learning models. PyTorch packages work on both GPU and CPU. PyTorch use-cases include:

 * Working with large datasets
 * Multimodal models, such as handwriting recognition
 * Building deep learning models
 * Deploying deep learning models in production, using TorchServe

To learn about practical projects that use PyTorch, visit these pages:

 * [Deploy a PyTorch Workspace](https://www.vultr.com/docs/deploy-a-pytorch-workspace-on-a-vultr-cloud-gpu-server/)
 * [Use WebDataset with PyTorch](https://www.vultr.com/docs/how-to-use-webdataset-and-pytorch-with-vultr-object-storage/)
 * [Perform Neural Style Transfer with PyTorch](https://www.vultr.com/docs/how-to-perform-neural-style-transfer-with-python3-and-pytorch-on-windows/)

### PyTorch and TensorFlow

At a high level, PyTorch and TensorFlow serve similar purposes. TensorFlow had dominated the ML landscape in its early years. However, since around 2020, PyTorch has been used more in research projects. Many new projects also prefer to use PyTorch. Most new models added on Hugging Face are based on PyTorch. In contrast, because of its early adoption, TensorFlow is used more in commercial applications. 

### PyTorch Lightning

[PyTorch Lightning](https://lightning.ai/docs/pytorch/stable/) is built on top of the tools available in PyTorch. From the developer's perspective, PyTorch Lightning is a high-level interface for raw PyTorch. The advantage of using PyTorch Lightning is it abstracts away the technical implementation details, such as multi-GPU, TPUs, 16-bit and 32-bit precision floats, and so on. PyTorch Lightning is used for applications like:

 * Training ML models on CPU/GPU-clusters
 * Evaluating model performance
 * Improving model performance by discovering bottlenecks
 * Reducing model size by using quantization

### Keras

[Keras](https://keras.io/) is a library of components and functions used for deep learning projects. It can be considered a high-level neural-network-focussed API based on TensorFlow. It packages many lower-level TensorFlow functions into single API endpoints. Keras includes modules to implement different types of neural network layers and to combine layers to create models. It also allows developers to choose from different optimizers, activation functions, and evaluation metrics. Keras is used for different applications, such as:

 * NLP projects, like sentiment analysis
 * Deep learning projects, like generative AI
 * Implementing deep learning tools for mobile devices
 * Running deep learning models on web browsers 

Visit these pages to see applications that use Keras in practice: 

 * [Implement Stable Diffusion using Keras and TensorFlow](https://www.vultr.com/docs/implement-stable-diffusion-with-keras-and-tensorflow/)

### KerasCV

[KerasCV](https://keras.io/keras_cv/) is a horizontal extension of the core Keras library. Specialized modules for Computer Vision (CV) are not relevant to all Keras users. Thus, these modules are separately packaged as KerasCV. It also allows access to many pre-trained models, such as Stable Diffusion and Vision Transformer.

KerasCV is used for:

 * Computer Vision applications like object detection and image segmentation
 * Generative AI, specifically image-processing applications

The following articles showcase examples of applications that use KerasCV:

 * [Implement Stable Diffusion using Keras and TensorFlow](https://www.vultr.com/docs/implement-stable-diffusion-with-keras-and-tensorflow/)

### NumPy

[NumPy](https://numpy.org/) is an open-source numerical computing library for Python. Developers use NumPy to perform mathematical operations like arithmetic, trigonometry, and complex numbers. Additionally, it also supports linear algebra operations on high-dimensional vectors and tensors. Many complex NumPy modules are based on optimized pre-compiled C programs. This results in efficient performance on CPUs. However, NumPy doesn't directly work on GPUs, TPUs, or multi-CPU clusters. 

NumPy is commonly used for applications such as:

 * Machine Learning
 * Generative AI
 * Recommendation systems
 * Imaging applications, like image classification and segmentation

For examples of applications that use NumPy, visit the following resources: 

 * [How to Use Stable Diffusion to Manipulate Images](https://www.vultr.com/docs/ai-image-manipulation-with-stable-diffusion-controlnet-on-vultr-cloud-gpu/)

### SciPy

[SciPy](https://scipy.org/) is an open-source Python library with modules for scientific computing. For many functions, the Python interface calls underlying pre-compiled code written in C, C++, and Fortran. It can be viewed as an extension of NumPy. It builds upon the basic mathematical tools of NumPy and extends their functionality to science-specific problems. SciPy is used for low-level scientific computation tasks like numerical integration and differentiation, differential equations and eigenvalue problems, signal processing, Fourier Transforms, and so on.

SciPy is used for applications like: 

 * Statistical analyses
 * Industrial optimization problems
 * Audio processing
 * Scientific computing applications, like modeling and simulations

### JAX and Flax

Google Brain introduced [JAX](https://github.com/google/jax) as a high-performance library for numerical computations. It was envisioned as a more advanced variant of NumPy. JAX and NumPy have similar syntax and structure. Unlike NumPy, JAX directly supports GPUs and TPUs. JAX supports complex low-level functions such as automatic and higher-order differentiation and Just in Time compilation. However, JAX is based on a pure functional programming paradigm - this means immutable variables and no side effects. This can present challenges to programmers used to the traditional imperative and object oriented style of programming. 

JAX is commonly used for projects related to:

 * Research in machine learning and deep learning
 * Complex deep learning applications

[Flax](https://github.com/google/flax) is an upcoming neural network library built on top of JAX. It is mainly used by JAX users for Neural network research and model development. Researchers use Flax's Neural Network API to:

 * Construct neural network layers
 * Train neural networks
 * Check-point models
 * Access pre-trained and fine-tuned models 

### Scikit-learn

[scikit-learn](https://github.com/scikit-learn/scikit-learn) is a Python-based library with tools for machine learning and data analysis. It supports a variety of ML techniques such as classification and clustering, random forests, Support Vector Machines (SVMs), and the like. The packages of scikit-learn are built on top of Numpy, SciPy, and matplotlib. Scikit-learn is commonly used for applications such as:

 * Machine learning
 * Statistical modeling
 * Image recognition 

For practical examples of building applications using Scikit-learn, visit these pages:

 * [Build a Machine Learning Classifier in Python](https://www.vultr.com/docs/how-to-build-a-machine-learning-classifier-in-python-with-scikit-learn/)

### Pandas

The [Pandas](https://pandas.pydata.org/) package is a collection of user-friendly data structures to serve as the building blocks of data-based projects. Pandas also includes tools to perform different operations on the data. It is typically used in data-based projects for operations like: 

 * Using dictionary-based data structures for real-time applications
 * Pre-processing data for ML applications
 * Viewing the data in different ways, such as transposes and pivot tables
 * Importing and exporting different file formats, such as CSV

Visit these pages to see example applications that use the Pandas library:

 * [Deploy a Deep Learning Model with Streamlit](https://www.vultr.com/docs/how-to-deploy-a-deep-learning-model-with-streamlit/)

## Hugging Face 

### Hugging Face Hub

[Hugging Face Hub](https://pypi.org/project/huggingface-hub/) is an online platform that hosts a large collection of deep learning models, datasets, demo apps, and documentation. The `huggingface_hub` package allows developers to log in and connect to the Hugging Face Hub from Python programs. This package is commonly used to:

 * Upload models, checkpoints, and datasets to Hugging Face

### Transformers

The [Hugging Face Transformers](https://www.vultr.com/docs/how-to-use-hugging-face-transformer-models-on-vultr-cloud-gpu/) package includes most of the tools a developer needs to work with transformer models. It makes available more than 25, 000 pre-trained models. Hugging Face Transformers is compatible with PyTorch, TensorFlow, and JAX. This package is used to access:

 * Large language Models (LLMs) like Falcon and Llama
 * Models for computer vision applications like object detection, segmentation, and classification
 * Multimodal models like character recognition, summarizing scanned documents, and more
 * Tools like tokenizers, pipelines, trainers, training configuration tools, data preprocessing tools, and more

The pages below feature applications that use Hugging Face Transformer models:
 
 * [Implement the Falcon LLM](https://www.vultr.com/docs/how-to-use-tii-falcon-large-language-model-on-vultr-cloud-gpu/)
 * [Implement the Llama 2 LLM](https://www.vultr.com/docs/how-to-use-meta-llama-2-large-language-model-on-vultr-cloud-gpu/)
 * [Build an Inference API Using Hugging Face Transformers and FastAPI](https://www.vultr.com/docs/how-to-build-an-inference-api-using-hugging-face-transformers-and-fastapi/)

### Diffusers

[Diffusers](https://www.vultr.com/docs/how-to-use-hugging-face-diffusion-models-on-vultr-cloud-gpu/) is a Hugging Face library that includes many pre-trained diffusion models. It is used for tasks like:
 
 * Accessing pre-trained image generation models, like Stable Diffusion 
 * Accessing pre-trained audio-diffusion models to generate audio
 * Fine-tuning generative AI models 
 * Using generative AI models to serve via an API

Visit the pages below to learn about implementing applications that use Diffuser models:

 * [Text Guided Image to Image Generation with Stable Diffusion](https://www.vultr.com/docs/text-guided-image-to-image-generation-with-stable-diffusion/)
 * [Deploy Dreambooth and Stable Diffusion](https://www.vultr.com/docs/deploy-dreambooth-and-stable-diffusion-on-a-vultr-cloud-gpu/)
 * [Build an Inference API Using Hugging Face Diffusers and FastAPI](https://www.vultr.com/docs/how-to-build-an-inference-api-using-hugging-face-diffusers-and-fastapi/)

## Datasets

### Hugging Face Datasets

[Hugging Face Datasets](https://huggingface.co/docs/datasets/index) standardizes the presentation and organization of datasets used to train and test ML models. It simplifies importing datasets and pre-processing the data. Because it is integrated with Hugging Face Hub, it also streamlines the process of exporting and sharing datasets. Hugging Face Datasets is used for applications such as:

 * Importing datasets to fine-tune models
 * Importing datasets to test model performance
 * Exporting datasets to share with others

Hugging Face Datasets is available via the `datasets` package. These articles feature the use of the Datasets package in practical applications:

 * [Fine-tune a Hugging Face Transformer Model](https://www.vultr.com/docs/fine-tune-a-hugging-face-transformer-model-on-vultr-cloud-gpu/)
 * [Fine-tune a Hugging Face Diffuser Model](https://www.vultr.com/docs/fine-tune-a-hugging-face-diffuser-model-on-vultr-cloud-gpu/)

### TensorFlow Datasets 

The package [TensorFlow Datasets](https://github.com/tensorflow/datasets) is a standardized approach for using datasets to work with TensorFlow-based models. It is used to:

 * Access several publicly available datasets
 * Pre-process the data to prepare it for training
 * Share your datasets with the community via the hub

## Plotting and Computer Vision Tools

### Matplotlib

[Matplotlib](https://matplotlib.org/) is a plotting package for Python. Developers use it to:
 
 * Create graphs, charts, and animations 
 * Create interactive visualizations
 * Export the created illustrations in different formats

### OpenCV

[OpenCV](https://www.vultr.com/docs/how-to-install-opencv-on-centos-7/) stands for Open Source Computer Vision Library. It implements hundreds of algorithms relevant to computer vision tasks. Many of these functions are optimized for real-time applications. The underlying code of OpenCV is written in C++. It offers a Python API interface, in addition to the standard C++ interface. OpenCV supports GPU-based operations but it doesn't automatically handle multi-GPU systems. 

OpenCV is used in many applications that use image processing, such as:

 * Face detection and recognition
 * Object tracking
 * Gesture recognition
 * Augmented Reality (AR)

The package `cv2` exposes the OpenCV Python API. For examples of applications based on OpenCV, refer to the pages below:

 * [Human Face Detection with OpenCV](https://www.vultr.com/docs/human-face-detection-with-opencv/)
 * [Perform Facial Recognition](https://www.vultr.com/docs/how-to-perform-facial-recognition-in-python-with-a-vultr-cloud-gpu/#Building_Face_Recognition)

### Pillow 

While working on computer vision models and diffusion models, it is often necessary to visually inspect images. The [Pillow](https://pypi.org/project/Pillow/) library enhances the Python command line with basic image processing capabilities. It supports many different image file formats. Pillow is forked from the popular Python Imaging Library (PIL) which is no longer actively maintained. Pilow is used for tasks like: 

 * Opening images in different formats
 * Preprocessing images for deep learning models
 * Switching between image formats and NumPy arrays
 * Fetching images from remote URLs

These articles on image processing and image generation applications feature the use of the Pillow package:

 * [Build a Super Resolution Image Server](https://www.vultr.com/docs/build-a-super-resolution-image-server-with-vultrs-cloud-gpus/#Build_a_Super_Resolution_Image_Web_Application)
 * [Text Guided Image to Image Generation](https://www.vultr.com/docs/text-guided-image-to-image-generation-with-stable-diffusion/)
 * [AI Image Manipulation Using InstructPix2Pix](https://www.vultr.com/docs/ai-image-manipulation-with-instruct-pix2pix-on-vultr-cloud-gpu/#Process_the_Image)

### PyTorch Image Models (TIMM)

[`timm`](https://github.com/huggingface/pytorch-image-models) includes a large collection of packages for machine learning specifically for Computer Vision (CV). TIMM is used to access:

 * Neural network layers Preconfigured for CV
 * Optimizers and schedulers for training CV models
 * Image-data preprocessing functions
 * AI models for image processing tasks like feature extraction and object detection

### Albumentations

Computer vision models need many images in the training set. It is not always realistic to have large training data sets. A common technique to solve this problem is to create new training images by slightly modifying the existing images. This process is called image augmentation. The [Albumentations](https://albumentations.ai/) library, which is based on OpenCV, is commonly used for this. It provides a user-friendly interface for tasks like:

 * Image augmentation using different techniques to modify the original images
 * Image segmentation and classification
 * Object detection

## Usability Tools

### Accelerate

Running programs on setups involving multiple CPUs, GPUs or TPUs requires writing boilerplate code for distributed computing. [Accelerate](https://huggingface.co/docs/accelerate/index) abstracts away this boilerplate code. Thus, the same PyTorch training code can be run on any system, whether it has a single CPU, single GPU, or multiple CPUs, GPUs, and TPUs. Its primary use-case is to: 

 * Make PyTorch code processor-agnostic

### xFormers

[xFormers](https://github.com/facebookresearch/xformers) is an open-source domain-agnostic library. It provides composable building blocks to construct transformer models. It is used to:

 * Speed up training transformer and diffuser models
 * Speed up inference processes, such as text and image generation
 * Reduce memory consumption in transformer-based models
 * Enhance the performance of diffuser models by optimizing the attention block

### Einops

Tensors are the building blocks of deep learning models. Numerical operations involving tensors often involve complex mathematical rules. Tensor variables are written with superscripts, subscripts, and other notational artefacts. These notations are not always consistent across different publishers, packages, and programmers. Einstein Inspired Notation for Operations ([einops](https://einops.rocks/)) is a notational standard for tensor operations. It helps to:

 * Use consistent notational conventions for tensor representation and operations across NumPy, PyTorch, TensorFlow, JAX, and other major packages 
 * Ensure that code involving tensor operations is readable and error-free

### Evaluate 

While training ML models, it is crucial to know how well the model is performing. This is called evaluating the model. Different models are evaluated according to different parameters. The [`evaluate`](https://huggingface.co/docs/evaluate/index) library
standardizes the evaluation process across models. The key use-case for Evaluate is:

 * Developers can choose different evaluation methods by changing just one line of code. 

## Development Environments

### Jupyter Notebook

Computational documents consist of code, descriptions, data, illustrations, and control elements. [Jupyter Notebook](https://www.vultr.com/docs/how-to-install-jupyter-notebook-on-a-vultr-ubuntu-16-04-server-instance/) is an interactive web application used to work on computational documents. With Jupyter Notebook, developers need only a web browser to write and test programs, and to enhance them further into computational documents. Jupyter Notebook supports Julia, Python, and R. It allows developers to: 

 * Create, edit, and save computational documents
 * Execute individual commands and whole programs 
 * Share the whole notebook or parts of it, via a URL or social media

 > The classic Jupyter Notebook is deprecated after version 6. It is succeeded by Notebook version 7 which is built based on the JupyterLab codebase and components. 

### JupyterLab

[JupyterLab](https://www.vultr.com/docs/how-to-set-up-a-jupyterlab-environment-on-ubuntu-22-04/) is a complete Interactive Development Environment (IDE), similar to VS Code and Eclipse. It integrates many different file formats, such as CSV, JSON, Markdown, and so on. Its core functionality is support for computational documents, similar to Jupyter Notebook. In addition, it offers IDE-like features, such as:

 * Visual debugging and code inspection
 * Opening multiple tabs in the same window 
 * Split views
 * Theming and customization options

## Conclusion

This article presents a quick introduction to commonly used software packages in machine learning and artificial intelligence. To learn how to use the packages in practical applications, follow the links in the descriptions of each package.

