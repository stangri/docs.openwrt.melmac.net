<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD030 -->
<!-- markdownlint-disable MD033 -->

# AdBlock-Fast

## Description

Fast and lean Dnsmasq/SmartDNS/Unbound-based AdBlocking service for OpenWrt. Featuring simultaneous remote allow/block-list downloading/processing, fast parsing code and simple WebUI allowing to easily add custom/user-specific remote lists.

## Features

- Super-fast due to the nature of supported allow/block-lists, parallel downloading/processing of the block-lists and fast block-list parsing code.
- Supports [multiple remote allow/block-lists formats](#recognized-remote-allowblock-list-file-formats), as well as using a [remote dnsmasq config file](#dnsmasq_config_file_url) with no parsing.
- Everything is configurable from Web UI.
- Allows you to easily add your own domains to allow-list or block-list.
- Allows you to easily add URLs to your own blocked hosts or domains lists to allow/block-list (just put allowed domains one per line in the file you're linking).
- Supports multiple modes of AdBlocking implementations with Dnsmasq, SmartDNS and Unbound.
- Doesn't stay in memory -- creates the list of blocked domains and then uses Dnsmasq/SmartDNS/Unbound and firewall rules to serve NXDOMAIN or 127.0.0.1 reply or to reject access (depending on settings) for blocked domains.
- As some of the default lists are using https, reliably works with either wget/libopenssl, uclient-fetch/libustream-mbedtls or curl (recommended).
- Very lightweight and easily hackable, the whole script is just one `/etc/init.d/adblock-fast` file.
- Retains the downloaded/sorted AdBlocking list on service stop and reuses it on service start (use `dl` command if you want to force re-download of the list).
- Has an option to store a compressed copy of the AdBlocking list in persistent memory which survives reboots.
- Blocks ads served over https (unlike PixelServ-derived solutions).
- Blocks ads inside browsers with built-in [DNS-over-HTTPS proxy](https://en.wikipedia.org/wiki/DNS_over_HTTPS) enabled, like [Mozilla Firefox](https://support.mozilla.org/en-US/kb/firefox-dns-over-https#w_about-dns-over-https) or [Google Chrome/Chromium](https://blog.chromium.org/2019/09/experimenting-with-same-provider-dns.html) -- with the `dnsmasq.ipset` or `dnsmasq.nftset` options (where supported by installed Dnsmasq).
- Proudly made in Canada, using locally-sourced electrons.

If you want an AdBlocking using pre-selected curated remote block-lists, supporting Internationalized Domain Names and richer WebUI, check out `net/adblock` on [GitHub](https://github.com/openwrt/packages/tree/master/net/adblock/files)/[jsDelivr](https://cdn.jsdelivr.net/gh/openwrt/packages/net/adblock/files/README.md).

## Screenshots (luci-app-adblock-fast)

Service Status

![screenshot](https://docs.openwrt.melmac.net/adblock-fast/screenshots/screenshot10-status.png "Service Status")

Configuration - Basic Configuration

![screenshot](https://docs.openwrt.melmac.net/adblock-fast/screenshots/screenshot10-config-basic.png "Configuration - Basic Configuration")

Configuration - Advanced Configuration

![screenshot](https://docs.openwrt.melmac.net/adblock-fast/screenshots/screenshot10-config-advanced.png "Configuration - Advanced Configuration")

Allowed and Blocked Domains Management

![screenshot](https://docs.openwrt.melmac.net/adblock-fast/screenshots/screenshot10-domains.png "Allowed and Blocked Domains Management")

Allowed and Blocked Lists Management

![screenshot](https://docs.openwrt.melmac.net/adblock-fast/screenshots/screenshot10-lists.png "Allowed and Blocked Lists Management")

## Requirements

The required packages should be automatically installed when you install `adblock-fast`.

### Requirements for DNS Resolver

In order to actually block the ads, this service requires one of the following DNS resolvers to be installed on your router: `dnsmasq`, `dnsmasq-full`, `smartdns` or `unbound`. The `dnsmasq` package is usually included in the default OpenWrt installation.

If you want to use the [`dnsmasq.ipset` option](#dns-resolver-option) you need to install `ipset` and `dnsmasq-full` with `ipset` support instead of the `dnsmasq`.

If you want to use the [`dnsmasq.nftset` option](#dns-resolver-option) you need to install `dnsmasq-full` with `nftset` support instead of the `dnsmasq`.

By default, `dnsmasq-full` in OpenWrt 22.03 and earlier supports only `ipset`, the `dnsmasq-full` in OpenWrt 23.05 and later supports only `nftset`.

To upgrade from `dnsmasq` to `dnsmasq-full`, connect to your router via ssh and run the following command:

```sh
opkg update; cd /tmp/ && opkg download dnsmasq-full; opkg install ipset libnettle8 libnetfilter-conntrack3;
opkg remove dnsmasq; opkg install dnsmasq-full --cache /tmp/; rm -f /tmp/dnsmasq-full*.ipk;
```

### Requirements for IPv6 Support

For IPv6 support additionally install `ip6tables-mod-nat` and `kmod-ipt-nat6` packages from Web UI or run the following in the command line:

```sh
opkg update; opkg install ip6tables-mod-nat kmod-ipt-nat6;
```

### Requirements for Faster Block-list Processing

The `gawk`, `grep`, `sed` and `coreutils-sort` are the optional, but recommended packages as they speed up processing dramatically. You can install these recommended packages by running the following command:

```sh
opkg --force-overwrite install gawk grep sed coreutils-sort
```

## Unmet Dependencies

If you are running a development (snapshot) build of OpenWrt on your router and your build is outdated (meaning that packages of the same revision/commit hash are no longer available and when you try to satisfy the [requirements](#requirements) you get errors), please flash either current OpenWrt release image or current development/snapshot image.

## How To Install

Install `adblock-fast` and `luci-app-adblock-fast` packages from Web UI or run the following in the command line:

```sh
opkg update; opkg install adblock-fast luci-app-adblock-fast
```

If `adblock-fast` and `luci-app-adblock-fast` packages are not found in the official feed/repo for your version of OpenWrt, you will need to add a custom repo to your router following instructions on [GitHub](https://docs.openwrt.melmac.net/#on-your-router)/[jsDelivr](https://cdn.jsdelivr.net/gh/stangri/docs.openwrt.melmac.net/README.md#on-your-router) first.

## Default Settings

Default configuration has the service disabled (use Web UI to enable/start service or run `uci set adblock-fast.config.enabled=1; uci commit adblock-fast;`) and selected ad/malware lists suitable for routers with 64Mb RAM.

If your router has less then 64Mb RAM, edit the configuration file, located at `/etc/config/adblock-fast`. The configuration file has lists in ascending order starting with smallest ones and each list has a preceding comment indicating its size, comment out or delete the lists you don't want or your router can't handle.

## How To Customize

You can use Web UI (found in Services/Simple AdBlock) to add/remove/edit links to:

- [Hosts files](<https://en.wikipedia.org/wiki/Hosts_(file)>) (127.0.0.1 or 0.0.0.0 followed by space and domain name per line) to be blocked.
- Domains lists (one domain name per line) to be blocked.
- AdBlockPlus lists of domains to be blocked.
- Domains lists (one domain name per line) to be allowed. It is useful if you want to run `adblock-fast` on multiple routers and maintain one centralized allow-list which you can publish on a web-server.

Please note that these lists **must** include either `http://` or `https://` (or, if `curl` is installed the `file://`) prefix. Some of the top block-lists (both hosts files and domains lists) suitable for routers with at least 8MB RAM are used in the default `adblock-fast` installation.

You can also use Web UI to add individual domains to be blocked or allowed.

If you want to use CLI to customize `adblock-fast` config, refer to the [Configuration](#configuration) section.

## How To Use

Once the service is enabled in the [config file](#default-settings), run `/etc/init.d/adblock-fast start` to start the service. Either `/etc/init.d/adblock-fast restart` or `/etc/init.d/adblock-fast reload` will only restart the service and/or re-download the lists if there were relevant changes in the config file since the last successful start. Had the previous start resulted in any error, either `/etc/init.d/adblock-fast start`, `/etc/init.d/adblock-fast restart` or `/etc/init.d/adblock-fast reload` will attempt to re-download the lists.

If you want to remove the domain(s) from the current block-list and add domain(s) to the config's allow-list, run `/etc/init.d/adblock-fast allow test-domain.com`.

If you want to check if the specific domain (or part of the domain name) is being blocked, run `/etc/init.d/adblock-fast check test-domain.com`.

If you want to force adblock-fast to re-download the lists, run `/etc/init.d/adblock-fast dl`.

### How to Schedule Redownloads

The command to force-redownload the lists is `/etc/init.d/adblock-fast dl` and this is what I use:

```sh
  echo '21 4 */3 * * /etc/init.d/adblock-fast dl' >> /etc/crontabs/root
```

## Adblock-fast Configuration

### Configuration

In the Web UI settings for the `adblock-fast` are split into `basic` and `advanced` tabs. The full list of configuration parameters of `adblock-fast.config` section is:

| Web UI Section | Option                                                        | Type      | Default                                                                                              | Description                                                                                                                                                                                                                                                                                                                                                                                                    |
| -------------- | ------------------------------------------------------------- | --------- | ---------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Basic          | <a name="enabled"></a>enabled                                 | boolean   | `0`                                                                                                  | Enable/disable the `adblock-fast` service.                                                                                                                                                                                                                                                                                                                                                                     |
| Basic          | <a name="config_update_enabled"></a>config_update_enabled     | boolean   | `0`                                                                                                  | Enable/disable the `adblock-fast` config update. Oftentimes, the URLs to the blocked hosts/domains files become obsolete/outdated, resulting in the error during lists download stage. `adblock-fast` already updates users' config files during install/reinstall, if you enable this variable it will also attempt to fetch and use the most recent config update file before downloading allow/block-lists. |
| -              | <a name="config_update_url"></a>config_update_url             | string    | [Link](https://cdn.jsdelivr.net/gh/openwrt/packages/net/adblock-fast/files/adblock-fast.conf.update) | By default, the config update URL is set to fetch the config update file from the jsDelivr CDN cache of the official OpenWrt source code repository. You can set it to a different URL from CLI/uci only if you wish.                                                                                                                                                                                          |
| -              | <a name="procd_trigger_wan6"></a>procd_trigger_wan6           | boolean   | `0`                                                                                                  | The service is started on WAN interface updates. As [OpenWrt may have floods of WAN6 updates](https://github.com/openwrt/openwrt/issues/5723#issuecomment-1040233237), the workaround for having the service started was to implement the `procd_trigger_wan6` boolean option (set to `0` as default) to enable/disable service start to be triggered by the WAN6 updates.                                     |
| -              | <a name="procd_boot_wan_timeout"></a>procd_boot_wan_timeout   | integer   | `60`                                                                                                 | The time (in seconds) the service will wait for the WAN interface discovery on boot.                                                                                                                                                                                                                                                                                                                           |
| Basic          | <a name="verbosity"></a>verbosity                             | integer   | `2`                                                                                                  | Can be set to `0`, `1` or `2` to control the console and system log output verbosity of the `adblock-fast` service.                                                                                                                                                                                                                                                                                            |
| Basic          | <a name="force_dns"></a>force_dns                             | boolean   | `1`                                                                                                  | Force router's DNS to local devices which may have different/hardcoded DNS server settings. If enabled, creates a firewall rule to intercept DNS requests from local devices to external DNS servers and redirect them to router.                                                                                                                                                                              |
|                | <a name="force_dns_interface"></a>force_dns_interface         | list      | `lan`                                                                                                | List of interfaces to force router DNS on. You only need to change it if you have changed/appended default LAN interface in `/etc/config/network`                                                                                                                                                                                                                                                              |
|                | <a name="force_dns_port"></a>force_dns_port                   | list      | `53 853`                                                                                             | List of ports to force router's DNS on.                                                                                                                                                                                                                                                                                                                                                                        |
| Basic          | <a name="led"></a>led                                         | string    | `none`                                                                                               | Use one of the router LEDs to indicate the AdBlocking status.                                                                                                                                                                                                                                                                                                                                                  |
| Basic          | <a name="dns"></a>dns                                         | string    | `dnsmasq.servers`                                                                                    | DNS Resolver option. See [table below](#dns-resolver-option) for addtional information.                                                                                                                                                                                                                                                                                                                        |
| Basic          | <a name="dnsmasq_instance"></a>dnsmasq_instance               | list      | `*`                                                                                                  | List of Dnsmasq instance numbers (or '\*' for all) to be affected by the service. This option is only visible in Web UI if the `dnsmasq.addnhosts` or `dnsmasq.servers` are selected as the DNS Resolver option.                                                                                                                                                                                               |
| Basic          | <a name="dnsmasq_config_file_url"></a>dnsmasq_config_file_url | string    |                                                                                                      | Link (URL) to the dnsmasq config file. If this option is set (non-empty), all the other lists are ignored (and will be deleted when you click Save & Apply in the WebUI). The dnsmasq config file is downloaded and used as is (not optimized). This option is only visible in Web UI if the `dnsmasq.config` is selected as the DNS Resolver.                                                                 |
| Basic          | <a name="smartdns_instance"></a>smartdns_instance             | list      | `*`                                                                                                  | List of SmartDNS instance numbers (or '\*' for all) to be affected by the service. This option is only visible in Web UI if the `smartdns.domainset` or `smartdns.nftset` are selected as the DNS Resolver option.                                                                                                                                                                                             |
| Advanced       | <a name="ipv6_enabled"></a>ipv6_enabled                       | boolean   | `0`                                                                                                  | Add IPv6 entries to block-list if `dnsmasq.addnhosts` is used. This option is only visible in Web UI if the `dnsmasq.addnhosts` is selected as the DNS Resolver option.                                                                                                                                                                                                                                        |
| Advanced       | <a name="download_timeout"></a>download_timeout               | integer   | `10`                                                                                                 | Time-out downloads if no reply received within that many last seconds.                                                                                                                                                                                                                                                                                                                                         |
| Advanced       | <a name="curl_additional_param"></a>curl_additional_param     | string    |                                                                                                      | If `curl` is installed and detected, you can pass any additional parameters to `curl` command line with this option.                                                                                                                                                                                                                                                                                           |
| Advanced       | <a name="curl_max_file_size"></a>curl_max_file_size           | integer   | `30000000`                                                                                           | If `curl` is installed and detected, it would not download files bigger than this.                                                                                                                                                                                                                                                                                                                             |
| Advanced       | <a name="curl_retry"></a>curl_retry                           | integer   | `3`                                                                                                  | If `curl` is installed and detected, attempt that many retries for failed downloads.                                                                                                                                                                                                                                                                                                                           |
| Advanced       | <a name="parallel_downloads"></a>parallel_downloads           | boolean   | `1`                                                                                                  | If enabled, all downloads are completed concurrently, if disabled -- sequentioally. Concurrent downloads dramatically speed up service loading.                                                                                                                                                                                                                                                                |
| Advanced       | <a name="debug"></a>debug                                     | boolean   | `0`                                                                                                  | If enabled, output service full debug to `/tmp/adblock-fast.log`. Please note that the debug file may clog up the router's RAM on some devices. Use with caution.                                                                                                                                                                                                                                              |
| Advanced       | <a name="allow_non_ascii"></a>allow_non_ascii                 | boolean   | `0`                                                                                                  | Enable support for non-ASCII characters in the final AdBlocking file. Only enable if your target service supports non-ASCII characters. If you enable this on the system where DNS resolver doesn't support non-ASCII characters, it will crash. Use with caution.                                                                                                                                             |
| Advanced       | <a name="compressed_cache"></a>compressed_cache               | boolean   | `0`                                                                                                  | Create compressed cache of the AdBlocking file in router's persistent memory. Only recommended to be used on routers with large ROM and/or routers with metered/flaky internet connection.                                                                                                                                                                                                                     |
| Advanced       | <a name="compressed_cache_dir"></a>compressed_cache_dir       | directory | `/etc`                                                                                               | Create compressed cache of the AdBlocking file in this directory. This option is only visible in Web UI if the `compressed_cache` option is enabled.                                                                                                                                                                                                                                                           |

### Allowed and Blocked Domains

| Web UI Section              | Option                                      | Type        | Default | Description              |
| --------------------------- | ------------------------------------------- | ----------- | ------- | ------------------------ |
| Allowed and Blocked Domains | <a name="allowed_domain"></a>allowed_domain | list/string |         | List of allowed domains. |
| Allowed and Blocked Domains | <a name="blocked_domain"></a>blocked_domain | list/string |         | List of blocked domains. |

### Allowed and Blocked Lists URLs

The lists of allowed and blocked domains are controlled by the unnamed `file_url` sections, the available options for each section are:

| Option  | Type    | Default | Description                                                                                                                                                                                                                        |
| ------- | ------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| action  | String  | `block` | Either `allow` or `block`.                                                                                                                                                                                                         |
| enabled | Boolean | `1`     | Set to `0` to disable processing of this list.                                                                                                                                                                                     |
| size    | Integer |         | Sizes for lists are automatically adjust upon download or migration from `simple-adblock`.                                                                                                                                         |
| url     | String  |         | URL to the remote allow/block-lists. **Must** include either `http://` or `https://` (with `curl` installed may use the `file://`) prefix. Recognized formats are listed [below](#recognized-remote-allowblock-list-file-formats). |

### DNS Resolver Option

Currently supported options are:

| Option               | Explanation                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `dnsmasq.addnhosts`  | Creates the Dnsmasq additional hosts file `/var/run/adblock-fast.addnhosts` and modifies Dnsmasq settings, so that Dnsmasq resolves all blocked domains to "local machine": 127.0.0.1. This option doesn't allow block-list optimization (by removing secondary level domains if the top-level domain is also in the block-list), so it results in a much larger block-list file, but, unlike other Dnsmasq-based options, it has almost no effect on the DNS look up speed. This option also allows quick reloads of Dnsmasq on block-list updates. This setting also allows you to configure which Dnsmasq instances would be affected by AdBlocking via `dnsmasq_instance` option.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `dnsmasq.conf`       | Creates the Dnsmasq config file `/var/dnsmasq.d/adblock-fast` so that Dnsmasq replies with NXDOMAIN: "domain not found". This option allows the block-list optimization (by removing secondary level domains if the top-level domain is also in the block-list), resulting in the smaller block-list file. This option also allows using top-level domains in the `blocked_domain` options to block the entire TLD. This option also allows the use of the `dnsmasq_config_file_url` setting which can be pointed to a pre-made `dnsmasq` config file which is used as is, without optimization or compressed cache creation. This option will slow down DNS look up speed somewhat.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `dnsmasq.ipset`      | Creates the Dnsmasq ipset file `/var/dnsmasq.d/adblock-fast.ipset` and the firewall rule to reject the matching requests. This is the only option for AdBlocking if you're using a browser with built-in [DNS-over-HTTPS proxy](https://en.wikipedia.org/wiki/DNS_over_HTTPS) enabled, like [Mozilla Firefox](https://support.mozilla.org/en-US/kb/firefox-dns-over-https#w_about-dns-over-https) or [Google Chrome/Chromium](https://blog.chromium.org/2019/09/experimenting-with-same-provider-dns.html). This option allows the block-list optimization (by removing secondary level domains if the top-level domain is also in the block-list), resulting in the smaller block-list file. This option requires installation of `dnsmasq-full` and `ipset` packages.<br/>PLEASE NOTE, that unlike other options which are truly domain name based blocking, this is essentially an IP address based blocking, ie: if you try to block `google-analytics.com` with this option, it may also block/break things like YouTube, Hangouts and other Google services if they share IP address(es) with `google-analytics.com`.                                                                                                                                  |
| `dnsmasq.nftset`     | Creates the Dnsmasq nft set file `/var/dnsmasq.d/adblock-fast.nftset` and the firewall rule to reject the matching requests. This is the only option for AdBlocking if you're using a browser with built-in [DNS-over-HTTPS proxy](https://en.wikipedia.org/wiki/DNS_over_HTTPS) enabled, like [Mozilla Firefox](https://support.mozilla.org/en-US/kb/firefox-dns-over-https#w_about-dns-over-https) or [Google Chrome/Chromium](https://blog.chromium.org/2019/09/experimenting-with-same-provider-dns.html). This option allows the block-list optimization (by removing secondary level domains if the top-level domain is also in the block-list), resulting in the smaller block-list file. This option requires installation of `dnsmasq-full` and `nft` packages.<br/>PLEASE NOTE, that unlike other options which are truly domain name based blocking, this is essentially an IP address based blocking, ie: if you try to block `google-analytics.com` with this option, it may also block/break things like YouTube, Hangouts and other Google services if they share IP address(es) with `google-analytics.com`.                                                                                                                                 |
| `dnsmasq.servers`    | Creates the Dnsmasq servers file `/var/run/adblock-fast.servers` and modifies Dnsmasq settings so that Dnsmasq replies with NXDOMAIN: "domain not found". This option allows the block-list optimization (by removing secondary level domains if the top-level domain is also in the block-list), resulting in the smaller block-list file. This is a default setting as it results in the smaller block-file and allows quick reloads of Dnsmasq. This setting also allows you to configure which Dnsmasq instances would be affected by AdBlocking via `dnsmasq_instance` option. This option will slow down DNS look up speed somewhat.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `smartdns.domainset` | Creates the SmartDNS domain-set file and modifies SmartDNS settings so that SmartDNS replies with NXDOMAIN: "domain not found". This option allows the block-list optimization (by removing secondary level domains if the top-level domain is also in the block-list), resulting in the smaller block-list file. This option will slow down DNS look up speed somewhat. This is a recommended SmartDNS setting as it results in the smaller block-file and allows quick reloads of SmartDNS. This setting also allows you to configure which SmartDNS instances would be affected by AdBlocking via `smartdns_instance` option.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `smartdns.ipset`     | Creates the SmartDNS ipset file and the firewall rule to reject the matching requests. This is the only option for AdBlocking if you're using a browser with [DNS-over-HTTPS proxy](https://en.wikipedia.org/wiki/DNS_over_HTTPS) built-in, like [Mozilla Firefox](https://support.mozilla.org/en-US/kb/firefox-dns-over-https#w_about-dns-over-https) or [Google Chrome/Chromium](https://blog.chromium.org/2019/09/experimenting-with-same-provider-dns.html). This option allows the block-list optimization (by removing secondary level domains if the top-level domain is also in the block-list), resulting in the smaller block-list file. This option is recommended for OpenWrt 21.02 and earlier only.<br/>PLEASE NOTE, that unlike other options which are truly domain name based blocking, this is essentially an IP address based blocking, ie: if you try to block `google-analytics.com` with this option, it may also block/break things like YouTube, Hangouts and other Google services if they share IP address(es) with `google-analytics.com`.                                                                                                                                                                                        |
| `smartdns.nftset`    | Creates the SmartDNS nft set file and and the firewall rule to reject the matching requests. This is the only option for AdBlocking if you're using a browser with [DNS-over-HTTPS proxy](https://en.wikipedia.org/wiki/DNS_over_HTTPS) built-in, like [Mozilla Firefox](https://support.mozilla.org/en-US/kb/firefox-dns-over-https#w_about-dns-over-https) or [Google Chrome/Chromium](https://blog.chromium.org/2019/09/experimenting-with-same-provider-dns.html). This option allows the block-list optimization (by removing secondary level domains if the top-level domain is also in the block-list), resulting in the smaller block-list file. This option allows the block-list optimization (by removing secondary level domains if the top-level domain is also in the block-list), resulting in the smaller block-list file. This option is recommended for OpenWrt 22.03 and later/newer.<br/>PLEASE NOTE, that unlike other options which are truly domain name based blocking, this is essentially an IP address based blocking, ie: if you try to block `google-analytics.com` with this option, it may also block/break things like YouTube, Hangouts and other Google services if they share IP address(es) with `google-analytics.com`. |
| `unbound.adb_list`   | Creates the Unbound config file `/var/lib/unbound/adb_list.adblock-fast` so that Unbound replies with NXDOMAIN: "domain not found". This option allows the block-list optimization (by removing secondary level domains if the top-level domain is also in the block-list), resulting in the smaller block-list file.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

### Default config

The default config file installed with the package can be found [here](https://github.com/openwrt/packages/blob/master/net/adblock-fast/files/adblock-fast.conf). The config file contains some comments to explain some settings and the lists which are too big for most routers are commented out by default as well.

## How Does It Work

This service downloads (and processes in the background, removing comments and other useless data) lists of hosts and domains to be blocked, combines those lists into one big block-list, removes duplicates and sorts it and then removes your allowed domains from the block-list before converting to to Dnsmasq/SmartDNS/Unbound-compatible file and restarting Dnsmasq/SmartDNS/Unbound if needed. The result of the process is that Dnsmasq/SmartDNS/Unbound return NXDOMAIN or 127.0.0.1 (depending on settings) for the blocked domains.

If you specify `google.com` as a domain to be allowed, you will have access to `google.com`, `www.google.com`, `analytics.google.com`, but not fake domains like `email-google.com` or `drive.google.com.verify.signin.normandeassociation.com` for example. If you only want to allow `www.google.com` while blocking all other `google.com` subdomains, just specify `www.google.com` as domain to be allowed.

In general, whatever domain is specified to be allowed; it, along with with its subdomains will be allowed, but not any fake domains containing it.

### Recognized remote allow/block-list file formats

The service can parse the remote allow/block-lists in the following formats:

- AdBlockPlus format
- Dnsmasq format with `address=` assignments
- Dnsmasq format with `local=` assignments
- Dnsmasq format with `server=` assignments
- Hosts file
- Domains list (plain text with one domain per line)

## How It Does Not Work

For most of the [DNS Resolver Options](#dns-resolver-option) to work, your local LAN clients need to be set to use your router's DNS (by default `192.168.1.1`). The `dnsmasq.addnhosts` is the only option which can help you block ads if your local LAN clients are NOT using your router's DNS. There are multiple ways your local LAN clients can be set to NOT use your router's DNS:

1.  Hardcoded on the device. Some Android Lollipop 5.0 phones, some media-centric tablets and some streaming devices for example are known to have hardcoded DNS servers and they ignore your router's DNS settings. You can fix this by either:
    - Rooting your device and changing it from hardcoded DNS servers to obtaining DNS servers from DHCP.
    - Enabling `adblock-fast`'s `force_dns` setting to override the hardcoded DNS on your device.
2.  Manually set on the device. Instead of setting your device to obtain the DNS settings via DHCP, you can set the DNS servers manually. There are some guides online which recommend manually changing the DNS servers on your computer to Google's (8.8.8.8) or Cloudflare's (1.1.1.1) or OpenDNS (208.67.222.222). You can fix this by either:
    - Changing the on-device DNS settings from manual to obtaining DNS servers from DHCP and changing your [router's DNS settings](https://openwrt.org/docs/guide-user/base-system/dhcp#all_options) to use the DNS from Google, Cloudflare or OpenDNS respectively.
    - Enabling `adblock-fast`'s `force_dns` setting to override the hardcoded DNS on your device.
3.  Sent to your device from router via [DHCP Options](https://openwrt.org/docs/guide-user/base-system/dhcp_configuration#dhcp_options). You can fix this by either:
    - Removing [DHCP Options](https://openwrt.org/docs/guide-user/base-system/dhcp_configuration#dhcp_options) 5 and 6 from your router's `/etc/config/dhcp` file.
    - Enabling `adblock-fast`'s `force_dns` setting to override the hardcoded DNS on your device.
4.  By using the DNS-over-TLS, DNS-over-HTTPS or DNSCrypt on your local device or (if supported) by browser on your local device. You can fix this only by:
    - Stopping/removing/disabling DNS-over-TLS, DNS-over-HTTPS or DNSCrypt on your local device and using the secure DNS on your router instead. There are merits to all three of the options above, I can recommend the `https-dns-proxy` and `luci-app-https-dns-proxy` packages for enabling DNS-over-HTTPS on your router.
5.  If you are running a wireguard "server" on your router and remote clients connect to it, the AdBlocking may not work properly for your remote clients until you add the following to `/etc/network` (credit to [dibdot](https://forum.openwrt.org/t/wireguard-and-adblock/49351/6)):

    ```sh
    config route
      option interface 'wg0'
      option target '192.168.1.0'
      option netmask '255.255.255.0'
    ```

## Documentation / Discussion

Please head to [OpenWrt Forum](https://forum.openwrt.org/t/adblock-fast-fast-lean-and-fully-uci-luci-configurable-adblocking/1327/) for discussion of this package.

## Thanks

I'd like to thank everyone who helped create, test and troubleshoot this service. Special thanks to [@hnyman](https://github.com/hnyman) for general package/luci guidance, [@dibdot](https://github.com/dibdot) for general guidance and block-list optimization code, [@ckuethe](https://github.com/ckuethe) for the curl support, non-ASCII filtering and compressed cache code, [@EricLuehrsen](https://github.com/EricLuehrsen) for the Unbound support information, [@vgaetera](https://github.com/vgaetera) for firewall-related advice, [@mushoz](https://github.com/mushoz) for performance testing and [@phasecat](https://forum.openwrt.org/u/phasecat/summary) for submitting various bugs and testing.

<!-- markdownlint-disable MD033 -->

<script defer src='https://static.cloudflareinsights.com/beacon.min.js' data-cf-beacon='{"token": "911798f2c34b45338f8f8182830a3eb6"}'></script>
