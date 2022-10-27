<!-- markdownlint-disable MD013 -->

<!-- markdownlint-disable MD030 -->

# Policy-Based Routing

## Statement about OpenWrt 22.03.0 release and this package

There are now two packages of this service available:

-   `pbr-iptables` which supports fw3, iptables, ipset and `dnsmasq.ipset` option
-   `pbr` which supports fw4, nft, nft sets and `dnsmasq.nftset` option (but because OpenWrt's `dnsmasq` doesn't support nft sets yet, you can't use `dnsmasq` to resolve domain names from policies) as well as fw3, iptables, ipset and `dnsmasq.ipset` option.

Both packages install the same init script (what you actually run when you invoke `service pbr ...` or `/etc/init.d/pbr ...`), however both packages install some specific files and `pbr` can run in either `nft` or `iptables`/`ipset` mode, whereas `pbr-iptables` can only run in `iptables`/`ipset` mode.

The package-specific files that `pbr-iptables` installs are:

-   the `/etc/config/pbr` file with the `resolver_set` set to `dnsmasq.ipset`
-   legacy iptables/ipset packages

The package-specific files that `pbr` installs are:

-   the `/etc/config/pbr` file with the `resolver_set` set to `none` (will be switched to `dnsmasq.nftset` when OpenWrt's `dnsmasq` supports it)
-   the `fw4`-specific `nft` scripts (installed into `/usr/share/nftables.d/`) to set up default service chains as part of the fw4 start/restart/reload processes

The `pbr` decides wherver to use `iptables`/`ipset` mode or `nft` mode on run time. If the `nft` binary is available, the `resolver_set` is not set to `dnsmasq.ipset` and a main `pbr_prerouting` chain has been created by the `fw4`-specific `nft` script, it runs in the `nft` mode, otherwise it runs in the `iptables`/`ipset` mode.

Each package of the service has its own dependencies, so only `pbr-iptables` can be installed on OpenWrt 21.02 and earlier, but either `pbr` or `pbr-iptables` can be installed on OpenWrt 22.03. It is recommended to install `pbr` on OpenWrt 22.03 and if you want to use [use dnsmasq ipset support](#use-dnsmasq-ipset-support), [install dnsmasq-full](#how-to-install-dnsmasq-full), also [install legacy iptables/ipset packages](#how-to-install-legacy-iptablesipset-packages) and then change `resolver_set` option to `dnsmasq.ipset` to force `iptables`/`ipset` mode.

Both `pbr-iptables` and `pbr` in `iptables`/`ipset` mode work just fine with recently released OpenWrt 22.03.0. You can safely ignore the warning on the Status -> Firewall page about legacy iptables rules created by either package.

## Description

This service allows you to define rules (policies) for routing traffic via WAN or your L2TP, Openconnect, OpenVPN, PPTP, Softether or Wireguard tunnels. Policies can be set based on any combination of local/remote ports, local/remote IPv4 or IPv6 addresses/subnets or domains. This service supersedes and obsoletes the `VPN Bypass` ([README at GitHub](https://docs.openwrt.melmac.net/vpnbypass/)/[README at jsDelivr](https://cdn.jsdelivr.net/gh/stangri/docs.openwrt.melmac.net/vpnbypass/README.md)) and `VPN Policy Routing` ([README at GitHub](https://docs.openwrt.melmac.net/vpn-policy-routing/)/[README at jsDelivr](https://cdn.jsdelivr.net/gh/stangri/docs.openwrt.melmac.net/vpn-policy-routing/README.md)) services. For information on transition from `vpn-policy-routing` package please read about [differences](#a-word-about-differences-from-vpn-policy-routing) and [migration](#a-word-about-migrating-from-vpn-policy-routing).

## Features

### Gateways/Tunnels

-   Any policy can target either WAN or a VPN tunnel interface.
-   L2TP tunnels supported (with protocol names l2tp\*).
-   Openconnect tunnels supported (with protocol names openconnect\*).
-   OpenVPN tunnels supported (with device names tun\*).[<sup>#1</sup>](#footnote1) [<sup>#2</sup>](#footnote2)
-   PPTP tunnels supported (with protocol names pptp\*).
-   Wireguard tunnels supported (with protocol names wireguard\*).

### IPv4/IPv6/Port-Based Policies

-   Policies based on local names, IPs or subnets. You can specify a single IP (as in `192.168.1.70`) or a local subnet (as in `192.168.1.81/29`) or a local device name (as in `nexusplayer`). IPv6 addresses are also supported.
-   Policies based on local ports numbers. Can be set as an individual port number (`32400`), a range (`5060-5061`), a space-separated list (`80 8080`) or a combination of the above (`80 8080 5060-5061`). Limited to 15 space-separated entries per policy.
-   Policies based on remote IPs/subnets or domain names. Same format/syntax as local IPs/subnets.
-   Policies based on remote ports numbers. Same format/syntax and restrictions as local ports.
-   You can mix the IP addresses/subnets and device (or domain) names in one field separating them by space (like this: `66.220.2.74 he.net tunnelbroker.net`).
-   See [Policy Options](#policy-options) section for more information.

### Domain-Based Policies

-   Policies based on (remote) domain names can be processed in different ways, please review the [Policy Options](#policy-options) section and [Footnotes/Known Issues](#footnotesknown-issues) section, specifically [<sup>#5</sup>](#footnote5) and any other information in that section relevant to domain-based routing/DNS.

### Physical Device Policies

-   Policies based on a local physical device (like a specially created wlan), please review the [Policy Options](#policy-options) section and [Footnotes/Known Issues](#footnotesknown-issues) section, specifically [<sup>#6</sup>](#footnote6) and any other information in that section relevant to handling physical device.

### DSCP Tag-Based Policies

You can also set policies for traffic with specific DSCP tag. On Windows 10, for example, you can mark traffic from specific apps with DSCP tags (instructions for tagging specific app traffic in Windows 10 can be found [here](http://serverfault.com/questions/769843/cannot-set-dscp-on-windows-10-pro-via-group-policy)).

### Custom User Files

If the custom user file includes are set, the service will load and execute them after setting up routing and the sets and processing policies. This allows, for example, to add large numbers of domains/IP addresses to ipsets or nft sets without manually adding all of them to the config file.

Two example custom user-files are provided: `/usr/share/pbr/pbr.user.aws` and `/usr/share/pbr/pbr.user.netflix`. They are provided to pull the AWS and Netflix IP addresses into the default WAN IPv4 sets the service sets up, indicated in the `TARGET_IPSET` variable at the top of each script.

### Strict Enforcement

-   Supports strict policy enforcement, even if the policy interface is down -- resulting in network being unreachable for specific policy (enabled by default).

### Use Resolver's Set Support

-   If supported on the system, service can be set to utilize resolver's set support. Currently supported resolver's set options are listed below.

#### Use DNSMASQ ipset Support

-   Either version of the service package can be configured to utilize `dnsmasq`'s `ipset` support, which requires the `dnsmasq-full` package with `ipset` support to be installed (see [How to install dnsmasq-full](#how-to-install-dnsmasq-full)). This significantly improves the start up time because `dnsmasq` resolves the domain names and adds them to appropriate `ipset` in background. Another benefit of using `dnsmasq`'s `ipset` support is that it also automatically adds third-level domains to the `ipset`: if `domain.com` is added to the policy, this policy will affect all `*.domain.com` subdomains. This also works for top-level domains as well, a policy targeting the `at` for example, will affect all the `*.at` domains.
-   Please review the [Footnotes/Known Issues](#footnotesknown-issues) section, specifically [<sup>#5</sup>](#footnote5) and [<sup>#7</sup>](#footnote7) and any other information in that section relevant to domain-based routing/DNS.

#### Use DNSMASQ nft sets Support

-   The `pbr` package can be configure to utilize `dnsmasq`'s `nft` `sets` support, which requires the `dnsmasq-full` package with `nft` `sets` support to be installed (see [How to install dnsmasq-full](#how-to-install-dnsmasq-full)). This significantly improves the start up time because `dnsmasq` resolves the domain names and adds them to appropriate `nft` `set` in background. Another benefit of using `dnsmasq`'s `nft` `set` support is that it also automatically adds third-level domains to the `set`: if `domain.com` is added to the policy, this policy will affect all `*.domain.com` subdomains. This also works for top-level domains as well, a policy targeting the `at` for example, will affect all the `*.at` domains.
-   Please review the [Footnotes/Known Issues](#footnotesknown-issues) section, specifically [<sup>#5</sup>](#footnote5) and [<sup>#7</sup>](#footnote7) and any other information in that section relevant to domain-based routing/DNS.

### Customization

-   Can be fully configured with `uci` commands or by editing `/etc/config/pbr` file.
-   Has a companion package (`luci-app-pbr`) so policies can be configured with Web UI.

### Other Features

-   Doesn't stay in memory, creates the routing tables and `iptables` rules/`sets` entries which are automatically updated when supported/monitored interface changes.
-   Proudly made in :maple_leaf: Canada :maple_leaf:, using locally-sourced electrons.

## Screenshots (luci-app-pbr)

Service Status
![screenshot](https://docs.openwrt.melmac.net/pbr/screenshots/01-status.png "Service Status")

Configuration - Basic Configuration
![screenshot](https://docs.openwrt.melmac.net/pbr/screenshots/02-config-basic.png "Basic Configuration")

Configuration - Advanced Configuration
![screenshot](https://docs.openwrt.melmac.net/pbr/screenshots/03-config-advanced.png "Advanced Configuration")

Configuration - WebUI Configuration
![screenshot](https://docs.openwrt.melmac.net/pbr/screenshots/04-config-webui.png "WebUI Configuration")

Policies
![screenshot](https://docs.openwrt.melmac.net/pbr/screenshots/05-policies.png "Policies")

DSCP Tagging
![screenshot](https://docs.openwrt.melmac.net/pbr/screenshots/06-dscp-tag.png "DSCP Tagging")

Custom User File Includes
![screenshot](https://docs.openwrt.melmac.net/pbr/screenshots/07-custom-user-files.png "Custom User File Includes")

## How It Works

### How It Works (`iptables`/`ipset` mode)

On start, this service creates routing tables for each supported interface (WAN/WAN6 and VPN tunnels) which are used to route specially marked packets. For the `mangle` table's `FORWARD`, `INPUT`, `OUTPUT`, `PREROUTING` and `POSTROUTING` chains, the service creates corresponding `PBR_*` chains to which policies are assigned. Evaluation of packets happens in these `PBR_*` chains after which the packets are sent for marking to the `PBR_MARK*` chains. If enabled, the service also creates the ipsets (and the corresponding `iptables` rule for marking packets matching the `ipset`) for `dest_addr` and `src_addr` entries. The service then processes the user-created policies.

### How It Works (`nft` mode)

On start, this service creates routing tables for each supported interface (WAN/WAN6 and VPN tunnels) which are used to route specially marked packets. Rules for the policies are created in the service-specific chains set up by the `fw4`-specific `nft` scripts installed with the package. Evaluation of packets happens in these `pbr_*` chains after which the packets are sent for marking to the `pbr_mark_*` chains. Whenever possible, the service also creates named sets for `dest_addr` and `src_addr` entries and anonymous sets for `dest_port` and `src_port`. The service then processes the user-created policies.

### Processing Policies

Each policy can result in either a new `iptables` or `nft` rule and possibly an `ipset` or a named `nft` `set` to match `dest_addr` and `src_addr`. Anonymous sets may be created within `nft` rules for `dest_port` and `src_port`.

#### Processing Policies (`iptables`/`ipset` mode)

-   Policies with the MAC-addresses, IP addresses or local device names in `src_addr` can be created as `iptables` rules or `ipset` entries.
-   Policies with non-empty `dest_port` and `src_port` are always created as `iptables` rules.
-   Policies with the netmasks in `dest_addr` or `src_addr` can be created as `iptables` rules or `ipset` entries.
-   Policies with empty `dest_port` and `src_port` may be created as `iptables` rules or `dnsmasq`'s `ipset` or an `ipset` (if enabled).

#### Processing Policies (`nft` mode)

-   Policies with the MAC-addresses, IP addresses, netmasks, local device names or domains will result in a rule targeting named `nft` `sets`.
-   Policies with non-empty `dest_port` and `src_port` will be created with anonymous `nft` `sets` within the rule.
-   The `dnsmasq` `nftset` entries will be used for domains (if supported and enabled).

### Policies Priorities

-   The policy priority is the same as its order as listed in Web UI and `/etc/config/pbr`. The higher the policy is in the Web UI and configuration file, the higher its priority is.
-   If set, the `DSCP` policies is set up first when creating the interface routing.
-   If enabled, it is highly recommended that the policies with `IGNORE` target are at the top of the policies list.

## How To Install

Until I send the requests to have this package (and accompanying WebUI package) to be included in the official OpenWrt 22.03 repo (or if you want to use the package on an unsupported older OpenWrt version), you will need to add my repo to your router following instructions at the [How to Use Section of repo documentation](https://docs.openwrt.melmac.net/#how-to-use).

Then, please make sure that the [requirements](#requirements) are satisfied and install `pbr` and `luci-app-pbr` from Web UI or connect to your router via ssh and run the following commands:

On OpenWrt 22.03 and newer:

```sh
opkg update
opkg install pbr luci-app-pbr
```

On OpenWrt 21.02 and older (not tested/supported):

```sh
opkg update
opkg install pbr-iptables luci-app-pbr
```

Again, until these packages are found in the official feed/repo for your version of OpenWrt, you will need to add a custom repo to your router following instructions on [GitHub](https://docs.openwrt.melmac.net/#on-your-router)/[jsDelivr](https://cdn.jsdelivr.net/gh/stangri/docs.openwrt.melmac.net/README.md#on-your-router) first.

These packages have been designed to be work on OpenWrt 22.03 and newer.

### Requirements

Depending on the package flavour, some packages may need to be installed on your router.

#### Requirements (pbr-iptables)

This service requires the following packages to be installed on your router: `ipset`, `resolveip`, `ip-full` (or a `busybox` built with `ip` support), `kmod-ipt-ipset` and `iptables`.

To satisfy the requirements, connect to your router via ssh and run the following commands:

```sh
opkg update; opkg install ipset resolveip ip-full kmod-ipt-ipset iptables
```

#### Requirements (pbr)

Default builds of OpenWrt 22.03.0 and later are fully compatible with `pbr` and require no additional packages. If you're using a non-standard build, you may have to install the following packages to be installed on your router: `resolveip`, `ip-full` (or a `busybox` built with `ip` support).

To satisfy the requirements, connect to your router via ssh and run the following commands:

```sh
opkg update; opkg install resolveip ip-full
```

### How to install dnsmasq-full

If you want to use `dnsmasq`'s `ipset` or `nft` `sets` support, you will need to install `dnsmasq-full` instead of the `dnsmasq`. To do that, connect to your router via ssh and run the following commands:

```sh
opkg update
cd /tmp/ && opkg download dnsmasq-full
opkg remove dnsmasq
opkg install dnsmasq-full --cache /tmp/
rm -f /tmp/dnsmasq-full*.ipk
```

### How to install legacy iptables/ipset packages

If you install `pbr` (and not the `pbr-iptables`, which brings all necessary legacy dependencies), but want to use `pbr` in `iptables` mode on OpenWrt 22.03, you will need to connect to your router via ssh and run the following commands:

```sh
opkg update
opkg install ipset libnettle8 libnetfilter-conntrack3 iptables kmod-ipt-ipset iptables-mod-ipopt
```

If you want to use `dnsmasq`'s `ipset` support, you will need to install [`dnsmasq-full` instead of the `dnsmasq`](#how-to-install-dnsmasq-full).

### Unmet dependencies

If you are running a development (trunk/snapshot) build of OpenWrt on your router and your build is outdated (meaning that packages of the same revision/commit hash are no longer available and when you try to satisfy the [requirements](#requirements) you get errors), please flash either current OpenWrt release image or current development/snapshot image.

## Service Configuration Settings

As per screenshots above, in the Web UI the `pbr` configuration is split into `Basic`, `Advanced` and `WebUI` settings. The full list of configuration parameters of `pbr.config` section is:

| Web UI Section      | Parameter                | Type        | Default                      | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ------------------- | ------------------------ | ----------- | ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Basic               | enabled                  | boolean     | 0                            | Enable/disable the `pbr` service.                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Basic               | verbosity                | integer     | 2                            | Can be set to 0, 1 or 2 to control the console and system log output verbosity of the `pbr` service.                                                                                                                                                                                                                                                                                                                                                           |
| Basic               | strict_enforcement       | boolean     | 1                            | Enforce policies when their interface is down. See [Strict enforcement](#strict-enforcement) for more details.                                                                                                                                                                                                                                                                                                                                                 |
| Basic               | resolver_set             | string      | dnsmasq.ipset dnsmasq.nftset | Enable/disable use of the resolver ipset option for domain-only remote policies (policies with only a domain as a remote address and no other fields set). This speeds up service start-up and operation. Currently supported options are `none`, `dnsmasq.ipset` and  `dnsmasq.nftset` (see  [Use Resolver's Set Support](#use-resolvers-set-support) and [<sup>#7</sup>](#footnote7) for more details). Make sure the [requirements](#requirements) are met. |
| Basic               | ipv6_enabled             | boolean     | 0                            | Enable/disable IPv6 support.                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Advanced            | supported_interface      | list/string |                              | Allows to specify the space-separated list of interface names (in lower case) to be explicitly supported by the `pbr` service. Can be useful if your OpenVPN tunnels have dev option other than tun\*.                                                                                                                                                                                                                                                         |
| Advanced            | ignored_interface        | list/string |                              | Allows to specify the space-separated list of interface names (in lower case) to be ignored by the `pbr` service. Can be useful if running both VPN server and VPN client on the router.                                                                                                                                                                                                                                                                       |
| Advanced            | boot_timeout             | number      | 30                           | Allows to specify the time (in seconds) for `pbr` service to wait for WAN gateway discovery on boot. Can be useful on devices with ADSL modem built in.                                                                                                                                                                                                                                                                                                        |
| Advanced            | rule_create_option       | add/insert  | add                          | Allows to specify the parameter for rules: `add` for `-A` in `iptables` and `add` in `nft` and `insert` for `-I` in `iptables` and `insert` for `nft`. Add is generally speaking more compatible with other packages/firewall rules. Recommended to change to `insert` only to enable compatibility with the `mwan3` package on OpenWrt 21.02 and earlier.                                                                                                     |
| Advanced            | icmp_interface           | string      |                              | Set the default ICMP protocol interface (interface name in lower case). Use with caution.                                                                                                                                                                                                                                                                                                                                                                      |
| Advanced            | wan_tid                  | integer     | 201                          | Starting (WAN) Table ID number for tables created by the `pbr` service.                                                                                                                                                                                                                                                                                                                                                                                        |
| Advanced            | wan_mark                 | hexadecimal | 0x010000                     | Starting (WAN) fw mark for marks used by the `pbr` service. High starting mark is used to avoid conflict with SQM/QoS, this can be changed by user. Change with caution together with `fw_mask`.                                                                                                                                                                                                                                                               |
| Advanced            | fw_mask                  | hexadecimal | 0xff0000                     | FW Mask used by the `pbr` service. High mask is used to avoid conflict with SQM/QoS, this can be changed by user. Change with caution together with `wan_mark`.                                                                                                                                                                                                                                                                                                |
| Hidden/Experimental | secure_reload            | boolean     | 0                            | When enabled, kills router traffic during service start/restart/reload operations to prevent traffic leaks on unwanted interface.                                                                                                                                                                                                                                                                                                                              |
| Web UI              | webui_supported_protocol | list        | 0                            | List of protocols to display in the `Protocol` column for policies.                                                                                                                                                                                                                                                                                                                                                                                            |
|                     | wan_dscp                 | integer     |                              | Allows use of [DSCP-tag based policies](#dscp-tag-based-policies) for WAN interface.                                                                                                                                                                                                                                                                                                                                                                           |
|                     | {interface_name}\_dscp   | integer     |                              | Allows use of [DSCP-tag based policies](#dscp-tag-based-policies) for a VPN interface.                                                                                                                                                                                                                                                                                                                                                                         |
|                     | procd_reload_delay       | integer     | 0                            | Time (in seconds) for PROCD_RELOAD_DELAY parameter.                                                                                                                                                                                                                                                                                                                                                                                                            |

### Default Settings

Default configuration has service disabled (use Web UI to enable/start service or run `uci set pbr.config.enabled=1; uci commit pbr;`).

### Policy Options

Each policy may have a combination of the options below, the `name` and `interface`  options are required.

The `src_addr`, `src_port`, `dest_addr` and `dest_port` options supports parameter negation, for example if you want to **exclude** remote port 80 from the policy, set `dest_port` to `"!80"` (notice lack of space between `!` and parameter).

| Option        | Default    | Description                                                                                                                                                                                                      |
| ------------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **name**      |            | Policy name, it **must** be set.                                                                                                                                                                                 |
| enabled       | 1          | Enable/disable policy. To display the `Enable` checkbox column for policies in the WebUI, make sure to select `Enabled` for `Show Enable Column` in the `Web UI` tab.                                            |
| **interface** |            | Policy interface, it **must** be set.                                                                                                                                                                            |
| src_addr      |            | List of space-separated local/source IP addresses, CIDRs, hostnames or mac addresses (colon-separated). You can also specify a local physical device (like a specially created wlan) prepended by an `@` symbol. |
| src_port      |            | List of space-separated local/source ports or port-ranges.                                                                                                                                                       |
| dest_addr     |            | List of space-separated remote/target IP addresses, CIDRs or hostnames/domain names.                                                                                                                             |
| dest_port     |            | List of space-separated remote/target ports or port-ranges.                                                                                                                                                      |
| proto         | auto       | Policy protocol, can be any valid protocol from `/etc/protocols` for CLI/uci or can be selected from the values set in `webui_supported_protocol`.                                                               |
| chain         | prerouting | Policy chain, can be either `forward`, `input`, `prerouting`, `postrouting` or `output`. This setting is case-sensitive.                                                                                         |

### Custom User Files Include Options

| Option   | Default | Description                                                                 |
| -------- | ------- | --------------------------------------------------------------------------- |
| **path** |         | Path to a custom user file (in a form of shell script), it **must** be set. |
| enabled  | 1       | Enable/disable setting.                                                     |

### Example Policies

#### Single IP, IP Range, Local Machine, Local MAC Address

The following policies route traffic from a single IP address, a range of IP addresses, a local machine (requires definition as DHCP host record in DHCP config), a MAC-address of a local device and finally all of the above via WAN.

```text
config policy
  option name 'Local IP'
  option interface 'wan'
  option src_addr '192.168.1.70'

config policy
  option name 'Local Subnet'
  option interface 'wan'
  option src_addr '192.168.1.81/29'

config policy
  option name 'Local Machine'
  option interface 'wan'
  option src_addr 'dell-ubuntu'

config policy
  option name 'Local MAC Address'
  option interface 'wan'
  option src_addr '00:0F:EA:91:04:08'

config policy
  option name 'Local Devices'
  option interface 'wan'
  option src_addr '192.168.1.70 192.168.1.81/29 dell-ubuntu 00:0F:EA:91:04:08'
  
```

#### Logmein Hamachi

The following policy routes LogMeIn Hamachi zero-setup VPN traffic via WAN.

```text
config policy
  option name 'LogmeIn Hamachi'
  option interface 'wan'
  option dest_addr '25.0.0.0/8 hamachi.cc hamachi.com logmein.com'
```

#### SIP Port

The following policy routes standard SIP port traffic via WAN for both TCP and UDP protocols.

```text
config policy
  option name 'SIP Ports'
  option interface 'wan'
  option dest_port '5060'
  option proto 'tcp udp'
```

#### Plex Media Server

The following policies route Plex Media Server traffic via WAN. Please note, you'd still need to open the port in the firewall either manually or with the UPnP.

```text
config policy
  option name 'Plex Local Server'
  option interface 'wan'
  option src_port '32400'

config policy
  option name 'Plex Remote Servers'
  option interface 'wan'
  option dest_addr 'plex.tv my.plexapp.com'
```

#### Emby Media Server

The following policy route Emby traffic via WAN. Please note, you'd still need to open the port in the firewall either manually or with the UPnP.

```text
config policy
  option name 'Emby Local Server'
  option interface 'wan'
  option src_port '8096 8920'

config policy
  option name 'Emby Remote Servers'
  option interface 'wan'
  option dest_addr 'emby.media app.emby.media tv.emby.media'
```

#### Ignore Requests (replace `append_src_rules`)

Since the `append_src_rules` option is no longer supported in pbr from version 0.3.x forward, replace:

```text
config pbr 'config'
  ...
  append_src_rules '! -d 192.168.200.0/24'
```

With the following policy allowing you to skip processing of some requests (like traffic to an OpenVPN or Wireguard server running on the router):

```text
config pbr 'config'
  ...
  option webui_show_ignore_target '1'

config policy
  option name 'Ignore Local Requests by Destination'
  option interface 'ignore'
  option dest_addr '192.168.200.0/24'
```

It's a good idea to keep the policies targeting `ignore` interface at the top of the config file/list of policies displayed in WebUI to make sure they are processed first.

#### Ignore Requests (replace `append_dest_rules`)

Since the `append_dest_rules` option is no longer supported in pbr from version 0.3.x forward, replace:

```text
config pbr 'config'
  ...
  append_dest_rules '! -s 192.168.1.1/24'
```

With the following policy allowing you to skip processing of some requests:

```text
config pbr 'config'
  ...
  option webui_show_ignore_target '1'

config policy
  option name 'Ignore Local Requests by Source'
  option interface 'ignore'
  option src_addr '192.168.1.1/24'
```

It's a good idea to keep the policies targeting `ignore` interface at the top of the config file/list of policies displayed in WebUI to make sure they are processed first.

#### Local OpenVPN Server + OpenVPN Client (Scenario 1)

If the OpenVPN client on your router is used as default routing (for the whole internet), make sure your settings are as following (three dots on the line imply other options can be listed in the section as well).

Relevant part of `/etc/config/pbr`:

```text
config pbr 'config'
  list ignored_interface 'vpnserver'
  ...

config policy
  option name 'OpenVPN Server'
  option interface 'wan'
  option proto 'tcp'
  option src_port '1194'
  option chain 'output'
```

The network/firewall/openvpn settings are below.

Relevant part of `/etc/config/network` (**DO NOT** modify default OpenWrt network settings for neither `wan` nor `lan`):

```text
config interface 'vpnclient'
  option proto 'none'
  option ifname 'ovpnc0'

config interface 'vpnserver'
  option proto 'none'
  option ifname 'ovpns0'
  option auto '1'
```

Relevant part of `/etc/config/firewall` (**DO NOT** modify default OpenWrt firewall settings for neither `wan` nor `lan`):

```text
config zone
  option name 'vpnclient'
  option network 'vpnclient'
  option input 'REJECT'
  option forward 'ACCEPT'
  option output 'REJECT'
  option masq '1'
  option mtu_fix '1'

config forwarding
  option src 'lan'
  option dest 'vpnclient'

config zone
  option name 'vpnserver'
  option network 'vpnserver'
  option input 'ACCEPT'
  option forward 'REJECT'
  option output 'ACCEPT'
  option masq '1'

config forwarding
  option src 'vpnserver'
  option dest 'wan'

config forwarding
  option src 'vpnserver'
  option dest 'lan'

config forwarding
  option src 'vpnserver'
  option dest 'vpnclient'

config rule
  option name 'Allow-OpenVPN-Inbound'
  option target 'ACCEPT'
  option src '*'
  option proto 'tcp'
  option dest_port '1194'
```

Relevant part of `/etc/config/openvpn`:

```text
config openvpn 'vpnclient'
  option client '1'
  option dev_type 'tun'
  option dev 'ovpnc0'
  option proto 'udp'
  option remote 'some.domain.com 1197' # DO NOT USE PORT 1194 for VPN Client
  ...

config openvpn 'vpnserver'
  option port '1194'
  option proto 'tcp'
  option server '192.168.200.0 255.255.255.0'
  ...
```

#### Local OpenVPN Server + OpenVPN Client (Scenario 2)

If the OpenVPN client is **not** used as default routing and you create policies to selectively use the OpenVPN client, make sure your settings are as following (three dots on the line imply other options can be listed in the section as well). Make sure that the policy mentioned below is at the top of your policies list.

Relevant part of `/etc/config/pbr`:

```text
config pbr 'config'
  list ignored_interface 'vpnserver'
  ...
config policy
  option name 'Ignore Local Traffic'
  option interface 'ignore'
  option dest_addr '192.168.200.0/24'
  ...
```

The network/firewall/openvpn settings are below.

Relevant part of `/etc/config/network` (**DO NOT** modify default OpenWrt network settings for neither `wan` nor `lan`):

```text
config interface 'vpnclient'
  option proto 'none'
  option ifname 'ovpnc0'

config interface 'vpnserver'
  option proto 'none'
  option ifname 'ovpns0'
  option auto '1'
```

Relevant part of `/etc/config/firewall` (**DO NOT** modify default OpenWrt firewall settings for neither `wan` nor `lan`):

```text
config zone
  option name 'vpnclient'
  option network 'vpnclient'
  option input 'REJECT'
  option forward 'ACCEPT'
  option output 'REJECT'
  option masq '1'
  option mtu_fix '1'

config forwarding
  option src 'lan'
  option dest 'vpnclient'

config zone
  option name 'vpnserver'
  option network 'vpnserver'
  option input 'ACCEPT'
  option forward 'REJECT'
  option output 'ACCEPT'
  option masq '1'

config forwarding
  option src 'vpnserver'
  option dest 'wan'

config forwarding
  option src 'vpnserver'
  option dest 'lan'

config forwarding
  option src 'vpnserver'
  option dest 'vpnclient'

config rule
  option name 'Allow-OpenVPN-Inbound'
  option target 'ACCEPT'
  option src '*'
  option proto 'tcp'
  option dest_port '1194'
```

Relevant part of `/etc/config/openvpn`:

```text
config openvpn 'vpnclient'
  option client '1'
  option dev_type 'tun'
  option dev 'ovpnc0'
  option proto 'udp'
  option remote 'some.domain.com 1197' # DO NOT USE PORT 1194 for VPN Client
  list pull_filter 'ignore "redirect-gateway"' # for OpenVPN 2.4 and later
  option route_nopull '1' # for OpenVPN earlier than 2.4
  ...

config openvpn 'vpnserver'
  option port '1194'
  option proto 'tcp'
  option server '192.168.200.0 255.255.255.0'
  ...
```

#### Local Wireguard Server + Wireguard Client (Scenario 1)

Yes, I'm aware that technically there are no clients nor servers in Wireguard, it's all peers, but for the sake of README readability I will use the terminology similar to the OpenVPN Server + Client setups.

If the Wireguard tunnel on your router is used as default routing (for the whole internet), sadly no `pbr` rule will allow it to intercept and properly route the `UDP` traffic of Wireguard server, please either use the OpenVPN server and configure it to use `TCP` protocol or use the Scenario 2 below.

#### Local Wireguard Server + Wireguard Client (Scenario 2)

Yes, I'm aware that technically there are no clients nor servers in Wireguard, it's all peers, but for the sake of README readability I will use the terminology similar to the OpenVPN Server + Client setups.

If the Wireguard client is **not** used as default routing and you create policies to selectively use the Wireguard client, make sure your settings are as following (three dots on the line imply other options can be listed in the section as well). Make sure that the policy mentioned below is at the top of your policies list.

Relevant part of `/etc/config/pbr`:

```text
config pbr 'config'
  list ignored_interface 'wgserver'
  ...
config policy
  option name 'Ignore Local Traffic'
  option interface 'ignore'
  option dest_addr '192.168.200.0/24'
  ...
```

The recommended network/firewall settings are below.

Relevant part of `/etc/config/network` (**DO NOT** modify default OpenWrt network settings for neither `wan` nor `lan`):

```text
config interface 'wgclient'
  option proto 'wireguard'
  ...

config wireguard_wgclient
  list allowed_ips '0.0.0.0/0'
  list allowed_ips '::0/0'
  option endpoint_port '51820'
  ...

config interface 'wgserver'
  option proto 'wireguard'
  option listen_port '61820'
  list addresses '192.168.200.1/24'
  ...

config wireguard_wgserver
  list allowed_ips '192.168.200.2/32'
  option route_allowed_ips '1'
  ...
```

Relevant part of `/etc/config/firewall` (**DO NOT** modify default OpenWrt firewall settings for neither `wan` nor `lan`):

```text
config zone
  option name 'wgclient'
  option network 'wgclient'
  option input 'REJECT'
  option forward 'ACCEPT'
  option output 'REJECT'
  option masq '1'
  option mtu_fix '1'

config forwarding
  option src 'lan'
  option dest 'wgclient'

config zone
  option name 'wgserver'
  option network 'wgserver'
  option input 'ACCEPT'
  option forward 'REJECT'
  option output 'ACCEPT'
  option masq '1'

config forwarding
  option src 'wgserver'
  option dest 'wan'

config forwarding
  option src 'wgserver'
  option dest 'lan'

config forwarding
  option src 'wgserver'
  option dest 'wgclient'

config rule
  option name 'Allow-WG-Inbound'
  option target 'ACCEPT'
  option src '*'
  option proto 'udp'
  option dest_port '61820'
```

#### Netflix Domains

The following policy should route US Netflix traffic via WAN. For capturing international Netflix domain names, you can refer to the getdomainnames.sh-specific instructions on [GitHub](https://github.com/Xentrk/netflix-vpn-bypass/blob/master/README.md#ipset_netflix_domainssh)/[jsDelivr](https://cdn.jsdelivr.net/gh/Xentrk/netflix-vpn-bypass/README.md#ipset_netflix_domainssh) and don't forget to adjust them for OpenWrt. This may not work if Netflix changes things. For more reliable US Netflix routing you may want to consider also using [custom user files](#custom-user-files).

```text
config policy
  option name 'Netflix Domains'
  option interface 'wan'
  option dest_addr 'amazonaws.com netflix.com nflxext.com nflximg.net nflxso.net nflxvideo.net dvd.netflix.com'
```

#### Example Custom User Files Includes

```text
config include
  option path '/usr/share/pbr/pbr.user.netflix'

config include
  option path '/usr/share/pbr/pbr.user.aws'
```

#### Basic OpenVPN Client Config

There are multiple guides online on how to configure the OpenVPN client on OpenWrt "the easy way", and they usually result either in a kill-switch configuration or configuration where the OpenVPN tunnel  cannot be properly (and separately from WAN) routed, either way, incompatible with the VPN Policy-Based Routing.

Below is the sample OpenVPN client configuration for OpenWrt which is guaranteed to work. If you have already deviated from the instructions below (ie: made any changes to any of the `wan` or `lan` configurations in either `/etc/config/network` or `/etc/config/firewall`), you will need to start from scratch with a fresh OpenWrt install.

Relevant part of `/etc/config/pbr`:

```text
config pbr 'config'
  list supported_interface 'vpnclient'
  ...
```

The recommended network/firewall settings are below.

Relevant part of `/etc/config/network` (**DO NOT** modify default OpenWrt network settings for neither `wan` nor `lan`):

```text
config interface 'vpnclient'
  option proto 'none'
  option ifname 'ovpnc0'
```

Relevant part of `/etc/config/firewall` (**DO NOT** modify default OpenWrt firewall settings for neither `wan` nor `lan`):

```text
config zone
  option name 'vpnclient'
  option network 'vpnclient'
  option input 'REJECT'
  option forward 'REJECT'
  option output 'ACCEPT'
  option masq '1'
  option mtu_fix '1'

config forwarding
  option src 'lan'
  option dest 'vpnclient'
```

If you have a Guest Network, add the following to the `/etc/config/firewall`:

```text
config forwarding
  option src 'guest'
  option dest 'vpnclient'
```

Relevant part of `/etc/config/openvpn` (configure the rest of the client connection for your specifics by either referring to an existing `.ovpn` file or thru the OpenWrt uci settings):

```text
config openvpn 'vpnclient'
  option enabled '1'
  option client '1'
  option dev_type 'tun'
  option dev 'ovpnc0'
  ...
```

#### Multiple OpenVPN Clients

If you use multiple OpenVPN clients on your router, the order in which their devices are named (tun0, tun1, etc) is not guaranteed by OpenWrt. The following settings are recommended in this case.

For `/etc/config/network`:

```text
config interface 'vpnclient0'
  option proto 'none'
  option ifname 'ovpnc0'

config interface 'vpnclient1'
  option proto 'none'
  option ifname 'ovpnc1'
```

For `/etc/config/openvpn`:

```text
config openvpn 'vpnclient0'
  option client '1'
  option dev_type 'tun'
  option dev 'ovpnc0'
  ...

config openvpn 'vpnclient1'
  option client '1'
  option dev_type 'tun'
  option dev 'ovpnc1'
  ...
```

For `/etc/config/pbr`:

```text
config pbr 'config'
  list supported_interface 'vpnclient0 vpnclient1'
  ...
```

## Footnotes/Known Issues

1.  <a name="footnote1"> </a> See [note about multiple OpenVPN clients](#multiple-openvpn-clients).

2.  <a name="footnote2"> </a> If your `OpenVPN` interface has the device name different from tun\*, is not up and is not explicitly listed in `supported_interface` option, it may not be available in the policies `Interface` drop-down within WebUI.

3.  <a name="footnote3"> </a> If your default routing is set to the VPN tunnel, then the true WAN interface cannot be discovered using OpenWrt built-in functions, so service will assume your network interface ending with or starting with `wan` is the true WAN interface.

4.  <a name="footnote4"> </a> The service does **NOT** support the "killswitch" router mode (where if you stop the VPN tunnel, you have no internet connection). For proper operation, leave all the default OpenWrt `network` and `firewall` settings for `lan` and `wan` intact.

5.  <a name="footnote5"> </a> When using the `dnsmasq.ipset` or `dnsmasq.nftset` option, please make sure to flush the DNS cache of the local devices, otherwise domain policies may not work until you do. If you're not sure how to flush the DNS cache (or if the device/OS doesn't offer an option to flush its DNS cache), reboot your local devices when starting to use the service and/or when connecting data-capable device to your WiFi.

6.  <a name="footnote6"> </a> When using the policies targeting physical devices, you may need to make sure you have the following packages installed: `kmod-br-netfilter`, `kmod-ipt-physdev` and `iptables-mod-physdev`. Also, if your physical device is a part of the bridge, you may have to set `net.bridge.bridge-nf-call-iptables` to `1` in your `/etc/sysctl.conf`.

7.  <a name="footnote7"> </a> If you're using domain names in the `dest_addr` option of the policy, it is recommended to use `dnsmasq.ipset` or `dnsmasq.nftset` options. Otherwise, the domain name will be resolved when the service starts up and the resolved IP address(es) will be added to an apropriate set or an `iptables` or `nft` rule. Resolving a number of domains on start is a time consuming operation, while using the `dnsmasq.ipset` or `dnsmasq.nftset` options allows transparent and fast addition of the correct domain IP addresses to the apropriate set on DNS request or when `dnsmasq` is idle.

8.  <a name="footnote8"> </a> When service is started, it subscribes to the supported interfaces updates thru the PROCD. While I was never able to reproduce the issue, some customers report that this method doesn't always work in which case you may want to [set up iface hotplug script](#a-word-about-interface-hotplug-script) to reload service when relevant interface(s) are updated.

## FAQ

You may find some useful information in sections below.

### A Word About Default Routing

Service does not alter the default routing. Depending on your VPN tunnel settings (and settings of the VPN server you are connecting to), the default routing might be set to go via WAN or via VPN tunnel. This service affects only routing of the traffic matching the policies. If you want to override default routing, follow the instructions below.

#### OpenVPN tunnel configured via uci (/etc/config/openvpn)

To unset an OpenVPN tunnel as default route, set the following to the appropriate section of your `/etc/config/openvpn`:

-   For OpenVPN 2.4 and newer client config:

    ```text
    list pull_filter 'ignore "redirect-gateway"'
    ```

-   For OpenVPN 2.3 and older client config:

    ```text
    option route_nopull '1'
    ```

-   For your Wireguard (client) config:

    ```text
    option route_allowed_ips '0'
    ```

#### OpenVPN tunnel configured with .ovpn file

To unset an OpenVPN tunnel as default route, set the following to the appropriate section of your `.ovpn` file:

-   For OpenVPN 2.4 and newer client `.ovpn` file:

    ```text
    pull-filter ignore "redirect-gateway"
    ```

-   For OpenVPN 2.3 and older client `.ovpn` file:

    ```text
    route-nopull "1"
    ```

#### Wireguard tunnel

To unset a Wireguard tunnel as default route, set the following to the appropriate section of your `/etc/config/network`:

-   For your Wireguard (client) config:

    ```text
    option route_allowed_ips '0'
    ```

-   Routing Wireguard traffic may require setting `net.ipv4.conf.wg0.rp_filter = 2` in `/etc/sysctl.conf`. Please refer to [issue #41](https://github.com/stangri/source.openwrt.melmac.net/issues/41) for more details.

### A Word About Cloudflare's 1.1.1.1 App

Cloudflare has released an app for [iOS](https://itunes.apple.com/us/app/1-1-1-1-faster-internet/id1423538627) and [Android](https://play.google.com/store/apps/details?id=com.cloudflare.onedotonedotonedotone), which can also be configured to route traffic thru their own VPN tunnel (WARP+).

If you use Cloudlfare's VPN tunnel (WARP+), none of the policies you set up with the VPN Policy Routing will take effect on your mobile device. Disable WARP+ for your home WiFi to keep VPN Policy Routing affecting your mobile device.

If you just use the private DNS queries (WARP), [A Word About DNS-over-HTTPS](#a-word-about-dns-over-https) applies. You can also disable WARP for your home WiFi to keep VPN Policy Routing affecting your mobile device.

### A Word About DNS-over-HTTPS

Some browsers, like [Mozilla Firefox](https://support.mozilla.org/en-US/kb/firefox-dns-over-https#w_about-dns-over-https) or [Google Chrome/Chromium](https://blog.chromium.org/2019/09/experimenting-with-same-provider-dns.html) have [DNS-over-HTTPS proxy](https://en.wikipedia.org/wiki/DNS_over_HTTPS) built-in. Their requests to web-sites listed in policies cannot be properly routed if the `resolver_set` is set to either `dnsmasq.ipset` or `dnsmasq.nftset`. To fix this, you can try either of the following:

1.  Disable the DNS-over-HTTPS support in your browser and use the OpenWrt's `net/https-dns-proxy` (README on [GitHub](https://docs.openwrt.melmac.net/https-dns-proxy)/[jsDelivr](https://cdn.jsdelivr.net/gh/stangri/docs.openwrt.melmac.net/https-dns-proxy/)) package with optional `https-dns-proxy` WebUI/luci app. You can then continue to use either `dnsmasq.ipset` or `dnsmasq.nftset` setting for the `resolver_set` in Policy-Based Routing.

2.  Continue using DNS-over-HTTPS in your browser (which, by the way, also limits your options for router-level AdBlocking as described in `net/simple-adblock` README on [GitHub](https://docs.openwrt.melmac.net/simple-adblock/#dns-resolution-option)/[jsDelivr](https://cdn.jsdelivr.net/gh/stangri/docs.openwrt.melmac.net/simple-adblock/README.md#dns-resolution-option)), you than would either have to switch the `resolver_set` to `none`. Please note, you will lose all the benefits of [`dnsmasq.ipset`](#use-dnsmasq-ipset-support) option.

### A Word About HTTP/3 (QUICK)

If you want to target traffic using HTTP/3 protocol, you can use the `AUTO` as the protocol (the policy will be either protocol-agnostic or `TCP/UDP`) or explicitly   use `UDP` as a protocol.

### A Word About IPv6 Routing

Due to the nature of IPv6, it's not supposed to be routed same way as IPv4 with this package, but a fellow user has graciously contributed a [gist detailing their experience to get IPv6 routing working](https://gist.github.com/ByteAndNibble/3bd8413029b1f728c1f00bc1ac0e98b4).

### A Word About Routing Netflix/Amazon Prime/Hulu Traffic

There are two following scenarios with VPN connections and Netflix/Amazon Prime/Hulu traffic.

#### Routing Netflix/Amazon Prime/Hulu Traffic via VPN Tunnel

If you live in a country where Netflix/Amazon Prime/Hulu are not available and want to circumvent geo-fencing, this package can't help you. The Netflix/Amazon Prime/Hulu do a great job detecing VPN usage when accessing their services and circumventing geographical restrictions is not only dubiously legal, it's also technically very challenging.

#### Routing Netflix/Amazon Prime/Hulu Traffic via WAN

If you live in a country where Netflix/Amazon Prime/Hulu are available, you obviously do NOT want to use VPN tunnel for their traffic.

If the VPN tunnel is not used as a default gateway on your router, you should not have a problem accessing Netflix/Amazon Prime/Hulu (just make sure that your DNS requests are not routed via VPN tunnel either).

If the VPN tunnel is used as a default gateway, either:

-   send ALL traffic from your multimedia devices (by using their IP addresses or device names in the `src_addr` option in config file or Local addresses /devices field in WebUI) accessing Netflix/Amazon Prime/Hulu to WAN; this is the more reliable and recommended method.
-   use the [Netflix/AWS custom user files](https://docs.openwrt.melmac.net/pbr/#custom-user-files) in combination with the [Netflix](https://docs.openwrt.melmac.net/pbr/#netflix-domains)/Amazon Prime/Hulu domains and `dnsmasq.ipset` option to route traffic to Netflix/Amazon via WAN; this is definitely less reliable method and may not work in all regions.

Either way make sure that your DNS requests are not routed via VPN Tunnel!

### A Word About Interface Hotplug Script

Sometimes[<sup>#8</sup>](#footnote8) the service doesn't get reloaded when supported interfaces go up or down. This can be an annoying experience since the service may start before all supported VPN connections are up and then not get updated when the VPN connections get established. In that case, run the following command from CLI to create the interface hotplug script to cause the service to be reloaded in interface updates:

```sh
mkdir -p /etc/hotplug.d/iface/
cat << 'EOF' > /etc/hotplug.d/iface/70-pbr
#!/bin/sh
logger -t pbr "Reloading $INTERFACE due to $ACTION of $INTERFACE ($DEVICE)"
/etc/init.d/pbr reload_interface "$INTERFACE"
EOF
```

### A Word About Differences from `vpn-policy-routing`

Unlike the `vpn-policy-routing`, the `pbr` package:

-   creates set and resolver's set (like `dnsmasq.ipset` or `dnsmasq.nftset`)-based policies respecting policy's priority.
-   can only make minimal changes on a single interface reload (it does not reload the whole service).
-   can only reload policies on service reload (it does not reload the network-related parts).
-   implements the new `secure_reload` option to kill all traffic during the service start/restart/reload.
-   only supports OpenWrt 21.02 and newer
-   supports fw4 and nft/nft sets (and dnsmasq nft set support) on OpenWrt 22.03
-   chain names in config file should be in lower case, not upper case

### A Word About Migrating from `vpn-policy-routing`

The new `pbr` package is largely config-compatible with the `vpn-policy-routing` package and config file can be migrated by running:

```sh
if [ -x /etc/init.d/vpn-policy-routing ]; then
  /etc/init.d/vpn-policy-routing stop
  /etc/init.d/vpn-policy-routing disable
fi
if [ -s /etc/config/vpn-policy-routing ]; then
  sed 's/vpn-policy-routing/pbr/g' /etc/config/vpn-policy-routing > /etc/config/pbr
  sed -i 's/resolver_ipset/resolver_set/g' /etc/config/pbr
  sed -i 's/iptables_rule_option/rule_create_option/g' /etc/config/pbr
  sed -i "s/'FORWARD'/'forward'/g" /etc/config/pbr
  sed -i "s/'INPUT'/'input'/g" /etc/config/pbr
  sed -i "s/'OUTPUT'/'output'/g" /etc/config/pbr
  sed -i "s/'PREROUTING'/'prerouting'/g" /etc/config/pbr
  sed -i "s/'POSTROUTING'/'postrouting'/g" /etc/config/pbr
  uci set vpn-policy-routing.config.enabled=0; uci commit;
fi
```

## Getting Help

If things are not working as intended, please include the following in your post:

-   content of `/etc/config/dhcp`
-   content of `/etc/config/firewall`
-   content of `/etc/config/network`
-   content of `/etc/config/pbr`
-   the output of `/etc/init.d/pbr support`
-   the output of `/etc/init.d/pbr reload` with verbosity setting set to 2

If you don't want to post the `/etc/init.d/pbr support` output in a public forum, there's a way to have the support details automatically uploaded to my account at paste.ee by running: `/etc/init.d/pbr support -p`. You need to have the following packages installed to enable paste.ee upload functionality: `curl libopenssl ca-bundle`.

WARNING: while paste.ee uploads are unlisted/not indexed at the web-site, they are still publicly available.

### First Troubleshooting Step

If your router is set to use [default routing via VPN tunnel](#a-word-about-default-routing) and the WAN-targeting policies do not work, you need to stop your VPN tunnel first and ensure that you still have internet connection. If your router is set up to use the default routing via VPN tunnel and when you stop the VPN tunnel you have no internet connection, this package can't help you. You first need to make sure that you do have internet connection when the VPN tunnel is stopped.

## Thanks

I'd like to thank everyone who helped create, test and troubleshoot this service. Without contributions from [@hnyman](https://github.com/hnyman), [@dibdot](https://github.com/dibdot), [@danrl](https://github.com/danrl), [@tohojo](https://github.com/tohojo), [@cybrnook](https://github.com/cybrnook), [@nidstigator](https://github.com/nidstigator), [@AndreBL](https://github.com/AndreBL), [@dz0ny](https://github.com/dz0ny), [@tew42](https://github.com/tew42), rigorous testing/bugreporting by [@dziny](https://github.com/dziny), [@bluenote73](https://github.com/bluenote73), [@buckaroo](https://github.com/pgera), [@Alexander-r](https://github.com/Alexander-r), [@n8v8R](https://github.com/n8v8R), [psherman](https://forum.openwrt.org/u/psherman), [@Vale-max](https://github.com/Vale-max), [@ByteAndNibble](https://github.com/ByteAndNibble), [dscpl](https://forum.openwrt.org/u/dscpl) multiple contributions from [@dl12345](https://github.com/dl12345) and [trendy](https://forum.openwrt.org/u/trendy) and feedback from other OpenWrt users it wouldn't have been possible. Wireguard/IPv6 support is courtesy of [IVPN](https://www.ivpn.net/).

<!-- markdownlint-disable MD033 -->

<script defer src='https://static.cloudflareinsights.com/beacon.min.js' data-cf-beacon='{"token": "911798f2c34b45338f8f8182830a3eb6"}'></script>
