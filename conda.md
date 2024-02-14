# Fundamental Operations Within a Conda Environment

## Introduction

Conda is an open-source software utility. It bundles together two key functionalities:

 * Environment management: Environments install software into their respective paths. Thus, you can install different versions of the same software into different environments without causing conflicts. Packages installed into an environment do not affect the host system either.
 * Package management: It is similar to pip and other package managers. However, Conda is better at handling complex dependency chains.

This guide introduces commonly used Conda commands. The goal is to get a new Conda user up to speed and to serve as a quick reference for experienced users. The commands in this guide work on recent versions of Conda on recent Linux/UNIX-based systems. Corresponding commands for Windows and MacOS operating systems can sometimes be different. 

### Prerequisites

To follow this guide, you need to:

 * [Generate SSH keys](https://www.vultr.com/docs/how-do-i-generate-ssh-keys/) and [add the keys to your account](https://www.vultr.com/docs/deploy-a-new-server-with-an-ssh-key/#Add_an_SSH_key_to_your_Vultr_Account), if you haven't already done so
 * Deploy a Vultr instance running a recent version of a Linux/UNIX-based operating system
 * [Connect to the server over SSH](https://www.vultr.com/docs/connect-to-a-server-using-an-ssh-key/). 

 * [Create a new user with sudo rights](https://www.vultr.com/docs/how-to-use-sudo-on-a-vultr-cloud-server/#Create_a_Sudo_User_on_Debian___Ubuntu). 
 * [Log into the server over SSH as the non-root user](https://www.vultr.com/docs/using-your-ssh-key-to-login-to-non-root-users/)

Before using Conda, you need to install it. You can do this in two ways:

 * [install Anaconda](google.com) or
 * [install Miniconda](google.com) 

## Basic Conda Usage

Check the installed Conda Version

    $ conda --version 

Get information about the Conda installation on the system:

    $ conda info

The above command outputs information that looks like the example below:

    active environment : env1 
    active env location : /home/pythonuser/miniconda3/envs/env1
    shell level : 1
    user config file : /home/pythonuser/.condarc
    conda version : 23.3.1
    python version : 3.10.10.final.0
    .
    .

Update Conda to the latest version:

    $ conda update conda -y

To access the man page for a specific command:

    $ # pseudo-code # conda COMMAND --help

For example, to learn more about the `conda env` command:

    $ conda env --help

Similarly, to know more about the `conda env create` sub-command:

    $ conda env create --help

## Manage Conda Environments

Create a new environment, `env1`:

    $ conda create --name env1 -y

Activate the Conda environment `env1`:

    $ conda activate env1

The name of the environment is prefixed to the prompt. For example, when the environment `env1` is active, the terminal prompt looks like this:

    (env1) pythonuser@test:~$ 

Deactivate the active Conda environment:

    $ conda deactivate

Nested activation refers to activating an environment from within another environment. Repeat the above command to exit each nested environment. In general, it is not advisable to use nested environments.

After deactivating the environment, the prompt looks like the default terminal prompt:

    pythonuser@test:~$ 

Create a new environment, `env2`, with a specific version of Python:

    $ conda create --name env2 python=3.9 -y

Check the list of Conda environments:

    $ conda env list

Delete an environment, `env1`:

    $ conda remove --name env1 --all

To rename an environment, `env2` to `env3`:

    $ conda rename --name env2 env3

To create a new environment `env4`, as a clone of an old environment, `env3`:

    $ conda create --clone env3 --name env4

To create a new environment, `env5`, with `pytorch` for CPU-only systems pre-installed from the channel `pytorch`:

    $ conda create --name env5 pytorch cpuonly -c pytorch -y

The above commands creates a new environment with two pre-installed packages, `pytorch` and `cpuonly`, and their dependencies.

Similarly, to create a new environment, `env6` with Python `3.9` and CPU-only `pytorch` installed from the channel `pytorch`:

    $ conda create --name env6 python=3.9 pytorch cpuonly -c pytorch -y

## Manage Conda Packages

For the next example, exit the active Conda environment:

    $ conda deactivate

Check the list of all installed packages:

    $ conda list

When the above command is run outside a Conda environment, it shows the list of all packages available in all Conda environments. 

Activate environment `env3`:

    $ conda activate env3 

Retest the `list` command:

    $ conda list

It shows the list of all packages installed in the active Conda environment. 

To get the list of packages in a different environment, pass the name of that environment as an option:

    $ conda list --name env6

Conda packages are available from different channels. Different publishers, such as NVIDIA and PyTorch, have their own channels. When the channel name is not specified, Conda uses the default channel. Search a package in a particular channel:

    $ # pseudo-code # conda search PKG_NAME -c CHANNEL_NAME

For example, to look for the package `pytorch` in the `pytorch` channel:

    $ conda search pytorch -c pytorch

To install a specific package, for example, `pytorch` from a particular channel, for example, `pytorch`:

    $ conda install pytorch -c pytorch -y

You can also use the syntax:

    $ # pseudo-code # conda install channel_name::pkg_name

    $ conda install pytorch::pytorch -y

The above two commands install the package into the active environment. To install it into a different environment, use the `--name` option. For example, to install `pytorch` and `cpuonly`, from the `pytorch` channel, into the `env4` environment:

    $ conda install pytorch cpuonly -c pytorch --name env4 -y

To install into the active environment a certain version, say `2.0.0` of a specific package, for example, `pytorch` from a particular channel, for example, `pytorch`:

    $ conda install pytorch=2.0.0 -c pytorch -y

If a newer version of PyTorch is already installed in the same environment, it will be downgraded to the specified version. Conversely, if an older version of the package is already installed in the same environment, it will be upgraded. 

Similarly, to install version `2.0.0` of PyTorch into `env2`, while a different environment is active:

    $ conda install pytorch=2.0.0 -c pytorch --name env2 -y

Update all installed packages in the active environment:

    $ conda update --all -y

Update all installed packages in environment `env6` (when a different environment is active):

    $ conda update --all --name env6 -y

To update a specific package in the active environment:

    $ conda update pytorch -y

To uninstall a package, `pytorch` from the environment `env4`:

    $ conda remove --name env4 pytorch -y

In the above command, if the name of the environment is not specified, it uninstalls the package from the active environment.

## Import and Export Environments

To create a `requirements` file based on the environment, `env6`:

    $ conda list --explicit --name env6 > requirements_env6.txt

The above commands creates a new text file `requirements_env6.txt` based on the packages in the `env6` environment. If the `--name` option is skipped, the `requirements` file is based on the active environment.

To create a new environment, `env7`, based on a `requirements` file:

    $ conda  create --name env7 --file requirements_env6.txt

The above command creates a new environment, `env7` based on the packages in `env6`.

To export a YAML file with the configuration of `env5`:

    $ conda env export --name env5 > env5.yml 

To create a new environment, `env8`, based on a `yml` configuration file:

    $ conda env create --name env8 --file env5.yml 

The above command creates a new environment, `env8` based on the packages in `env5`.

## Use pip with Conda

You can also use pip to install packages within a Conda environment. These packages are visible only inside the Conda environment where they are installed. Other Conda environments cannot access these packages. 

For example, activate the environment `env3`:

    $ conda activate env3

Install `scipy` using pip while `env3` is active:

    (env3) pythonuser@test:~$ pip install scipy

Scipy is available in `env3` but not in other environments.

 > When exporting environment configurations, YAML files contain information on packages installed with both Conda and pip. `requirements` files contain information on packages installed with only Conda. 

## Conclusion

This guide presents operations commonly used in a Conda environment. The example commands discuss how to manage environments and packages. 

### Further Information

 * The Conda project has [cheat sheets](https://conda.io/projects/conda/en/latest/user-guide/cheatsheet.html) for different Conda versions. These are helpful resources with quick reminders of various functionalities.
 * The [official Conda documentation](https://docs.conda.io/projects/conda/en/stable/) is a good learning resource to learn more about how Conda works and how to use it.
 * [Conda Basics](https://learning.anaconda.cloud/conda-basics) on the Anaconda website is a useful short course in learning how to use Conda

