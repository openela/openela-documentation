<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Working With System Authentication Profiles

The `authselect` command has various subcommands, arguments, and options to create, delete, switch to a different profile, and modify profile features. A user must have the appropriate privileges to be able to use this configuration tool.

## Displaying Profile Information

To determine which profile is currently active in a system, type:

```
sudo authselect current
```

```
Profile ID: sssd
Enabled features:
- with-fingerprint
- with-silent-lastlog
```

The output of the command indicates that the `ssd` profile is currently active. At a minimum, authentication with the fingerprint reader is enforced through `pam_fprintd`. Additionally, no `pam_lastlog` message is displayed on the screen when users log in.

## Configuring Profile Features

Enabled features of a profile determine the manner of authentication on the system. You can enable profile features in one of two ways:

- Specify additional features to be enabled in the current profile.

- Replace current features of a selected profile. This method is discussed in [Selecting the Winbind Profile](userauth-WorkingWithSystemAuthenticationProfiles.md#).

### Enabling Profile Features

1. \(Optional\): Identify the current profile.

   Enabling additional features works only on the current profile. The procedure does not work on unselected profiles.

   ```
   sudo authselect current
   ```

2. If necessary, identify the feature requirements for the feature to work properly.

   ```
   sudo authselect requirements *profile* *feature*
   ```

3. Complete the indicated listed feature requirements as needed.

4. Enable the feature.

   ```
   sudo authselect enable-feature *feature*
   ```

   Note that you can only enable features one at a time.

### Disabling Profile Features

Use the `disable-feature` subcommand.

```
sudo authselect disable-feature *feature*                  
```

### Adding Functionalities to a Profile

The following example shows how you can set account locking and define home directories as additional features of the default `sssd` profile.

1. Determine the requirements to automatically lock an account after too many authentication failures \(`with-faillock`\):

   ```
   sudo authselect requirements sssd with-faillock
   ```

   ```
   ﻿Make sure that SSSD service is configured and enabled. See SSSD documentation for more 
   information.
   ```

2. Determine the requirements to automatically create a user home directory at the user's first time log in \(`with-mkhomedir`\).

   ```
   sudo authselect requirements sssd with-mkhomedir
   ```

   ```
   ﻿Make sure that SSSD service is configured and enabled. See SSSD documentation for more 
   information.
    
   - with-mkhomedir is selected, make sure pam_oddjob_mkhomedir module
     is present and oddjobd service is enabled
     - systemctl enable oddjobd.service
     - systemctl start oddjobd.service
   ```

3. Fulfill the requirements of the features you want to enable.

4. Enable both profile features:

   ```
   sudo authselect enable-feature with-faillock
   ```

   ```
   sudo authselect enable-feature with-mkhomedir
   ```

5. Confirm that both profile features have been enabled:

   ```
   sudo authselect current
   ```

   ```
   Profile ID: sssd
   Enabled features:
   - with-fingerprint
   - with-silent-lastlog
   - with-faillock
   - with-mkhomedir
   ```

### Enabling the PAM Access Feature

The following example shows how you can direct the system to check `/etc/security/access.conf` to authenticate and authorize users. In this case, the PAM access feature needs to be added as an enabled feature for `sssd`.

1. Automatically enable PAM access:

   ```
   sudo authselect requirements sssd with-pamaccess
   ```

   ```
   ﻿Make sure that SSSD service is configured and enabled. See SSSD documentation for more 
   information.
   ```

2. Enable the PAM access profile feature:

   ```
   sudo authselect enable-feature sssd with-pamaccess
   ```

3. Confirm that the PAM access profile feature has been enabled:

   ```
   sudo authselect current
   ```

   ```
   Profile ID: sssd
   Enabled features:
   - with-fingerprint
   - with-silent-lastlog
   - with-faillock
   - with-mkhomedir
   - with-pamaccess
   ```

**Note:**

The prevous example assumes that you have configured `/etc/security/access.conf` so that the feature functions correctly. For more information, see the `access.conf(5)` manual page.

## Selecting the Winbind Profile

Winbind is a client-side service that resolves user and group information on a Windows server. Use this profile to enable Enterprise Linux to work with Windows users and groups.

1. Install the `samba-winbind` package.

   ```
   sudo dnf install samba-winbind -y
   ```

2. Select the `winbind` profile.

   When selecting a profile, you can enable multiple features in the same command, for example:

   ```
   sudo authselect select winbind with-faillock with-mkhomedir [*options*]
   ```

   ```
   Profile "winbind" was selected.
   The following nsswitch maps are overwritten by the profile:
   - passwd
   - group

   Make sure that winbind service is configured and enabled. See winbind documentation for 
   more information.
    
   - with-mkhomedir is selected, make sure pam_oddjob_mkhomedir module
     is present and oddjobd service is enabled
     - systemctl enable oddjobd.service
     - systemctl start oddjobd.service
   ```

   For other options you can use with the `authselect select` command, see the `authselect(8)` manual page.

3. Fulfill the requirements of the features you enabled for the profile.

4. Start the `winbind` service.

   ```
   sudo systemctl start winbind
   ```

   ```
   sudo systemctl enable winbind
   ```

**Note:**

If you modify features of an already current and active profile, the revised features will replace whatever features were previously enabled.

## Modifying Ready-Made Profiles

Profiles also use information stored in the `/etc/nsswitch.conf` file to enforce authentication. However, to modify and customize a ready-made profile, specify its configuration properties in the `/etc/user-nsswitch.conf` file. Do not edit the `/etc/nsswitch.conf` directly.

1. If necessary, select the profile to make it current, for example:

   ```
   sudo authselect select sssd
   ```

2. Edit the `/etc/authselect/user-nsswitch.conf` file as required.

   **Note:**

   Do not modify the any of following configurations in the file. If you do, those modifications will be ignored:

   - `passwd`

   - `group`

   - `netgroup`

   - `automount`

   - `services`

3. Apply the changes.

   ```
   sudo authselect apply-changes
   ```

   The changes in `/etc/authselect/user-nsswitch.conf` are applied to `/etc/nsswitch.conf` and will be used by the current profile.

**Important:**

If the system is part of an environment that uses either Identity Management or Active Directory, do not use `authselect` to manage authentication. When the host is made to join either Identity Management or Active Directory, their respective tools take care of managing authentication of the environment.

## Creating Custom Profiles

If you do not want to use the profiles included in Enterprise Linux or those provided by vendors, you can create your own specific profile.

1. Create the profile.

   ```
   sudo authselect create-profile *newprofile* -b *template* --symlink-meta --symlink-pam
   ```

   - **_newprofile_**

     Name of your custom profile.

   - **_template_**

     Base to be used for the custom profile, which is either `sssd` or `winbind`.

   - **--symlink-meta**

     Creates symbolic links to the meta files in the original directory of the template profile you are using as base.

   - **--symlink-pam**

     Creates symbolic links to the PAM templates in the original directory of the template profile you are using as base.

   This command creates an `/etc/authselect/custom/*newprofile*` directory that contains the symbolic links to the files in the base's original directory. The only file that is **not** a symbolic link in this directory is `nsswitch.conf`.

2. Edit the `/etc/authselect/custom/*newprofile*/nsswitch.conf` file according to your preference.

3. Select your custom profile.

   ```
   sudo authselect select custom/*newprofile*                        
   ```

   This command also creates a backup of the original `/etc/nsswitch.conf` file and replaces it with a symbolic link to the corresponding file in your custom profile's directory.

   You can test this result by comparing the symbolic link `/etc/nsswitch.conf` with the original `/etc/nsswitch.conf.bak` and verify that the original file's contents remain intact.

4. Enable features for your new profile as needed.

   See [Configuring Profile Features](userauth-WorkingWithSystemAuthenticationProfiles.md#) for reference.

5. \(Optional\) Verify the configuration of the custom profile.

   ```
   sudo authselect current
   ```
