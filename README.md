Ansible Role: postfix
=========

An Ansible Role that install and configure postfix on RHEL/CentOS, Fedora and Debian/Ubuntu. This role is not suited for email servers but other Linux systems that uses postfix to relay emails.

Requirements
------------

No requirements.

Role Variables
--------------

**Available variables are listed below, along with default values (see defaults/main.yml):**

> **NOTE**: if a variable is not defined (commented) the role will not change its original value on the system.

    pf_inet_interfaces: loopback-only

Interface addresses that can receive email. Be careful changing this value as it can make is system vulnerable. Recommended to leave as loopback-only.


    #pf_email_address: yourgmailaccount@gmail.com
    #pf_email_password: gmail_app_password
    #pf_smtp_server: smtp.gmail.com
    #pf_smtp_server_port: 587

Variables to setup relayhost and password maps file, i.e., information needed to relay emails to other systems. In the example above, just change username and password in order to send emails via gmail.

    #pf_mydomain: mydomain.com

The mydomain parameter specifies the local internet domain name.
The default is to use $myhostname minus the first component.
$mydomain is used as a default value for many other configuration parameters.

    #pf_smtp_sasl_auth_enable: yes

Enable SASL authentication

    #pf_smtp_sasl_security_options:
    #  - noanonymous

Disallow any methods that do allow anonymous authentication

    #pf_smtp_sasl_password_maps: /etc/postfix/sasl/sasl_passwd

Define the sasl_passwd file location

    #pf_smtp_use_tls: yes

Enable STARTTLS encryption. This option is deprecated, use pf_smtp_tls_security_level instead.

    #pf_smtp_tls_security_level: verify

The default SMTP TLS security level for the Postfix SMTP client;
when a non-empty value is specified, this overrides the obsolete parameters smtp_use_tls, smtp_enforce_tls, and smtp_tls_enforce_peername.

    #pf_smtp_tls_mandatory_ciphers: high

[Documentation about tls mandatory ciphers](http://www.postfix.org/postconf.5.html#smtp_tls_mandatory_ciphers)

    #pf_smtp_tls_cafile: /etc/ssl/certs/ca-certificates.crt
    #pf_smtp_tls_capath: /etc/ssl/certs

Location of CA certificates

    #pf_disable_vrfy_command: yes

The VRFY command is short for ‘verify’. It can be used to see if an email address is valid on the mail server. While this is great for troubleshooting, it also allows others to make educated guesses if an account exists and deliver possibly spam.
The VRFY command is not normally not needed for delivery between two mail servers

    #pf_mynetworks: "127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128"

Which network can use this relay server

    #pf_smtpd_helo_required: yes

Send HELO command

    #pf_smtpd_helo_restrictions:
    #  - permit_mynetworks
    #  - reject_non_fqdn_helo_hostname
    #  - reject_invalid_helo_hostname

Optional restrictions that the Postfix SMTP server applies in the context of a client HELO command. 

    #pf_smtpd_tls_loglevel: 1

Server side log level:

  - 0 	Disable logging of TLS activity.
  - 1 	Log only a summary message on TLS handshake completion — no logging of client certificate trust-chain verification errors if client certificate verification is not required. Log the summary message, peer certificate summary information and unconditionally log trust-chain verification errors.
  - 2 	Also log levels during TLS negotiation.
  - 3 	Also log hexadecimal and ASCII dump of TLS negotiation process.
  - 4 	Also log hexadecimal and ASCII dump of complete transmission after STARTTLS.

.

    #pf_smtp_tls_loglevel: 1

Client side log level:

- 0 	Disable logging of TLS activity.
- 1 	Log only a summary message on TLS handshake completion — no logging of remote SMTP server certificate trust-chain verification errors if server certificate verification is not required. Log the summary message and unconditionally log trust-chain verification errors.
- 2 	Also log levels during TLS negotiation.
- 3 	Also log hexadecimal and ASCII dump of TLS negotiation process.
- 4 	Also log hexadecimal and ASCII dump of complete transmission after STARTTLS.

.

    #pf_smtpd_recipient_restrictions:
    #  - reject_invalid_hostname
    #  - reject_non_fqdn_hostname
    #  - reject_non_fqdn_sender
    #  - reject_unauth_destination
    #  - reject_unknown_sender_domain
    #  - reject_rbl_client bl.spamcop.net
    #  - reject_rbl_clienti dnsbl.sorbs.net

[smtp_recipient_restrictions documentation](http://www.postfix.org/postconf.5.html#smtpd_recipient_restrictions)

    #pf_strict_rfc821_envelopes: yes

Require that addresses received in SMTP MAIL FROM and RCPT TO commands are enclosed with <>, and that those addresses do not contain RFC 822 style comments or phrases. This stops mail from poorly written software. By default, the Postfix SMTP server accepts RFC 822 syntax in MAIL FROM and RCPT TO addresses.

    #pf_smtpd_delay_reject: yes

[smtpd_relay_reject documentation](http://www.postfix.org/postconf.5.html#smtpd_delay_reject)

    #pf_smtpd_data_restrictions:
    #  - reject_unauth_pipelining

[smtpd_data_restrictions documentation](http://www.postfix.org/postconf.5.html#smtpd_data_restrictions)

    #pf_inet_protocols: ipv4

The Internet protocols Postfix will attempt to use when making or accepting connections.
Specify one or more of "ipv4" or "ipv6", separated by whitespace or commas. The form "all" is equivalent to "ipv4, ipv6" or "ipv4", depending on whether the operating system implements IPv6.

**The variables listed below do not need to be changed for targeted systems (see vars/main.yml):**

    pf_main_cf_path: /etc/postfix/main.cf

Postfix main.cf path.

    pf_main_cf_owner: root
    pf_main_cf_group: root
    pf_main_cf_mode: '0644'

Owner, group and privileges of configuration files.

    pf_packages:

Contains the list of packages to be installed based on linux distribution and version.

    pf_conflicting: ['sendmail']

Packages that conflics with Postfix and that will be removed by this role.

    pf_relayhost: "[{{ pf_smtp_server | default('none') }}]:{{ pf_smtp_server_port | default(50000) }}"

The relayhost parameter specifies the default host to send mail to when no entry is matched in the optional transport(5) table. When no relayhost is given, mail is routed directly to the destination. Default values are just to prevent error if these variables do not exist.

# Add/remove entries from /etc/aliases
    #pf_add_aliases:
    #  - { from: X, to: Y }
    #  - { from: W, to: Z }

Add/modify aliases.

    #pf_remove_aliases:
    #  - X

Remove aliases.

Dependencies
------------

No dependencies

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      vars:
        pf_inet_interfaces: loopback-only
        pf_email_address: yourgmailaccount@gmail.com
        pf_email_password: gmail_app_password
        pf_smtp_server: smtp.gmail.com
        pf_smtp_server_port: 587
      roles:
         - { role: guidugli.postfix }

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2020 by Carlos Guidugli.
