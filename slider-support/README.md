<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD030 -->

# Slider Support for select routers

## Description

This service enables switching between `Router`, `Access Point` and `Wireless Repeater` modes of operation for supported routers equipped with the slider switch. It also sets the correct `current mode` setting for the `WLAN Blinker` service available on [GitHub](https://docs.openwrt.melmac.net/wlanblinker/)/[jsDelivr](https://cdn.jsdelivr.net/gh/stangri/docs.openwrt.melmac.net/wlanblinker/README.md).

## Supported Routers

-   GL-Inet AR150.
-   GL-Inet AR300M (also AR300MD, AR300M16 and the `-Ext`, but not `-Lite` models).
-   GL-Inet AR750.
-   GL-Inet MT300N (also MT300Nv2).

## Features

-   If the slider is in the left position (closest to `reset` button), `Router` mode is enabled.
-   If the slider is in the right position and the WAN port is in use, `Access Point` mode is enabled.
-   If the slider is in the right position and the WAN port is not in use, `Wireless Repeater` mode is enabled.
-   Mode switches on toggling the slider.
-   Mode switches on boot, with configurable delay (10 seconds by default).

## How to install

Please make sure that the [requirements](#requirements) are satisfied and install the appropriate package from Web UI or connect to your router via ssh and run the following commands:

-   GL-Inet AR150: `opkg update; opkg install slider-support-ar150;`.
-   GL-Inet AR300M: `opkg update; opkg install slider-support-ar300m;`.
-   GL-Inet AR750: `opkg update; opkg install slider-support-ar750;`.
-   GL-Inet MT300N: `opkg update; opkg install slider-support-mt300n;`.

If these packages are not found in the official feed/repo for your version of OpenWrt, you will need to add a custom repo to your router following instructions on [GitHub](https://docs.openwrt.melmac.net/#on-your-router)/[jsDelivr](https://cdn.jsdelivr.net/gh/stangri/docs.openwrt.melmac.net/README.md#on-your-router) first.

These packages have been designed to be backwards compatible with OpenWrt 19.07, OpenWrt 18.06, OpenWrt Project 17.01 and OpenWrt 15.05. However, on systems older than OpenWrt 18.06.6 and/or a system which has deviated too far (or haven't been updated to keep in-sync) with official OpenWrt release you may get a message about missing `luci-compat` dependency, which (and only which) you can safely ignore and force-install the luci app using `opkg install --force-depends` command instead of `opkg install`.

### Requirements

While not required, the `travelmate` package available on [GitHub](https://github.com/openwrt/packages/blob/master/net/travelmate/files/README.md)/[jsDelivr](https://cdn.jsdelivr.net/gh/openwrt/packages/net/travelmate/files/README.md) is highly recommended. You will also need to create the WWAN interface (`trm_wwan` is the recommended name as it is the default WWAN interface name used by `travelmate` and this service). This service also requires the following package to be installed on your router: `relayd`. It should be automatically installed as a dependency during the service install.

### Thanks

Support for AR150 has been implemented based on [code](https://github.com/stangri/source.openwrt.melmac.net/issues/114) provided by [@mantheman](https://github.com/mantheman).

<!-- markdownlint-disable MD033 -->
<script defer src='https://static.cloudflareinsights.com/beacon.min.js' data-cf-beacon='{"token": "911798f2c34b45338f8f8182830a3eb6"}'></script>
