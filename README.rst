DANE Generator
==============

This is a collection of scripts allowing to generate automatically
DANE TLSA records and SSHFP record by collecting certificates and ssh
public keys across multiples servers.

Features
--------

 * Collect informations from multiples servers
 * Support the letsencrypt files tree
 * Generate dns zone files to be included with
   `$INCLUDE /etc/bind/generated/db.tlsa.example.com example.com.`
 * Only support plain certificate pinning for TLSA (not public key pinning, selector 0)
 * No CA pinning, only Certificate pinning (certytype 1 or 3)
 * Support usage of multiple hash algorithm (reftype 0, 1 or 2)

Requirements
------------
Theses scripts have only been testes on debian and ubuntu. They should work on other UNIX like systems.
They most probably will not work on windows systems.

You need the OpenSSL python module installed (`apt-get install python-openssl` on debian/ubuntu like systems).
After that, we only use python 2.7 standard library modules.

You will also need to have an `inetd` daemon intalled, like `openbsd-inetd` on debian/ubuntu like systems.

The script `list_certs` is a pseudo http server.
`echo "GET / DD" | ./list_certs` should produce a json on stdout with certificates infos.
We use inetd for binding it to a ip:port. I recommand to bind ip to a VPN ip (like tinc)
and to only call ip throuth the VPN to ensure validity of the informations gathered.
So you should probably also setup a VPN between the servers you want to collect
certificates infos. `Tinc <https://www.tinc-vpn.org>`_ is a VPN software that
is really simple to setup.


Installation
------------

Copy `config.py.dist` to `config.py` and edit to match you setup


Add `/path/to/list_certs` to you `/etc/inetd.conf`:
```
ip_vpn:PORT    stream  tcp     nowait  root    /path/to/list_certs
```


Usage
-----


`list_certs` will search for certificates (expired certificates are excluded) in the following locations:
 *   `/etc/ssl/private/[dns_zone]/[dns_local_name]/ssl.crt`
 *   `/etc/ssl/private/[dns_zone]/[dns_local_name]/ssl.crt.new`
 *   `/etc/ssl/private/[dns_zone]/[dns_local_name]/ssl.crt.old`
 *   `/etc/letsencrypt/live/[dns_local_name].[dns_zone]/cert.pem  --> ../../archive/[dns_name].[dns_zone]/cert(i).pem`
 *   `/etc/letsencrypt/archive/[dns_name].[dns_zone]/cert(i-1).pem`


It will also read the files for finding which services use the certificates:
 *   `/etc/ssl/private/[dns_zone]/[dns_name]/services`
 *   `/etc/letsencrypt/live/[dns_name].[dns_zone]/services`

The file `services` must have one service names by line


Alias are search in `/etc/ssl/private/[dns_zone]/[dns_name]/alias`,
one by line, fqdn must ends with a `'.'`.
We use subjectAltnames and CN for letsencrypt certificates and any
alias files in  `/etc/letsencrypt/live/*` will be ignored.


Service definitions are made in `/etc/ssl/private/service.ini` using this format:
```
[service_name]
transport=(tcp|udp)
ports=port1 port2 port3
reload=some_command
```
The reload command is optional and is not used here, but it can be used by a certificate generation script
to make `service_name` aware of a new certificate.


`list_certs` will also search for ssh public keys in `/etc/ssh/*.pub`


Finally `make_tlsa` will ask to all servers listed in `config.py` for json config
and generate TLSA and SSHFP records in `[BIND_BASE_PATH].[dns_zone]`.
