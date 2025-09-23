<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# About System Authentication

Authentication is a way of implementing system security by verifying the identity of an entity, such as a user, to a system. A user logs in by providing a username and a password, and the OS authenticates the user's identity by comparing this information to data stored on the system. If the login credentials match and the user account is active, the user is authenticated and can successfully access the system.

## Authentication in Enterprise Linux

In Enterprise Linux, authentication is profile-based. Each profile has predefined features that use different mechanisms to authenticate system access.

The following profiles are available:

- `sssd` profile: Uses the `sssd` service to perform system authentication.

- `winbind` profile: Uses the `winbind` service to perform system authentication.

- The `minimal` profile: Uses system files to perform system authentication for local users.

After an Enterprise Linux installation, the `sssd` profile is selected by default to manage authentication on the system. This profile covers most authentication cases including PAM authentication, Kerberos, and so on.

System authentication isn't restricted to using only the profiles in Enterprise Linux. If preferred, you can also use profiles that might be supplied by vendors. You can also create customized profiles to enforce authentication that complies organizational requirements.

As an added flexibility, you can also reconfigure profiles by revising their active features. For example, you can set the profile to use various different backend directory services such as LDAP, FreeIPA, and Active Directory. Also, you can use SSSD with a directory service to centralize and simplify user and group management in an environment where many users and systems with different access requirements exist.

## Profiles and Supported Features

Each profile has associated features you can enable to make the profile's service perform a particular method of authentication, such as smart card authentication, fingerprint authentication, kerberos, and so on. After you select a profile and enable preferred features, `authselect` automatically reads the appropriate configuration files of those features to run the relevant authentication processes. Every user who logs in to the host is authenticated based on that configured profile.

The following tables shows the profiles and their corresponding supported features:

| Feature Name                     | Description                                                                                                                                                            |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `with-faillock`                  | Lock the account after too many authentication failures.                                                                                               |
| `with-mkhomedir`                 | Create home directory on user's first log in.                                                                                                          |
| `with-ecryptfs`                  | Enable automatic per-user ecryptfs.                                                                                                                    |
| `with-smartcard`                 | Authenticate smart cards through SSSD.                                                                                                                 |
| `with-smartcard-lock-on-removal` | Lock the screen when the smart card is removed. Requires that `with-smartcard` is also enabled.                                        |
| `with-smartcard-required`        | Only smart card authentication is operative; others, including password, are disabled. Requires that `with-smartcard` is also enabled. |
| `with-fingerprint`               | Authenticate through fingerprint reader.                                                                                                               |
| `with-silent-lastlog`            | Disable generation of `pam_lostlog` messages during login                                                                                                              |
| `with-sudo`                      | Enable `sudo` to use SSSD for rules besides `/etc/sudoers`.                                                                                            |
| `with-pamaccess`                 | Refer to `/etc/access.conf` for account authorization.                                                                                                 |
| `without-nullock`                | Do not add the `nullock` parameter to `pam_unix`                                                                                                                       |

| Feature Name          | Description                                                                    |
| --------------------- | ------------------------------------------------------------------------------ |
| `with-faillock`       | Lock the account after too many authentication failures.       |
| `with-mkhomedir`      | Create home directory on user's first log in.                  |
| `with-ecryptfs`       | Enable automatic per-user ecryptfs.                            |
| `with-fingerprint`    | Authenticate through fingerprint reader.                       |
| `with-krb5`           | Use Kerberos authentication.                                   |
| `with-silent-lastlog` | Disable generation of pam\_lostlog messages during login |
| `with-pamaccess`      | Refer to `/etc/access.conf` for account authorization.         |
| `without-nullock`     | Do not add the `nullock` parameter to `pam_unix`                               |

For details about each profile, refer to the profile's corresponding `/usr/share/authselect/default/*profile*/README` file. See also the `authselect-profiles(5)` manual page.

## About the authselect Utility

The `authselect` utility is the Enterprise Linux tool for configuring authentication on the system. The tool manages system authentication profiles and is automatically included in any Enterprise Linux 8 installation.

The `authselect` utility consists of the following components:

- `authselect` command to manage system authentication. Only users with the appropriate administrator privileges can run this command.

- Profiles that apply specific authentication mechanisms. These profiles can be those supplied by OpenELA, provided by vendors, or created by an organization.

To efficiently manage the variety of profiles, `authselect` stores different types of profiles in corresponding files:

- `/usr/share/authselect/default` contains the OpenELA-supplied profiles provided by Enterprise Linux.

- `/usr/share/authselect/vendor` contains the profiles that are provided by vendors. These profiles can override those that are in the default directory.

- `/etc/authselect/custom` contains any profiles you create for the specific environment.

**Important:**

The `authselect` utility applies the specifications in the selected profile. However, `authselect` doesn't change the configuration files of the service on which the profile is based. If, for example, you use the `sssd` profile, you must configure SSSD for the service to function properly. Consult the proper documentation to configure the profile's service. You must also ensure that the service is started and enabled.

For more details about the utility, see the `authselect(8)` manual page.

