<!-- markdownlint-disable MD013 -->

<!-- markdownlint-disable MD030 -->

# SlackHQ's Nebula

## Description

Nebula is a scalable overlay networking tool with a focus on performance, simplicity and security. It lets you seamlessly connect computers anywhere in the world. [Nebula is being developed by Slack](https://github.com/slackhq/nebula).

## Packages

OpenWrt repositories contain following packages:

-   `nebula`: This is the principal binary package. This package is required for nebula peer or lighthouse operations. Unless you want to start nebula manually, you may want to also install _either_ `nebula-service` _or_ `nebula-proto` package.
-   `nebula-cert`: This package contains only `nebula-cert` binary required to generate certificates not _not_ necessary for the nebula peer or lighthouse operations.
-   `nebula-proto`: This package contains only OpenWrt protocol/interface support for nebula. You will need to create a new interface for nebula node/lighthouse if you want to use this package.
-   `nebula-service`: This package contains only OpenWrt-specific init.d script for nebula. This package starts a node/lighthouse for each `.yml` config file it finds in `/etc/nebula/` directory.

## Requirements

-   `nebula`: The principal package requires (auto-installs) the following package: `kmod-tun` and its dependencies.
-   `nebula-cert`: This package has no dependencies/requirements and can be installed stand-alone.
-   `nebula-proto`: This package requires the following package: `nebula` and its dependencies.
-   `nebula-service`: This package requires the following package: `nebula` and its dependencies.

### Unmet dependencies

If you are running a development (trunk/snapshot) build of OpenWrt on your router and your build is outdated (meaning that packages of the same revision/commit hash are no longer available and when you try to satisfy the [requirements](#requirements) you get errors), please flash either current OpenWrt release image or current development/snapshot image.

## How to install

If you want to run the nebula binary manually, you will need to install just the `nebula` package and it will auto-install all dependencies (`kmod-tun` and its dependencies):

```sh
opkg update; opkg install nebula;
```

If you want to manage the certificates, you will need to install just the `nebula-cert` package:

```sh
opkg update; opkg install nebula-cert;
```

If you want to create a manage a new protocol/interface for the nebula, you will need to install the `nebula-proto` package and it will auto-install all dependencies (`nebula`, `kmod-tun` and its dependencies):

```sh
opkg update; opkg install nebula-proto;
```

If, in addition to the `nebula-proto`, you also want to install luci/WebUI support for nebula protocol/interface, you will need to install the `luci-proto-nebula` package and it will auto-install all dependencies (`nebula-proto`, `nebula`, `kmod-tun` and its dependencies):

```sh
opkg update; opkg install luci-proto-nebula;
```

If you want to have nebula as a service on your router (with the `init.d` script), you will need to install the `nebula-service` package and it will auto-install all dependencies (`nebula`, `kmod-tun` and its dependencies):

```sh
opkg update; opkg install nebula-service;
```

## Default Settings

-   `nebula`: This package installs just the principal binary and doesn't have any settings/actions.
-   `nebula-cert`: This package installs just the nebula certification binary and doesn't have any settings/actions.
-   `nebula-proto`: This package allows you to create a new interface with the nebula protocol pointing to the `.yml` config file. Here's an example of the `/etc/config/network` section:

```text
config interface 'nebula1'
        option proto 'nebula'
        option config '/etc/nebula/config1.yml'
```

   When the nebula interface is brough up, it will automatically open the UDP port referenced in the `.yml` config file in the router's firewall.

-   `nebula-service`: This package contains and `init.d`/service script which scans the `/etc/nebula/` directory for `.yml` config files and creates a nebula node/lighthouse for each located `.yml` config file. When each node/lighthouse is started, it will automatically open the UDP port referenced in the `.yml` config file in the router's firewall.

## Documentation / Discussion

Please head to [OpenWrt Forum](https://forum.openwrt.org/t/slacks-nebula-on-openwrt-discussion-thread/85431) for discussions of these packages.

<!-- markdownlint-disable MD033 -->

<script defer src='https://static.cloudflareinsights.com/beacon.min.js' data-cf-beacon='{"token": "911798f2c34b45338f8f8182830a3eb6"}'></script>
