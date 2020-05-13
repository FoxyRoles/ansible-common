# ansible-common

Common task to setup Ubuntu machine after instalation

This role is highly opinionated. You should fork it and make your own adjustments.

## Example playboot

```yaml
---
- hosts: myserver
  user: root
  sudo: False
  vars:
    admin_email: my@mail.tld
    allow_global_ip: 192.168.1.1
    ns1: 1.1.1.1 # optional
    ns2: 1.0.0.1 # optional
    postfix_myhostname: domain.tld
    root_default_password: encryptedRootPassword
  roles:
   - sunfoxcz.common
```

## License

Licensed under MIT license. See [LICENSE](LICENSE.md) for details.
