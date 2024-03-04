<!-- markdownlint-disable MD013 -->

<!-- markdownlint-disable MD033 -->

# DNS Over HTTPS Proxy (https-dns-proxy)

## Description

A lean RFC8484-compatible (no JSON API support) DNS-over-HTTPS (DoH) proxy service which supports DoH servers. The accmponying WebUI (`luci-app-https-dns-proxy`) supports [more than 40 public resolvers](https://github.com/stangri/source.openwrt.melmac.net/tree/master/luci-app-https-dns-proxy/root/usr/share/https-dns-proxy/providers). Based on [@aarond10](https://github.com/aarond10)'s [https-dns-proxy](https://github.com/aarond10/https_dns_proxy).

## Features

- [RFC8484](https://tools.ietf.org/html/rfc8484)-compatible DoH Proxy.
- Compact size (about 40Kb installed).
- (By default) automatically updates `dnsmasq` settings to use DoH proxy when it's started and reverts to old `dnsmasq` resolvers when DoH proxy is stopped.
- (By default) automatically adds records for canary domains[<sup>1</sup>](https://support.mozilla.org/en-US/kb/canary-domain-use-application-dnsnet)<sup>,</sup>[<sup>2</sup>](https://developer.apple.com/support/prepare-your-network-for-icloud-private-relay) upon start and removes them upon service stop.
- Web UI (`luci-app-https-dns-proxy`) available. [More than 40 public resolvers](https://github.com/stangri/source.openwrt.melmac.net/tree/master/luci-app-https-dns-proxy/root/usr/share/https-dns-proxy/providers) are supported within the WebUI for easy configuration.

## Screenshots (luci-app-https-dns-proxy)

Service Status

![screenshot](https://docs.openwrt.melmac.net/https-dns-proxy/screenshots/screenshot02-status.png "Service Status")

Service Configuration

![screenshot](https://docs.openwrt.melmac.net/https-dns-proxy/screenshots/screenshot02-config.png "Service Configuration")

Service Instances

![screenshot](https://docs.openwrt.melmac.net/https-dns-proxy/screenshots/screenshot02-instances.png "Service Instances")

## Requirements

This proxy requires the following packages to be installed on your router: `libc`, `libcares`, `libcurl`, `libev`, `ca-bundle`. They will be automatically installed when you're installing `https-dns-proxy`.

### HTTP/2 Support

Some resolvers may require `HTTP/2`. By default, `HTTP/2` is supported by `curl` in OpenWrt 22.03 and later, if you run an older version of OpenWrt I'd recommend you upgrade to a most recent released version and make sure the following packages are installed: `curl`, `libcurl4`, `libnghttp2`.

### HTTP/3 (QUIC) Support

As of OpenWrt version 23.05.2, the OpenWrt installation/repositories do not contain packages required for HTTP/3 support. However the `luci-app-https-dns-proxy` has facilities for marking some pre-configured providers as `HTTP/3` only and can detect the `HTTP/3` support on the OpenWrt device. You can compile `curl` with `HTTP/3` support using the out-of-tree fork of `OpenSSL`, additional information is available in [curl README](https://docs.openwrt.melmac.net/curl/).

## Unmet Dependencies

If you are running a development (trunk/snapshot) build of OpenWrt on your router and your build is outdated (meaning that packages of the same revision/commit hash are no longer available and when you try to satisfy the [requirements](#requirements) you get errors), please flash either current OpenWrt release image or current development/snapshot image.

## How To Install

Install `https-dns-proxy` and `luci-app-https-dns-proxy` packages from Web UI or run the following in the command line:

```sh
opkg update; opkg install https-dns-proxy luci-app-https-dns-proxy;
```

## Default Settings

Default configuration has service enabled and starts the service with Google and Cloudflare DoH servers. In most configurations, you will keep the default `dnsmasq` service installed to handle requests from devices in your local network and point `dnsmasq` to use `https-dns-proxy` for name resolution.

By default, the service will intelligently override existing `dnsmasq` servers settings on start to use the DoH servers and restores original `dnsmasq` servers on stop. See the [Configuration Settings](#configuration-settings) section below for more information and how to disable this behavior.

## Configuration Settings

Configuration contains the general (named) "main" config section where you can configure which `dnsmasq` settings the service will automatically affect and the typed (unnamed) https-dns-proxy instance settings. The original config file is included below:

```text
config main 'config'
  option dnsmasq_config_update '*'
  option canary_domains_icloud '1'
  option canary_domains_mozilla '1'
  option force_dns '1'
  list force_dns_port '53'
  list force_dns_port '853'

config https-dns-proxy
  option bootstrap_dns '8.8.8.8,8.8.4.4'
  option resolver_url 'https://dns.google/dns-query'
  option listen_addr '127.0.0.1'
  option listen_port '5053'
  option user 'nobody'
  option group 'nogroup'

config https-dns-proxy
  option bootstrap_dns '1.1.1.1,1.0.0.1'
  option resolver_url 'https://cloudflare-dns.com/dns-query'
  option listen_addr '127.0.0.1'
  option listen_port '5054'
  option user 'nobody'
  option group 'nogroup'
```

### General Settings

#### canary_domains_icloud

This setting enables router to block requests to iCloud Private Relay canary domains, indicating that the local device should use the router's dns resolution (encrypted with `https-dns-proxy`) instead of the encrypted/proprietary iCloud Private Relay resolvers. This is set to `1` (enabled) by default. Shown in WebUI and processed only if `force_dns` is also set to 1.

#### canary_domains_mozilla

This setting enables router to block requests to Mozilla canary domains, indicating that the local device should use the router's dns resolution (encrypted with `https-dns-proxy`) instead of the encrypted Mozilla resolvers. This is set to `1` (enabled) by default. Shown in WebUI and processed only if `force_dns` is also set to 1.

#### dnsmasq_config_update

The `dnsmasq_config_update` option can be set to dash (set to `'-'` to not change `dnsmasq` server settings on start/stop), can be set to `'*'` to affect all `dnsmasq` instance server settings or have a space-separated list of `dnsmasq` instances or named sections to affect (like `'0 4 5'` or `'0 backup_dns 5'`). If this option is omitted, the default setting is `'*'`. When the service is set to update the `dnsmasq` servers setting on start/stop, it does not override entries which contain either `#` or `/`, so the entries like listed below will be kept in use:

```test
  list server '/onion/127.0.0.1#65453'
  list server '/openwrt.org/8.8.8.8'
  list server '/pool.ntp.org/8.8.8.8'
  list server '127.0.0.1#15353'
  list server '127.0.0.1#55353'
  list server '127.0.0.1#65353'
```

#### force_dns

The `force_dns` setting is used to force the router's default resolver to all connected devices even if they are set to use other DNS resolvers or if other DNS resolvers are hardcoded in connected devices' settings. You can additionally control which ports the `force_dns` setting should be actvive on, the default values are `53` (regular DNS) and `853` (DNS over TLS). If the listed port is open/active on OpenWrt router, the service will create a `redirect` to the indicated port number, otherwise the service will create a `REJECT` rule. The intention for `REJECT` is that if the encrypted DNS requests has failed for your local device, it will fall-back on an unencrypted DNS request which will be then intercepted by the router and sent to the https-dns-proxy service. This is set to `1` (enabled) by default.

#### procd_fw_src_interfaces

This option allows you to override the interface (`lan` by default) which is used in the PROCD firewall redirects/rules the service creates if `force_dns` is enabled. Only needed if you have renamed or deleted your `lan` interface. If you indicate more than one interface, separate them by spaces.

#### procd_trigger_wan6

The service is restarted on WAN interface updates. As [OpenWrt may have floods of WAN6 updates](https://github.com/openwrt/openwrt/issues/5723#issuecomment-1040233237), the workaround for having the service restarted (and cause two `dnsmasq` restarts in turn) was to implement the `procd_trigger_wan6` boolean option (set to '0' as default) to enable/disable service restarts to be triggered by the WAN6 updates.

### Instance Settings

The https-dns-proxy instance settings are:

| Parameter               | Type             | Default     | Description                                                                                                                                                                                                                            |
| ----------------------- | ---------------- | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| bootstrap_dns           | IP Address       |             | The non-encrypted DNS servers to be used to resolve the DoH server name on start.                                                                                                                                                      |
| dscp_codepoint          | Integer [0-63]   |             | Optional DSCP codepoint [0-63] to set on upstream DNS server connections.                                                                                                                                                              |
| listen_addr             | IP Address       | 127.0.0.1   | The local IP address to listen to requests.                                                                                                                                                                                            |
| listen_port             | Port             | 5053 and up | If this setting is omitted, the service will start the first https-dns-proxy instance on port 5053, second on 5054 and so on.                                                                                                          |
| logfile                 | Filepath         |             | Full filepath to the file to log the instance events to.                                                                                                                                                                               |
| polling_interval        | Integer [5-3600] | 120         | Optional polling interval of DNS servers.                                                                                                                                                                                              |
|                         |                  |             |                                                                                                                                                                                                                                        |
| resolver_url            | URL              |             | The https URL to the RFC8484-compatible resolver.                                                                                                                                                                                      |
| proxy_server            | URL              |             | Optional HTTP proxy. e.g. socks5://127.0.0.1:1080. Remote name resolution will be used if the protocol supports it (http, https, socks4a, socks5h), otherwise initial DNS resolution will still be done via the bootstrap DNS servers. |
| user                    | String           | nobody      | Local user to run instance under.                                                                                                                                                                                                      |
| group                   | String           | nogroup     | Local group to run instance under.                                                                                                                                                                                                     |
| use_http1               | Boolean          | 0           | If set to 1, use HTTP/1 on installations with broken/outdated `curl` package. Included for posterity reasons, you will most likely not ever need it on OpenWrt.                                                                        |
| verbosity               | Integer          | 0           | Logging verbosity level. Fatal = 0, error = 1, warning = 2, info = 3, debug = 4.                                                                                                                                                       |
| use_ipv6_resolvers_only | Boolean          | 0           | If set to 1, forces IPv6 DNS resolvers instead of IPv4.                                                                                                                                                                                |

Please also refer to the [Usage section at upstream README](https://github.com/aarond10/https_dns_proxy/blob/master/README.md#usage) which may contain additional/more details on some parameters.

## <a name='GettingHelp'></a>Getting Help

If things are not working as intended, please run the following commands and include their output in your post/issue:

```sh
ubus call system board
uci export dhcp
uci export network
uci export https-dns-proxy
/etc/init.d/https-dns-proxy status
```

## Thanks

This OpenWrt package wouldn't have been possible without [@aarond10](https://github.com/aarond10)'s [https-dns-proxy](https://github.com/aarond10/https_dns_proxy) and his active participation in the OpenWrt package itself. Thanks to [@oldium](https://github.com/oldium) and [@curtdept](https://github.com/curtdept) for their contributions to this package. Special thanks to [@jow-](https://github.com/jow-) for general package/luci guidance.

<!-- markdownlint-disable MD033 -->

<script defer src='https://static.cloudflareinsights.com/beacon.min.js' data-cf-beacon='{"token": "911798f2c34b45338f8f8182830a3eb6"}'></script>
