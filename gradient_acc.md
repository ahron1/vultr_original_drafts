# How to use Gradient Accumulation to Overcome GPU Memory Limitations

## 1. Introduction

To train a machine learning model, the training dataset is split into batches. The model processes this data batch-wise. To process (evaluate) a batch, the computer loads it into memory. Since the model trains on one batch at a time, the computer needs sufficient memory for the model to completely load each batch and to process it. Complex models are typically trained on large volumes of detailed input data (graphics, audio transcripts, etc.). The more detailed the data, the more memory is needed to hold it while processing. The available memory can be insufficient to accommodate the desired batch size. When a model's memory requirement exceeds the available memory, it crashes with an out-of-memory error.

The larger the batch size, the more memory is needed to load its data, and to evaluate it. One way of working around memory constraints is to reduce the batch size. However, reducing the batch size is not always desirable. Many models learn better and faster on larger (up to a limit) batch sizes.

Another approach is to use smaller batches, but to not update the model parameters based on the feedback from every batch. Instead, the updates are accumulated for several batches, and then applied. To a certain extent, this mimics the effect of having used a larger batch size. This technique is called *gradient accumulation*.

### Required Background

To follow along this guide, some hands-on experience with training neural network based machine learning models is necessary. It is assumed that you can install and use PyTorch, Torchvision and other software. This guide has been tested on Vultr GPU VMs featuring the NVIDIA A100 GPU. The GPU specific commands may be different on other GPU devices.

While this guide does include the setup of a rudimentary machine learning model, it is only to have an example to apply gradient accumulation on, and not to explain the implementation of machine learning models per se. The configuration of the model is only for demonstration and may not be optimal for delivering the best training outcomes.

## 2. Motivation

Neural networks are used to recognize patterns. A neural network accepts input data in the form of a tensor and sequentially applies on it multiple layers of transformations. The model's output is a prediction about which of several patterns the input matches. For instance, a neural network might be trained to take as input the photo of an animal and predict which species it belongs to.

Each individual transformation is a mathematical operation, typically a linear equation. A layer consists of a large number of individual transformations. Each transformation is parametrized by the model weights. These weights (parameters) determine the coefficients of the linear equations.

Training a neural network starts with a random set of model weights. The weights are iteratively updated to get the model's output to match the expected (correct) output as closely as possible.

At a high level, the main steps in the training loop of a machine learning model are:

1. Evaluate the model output based on the (tentative) model weights.

1. Compute the loss (difference between expected and actual outputs).

1. Feed the loss back into the model. The value of the loss determines the amount by which the model weights are updated.

1. Update the model weights

These steps are repeated for each batch until the model passes through the entire training dataset.

Each pass of the training dataset is called one epoch. A typical training exercise involves several epochs.

The gradient accumulation technique modifies Step 4 as:

  4'. Update the model weights once **every N batches**.

This leads to saving (accumulating) the updates (gradient values) for each batch and moving on to the next batch. After N batches, the gradients thus accumulated are applied. The outcome is *almost* the same as if the data of the N batches had been processed as one batch to estimate the gradients. Choose the value of N depending on how many batches your hardware is able to handle.

## 3. Implementation - Hand-coding the Algorithm

This section shows how to manually implement gradient accumulation. The example in this section uses PyTorch. When using Keras, the procedure remains the same, only the package names differ.

The goal in this example is to demonstrate that a small batch size using gradient accumulation delivers similar results as a larger batch size. Tuning an accurate model is outside the scope of this guide.

### 3.1. Basic Machine Learning Example in PyTorch

