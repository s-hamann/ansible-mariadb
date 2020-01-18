MariaDB
=======

This role configures a MariaDB database server.

Requirements
------------

If TLS encryption is desired, the target system needs to have a suitable X.509 certificate.
Likewise, when client certificates are required of the LDAP clients, the issuing CA's certificate needs to be present on the target system.
This roles does not handle deploying certificates.

Role Variables
--------------

* `mariadb_port`  
  The TCP port to listen on.
  When set to `false`, networking is disabled completely and MariaDB is reachable only via it's UNIX socket.
  Defaults to `3306`.
* `mariadb_bind_address`  
  The IP address to bind to.
  Set to `0.0.0.0` to listen on all IP addresses.
  Defaults to `127.0.0.1`.
* `mariadb_databases`  
  A list of databases to set up.
  Each list item needs to be a dictionary with the following keys:
  * `name`  
    The name of the database.
    Mandatory.
  * `encoding`  
    Character encoding to use in the database.
    Defaults to `utf8mb4`.
  * `collation`  
    Collation to use in the database.
    Defaults to `utf8mb4_unicode_520_ci`.
* `mariadb_users`
  A list of user accounts to set up within MariaDB.
  Each list item needs to be a dictionary with the following keys:
  * `name`  
    The user's name.
    Mandatory.
  * `host`  
    The host from which the user may log in.
    Defaults to `localhost`.
  * `password`  
    The user's password.
    Optional.
  * `plugin`  
    The authentication plugin for the user.
    Defaults to `mysql_native_password` if networking is enabled and to `unix_socket` if it is not (cf. `mariadb_port`).
  * `grants`  
    A list of privileges to grant to the user account.
    Each list item is a string of the form `db.table:priv1,priv2,...`.
    Privileges not in this list are removed from the account, if any.
    Optional.
* `mariadb_error_log`  
  Path to a file where errors, warning and other important status information is logged.
  Defaults to `/var/log/mysql/error.log`.
* `mariadb_slow_query_log`  
  Path to a file where slow queries are logged.
  Defaults to `false`, which disables this feature.
* `mariadb_slow_query_time`  
  Queries that take this many seconds are considered slow for the purpose of logging.
  Defaults to `10.0`.
* `mariadb_tls_cert`  
  Path to a PEM-encoded X.509 certificate for MariaDB to use.
  The file needs to exist and be readable by the MariaDB user.
  Default is unset, which disables TLS support.
* `mariadb_tls_cert_key`  
  Path to the PEM-encoded private key file for the certificate.
  The file needs to exist and be readable by the MariaDB user.
  Default is unset.
* `mariadb_tls_client_ca`  
  Path to a PEM-encoded list of X.509 CA certificates that can sign client certificates.
  The file needs to exist and be readable by the MariaDB user.
  Default is unset.
* `mariadb_tls_13_only`  
  Set to `true` to enforce TLSv1.3 only.
  If set to `false` (the default), TLSv1.2 is enforced as the minimal supported protocol version.
  This setting only has an effect on MariaDB-10.4.6 or newer.
* `mariadb_extra_options`  
  A dictionary of additional server options to set.
  Dictionary keys are option names and values their respective values.
  Refer to the [MariaDB documentation](https://mariadb.com/kb/en/server-system-variables/) for options and their meaning.
  Optional.
* `mariadb_extra_groups`  
  A list of groups that the MariaDB system user is added to.
  This allows granting access to additional resources, such as the private key file.
  All groups need to exist on the target system; this role does not create them.
  Empty by default.
* `mariadb_inaccessible_paths`  
  If the target system uses systemd, this option takes a list of paths, that should not be accessible at all for MariaDB.
  Optional.
* `mariadb_use`  
  A list of USE flags to set (or unset) on `dev-db/mariadb`.
  Empty by default.
  Only used on Gentoo.

Dependencies
------------

This role does not set up TLS certificates and therefore depends on a role that generates and deploys them, if TLS support is desired.

Example Configuration
---------------------

The following is short example for some of the configuration options this role provides:

```yaml
mariadb_port: false
mariadb_databases:
  - name: exampledb1
    encoding: latin1
    collation: latin1_swedish_ci
  - name: exampledb2
mariadb_users:
  - name: exampleuser1
    host: app.host.tld
    password: example
    plugin: mysql_native_password
    grants:
      - '*.*:USAGE'
      - 'exampledb1.*:USAGE,SELECT,INSERT,UPDATE'
  - name: systemuser
    plugin: unix_socket
    grants:
      - '*.*:USAGE'
      - 'exampledb1.*:ALL'
      - 'exampledb2.exampletable:SELECT'
```

License
-------

MIT
