# Letsencrypt

The letsencrypt role, installs letsencrypt, and tries to acquire certificates for all domains defined in the letsencrypt_domains_list variable, defined in the host_vars/<host name> file. To get new certificates (for new domains,subdomains), simply add the domain to the list, and run ansible.

Note that, because letsencrypt is configured to acquire certificates via webroot authentication, proper nginx configuration must be in place.

### First time certificate acquiry
To acquire certificates for a new domain for the first time with minimal/zero downtime:

* Update nginx config files to include the letsencrypt.inc file, which defines the location necessary for the webroot authentication. This file must be included in all nginx server definitions for which we will be fetching certificates. And run ansible with the nginx role.
* Run ansible letsencrypt role to get certificates.
* Update nginx config files and run ansible with nginx role:
    * listen directive: `listen 443 ssl http2;`
    * ssl_certificate: `ssl_certificate <path to certificate>;`
    * ssl_certificate_key: `ssl_certificate_key <path to key file>;`

Note: Make sure that firewall port 443/tcp is open.

### Adding certificates for other domains/subdomains
To add a certificate for a subdomain of an already configured domain, add the subdomain to the letsencrypt_domains_list variable and run ansible with letsencrypt role. Don't forget to reload nginx afterwards so it starts using the new certificate.
