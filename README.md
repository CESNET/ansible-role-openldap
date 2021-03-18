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
* **ldap_access_rules_additional** - access rules to be added to the default rules (default empty list)
* **ldap_size_limit** - limit for the number of returned records, default is unlimited
* **ldap_master_replica** - whether to configure the server as master replica (default no)
* **ldap_replication_password** - password for replication user
* **ldap_slave_replica** - whether to configure the server as slave replica (default no)
* **ldap_master_url** - URL ot the master replica that the slave should connect to
* **ldap_users** - list of users to create, keys user, password and description are required for each one
* **ldap_memberOf_overlay** - whether to configure memberOf overlay for adding the attribute memberOf to group members and refint overlay for keeping consistency (default no)
* **ldap_sssvlv_overlay** - whether to add Server Side Sorting and Virtual List View overlay (default no)
* **ldap_allow_empty_groups** - whether to modify core schema to allow empty groups (default no)
* **ldap_strong_password_hashing** - whether to configure strong password hashing as the default hashing method (default no)

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
    - name: "create masterca.pem"
      copy:
        dest: /etc/ldap/masterca.pem
        content: |
          -----BEGIN CERTIFICATE-----
          MIIEZTCCA02gAwIBAgIQQAF1BIMUpMghjISpDBbN3zANBgkqhkiG9w0BAQsFADA/
          MSQwIgYDVQQKExtEaWdpdGFsIFNpZ25hdHVyZSBUcnVzdCBDby4xFzAVBgNVBAMT
          DkRTVCBSb290IENBIFgzMB4XDTIwMTAwNzE5MjE0MFoXDTIxMDkyOTE5MjE0MFow
          MjELMAkGA1UEBhMCVVMxFjAUBgNVBAoTDUxldCdzIEVuY3J5cHQxCzAJBgNVBAMT
          AlIzMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAuwIVKMz2oJTTDxLs
          jVWSw/iC8ZmmekKIp10mqrUrucVMsa+Oa/l1yKPXD0eUFFU1V4yeqKI5GfWCPEKp
          Tm71O8Mu243AsFzzWTjn7c9p8FoLG77AlCQlh/o3cbMT5xys4Zvv2+Q7RVJFlqnB
          U840yFLuta7tj95gcOKlVKu2bQ6XpUA0ayvTvGbrZjR8+muLj1cpmfgwF126cm/7
          gcWt0oZYPRfH5wm78Sv3htzB2nFd1EbjzK0lwYi8YGd1ZrPxGPeiXOZT/zqItkel
          /xMY6pgJdz+dU/nPAeX1pnAXFK9jpP+Zs5Od3FOnBv5IhR2haa4ldbsTzFID9e1R
          oYvbFQIDAQABo4IBaDCCAWQwEgYDVR0TAQH/BAgwBgEB/wIBADAOBgNVHQ8BAf8E
          BAMCAYYwSwYIKwYBBQUHAQEEPzA9MDsGCCsGAQUFBzAChi9odHRwOi8vYXBwcy5p
          ZGVudHJ1c3QuY29tL3Jvb3RzL2RzdHJvb3RjYXgzLnA3YzAfBgNVHSMEGDAWgBTE
          p7Gkeyxx+tvhS5B1/8QVYIWJEDBUBgNVHSAETTBLMAgGBmeBDAECATA/BgsrBgEE
          AYLfEwEBATAwMC4GCCsGAQUFBwIBFiJodHRwOi8vY3BzLnJvb3QteDEubGV0c2Vu
          Y3J5cHQub3JnMDwGA1UdHwQ1MDMwMaAvoC2GK2h0dHA6Ly9jcmwuaWRlbnRydXN0
          LmNvbS9EU1RST09UQ0FYM0NSTC5jcmwwHQYDVR0OBBYEFBQusxe3WFbLrlAJQOYf
          r52LFMLGMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjANBgkqhkiG9w0B
          AQsFAAOCAQEA2UzgyfWEiDcx27sT4rP8i2tiEmxYt0l+PAK3qB8oYevO4C5z70kH
          ejWEHx2taPDY/laBL21/WKZuNTYQHHPD5b1tXgHXbnL7KqC401dk5VvCadTQsvd8
          S8MXjohyc9z9/G2948kLjmE6Flh9dDYrVYA9x2O+hEPGOaEOa1eePynBgPayvUfL
          qjBstzLhWVQLGAkXXmNs+5ZnPBxzDJOLxhF2JIbeQAcH5H0tZrUlo5ZYyOqA7s9p
          O5b85o3AM/OJ+CktFBQtfvBhcJVd9wvlwPsk+uyOy2HI7mNxKKgsBTt375teA2Tw
          UdHkhVNcsAKX1H7GNNLOEADksd86wuoXvg==
          -----END CERTIFICATE-----
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