[![CI](https://github.com/guidugli/ansible-role-postfix/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-postfix/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/tag/guidugli/ansible-role-postfix?sort=semver)](https://github.com/guidugli/ansible-role-postfix/tags)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.postfix-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/postfix/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

# Ansible Role: postfix

Installs and configures Postfix as a local relay client. The role is intended for Linux hosts that need to relay mail through an upstream SMTP service, not for operating a public inbound mail server. It manages selected `main.cf` options, optional SASL password maps, optional aliases, and validates the resulting Postfix configuration.

## Requirements

- Ansible Core 2.17 for local development, as pinned in `requirements-dev.txt`.
- Supported test platforms are defined by `molecule/shared/vars.yml` and rendered into the default and systemd Molecule scenarios.
- The caller must provide privilege escalation externally when the target host requires root access for package installation or files under `/etc`.
- `containers.podman` collection `>=1.10.0` is required for Molecule scenarios.

## Variables

Only `pf_inet_interfaces` is enabled by default. Optional variables may be supplied by inventory, group vars, host vars, or play vars. When an optional variable is not defined, the role leaves the corresponding Postfix setting untouched.

| Variable | Type | Default | Explanation |
| --- | --- | --- | --- |
| `pf_inet_interfaces` | string | `loopback-only` | Interfaces on which Postfix receives mail. The default keeps the service local-only. |
| `pf_email_address` | string | undefined | SMTP account or sender identity used in the SASL password map. |
| `pf_email_password` | string | undefined | SMTP password or app password written to the SASL password map. Mark this value secret in your inventory. |
| `pf_smtp_server` | string | undefined | Upstream SMTP relay host. |
| `pf_smtp_server_port` | integer | undefined | Upstream SMTP relay port, commonly `587` for submission. |
| `pf_mydomain` | string | undefined | Value for Postfix `mydomain`. |
| `pf_smtp_sasl_auth_enable` | boolean | undefined | Writes `smtp_sasl_auth_enable = yes/no`. |
| `pf_smtp_sasl_security_options` | list(string) | undefined | Writes `smtp_sasl_security_options`, for example `noanonymous`. |
| `pf_smtp_sasl_password_maps` | string | undefined | Lookup table for the credentials file, for example `hash:/etc/postfix/sasl/sasl_passwd`. |
| `pf_smtpd_sender_login_maps` | string | undefined | Optional map of SASL login names to permitted envelope senders. |
| `pf_smtp_tls_security_level` | string | undefined | TLS policy for outbound SMTP, such as `may`, `encrypt`, or `verify`. |
| `pf_smtp_tls_mandatory_ciphers` | string | undefined | Mandatory TLS cipher grade, such as `high`. |
| `pf_smtp_tls_cafile` | string | undefined | CA certificate file path. |
| `pf_smtp_tls_capath` | string | undefined | CA certificate directory path. |
| `pf_disable_vrfy_command` | boolean | undefined | Writes `disable_vrfy_command = yes/no`. |
| `pf_mynetworks` | string | undefined | Networks permitted to relay through the host. |
| `pf_smtpd_helo_required` | boolean | undefined | Requires SMTP clients to send HELO/EHLO when enabled. |
| `pf_smtpd_helo_restrictions` | list(string) | undefined | HELO restrictions written as a comma-separated Postfix value. |
| `pf_smtpd_tls_loglevel` | integer | undefined | Server-side TLS log level. |
| `pf_smtp_tls_loglevel` | integer | undefined | Client-side TLS log level. |
| `pf_smtpd_recipient_restrictions` | list(string) | undefined | Recipient restrictions written as a comma-separated Postfix value. |
| `pf_strict_rfc821_envelopes` | boolean | undefined | Enforces strict RFC 821 envelope syntax when enabled. |
| `pf_smtpd_delay_reject` | boolean | undefined | Writes `smtpd_delay_reject = yes/no`. |
| `pf_smtpd_data_restrictions` | list(string) | undefined | DATA restrictions written as a comma-separated Postfix value. |
| `pf_inet_protocols` | string | undefined | Protocol family used by Postfix: `all`, `ipv4`, or `ipv6`. |
| `pf_add_aliases` | list(dict) | undefined | Adds or updates `/etc/aliases` entries with `from` and `to` keys. |
| `pf_remove_aliases` | list(string) | undefined | Removes matching aliases from `/etc/aliases`. |

Internal variables in `vars/main.yml` define OS-specific package lists, service name, file ownership, validation choices, and variable-to-Postfix option mappings. They normally do not need to be overridden.

## Example Playbook

```yaml
---
- name: Configure local mail relay
  hosts: linux
  become: true
  vars:
    pf_inet_interfaces: loopback-only
    pf_smtp_server: smtp.example.com
    pf_smtp_server_port: 587
    pf_email_address: relay@example.com
    pf_email_password: "{{ vault_postfix_relay_password }}"
    pf_smtp_sasl_auth_enable: true
    pf_smtp_sasl_security_options:
      - noanonymous
    pf_smtp_sasl_password_maps: hash:/etc/postfix/sasl/sasl_passwd
    pf_smtp_tls_security_level: encrypt
    pf_inet_protocols: ipv4
    pf_add_aliases:
      - from: root
        to: ops@example.com
  roles:
    - role: guidugli.postfix
```

## Molecule Testing

The role uses shared Molecule playbooks in `molecule/shared` with scenario-specific container orchestration in `molecule/default` and `molecule/systemd`.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

The `default` scenario runs rootful containers with a simple sleep command. The `systemd` scenario prepares containers with systemd and validates service-manager behavior separately.

## Execution Notes

- **Privilege model:** the role never sets `become`, `become_user`, or `become_method`. Package management, `/etc/postfix`, `/etc/aliases`, `postmap`, and `newaliases` require root on real hosts, so playbooks should set `become: true` when needed.
- **Container behavior:** Molecule containers run as root, so the converge playbook intentionally uses `become: false`. This keeps role logic independent from privilege escalation policy.
- **Systemd behavior:** service enable and restart tasks run only when `ansible_facts['service_mgr'] == 'systemd'`. Non-systemd containers still install packages, write configuration, and run `postfix check`, but service management is skipped.
- **Idempotency:** configuration is managed with Ansible modules, command validation uses `changed_when: false`, and handlers run only when notified by changed files.