Before implementing gradient accumulation, set up a rudimentary machine learning example using [PyTorch](https://pytorch.org/tutorials/beginner/basics/optimization_tutorial.html). It is assumed you are already familiar with similar examples. The next section applies gradient accumulation on this example.

**Note:** To follow along the code samples, it is advisable to make a new Python file and copy the code snippets into it.

#### Import Required Modules

    import torch
    from torch import nn
    from torch.utils.data import DataLoader
    from torchvision import datasets
    from torchvision.transforms import ToTensor

#### Declare Configuration Variables

    batch_size = 64

`batch_size` denotes the size of each batch. This example uses a batch size of 64. The example in the next section applies gradient accumulation using a smaller batch size of 16.

    learning_rate = 1e-3

`learning_rate` is a hyperparameter - it affects the amount by which the model parameters are updated for a given value of the loss function. A large learning rate makes the learning process unstable. If the learning rate is too small, the model takes too long to converge.

    epochs = 20

`epochs` is the number of times the model iterates over the entire training dataset. Larger batch sizes can need more iterations.

Declare a `device` variable:

    device = "cuda"

To run the model on the GPU, assign to `device` the value "cuda", assuming an NVIDIA GPU. Models and data are instantiated on the CPU and need to be moved to the GPU. To run the model on the CPU, use the value "cpu".

#### Import the Dataset

This example uses the standard FashionMNIST dataset. This dataset consists of pictures of items of clothing and their corresponding labels. It is organized as a training set of 60, 000 images and a test set of 10, 000 images. The images are downscaled to 28 x 28 pixels. It has a total of 10 labels, corresponding to 10 types of clothing items.

    training_data = datasets.FashionMNIST(
        root = "data",
        train = True,
        download = True,
        transform = ToTensor()
        )

    test_data = datasets.FashionMNIST(
        root = "data",
        train = False,
        download = True,
        transform = ToTensor()
        )

The DataLoader utility module of PyTorch creates an iterable structure that splits the dataset into mini-batches.

    loader_training_data = DataLoader(training_data, batch_size = batch_size)
    loader_test_data = DataLoader(test_data, batch_size = batch_size)

#### Define the Neural Network

Define a neural network with one hidden layer. The input layer maps the 28 x 28 pixel images into 512 "features". The hidden layer transforms these 512 features and the output layer maps the transformed data to the 10 labels. ReLU is used for intra-layer nonlinear activations.

    class NeuralNetwork(nn.Module):
        def __init__(self):
            super(NeuralNetwork, self).__init__()
            self.flatten = nn.Flatten()
            self.linear_relu_stack = nn.Sequential(
                nn.Linear(28*28, 512),
                nn.ReLU(),
                nn.Linear(512, 512),
                nn.ReLU(),
                nn.Linear(512, 10),
            )

        def forward(self, x):
            x = self.flatten(x)
            logits = self.linear_relu_stack(x)
            return logits

#### Declare Training Components as Variables

Instantiate a model (based on the neural network defined earlier), a loss function, and an optimizer:

    model = NeuralNetwork()
    # move the model to the GPU
    model = model.to(device)

    loss_function = nn.CrossEntropyLoss()

    optimizer = torch.optim.SGD(
            model.parameters(), lr = learning_rate
            )

#### Define the Training Loop

The simplified training loop below computes and backpropagates the losses, and updates the model weights for each batch.

    def training_loop(dataloader, model, loss_function, optimizer):
        size = len(dataloader.dataset)
        for batch, (X, Y_expected) in enumerate(dataloader):

            # move the data to the GPU
            X = X.to(device)
            Y_expected = Y_expected.to(device)

            Y_computed = model(X)

            # compute the loss
            loss = loss_function(Y_computed, Y_expected)

            # backpropagate the loss
            loss.backward()
            
            # update the model weights
            optimizer.step()

            # reset the optimizer for the next iteration
            optimizer.zero_grad()

#### Define the Testing Loop

The testing loop consists of the following steps:

1. Evaluate the model output using the updated parameters at the end of the epoch.

1. Compute the loss for each item and add up all the losses to get the cumulative loss.

1. Compute the total number of correct results (where the expected output matches the model output).

1. Average the cumulative loss over the number of batches. This gives the average loss per batch for that epoch.

1. Average the number of correct results over the size of the dataset. This gives the accuracy of the model.

1. Format and print the results.

The following code below implements these steps.

    def test_loop(dataloader, model, loss_function):
        size = len(dataloader.dataset)
        num_batches = len(dataloader)
        cumulative_loss, correct_results = 0, 0

        with torch.no_grad():
            for X, Y_expected in dataloader:

                # move the data to the GPU
                X = X.to(device)
                Y_expected = Y_expected.to(device)

                Y_computed = model(X)
                cumulative_loss += loss_function(Y_computed, Y_expected).item()
                correct_results += (Y_computed.argmax(1) == Y_expected).type(torch.float).sum().item()

        # average out the cumulative loss and the number of correct results 
        cumulative_loss /= num_batches
        correct_results /= size
        print(f"Test Results: \n Accuracy: {(100*correct_results):>0.1f}%, Avg loss: {cumulative_loss:>8f} \n")

#### Run the Program

Call the training and testing loops over the desired number of epochs. This iteratively trains the model, tests the model's output, and prints out the results for each epoch.

    for t in range(epochs):
        print(f'Epoch {t+1} \n -------------------')
        training_loop(loader_training_data, model, loss_function, optimizer)
        test_loop(loader_test_data, model, loss_function)
    print('Done')

### 3.2. Gradient Accumulation

To use gradient accumulation:

1. Use a smaller batch size.

1. Declare a variable for the accumulation interval, N.

1. Normalize the loss value by averaging it over N batches.

1. Modify the training loop to call the optimizer every N batches.

Update the `batch_size` variable:

    batch_size = 16

Declare a variable to hold the accumulation interval, N:

    N = 4

Update the training loop:

    def training_loop(dataloader, model, loss_function, optimizer):
        size = len(dataloader.dataset)
        for batch, (X, Y_expected) in enumerate(dataloader):

            X = X.to(device)
            Y_expected = Y_expected.to(device)

            Y_computed = model(X)

            loss = loss_function(Y_computed, Y_expected)

            # normalize the loss value
            loss = loss / N

            loss.backward()
            
            # call the optimizer every N steps
            if ((batch + 1) % N == 0) or (batch + 1 == len(dataloader)):
                optimizer.step()
                optimizer.zero_grad()

Lines of code added/modified to implement gradient accumulation are preceded by comments. The rest of the code is the same as before.

#### Explanation

`loss.backward()` - the backpropagation function, computes the *loss per parameter* (the partial derivative of the loss with respect to each parameter) and accumulates it in the gradient of that parameter. For a parameter `x`:

    # pseudo-code
    x.gradient += d_loss / d_x

Calling `optimizer.step()` updates the model weights based on the gradients (and learning rate):

    # pseudo-code
    x += -learning_rate * x.gradient

`optimizer.zero_grad()` resets the gradients at the end of N iterations - so the gradients from the next N batches can be accumulated.

In the standard case, both the optimizer and the backpropagation functions are called every batch. So, the update after each batch is based on the gradients of only that batch. With gradient accumulation, the losses are still backpropagated (and accumulated) every batch, but the optimizer is called every N batches. So, the model weights are updated every N batches based on the gradients accumulated over the N batches. After updating the weights, the gradients are reset.

Before calling the backpropagation function, it is necessary to normalize the loss by averaging it over N batches. Because backpropagating the loss per batch over N batches makes the gradient much larger that what it should be, and leads to overcorrection. Normalizing the loss corrects for this. The gradients accumulated using the normalized loss are a proxy for what the gradients would have been, had the model weights been updated using a single large batch.

The original example in Section 3.1 uses a batch size of 64. Using gradient accumulation gets a similar outcome by using a batch size of 16 and accumulating the gradients for N = 4 batches. After the initial few epochs, the results with and without gradient accumulation should converge to similar figures (the results will not be identical).

**Note:** To measure the resource (time and memory) consumption of a model on PyTorch, use the [PyTorch Profiler](https://pytorch.org/tutorials/recipes/recipes/profiler_recipe.html). Discussion of usage of the profiler is outside the scope of this guide.

## 4. Implementation - Using Pre-packaged Tools

Gradient accumulation can also be implemented using a pre-packaged wrapper on the optimizer. Some popular machine learning frameworks (e.g. PyTorch Lightning) include support to directly use gradient accumulation. Keras and PyTorch do not, by default, include gradient accumulation support. However, some machine learning libraries include modules to directly implement gradient accumulation in Keras and PyTorch.

The code snippets in this section only illustrate how to use the pre-packaged tools in existing code. These are not complete code samples.

### 4.1. Run:AI Wrapper

Run:AI is a machine learning infrastructure and platform company. Their publicly available library of Python tools includes a gradient accumulation wrapper for Keras as well as for PyTorch.

Install `runai`:

    $ pip install runai

#### 4.1.1. Keras

Keras is a deep learning framework that provides an API interface to underlying TensorFlow modules. To use the `runai` wrapper in a Keras model, import the Keras-specific gradient accumulation module:

    >>> import tensorflow as tf
    >>> import runai.ga.keras

Instantiate an optimizer from Keras:

    >>> optimizer = tf.keras.optimizers.Adam()

The line above creates an optimizer based on the Adam algorithm.

Update the optimizer instance with the Run:AI wrapper:

    >>> optimizer = runai.ga.keras.optimizers.Optimizer(optimizer, steps=N)

This should output something like:

    Wrapping 'Adam' Keras optimizer with GA of N steps

Use the new optimizer instance to run the training loop with gradient accumulation.

#### 4.1.2. PyTorch

Import the PyTorch-specific gradient accumulation module from `runai`:

    >>> import runai.ga.torch

Instantiate an optimizer from the PyTorch library:

    >>> optimizer = torch.optim.SGD(model.parameters(), lr = learning_rate)

This creates a Stochastic Gradient Descent algorithm based optimizer.

Wrap the `optimizer` with the gradient accumulation module:

    >>> optimizer = runai.ga.torch.optim.Optimizer(optimizer, steps=N)

The output should resemble:

    Wrapping 'SGD' PyTorch optimizer with GA of N steps

Use this optimizer in the training loop to take advantage of gradient accumulation.

### 4.2. PyTorch Lightning

Lightning is a framework based on PyTorch. It comes with many pre-packaged routines to eliminate the need to write boilerplate code. Lightning also includes support for gradient accumulation.

Lightning's Trainer module handles the training process. Import Trainer:

    from pytorch_lightning import Trainer

Instantiate a Trainer object using the gradient accumulation interval, `accumulate_grad_batches`:

    trainer = Trainer(accumulate_grad_batches=4)

Note that the default value of `accumulate_grad_batches` is 1. The Trainer instance defined above accumulates gradients for 4 batches before calling the optimizer. It is also possible to specify the accumulation interval per epoch:

    trainer = Trainer(accumulate_grad_batches={0: 8, 4: 2})

The above instance accumulates gradients for 8 batches for the initial epochs starting from epoch 0. Epoch 4 onwards, it accumulates for 2 batches.

## 5. Remarks

Gradient accumulation applies to the training process. It does not apply to running (pre-trained) large models, like Stable Diffusion, on systems with limited memory.

Note that larger batch sizes do not always correspond to better training - it depends on the specifics of the model and the data. It can take some trial and error to determine the right batch size for a given use-case. In practice, getting good results using larger batches requires tweaks to other variables, like the number of epochs and the learning rate.

The technique is not specific to GPU memory. It applies to whatever memory the model is being trained on. If a model is being trained on CPU, gradient accumulation is applied on main memory (RAM). In practice, models (and datasets) large enough to warrant the use of such techniques are too large to be efficiently trained on CPU.
