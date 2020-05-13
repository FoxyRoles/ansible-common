# ansible-common
Common task to setup Ubuntu machine after instalation

## Contains tasks

1. Update apt
2. Setup unattend upgrades
3. Activates swap (if present and not added to /etc/fstab)
4. Uninstall packages
   - ppp
   - pppconfig
   - pppoeconf
   - popularity-contest
   - ntfs-3g
   - laptop-detect
   - apparmor
   - libapparmor-perl
5. Uninstall useless en_* locales except en_US and install cs_CZ
6. Install packages
   - acpid
   - htop
   - mc
   - vim
7. Set bash as default shell
8. Install and configure ntp client
9. Install local postfix SMTP
10. Install and configure monit (allow access from allow_global_ip)
11. Install and configure newrelic-sysmond
12. Install ipmitool (if not on VPS)
13. Install smartmontools (if not on VPS)
14. Set MAILTO for crontab
15. Set net.ifnames=0 and biosdevname=0 for grub to allow old ethX nic names

## Example playboot

```yaml
---
- hosts: myserver
  user: root
  sudo: False
  vars:
    newrelic_license_key: xxx
    admin_email: my@mail.tld
    allow_global_ip: 192.168.1.1
  roles:
   - common
```

### License

Licensed under MIT license. See [LICENSE](LICENSE.md) for details.
