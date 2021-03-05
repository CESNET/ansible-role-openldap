# cesnet.openldap

Ansible role for installing OpenLDAP server on Debian.

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
        ldap_users:
          - user: proxy
            password: test
            description: "user for IdP Proxy"
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
        ldap_slave_tls_cacert: '/etc/ldap/masterca.pem'
        ldap_master_url: 'ldaps://cloud6.perun-aai.org/'

```