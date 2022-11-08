# ansible-common

Common task to setup Ubuntu machine after instalation

This role is highly opinionated. You should fork it and make your own adjustments.

## Example playbook

```yaml
---
- hosts: myserver
  user: root
  sudo: False
  vars:
    admin_email: my@mail.tld
    allow_global_ip: 192.168.1.1
  roles:
   - sunfoxcz.common
```

## Mandatory variables

 * admin_email
 * allow_global_ip

## Optional variables

 * postfix_myhostname
 * root_default_password

## Default variables

```yaml
apt_cache_valid_time: 86400

dns_servers:
  - 1.1.1.1
  - 1.0.0.1

ntp_servers:
  - 0.cz.pool.ntp.org
  - 1.cz.pool.ntp.org
  - 2.cz.pool.ntp.org
  - 3.cz.pool.ntp.org

systemd_journal_max_size: 100M

```

## License

Licensed under MIT license. See [LICENSE](LICENSE.md) for details.
