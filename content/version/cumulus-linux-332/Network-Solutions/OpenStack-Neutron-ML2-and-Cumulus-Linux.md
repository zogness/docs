---
title: OpenStack Neutron ML2 and Cumulus Linux
author: Cumulus Networks
weight: 241
aliases:
 - /display/CL332/OpenStack+Neutron+ML2+and+Cumulus+Linux
 - /pages/viewpage.action?pageId=5869263
pageID: 5869263
product: Cumulus Linux
version: 3.3.2
imgData: cumulus-linux-332
siteSlug: cumulus-linux-332
---
{{%notice warning%}}

**Early Access Feature**

The REST API component is an [early access
feature](https://support.cumulusnetworks.com/hc/en-us/articles/202933878)
in Cumulus Linux 3.3. Before you can install the API, you must enable
the Early Access repository. For more information about the Cumulus
Linux repository, read [this knowledge base
article](https://support.cumulusnetworks.com/hc/en-us/articles/217422127).

{{%/notice%}}

The Modular Layer 2 (ML2) plugin is a framework that allows OpenStack
Networking to utilize a variety of non-vendor-specific layer 2
networking technologies. The ML2 framework simplifies adding support for
new layer 2 networking technologies, requiring much less initial and
ongoing effort — specifically, it enables dynamic provisioning of
VLAN/VXLAN on switches in OpenStack environment instead of manually
provisioning L2 connectivity for each VM.

The plugin supports configuration caching. The cached configuration is
replayed back to the Cumulus Linux switch from Cumulus ML2 mechanism
driver when a switch or process restart is detected.

In order to deploy [OpenStack
ML2](https://wiki.openstack.org/wiki/Neutron/ML2) in a network with
Cumulus Linux switches, you need to install two packages:

  - A REST API, which you install on your Cumulus Linux switches. It is
    in the Cumulus Linux [early
    access](https://support.cumulusnetworks.com/hc/en-us/articles/202933878)
    repository.

  - The Cumulus Networks Modular Layer 2 (ML2) mechanism driver for
    OpenStack, which you install on the OpenStack Neutron controller
    node. It's available as a Python package from upstream.

{{% imgOld 0 %}}

## Installing and Configuring the REST API</span>

To install the `python-falcon` and `python-cumulus-restapi` packages,
follow these instructions:

1.  Open the `/etc/apt/sources.list` file in a text editor.

2.  Uncomment the early access repository lines and save the file:
    
        #deb http://repo3.cumulusnetworks.com/repo CumulusLinux-3-early-access cumulus
        #deb-src http://repo3.cumulusnetworks.com/repo CumulusLinux-3-early-access cumulus

3.  Run the following commands in a terminal to install the early access
    packages:
    
        cumulus@switch:~$ sudo -E apt-get update
        cumulus@switch:~$ sudo -E apt-get install python-falcon python-cumulus-restapi

4.  Then configure the relevant settings in `/etc/restapi.conf`:
    
        [ML2]
        #local_bind = 10.40.10.122
        #service_node = 10.40.10.1
         
        # Add the list of inter switch links that
        # need to have the vlan included on it by default
        # Not needed if doing Hierarchical port binding
        #trunk_interfaces = uplink

5.  Restart the REST API service for the configuration changes to take
    effect:
    
        cumulus@switch:~$ sudo systemctl restart restserver

Additional REST API calls have been added to support the configuration
of bridge using the bridge name instead of network ID.

## Installing and Configuring the Cumulus Networks Modular Layer 2 Mechanism Driver</span>

You need to install the Cumulus Networks ML2 mechanism driver on your
Neutron host, which is available upstream:

    root@neutron:~# git clone https://github.com/openstack/networking-cumulus.git
    root@neutron:~# cd networking-cumulus
    root@neutron:~# python setup.py install
    root@neutron:~# neutron-db-manage upgrade head

Then configure the host to use the ML2 driver:

    root@neutron:~# openstack-config --set /etc/neutron/plugins/ml2/ml2_conf_cumulus.ini mechanism_drivers linuxbridge,cumulus

Finally, list the Cumulus Linux switches to configure. Edit
`/etc/neutron/plugins/ml2/ml2_conf_cumulus.ini` in a text editor and add
the IP addresses of the Cumulus Linux switches to the `switches` line.
For example:

    [ml2_cumulus]
    switches="192.168.10.10,192.168.20.20"

The ML2 mechanism driver contains the following configurable parameters.
You configure them in the
`/etc/neutron/plugins/ml2/ml2_conf_cumulus.ini` file.

  - `switches` — The list of Cumulus Linux switches connected to the
    Neutron host. Specify a list of IP addresses.

  - `scheme` — The scheme (for example, HTTP) for the base URL for the
    ML2 API.

  - `protocol_port` — The protocol port for the bast URL for the ML2
    API. The default value is *8000*.

  - `sync_time` — A periodic time interval for polling the Cumulus Linux
    switch. The default value is *30* seconds.

  - `spf_enable` — Enables/disables SPF for the bridge. The default
    value is *False*.

  - `new_bridge` — Enables/disables [VLAN-aware bridge
    mode](/version/cumulus-linux-332/Layer-One-and-Two/Ethernet-Bridging-VLANs/VLAN-aware-Bridge-Mode-for-Large-scale-Layer-2-Environments)
    for the bridge configuration. The default value is *False*, so a
    traditional mode bridge is created.

## Demo</span>

A demo involving OpenStack with Cumulus Linux is available in the
[Cumulus Networks knowledge
base](https://support.cumulusnetworks.com/hc/en-us/articles/226174767).
It demonstrates dynamic provisioning of VLANs using a virtual simulation
of two Cumulus VX leaf switches and two CentOS 7 (RDO Project) servers;
collectively they comprise an OpenStack environment.

<article id="html-search-results" class="ht-content" style="display: none;">

</article>

<footer id="ht-footer">

</footer>
