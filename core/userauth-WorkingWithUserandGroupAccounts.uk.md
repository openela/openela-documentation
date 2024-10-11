<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Working With User and Group Accounts

By default, a new installation of Enterprise Linux uses local user and group accounts for authentication, permissions handling, and access to resources. When working with local accounts for users and groups, you use three main commands: `useradd`, `groupadd`, and `usermod`. Through these commands and their different options, you can add or delete users and groups, as well as modify user or group settings.

## About User and Group Accounts

To implement system authentication, Enterprise Linux uses two types of accounts: user and group. Together, these accounts hold information such as passwords, home directories for users, login shells, group settings and memberships, and so on. The information is used to ensure that only authorized logins are granted access to the system. Users without credentials, or whose credentials do not match the information in these accounts, are locked out of the system.

By default, user and group information is located locally in the system. However, in an enterprise environment that might have hundreds of servers and thousands of users, user and group account information is better stored in a central repository rather than in files on individual servers. User and group information is configured on a central server and then retrieved through services such as the Lightweight Directory Access Protocol \(LDAP\) or the Network Information Service \(NIS\). Central management of this information is more efficient than storing and configuring user and group information locally.

## Where User and Group Information Is Stored Locally

Unless you select a different authentication mechanism during installation or use the `authselect` command to create an authentication profile, Enterprise Linux verifies a user's identity by using the information that is stored in the `/etc/passwd` and `/etc/shadow` files.

The `/etc/passwd` file stores account information for each user such as his or her unique user ID \(or _UID_, which is an integer\), username, home directory, and login shell. A user logs in using his or her username, but the operating system uses the associated UID. When the user logs in, he or she is placed in his or her home directory and his or her login shell runs.

The `/etc/group` file stores information about groups of users. A user also belongs to one or more groups, and each group can contain one or more users. If you can grant access privileges to a group, all members of the group receive the same access privileges. Each group account has a unique group ID \(_GID_, again an integer\) and an associated group name.

By default, Enterprise Linux implements the _user private group_ \(_UPG_\) scheme where adding a user account also creates a corresponding UPG with the same name as the user, and of which the user is the only member.

By default, both users and groups use shadow passwords, which are cryptographically hashed and stored in `/etc/shadow` and `/etc/gshadow` respectively. These shadow password files are readable only by the administraor. The administrator can set a group password that a user must enter to become a member of the group. If a group does not have a password, a user can only join the group if the administrator adds that user as a member.

A user can use the `newgrp` command to log into a new group or to change the current group ID during a login section. If the user has a password, he or she can add group membership on a permanent basis. See the `newgrp(1)` manual page.

The `/etc/login.defs` file defines parameters for password aging and related security policies.

For more information about the content of these files, see the `group(5)`, `gshadow(5)`, `login.defs(5)`, `passwd(5)`, and `shadow(5)` manual pages.

## Creating User Accounts

1. Type the following command:

   ```
   sudo useradd [*options*] *username*
   ```

   You can specify options to change the account's settings from the default ones.

   By default, if you specify a username argument with no additional options, `useradd` creates a locked user account using the next available UID and assigns a user private group \(UPG\) rather than the value defined for `GROUP` as the user's group.

   **Note:**

   A maxmimum of 32 characters can be used for a username.

   Usernames can begin with lowercase \(a-z\) and uppercase \(A-Z\) letters, digits \(0-9\), or underscores \(\_\).

   Usernames can contain all starting characters but can also contain dashes \(-\) and can end with a dollar character \($\).

   Fully numeric usernames and usernames containing only a period \(.\) or double period \(..\) are disallowed.

   Usernames starting with a period \(.\) character, although allowed, are discouraged because they can cause issues for some software and resulting home directories are also likely to be hidden.

2. Assign a password to the account.

   ```
   sudo passwd *username*         
   ```

   The command prompts you to enter a password for the account.

   To change the password non-interactively \(for example, from a script\), use the `chpasswd` command instead:

   ```
   echo "*username*:*password*" | chpasswd
   ```

You can use the `newusers` command to create several user accounts at the same time.

For more information, see the `chpasswd(8)`, `newusers(8)`, `passwd(1)`, and `useradd(8)` manual pages.

## Locking an Account

To lock a user's account, use the `passwd -l` command.

```
sudo passwd -l *username*
```

To unlock the account, use the`passwd -u` command.

