# How to Harden Server SSH Access Using Advanced OpenSSH Features

## Introduction

SSH (Secure Shell Protocol) is a network communication protocol. It allows users (clients) to securely connect to a remote computer (server) over an insecure network (such as the Internet). SSH works on a client-server architecture. The client application is commonly referred to as SSH. The server application is commonly referred to as SSHD (SSH Daemon). A daemon is a long-running process that manages a particular service, for example, SSH access. This article discusses SSH configuration in the context of servers, not clients. Hence, SSH configuration implicitly refers to configuring the SSH daemon.

OpenSSH (OpenBSD Secure Shell) refers to a set of related utilities based on the SSH protocol. It includes [`scp`](https://www.vultr.com/docs/securely-transfer-files-over-the-private-network-using-scp-or-rsync/), [`sftp`](https://www.vultr.com/docs/setup-sftp-user-accounts-on-ubuntu-20-04/), [`ssh-keygen`](https://www.vultr.com/docs/how-to-use-ssh-with-vultr-servers/), and a few other applications. OpenSSH, being a spin-off of OpenBSD, was developed with many security-specific features. This article shows how to use advanced features of OpenSSH to enhance the security of an SSH server.

### Prerequisites 

A server exposed to the internet has many attack vectors. SSH is a common target because it gives the intruder direct access to the remote computer. The scope of this article is limited to hardening SSH security on the server. A secure SSH configuration minimizes the chances of an intruder gaining unauthorized SSH access. It does not protect against other attack vectors (such as SQL injection).

This guide is based on OpenSSH 9.2 on Debian 12. The commands and recommendations in this guide apply to versions of OpenSSH higher than 9.0. To take advantage of [security enhancements](https://www.openssh.com/txt/release-9.0), it is advised to use OpenSSH 9.0 or higher on a production server. Debian 11 uses OpenSSH 8.4 and Debian 10 uses OpenSSH 7.9. In case you must use an older Operating System, consider [manually installing the latest OpenSSH from the source](https://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/INSTALL).

Being a security oriented framework, OpenSSH has sensible default values for many configuration parameters (such as, `PermitEmptyPasswords`, `PubkeyAuthentication`, `X11Forwarding`, and many others). These settings do not need to be changed, and hence, are not discussed here.

It is assumed you are comfortable [setting up and using SSH to connect to a server](https://www.vultr.com/docs/how-to-use-ssh-with-vultr-servers/). Before hardening SSH Access, set up SSH on the server and connect to it:

 * [Generate SSH Keys locally](https://www.vultr.com/docs/how-do-i-generate-ssh-keys/)
 * [Deploy a new server with the key](https://www.vultr.com/docs/deploy-a-new-server-with-an-ssh-key/)
 * [Connect to the server using the key](https://www.vultr.com/docs/connect-to-a-server-using-an-ssh-key/)

Use an editor like Nano or [Vim](https://www.vultr.com/docs/getting-started-with-vim/) to edit text files.

### First Principles

Broadly, hardening an SSH server involves:

 * Controlling how users can log in
 * Limits on login attempts 
 * Deciding which users can log in
 * Restricting what users can do after logging in

Most configuration changes are based on one of the themes above.

SSH (daemon) configuration is controlled using a text file titled `sshd_config`. On most Unix/Linux-based systems (Debian, Ubuntu, FreeBSD, Red Hat, Arch Linux, and others), this file is located at `/etc/ssh/sshd_config`. Each line in this file represents a specific setting. To implement the changes discussed in the following sections, add or update the lines in the `sshd_config` file. Comments in the configuration file are prefixed with a `#`. 

Run SSHD-related commands in `root` mode. Before making any changes, save a working copy of the file:

    # cp /etc/ssh/sshd_config /etc/ssh/sshd_config.BAK.DATE

After making changes to the configuration file, test it:

    # sshd -t

The above command shows errors in the configuration. If it finds no errors, it exits quietly. To run an extended test and output the complete configuration, use the `-T` option:

    # sshd -T

After editing and testing the file, restart the daemon to implement the changes. On Linux-based (Debian, Red Hat, and others) systems, use `systemctl`:

    # systemctl restart sshd

On FreeBSD:

    # service sshd restart

Before updating the `sshd_config` file with the configuration lines in the following sections, check if the file already has a setting for that parameter. If it does, edit and update that line, instead of adding that parameter again. In general, SSHD considers only the first value of a setting, and ignores later occurrences.

## Control How Users can Log in

This section discusses the first layer of defense in hardening SSH access. It shows how to impose controls on the authentication process. 

### Generate Strong Key Pairs

The `ssh-keygen` command generates keys based on different algorithms: RSA, DSA, ECDSA, and ED25519. DSA is an older algorithm that is no longer considered secure. RSA keys are considered secure only with keys longer than 2048 bits. ECDSA and ED25519 are both newer algorithms based on elliptic curve cryptography. Some security researchers have raised concerns about the risk of back-doors in ECDSA. Hence, it is advisable to use ED25519 keys. Generate an ED25519 key pair:

    $ ssh-keygen -t ed25519 -f /path/to/key/file

By default, `ssh-keygen` generates 2048-bit RSA keys. This is to support older SSH clients. 

### Accept Only Secure Keys

By default, SSHD accepts RSA, ECDSA, and ED25519 keys. To restrict it further to only accept ED25519 keys:

    PubkeyAcceptedAlgorithms ssh-ed25519-cert-v01@openssh.com,sk-ssh-ed25519-cert-v01@openssh.com,ssh-ed25519,sk-ssh-ed25519@openssh.com

Do this only after educating all users to log in with ED25519 keys. If the server accepts only one type of key, and the user tries to log in with another key type, they get an error.

### Secure Key Exchange Algorithms

Before the server can validate (authenticate) the client, it securely exchanges cryptographic keys with the client. By default, recent versions of SSH use only Diffie-Hellman algorithms based on 256-character and 512-character hashes to exchange keys. OpenSSH, [since version 8.2](https://www.openssh.com/txt/release-8.2), no longer accepts any 128-character (1024-bit) SHA1 hashes by default for the initial key exchange. 

    KexAlgorithms sntrup761x25519-sha512@openssh.com,curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group14-sha256

On recent versions of OpenSSH, there is no need to add the above line. It represents the default settings.

### Allow Only Key-based Logins

Explicitly specify that keys are the only accepted authentication method: 

    AuthenticationMethods publickey

### Disallow Passwords and Other Authentication Methods

Compared to SSL keys, it is more realistic to brute-force a password. Hence, it is safer to disable password-based logins to begin with.

    PasswordAuthentication no

By default, this is set to `yes`. 

Also disable other authentication mechanisms:

    ChallengeResponseAuthentication no
    KerberosAuthentication no
    GSSAPIAuthentication no

### Disable Root Login

If the `root` account is compromised, it automatically compromises the entire server. Disable root logins over SSH:

    PermitRootLogin no

By default, this is parameter is set to `prohibit-password`. This means `root` can log in with an SSH key, but not with a password.

By default, when you start a new instance, `root` is the only user. [Set up SSH for a new non-root user](https://www.vultr.com/docs/using-your-ssh-key-to-login-to-non-root-users/) (with `sudo` rights) before disabling root logins. If you disable root logins and root is the only available user, you will get locked out of your own server.

### Change the Default Port

By default, SSH connects over port 22. Hence, most malicious actors (scripts, bots, worms, and so on) try to gain unauthorized access to this port. Update the default port:

    Port 12345

Changing the port number is considered "security by obscurity". It does not prevent a determined attacker from quickly discovering the correct port number. However, it can save the server from a majority of scripts scanning for vulnerabilities. This keeps the log files cleaner and lets you focus on more serious threats. It also necessitates educating users about the right port number to use. Use the `-p` option to specify the correct port number while logging in via `ssh`:

    $ ssh -p 12345 root@remote.server.ip.address

If you change the default port number, ensure that the firewall also allows connections on that port. For example, while using UFW on Ubuntu:

    # ufw allow 12345

### Idle Timeout

An idle open connection is a threat waiting to be exploited. When a client has not issued any commands for a while (`ClientAliveInterval`), the server sends a message to check if the client is still active. If the client is active, it responds to this message. The server tries this process a few times (`ClientAliveCountMax`) before concluding that the client is inactive and terminating the connection.

    ClientAliveInterval 300 # in seconds

    ClientAliveCountMax 3

The settings above allow a client to stay inactive for 15 minutes before the connection is closed. Values of these parameters must balance the need for security and user experience. Low values cause the SSH session to exit after a short idle period. This makes it more secure. However, users will be forced to log in repeatedly, degrading their experience.

## Limits on Login Attempts

The controls shown in the previous section prevent an attacker from gaining unauthorized access to the server. However, the server still needs to process each login attempt. So, many unsuccessful login attempts can throttle the processor and construe a Denial of Service (DoS) attack. This section shows some ways to mitigate potential DoS attacks.

### Limit Number of Login Attempts

Repeatedly trying to brute-force the key wastes server resources. Limit the number of times someone can try to log in:

    MaxAuthTries 6

The default value is 6. When a user exceeds the number of `MaxAuthTries`, they get an error message. 

Be careful with this setting, it can backfire on legitimate users. For example, if a user has many keys, SSH tries each of the keys sequentially till it finds a match. Each key tried counts as one login attempt. Inform users with many SSH keys on their local machine to enter the path to the right SSH key while logging in. For example:

    $ ssh -i /path/to/key/file user@remote.server.ip.address

### Lower Login Grace Times

From the time the user connects to the server, SSHD waits for 120 seconds (by default) for the user to successfully authenticate. If the user has not authenticated within this time, the connection is closed. Setting this value to a lower number helps to limit the number of pending connections. Lower the login grace time to 30 seconds.

    LoginGraceTime 30

### Limit the Number of Unauthenticated Connections

By default, if 10 open connections are pending authentication, SSHD starts rejecting 30% (at random) of new connection attempts. When  100 connections are pending authentication, it rejects all new connections. Control these numbers using `MaxStartUps`. The default value is set to `10:30:100`. Update it so that SSHD 1) starts rejecting new connections if 5 open connections are pending authentication and 2) does not accept new connections if more than 10 unauthenticated connections are pending.

    MaxStartUps 5:30:10

These values depend on the server's use case. Larger enterprise servers with many users need higher values. The example shown above is suitable for smaller servers with 1 or 2 simultaneous SSH users.

## Decide Which Users can Log in

The previous sections discuss how to secure login mechanisms. It is also advisable to restrict which users and groups can log into the server. SSHD has four primitives for this: `AllowUsers`, `DenyUsers`, `AllowGroups`, and `DenyGroups`. 

To allow only `user1` and `user2` to log in via SSH:

    AllowUsers user1 user2

With the above rule, `user1` and `user2` can log in from any IP address. To allow specific users to log in only from specific IP addresses:

    AllowUsers user1@trusted.ip.address.1 user2@trusted.ip.address.2 

To allow any user to log in from a few different (trusted) IP addresses:

    AllowUsers *@trusted.ip.address.1 *@trusted.ip.address.2 *@trusted.ip.address.3 

To allow any user from a block 8 subnet:

    AllowUsers *@trusted.ip.subnet/8

For example, if you allow users from the subnet `192.168.121/8`, it means users from IP addresses `192.168.121.1` to `192.168.121.255` can log in.
    
It is also possible to use wild cards in IP addresses. The above rule can also be written as:

    AllowUsers *@192.168.121.*

Rules using `AllowGroups` have a similar syntax to the above examples using `AllowUsers`. Rules on a group apply to all members of the group. For example:

    AllowGroups ssh_group

The above rule specifies that only users belonging to a specific group, `ssh_group`, can log in:

If the `AllowUsers` and `AllowGroups` directives are not used, all users and groups are allowed to log in (subject to other restrictions discussed in the previous sections). However, when the `AllowUsers` and/or `AllowGroups` directives are used to explicitly grant access to specific users/groups, then no other users/groups are allowed to log in. 

For example, if the configuration includes a line `AllowUsers user1`, then `user2` is no longer allowed to log in. To allow `user2` to log in, add the username to the `AllowUsers` rule (`AllowUsers user1 user2`). You can also use the `AllowGroups` rule to allow a group `user2` belongs to (`AllowGroups group_containing_user2`). 

For a particular user to log in, there must be no rules blocking their access. If, for a particular user, the `AllowUsers` and `DenyUsers` (or `DenyGroups`) directives conflict, the blocking rules take precedence. For example, if you have a rule `AllowUsers user1` and another rule `DenyUsers user1`, then `user1` cannot log in. Similarly, if `AllowGroups` is used, it must include the groups of all the users who need to be able to log in. 

By default, SSH denies connections. So, it is advisable to only allow specific users/groups and deny everyone else, instead of allowing everyone and denying specific users/groups. If you have a single rule `DenyUsers user1` and no other `AllowUsers` or `AllowGroups` rules, then all users except `user1` are allowed to log in. This approach is unsafe and not recommended.

## Restrict What Actions Logged-in Users can Take

In addition to allowing specific users/groups and limiting the authentication methods as shown in earlier sections, it is also possible to restrict the actions a particular user can perform on the server. Having different layers of security is called "security-in-depth". 

### User-specific and Group-specific Restrictions

Apply custom rules for individual users with the `Match` directive: 

    Match User user1

`Match` statements appear at the end of the file. The `Match` block continues till the end of the file, or till another `Match` statement is encountered. The configuration in the `Match` block overrides the default settings in the rest of the file (for the respective user or group). Use a `Match` block to restrict a user to run only a specific command:

    Match User user1
    ForceCommand "top"

The above `Match` block allows `user1` to only run the command `top`. After logging in, the command `top` runs. When the user exits `top` (Press `Q` to exit `top`), the session exits. 

A common use case for the `Match` block is [setting up SFTP on the server. You can configure it with a dedicated user account that can only run SFTP and nothing else](https://www.vultr.com/docs/setup-sftp-user-accounts-on-ubuntu-20-04/).

In addition to specific commands, you can also restrict a user to run only a specific script:

    ForceCommand "/path/to/script.sh"

To Match a specific group instead of a specific user, use `Match Group group1` instead of `Match User user1`.

### Key-specific Restrictions

To achieve another layer of security, attach restrictions to the use of individual keys. The `~/.ssh/authorized_keys` file contains the list of all the keys authorized to access that specific user account. By default, each key listing is in the following format:

    ALGORITHM KEY USER@HOST

For example:

    ecdsa-sha2-nistp521 AAAAE2VjZ.....KEY-BODY...Lf+FNXv00Pd+A== an@fbsdvm

In the above example, `ecdsa-sha-nistp521` is the algorithm used to generate the key. The body of the key is the block of cipher text starting with `AAAA` and ending with `00Pd+A==`. `an@fbsdvm` is the user ID and hostname of the user authorized to use this key. Key-specific restrictions are prepended to the line corresponding to the key. To restrict the user of the key to run only a specific command, use the `command=` primitive. For example, to allow to use only the `ls` command, prepend the following to the line with the key:

    command="ls"

After prepending, the example key line looks like this:

    command="ls" ecdsa-sha2-nistp521 AAAAE2VjZ.....KEY-BODY...Lf+FNXv00Pd+A== an@fbsdvm

When a user logs in with this key, the command `ls` executes, and then the session exits.

### Restrict Users to a Set of Commands

The previous subsections showed how to restrict a user to executing a single command or script. To allow a user to run a few different commands, make multiple keys, one to run each command. You can also make a custom script to allow the user to choose one of several commands. Execute this script using the `ForceCommand` or `command=` primitives. 

Below is a rudimentary example of such a script:

    #!/bin/bash

    echo "1. ls"
    echo "2. top"
    read -p 'Choose the number corresponding to one of the above options: ' choice # Read the choice from the user
    case $choice in
        1)
            ls
            ;;
        2)
            top
            ;;
        *)
            exit
            ;;
    esac

Save this script as `/home/user1/ssh_script.sh`. Change the permissions to make the script executable:

    # chmod u+x /home/user1/ssh_script.sh

Restrict the key of the user to only run this specific script:

    command="/home/user1/ssh_script.sh" ecdsa-sha2-nistp521 AAAAE2VjZ.....KEY-BODY...Lf+FNXv00Pd+A== an@fbsdvm

You can also use `Match` and restrict `user1` to running only this script:

    Match User user1
    ForceCommand "/home/user1/ssh_script.sh"

When `user1` logs in, this script executes. The user can then choose from the options shown: 1 for `ls` and 2 for `top`. The system runs the chosen command and then closes the connection.

## Display Banner

Finally, It is helpful to show a banner message to logged-in users to catch their attention and educate them about adopting the right security policies. Banners also inform users about their legal rights and responsibilities. Create a new file `/etc/motd`:

    # touch /etc/motd

Below is an example of a banner message:

    ************************************************************************
    This is a private computer. 
    By using this system, you consent to having all activity monitored. 
    System administrators may provide activity logs to law enforcement. 
    Unauthorized users or usage may be subject to legal action. 
    Do not store private data on this computer. 
    Users are responsible for securely storing their personal access keys.
    ************************************************************************

Edit the `/etc/motd` file using a text editor and save it. After the file is saved, users get the banner message in all new sessions.

## Conclusion

This article gives an overview of the OpenSSH settings commonly used to harden SSH security. To get a better understanding of the configuration options, use the fifth section man page for `sshd_config`:

    $ man 5 sshd_config

The fifth section of the man pages describes file formats and conventions. For the `sshd_config` configuration file, the man page explains the meaning of each option and specifies the default settings for it. Many configuration items are by default set to sensible values.

In high-security environments, in addition to the settings described in this article, you can further enhance security by restricting users to sandboxed environments. On Linux systems, use [chroot jails](https://help.ubuntu.com/community/BasicChroot) to create self-contained sandboxes. Use the `ChrootDirectory` directive in a `Match` block to restrict the user to the chroot jail. Each chroot jail must be self-contained - it should include the necessary binaries to run a shell and any other programs the user needs access to. 

