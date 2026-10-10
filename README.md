# Ansible msmtp

Forked from the original work by Adham Helal and maintained by Sebastian Plocek.

This Ansible role configures `msmtp` for Debian-based systems and is intended for modern Debian releases.

## Prerequisites

* Ansible installed on your workstation
* A reachable SMTP server

## How to install

Install directly from Ansible Galaxy:

```bash
ansible-galaxy role install sebastian13.msmtp -p roles
```

Or clone it into your roles directory:

```bash
git clone https://github.com/sebastian13/ansible-msmtp.git roles/msmtp
```

## Variables

All default variables are defined in **defaults/main.yml**. Most setups need at least the following values:

- `msmtp_accounts`: one or more SMTP accounts
 
      ```
      msmtp_accounts:
      - account  : "gmail"
        host     : "smtp.gmail.com"
        port     : 587
        auth     : "on"
        user     : "example@gmail.example"
        password : "some password"
      - account  : "mysmtp"
        host     : "smtp.example"
        port     : 587
        auth     : "on"
        user     : "myuser"
        password : "123456"
      ```
- `msmtp_default_account`: default SMTP account to use

  ```yaml
  msmtp_default_account: "gmail"
  ```

- `msmtp_from`: sender address / From field

  ```yaml
  msmtp_from: "No Reply"
  ```

### Logging

- Option A: syslog

  ```yaml
  msmtp_log: "syslog"
  ```

- Option B: file logging

  ```yaml
  msmtp_log: "file"
  msmtp_logfile: /var/log/msmtp.log
  ```

- Option C: no logging

  ```yaml
  msmtp_log: "no"
  ```

### Mail aliases

- `msmtp_alias_default`: default address

  ```yaml
  msmtp_alias_default: ops@example.com
  ```

- `msmtp_alias_root`: root alias

  ```yaml
  msmtp_alias_root: root@example.com
  ```

- `msmtp_alias_cron`: cron alias

  ```yaml
  msmtp_alias_cron: cron@example.com
  ```

## Configure

You can configure the role with any of the following methods:

* host/group vars (recommended)
* editing `vars/main.yml`
* passing vars via `ansible-playbook -e`
* editing `defaults/main.yml` (not recommended)

## Run

> By default, the role should not be used with a bogus SMTP server. Configure a valid SMTP endpoint before applying it.

```bash
ansible-playbook -l hostname msmtp.yml
```

## Test

You should receive a test mail in the configured root or alias address when the role is working correctly.

## Possible issues

The From field may require additional tuning depending on your mail provider and envelope settings:

[msmtp documentation](http://msmtp.sourceforge.net/doc/msmtp.html#Envelope_002dfrom-address)
