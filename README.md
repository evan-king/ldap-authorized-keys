# ldap-authorized-keys

Perform LDAP lookup for authorized keys by user.

Intended as a minimal-footprint solution to implementing OpenSSH-LPK as part of an overall LDAP-backed account management strategy.

# Usage

List SSH public keys stored in LDAP for the specified user:

    ldap-authorized-keys username

Relies on working LDAP integration for user accounts (i.e. `getent passwd username` should work on LDAP-only accounts).

# Configuration

The following configuration options are used from `/etc/ldap.conf`.

### Search options

* **uri** (default `ldap://localhost`)  
    URI of the LDAP server to connect. The URI scheme may be ldap, or ldaps.
               
* **base**  
    Distinguished name (DN) of the search base.
    
* **scope** (default _sub_)  
    Search scope: _sub_, _one_, or _base_.

* **pam_login_attribute** (default is _uid_)  
     The attribute to match as user id.

* **pam_filter** (default _objectClass=PosixAccount_)  
    Filter to use when searching for the user’s entry.  
    Additional to the match on _pam_login_attribute_.

### SSL options

These options specifically support ssl operation via `start_tls`:

* **ssl**  
    Encryption configuration.  Valid values are _off_, _on_, and _start_tls_.  
    Currently only _start_tls_ is recognized and supported.  Not needed with an `ldaps://` URI.

* **tls_checkpeer**  
    Whether to require server certificate validation (recommended).  Valid values are _yes_ and _no_.

* **tls_cacertfile**  
    Path to the ca file validating the server.

* **tls_cacertpath**  
    Path to a directory containing all ca files that can validate the server.  
    It is not necessary to set both `tls_cacertfile` and `tls_cacertpath`.

# Setup LDAP server

Add [openssh-lpk.schema](openssh-lpk.schema) to your LDAP server's config.

# Setup OpenSSH server

To configure OpenSSH server to fetch users’ authorized keys from LDAP server:

1. Install perl module dependencies:
   - With apt (Ubuntu): `apt install libconfig-general-perl` (`libnet-ldap-perl` should already be installed)
   - with cpan (Any): `cpanm -S Config::General Net::LDAP`
2. Copy the script ldap-authorized-keys to `/usr/local/bin` (ensure owner `root` and mode `0755`)
3. Add these two lines to /etc/ssh/sshd_config:

        AuthorizedKeysCommand /usr/local/bin/ldap-authorized-keys
        AuthorizedKeysCommandUser nobody

4. Restart sshd.

# Similar Projects

 - [ssh-ldap-pubkey](https://github.com/jirutka/ssh-ldap-pubkey): More robust system for managing ssh keys in ldap, but using Python.
 - [ssh-getkey-ldap](https://github.com/jirutka/ssh-getkey-ldap): Similar lightweight Lua-based script, but with its own independent config file.
 - [ldapsearch wrapper](https://gist.github.com/jirutka/b15c31b2739a4f3eab63): Simple bash script using ldapsearch, but with no configuration.

# License

This project is licensed under [MIT license](http://opensource.org/licenses/MIT).
