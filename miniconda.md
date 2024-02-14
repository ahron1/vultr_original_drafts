# Install Miniconda on Ubuntu 22.04

## Introduction

Conda is an environment manager and a package manager bundled into a single package. Environments are a way to have different versions of the same software on the same computer. They are necessary when different applications depend on different versions of a common dependency. An environment manager, such as Conda, creates different paths for each environment. An environment installs packages into and runs packages from only its path, which is different from the paths of other environments. Thus, environments are independent of each other. 

Conda is distributed with the Anaconda and Miniconda packages. In principle, Conda is not Python-specific. However, it is most commonly used for Python-based applications. Anaconda includes Conda, a Python runtime, and many packages useful for data-science applications, such as PyTorch, Transformers, Numpy, and a few hundred others. Anaconda saves a lot of time that would otherwise have been spent on installing packages. Thus, projects which need a large variety of software tools prefer Anaconda. Anaconda is uniquely suited to large projects and exploratory projects.

Unlike Anaconda, Miniconda includes only the most important utility packages on top of Conda and Python. Thus, it is preferable on projects that need fewer tools. It also suits remote servers with limited space. 

Choose Miniconda if:

 * The project is small or needs only a few packages
 * You already know the tools that the project needs
 * You prefer to manually install the needed software
 * There is limited space available on the computer 

If the above points don't describe your use case, consider [installing Anaconda](https://google.com) instead.

The steps in this guide illustrate how to install Miniconda on Ubuntu 22.04. 

### Prerequisites

Conda, in the context of this guide, is a Python-focussed tool. Thus, it is necessary to have a working knowledge of Python programming.

