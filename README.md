# PowerDNS Authoritative Server - MySQL Enabled Docker Image

The PowerDNS Auth Server is a powerful, authoritative DNS server with several exciting features. A [full description](https://www.powerdns.com/auth.html) can be found on the [project website](https://www.powerdns.com/).

## Credits

This image was heavily based on the great work of [Peter Schiffer](https://github.com/pschiffe/).

The reason for creating another version of the image, was:

* To use the upstream packages from the [PowerDNS Repositories](https://repo.powerdns.com/).
* To use a base image with a longer lifecycle.

## The Image

<https://hub.docker.com/r/tmuncks/pdns-auth/>

Docker image with PowerDNS Authoritative Server.

The image is MySQL enabled, so needs an external MySQL server to function. ENV variables can be used for MySQL configuration:

```text
(name=value)

PDNS_gmysql_host=mysql
PDNS_gmysql_port=3306
PDNS_gmysql_user=root
PDNS_gmysql_password=passw0rd
PDNS_gmysql_dbname=powerdns
```

If linked with official [mariadb](https://hub.docker.com/_/mariadb/) image with alias `mysql`, the connection can be automatically configured, so you don't need to specify any of the above.

The database is automatically created if missing and initialized if tables are missing. This can be avoided by placing `SKIP_DB_CREATE=true` in the list of ENV variables.

The PowerDNS server is configurable via env vars. Every variable starting with `PDNS_` will be inserted into the default `/etc/powerdns/pdns.conf` config file in the following way:

* Prefix `PDNS_` will be stripped
* Every `_` will be replaced with `-`

Example from above mysql config; `PDNS_gmysql_host=mysql` will become `gmysql-host=mysql` in `/etc/powerdns/pdns.conf` file. This way, you can configure PowerDNS server any way you need, within a `docker run` command.

There is also a `SUPERMASTER_IPS` env var supported, which can be used to configure supermasters for slave dns server. [Docs](https://doc.powerdns.com/md/authoritative/modes-of-operation/#supermaster-automatic-provisioning-of-slaves). Multiple ip addresses separated by space should work.

You can find [here](https://doc.powerdns.com/authoritative/settings.html) all available settings.

## Examples

Master server with API enabled and with one slave server configured:

```bash
docker run -d -p 53:53 -p 53:53/udp --name pdns-master \
  --hostname ns1.example.com --link mariadb:mysql \
  -e PDNS_master=yes \
  -e PDNS_api=yes \
  -e PDNS_api_key=secret \
  -e PDNS_webserver=yes \
  -e PDNS_webserver_address=0.0.0.0 \
  -e PDNS_webserver_password=secret2 \
  -e PDNS_version_string=anonymous \
  -e PDNS_default_ttl=1500 \
  -e PDNS_allow_axfr_ips=172.5.0.21 \
  -e PDNS_only_notify=172.5.0.21 \
  tmuncks/pdns-server
```

Slave server with supermaster:

```bash
docker run -d -p 53:53 -p 53:53/udp --name pdns-slave \
  --hostname ns2.example.com --link mariadb:mysql \
  -e PDNS_gmysql_dbname=powerdnsslave \
  -e PDNS_slave=yes \
  -e PDNS_version_string=anonymous \
  -e PDNS_disable_axfr=yes \
  -e PDNS_allow_notify_from=172.5.0.20 \
  -e SUPERMASTER_IPS=172.5.0.20 \
  tmuncks/pdns-server
```
