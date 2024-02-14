## Terminology

Server - A server holds multiple sessions. It runs in the background and ensures the persistence of the sessions.

## Common Key Bindings

`CTRL` + `B`, then `X`: Kill the active pane (after pressing `Y` to confirm).

`CTRL` + `B`, then `W`: Show an overview of all active windows and panes. Use the arrow keys to move through the list of windows and see a preview of the selected window. Press `ENTER` to switch to the selected window. Press `ESC` to exit the overview.

## Switch Between Panes

To switch between different open panes, use the arrow keys with the prefix.

`CTRL` + `B`, then `Up`: Go to the pane above the current pane.

`CTRL` + `B`, then `Down`: Go to the pane below the current pane.

`CTRL` + `B`, then `Left`: Go to the pane on the left of the current pane.

`CTRL` + `B`, then `Right`: Go to the pane on the right of the current pane.

### Use Vim Key Bindings to Switch Between Panes

Instead of using the arrow keys to switch between different panes, you can use the Vim keys `h`, `j`, `k`, `l`. To do this, update the Tmux configuration file `~/.tmux.conf`. If this file doesn't exist, create it.

Add these lines (the first line is a comment) to the configuration file to enable pane navigation using Vim keys:

    # hjkl pane traversal
    bind h select-pane -L
    bind j select-pane -D
    bind k select-pane -U
    bind l select-pane -R

After updating the configuration file, restart Tmux. You can load the new configuration without restarting Tmux by *sourcing* the updated configuration file:

    tmux source-file ~/.tmux.conf

## The Meta Key

Many of the key bindings in the following sections are based on the Meta key. On most modern computers and keyboards, the Meta key is not one specific key. Depending on the system settings and the terminal emulator in use, the Meta key could be bound to either `Esc`, `Alt`, `Cmd`, `Windows`, or `Opt`. Where you see `Meta` as part of a key binding, try using the `Esc` key. If it doesn't work, try one of the other keys mentioned above.

## Adjust Pane Size

When a window has multiple panes, you can change the size of individual panes. To change the pane size, press and hold the prefix (`CTRL` + `B`), and simultaneously press one of the arrow keys.

Hold down `CTRL` + `B`, and simultaneously press either `Up` or `Down`: Change the height of the pane.

Hold down `CTRL` + `B`, and simultaneously press either `Left` or `Right`: Change the width of the pane.

The above key combinations resize the pane "continuously" as long as they are held pressed.

To resize the pane one step at a time:

`CTRL` + `B`, then `Meta` + either `Up` or `Down`: Change the height of the pane.

`CTRL` + `B`, then `Meta` + either `Left` or `Right`: Change the width of the pane.

Whether a particular key combination (above) increases or decreases the dimensions of the pane depends on the positioning of that pane relative to other panes.

If the terminal emulator supports mouse use, you can resize panes by dragging their borders.

## Choose a Preset Pane Layout

Tmux has five different preset layouts for arranging multiple panes in a window:

1. All panes are stacked side-by-side.

1. All panes are stacked on top of each other.

1. The main pane takes up the entire screen width and the other panes are stacked side-by-side below it

1. The main pane takes up the entire screen height and the others panes are stacked beside it on top of each other

1. All panes are arranged in a grid.

Start Tmux and create 4 (or any number of) panes. To cycle through the different preset layouts:

`CTRL` + `B`, then `Space`

You can also choose a specific layout:

`CTRL` + `B`, then `Meta` + `1`: Switch to layout 1.

`CTRL` + `B`, then `Meta` + `2`: Switch to layout 2.

`CTRL` + `B`, then `Meta` + `3`: Switch to layout 3.

`CTRL` + `B`, then `Meta` + `4`: Switch to layout 4.

`CTRL` + `B`, then `Meta` + `5`: Switch to layout 5.

## Swapping Pane Positions

When a window has multiple panes, you can move the panes around and swap their positions.

`CTRL` + `B`, then `Meta` + `O`: cycles the positions of the panes clockwise

Note: As described previously, `CTRL` + `B`, then `O` cycles the cursor through the different panes. The above sequence is for cycling the positions of the panes themselves.

`CTRL` + `B`, then `CTRL` + `O`: cycles the positions of the panes counterclockwise.

## Detach and Reattach Individual Panes

In a layout with multiple panes in a window, it is sometimes necessary to move a pane to a different location. Swapping pane positions (as described above) isn't always sufficient, because it disturbs the positions of all the other panes. It is also sometimes necessary to move a pane to a different window in the same session.

Use the command `break-pane` in the Tmux command prompt to detach a pane from the window. `join-pane` reattaches the pane.

To access the Tmux command prompt, press the prefix `CTRL` + `B`, then press `:`. This will open up a prompt at the bottom of the terminal. To exit it without entering any commands, press `CTRL` + `C`.

To detach a pane, switch to the pane to be detached, activate the Tmux command prompt (`CTRL` + `B`, then `:`), and enter the command:

    break-pane -dP

This will detach the active pane and show its "ID". This ID is of the format `X:Y.Z` where X, Y, and Z are numbers. The previous pane becomes the new active pane. The ID is shown on top of the (new) active pane. Note down this ID and then dismiss it by pressing `Esc`.