On a new Vultr VPS, by default, you log in as the root user. It is good practice to not run applications as the root. Use `root` only for system administration tasks. Furthermore, while working on remote servers, it is recommended log in using SSH keys as a non-root user. Thus, before continuing with this guide, it is advised to:

 * know how to [generate SSH keys](https://www.vultr.com/docs/how-do-i-generate-ssh-keys/)
 * know how to [deploy a new Vultr VPS or dedicated server to use the keys](https://www.vultr.com/docs/deploy-a-new-server-with-an-ssh-key/)
 * know how to [use SSH to connect to the server](https://www.vultr.com/docs/connect-to-a-server-using-an-ssh-key/)
 * [create a new user and grant sudo rights to the user](https://www.vultr.com/docs/how-to-use-sudo-on-a-vultr-cloud-server/#Create_a_Sudo_User_on_Debian___Ubuntu)
 * be comfortable [creating a new non-root user to log in to the server over SSH](https://www.vultr.com/docs/using-your-ssh-key-to-login-to-non-root-users/)

The example commands in this guide assume that you are logged in with the username `pythonuser` with the home directory `/home/pythonuser`.

## Install Miniconda

Before installing Miniconda, you need the link to the installer file. On the Miniconda downloads page, visit the section [Latest Miniconda Installer Links](https://docs.conda.io/en/latest/miniconda.html#latest-miniconda-installer-links). The table shows links to download the installer for different operating systems. You need the installer for 64-bit Linux. Right-click the text *Miniconda3 Linux 64-bit* and copy the link.

As of August 2023, the link resembles:

    https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

This link is used as an example for the rest of this guide.

Download the installer:

    $ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

This downloads the installer, `Miniconda3-latest-Linux-x86_64.sh`. The installer file is a shell script, nearly 100 Megabytes in size.

There are two options to install Miniconda. The first (default) option requires the user to manually accept the license agreement. It also allows to specify the installation directory. In the second option, the installer automatically accepts the license agreement. The installation directory can be passed as a command line argument.

### Option 1 - Normal Installation

Run the shell script and start the installation:

    $ bash Miniconda3-latest-Linux-x86_64.sh

The installer displays an on-screen message:

    In order to continue the installation process, please review the license agreement.
    Please, press ENTER to continue

Press `Enter`. It displays the License Agreement on the screen. Use the `Up` and `Down` cursor keys to browse through the agreement. Press `Q` to close the agreement and continue with the installation process. After pressing `Q`, it shows a message on the screen:

    Do you accept the license terms? [yes|no]

Type `yes` and press `Enter`. It shows another message on the screen asking to confirm the installation directory:

    Miniconda3 will now be installed in this location: 
    /home/pythonuser/miniconda3    

    - Press ENTER to confirm the location   
    - Press CTRL-C to abort the installation   
    - Or specify a different location below

If you don't specify a location, it installs Miniconda to the user's home directory. The installation path is `~/miniconda3`. For the user `pythonuser`, the default installation directory is `/home/pythonuser/miniconda3`. For the root user, the default installation directory is `/root/miniconda3`.

Press `Enter` to install Miniconda to `/home/pythonuser/miniconda3`.

If you need to install Miniconda to a different directory, type the location of that directory and press `Enter`. The user (`pythonuser`) must have write access to the installation directory.

The installer downloads the compressed files into the installation directory and extracts them. After finishing, the on-screen message asks whether you want to initialize Miniconda:

    installation finished.
    Do you wish the installer to initialize Miniconda3
    by running conda init? [yes|no]

Before using Miniconda, you need to initialize it. Type `yes` and hit `Enter`. 

This completes the installation process. Before using Miniconda, you need to start a fresh terminal session. Log out of the terminal and log back in. Miniconda is now installed for the user `pythonuser`. 

The next section discusses silent installation. Skip it if you already installed Miniconda based on the instructions in this section.

### Option 2 - Silent Installation

The previous section shows the standard interactive way to install Miniconda. The interactive method is not suitable for automatic installations using a script. The batch mode solves this problem. The `-b` option runs the installer in batch mode. This automatically accepts the license agreement to install Miniconda:

    $ bash Miniconda3-latest-Linux-x86_64.sh -b

The default installation directory is `~/miniconda3`. For the user `pythonuser`, this is `/home/user/miniconda3`. 

Instead of the default directory, you can install it into a specific location, such as  `/var/miniconda`. To do this, use the `-p` option:

    $ bash Miniconda3-latest-Linux-x86_64.sh -b -p /var/miniconda

The user must have write access for the installation directory. The following steps assume that Miniconda is installed into `/home/pythonuser/miniconda3`.

After completing the installation, the installer displays an on-screen message: 

    Preparing transaction: done 
    Executing transaction: done 
    installation finished.

Before using Conda, you must initialize it. Use the activation script to do this: 

    $ source /home/pythonuser/miniconda3/bin/activate

Use the `init` command to initialize Conda before using it:

    $ conda init

This completes the installation. To use it, start a new terminal session. You can also log out of the current session and log back in. 

## Verify the Installation

If Miniconda is successfully installed, it modifies the system prompt. For the user `pythonuser`, the prompt now looks like this:

    (base) pythonuser@servername:~$ 

The default Conda environment is `base`. The prompt message includes the name of the Conda environment as shown above.

Verify that the system `PATH` includes the Miniconda directory:

    $ echo $PATH

The output should include the paths to the Conda executable and the ancillary programs installed by Miniconda:

    /home/pythonuser/miniconda3/bin:/home/pythonuser/miniconda3/condabin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

Use the command `conda list` to see the list of programs installed by Miniconda.

To go back to the Linux shell, exit (deactivate) the Conda environment:

    $ conda deactivate 

## Uninstall

The uninstallation process involves deleting a few directories and updating a config file. Before doing this, it is advisable to deactivate the current environment as shown above.

The Miniconda installation directory is `/home/pythonuser/miniconda3`. Delete this directory:

    $ rm -rf /home/pythonuser/miniconda3

Conda also installs a hidden folder, `~/.conda`. Delete the hidden folder: 

    $ rm -rf /home/pythonuser/.conda

To add the Conda paths to the system path, the installer modifies the `.bashrc` file. This is the configuration file for the Bash shell. To completely uninstall Conda, remove the relevant section from the `~/.bashrc` file. Use a text editor and open the file. The end of the file consists of a section resembling:

    # >>> conda initialize >>>
    # !! Contents within this block are managed by 'conda init' !!
    __conda_setup="$('/home/pythonuser/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
    if [ $? -eq 0 ]; then
        eval "$__conda_setup"
    else
        if [ -f "/home/pythonuser/miniconda3/etc/profile.d/conda.sh" ]; then
            . "/home/pythonuser/miniconda3/etc/profile.d/conda.sh"
        else
            export PATH="/home/pythonuser/miniconda3/bin:$PATH"
        fi
    fi
    unset __conda_setup
    # <<< conda initialize <<<

Delete this section (the entire block of code between `# >>> conda initialize >>>` and `# <<< conda initialize <<<`).

Miniconda is now uninstalled from the system. Log out of the terminal session and log in again. 

## Conclusion

This guide explains the steps to install Miniconda on a fresh Ubuntu 22.04 server. The explanations include both the regular installation method and batch mode installation. The last section explains how to completely uninstall Miniconda from the computer. To know more about installing Miniconda, refer the [official documentation](https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html).