```
sudo passwd -u *username*
```

For more information, see the `passwd(1)` manual page.

## Modifying or Deleting User Accounts

To modify a user account, use the `usermod` command.

```
sudo usermod [*options*] *username*
```

For example, to add a user to a supplementary group \(other than the user's default login group\):

```
sudo usermod -aG *groupname* *username*
```

You can use the `groups` command to display the groups to which a user belongs, for example:

```
sudo groups *username*
```

To delete a user's account, use the `userdel` command:

```
sudo userdel *username*               
```

For more information, see the `groups(1)`, `userdel(8)` and `usermod(8)` manual pages.

## Changing Default Settings for User Accounts

To display the default settings for a user account, use the following command:

```
sudo useradd -D
```

```
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
```

`INACTIVE`: Specifies after how many days the system locks an account if a user's password expires. If set to 0, the system locks the account immediately. If set to -1, the system doesn't lock the account.

`SKEL`: Defines a template directory, whose contents are copied to a newly created user’s home directory. The contents of this directory matches the default shell defined by `SHELL`.

You can specify options to `useradd -D` to change the default settings for user accounts. For example, to change the defaults for `INACTIVE`, `HOME` and `SHELL`:

```
sudo useradd -D -f 3 -b /home2 -s /bin/sh
```

**Note:**

If you change the default login shell, you would probably also create a `SKEL` template directory that contains contents that are appropriate to the new shell.

If you specify `/sbin/nologin` for a user's `SHELL`, that user can't log into the system directly but processes can run with that user's ID. This setting is typically used for services that run as users other than `root`.

The default settings are stored in the `/etc/default/useradd` file.

For more information, see [Configuring Password Ageing](userauth-WorkingWithUserandGroupAccounts.md#) and the `useradd(8)` manual page.

## Creating Groups

To create a group, use the `groupadd` command.

```
sudo groupadd [*options*] *groupname*
```

Typically, you might want to use the `-g` option to specify the group ID \(GID\). For example:

```
sudo groupadd -g 1000 devgrp
```

For more information, see the `groupadd(8)` manual page.

## Modifying or Deleting Groups

To modify a group, use the `groupmod` command:

```
sudo groupmod [*options*] *username*
```

To delete a user's account, use the `groupdel` command:

```
sudo groupdel *username*
```

For more information, see the `groupdel(8)` and `groupmod(8)` manual pages.

## Configuring Group Access Modes to Directories

Users whose primary group isn't a UPG have a `umask` of 0022 set by `/etc/profile` or `/etc/bashrc`, which prevents other users, including other members of the primary group, from modifying any file that the user owns.

A user whose primary group is a UPG has a `umask` of 0002. No other user has the same group.

To grant users in the same group write access to files within the same directory, change the group ownership on the directory to the group, and set the `setgid` bit on the directory:

```
sudo chgrp *groupname* *directory*
sudo chmod g+s *directory*
```

Files that are created in such a directory have their group set to that of the directory rather than the primary group of the user who creates the file.

The restricted deletion bit prevents unprivileged users from removing or renaming a file in the directory unless they own either the file or the directory.

To set the restricted deletion bit on a directory:

```
sudo chmod a+t *directory*    
```

For more information, see the `chmod(1)` manual page.

## Configuring Password Ageing

To specify how users' passwords are aged, edit the following settings in the `/etc/login.defs` file:

<table><thead><tr><th>

Setting

</th><th>

Description

</th></tr></thead><tbody><tr><td>

`PASS_MAX_DAYS`

</td><td>

Maximum number of days for which a password can be used before it must be changed. The default value is 99,999 days.

</td></tr><tr><td>

`PASS_MIN_DAYS`

</td><td>

Minimum number of days allowed between password changes. The default value is 0 days.

</td></tr><tr><td>

`PASS_WARN_AGE`

</td><td>

Number of days before a password expires that a warning is displayed. The default value is 7 days.

</td></tr><tbody></table>
For more information, see the `login.defs(5)` manual page.

To change how long a user's account can be inactive before it's locked, use the `usermod` command. For example, to set the inactivity period to 30 days:

```
sudo usermod -f 30 *username*
```

To change the default inactivity period for new user accounts, use the `useradd` command:

```
sudo useradd -D -f 30
```

A value of -1 specifies that user accounts aren't locked because of inactivity.

For more information, see the `useradd(8)` and `usermod(8)` manual pages.