The detached pane can be reattached next to any pane in any window in the same session. Switch to the pane next to which you want to reattach the detached pane. Activate the Tmux command prompt and enter:

    join-pane -vs X:Y.Z

This reattaches the detached pane below the active pane (`vs` stands for *vertical split*). To reattach the detached pane on the right side of the active pane:

    join-pane -hs X:Y.Z

In the above command, `hs` stands for *horizontal split*.

## Share Tmux Sessions Between Users

### How Sessions Work

When you start Tmux, it creates a new server instance and creates a session within that server.

Tmux clients and servers are separate processes. They communicate via a UNIX socket. By default, Tmux stores sockets in the `/tmp` directory. Sockets for servers created by an individual user are stored under a subdirectory named `tmux-UID` where `UID` is the UNIX UID of that user. A user whose `UID` is 1001 will have its sockets stored in `/tmp/tmux-1001`. The owner of this subdirectory is `user1`. By default, no other user or group can access this subdirectory.

### Compatibility

The steps described in this section have been tested on Tmux versions 3.1 and 3.3 on a vanilla Debian 11 installation. They should be applicable to all recent Tmux versions. 

**Note:** On security-hardened Operating Systems, such as those involving SELinux, changing groups and permissions can need additional steps. Discussing those is beyond the scope of this guide. It is assumed you know how to change groups and permissions on your Operating System.

Tmux version 3.3 (released in June 2022) introduced security related changes in the way session sharing works. These changes involve the use of the newly added `server-access` command. Check the version of your Tmux installation using `tmux -V`. In practice, this leads to one additional step for Tmux versions 3.3 and higher.

### System Description

Consider a system with two users: `user1` with UID 1001 and `user2` with UID 1002.

In order for `user2` to connect to Tmux sessions created by `user1`, it (`user2`) must:

1. have access to the socket of the server created by `user1` - so that clients created by `user2` can communicate with the server created by `user1`

1. (for versions 3.3 and higher) have access to the server, granted (by `user1`) with the `server-access` command

In following the steps below, use the appropriate user IDs and usernames based on your own system. Check the UID (and other details) of a user `user1`:

    $ id user1

The command `id` without any parameters shows the UID (and other details) of the current user.

If there are no additional user accounts on your system, [create a new user](https://www.vultr.com/docs/basics-of-managing-users-on-centos-systems/) before testing these steps.

### Steps

The steps below describe how `user1` can grant access to its Tmux sessions to `user2`.

In a terminal session, log in as `user1`. Start a Tmux session:

    $ tmux

This will create the socket (sub)directory if it did not previously exist. Check the ownership and permissions for the socket directory:

    $ ls -hdl /tmp/tmux-1001

Create a common group for `user1` and `user2`, this will make it easier to manage permissions. Create a new group `tmuxusers`:

    # groupadd tmuxusers

Add `user1` and `user2` to this group:

    # usermod -a -G tmuxusers user1
    # usermod -a -G tmuxusers user2

Change the group of the `/tmp/tmux-1001` directory to `tmuxusers`:

    # chgrp -R tmuxusers /tmp/tmux-1001 

`user2` needs full access to the sockets of Tmux servers started by `user1`. Give the group full permissions on the directory:

    # chmod -R g+rwx /tmp/tmux-1001

Verify that the group of the directory has been changed to `tmuxusers` and that the group has full permissions on the directory:
    
    $ ls -hdl /tmp/tmux-1001

Exit the running Tmux session:

    $ exit

As `user1` (again), create a new Tmux session and specify the name of the socket:

    $ tmux -L socket1

The `-L` option creates a named socket `socket1`, with the path `/tmp/tmux-1001/socket1`.

All commands so far are applicable on all recent versions of Tmux. The next 3 commands are only for Tmux version 3.3 and higher.

Within the newly created Tmux session, give `user2` access to the Tmux server:

    $ tmux server-access -w -a user2

The above command gives `user2` write access to the session. So, `user2` can also enter commands in the session. It is possible to give `user2` only read access. With read access, `user2` can only view the contents of the session but cannot enter any commands. To give `user2` read access, use `-r` instead of `-w` in the above command:

    $ tmux server-access -r -a user2

`user1` can revoke the permissions granted to `user2`:

    $ tmux server-access -d user2

The last 3 commands are only for Tmux version 3.3 and higher. Further steps are applicable to all recent versions of Tmux.

Change the group of the new socket file:

    # chgrp tmuxusers /tmp/tmux-1001/socket1

Verify that the group of the socket file is `tmuxusers` and that group has access to the file:

    # ls -l /tmp/tmux-1001/socket1

Check the ID(s) of the open session(s):

    $ tmux ls

In another terminal, log in as `user2`. Start Tmux and specify the path of the named socket created by `user1`:

    $ tmux -S /tmp/tmux-1001/socket1 ls

The above command shows the list of sessions on the server started by `user1` and listening on `socket1`. Attach to the session:

    $ tmux -S /tmp/tmux-1001/socket1

If there are many open sessions, attach to a specific session (whose ID is N):

    $ tmux -S /tmp/tmux-1001/socket1 attach -t N

`user2` is now connected to the Tmux session of `user1`.

