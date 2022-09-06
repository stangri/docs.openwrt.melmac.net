<!-- markdownlint-disable MD013 -->

<!-- markdownlint-disable MD030 -->

# VPN Bypass

## Statement about OpenWrt 22.03.0 release and this package

TLDR: Even tho this package depends on iptables/ipset and dnsmasq support for ipset, it works just fine with recently released OpenWrt 22.03.0. You can safely ignore the warning on the Status -> Firewall page about legacy iptables rules created by this package.

There are plans to transition `pbr` package to nftables when the nft sets support is integrated in OpenWrt's `dnsmasq` package. Until then, the transition is impossible since OpenWrt's `dnsmasq` only supports ipsets and it's a recommended and very frequently used feature to enforce proper policies for domains. Please note that both `vpn-policy-routing` and `vpnbypass` packages will not be transitioned to nftables and will become obsolete once OpenWrt's `dnsmasq` package no longer supports ipset.

## Description

A simple [procd](https://openwrt.org/docs/techref/procd)-based `vpnbypass` service for OpenWrt. This is useful if your router accesses Internet through a VPN client/tunnel, but you want specific traffic (ports, IP ranges, domains or local IP ranges) to be routed outside of this tunnel.

## Features

-   Define local ports so traffic to them is routed outside of the VPN tunnel (by default it routes Plex Media Server traffic (port 32400) outside of the VPN tunnel).
-   Define IPs/subnets in local networks so their traffic is routed outside the VPN tunnel (by default it routes traffic from 192.168.1.81-192.168.1.87 outside the VPN tunnel).
-   Define remote IP ranges that are accessed outside the VPN tunnel (by default, LogmeIn Hamachi traffic (25.0.0.0/8) is routed outside the VPN tunnel).
-   Creates list of domain names which should be accessed outside the VPN tunnel (useful for Netflix, Hulu, etc).
-   Does not reside in RAM -- creates `iptables` rules which are automatically updated on WAN up/down events.
-   A companion package (`luci-app-vpnbypass`) is provided so all features may be configured from the Web UI.
-   Proudly made in Canada, using locally-sourced electrons.

## Screenshot (luci-app-vpnbypass)

![screenshot](https://docs.openwrt.melmac.net/vpnbypass/screenshots/screenshot02.png "screenshot")

## Requirements

This service requires the following packages to be installed on your router: `ipset` and `iptables`. Additionally, if you want to use the Domain Bypass feature, you need to install `dnsmasq-full` (`dnsmasq-full` requires you uninstall `dnsmasq` first).

To fully satisfy the requirements for both IP/Port VPN Bypass and Domain Bypass features connect via ssh to your router and run the following commands:

```sh
opkg update; cd /tmp/ && opkg download dnsmasq-full; opkg install ipset iptables libnettle8 libnetfilter-conntrack3;
opkg remove dnsmasq; opkg install dnsmasq-full --cache /tmp/; rm -f /tmp/dnsmasq-full*.ipk;
```

To satisfy the requirements for just IP/Port VPN Bypass connect to your router via ssh and run the following commands:

```sh
opkg update; opkg install ipset iptables
```

### Unmet dependencies

If you are running a development (trunk/snapshot) build of OpenWrt on your router and your build is outdated (meaning that packages of the same revision/commit hash are no longer available and when you try to satisfy the [requirements](#requirements) you get errors), please flash either current OpenWrt release image or current development/snapshot image.

## How to install

<!---
#### From Web UI/Luci
Navigate to System->Software page on your router and then perform the following actions:
1. Click "Update Lists"
2. Wait for the update process to finish.
3. In the "Download and install package:" field type ```vpnbypass luci-app-vpnbypass```
4. Click "OK" to install ```vpnbypass``` and ```luci-app-vpnbypass```

If you get an ```Unknown package 'vpnbypass'``` error, your router is not set up with the access to a repository containing these packages and you need to add the custom repository to your router first.

#### From console/ssh
--->

Please ensure that the [requirements](#requirements) are satisfied and install `vpnbypass` and `luci-app-vpnbypass` from the Web UI or connect to your router via ssh and run the following commands:

```sh
opkg update
opkg install vpnbypass luci-app-vpnbypass
```

If these packages are not found in the official feed/repo for your version of OpenWrt, you will need to add a custom repo to your router following instructions on [GitHub](https://docs.openwrt.melmac.net/#on-your-router)/[jsDelivr](https://cdn.jsdelivr.net/gh/stangri/docs.openwrt.melmac.net/README.md#on-your-router) first.

These packages have been designed to be backwards compatible with OpenWrt 19.07, OpenWrt 18.06, OpenWrt Project 17.01 and OpenWrt 15.05. However, on systems older than OpenWrt 18.06.6 and/or a system which has deviated too far (or haven't been updated to keep in-sync) with official OpenWrt release you may get a message about missing `luci-compat` dependency, which (and only which) you can safely ignore and force-install the luci app using `opkg install --force-depends` command instead of `opkg install`.

## Default Settings

The default configuration ships with the service disabled, use the Web UI to enable/start the service or run `uci set vpnbypass.config.enabled=1; uci commit vpnbypass;`. It routes Plex Media Server traffic (port 32400) and LogmeIn Hamachi traffic (25.0.0.0/8) outside of the VPN tunnel. Internet traffic from local IPs `192.168.1.81-192.168.1.87` is also routed outside the VPN tunnel. You can safely delete these example rules if they do not apply to you.

## Documentation / Discussion

Please head to [OpenWrt Forum](https://forum.openwrt.org/t/vpn-bypass-split-tunneling-service-luci-ui/1106) for discussions of this service.

### Bypass Domains Format/Syntax

Domain lists should be in the following format/syntax: `/domain1.com/domain2.com/vpnbypass`. Please do not forget the leading `/` and trailing `/vpnbypass`. There is no validation if you enter something incorrectly -- it simply will not work. Please see [Notes/Known Issues](#notesknown-issues) if you wish to edit this setting manually, without using the Web UI.

## What's New

1.3.0:

-   No longer depends on hardcoded WAN interface name (`wan`) works with other interface names (like `wwan`).
-   Table ID, IPSET name and FW_MARK as well as FW_MASK can be defined in config file.
-   Uses iptables, not ip rules for handling local IPs/ranges.
-   More reliable creation/destruction of VPNBYPASS iptables chain.
-   Updated Web UI enables, starts and stops the service.

## Notes/Known Issues

1.  Domains to be accessed outside of VPN tunnel are handled by dnsmasq and thus are not defined in `/etc/config/vpnpass`, but rather in `/etc/config/dhcp`. To add/delete/edit domains you can use VPN Bypass Web UI or you can edit `/etc/config/dhcp` manually or run the following commands:

```sh
uci add_list dhcp.@dnsmasq[-1].ipset='/github.com/plex.tv/google.com/vpnbypass'
uci add_list dhcp.@dnsmasq[-1].ipset='/hulu.com/netflix.com/nhl.com/vpnbypass'
uci commit dhcp
/etc/init.d/dnsmasq restart
```

This feature requires `dnsmasq-full` to work. See the [Requirements](#requirements) section for more details.

<!-- markdownlint-disable MD033 -->

<script defer src='https://static.cloudflareinsights.com/beacon.min.js' data-cf-beacon='{"token": "911798f2c34b45338f8f8182830a3eb6"}'></script>

# Thanks

I'd like to thank everyone who helped create, test and troubleshoot this package. I would also like to specifically thank: [T81](https://github.com/T81) for thorough testing and assistance with bugfixing and [vsviridov](https://github.com/vsviridov) and [jow](https://github.com/jow-) for their invaluable contributions in migrating the WebUI for this package to client-side rendering.
