# Docker Pi-hole

<p align="center">
<a href="https://pi-hole.net"><img src="https://pi-hole.github.io/graphics/Vortex/Vortex_with_text.png" width="150" height="255" alt="Pi-hole"></a><br/>
</p>
<!-- Delete above HTML and insert markdown for dockerhub : ![Pi-hole](https://pi-hole.github.io/graphics/Vortex/Vortex_with_text.png) -->

## Quick Start

1. Copy docker-compose example to PI and update as needed.

2. Run `docker-compose up -d` to build and start pi-hole

## Overview

**Automatic Ad List Updates** - since the 3.0+ release, `cron` is baked into the container and will grab the newest versions of your lists and flush your logs.  **Set your TZ** environment variable to make sure the midnight log rotation syncs up with your timezone's midnight.


## Environment Variables

There are other environment variables if you want to customize various things inside the docker container:

### Recommended Variables

| Variable | Default | Value | Description |
| -------- | ------- | ----- | ---------- |
| `TZ` | UTC | `<Timezone>` | Set your [timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) to make sure logs rotate at local midnight instead of at UTC midnight.
| `WEBPASSWORD` | random | `<Admin password>` | http://pi.hole/admin password. Run `docker logs pihole \| grep random` to find your random pass.
| `FTLCONF_REPLY_ADDR4` | unset | `<Host's IP>` | Set to your server's LAN IP, used by web block modes and lighttpd bind address.

### Optional Variables

| Variable | Default | Value | Description |
| -------- | ------- | ----- | ---------- |
| `ADMIN_EMAIL` | unset | email address | Set an administrative contact address for the Block Page |
| `PIHOLE_DNS_` |  `8.8.8.8;8.8.4.4` | IPs delimited by `;` | Upstream DNS server(s) for Pi-hole to forward queries to, separated by a semicolon <br/> (supports non-standard ports with `#[port number]`) e.g `127.0.0.1#5053;8.8.8.8;8.8.4.4` <br/> (supports [Docker service names and links](https://docs.docker.com/compose/networking/) instead of IPs) e.g `upstream0;upstream1` where `upstream0` and `upstream1` are the service names of or links to docker services <br/> Note: The existence of this environment variable assumes this as the _sole_ management of upstream DNS. Upstream DNS added via the web interface will be overwritten on container restart/recreation |
| `DNSSEC` | `false` | `<"true"\|"false">` | Enable DNSSEC support |
| `DNS_BOGUS_PRIV` | `true` |`<"true"\|"false">`| Never forward reverse lookups for private ranges |
| `DNS_FQDN_REQUIRED` | `true` | `<"true"\|"false">`| Never forward non-FQDNs |
| `REV_SERVER` | `false` | `<"true"\|"false">` | Enable DNS conditional forwarding for device name resolution |
| `REV_SERVER_DOMAIN` | unset | Network Domain | If conditional forwarding is enabled, set the domain of the local network router |
| `REV_SERVER_TARGET` | unset | Router's IP | If conditional forwarding is enabled, set the IP of the local network router |
| `REV_SERVER_CIDR` | unset | Reverse DNS | If conditional forwarding is enabled, set the reverse DNS zone (e.g. `192.168.0.0/24`) |
| `DHCP_ACTIVE` | `false` | `<"true"\|"false">` | Enable DHCP server. Static DHCP leases can be configured with a custom `/etc/dnsmasq.d/04-pihole-static-dhcp.conf`
| `DHCP_START` | unset | `<Start IP>` | Start of the range of IP addresses to hand out by the DHCP server (mandatory if DHCP server is enabled).
| `DHCP_END` | unset | `<End IP>` | End of the range of IP addresses to hand out by the DHCP server (mandatory if DHCP server is enabled).
| `DHCP_ROUTER` | unset | `<Router's IP>` | Router (gateway) IP address sent by the DHCP server (mandatory if DHCP server is enabled).
| `DHCP_LEASETIME` | 24 | `<hours>` | DHCP lease time in hours.
| `PIHOLE_DOMAIN` | `lan` | `<domain>` | Domain name sent by the DHCP server.
| `DHCP_IPv6` | `false` | `<"true"\|"false">` | Enable DHCP server IPv6 support (SLAAC + RA).
| `DHCP_rapid_commit` | `false` | `<"true"\|"false">` | Enable DHCPv4 rapid commit (fast address assignment).
| `VIRTUAL_HOST` | `$FTLCONF_REPLY_ADDR4` | `<Custom Hostname>` | What your web server 'virtual host' is, accessing admin through this Hostname/IP allows you to make changes to the whitelist / blacklists in addition to the default 'http://pi.hole/admin/' address
| `IPv6` | `true` | `<"true"\|"false">` | For unraid compatibility, strips out all the IPv6 configuration from DNS/Web services when false.
| `TEMPERATUREUNIT` | `c` | `<c\|k\|f>` | Set preferred temperature unit to `c`: Celsius, `k`: Kelvin, or `f` Fahrenheit units.
| `WEBUIBOXEDLAYOUT` | `boxed` | `<boxed\|traditional>` | Use boxed layout (helpful when working on large screens)
| `QUERY_LOGGING` | `true` | `<"true"\|"false">` | Enable query logging or not.
| `WEBTHEME` | `default-light` | `<"default-dark"\|"default-darker"\|"default-light"\|"default-auto"\|"lcars">`| User interface theme to use.
| `WEBPASSWORD_FILE`| unset | `<Docker secret path>` |Set an Admin password using [Docker secrets](https://docs.docker.com/engine/swarm/secrets/). If `WEBPASSWORD` is set, `WEBPASSWORD_FILE` is ignored. If `WEBPASSWORD` is empty, and `WEBPASSWORD_FILE` is set to a valid readable file path, then `WEBPASSWORD` will be set to the contents of `WEBPASSWORD_FILE`.

### Advanced Variables
| Variable | Default | Value | Description |
| -------- | ------- | ----- | ---------- |
| `INTERFACE` | unset | `<NIC>` | The default works fine with our basic example docker run commands.  If you're trying to use DHCP with `--net host` mode then you may have to customize this or DNSMASQ_LISTENING.
| `DNSMASQ_LISTENING` | unset | `<local\|all\|single>` | `local` listens on all local subnets, `all` permits listening on internet origin subnets in addition to local, `single` listens only on the interface specified.
| `WEB_PORT` | unset | `<PORT>` | **This will break the 'webpage blocked' functionality of Pi-hole** however it may help advanced setups like those running synology or `--net=host` docker argument.  This guide explains how to restore webpage blocked functionality using a linux router DNAT rule: [Alternative Synology installation method](https://discourse.pi-hole.net/t/alternative-synology-installation-method/5454?u=diginc)
| `SKIPGRAVITYONBOOT` | unset | `<unset\|1>` | Use this option to skip updating the Gravity Database when booting up the container.  By default this environment variable is not set so the Gravity Database will be updated when the container starts up.  Setting this environment variable to 1 (or anything) will cause the Gravity Database to not be updated when container starts up.
| `CORS_HOSTS` | unset | `<FQDNs delimited by ,>` | List of domains/subdomains on which CORS is allowed. Wildcards are not supported. Eg: `CORS_HOSTS: domain.com,home.domain.com,www.domain.com`.
| `CUSTOM_CACHE_SIZE` | `10000` | Number | Set the cache size for dnsmasq. Useful for increasing the default cache size or to set it to 0. Note that when `DNSSEC` is "true", then this setting is ignored.
| `FTL_CMD` | `no-daemon` | `no-daemon -- <dnsmasq option>` | Customize the options with which dnsmasq gets started. e.g. `no-daemon -- --dns-forward-max 300` to increase max. number of concurrent dns queries on high load setups. |
| `FTLCONF_[SETTING]` | unset | As per documentation | Customize pihole-FTL.conf with settings described in the [FTLDNS Configuration page](https://docs.pi-hole.net/ftldns/configfile/). For example, to customize REPLY_ADDR6, ensure you have the `FTLCONF_REPLY_ADDR6` environment variable set.

### Experimental Variables
| Variable | Default | Value | Description |
| -------- | ------- | ----- | ---------- |
| `DNSMASQ_USER` | unset | `<pihole\|root>` | Allows changing the user that FTLDNS runs as. Default: `pihole`|
| `PIHOLE_UID` | debian system value | Number | Overrides image's default pihole user id to match a host user id  |
| `PIHOLE_GID` | debian system value | Number | Overrides image's default pihole group id to match a host group id |
| `WEB_UID` | debian system value | Number | Overrides image's default www-data user id to match a host user id |
| `WEB_GID` | debian system value | Number | Overrides image's default www-data group id to match a host group id |
| `WEBLOGS_STDOUT` | 0 | 0&vert;1 | 0 logs to defined files, 1 redirect access and error logs to stdout |

## Deprecated environment variables:
While these may still work, they are likely to be removed in a future version. Where applicable, alternative variable names are indicated. Please review the table above for usage of the alternative variables

| Docker Environment Var. | Description | Replaced By |
| ----------------------- | ----------- | ----------- |
| `CONDITIONAL_FORWARDING` | Enable DNS conditional forwarding for device name resolution | `REV_SERVER`|
| `CONDITIONAL_FORWARDING_IP` | If conditional forwarding is enabled, set the IP of the local network router | `REV_SERVER_TARGET` |
| `CONDITIONAL_FORWARDING_DOMAIN` | If conditional forwarding is enabled, set the domain of the local network router | `REV_SERVER_DOMAIN` |
| `CONDITIONAL_FORWARDING_REVERSE` | If conditional forwarding is enabled, set the reverse DNS of the local network router (e.g. `0.168.192.in-addr.arpa`) | `REV_SERVER_CIDR` |
| `DNS1` | Primary upstream DNS provider, default is google DNS | `PIHOLE_DNS_` |
| `DNS2` | Secondary upstream DNS provider, default is google DNS, `no` if only one DNS should used | `PIHOLE_DNS_` |
| `ServerIP` | Set to your server's LAN IP, used by web block modes and lighttpd bind address | `FTLCONF_REPLY_ADDR4` |
| `ServerIPv6` | **If you have a v6 network** set to your server's LAN IPv6 to block IPv6 ads fully | `FTLCONF_REPLY_ADDR6` |

To use these env vars in docker run format style them like: `-e DNS1=1.1.1.1`

Here is a rundown of other arguments for your docker-compose / docker run.

| Docker Arguments | Description |
| ---------------- | ----------- |
| `-p <port>:<port>` **Recommended** | Ports to expose (53, 80, 67), the bare minimum ports required for Pi-holes HTTP and DNS services
| `--restart=unless-stopped`<br/> **Recommended** | Automatically (re)start your Pi-hole on boot or in the event of a crash
| `-v $(pwd)/etc-pihole:/etc/pihole`<br/> **Recommended** | Volumes for your Pi-hole configs help persist changes across docker image updates
| `-v $(pwd)/etc-dnsmasq.d:/etc/dnsmasq.d`<br/> **Recommended** | Volumes for your dnsmasq configs help persist changes across docker image updates
| `--net=host`<br/> *Optional* | Alternative to `-p <port>:<port>` arguments (Cannot be used at same time as -p) if you don't run any other web application. DHCP runs best with --net=host, otherwise your router must support dhcp-relay settings.
| `--cap-add=NET_ADMIN`<br/> *Recommended* | Commonly added capability for DHCP, see [Note on Capabilities](#note-on-capabilities) below for other capabilities.
| `--dns=127.0.0.1`<br/> *Optional* | Sets your container's resolve settings to localhost so it can resolve DHCP hostnames from Pi-hole's DNSMasq, may fix resolution errors on container restart.
| `--dns=1.1.1.1`<br/> *Optional* | Sets a backup server of your choosing in case DNSMasq has problems starting
| `--env-file .env` <br/> *Optional* | File to store environment variables for docker replacing `-e key=value` settings. Here for convenience

## Tips and Tricks

* A good way to test things are working right is by loading this page: [http://pi.hole/admin/](http://pi.hole/admin/)
* [How do I set or reset the Web interface Password?](https://discourse.pi-hole.net/t/how-do-i-set-or-reset-the-web-interface-password/1328)
  * `docker exec -it pihole_container_name pihole -a -p` - then enter your password into the prompt
* Port conflicts?  Stop your server's existing DNS / Web services.
  * Don't forget to stop your services from auto-starting again after you reboot
  * Ubuntu users see below for more detailed information
* You can map other ports to Pi-hole port 80 using docker's port forwarding like this `-p 8080:80` if you are using the default blocking mode. If you are using the legacy IP blocking mode, you should not remap this port.
  * [Here is an example of running with nginxproxy/nginx-proxy](https://github.com/pi-hole/docker-pi-hole/blob/master/examples/docker-compose-nginx-proxy.yml) (an nginx auto-configuring docker reverse proxy for docker) on my port 80 with Pi-hole on another port.  Pi-hole needs to be `DEFAULT_HOST` env in nginxproxy/nginx-proxy and you need to set the matching `VIRTUAL_HOST` for the Pi-hole's container.  Please read nginxproxy/nginx-proxy readme for more info if you have trouble.
* Docker's default network mode `bridge` isolates the container from the host's network. This is a more secure setting, but requires setting the Pi-hole DNS option for *Interface listening behavior* to "Listen on all interfaces, permit all origins".

### Pi-hole features

Here are some relevant wiki pages from [Pi-hole's documentation](https://github.com/pi-hole/pi-hole/blob/master/README.md#get-help-or-connect-with-us-on-the-web).  The web interface or command line tools can be used to implement changes to pihole.

We install all pihole utilities so the the built in [pihole commands](https://discourse.pi-hole.net/t/the-pihole-command-with-examples/738) will work via `docker exec <container> <command>` like so:

* `docker exec pihole_container_name pihole updateGravity`
* `docker exec pihole_container_name pihole -w spclient.wg.spotify.com`
* `docker exec pihole_container_name pihole -wild example.com`
