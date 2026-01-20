# cesnet.openldap

Ansible role for installing OpenLDAP server on Debian. It supports:
* installing a single server
* installing a **master** replica server
* installing a **slave** replica server
* activating **memberOf overlay** for consistent memberOf attribute of users
* setting **strong password hashing** algorithm ARGON2 as the default hashing method

## Variables

* **ldap_domain** - DNS domain used for generating the base DN
* **ldap_top_organization** - the value of `o` attribute of the base path
* **ldap_data_password** - password for administrator account of the data tree
* **ldap_config_password** - password for administrator account of the config tree
* **ldap_hold_package** - whether to hold slapd package against upgrades (default no)
* **ldap_certificate_file** - path to TLS certificate
* **ldap_certificate_key_file** - path to TLS private key
* **ldap_certificate_chain_file** - path to TLS certificate chain
* **ldap_access_rules_set** - whether to set ACL, default is yes, may be set to "no" if ACL contains attributes not yet defined
* **ldap_access_rules_additional** - access rules to be added to the default rules (default empty list)
* **ldap_size_limit** - limit for the number of returned records, default is unlimited
* **ldap_master_replica** - whether to configure the server as master replica (default no)
* **ldap_replication_password** - password for replication user
* **ldap_slave_replica** - whether to configure the server as slave replica (default no)
* **ldap_master_url** - URL ot the master replica that the slave should connect to
* **ldap_accesslog_db** - Name for accesslog LDAP DB used for 
  delta-replication, by default it's `{2}mdb`, but if you already have multiple 
  DBs in LDAP you might need to change this to prevent your DBs from being 
  overwritten when enabling replication. Only `{0}config` and `{1}mdb` 
  DBs are replicated (schema + data of primary DB).
* **ldap_users** - list of users to create, keys user, password and description are required for each one
* **ldap_memberOf_overlay** - whether to configure memberOf overlay for adding the attribute memberOf to group members and refint overlay for keeping consistency (default no)
* **ldap_sssvlv_overlay** - whether to add Server Side Sorting and Virtual List View overlay (default no)
* **ldap_allow_empty_groups** - whether to modify core schema to allow empty groups (default no)
* **ldap_strong_password_hashing** - whether to configure strong password hashing as the default hashing method (default no)
* **ldap_pass_through_authentication** - whether to configure pass-through authentication using Kerberos
* **ldap_log_level** - list of log levels to use, eg. "stats" or multi-value 
  like "acl trace".

For midPoint, set ldap_memberOf_overlay, ldap_sssvlv_overlay and ldap_allow_empty_groups to yes.

## Examples

Example of installing a master server for replication:
```yaml
- hosts: cloud6.perun-aai.org
  remote_user: root
  tasks:
    - name: "create ldap master"
      import_role:
        name: cesnet.openldap
      vars:
        ldap_domain: "cesnet.cz"
        ldap_top_organization: "perun"
        ldap_data_password: "test1"
        ldap_config_password: "test2"
        ldap_certificate_file: "/etc/letsencrypt/live/cloud6.perun-aai.org/cert.pem"
        ldap_certificate_key_file: "/etc/letsencrypt/live/cloud6.perun-aai.org/privkey.pem"
        ldap_certificate_chain_file: "/etc/letsencrypt/live/cloud6.perun-aai.org/chain.pem"
        ldap_master_replica: yes
        ldap_accesslog_db: "{2}mdb"
        ldap_replication_password: "test"
        ldap_memberOf_overlay: yes
        ldap_sssvlv_overlay: yes
        ldap_allow_empty_groups: yes
        ldap_users:
          - user: proxy
            password: test
            description: "user for IdP Proxy"
        ldap_access_rules_additional:
          - >-
            to dn.subtree="dc=cesnet,dc=cz"
            by dn.exact="cn=proxy,dc=cesnet,dc=cz" read
            by * break
```

Example of installing a slave replica:
```yaml
- hosts: cloud4.perun-aai.org
  remote_user: root
  tasks:
    - name: "create ldap slave replica"
      import_role:
        name: cesnet.openldap
      vars:
        ldap_domain: "cesnet.cz"
        ldap_top_organization: "perun"
        ldap_data_password: "test1"
        ldap_config_password: "test2"
        ldap_certificate_file: "/etc/letsencrypt/live/cloud4.perun-aai.org/cert.pem"
        ldap_certificate_key_file: "/etc/letsencrypt/live/cloud4.perun-aai.org/privkey.pem"
        ldap_certificate_chain_file: "/etc/letsencrypt/live/cloud4.perun-aai.org/chain.pem"
        ldap_slave_replica: yes
        ldap_replication_password: "test"
        ldap_master_url: 'ldaps://cloud6.perun-aai.org/'
        ldap_memberOf_overlay: yes

```
