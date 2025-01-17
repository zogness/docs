---
title: HTTP API
author: Cumulus Networks
weight: 79
aliases:
 - /display/CL34/HTTP+API
 - /pages/viewpage.action?pageId=7112348
pageID: 7112348
product: Cumulus Linux
version: 3.4.3
imgData: cumulus-linux-343
siteSlug: cumulus-linux-343
---
Cumulus Linux 3.4 implements an HTTP application programing interface to
[OpenStack ML2
driver](/version/cumulus-linux-343/Network-Solutions/OpenStack-Neutron-ML2-and-Cumulus-Linux)
and
[NCLU](/version/cumulus-linux-343/System-Configuration/Network-Command-Line-Utility-NCLU).
Rather than accessing a Cumulus Linux host with SSH, you can interact
with a server with a HTTP client, such as cURL, HTTPie or a web browser.

{{%notice note%}}

The HTTP API service is enabled by default on chassis hardware only.
However, the associated server is configured to only listen to traffic
originating from within the chassis.

The service is not enabled by default on non-chassis hardware.

{{%/notice%}}

### Getting Started</span>

{{%notice note%}}

If upgrading from a version of Cumulus Linux before v3.4.0 the
supporting software for the API may not be installed. Install the
required software with the following command.

    cumulus@switch:~$ sudo apt-get install python-cumulus-restapi

{{%/notice%}}

To enable the HTTP API service, run the following `systemd` commands:

    cumulus@switch:~$ sudo systemctl enable restserver-gunicorn
    cumulus@switch:~$ sudo systemctl enable restserver

Use the `systemctl start` and `systemctl stop` commands to start/stop
the HTTP API service:

    cumulus@switch:~$ sudo systemctl start restserver
    cumulus@switch:~$ sudo systemctl stop restserver

{{%notice note%}}

Each service runs as a background daemon once started.

{{%/notice%}}

### Configuration</span>

There are two configuration files associated with the HTTP API services:

  - `/etc/nginx-restapi.conf`

  - `/etc/nginx-restapi-chassis.conf`

The first configuration file is used for non-chassis hardware; the
second, for chassis hardware.

Generally, only the configuration file relevant to your hardware needs
to be edited, as the associated services determine the appropriate
configuration file to use at run time.

#### Enable External Traffic on a Chassis</span>

The HTTP API services are configured to listen on port 8080 for chassis
hardware by default. However, only HTTP traffic originating from
internal link local management IPv6s will be allowed. To configure the
services to also accept HTTP requests originating from external sources:

1.  Open `/etc/nginx-restapi-chassis.conf` in a text editor.

2.  Uncomment the following line near the beginning of the file:
    
        #user root;

3.  Uncomment the `server` block lines near the end of the file.

4.  Change the port on the now uncommented `listen` line if the default
    value, 8088, is not the preferred port, and save the configuration
    file.

5.  Verify the configuration file is still valid:
    
        cumulus@switch:~$ sudo nginx -c /etc/nginx-restapi-chassis.conf -t
    
    If the configuration file is not valid, return to step 1; review any
    changes that were made, and correct the errors.

6.  Restart the daemons:
    
        cumulus@switch:~$ sudo systemctl restart restserver

#### IP and Port Settings</span>

The IP:port combinations that services listen to can be modified by
changing the parameters of the `listen` directive(s). By default,
`nginx-restapi.conf` has only one `listen` parameter, whereas
`/etc/nginx-restapi-chassis.conf` has two independently configurable
`server` blocks, each with a `listen` directive. One server block is for
external traffic, and the other for internal traffic.

{{%notice note%}}

All URLs must use HTTPS, rather than HTTP.

{{%/notice%}}

For more information on the listen directive, refer to the [NGINX
documentation](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen).

{{%notice warning%}}

Do not set the same listening port for internal and external chassis
traffic.

{{%/notice%}}

### <span id="src-7112348_HTTPAPI-security" class="confluence-anchor-link"></span>Security</span>

#### Authentication</span>

The default configuration requires all HTTP requests from external
sources (not internal switch traffic) to set the HTTP Basic
Authentication header.

The user and password should correspond to a user on the host switch.

#### Transport Layer Security</span>

All traffic must be secured in transport using TLSv1.2 by default.
Cumulus Linux contains a self-signed certificate and private key used
server-side in this application so that it works out of the box, but
Cumulus Networks recommends you use your own certificates and keys.
Certificates must be in the PEM format.

For step by step documentation for generating self-signed certificates
and keys, and installing them to the switch, refer to the [Ubuntu
Certificates and Security
documentation](https://help.ubuntu.com/lts/serverguide/certificates-and-security.html).

{{%notice warning%}}

Do not copy the `cumulus.pem` or `cumulus.key` files. After
installation, edit the “ssl\_certificate” and “ssl\_certificate\_key”
values in the configuration file for your hardware.

{{%/notice%}}

### cURL Examples</span>

This section contains several example cURL commands for sending HTTP
requests to a non-chassis host. The following settings are used for
these examples:

  - Username: `user`

  - Password: `pw`

  - IP: `1.1.1.1`

  - Port: `8080`

{{%notice note%}}

Requests for NCLU require setting the Content-Type request header to be
set to `application/json`.

{{%/notice%}}

{{%notice info%}}

cURL’s `-k` flag is necessary when the server uses a self-signed
certificate. This is the default configuration (see the [Security
section](#src-7112348_HTTPAPI-security)). To display the response
headers, include `-D` flag in the command.

{{%/notice%}}

To retrieve a list of all available HTTP endpoints:

    cumulus@switch:~$ curl -X GET -k -u user:pw https://1.1.1.1:8080

To run `net show counters` on the host as a remote procedure call:

    cumulus@switch:~$ curl -X POST -k -u user:pw -H "Content-Type: application/json" -d '{"cmd": "show counters"}' https://1.1.1.1:8080/nclu/v1/rpc

To add a bridge using ML2:

    cumulus@switch:~$ curl -X PUT -k -u user:pw https://1.1.1.1:8080/ml2/v1/bridge/"br1"/200

<article id="html-search-results" class="ht-content" style="display: none;">

</article>

<footer id="ht-footer">

</footer>
