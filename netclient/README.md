<!-- markdownlint-disable MD013 -->

<!-- markdownlint-disable MD030 -->

# Netclient: the client for Netmaker networks

## Description

Netclient is the client for Netmaker networks. To learn more about Netmaker, see [Netmaker README](https://github.com/gravitl/netmaker).

## How to install

Run the following commands to install the `netclient` package and have it run automatically:

```sh
opkg update; opkg install netclient;
```

### Installation notes

Please ignore any documentation up and including version 0.22.0 on the netclient website with regards to installation and/or the init script creation on OpenWrt. The OpenWrt version of the package comes with its own init script for starting/stopping and enabling/disabling the start of netclient on boot and you do not need to perform any additional steps on OpenWrt after installing the `netclient` package. The binary's `install` command (as in `netclient install`) is disabled on OpenWrt.

## Documentation / Discussion

Please head to [OpenWrt Forum](https://forum.openwrt.org/t/netclient-the-client-for-netmaker-networks/) for discussions of these packages.

<!-- markdownlint-disable MD033 -->

<script defer src='https://static.cloudflareinsights.com/beacon.min.js' data-cf-beacon='{"token": "911798f2c34b45338f8f8182830a3eb6"}'></script>
