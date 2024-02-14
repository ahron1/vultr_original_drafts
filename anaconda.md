# Install Anaconda on Ubuntu 22.04

## Introduction

Conda is an open-source tool that does two things - package management and environment management. As package manager, its functionality is similar to any other package management tool, like APT for Ubuntu or Pip for Python. However, compared to Pip, Conda deals with dependencies more carefully. As an environment manager, it helps to create and manage different environments. Different applications sometimes have dependencies on different versions of the same software. Managing different versions of the same software on one computer is difficult. Environments solve this problem. Each environment can have its own set of software and dependencies, independent of other environments. Conda is available to install as part of either Anaconda and Miniconda. 

Anaconda includes Conda, Python, and many other data-science-based tools in a single package. This makes it faster to get started on data-science-related projects without having to spend time installing the required software. Anaconda is the preferred option when many software tools are needed, such as for large projects. It is also suitable when it is not known in advance what tools the project will need. This is often the case with exploratory projects.

Miniconda is similar to Anaconda, but it is pre-packaged with a smaller set of Python packages. This makes it the preferred choice for servers where space might be at a premium. It is also the preferred option in cases where only a small set of tools is needed. Where Anaconda pre-installs a few hundred common packages like PyTorch, Transformers, Numpy, Scipy, and so on, Miniconda only installs a few dozen basic utility packages.

Choose Anaconda if:

 * The project is large or needs many different packages
 * The project is exploratory and while starting, you do not yet know what tools it might need
 * You do not want to individually install many software packages
 * The computer has enough space 

