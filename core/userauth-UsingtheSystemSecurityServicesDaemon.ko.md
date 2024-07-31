<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Using the System Security Services Daemon

The System Security Services Daemon \(SSSD\) feature provides access on a client system to remote identity and authentication providers. The SSSD acts as an intermediary between local clients and any back-end provider that you configure.

The benefits of configuring SSSD include the following:

- Reduced system load

  Clients do not have to contact the identification or authentication servers directly.

- Offline authentication

  You can configure SSSD to maintain a cache of user identities and credentials.

- Single sign-on access

  If you configure SSSD to store network credentials, users need only authenticate once per session with the local system to access network resources.

Because the Enterprise Linux `sssd` profile is used by default, the SSSD service is also automatically installed and enabled on a newly installed system. The default configuration uses the Pluggable Authentication Modules \(PAM\) and the Name Service Switch \(NSS\) for managing access and authentication on a system. No further configuration is required, unless you wish to use different authentication services or wish to customize the configuration to use alternative values to the default settings.

See [https://sssd.io/](https://sssd.io/) for more information about SSSD.

## Customizing SSSD

By default the SSSD service used by the `sssd` profile uses Pluggable Authentication Modules \(PAM\) and the Name Service Switch \(NSS\) for managing access and authentication on a system. As you enable additional features for the profile to customize SSSD authentication, you must also configure SSSD for the enabled feature.

You customize an SSSD configuration by creating configuration files within the `/etc/sssd/conf.d` directory. Each configuration file must have the `.conf` suffix to enable it when SSSD is started. Configuration files use ini-style syntax as format. The content consist of sections, which are identified by square brackets, and parameters, which are listed as `key = value` entries. The manual pages provided for SSSD are comprehensive and provide detailed information on the options that are available.

The following example shows how you might configure SSSD to authenticate against an LDAP provider with Kerberos configured:

1. Create a configuration file for the feature and store it in `/etc/sssd/conf.d`, for example `/etc/sssd/conf.d/00-ldap.conf`

2. Configure `/etc/sssd/conf.d/00-ldap.conf` with parameter definitions, such as the following:

   ```
   [sssd]
   config_file_version = 2
   domains = LDAP
   services = nss, pam

   [domain/LDAP]
   id_provider = ldap
   ldap_uri = ldap://ldap.mydom.com
   ldap_search_base = dc=mydom,dc=com

   auth_provider = krb5
   krb5_server = krbsvr.mydom.com
   krb5_realm = MYDOM.COM
   cache_credentials = true

   min_id = 5000
   max_id = 25000
   enumerate = false

   [nss]
   filter_groups = root
   filter_users = root
   reconnection_retries = 3
   entry_cache_timeout = 300

   [pam]
   reconnection_retries = 3
   offline_credentials_expiration = 2
   offline_failed_login_attempts = 3
   offline_failed_login_delay = 5
   ```

   - **`[sssd]`**

     Contains configuration settings for SSSD monitor options, domains, and services. The SSSD monitor service manages the services that SSSD provides.

     - `services` defines the supported services, which should include `nss` for the Name Service Switch and `pam` for Pluggable Authentication Modules.

     - The `domains` entry specifies the name of the sections that define authentication domains.

   - **`[domain/LDAP]`**

     Defines a domain for an LDAP identity provider that uses Kerberos authentication. Each domain defines where user information is stored, the authentication method, and any configuration options. SSSD can work with LDAP identity providers such as OpenLDAP, Red Hat Directory Server, IPA, and Microsoft Active Directory, and it can use either native LDAP or Kerberos authentication.

     - `id_provider` specifies the type of provider \(in this example, LDAP\).

     - `ldap_uri` specifies a comma-separated list of the Universal Resource Identifiers \(URIs\) of the LDAP servers, in order of preference, to which SSSD can connect.

     - `ldap_search_base` specifies the base distinguished name \(`dn`\) that SSSD should use when performing LDAP user operations on a relative distinguished name \(RDN\) such as a common name \(`cn`\).

     - `auth_provider` entry specifies the authentication provider \(in this example, Kerberos\).

     - `krb5_server` specifies a comma-separated list of Kerberos servers, in order of preference, to which SSSD can connect.

     - `krb5_realm` specifies the Kerberos realm.

     - `cache_credentials` specifies if SSSD caches user credentials such as tickets, session keys, and other identifying information to support offline authentication and single sign-on.

       **Note:**

       To allow SSSD to use Kerberos authentication with an LDAP server, you must configure the LDAP server to use both Simple Authentication and Security Layer \(SASL\) and the Generic Security Services API \(GSSAPI\). For more information about configuring SASL and GSSAPI for OpenLDAP, see [https://www.openldap.org/doc/admin24/sasl.html](https://www.openldap.org/doc/admin24/sasl.html).

     - `min_id` and `max_id` specify upper and lower limits on the values of user and group IDs.

     - `enumerate` specifies whether SSSD caches the complete list of users and groups that are available on the provider. The recommended setting is `False` unless a domain contains relatively few users or groups.

   - **`[nss]`**

     Configures the Name Service Switch \(NSS\) module that integrates the SSS database with NSS.

     - `filter_users` and `filter_groups` prevent NSS from extracting information about the specified users and groups being retrieved from SSS.

     - `reconnection_retries` specifies the number of times that SSSD should try to reconnect if a data provider crashes.

     - `enum_cache_timeout` specifies the number of seconds for which SSSD caches user information requests.

   - **`[pam]`**

     Configures the PAM module that integrates SSSD with PAM.

     - `offline_credentials_expiration` specifies the number of days for which to allow cached logins if the authentication provider is offline.

     - `offline_failed_login_attempts` specifies how many failed login attempts are allowed if the authentication provider is offline.

     - `offline_failed_login_delay` specifies how many minutes after the limit of allowed failed login attempts have been exceeded before a new login attempt is permitted.

3. Change the mode of `/etc/sssd/conf.d/00-ldap.conf` to 0600:

   ```
   sudo chmod 0600 /etc/sssd/conf.d/00-ldap.conf
   ```

4. If it's not started yet, enable the SSSD service:

5. Select the `sssd` profile.

   ```
   sudo authselect select sssd
   ```

For more information about the SSSD service, see the `sssd(8)` manual page and [https://pagure.io/SSSD/sssd/](https://pagure.io/SSSD/sssd/). Also, you can consult `sssd.conf(5)`, `sssd-ldap(5)`, `sssd-krb5(5)`, `sssd-ipa(5)`, and other manual pages.

## About Pluggable Authentication Modules

The Pluggable Authentication Modules \(PAM\) feature is an authentication mechanism used by the `sssd` profile that allows you to configure how applications use authentication to verify the identity of a user. The PAM configuration files, which are located in the `/etc/pam.d` directory, describe the authentication procedure for an application. The name of each configuration file is the same as, or is similar to, the name of the application for which the module provides authentication. For example, the configuration files for `passwd` and `sudo` are named `passwd` and `sudo`.

Each PAM configuration file contains a list or _stack_ of calls to authentication modules. For example, the following listing shows the default content of the `login` configuration file:

```
#%PAM-1.0
auth [user_unknown=ignore success=ok ignore=ignore default=bad] pam_securetty.so
auth       include      system-auth
auth       include      postlogin
account    required     pam_nologin.so
account    include      system-auth
password   include      system-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
session    optional     pam_console.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    include      system-auth
session    include      postlogin
-session   optional     pam_ck_connector.so
```

Comments in the file start with a `#` character. The remaining lines each define an operation type, a control flag, the name of a module such as `pam_rootok.so` or the name of an included configuration file such as `system-auth`, and any arguments to the module. PAM provides authentication modules as shared libraries in `/usr/lib64/security`.

For a particular operation type, PAM reads the stack from top to bottom and calls the modules listed in the configuration file. Each module generates a success or failure result when called.

The following operation types are defined for use:

- **`auth`**

  The module tests whether a user is authenticated or authorized to use a service or application. For example, the module might request and verify a password. Such modules can also set credentials, such as a group membership or a Kerberos ticket.

- **`account`**

  The module tests whether an authenticated user is allowed access to a service or application. For example, the module might check if a user account has expired or if a user is allowed to use a service at a given time.

- **`password`**

  The module handles updates to an authentication token.

- **`session`**

  The module configures and manages user sessions, performing tasks such as mounting or unmounting a user's home directory.

If the operation type is preceded with a dash \(`-`\), PAM does not add an create a system log entry if the module is missing.

Except for `include`, the control flags tell PAM what to do with the result of running a module. The following control flags are defined for use:

- **`optional`**

  The module is required for authentication if it's the only module listed for a service.

- **`required`**

  The module must succeed for access to be granted. PAM continues to mprocess the remaining modules in the stack whether the module succeeds or fails. PAM doesn't immediately inform the user of the failure.

- **`requisite`**

  The module must succeed for access to be granted. If the module succeeds, PAM continues to process the remaining modules in the stack. However, if the module fails, PAM notifies the user immediately and doesn't continue to process the remaining modules in the stack.

- **`sufficient`**

  If the module succeeds, PAM doesn't process any remaining modules of the same operation type. If the module fails, PAM processes the remaining modules of the same operation type to determine overall success or failure.

The control flag field can also define one or more rules that specify the action that PAM takes depending on the value that a module returns. Each rule takes the form `*value*=*action*`, and the rules are enclosed in square brackets, for example:

```
[user_unknown=ignore success=ok ignore=ignore default=bad]
```

If the result that's returned by a module matches a value, PAM uses the corresponding action, or, if there is no match, it uses the default action.

The `include` flag specifies that PAM must also consult the PAM configuration file specified as the argument.

Most authentication modules and PAM configuration files have their own manual pages. Relevant files are stored in the `/usr/share/doc/pam` directory.

For more information, see the `pam(8)` manual page. In addition, each PAM module has its own manual page, for example `pam_unix(8)`, `postlogin(5)`, and `system-auth(5)`.
