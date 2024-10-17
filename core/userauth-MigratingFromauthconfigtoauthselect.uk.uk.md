<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Migrating From authconfig to authselect

Beginning with Enterprise Linux 8, `authselect` has replaced `authconfig` that was used in prior releases. Compatibility between the two utilities is minimal. Thus, migrating to `authselect` is highly recommended. Migrating requires you to complete several actions, including the following:

- Convert scripts.

  If you use the `ipa-client-install` command or the `realm join` command to make the host join a domain, you can remove any `authconfig` call in any scripts. Otherwise, you need to replace each `authconfig` call with its matching `authselect` call.

- Update configuration files.

  You must configure files for the various services, including those that apply to the following: Kerberos, LDAP, NIS, SSSD, and Winbind.

- Enforce password quality restrictions for `authselect`.

  The `pam_pwquality` module enforces password quality restrictions for local users. You configure this module in the `/etc/security/pwquality.conf` file, according to the information that's provided in the `pam_pwquality(8)` manual page.

- Switch from the `authconfig`'s `cacertdir_rehash` tool to the native `openssl rehash` _directory_ command.

- Start the appropriate services.

  Depending on the profile you select for the `authselect` implementation, start the service for that profile. If you select the `sssd` profile, for example, then you would enable and start the SSSD service.

  ```
  sudo systemctl enable --now sssd
  ```

For complete migration instructions and examples, see the `authselect-migration(7)` manual page. See also the `authselect(8)` manual page.
