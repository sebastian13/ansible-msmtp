# Ansible msmtp

Configures [msmtp](https://marlam.de/msmtp/) as system MTA (`msmtp-mta`) on Debian.

Based on the original role by Adham Helal and its extended version
[chriswayg/ansible-msmtp-mailer](https://github.com/chriswayg/ansible-msmtp-mailer) by Christian Wagner.
Variable names are compatible with `chriswayg.msmtp-mailer`, so this role is a drop-in replacement.

Additions compared to `chriswayg.msmtp-mailer`:

- `set_from_header` (msmtp >= 1.8.8), so cron mails get a From header with a domain
  instead of `root (Cron Daemon)`. Required for DMARC alignment, e.g. at Microsoft 365.
- `/etc/msmtprc` is `root:msmtp 0640` and `/usr/bin/msmtp` is setgid `msmtp`:
  SMTP passwords are not readable by other users, but every user can still send mail.
- Test mail is sent with explicit `From`/`To` headers (providers like Lettermint reject `To: root`).
- Per-account `tls` option.
- Per-account `from_full_name` (msmtp >= 1.8.24) for a display name in the From header,
  e.g. `EXAMPLE.web01 <EXAMPLE.web01@example.com>` instead of the bare address.
- Fails on hosts with postfix instead of replacing it (Proxmox VE, Backup Server, Mail Gateway,
  mail servers, relays). To switch such a host to msmtp, remove postfix manually first.
- Debian only (bookworm, trixie).

## Installation

```yaml
# requirements.yml
roles:
  - name: sebastian13.msmtp
    src: https://github.com/sebastian13/ansible-msmtp.git
```

```bash
ansible-galaxy install -r requirements.yml
```

## Example

```yaml
- hosts: all
  become: true
  roles:
    - role: sebastian13.msmtp
      vars:
        msmtp_accounts:
          - account: "mail"
            host: "smtp.example.com"
            port: 587
            auth: "on"
            from: "{{ inventory_hostname }}@example.com"
            user: "{{ inventory_hostname }}@example.com"
            password: "{{ vault_smtp_password }}"
        msmtp_default_account: "mail"
        msmtp_domain: "example.com"
        msmtp_alias_default: "ops@example.com"
```

## Variables

See [defaults/main.yml](defaults/main.yml) for all variables and their defaults.

| Variable | Default | Description |
|---|---|---|
| `msmtp_accounts` | example | List of accounts: `account`, `host`, `port`, `auth`, `from`, optional `user`, `password`, `tls`, `tls_starttls`, `from_full_name` |
| `msmtp_default_account` | `mysmtp` | Account used by default |
| `msmtp_domain` | host FQDN | EHLO/HELO domain |
| `msmtp_tls_starttls` | `on` | Default for all accounts |
| `msmtp_tls_trust_file` | `/etc/ssl/certs/ca-certificates.crt` | CA bundle |
| `msmtp_set_from_header` | `on` | `on`, `auto`, `off`, or `""` to omit; only written for msmtp >= 1.8.8 |
| `msmtp_log` | `syslog` | `syslog`, `file` or `no` |
| `msmtp_syslog_facility` | `LOG_MAIL` | Used with `syslog` |
| `msmtp_logfile` | `/var/log/msmtp.log` | Used with `file` |
| `msmtp_alias_default` | `ops@example.com` | Default target in `/etc/aliases` |
| `msmtp_alias_root`, `msmtp_alias_cron` | undefined | Optional targets for `root` and `cron` |
| `msmtp_send_test_mail` | `false` | Send a test mail when the configuration changes |
| `msmtp_test_mail_recipient` | `ops@example.com` | Recipient of the test mail |
| `msmtprc_owner` / `msmtprc_group` / `msmtprc_mode` | `root` / `msmtp` / `0640` | Permissions of `/etc/msmtprc` |
| `msmtp_binary_setgid` | `true` | Make `/usr/bin/msmtp` setgid `msmtprc_group` via `dpkg-statoverride` |

## Migration from chriswayg.msmtp-mailer

- Replace the role name. Existing variables keep working.
- Arch Linux and Alpine are no longer supported.
- Hosts with postfix fail (chriswayg replaced postfix silently). Exclude them from the play.
- `msmtp_remove_mtas` is gone: installing `msmtp-mta` already removes exim4 or sendmail via apt.
- Defaults changed: `msmtp_log` is `syslog` (was `file` in `~/.msmtp.log`), `/etc/msmtprc` is `0640`
  (was `0644`), `set_from_header on` is added.
- With `set_from_header on`, the From header is always the account's `from`. A `MAILFROM` set in crontab
  is overridden in the header.
- The handler is called `Send msmtp test mail` and also listens to `test mail`.
- The test mail goes to `msmtp_test_mail_recipient` only, no longer additionally to `root`.

## Test

As a non-root user on the host:

```bash
cat /etc/msmtprc                                   # Permission denied
echo test | msmtp ops@example.com                  # mail is delivered
ls -l /usr/bin/msmtp                               # -rwxr-sr-x root msmtp
```

## License

Apache License 2.0, see [LICENSE](LICENSE).
