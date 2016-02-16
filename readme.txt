INTRODUCTION

This is a wrapper for acme-tiny
(https://github.com/diafygi/acme-tiny/) to automate certificate
renewal with Apache.

CONFIGURATION

You will need to configure the script for your setup; this was written
for my own use, but people have been asking about it so I'm uploading
it. I have this running as user acme with its own home directory,
with:

acme-tiny/acme_tiny.py
bin/newcert
lets-encrypt-x1-cross-signed.pem

and CSRs will get left in $HOME too. The PEM file comes from
https://letsencrypt.org/certs/lets-encrypt-x1-cross-signed.pem.txt .

/etc/apache2/certs and /var/www/challenges are owned by acme, both
world-readable. Each vhost listing in apache includes

       Alias /.well-known/acme-challenge/ /var/www/challenges/

Once the cert has been obtained, the SSL section of the config file
for that site should include:

SSLCertificateFile /etc/apache2/certs/$VHOSTNAME.crt
SSLCertificateKeyFile /etc/apache2/certs/$VHOSTNAME.key

The domain list (after the __DATA__ section in the script) consists of
one line per certificate request. Multiple domain names for the same
certificate are on the same line, joined by commas. A + after the
domain name indicates that www.domain should be added as well as
domain.

The _first_ domain in the list is used as the filename for
certificates, etc.

Examples:

foo.org - generate a cert for foo.org

foo.org+ - generate a cert for foo.org and www.foo.org

foo.org,bar.org+ - generate a cert for foo.org, bar.org and www.bar.org

OPERATION

Just run the script.

For each domain in the list, it will check:

- does a cert already exist? If so, does it have more than 30 days to
  run? If both, skip this domain.

- does a key exist? If not, generate it.

- does a CSR exist? If not, generate it. (Including SANs. If you
  change SANs, delete the CSR as well as the cert before re-running.)

- do the ACME dance.

- at the end, list all domains with new certs, to remind the user to
  restart Apache.
