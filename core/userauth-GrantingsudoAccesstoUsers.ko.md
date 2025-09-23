<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Granting sudo Access to Users

In Enterprise Linux, only administrators can perform privileged tasks on the system.

To grant users additional privileges, an administrator can use the `visudo` command to either create a new configuration file in the `/etc/sudoers.d` directory or modify the `/etc/sudoers` file.

Privileges that an administrator assigns by using configuration files in the `/etc/sudoers.d` directory are preserved between system upgrades and skipped automatically by the `sudo` command if they are invalid. Administrators can also change file ownership and permissions for each configuration file. For more information, see [Adding User Authorizations in the sudoers.d Directory](userauth-GrantingsudoAccesstoUsers.md#).

Alternatively, an administrator can assign privileges directly in the `/etc/sudoers` file by using the `visudo` command. For more information, see [Adding User Authorizations in the sudoers File](userauth-GrantingsudoAccesstoUsers.md#).

## About Administrative Access on Enterprise Linux

By default, any user can elevate to a `root` shell by running the `su` command and provide the `root` user password:

```
su
```

```nocopybutton
Password:
```

Any user can also perform single administrative tasks in the same shell, but those commands can't be run until that user provides the `root` user password:

```
su -c "whoami"
```

```nocopybutton
Password:
```

```nocopybutton
root
```

Elevating to a `root` shell by using the `su` command can work for single user environments and workstations because only one person needs to administer the system and know the `root` user password. However, this approach is inadequate for shared systems with several users and administrators that require varying levels of access.

Don't share the `root` user password with anyone else or let remote users sign in as the `root` user, both of these actions constitute poor and highly risky security practices.

The `sudo` command is better suited for shared systems because any user can supply their own credentials when they elevate to a `root` shell:

```
sudo -s
```

Users can exit from the `root` shell in the same way they would have if they had elevated directly with the `su` command and provided the `root` user password:

```
exit
```

In addition, users can run the `sudo` command to perform single administrative tasks with elevated permissions:

```
sudo whoami
```

```nocopybutton
root
```

For more information, see the `su(1)`, `sudo(8)` and `sudoers(5)` manual pages.

**Note:**

You can optionally disable the `root` user during the Enterprise Linux installation process and grant `sudo` administrator privileges to the first user.

## Using the sudo Command

If a user has been granted `sudo` access then that user can run administrative commands with elevated privileges:

```
sudo *command*
```

Depending on the `sudoer` configuration, the user might also be prompted for a password.

In some situations, a user might have set environment variables that they want to reuse or preserve while running elevated commands, and they can do so by using the `-E` option.

For example, if the Enterprise Linux system is connected to an enterprise intranet or virtual private network \(VPN\), proxy settings might apply to obtain outbound Internet access.

The environment variables on which terminal commands rely for proxy access are `http_proxy`, `https_proxy` and `no_proxy`, and you can set them in the `~/.bashrc` configuration file:

```
export http_proxy=http://proxy.example.com:8080
export https_proxy=https://proxy.example.com:8080
export no_proxy=localhost,127.0.0.1
```

Run the `source` command to refresh the session environment variables without signing out:

```
source ~/.bashrc
```

The `sudo` command can use the proxy settings that you have configured as environment variables within the user's session. For example, to run the `curl` command with administrative privileges:

```
sudo -E curl https://www.example.com
```

**Note:**

An administrator can optionally set system-wide proxy environment variables by configuring them in a shell script and then saving that file in the `/etc/profile.d/` directory.

You can also use `sudo` access to start an elevated `root` shell. The `-s` option elevates the user to a `root` shell as the `root` user. The `-i` option elevates the user to a `root` shell while preserving both the user profile and shell configuration:

```
sudo -i
```

When you have finished running administrative commands, exit the `root` shell and return to the standard user privilege level by using the `exit` command.

## Using the visudo Command

To edit the `/etc/sudoers` file in the `vi` text editor without risking any change conflicts from other users on the system, use the `visudo` command:

```
sudo visudo
```

To learn more about how to configure the the `/etc/sudoers` file, see [Adding User Authorizations in the sudoers File](userauth-GrantingsudoAccesstoUsers.md#) and the `visudo(8)` manual page.

Administrators can also use the `visudo` command to manage permission files for individual users in the `/etc/sudoers.d/` directory. For more information, see [Adding User Authorizations in the sudoers.d Directory](userauth-GrantingsudoAccesstoUsers.md#).

## Adding User Authorizations in the sudoers.d Directory

To set privileges for a specific user, add a file for them in the `/etc/sudoers.d` directory. For example, to set `sudo` permissions for the user `alice`:

```
sudo visudo -f /etc/sudoers.d/alice
```

You can append permissions to `/etc/sudoers.d/alice` in the following format:

```
*username*        *hostname*=*command*
```

`username` is the name of the user, `hostname` is the name of any hosts for which you're defining permissions, and `command` is the permitted command with full executable path and options. If you don't specify options, then the user can run the command with full options.

For example, to grant the user `alice` permission to install packages with the `sudo dnf` command on all hosts:

```
alice           ALL = /usr/bin/dnf
```

You can also add several comma separated commands on the same line. To allow the user `alice` to run both the `sudo dnf` and `sudo yum` commands on all hosts:

```
alice           ALL = /usr/bin/dnf, /usr/bin/yum
```

The `alice` user still needs to use `sudo` when they run privileged commands:

```
sudo dnf install *package*
```

## Adding User Authorizations in the sudoers File

To set user privileges directly in the `/etc/sudoers` file, run the `visudo` command without specifying a file location:

```
sudo visudo
```

You can append permissions to the `/etc/sudoers` file in the same format that you would if you were adding those permissions to user files in the `/etc/sudoers.d/` directory.

In both cases, you can use aliases to permit broader permission categories instead of specifying each command individually. The `ALL` alias functions as a wildcard for all permissions, so to set the user `bob` to have sudo permission for all commands on all hosts:

```
bob             ALL=(ALL)       ALL
```

More aliased categories are listed in the `/etc/sudoers` file and the `sudoers(5)` manual page. You can create aliases in the following format:

```
Cmnd_Alias *ALIAS* = *command*
```

In addition, you can also add several comma separated aliases on the same line. For example, to grant the user `alice` permission to manage system services and software packages:

```
Cmnd_Alias SOFTWARE = /bin/rpm, /usr/bin/up2date, /usr/bin/yum
Cmnd_Alias SERVICES = /sbin/service, /sbin/chkconfig, /usr/bin/systemctl start, /usr/bin/systemctl stop, /usr/bin/systemctl reload, /usr/bin/systemctl restart, /usr/bin/systemctl status, /usr/bin/systemctl enable, /usr/bin/systemctl disable
alice           ALL= SERVICES, SOFTWARE
```

Both users still need to use `sudo` when they run privileged commands:

```
sudo systemctl restart *service*
```

## Using Groups to Manage User Authorizations

Instead of specifying different levels of `sudo` access for each individual user you can optionally manage `sudo` access at group level by adding the `%` symbol to the group name.

For example, to define permissions for an existing group called `example` in the `/etc/sudoers.d/` directory and then add the user `alice` to that group:

1. Create the `/etc/sudoers.d/example` file by using the `visudo` command:

    ```
    sudo visudo /etc/sudoers.d/example
    ```

2. Grant the `example` group permissions to manage system services and software packages:

    ```
    %example        ALL= SERVICES, SOFTWARE
    ```

3. Add the the `alice` user to the `example` group:

    ```
    sudo usermod -aG example alice
    ```

Or, you can set group permissions directly in the `/etc/sudoers` file. For example, to grant the user `bob` full `sudo` access on all hosts, enable the existing group `wheel`, and then add the user `bob` to it:

1. Open the `/etc/sudoers` file by using the `visudo` command:

    ```
    sudo visudo
    ```

2. Remove the comment `#` symbol from the beginning of the following line in the `/etc/sudoers` file:

    ```
    %wheel          ALL=(ALL)       ALL
    ```

3. Add the `bob` user to the `wheel` group to grant them full `sudo` access on all hosts:

    ```
    sudo usermod -aG wheel bob
    ```


