# Letsencrypt

The letsencrypt role, installs letsencrypt, and tries to acquire certificates for all domains defined in the letsencrypt_domains_list variable, defined in the host_vars/<host name> file. To get new certificates (for new domains,subdomains), simply add the domain to the list, and run ansible.
This role was made to work with nginx.

Note that, because letsencrypt is configured to acquire certificates via webroot authentication, proper nginx configuration must be in place.

## Important note
Certbot can be installed and used in many ways.
This role started by using certbot bare - by cloning the git repository and running it from there.
It has since moved to installation via the native package manager, snap and pip.
Pip install has been necessary in order to use 3rd party plugins. But it hsa proven to be problematic as it can and will break Python dependencies.
So if you need plugins, use the "docker" letsencrypt_install_method. Otherwise use "native"
The docker image will be further expanded in the future, to cover more plugins.

## Variables
mandatory variables:
* letsencrypt_email: "it@cosylab.com"
* letsencrypt_domains_list: - list of domains to get cert for
  - name: "<name>" - user friendly name of the config
    domains: - list of domain names this cert is valid for
    plugin: overrides letsencrypt_plugin
    dns_secret: optional - which secret to use (see letsencrypt_dns_secrets.name below)
* letsencrypt_install_method: string representing the install method
* letsencrypt_service_hooks: List of service to restart after renew

Optional variables:
* letsencrypt_root_dir: Where to store configs and challanges default("/opt/letsencrypt")
* letsencrypt_config_dir: default config dir - defaults inside the root dir.
* letsencrypt_challenge_dir: challane dir, default inside root dir.
* letsencrypt_key_type: Type of key to use "rsa" or "ecdsa"
* letsencrypt_rsa_key_strength: default(4096)
* letsencrypt_plugin: Which plugin to use to get certs - default "webroot"
Optional variables for DNS authentication:
* letsencrypt_dns_secrets: - object - list of items that represent dns secrets
  - name: user friendly name - used for identification in configuration
    type: can be "dns-godaddy" or "dns-cloudflare" for now. Can add more if needed
    key: identifier "key" in godaddy case or email in cloudflare case
    secret: the secret or key(as refered to at cloudflare)
* letsencrypt_dns_secret: Default secret to use (secret name) as used in the letsencrypt_dns_secrets objects. This can be ovverriden per domain.
* letsencrypt_dns_secrets_dir: Directory where .ini secrets files are stored.

## First time certificate acquiry
To acquire certificates for a new domain for the first time with minimal/zero downtime:

* Update nginx config files to include the letsencrypt.inc file, which defines the location necessary for the webroot authentication. This file must be included in all nginx server definitions for which we will be fetching certificates. And run ansible with the nginx role.
* Run ansible letsencrypt role to get certificates.
* Update nginx config files and run ansible with nginx role:
    * listen directive: `listen 443 ssl http2;`
    * ssl_certificate: `ssl_certificate <path to certificate>;`
    * ssl_certificate_key: `ssl_certificate_key <path to key file>;`

Note: Make sure that firewall port 443/tcp is open.

## Adding certificates for other domains/subdomains
To add a certificate for a subdomain of an already configured domain, add the subdomain to the letsencrypt_domains_list variable and run ansible with letsencrypt role. Don't forget to reload nginx afterwards so it starts using the new certificate.

## Important details
If a (sub)domain is removed from the list or the order is changed, a new certificate will be generated. For example:
If a certificate for example.com exists in /etc/letsencrypt/example.com, removing a domain from list will create a new directory "/etc/letsencrypt/example.com-001" where the new certificate will be placed. It is safe in such cases to remove the whole /etc/letsencrypt directory and issue all certificates anew.
