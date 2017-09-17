# Postfix Docker Container

Postfix SMTP Relay based on the work from <https://hub.docker.com/r/panubo/postfix/>.

This version adds capabilities for correcting the MAIL_FROM SMTP header and rewriting it to the email of your choice.  Useful to ensure SPF passes when email is received by destination email servers.

Drop-in container for SMTP relaying. Use wherever a connected service
requires SMTP sending capabilities. Supports TLS out of the box and DKIM
(if enabled and configured).

## Environment Variables

### General Configuration

- `MAILNAME` - set this to a legitimate FQDN hostname for this service (required).
- `MYNETWORKS` - comma separated list of IP subnets that are allowed to relay. Default `127.0.0.0/8, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16`
- `LOGOUTPUT` - Syslog log file location. eg `/var/log/maillog`. Default `/dev/stdout`.

### Relay Correction

- `REWRITE_MAIL_FROM` - The email address to rewrite `original@example.com`
- `MAIL_FROM` - The desired value to have for the orginial address `corrected@example.com`
- `CANONICAL_CLASSES` - The classes that this rewrite should be applied to.  Default `envelope_sender`

### General Postfix

- `SIZELIMIT` -  Postfix `message_size_limit`. Default `15728640`.
- `RELAYHOST` -  Postfix `relayhost`. Default `empty`.
- `POSTFIX_ADD_MISSING_HEADERS` - add missing headers. Default `no`
- `INET_PROTOCOLS` - IP protocols, eg `ipv4` or `ipv6`. Default `all`
- `BOUNCE_ADDRESS` - Email address to receive delivery failure notifications. Default is to log the delivery failure.

### TLS parameters

- `USE_TLS` - Enable opportunistic TLS. Default `yes`
- `TLS_KEY` - Default `/etc/ssl/private/ssl-cert-snakeoil.key`
- `TLS_CRT` - Default `/etc/ssl/certs/ssl-cert-snakeoil.pem`
- `TLS_CA` - Default ''

_NB. A "snake-oil" certificate will generated on start if required._

### DKIM parameters

- `USE_DKIM` - Enable DKIM. Default `no`
- `DKIM_KEYFILE` - DKIM Keyfile location. Default `/etc/opendkim/dkim.key`
- `DKIM_DOMAINS` - Domains to sign. Defaults to `MAILNAME`. Multiple domains will use the same key and selector.
- `DKIM_SELECTOR` - DKIM key selector. Default `mail`. `<selector>._domainkey.<domain>` is used for resolving the public key in DNS.
- `DKIM_INTERNALHOSTS` - Defaults to `MYNETWORKS`.
- `DKIM_EXTERNALIGNORE` - Defaults to `MYNETWORKS`.

## Usage Example

`docker run -e MAILNAME=mail.example.com panubo/postfix`

## Volumes

No volumes are defined. If you want persistent spool storage then mount
`/var/spool/postfix` outside of the container.

## Test email

To send a test email via the command line, make sure heirloom-mailx is installed.

```
echo -e "To: Bob <bob@example.com>\nFrom: Bill <bill@example.com>\nSubject: Test email\n\nThis is a test email message" | mailx -v -S smtp=smtp://... -S from=bill@example.com -t

# With TLS
echo -e "To: Bob <bob@example.com>\nFrom: Bill <bill@example.com>\nSubject: Test email\n\nThis is a test email message" | mailx -v -S smtp-use-starttls -S ssl-verify=ignore -S smtp=smtp://... -S from=bill@example.com -t
```

## Developing

See the `Makefile` for make targets. eg To build run `make build`.

## Status

Production ready and stable.