If your use case does not meet the above requirements, it may be better to [install Miniconda](https://google.com) instead.

This guide shows how to install Anaconda on Ubuntu 22.04. 

### Prerequisites

This guide presents Conda as a Python-focussed tool. Therefore, you are expected to have a working knowledge of Python programming.

When you create a new server, you get only a root account. It is strongly advised to run application programs as a regular user. It is also advised to log into remote servers as a non-root user using SSH keys. To do this, you are expected to be able to:

 * [generate SSH keys](https://www.vultr.com/docs/how-do-i-generate-ssh-keys/)
 * [deploy a new server using the keys](https://www.vultr.com/docs/deploy-a-new-server-with-an-ssh-key/)
 * [connect to the server over SSH](https://www.vultr.com/docs/connect-to-a-server-using-an-ssh-key/)
 * [create a new non-root user with sudo rights](https://www.vultr.com/docs/how-to-use-sudo-on-a-vultr-cloud-server/#Create_a_Sudo_User_on_Debian___Ubuntu)
 * [set up SSH access for a non-root user](https://www.vultr.com/docs/using-your-ssh-key-to-login-to-non-root-users/)

The steps in the following sections assume you are logged in as the user `pythonuser` and that the home directory of this user is `/home/pythonuser`.

## Install Anaconda

To install Anaconda, you need the installer file. To get the download link for this file, visit the [Downloads section](https://www.anaconda.com/download#downloads) on the Anaconda downloads page. It shows the download links for different operating systems. To install Anaconda on Ubuntu 22.04 on Vultr servers, you need the installer for 64-bit Linux. Right-click the text that starts with *64-Bit (x86) Installer* and copy the link.

In August 2023, the link looks like this:

    https://repo.anaconda.com/archive/Anaconda3-2023.07-1-Linux-x86_64.sh

The rest of this guide is based on the above link. 

Download the installer:

    $ wget https://repo.anaconda.com/archive/Anaconda3-2023.07-1-Linux-x86_64.sh

This downloads the installer, `Anaconda3-2023.07-1-Linux-x86_64.sh` onto your computer. The installer is a large shell script, about 1 Gigabyte in size.

You can install Anaconda in two ways. The first is the normal process in which the user interactively uses the installer application. The user manually accepts the license agreement and selects the installation directory. The second option is an automated approach, suitable for script-based installations. The automated approach involves a few additional commands.

### Option 1 - Normal Installation

To start the installation, run the shell script:

    $ bash Anaconda3-2023.07-1-Linux-x86_64.sh

It shows a message on the screen:

    In order to continue the installation process, please review the license agreement.
    Please, press ENTER to continue

After you press `Enter`, it shows the License Agreement. To browse through the agreement, use the `Up` and `Down` arrows. To  close the agreement and proceed with the installation process, press `Q`. This displays the following message:

    Do you accept the license terms? [yes|no]

Type `yes` and press `Enter`. This displays the following message on the screen:

    Anaconda3 will now be installed in this location:
    /home/pythonuser/anaconda3

    - Press ENTER to confirm the location   
    - Press CTRL-C to abort the installation   
    - Or specify a different location below

By default, the installer installs Anaconda in the home directory of the user, under the path `~/anaconda3`. If you are logged in as `pythonuser`, it installs in the directory `/home/pythonuser/anaconda3`. If you are logged in as `root`(not recommended), the default installation directory is `/root/anaconda3`.

To install Anaconda in the default installation directory, `/home/pythonuser/anaconda3`, press `Enter`.

To install it in a different directory, manually type the directory location and then press `Enter`. Ensure that you have write access to the installation directory.

The installer proceeds with downloading and extracting the files into the right locations. After finishing the installation process, it shows this message on the screen:

    installation finished.
    Do you wish the installer to initialize Anaconda3
    by running conda init? [yes|no]

Initialize Conda before using it. Type `yes` and press `Enter`. 

Anaconda is now installed on the system. Log out of the session and log back in as `pythonuser`. If you followed this section, skip the next section on silent installation.

### Option 2 - Silent Installation

The steps in the previous section show the standard way to install Anaconda. It is sometimes necessary to install it automatically through a script. In such situations, it is not practical to interact with the installer and press `Enter`, type `yes`, and so on. Run the installer in batch mode to automatically accept the license agreement and install Anaconda:

    $ bash Anaconda3-2023.07-1-Linux-x86_64.sh -b

This installs Anaconda into the default directory `~/anaconda3`. Use the `-p` option to install Anaconda in a specific directory, for example, `/var/anaconda`:

    $ bash Anaconda3-2023.07-1-Linux-x86_64.sh -b -p /var/anaconda

Ensure that the user has permission to write into the installation directory. This guide assumes you install Anaconda into the default location as the user `pythonuser`. After the installation finishes, it shows a message on the screen:

    Preparing transaction: done 
    Executing transaction: done 
    installation finished.

It is necessary to initialize Conda before using it. Run the activation script:

    $ source /home/pythonuser/anaconda3/bin/activate

Initialize Conda:

    $ conda init

Anaconda is now installed on the system. Log out of the session and log back in as `pythonuser`. 

## Verify the Installation

After logging back into the system as `pythonuser`, notice that the prompt looks like this:

    (base) pythonuser@servername:~$ 

The name of the Conda environment is prefixed to the prompt. 

Check the `PATH`:

    $ echo $PATH

Verify that the `PATH` contains the path to the ancillary programs installed by Anaconda and the path to the Conda executable:

    /home/pythonuser/anaconda3/bin:/home/pythonuser/anaconda3/condabin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

Check the list of programs installed by Anaconda:

    $ conda list

To exit (deactivate) the Conda environment and go back to the Linux shell:

    $ conda deactivate 

## Uninstall

Before uninstalling Anaconda, deactivate the current environment (shown above). 

Delete the Anaconda installation directory:

    $ rm -rf ~/anaconda3

Delete the Conda hidden folder:

    $ rm -rf ~/.conda

Finally, delete the Conda section from the `~/.bashrc` file. Open the `~/.bashrc` file using a text editor. Go to the end of the file and find the section that looks like this:

    # >>> conda initialize >>>
    # !! Contents within this block are managed by 'conda init' !!
    __conda_setup="$('/home/pythonuser/anaconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
    if [ $? -eq 0 ]; then
        eval "$__conda_setup"
    else
        if [ -f "/home/pythonuser/anaconda3/etc/profile.d/conda.sh" ]; then
            . "/home/pythonuser/anaconda3/etc/profile.d/conda.sh"
        else
            export PATH="/home/pythonuser/anaconda3/bin:$PATH"
        fi
    fi
    unset __conda_setup
    # <<< conda initialize <<<

Delete the entire block of code between `# >>> conda initialize >>>` and `# <<< conda initialize <<<`

Log out of the terminal session and log back in. Anaconda is uninstalled from the system.

## Conclusion

This guide discussed how to install Anaconda, starting from scratch on a Ubuntu 22.04 system. It showed two different installation methods - regular and silent. It also showed how to verify the installation, Finally, it described how to completely remove Anaconda from the system. The [official Anaconda documentation](https://docs.anaconda.com/free/anaconda/install/linux/) contains further information on the installation process.

