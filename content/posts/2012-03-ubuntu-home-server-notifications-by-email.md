---
title: "Ubuntu home server: Notifications by email"
date: 2012-03-16T19:28:08Z
draft: false
disclaimer: represent
aliases:
    - /articles/ubuntu-home-server-notifications-by-email/
---

This walkthrough tells you how to provide an email service to daemons on a home server so that it can send emails to a server admin's Gmail account.

Key: **Actions look like this**, _results look like this_ and `commands you enter on a terminal look like this`. Replace `[my_username]` with your login on this server e.g. `andrew`. Replace `[external_FQDN]` with the domain name that you use to access your server from outside your local network. (FQDN is Fully Qualified Domain Name.) Replace `[gmail address]` with your normal email address. This should work just as well for non-gmail addresses, but it's a useful distinction to show we'll be sending mail outside our local network.

### Pre-requisites

- Computer running Ubuntu (This was done on 10.04, but it's fairly standard stuff)
- Domain name and DNS provider who can make this work - e.g. dyn.com

```bash
sudo aptitude install postfix
```

_Postfix installs_
\
_Postfix starts its configuration gui_

**Select defaults for:**

- General type of mail configuration
- System mail name

```bash
sudo dpkg-reconfigure postfix
```

_Postfix starts its configuration gui_

**Select the following options:**

- General type of mail configuration: `Internet Site`
- System mail name: `[external_FQDN]`
- Root and postmaster mail recipient: `[my_username]`
- Other destinations to accept mail for (blank for none): `localhost.[external_FQDN], localhost`
- Force synchronous updates on mail queue?: `No`
- Local networks: `127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 [internal CIDR block, e.g. 192.168.0.0/24]`
- Mailbox size limit (bytes): `0`
- Local address extension character: `+`
- Internet protocols to use: `all`

### Configure Postfix for SMTP-AUTH using Dovecot SASL

```bash
sudo postconf -e 'smtpd_sasl_type = dovecot'
sudo postconf -e 'smtpd_sasl_path = private/auth-client'
sudo postconf -e 'smtpd_sasl_local_domain ='
sudo postconf -e 'smtpd_sasl_security_options = noanonymous'
sudo postconf -e 'broken_sasl_auth_clients = yes'
sudo postconf -e 'smtpd_sasl_auth_enable = yes'
sudo postconf -e 'smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination'
sudo postconf -e 'inet_interfaces = all'
```

_Postfix is configured silently - there is no output from these commands unless there's a problem._

### Generate the keys for the Certificate Signing Request (CSR)

```bash
openssl genrsa -des3 -out server.key 1024
```

**Enter passphrase when prompted**
\
_server.key file is created in your current working directory._

Now create the insecure key (no passphrase):

```bash
openssl rsa -in server.key -out server.key.insecure
```

**Enter passphrase when prompted**
\
_server.key.insecure file is created in your current working directory._

Name the key files appropriately:

```bash
mv server.key server.key.secure
mv server.key.insecure server.key
```

_`server.key.secure` and `server.key` files are present in your current working directory._

Create the CSR using the insecure key:

```bash
openssl req -new -key server.key -out server.csr
```

In the next step, you'll fill in some details. The only important option is the Common Name, which should be the FQDN of the server. This is slightly different to the [advice](https://en.wikipedia.org/wiki/Certificate_signing_request) on Wikipedia which indicates that the CN(Common Name) is used as part of the DN(Distinguished Name).

**Fill in some details about:** Country Name; State or Province Name; Locality Name; Organization Name; Organizational Unit Name; Common Name; Email address.

**When prompted for the following optional attributes, leave them blank:** A challenge password; An optional company name.

_`server.csr` file is create in your current working directory._

### Create a self-signed certificate and install it

Note that this certificate will be valid from now until an end date determined by the number after the `-days` option.

```bash
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
sudo cp server.crt /etc/ssl/certs
sudo cp server.key /etc/ssl/private
```

### Configure Postfix to provide TLS encryption for incoming and outgoing mail

```bash
sudo postconf -e 'smtpd_tls_auth_only = no'
sudo postconf -e 'smtp_tls_security_level = may'
sudo postconf -e 'smtpd_tls_security_level = may'
sudo postconf -e 'smtp_tls_note_starttls_offer = yes'
sudo postconf -e 'smtpd_tls_key_file = /etc/ssl/private/server.key'
sudo postconf -e 'smtpd_tls_cert_file = /etc/ssl/certs/server.crt'
sudo postconf -e 'smtpd_tls_loglevel = 1'
sudo postconf -e 'smtpd_tls_received_header = yes'
sudo postconf -e 'smtpd_tls_session_cache_timeout = 3600s'
sudo postconf -e 'tls_random_source = dev:/dev/urandom'
sudo postconf -e 'myhostname = [external_FQDN]'
```

Now restart postfix:

```bash
sudo /etc/init.d/postfix restart
```

_Postfix should restart with no errors_

### Configuring SASL (Simple Authentication and Security Layer)

```bash
sudo apt-get install dovecot-common
```

_Dovecot will install._

Edit `/etc/dovecot/dovecot.conf` as root (e.g. `sudoedit /etc/dovecot/dovecot.conf`
    
On line 1116, or thereabouts, **uncomment the socket listen option and modify the section so it looks like this**:

```bash
 socket listen {
    #master {
      # Master socket provides access to userdb information. It's typically
      # used to give Dovecot's local delivery agent access to userdb so it
      # can find mailbox locations.
      #path = /var/run/dovecot/auth-master
      #mode = 0600
      # Default user/group is the one who started dovecot-auth (root)
      #user = 
      #group = 
    #}
    client {
      # The client socket is generally safe to export to everyone. Typical use
      # is to export it to your SMTP server so it can do SMTP AUTH lookups
      # using it.
      path = /var/spool/postfix/private/auth-client
      mode = 0660
      user = postfix
      group = postfix
    }
  }
```

Now restart Dovecot

```bash
sudo /etc/init.d/dovecot restart
```

### Setting up Aliases

Edit `/etc/aliases` as root (e.g. `sudoedit /etc/aliases`) to add your gmail address. Once you've finished, it should look like this:

```bash
# See man 5 aliases for format
postmaster:    root
root:          [gmail address]
[my_username]: [gmail address]
```

### Testing

Lets see if we can connect to our postfix instance with telnet.

```bash
telnet localhost 25
```

_... results in the following:_

```bsah
Trying ::1...
Connected to localhost.
Escape character is '^]'.
220 [external_FQDN] ESMTP Postfix (Ubuntu)
```

**Type the following command into the telnet session:**

```bash
ehlo [external_FQDN]
```

_The output should include the following lines (and probably a bunch of others):_

```bash
250-[external_FQDN]
250-STARTTLS
250-AUTH PLAIN
250-AUTH=PLAIN
250-8BITMIME
```


Let's follow that up by sending an email directly from the telnet session.

**Type the following commands into the telnet session:**

```bash
mail from: root@localhost
rcpt to: [my_username]@localhost
data
Subject: My first mail on Postfix

Hello,
Are you there, Charlie Bear?
Regards,
Me
. (I.e. Type the . [dot] in a new Line and press Enter)
quit
```

_Postfix will acknowledge each command with a message ending in 'Ok' (except when you type the message contents). The output should look a bit like this:_

```bash
250 2.0.0 Ok: queued as 402DA9FCD4
quit
221 2.0.0 Bye
Connection closed by foreign host.
```

Wait for it... Okay - now **check your email**. If all has gone well, _you've got an email from yourself sitting in your inbox_.

These instructions were pieced together from [Postfix: Ubuntu server guide](https://help.ubuntu.com/10.04/serverguide/C/postfix.html) and [Certificates: Ubuntu server guide](https://help.ubuntu.com/10.04/serverguide/C/certificates-and-security.html).
