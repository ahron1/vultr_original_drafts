## Introduction

### The Problem

Suppose you are logged into a remote shell session via ssh and you lose internet connectivity due a network issue. The connection to the server is lost and the remote shell session is terminated. Any programs and files open in the session are also terminated abruptly. Any unsaved changes to open files are lost. You have to re-open everything after logging back in. Some files could even get corrupted.

### The Solution

Screen is the solution to this problem. Screen is a window manager that works over an SSH connection. It does two things:

1. Act as a persistent wrapper for other text-based applications
1. Run multiple terminal windows within itself (terminal multiplexing)

### How it Works

Screen starts a window manager and connects you to it. Within the window manager, you start, run, and terminate different windows. Windows contain different applications. If the connection is lost or closed, the window manager continues to run on the server. All open windows inside the window manager continue to run as well. On logging back in, you reconnect to the running session of the window manager. You will find all previously open files and programs still running in the window manager.

## Prerequisites

To follow this guide, you need:

* Access (either remote or local) to a machine running either a Linux or BSD-based operating system with at least 1 GB of RAM.

* Some experience working in the shell and basic terminal operations.

To do this on a remote server, follow these steps to get started:

* [Deploy a cloud server at Vultr](https://my.vultr.com/deploy/)

* [Update the server](https://www.vultr.com/docs/update-debian-server-best-practices).

* [Create a non-root user with sudo privileges](https://www.vultr.com/docs/create-a-sudo-user-on-debian-best-practices).

* [Log in to your server](https://www.vultr.com/docs/how-to-access-your-vultr-vps) as a non-root user.

* It is strongly recommended [to use SSH login with remote servers](https://www.vultr.com/docs/how-to-use-ssh-with-vultr-servers/) and to [disable password-based login](https://www.vultr.com/docs/how-to-secure-ssh-on-arch-linux/).

### Compatibility

This guide has been thoroughly tested on **Ubuntu 20.04 LTS** running **GNU Screen 4.08**. However, it should be compatible with all recent versions of Screen and all recent Linux and BSD-based operating systems.

## 1. Getting Started

The first step is obviously to install Screen. Note that you need to do this with root privileges. The application package is called *screen* on most operating systems.

On Ubuntu:

    # apt install screen

On FreeBSD:

    # pkg install screen

This installs the latest version of Screen. Quickly check if it is now installed on the system.

    # screen -v

This shows the version number of the Screen installation. The output should be something like

    Screen version 4.08.00 (GNU) 05-Feb-20

## 2. Launch Screen

To launch Screen, issue the command:

    $ screen

This starts a new Screen session and connects you to it.

**Best Practice:** It is advisable to start a new Screen session with a unique name. This makes it easier to access the session later on.

Start a named session with:

    $ screen -S <session_name>

It is possible to rename the active session (if, for example, you did not name it while starting it). Do this from Screen's built-in command line. Pressing `Ctrl+A` `:` in an active Screen session opens the built-in command line. On the built-in command line, issue the command

    sessionname <new_session_name>

**Note:** The built-in command line is for issuing commands to the Screen application itself. To access it, press `Ctrl+A` followed by `:` (the colon symbol). Now you type the command you want to issue to Screen. To get out of the built-in command line press `Esc`.

## 3. Windows

Within a Screen session, you create one or more windows. In these windows, you open files and run programs.

### 3.1 Create New Windows

After starting a new session, create a new window within the session using the command key sequence `Ctrl+A` `C`. Hold down `Ctrl` and press `A`, then release both keys and press `C`. You can create as many new windows as you want like this.

**Tip:** In practice, you might want to have different windows for different uses. For example, you could have one window for monitoring logs, another window for checking the database, and so on. You might also want to give each window a unique and relevant name for easier future reference.

### 3.2 Name Windows

Naming windows makes it easier while navigating between multiple windows. To name the active window, press `Ctrl+A` `:` to access the built-in command line and enter the command

    title <window_name>

There is no immediate feedback on pressing `Enter` after the above command. When you check the list of open windows (described in the next section) you will see the (re)named window.

### 3.3 Navigate Different Windows

Press the command key sequence `Ctrl+A` `"` to see the list of all open windows. The list of windows shows the *number* of each window in the leftmost column. The second column shows the name of the window (if you had given it a name) or the name of the shell. You navigate windows in three ways:

1. Navigate the list of open windows by using the arrow keys and press `Enter` to switch to the selected window. Pressing `Esc` hides the list of open windows and brings back the last active window (if you want to just see the list of open windows without navigating to another window).

1. Press `Ctrl+A` `Number` to navigate to a window using its number.

1. Press `Ctrl+A` `P` for going to the previous window, and `Ctrl+A` `N` for going to the next window. These two command key sequences cycle through all the open windows.

### 3.4 Close Windows

To kill a window you no longer need, activate that window, close any open programs in it, and press `Ctrl+D` at the command line of that window to terminate it. When the last window of a session is killed, the Screen instance also terminates.

It is also possible to kill the active window by pressing the sequence `Ctrl+A` `K`. This is not recommended because it directly kills the window and any programs or files open in it. This can lead to data corruption.

## 4. Sessions

Each instance of Screen is called a session. Sessions contain windows.

### 4.1 Connect to a New Session

When you start Screen with the `screen` command, it starts a new session and connects you to it. You can have many sessions running simultaneously. Make sure to start each session from a different terminal.

**Important Note:** Nesting sessions (starting a new session inside another session), while possible, is neither recommended nor generally necessary, nor straightforward to do. Doing this right needs config file changes, and not doing this right leads to problems. It is also beyond the scope of this introductory guide.

### 4.2 Sessions Overview

 To see the list of ongoing sessions, use the command:

    $ screen -ls

Enter the above command from the terminal directly. It also works inside a Screen session.

The output looks something like this:

    There are screens on:
            1306228.test    (30/06/22 12:24:43 AM IST)      (Attached)
            1216515.pts-0.TP15      (29/06/22 10:57:17 AM IST)      (Detached)
    2 Sockets in /run/screen/S-user.

The first column is the session ID. The first part of the ID is the `pid` of the session. In the example above, the name of the first session is `test`. The second session is not named by the user and takes on a default name. The second column is the time when the session started. The last column is the attached/detached status.

**Tip:** If you find yourself at a terminal, and do not know whether you are in a Screen session, check the `STY` environment variable.

    $ echo $STY

If you are in a Screen session, the output is the ID of the session. Otherwise, the output is an empty line.

### 4.3 Disconnect from a Session

To disconnect from the session and leave it running, press `Ctrl+A` `D` to detach from the session. Later, you can reconnect to the running session.

### 4.4 (Re-)connect to an Old Session

To reconnect to a session, you need to choose which session to reconnect to. Type `screen -ls` at the command line to see the list of ongoing sessions. To connect to an active session, enter at the terminal:

    $ screen -x <session_name>

or

    $ screen -x <session_pid_or_ID>

**Example:** To reconnect to the first session in the example output in Section 4.2, enter any of these commands at the terminal:

    $ screen -x 1306228
    $ screen -x 1306228.test
    $ screen -x test

When you reconnect, the files and programs that were open before you disconnected will still be open.

**Note:** To avoid nesting sessions, do not reconnect to a detached session from inside another running session.

### 4.5 Terminate a Session

When you no longer need a session, terminate it.

There are two ways to terminate an active session:

1. Close all windows in the session. When the last window of the session is closed, the session itself automatically terminates.

1. Access the built-in command line with `Ctrl+A` `:` and issue the command `quit`.

Terminate a detached session by entering this command at the terminal:

    $ screen -S <session_name_or_session_id> -X quit

Recall that you get the list of session names and id's using the command `screen -ls` at the terminal.

The last two methods are not recommended because they directly kill the session and any programs or files open in it. This can sometimes lead to data corruption.

## 5. Regions

In practice, you might want to have different files open side by side. For example, if you are writing and testing a script to process logs, you might want to have the logs open right beside the script. Layouts allow you to arrange windows side by side or one above the other. To do this, split the visible area into panes; these panes are called regions. In each region, you create or attach windows.

**Note:** Regions are a slightly complex topic and need a bit of practice to get comfortable.

### 5.1 Create Regions

`Ctrl+A` `|` splits regions vertically, and `Ctrl+A` `Shift+S` splits horizontally. Splitting resizes (shrinks) the active region and adds a new region either beside it or below it. The new region is initially an empty pane.

Navigate to the new region (described in the next section) and create a new window with `Ctrl+A` `C`. You can also attach an existing window to the active region by navigating to that window (Section 3.3).

### 5.2 Navigate Regions

Press `Ctrl+A` `Tab` to cycle through each region in turn. Similarly, `Ctrl+A` `Shift+Tab` cycles through the regions in reverse order.

**Important Note:** Regions are not retained after detaching from a session. On re-attaching to a session with split regions, the splits are not retained. Only the last active region is retained after reattaching.

### 5.3 Save Layouts

To preserve the split regions while detaching, you need to *save the layout*. Press`Ctrl+A` `:` in the active Screen session and issue this command at the built-in command line :

    layout save default

**Note:** Screen layouts is a large topic and the details are beyond the scope of this introductory guide.

### 5.4 Close Regions

`Ctrl+A` `Shift+X` kills the active region, while `Ctrl+A` `Shift+Q` kills all regions except the active one.

Killing a region does not terminate the window attached to it.

When you kill the active window in a region, the next open window gets attached to the region.

## 6. Copy and Paste Test

For copying text, first enter *copy mode* with `Ctrl+A` `[`. In copy mode, use the arrow keys to navigate the contents of the window. Press the space bar to start marking the text you want to copy. Press the space bar again to stop marking the text, copy the marked text into the buffer, and exit copy mode. Now press `Ctrl+A` `]` to paste the copied text at the prompt.

## 7. An Example

Before using Screen in practice, it is a good idea to check that it works - that you can lose or close the connection to the running session and re-connect to it.

1. Fire up a terminal on your local machine.
1. Optional: SSH into a terminal on a remote server (if you have have access to one).
1. Run `screen -S test` to start a new session named *test*. If you did not log in to a remote server, the Screen session runs on the local machine.
1. Create a new window in this session with `Ctrl+A` `C`.
1. Run `ls` in this window to see the directory listing of the working directory.
1. Close the terminal. It might throw a warning that you have a program running. Close it anyway.
1. Understand here that the terminal is closed but the Screen program is still running on the computer.
1. Fire up a new terminal.
1. Run `screen -ls` to see the list of running sessions. The session named *test* should be on the list.
1. Connect to it using `screen -x test` and you should see the window as it was when you closed the previous terminal.

## 8. Conclusion and Next Steps

This was a quick intro to Screen and how to use it in real life. [The official GNU Screen documentation](https://www.gnu.org/software/screen/manual/screen.html) is a good place to learn all about it. An interesting topic to learn about is [sharing a remote Screen session between multiple users](https://wiki.networksecuritytoolkit.org/index.php/HowTo_Share_A_Terminal_Session_Using_Screen).
