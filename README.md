# ansible-common

Ansible role for initial Ubuntu server configuration, including VPS instances
created from official Ubuntu cloud images.

This role is highly opinionated. You should fork it and make your own adjustments.

## Additional task files

The `grub.yml`, `monit.yml` and `purge.yml` task files are available for explicit
import if needed.

## Requirements

- An Ubuntu target with APT, systemd and Python available for Ansible.
- Root privileges, either by connecting as root or using `become: true`.
- Fact gathering enabled for tasks that inspect the Ubuntu version or
  virtualization role.
- The `community.general` collection for timezone, locale and package diversion
  tasks.
- The `ansible.utils` collection and Python `netaddr` in the Ansible controller
  environment for the Fail2ban IP address filter. See the
  [filter requirements](https://docs.ansible.com/projects/ansible/latest/collections/ansible/utils/ipaddr_filter.html#requirements).
- The `ansible.posix` collection for sysctl tasks.
- The `locales` and `tzdata` packages on the target when enabling locale and
  timezone tasks.

## Example playbook

```yaml
---
- name: Configure Ubuntu servers
  hosts: myserver
  remote_user: root
  vars:
    admin_email: admin@example.com
    admin_ips:
      - 192.0.2.10
      - 2001:db8::10
    postfix_myhostname: vps01.example.com
  roles:
    - sunfoxcz.common
```

When connecting as an unprivileged user, set `remote_user` accordingly and add
`become: true` to the play.

## Variables without defaults

| Variable | Used by | Description |
| --- | --- | --- |
| `admin_email` | Postfix and unattended-upgrades | Recipient for local root mail and unattended-upgrades error reports. Required by these task groups. |
| `admin_ips` | Fail2ban and Monit | List of administrator IPv4 or IPv6 addresses to allow. Required when enabling these tasks. |

## Optional variables

### `postfix_myhostname`

The fully qualified hostname Postfix should use to identify this server, for
example:

```yaml
postfix_myhostname: vps01.example.com
```

When defined, the role sets `myhostname` in `/etc/postfix/main.cf` and notifies
the Postfix restart handler. The role always sets
`mydestination = $myhostname, localhost`, so this hostname is also treated as a
local mail destination. With Postfix's default `smtp_helo_name`, it is the name
sent in outgoing SMTP HELO/EHLO commands.

When omitted, the role leaves `myhostname` untouched. An existing configured
value remains in use; otherwise Postfix derives its default from the system
hostname and domain configuration.

This variable changes Postfix configuration only. It does not rename the server,
update DNS records or write `/etc/mailname`. The role does not set `myorigin`,
which controls the domain added to locally submitted unqualified addresses, so
the sender address also depends on the existing Postfix configuration.

See the Postfix documentation for
[`myhostname`](https://manpages.debian.org/trixie/postfix/postconf.5.en.html#myhostname),
[`smtp_helo_name`](https://manpages.debian.org/trixie/postfix/postconf.5.en.html#smtp_helo_name)
and [`myorigin`](https://manpages.debian.org/trixie/postfix/postconf.5.en.html#myorigin).

### `root_default_password`

An encrypted password hash passed to `ansible.builtin.user` for the root account
when `ssh.yml` is enabled. Supply a password hash, not a plaintext password.
If omitted, the role does not change the root password. This task does not
configure SSH password authentication or root login permissions.

## Default variables

The following values match [defaults/main.yml](defaults/main.yml) and can be
overridden in inventory or play variables:

```yaml
apt_cache_valid_time: 86400
apt_mirror: cz.archive.ubuntu.com

dns_servers:
  - 1.1.1.1
  - 1.0.0.1

systemd_journal_max_size: 100M

timezone: Europe/Prague
locales:
  - en_US.UTF-8
  - cs_CZ.UTF-8
```

| Variable | Behavior |
| --- | --- |
| `apt_cache_valid_time` | APT cache validity in seconds. A mirror change forces a cache refresh. |
| `apt_mirror` | Archive hostname, without a URL scheme or path. On Ubuntu 24.04 and later, replaces standard, regional or cloud `archive.ubuntu.com` hostnames in `/etc/apt/sources.list.d/ubuntu.sources`, preserving the rest of the file. Security and ports archive URLs are left unchanged. |
| `dns_servers` | Nameservers written to `/etc/resolv.conf` when creating it. The DNS tasks use service facts to stop and disable systemd-resolved when installed, remove a resolv.conf symlink and create a regular file. An existing regular file is not overwritten. |
| `systemd_journal_max_size` | Value of `SystemMaxUse` in `/etc/systemd/journald.conf`. Changes restart journald. |
| `timezone` | System timezone managed by `community.general.timezone`. Changes notify the cron restart handler. |
| `locales` | Locales ensured present by `community.general.locale_gen`; other enabled locales are not automatically removed. Before Ubuntu 26.04, an existing `/var/lib/locales/supported.d/en` is restricted to `en_US.UTF-8 UTF-8`, independently of this list. |

## Mail delivery

The Postfix tasks install and enable Postfix, forward mail addressed to the local
root user to `admin_email` through `/etc/aliases`, and run `newaliases` when the
alias changes. They also configure IPv4-only mail transport and opportunistic
outgoing TLS (`smtp_tls_security_level = may`).

Cron jobs that send their output to root use this alias without needing a
`MAILTO` entry. Existing `MAILTO` settings in crontab files are left unchanged:
explicit recipients and an empty value that disables mail take precedence.
Mail addressed to other local users is not redirected by the root alias.

The role does not configure an SMTP relay or SMTP authentication. External mail
delivery uses the target's existing Postfix routing configuration.

## Automatic updates

The APT tasks configure daily package list refreshes and
unattended upgrades in `/etc/apt/apt.conf.d/20auto-upgrades`. They also manage
`/etc/apt/apt.conf.d/50unattended-upgrades` from a template, validated with
`apt-config` before installation.

The template replaces the `Allowed-Origins` list with the Ubuntu release,
`-security` and `-updates` archives. Updates from a PPA or another third-party
origin are not enabled by this list. Error reports go to `admin_email` using
`MailReport = only-on-error`. Unused automatically installed packages are removed
after upgrades, equivalent to `apt autoremove`.

The APT tasks also divert the Ubuntu ESM APT hook and ESM Apps source file to
names ending in `.disabled`.

## Other task groups

- `chrony.yml` installs and enables chrony, removes `pool` lines from
  `/etc/chrony/chrony.conf` and installs the Czech time sources from
  [files/chrony.sources](files/chrony.sources): `tik.cesnet.cz`, `tak.cesnet.cz`
  and `ntp.nic.cz`. These sources are configured in the file rather than a role
  variable. `ntp.yml` and `systemd-timesyncd.yml` remove the previous time clients.
- `packages.yml`, `bash.yml`, `motd.yml` and `systemd.yml` provide basic packages,
  shell configuration, MOTD changes and the journal size limit.
- `fail2ban.yml` configures Fail2ban logging and administrator IP exclusions.
- `ipmitool.yml` and `smartmontools.yml` install hardware tools, including
  `nvme-cli`; their import conditions exclude guests.
- `cloud-init.yml` removes the `cloud-init` package.
- `sysctl.yml` applies fixed swap, file limit and network settings.
- `zfs.yml` diverts `zpool_influxdb` to `influxdb_zpool` when `zpool` exists;
  its import is limited to Ubuntu 24.04.

## License

Licensed under MIT license. See [LICENSE](LICENSE.md) for details.
