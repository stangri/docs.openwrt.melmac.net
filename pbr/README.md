<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD030 -->
<!-- markdownlint-disable MD033 -->

# Policy-Based Routing

<!-- vscode-markdown-toc -->

- 1. [OpenWrt 23.05.0 release and this package](#OpenWrt23.05.0releaseandthispackage)
- 2. [OpenWrt 22.03.0 release and this package](#OpenWrt22.03.0releaseandthispackage)
- 3. [Description](#Description)
- 4. [Features](#Features)
  - 4.1. [Gateways/Tunnels](#GatewaysTunnels)
  - 4.2. [IPv4/IPv6/Port-Based Policies](#IPv4IPv6Port-BasedPolicies)
  - 4.3. [Domain-Based Policies](#Domain-BasedPolicies)
  - 4.4. [Physical Device Policies](#PhysicalDevicePolicies)
  - 4.5. [DSCP Tag-Based Policies](#DSCPTag-BasedPolicies)
  - 4.6. [Custom User Files](#CustomUserFiles)
  - 4.7. [Strict Enforcement](#StrictEnforcement)
  - 4.8. [Use Resolver's Set Support](#UseResolversSetSupport)
    - 4.8.1. [Use AdGuardHome ipset Support](#UseAdGuardHomeipsetSupport)
    - 4.8.2. [Use DNSMASQ ipset Support](#UseDNSMASQipsetSupport)
    - 4.8.3. [Use DNSMASQ nft sets Support](#UseDNSMASQnftsetsSupport)
  - 4.9. [Customization](#Customization)
  - 4.10. [Other Features](#OtherFeatures)
- 5. [Screenshots (luci-app-pbr)](#Screenshotsluci-app-pbr)
- 6. [How It Works](#HowItWorks)
  - 6.1. [How It Works (`nft` mode)](#HowItWorksnftmode)
  - 6.2. [How It Works (`iptables`/`ipset` mode)](#HowItWorksiptablesipsetmode)
  - 6.3. [Processing Policies](#ProcessingPolicies)
    - 6.3.1. [Processing Policies (`nft` mode)](#ProcessingPoliciesnftmode)
    - 6.3.2. [Processing Policies (`iptables`/`ipset` mode)](#ProcessingPoliciesiptablesipsetmode)
  - 6.4. [Policies Priorities](#PoliciesPriorities)
  - 6.5. [Processing Custom User Files](#ProcessingCustomUserFiles)
    - 6.5.1. [Processing Custom User Files (`nft` mode)](#ProcessingCustomUserFilesnftmode)
    - 6.5.2. [Processing Custom User Files (`iptables`/`ipset` mode)](#ProcessingCustomUserFilesiptablesipsetmode)
- 7. [How To Install](#HowToInstall)
  - 7.1. [How To Install - OpenWrt 22.03 and newer](#HowToInstall-OpenWrt22.03andnewer)
  - 7.2. [How To Install - OpenWrt 21.02 and older](#HowToInstall-OpenWrt21.02andolder)
  - 7.3. [Requirements](#Requirements)
    - 7.3.1. [Requirements (pbr)](#Requirementspbr)
    - 7.3.2. [Requirements (pbr-iptables)](#Requirementspbr-iptables)
  - 7.4. [How to install legacy iptables/ipset packages](#Howtoinstalllegacyiptablesipsetpackages)
  - 7.5. [How to install dnsmasq-full](#Howtoinstalldnsmasq-full)
  - 7.6. [Unmet dependencies](#Unmetdependencies)
- 8. [How to use](#Howtouse)
  - 8.1. [Helpful Instructional Videos](#HelpfulInstructionalVideos)
  - 8.2. [Service Configuration Settings](#ServiceConfigurationSettings)
    - 8.2.1. [Default Settings](#DefaultSettings)
    - 8.2.2. [Policy Options](#PolicyOptions)
    - 8.2.3. [Custom User Files Include Options](#CustomUserFilesIncludeOptions)
  - 8.3. [Example Policies](#ExamplePolicies)
    - 8.3.1. [Single IP, IP Range, Local Machine, Local MAC Address](#SingleIPIPRangeLocalMachineLocalMACAddress)
    - 8.3.2. [Logmein Hamachi](#LogmeinHamachi)
    - 8.3.3. [SIP Port](#SIPPort)
    - 8.3.4. [Plex Media Server](#PlexMediaServer)
    - 8.3.5. [Emby Media Server](#EmbyMediaServer)
    - 8.3.6. [Ignore Target](#IgnoreTarget)
    - 8.3.7. [Basic OpenVPN Client Config](#BasicOpenVPNClientConfig)
    - 8.3.8. [Multiple OpenVPN Clients](#MultipleOpenVPNClients)
    - 8.3.9. [Local OpenVPN Server + OpenVPN Client (Scenario 1)](#LocalOpenVPNServerOpenVPNClientScenario1)
    - 8.3.10. [Local OpenVPN Server + OpenVPN Client (Scenario 2)](#LocalOpenVPNServerOpenVPNClientScenario2)
    - 8.3.11. [Local WireGuard Server + WireGuard Client (Scenario 1)](#LocalWireGuardServerWireGuardClientScenario1)
    - 8.3.12. [Local WireGuard Server + WireGuard Client (Scenario 2)](#LocalWireGuardServerWireGuardClientScenario2)
    - 8.3.13. [Local WireGuard Server + Another VPN Client (Scenario 1)](#LocalWireGuardServerAnotherVPNClientScenario1)
    - 8.3.14. [Local WireGuard Server + Another VPN Client (Scenario 2)](#LocalWireGuardServerAnotherVPNClientScenario2)
    - 8.3.15. [Netflix Domains](#NetflixDomains)
    - 8.3.16. [Example Custom User Files Includes](#ExampleCustomUserFilesIncludes)
- 9. [Footnotes/Known Issues](#FootnotesKnownIssues)
- 10. [FAQ](#FAQ)
  - 10.1. [A Word About Default Routing](#AWordAboutDefaultRouting)
    - 10.1.1. [OpenVPN tunnel configured via uci (/etc/config/openvpn)](#OpenVPNtunnelconfiguredviaucietcconfigopenvpn)
    - 10.1.2. [OpenVPN tunnel configured with .ovpn file](#OpenVPNtunnelconfiguredwith.ovpnfile)
    - 10.1.3. [WireGuard tunnel](#WireGuardtunnel)
  - 10.2. [A Word About Cloudflare's 1.1.1.1 App](#AWordAboutCloudflares1.1.1.1App)
  - 10.3. [A Word About DNS-over-HTTPS](#AWordAboutDNS-over-HTTPS)
  - 10.4. [A Word About HTTP/3 (QUIC)](#AWordAboutHTTP3QUIC)
  - 10.5. [A Word About IPv6 Routing](#AWordAboutIPv6Routing)
  - 10.6. [A Word About Routing Netflix/Amazon Prime/Hulu Traffic](#AWordAboutRoutingNetflixAmazonPrimeHuluTraffic)
    - 10.6.1. [Routing Netflix/Amazon Prime/Hulu Traffic via VPN Tunnel](#RoutingNetflixAmazonPrimeHuluTrafficviaVPNTunnel)
    - 10.6.2. [Routing Netflix/Amazon Prime/Hulu Traffic via WAN](#RoutingNetflixAmazonPrimeHuluTrafficviaWAN)
  - 10.7. [A Word About Interface Hotplug Script](#AWordAboutInterfaceHotplugScript)
  - 10.8. [A Word About Differences from `vpn-policy-routing`](#AWordAboutDifferencesfromvpn-policy-routing)
  - 10.9. [A Word About Migrating from `vpn-policy-routing`](#AWordAboutMigratingfromvpn-policy-routing)
- 11. [Getting Help](#GettingHelp)
  - 11.1. [First Troubleshooting Step](#FirstTroubleshootingStep)
- 12. [Thanks](#Thanks)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

## 1. <a name='OpenWrt23.05.0releaseandthispackage'></a>OpenWrt 23.05.0 release and this package

Unlike the previous release, the OpenWrt 23.05 includes `dnsmasq-full` which supports nft sets, so just [install dnsmasq-full](#Howtoinstalldnsmasq-full), install the `pbr` package and configure `pbr` to use `dnsmasq.nftset` option for resolver.

## 2. <a name='OpenWrt22.03.0releaseandthispackage'></a>OpenWrt 22.03.0 release and this package

There are now two packages of this service available:

- `pbr` which supports fw4, nft, nft sets and `dnsmasq.nftset` option (but because OpenWrt's `dnsmasq` doesn't support nft sets yet, you can't use `dnsmasq` to resolve domain names from policies) as well as fw3, iptables, ipset and `dnsmasq.ipset` option.
- `pbr-iptables` which supports fw3, iptables, ipset and `dnsmasq.ipset` option.

Both packages install the same init script (what you actually run when you invoke `service pbr ...` or `/etc/init.d/pbr ...`), however both packages install some specific files and `pbr` can run in either `nft` or `iptables`/`ipset` mode, whereas `pbr-iptables` can only run in `iptables`/`ipset` mode.

The package-specific files that `pbr` installs are:

- the `/etc/config/pbr` file with the `resolver_set` set to `none` (will be switched to `dnsmasq.nftset` when OpenWrt's `dnsmasq` supports it)
- the `fw4`-specific `nft` scripts (installed into `/usr/share/nftables.d/`) to set up default service chains as part of the fw4 start/restart/reload processes

The package-specific files that `pbr-iptables` installs are:

- the `/etc/config/pbr` file with the `resolver_set` set to `dnsmasq.ipset`
- legacy iptables/ipset packages

The `pbr` decides when to use `iptables`/`ipset` mode or `nft` mode at run time. If the `nft` binary is available, the `resolver_set` is not set to `dnsmasq.ipset` and a main `pbr_prerouting` chain has been created by the `fw4`-specific `nft` script, it runs in the `nft` mode, otherwise it runs in the `iptables`/`ipset` mode.

Each package of the service has its own dependencies, so only `pbr-iptables` can be installed on OpenWrt 21.02 and earlier, but either `pbr` or `pbr-iptables` can be installed on OpenWrt 22.03. It is recommended to install `pbr` on OpenWrt 22.03 and if you want to use [use dnsmasq ipset support](#UseDNSMASQipsetSupport), [install dnsmasq-full](#Howtoinstalldnsmasq-full), also [install legacy iptables/ipset packages](#Howtoinstalllegacyiptablesipsetpackages) and then change `resolver_set` option to `dnsmasq.ipset` to force `iptables`/`ipset` mode.

Both `pbr-iptables` and `pbr` in `iptables`/`ipset` mode work just fine on OpenWrt 22.03. You can safely ignore the warning on the Status -> Firewall page about legacy iptables rules created by either package.

## 3. <a name='Description'></a>Description

This service allows you to define rules (policies) for routing traffic via WAN or your L2TP, OpenConnect, OpenVPN, PPTP, Softether or WireGuard tunnels. Policies can be set based on any combination of local/remote ports, local/remote IPv4 or IPv6 addresses/subnets or domains. This service supersedes and obsoletes the `VPN Bypass` ([README at GitHub](https://docs.openwrt.melmac.net/vpnbypass/)/[README at jsDelivr](https://cdn.jsdelivr.net/gh/stangri/docs.openwrt.melmac.net/vpnbypass/README.md)) and `VPN Policy Routing` ([README at GitHub](https://docs.openwrt.melmac.net/vpn-policy-routing/)/[README at jsDelivr](https://cdn.jsdelivr.net/gh/stangri/docs.openwrt.melmac.net/vpn-policy-routing/README.md)) services. For information on transition from `vpn-policy-routing` package please read about [differences](#AWordAboutDifferencesfromvpn-policy-routing) and [migration](#AWordAboutMigratingfromvpn-policy-routing).

## 4. <a name='Features'></a>Features

### 4.1. <a name='GatewaysTunnels'></a>Gateways/Tunnels

- Any policy can target either WAN or a VPN tunnel interface.
- L2TP tunnels supported (with protocol names l2tp\*).
- OpenConnect tunnels supported (with protocol names openconnect\*).
- OpenVPN tunnels supported (with device names tun\*).[<sup>#1</sup>](#footnote1) [<sup>#2</sup>](#footnote2)
- PPTP tunnels supported (with protocol names pptp\*).
- Tailscale tunnels supported (with device name tailscale\*).
- Tor tunnels supported in nft mode only (interface name must match tor).
- WireGuard tunnels supported (with protocol names wireguard\*).

### 4.2. <a name='IPv4IPv6Port-BasedPolicies'></a>IPv4/IPv6/Port-Based Policies

- Policies based on local names, IPs or subnets. You can specify a single IP (as in `192.168.1.70`) or a local subnet (as in `192.168.1.81/29`) or a local device name (as in `nexusplayer`). IPv6 addresses are also supported.
- Policies based on local ports numbers. Can be set as an individual port number (`32400`), a range (`5060-5061`), a space-separated list (`80 8080`) or a combination of the above (`80 8080 5060-5061`). Limited to 15 space-separated entries per policy.
- Policies based on remote IPs/subnets or domain names. Same format/syntax as local IPs/subnets.
- Policies based on remote ports numbers. Same format/syntax and restrictions as local ports.
- You can mix the IP addresses/subnets and device (or domain) names in one field separating them by a space (like this: `66.220.2.74 he.net tunnelbroker.net`).
- See [Policy Options](#PolicyOptions) section for more information.

### 4.3. <a name='Domain-BasedPolicies'></a>Domain-Based Policies

- Policies based on (remote) domain names can be processed in different ways. Please review the [Policy Options](#PolicyOptions) section and [Footnotes/Known Issues](#FootnotesKnownIssues) section, specifically [<sup>#5</sup>](#footnote5) and any other information in that section relevant to domain-based routing/DNS.

### 4.4. <a name='PhysicalDevicePolicies'></a>Physical Device Policies

- Policies based on a local physical device (like a specially created wlan). Please review the [Policy Options](#PolicyOptions) section and [Footnotes/Known Issues](#FootnotesKnownIssues) section, specifically [<sup>#6</sup>](#footnote6) and any other information in that section relevant to handling physical device.

### 4.5. <a name='DSCPTag-BasedPolicies'></a><a name='dscp-tag-based-policies'></a>DSCP Tag-Based Policies

You can also set policies for traffic with a specific DSCP tag. On Windows 10, for example, you can mark traffic from specific apps with DSCP tags (instructions for tagging specific app traffic in Windows 10 can be found [here](http://serverfault.com/questions/769843/cannot-set-dscp-on-windows-10-pro-via-group-policy)).

### 4.6. <a name='CustomUserFiles'></a><a name='custom-user-files'></a>Custom User Files

If the custom user file includes are set, the service will load and execute them after setting up routing and the sets and processing policies. This allows, for example, to add large numbers of domains/IP addresses to ipsets or nft sets without manually adding all of them to the config file.

The following custom user files are provided:

- `/etc/pbr/pbr.user.aws`: provided to pull the Continental US AWS IPv4 addresses into the WAN IPv4 sets that the service sets up.
- `/etc/pbr/pbr.user.netflix`: provided to pull the Continental US Netflix IPv4 addresses into the WAN IPv4 sets that the service sets up.
- `/etc/pbr/pbr.user.wg_server_and_client`: provided to overcome the protocol limitations (see [Local WireGuard Server + WireGuard Client (Scenario 1)](#LocalWireGuardServerWireGuardClientScenario1)), to allow running a WireGuard "server" on your router when a WireGuard "client" is set up as default routing.

If you want to create your own custom user files, please refer to [Processing Custom User Files](#ProcessingCustomUserFiles).

### 4.7. <a name='StrictEnforcement'></a><a name='strict-enforcement'></a>Strict Enforcement

- Supports strict policy enforcement, even if the policy interface is down -- resulting in network being unreachable for specific policy (enabled by default).

### 4.8. <a name='UseResolversSetSupport'></a><a name='use-resolvers-set-support'></a>Use Resolver's Set Support

- If supported on the system, service can be set to utilize resolver's set support. Currently supported resolver's set options are listed below.

#### 4.8.1. <a name='UseAdGuardHomeipsetSupport'></a>Use AdGuardHome ipset Support

- Either version of the service package can be configured to utilize `adguardhome`'s `ipset` support, which requires the `AdGuardHome` package version `0.107.13` or higher to be installed. This significantly improves the start up time because `adguardhome` resolves the domain names and adds them to the appropriate `ipset` in background. `adguardhome`'s `ipset` also automatically adds third-level domains to the `ipset`: if `domain.com` is added to the policy, this policy will affect all `*.domain.com` subdomains. This also works for top-level domains (TLDs) as well, a policy targeting the `at` TLD for example, will affect all the `*.at` domains.
- Please review the [Footnotes/Known Issues](#FootnotesKnownIssues) section, specifically [<sup>#5</sup>](#footnote5) and [<sup>#7</sup>](#footnote7) and any other information in that section relevant to domain-based routing/DNS.

#### 4.8.2. <a name='UseDNSMASQipsetSupport'></a>Use DNSMASQ ipset Support

- Either version of the service package can be configured to utilize `dnsmasq`'s `ipset` support, which requires the `dnsmasq-full` package with `ipset` support to be installed (see [How to install dnsmasq-full](#Howtoinstalldnsmasq-full)). This significantly improves the start up time because `dnsmasq` resolves the domain names and adds them to the appropriate `ipset` in background. `dnsmasq`'s `ipset` also automatically adds third-level domains to the `ipset`: if `domain.com` is added to the policy, this policy will affect all `*.domain.com` subdomains. This also works for top-level domains (TLDs) as well, a policy targeting the `at` TLD for example, will affect all the `*.at` domains.
- Please review the [Footnotes/Known Issues](#FootnotesKnownIssues) section, specifically [<sup>#5</sup>](#footnote5) and [<sup>#7</sup>](#footnote7) and any other information in that section relevant to domain-based routing/DNS.

#### 4.8.3. <a name='UseDNSMASQnftsetsSupport'></a>Use DNSMASQ nft sets Support

- The `pbr` package can be configured to utilize `dnsmasq`'s `nft` `sets` support, which requires the `dnsmasq-full` package with `nft` `sets` support to be installed (see [How to install dnsmasq-full](#Howtoinstalldnsmasq-full)). This significantly improves the start up time because `dnsmasq` resolves the domain names and adds them to the appropriate `nft` `set` in background. `dnsmasq`'s `nft` `set` also automatically adds third-level domains to the `set`: if `domain.com` is added to the policy, this policy will affect all `*.domain.com` subdomains. This also works for top-level domains (TLDs) as well, a policy targeting the `at` TLD for example, will affect all the `*.at` domains.
- Please review the [Footnotes/Known Issues](#FootnotesKnownIssues) section, specifically [<sup>#5</sup>](#footnote5) and [<sup>#7</sup>](#footnote7) and any other information in that section relevant to domain-based routing/DNS.

### 4.9. <a name='Customization'></a>Customization

- Can be fully configured with `uci` commands or by editing `/etc/config/pbr` file.
- Has a companion package (`luci-app-pbr`) so policies can be configured with Web UI.

### 4.10. <a name='OtherFeatures'></a>Other Features

- Doesn't stay in memory, creates the routing tables and `iptables` rules/`sets` entries which are automatically updated when supported/monitored interface changes.
- Proudly made in :maple_leaf: Canada :maple_leaf:, using locally-sourced electrons.

## 5. <a name='Screenshotsluci-app-pbr'></a>Screenshots (luci-app-pbr)

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

## 6. <a name='HowItWorks'></a>How It Works

### 6.1. <a name='HowItWorksnftmode'></a>How It Works (`nft` mode)

On start, this service creates routing tables for each supported interface (WAN/WAN6 and VPN tunnels) which are used to route specially marked packets. Rules for the policies are created in the service-specific chains set up by the `fw4`-specific `nft` scripts installed with the package. Evaluation of packets happens in these `pbr_*` chains after which the packets are sent for marking to the `pbr_mark_*` chains. Whenever possible, the service also creates named sets for `dest_addr` and `src_addr` entries and anonymous sets for `dest_port` and `src_port`. The service then processes the user-created policies.

### 6.2. <a name='HowItWorksiptablesipsetmode'></a>How It Works (`iptables`/`ipset` mode)

On start, this service creates routing tables for each supported interface (WAN/WAN6 and VPN tunnels) which are used to route specially marked packets. For the `mangle` table's `FORWARD`, `INPUT`, `OUTPUT`, `PREROUTING` and `POSTROUTING` chains, the service creates corresponding `PBR_*` chains to which policies are assigned. Evaluation of packets happens in these `PBR_*` chains after which the packets are sent for marking to the `PBR_MARK*` chains. If enabled, the service also creates the ipsets (and the corresponding `iptables` rule for marking packets matching the `ipset`) for `dest_addr` and `src_addr` entries. The service then processes the user-created policies.

### 6.3. <a name='ProcessingPolicies'></a>Processing Policies

Each policy can result in either a new `iptables` or `nft` rule and possibly an `ipset` or a named `nft` `set` to match `dest_addr` and `src_addr`. Anonymous sets may be created within `nft` rules for `dest_port` and `src_port`.

#### 6.3.1. <a name='ProcessingPoliciesnftmode'></a>Processing Policies (`nft` mode)

- Policies with the MAC-addresses, IP addresses, netmasks, local device names or domains will result in a rule targeting named `nft` `sets`.
- Policies with non-empty `dest_port` and `src_port` will be created with anonymous `nft` `sets` within the rule.
- The `dnsmasq` `nftset` entries will be used for domains (if supported and enabled).

#### 6.3.2. <a name='ProcessingPoliciesiptablesipsetmode'></a>Processing Policies (`iptables`/`ipset` mode)

- Policies with the MAC-addresses, IP addresses or local device names in `src_addr` can be created as `iptables` rules or `ipset` entries.
- Policies with non-empty `dest_port` and `src_port` are always created as `iptables` rules.
- Policies with the netmasks in `dest_addr` or `src_addr` can be created as `iptables` rules or `ipset` entries.
- Policies with empty `dest_port` and `src_port` may be created as `iptables` rules or `dnsmasq`'s `ipset` or an `ipset` (if enabled).

### 6.4. <a name='PoliciesPriorities'></a>Policies Priorities

- The policy priority is the same as its order as listed in Web UI and `/etc/config/pbr`. The higher the policy is in the Web UI and configuration file, the higher its priority is.
- If set, the `DSCP` policies is set up first when creating the interface routing.
- If enabled, it is highly recommended that the policies with `IGNORE` target are at the top of the policies list.

### 6.5. <a name='ProcessingCustomUserFiles'></a>Processing Custom User Files

If at least one custom user file is enabled, the service will create the following `nft` `sets` or `ipsets` and set up the proper routing for them.

#### 6.5.1. <a name='ProcessingCustomUserFilesnftmode'></a>Processing Custom User Files (`nft` mode)

For each `interface` the service will create `nft` `sets` and routing:

- `pbr_interface_4_dst_ip_user`: for destination/remote IPv4 addresses and IPv4 CIDR netblocks
- `pbr_interface_6_dst_ip_user`: for destination/remote IPv6 addresses and IPv6 CIDR netblocks
- `pbr_interface_4_src_ip_user`: for source/local IPv4 addresses and IPv4 CIDR netblocks
- `pbr_interface_6_src_ip_user`: for source/local IPv6 addresses and IPv6 CIDR netblocks
- `pbr_interface_4_dst_mac_user`: for source/local MAC addresses
- `pbr_interface_6_dst_mac_user`: for source/local MAC addresses

#### 6.5.2. <a name='ProcessingCustomUserFilesiptablesipsetmode'></a>Processing Custom User Files (`iptables`/`ipset` mode)

For each `interface` the service will create `ipsets` and routing:

- `pbr_interface_4_dst_ip_user`: for destination/remote IPv4 addresses
- `pbr_interface_6_dst_ip_user`: for destination/remote IPv6 addresses
- `pbr_interface_4_dst_net_user`: for destination/remote IPv4 CIDR netblocks
- `pbr_interface_6_dst_net_user`: for destination/remote IPv6 CIDR netblocks
- `pbr_interface_4_src_ip_user`: for source/local IPv4 addresses
- `pbr_interface_6_src_ip_user`: for source/local IPv6 addresses
- `pbr_interface_6_src_net_user`: for source/local IPv6 CIDR netblocks
- `pbr_interface_4_src_net_user`: for source/local IPv4 CIDR netblocks
- `pbr_interface_4_dst_mac_user`: for source/local MAC addresses
- `pbr_interface_6_dst_mac_user`: for source/local MAC addresses

## 7. <a name='HowToInstall'></a>How To Install

### 7.1. <a name='HowToInstall-OpenWrt22.03andnewer'></a>How To Install - OpenWrt 22.03 and newer

Please make sure that the [requirements](#Requirementspbr) are satisfied and install `pbr` and `luci-app-pbr` from Web UI or connect to your router via ssh and run the following commands:

```sh
opkg update
opkg install pbr luci-app-pbr
```

### 7.2. <a name='HowToInstall-OpenWrt21.02andolder'></a>How To Install - OpenWrt 21.02 and older

First, add my repo to your router following instructions at the [How to Use Section of repo documentation](https://docs.openwrt.melmac.net/#how-to-use).

Then, please make sure that the [requirements](#Requirementspbr-iptables) are satisfied and install `pbr-iptables` and `luci-app-pbr` from Web UI or connect to your router via ssh and run the following commands:

```sh
opkg update
opkg install pbr-iptables luci-app-pbr
```

### 7.3. <a name='Requirements'></a>Requirements

Depending on the package flavour, some packages may need to be installed on your router.

#### 7.3.1. <a name='Requirementspbr'></a>Requirements (pbr)

Default builds of OpenWrt 22.03.0 and later are fully compatible with `pbr` and require no additional packages. If you're using a non-standard build, you may have to install the following packages to be installed on your router: `resolveip`, `ip-full` (or a `busybox` built with `ip` support).

To satisfy the requirements, connect to your router via ssh and run the following commands:

```sh
opkg update; opkg install resolveip ip-full
```

#### 7.3.2. <a name='Requirementspbr-iptables'></a>Requirements (pbr-iptables)

This service requires the following packages to be installed on your router: `ipset`, `resolveip`, `ip-full` (or a `busybox` built with `ip` support), `kmod-ipt-ipset` and `iptables`.

To satisfy the requirements, connect to your router via ssh and run the following commands:

```sh
opkg update; opkg install ipset resolveip ip-full kmod-ipt-ipset iptables
```

### 7.4. <a name='Howtoinstalllegacyiptablesipsetpackages'></a>How to install legacy iptables/ipset packages

If you install `pbr` (and not the `pbr-iptables`, which brings all necessary legacy dependencies), but want to use `pbr` in `iptables` mode on OpenWrt 22.03, you will need to connect to your router via ssh and run the following commands:

```sh
opkg update
opkg install ipset libnettle8 libnetfilter-conntrack3 iptables kmod-ipt-ipset iptables-mod-ipopt
```

If you want to use `dnsmasq`'s `ipset` support, you will need to install [`dnsmasq-full` instead of the `dnsmasq`](#Howtoinstalldnsmasq-full).

### 7.5. <a name='Howtoinstalldnsmasq-full'></a>How to install dnsmasq-full

If you want to use `dnsmasq`'s `ipset` or `nft` `sets` support, you will need to install `dnsmasq-full` instead of the `dnsmasq`. To do that, connect to your router via ssh and run the following commands:

```sh
opkg update
opkg install ipset libnettle8 libnetfilter-conntrack3
cd /tmp/ && opkg download dnsmasq-full
opkg remove dnsmasq
opkg install dnsmasq-full --cache /tmp/
rm -f /tmp/dnsmasq-full*.ipk
```

### 7.6. <a name='Unmetdependencies'></a>Unmet dependencies

If you are running a development (trunk/snapshot) build of OpenWrt on your router and your build is outdated (meaning that packages of the same revision/commit hash are no longer available and when you try to satisfy the [requirements](#Requirements) you get errors), please flash either current OpenWrt release image or current development/snapshot image.

## 8. <a name='Howtouse'></a>How to use

### 8.1. <a name='HelpfulInstructionalVideos'></a>Helpful Instructional Videos

If you want to use WebUI to configure `pbr` you may want to review the following YouTube videos:

- [DevOdyssey](https://www.youtube.com/watch?v=FN2qfxNIs2g)

### 8.2. <a name='ServiceConfigurationSettings'></a><a name='service-configuration-settings'></a>Service Configuration Settings

As per screenshots above, in the Web UI the `pbr` configuration is split into `Basic`, `Advanced` and `WebUI` settings. The full list of configuration parameters of `pbr.config` section is:

| Web UI Section      | Parameter                                                       | Type        | Default                                                  | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ------------------- | --------------------------------------------------------------- | ----------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Basic               | <a name="enabled"></a>enabled                                   | boolean     | 0                                                        | Enable/disable the `pbr` service.                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| Basic               | <a name="verbosity"></a>verbosity                               | integer     | 2                                                        | Can be set to 0, 1 or 2 to control the console and system log output verbosity of the `pbr` service.                                                                                                                                                                                                                                                                                                                                                                                  |
| Basic               | <a name="strict_enforcement"></a>strict_enforcement             | boolean     | 1                                                        | Enforce policies when their interface is down. See [Strict enforcement](#StrictEnforcement) for more details.                                                                                                                                                                                                                                                                                                                                                                         |
| Basic               | <a name="resolver_set"></a>resolver_set                         | string      | `pbr`: dnsmasq.nftset<br />`pbr-iptables`: dnsmasq.ipset | Configure the use of the resolver's set support for domains. For most policies targeting domains, this must be enabled to guarantee the reliable operation/routing. Enabling it also speeds up service start-up. Currently supported options are `none`, `adguardhome.ipset`, `dnsmasq.ipset` and `dnsmasq.nftset` (see [Use Resolver's Set Support](#UseResolversSetSupport) and [<sup>#7</sup>](#footnote7) for more details). Make sure the [requirements](#Requirements) are met. |
|                     | <a name="resolver_instance"></a>resolver_instance               | list        | \*                                                       | Configure the use of the resolver's set support for specific instances only by setting a list of instance numbers or names. Currently supported for the following `resolver_set` options: `dnsmasq.ipset` and `dnsmasq.nftset`.                                                                                                                                                                                                                                                       |
| Basic               | <a name="ipv6_enabled"></a>ipv6_enabled                         | boolean     | 0                                                        | Enable/disable IPv6 support.                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| Advanced            | <a name="supported_interface"></a>supported_interface           | list/string |                                                          | Allows to specify the space-separated list of interface names (in lower case) to be explicitly supported by the `pbr` service. Can be useful if your OpenVPN tunnels have dev option other than tun\*.                                                                                                                                                                                                                                                                                |
| Advanced            | <a name="ignored_interface"></a>ignored_interface               | list/string |                                                          | Allows to specify the space-separated list of interface names (in lower case) to be ignored by the `pbr` service. Can be useful if running both VPN server and VPN client on the router.                                                                                                                                                                                                                                                                                              |
| Advanced            | <a name="boot_timeout"></a>boot_timeout                         | number      | 30                                                       | Allows to specify the time (in seconds) for `pbr` service to wait for WAN gateway discovery on boot. Can be useful on devices with ADSL modem built in.                                                                                                                                                                                                                                                                                                                               |
| Advanced            | <a name="rule_create_option"></a>rule_create_option             | add/insert  | add                                                      | Allows to specify the parameter for rules: `add` for `-A` in `iptables` and `add` in `nft` and `insert` for `-I` in `iptables` and `insert` for `nft`. Add is generally speaking more compatible with other packages/firewall rules. Recommended to change to `insert` only to enable compatibility with the `mwan3` package on OpenWrt 21.02 and earlier.                                                                                                                            |
| Advanced            | <a name="icmp_interface"></a>icmp_interface                     | string      |                                                          | Set the default ICMP protocol interface (interface name in lower case). Use with caution.                                                                                                                                                                                                                                                                                                                                                                                             |
| Hidden              | <a name="wan_ip_rules_priority"></a>wan_ip_rules_priority       | integer     | 30000                                                    | Starting (WAN) ip rules priority used by the `pbr` service. High starting priority is used to avoid conflict with other services, this can be changed by user.                                                                                                                                                                                                                                                                                                                        |
| Advanced            | <a name="wan_mark"></a>wan_mark                                 | hexadecimal | 010000                                                   | Starting (WAN) fw mark for marks used by the `pbr` service. High starting mark is used to avoid conflict with SQM/QoS, this can be changed by user. Change with caution together with `fw_mask`.                                                                                                                                                                                                                                                                                      |
| Advanced            | <a name="fw_mask"></a>fw_mask                                   | hexadecimal | ff0000                                                   | FW Mask used by the `pbr` service. High mask is used to avoid conflict with SQM/QoS, this can be changed by user. Change with caution together with `wan_mark`.                                                                                                                                                                                                                                                                                                                       |
| Hidden/Experimental | <a name="secure_reload"></a>secure_reload                       | boolean     | 0                                                        | When enabled, kills router traffic (activates killswitch) during service start/restart/reload operations to prevent traffic leaks on unwanted interface.                                                                                                                                                                                                                                                                                                                              |
| Hidden/Experimental | <a name="nft_file_mode"></a>nft_file_mode                       | boolean     | 1                                                        | When running in `nft` mode and this setting is enabled, generates the `fw4`-compatible atomic `nft` file for all the policies/`nft` commands resulting in much faster service start and consequent service reloads on interface status changes.                                                                                                                                                                                                                                       |
| Web UI              | <a name="webui_show_ignore_target"></a>webui_show_ignore_target | boolean     | 0                                                        | When enabled, show `ignore` in the list of interfaces.                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Web UI              | <a name="webui_supported_protocol"></a>webui_supported_protocol | list        | 0                                                        | List of protocols to display in the `Protocol` column for policies.                                                                                                                                                                                                                                                                                                                                                                                                                   |
|                     | <a name="wan_dscp"></a>wan_dscp                                 | integer     |                                                          | Allows use of [DSCP-tag based policies](#DSCPTag-BasedPolicies) for WAN interface.                                                                                                                                                                                                                                                                                                                                                                                                    |
|                     | <a name="interface_name_dscp"></a>{interface_name}\_dscp        | integer     |                                                          | Allows use of [DSCP-tag based policies](#DSCPTag-BasedPolicies) for a VPN interface.                                                                                                                                                                                                                                                                                                                                                                                                  |
| Hidden              | <a name="procd_boot_delay"></a>procd_boot_delay                 | integer     | 0                                                        | Time (in seconds) to sleep on boot before trying to start `pbr` service.                                                                                                                                                                                                                                                                                                                                                                                                              |
| Hidden              | <a name="procd_reload_delay"></a>procd_reload_delay             | integer     | 0                                                        | Time (in seconds) for PROCD_RELOAD_DELAY parameter.                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Hidden              | <a name="procd_lan_interface"></a>procd_lan_interface           |             |                                                          | Override `lan` interface name with this interface.                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Hidden              | <a name="procd_wan_ignore_status"></a>procd_wan_ignore_status   | boolean     |                                                          | Ignore `wan` interface status (do not wait for the interface to be up) when starting up.                                                                                                                                                                                                                                                                                                                                                                                              |
| Hidden              | <a name="procd_wan_interface"></a>procd_wan_interface           |             |                                                          | Override `wan` interface name with this interface. Newer versions do not attempt to auto-detect WAN interface name and setting this from CLI may be required.                                                                                                                                                                                                                                                                                                                         |
| Hidden              | <a name="procd_wan6_interface"></a>procd_wan6_interface         |             |                                                          | Override `wan6` interface name with this interface.                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Hidden              | <a name="nft_set_auto_merge"></a>nft_set_auto_merge             | boolean     | 1                                                        | Use `auto-merge` option when creating all `nft` sets. See [nftables wiki](https://wiki.nftables.org/wiki-nftables/index.php/Sets) for details.                                                                                                                                                                                                                                                                                                                                        |
| Hidden              | <a name="nft_set_counter"></a>nft_set_counter                   | boolean     | 1                                                        | Use `counter` option when creating all `nft` sets. See [nftables wiki](https://wiki.nftables.org/wiki-nftables/index.php/Sets) for details.                                                                                                                                                                                                                                                                                                                                           |
| Hidden              | <a name="nft_set_flags_interval"></a>nft_set_flags_interval     | boolean     | 1                                                        | Use `flags interval` option when creating all `nft` sets. See [nftables wiki](https://wiki.nftables.org/wiki-nftables/index.php/Sets) for details.                                                                                                                                                                                                                                                                                                                                    |
| Hidden              | <a name="nft_set_flags_timeout"></a>nft_set_flags_timeout       | boolean     | 0                                                        | Use `flags timeout` option when creating all `nft` sets. See [nftables wiki](https://wiki.nftables.org/wiki-nftables/index.php/Sets) for details.                                                                                                                                                                                                                                                                                                                                     |
| Hidden              | <a name="nft_set_gc_interval"></a>nft_set_gc_interval           | string      |                                                          | Use `gc-interval` option when creating all `nft` sets. See [nftables wiki](https://wiki.nftables.org/wiki-nftables/index.php/Sets) for details.                                                                                                                                                                                                                                                                                                                                       |
| Hidden              | <a name="nft_set_policy"></a>nft_set_policy                     | string      | performance                                              | Set `policy` option when creating all `nft` sets. See [nftables wiki](https://wiki.nftables.org/wiki-nftables/index.php/Sets) for details.                                                                                                                                                                                                                                                                                                                                            |
| Hidden              | <a name="nft_set_timeout"></a>nft_set_timeout                   | string      |                                                          | Use `timeout` option when creating all `nft` sets. See [nftables wiki](https://wiki.nftables.org/wiki-nftables/index.php/Sets) for details.                                                                                                                                                                                                                                                                                                                                           |

#### 8.2.1. <a name='DefaultSettings'></a>Default Settings

Default configuration has service disabled (use Web UI to enable/start service or run `uci set pbr.config.enabled=1; uci commit pbr;`).

#### 8.2.2. <a name='PolicyOptions'></a>Policy Options

Each policy may have a combination of the options below, the `name` and `interface` options are required.

The `src_addr`, `src_port`, `dest_addr` and `dest_port` options supports parameter negation, for example if you want to **exclude** remote port 80 from the policy, set `dest_port` to `"!80"` (notice lack of space between `!` and parameter).

| Option        | Default    | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **name**      |            | Policy name, it **must** be set.                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| enabled       | 1          | Enable/disable policy. To display the `Enable` checkbox column for policies in the WebUI, make sure to select `Enabled` for `Show Enable Column` in the `Web UI` tab.                                                                                                                                                                                                                                                                                                         |
| **interface** |            | Policy interface, it **must** be set.                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| src_addr      |            | List of space-separated local/source IP addresses, CIDRs, hostnames or mac addresses (colon-separated). You can also specify a local physical device (like a specially created wlan) prepended by an `@` symbol. Versions 1.1.2 and later allow using URLs to list of addresses. If `curl` is installed you can use the `file://` schema, otherwise you can use `ftp://`, `http://` and `https://` schemas. This is obviously not compatible with the `secure_reload` option. |
| src_port      |            | List of space-separated local/source ports or port-ranges.                                                                                                                                                                                                                                                                                                                                                                                                                    |
| dest_addr     |            | List of space-separated remote/target IP addresses, CIDRs or hostnames/domain names. Versions 1.1.2 and later allow using URLs to list of addresses. If `curl` is installed you can use the `file://` schema, otherwise you can use `ftp://`, `http://` and `https://` schemas. This is obviously not compatible with the `secure_reload` option.                                                                                                                             |
| dest_port     |            | List of space-separated remote/target ports or port-ranges.                                                                                                                                                                                                                                                                                                                                                                                                                   |
| proto         | auto       | Policy protocol, can be any valid protocol from `/etc/protocols` for CLI/uci or can be selected from the values set in `webui_supported_protocol`.                                                                                                                                                                                                                                                                                                                            |
| chain         | prerouting | Policy chain, can be either `forward`, `input`, `prerouting`, `postrouting` or `output`. This setting is case-sensitive.                                                                                                                                                                                                                                                                                                                                                      |

#### 8.2.3. <a name='CustomUserFilesIncludeOptions'></a>Custom User Files Include Options

| Option   | Default | Description                                                                 |
| -------- | ------- | --------------------------------------------------------------------------- |
| **path** |         | Path to a custom user file (in a form of shell script), it **must** be set. |
| enabled  | 1       | Enable/disable setting.                                                     |

### 8.3. <a name='ExamplePolicies'></a>Example Policies

#### 8.3.1. <a name='SingleIPIPRangeLocalMachineLocalMACAddress'></a>Single IP, IP Range, Local Machine, Local MAC Address

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

#### 8.3.2. <a name='LogmeinHamachi'></a>Logmein Hamachi

The following policy routes LogMeIn Hamachi zero-setup VPN traffic via WAN.

```text
config policy
  option name 'LogmeIn Hamachi'
  option interface 'wan'
  option dest_addr '25.0.0.0/8 hamachi.cc hamachi.com logmein.com'
```

#### 8.3.3. <a name='SIPPort'></a>SIP Port

The following policy routes standard SIP port traffic via WAN for both TCP and UDP protocols.

```text
config policy
  option name 'SIP Ports'
  option interface 'wan'
  option dest_port '5060'
  option proto 'tcp udp'
```

#### 8.3.4. <a name='PlexMediaServer'></a>Plex Media Server

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

#### 8.3.5. <a name='EmbyMediaServer'></a>Emby Media Server

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

#### 8.3.6. <a name='IgnoreTarget'></a><a name='ignore-target'></a>Ignore Target

The service allows you to set an interface for a specific policy to `ignore` to skip further processing of matched traffic. This option needs to be explicitly enabled for use in WebUI, check [Service Configuration Settings](#ServiceConfigurationSettings) for details. Some use cases are listed below.

##### Ignore Requests

The following policy allows you to skip processing some requests (like traffic to an OpenVPN or WireGuard server running on the router):

```text
config pbr 'config'
  ...
  option webui_show_ignore_target '1'

config policy
  option name 'Ignore Local Requests by Destination'
  option interface 'ignore'
  option dest_addr '192.168.200.0/24'
```

Please note, you need to enable `Show Ignore Target` option for the WebUI to list`ignore\` in the list of gateways.

It's a good idea to keep the policies targeting `ignore` interface at the top of the config file/list of policies displayed in WebUI to make sure they are processed first.

#### 8.3.7. <a name='BasicOpenVPNClientConfig'></a>Basic OpenVPN Client Config

There are multiple guides online on how to configure the OpenVPN client on OpenWrt "the easy way", and they usually result either in a kill-switch configuration or configuration where the OpenVPN tunnel cannot be properly (and separately from WAN) routed, either way, incompatible with the VPN Policy-Based Routing.

Below is the sample OpenVPN client configuration for OpenWrt which is guaranteed to work. If you have already deviated from the instructions below (ie: made any changes to any of the `wan` or `lan` configurations in either `/etc/config/network` or `/etc/config/firewall`), you will need to start from scratch with a fresh OpenWrt install.

Relevant part of `/etc/config/pbr`:

```text
config pbr 'config'
  list supported_interface 'vpnclient'
  ...
```

The recommended network/firewall settings are below.

Relevant part of `/etc/config/network` (**DO NOT** modify default OpenWrt network settings for either `wan` or `lan`):

```text
config interface 'vpnclient'
  option proto 'none'
  option device 'ovpnc0'
```

Relevant part of `/etc/config/firewall`:

```text
config zone
  option name 'wan'
  list network 'wan'
  list network 'vpnclient'
  ...
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

#### 8.3.8. <a name='MultipleOpenVPNClients'></a>Multiple OpenVPN Clients

If you use multiple OpenVPN clients on your router, the order in which their devices are named (tun0, tun1, etc) is not guaranteed by OpenWrt. The following settings are recommended in this case.

For `/etc/config/network`:

```text
config interface 'vpnclient0'
  option proto 'none'
  option device 'ovpnc0'

config interface 'vpnclient1'
  option proto 'none'
  option device 'ovpnc1'
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

#### 8.3.9. <a name='LocalOpenVPNServerOpenVPNClientScenario1'></a>Local OpenVPN Server + OpenVPN Client (Scenario 1)

If the OpenVPN client on your router is used as default routing (for the whole Internet), make sure your settings are as following (three dots on the line imply other options can be listed in the section as well).

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

Relevant part of `/etc/config/network` (**DO NOT** modify default OpenWrt network settings for either `wan` or `lan`):

```text
config interface 'vpnclient'
  option proto 'none'
  option device 'ovpnc0'

config interface 'vpnserver'
  option proto 'none'
  option device 'ovpns0'
  option auto '1'
```

Relevant part of `/etc/config/firewall`:

```text
config zone
  option name 'lan'
  list network 'lan'
  list network 'vpnserver'
  ...

config zone
  option name 'wan'
  list network 'wan'
  list network 'vpnclient'
  ...

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

#### 8.3.10. <a name='LocalOpenVPNServerOpenVPNClientScenario2'></a>Local OpenVPN Server + OpenVPN Client (Scenario 2)

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

Relevant part of `/etc/config/network` (**DO NOT** modify default OpenWrt network settings for either `wan` or `lan`):

```text
config interface 'vpnclient'
  option proto 'none'
  option device 'ovpnc0'

config interface 'vpnserver'
  option proto 'none'
  option device 'ovpns0'
  option auto '1'
```

Relevant part of `/etc/config/firewall`:

```text
config zone
  option name 'lan'
  list network 'lan'
  list network 'vpnserver'
  ...

config zone
  option name 'wan'
  list network 'wan'
  list network 'vpnclient'
  ...

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

#### 8.3.11. <a name='LocalWireGuardServerWireGuardClientScenario1'></a>Local WireGuard Server + WireGuard Client (Scenario 1)

Yes, I'm aware that technically there are no clients nor servers in WireGuard, only peers in terms of topology. When you configure WireGuard, however, one of the instances will have the `listen_port`, so we'd call that one a server and if it doesn't have the `listen_port`, then it's a client. This terminology is similar to the OpenVPN Server + Client setups.

If the WireGuard tunnel on your router is used as default routing (for the whole Internet), sadly no `pbr` rule will allow it to intercept and properly route the `UDP` traffic of the WireGuard server, please either use the OpenVPN server and configure it to use `TCP` protocol or use Scenario 2 below.

#### 8.3.12. <a name='LocalWireGuardServerWireGuardClientScenario2'></a>Local WireGuard Server + WireGuard Client (Scenario 2)

Yes, I'm aware that technically there are no clients nor servers in WireGuard, only peers in terms of topology, When you configure WireGuard, however, one of the instances will have the `listen_port`, so we'd call that one a server and if it doesn't have the `listen_port`, then it's a client. This terminology is similar to the OpenVPN Server + Client setups.

If the WireGuard client is **not** used as default routing and you create policies to selectively use the WireGuard client, make sure your settings are as following (three dots on the line imply other options can be listed in the section as well). Make sure that the policy mentioned below is at the top of your policies list.

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

Relevant part of `/etc/config/network` (**DO NOT** modify default OpenWrt network settings for either `wan` or `lan`):

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

Relevant part of `/etc/config/firewall`:

```text
config zone
  option name 'lan'
  list network 'lan'
  list network 'wgserver'
  ...

config zone
  option name 'wan'
  list network 'wan'
  list network 'wgclient'
  ...

config rule
  option name 'Allow-WG-Inbound'
  option target 'ACCEPT'
  option src '*'
  option proto 'udp'
  option dest_port '61820'
```

#### 8.3.13. <a name='LocalWireGuardServerAnotherVPNClientScenario1'></a>Local WireGuard Server + Another VPN Client (Scenario 1)

Yes, I'm aware that technically there are no clients nor servers in WireGuard, only peers in terms of topology, When you configure WireGuard, however, one of the instances will have the `listen_port`, so we'd call that one a server and if it doesn't have the `listen_port`, then it's a client. This terminology is similar to the OpenVPN Server + Client setups.

If another VPN client is used as default routing (for the whole Internet), sadly no `pbr` rule will allow it to intercept and properly route the `UDP` traffic of WireGuard server, please either use the OpenVPN server and configure it to use `TCP` protocol or use the Scenario 2 below.

#### 8.3.14. <a name='LocalWireGuardServerAnotherVPNClientScenario2'></a>Local WireGuard Server + Another VPN Client (Scenario 2)

Yes, I'm aware that technically there are no clients nor servers in WireGuard, only peers in terms of topology, When you configure WireGuard, however, one of the instances will have the `listen_port`, so we'd call that one a server and if it doesn't have the `listen_port`, then it's a client. This terminology is similar to the OpenVPN Server + Client setups.

If another VPN client is **not** used as default routing and you create policies to selectively use the VPN client, make sure your settings are as following (three dots on the line imply other options can be listed in the section as well). Make sure that the policy mentioned below is at the top of your policies list.

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

#### 8.3.15. <a name='NetflixDomains'></a>Netflix Domains

The following policy should route US Netflix traffic via WAN. For capturing international Netflix domain names, you can refer to the getdomainnames.sh-specific instructions on [GitHub](https://github.com/Xentrk/netflix-vpn-bypass/blob/master/README.md#ipset_netflix_domainssh)/[jsDelivr](https://cdn.jsdelivr.net/gh/Xentrk/netflix-vpn-bypass/README.md#ipset_netflix_domainssh) and don't forget to adjust them for OpenWrt. This may not work if Netflix changes things. For more reliable US Netflix routing you may want to consider also using [custom user files](#CustomUserFiles).

```text
config policy
  option name 'Netflix Domains'
  option interface 'wan'
  option dest_addr 'amazonaws.com netflix.com nflxext.com nflximg.net nflxso.net nflxvideo.net dvd.netflix.com'
```

#### 8.3.16. <a name='ExampleCustomUserFilesIncludes'></a>Example Custom User Files Includes

```text
config include
  option path '/etc/pbr/pbr.user.aws'
  option enabled '0'

config include
  option path '/etc/pbr/pbr.user.netflix'
  option enabled '0'

config include
  option path '/etc/pbr/pbr.user.wg_server_and_client'
  option enabled '0'
```

## 9. <a name='FootnotesKnownIssues'></a>Footnotes/Known Issues

1.  <a name="footnote1"> </a> See [note about multiple OpenVPN clients](#MultipleOpenVPNClients).

2.  <a name="footnote2"> </a> If your `OpenVPN` interface has the device name different from tun\*, is not up and is not explicitly listed in `supported_interface` option, it may not be available in the policies `Interface` drop-down within WebUI.

3.  <a name="footnote3"> </a> If your default routing is set to the VPN tunnel, then the true WAN interface cannot be discovered using OpenWrt built-in functions, so service will assume your network interface ending with or starting with `wan` is the true WAN interface.

4.  <a name="footnote4"> </a> The service does **NOT** support the "killswitch" router mode (where there is no firewall forwarding from `lan` interface to `wan` interface, so if you stop the VPN tunnel, you have no Internet connection). For proper operation, leave all the default OpenWrt `network` and `firewall` settings for `lan` and `wan` intact.

5.  <a name="footnote5"> </a> When using the `adguardhome.ipset`, `dnsmasq.ipset` or `dnsmasq.nftset` option, please make sure to flush the DNS cache of the local devices, otherwise domain policies may not work until you do. If you're not sure how to flush the DNS cache (or if the device/OS doesn't offer an option to flush its DNS cache), reboot your local devices when starting to use the service and/or when connecting data-capable device to your WiFi.

6.  <a name="footnote6"> </a> When using the policies targeting physical devices, you may need to make sure you have the following packages installed: `kmod-br-netfilter`, `kmod-ipt-physdev` and `iptables-mod-physdev`. Also, if your physical device is a part of the bridge, you may have to set `net.bridge.bridge-nf-call-iptables` to `1` in your `/etc/sysctl.conf`.

7.  <a name="footnote7"> </a> If you're using domain names in the `dest_addr` option of the policy, it is recommended to use `adguardhome.ipset`, `dnsmasq.ipset` or `dnsmasq.nftset` options for `resolver_set`. Otherwise, the domain name will be resolved when the service starts up and the resolved IP address(es) will be added to an apropriate set or an `iptables` or `nft` rule. Resolving a number of domains on start is a time consuming operation, while using the `adguardhome.ipset`, `dnsmasq.ipset` or `dnsmasq.nftset` options allows transparent and fast addition of the correct domain IP addresses to the apropriate set on DNS request or when resolver is idle.

8.  <a name="footnote8"> </a> When service is started, it subscribes to the supported interfaces updates thru the PROCD. While I was never able to reproduce the issue, some customers report that this method doesn't always work in which case you may want to [set up iface hotplug script](#AWordAboutInterfaceHotplugScript) to reload service when the relevant interface(s) are updated.

## 10. <a name='FAQ'></a>FAQ

You may find some useful information below.

### 10.1. <a name='AWordAboutDefaultRouting'></a><a name='a-word-about-default-routing'></a>A Word About Default Routing

Service does not alter the default routing. Depending on your VPN tunnel settings (and settings of the VPN server you are connecting to), the default routing might be set to go via WAN or via VPN tunnel. This service affects only routing of the traffic matching the policies. If you want to override default routing, follow the instructions below.

#### 10.1.1. <a name='OpenVPNtunnelconfiguredviaucietcconfigopenvpn'></a>OpenVPN tunnel configured via uci (/etc/config/openvpn)

To unset an OpenVPN tunnel as default route, set the following to the appropriate section of your `/etc/config/openvpn`:

- For OpenVPN 2.4 and newer client config:

  ```text
  list pull_filter 'ignore "redirect-gateway"'
  ```

- For OpenVPN 2.3 and older client config:

  ```text
  option route_nopull '1'
  ```

- For your WireGuard (client) config:

  ```text
  option route_allowed_ips '0'
  ```

#### 10.1.2. <a name='OpenVPNtunnelconfiguredwith.ovpnfile'></a>OpenVPN tunnel configured with .ovpn file

To unset an OpenVPN tunnel as default route, set the following to the appropriate section of your `.ovpn` file:

- For OpenVPN 2.4 and newer client `.ovpn` file:

  ```text
  pull-filter ignore "redirect-gateway"
  ```

- For OpenVPN 2.3 and older client `.ovpn` file:

  ```text
  route-nopull
  ```

#### 10.1.3. <a name='WireGuardtunnel'></a>WireGuard tunnel

To unset a WireGuard tunnel as default route, set the following to the appropriate section of your `/etc/config/network`:

- For your WireGuard (client) config:

  ```text
  option route_allowed_ips '0'
  ```

- Routing WireGuard traffic may require setting `net.ipv4.conf.wg0.rp_filter = 2` in `/etc/sysctl.conf`. Please refer to [issue #41](https://github.com/stangri/source.openwrt.melmac.net/issues/41) for more details.

### 10.2. <a name='AWordAboutCloudflares1.1.1.1App'></a>A Word About Cloudflare's 1.1.1.1 App

Cloudflare has released an app for [iOS](https://itunes.apple.com/us/app/1-1-1-1-faster-internet/id1423538627) and [Android](https://play.google.com/store/apps/details?id=com.cloudflare.onedotonedotonedotone), which can also be configured to route traffic thru their own VPN tunnel (WARP+).

If you use Cloudflare's VPN tunnel (WARP+), none of the policies you set up with the VPN Policy Routing will take effect on your mobile device. Disable WARP+ for your home WiFi to keep VPN Policy Routing affecting your mobile device.

If you just use the private DNS queries (WARP), [A Word About DNS-over-HTTPS](#AWordAboutDNS-over-HTTPS) applies. You can also disable WARP for your home WiFi to keep VPN Policy Routing affecting your mobile device.

### 10.3. <a name='AWordAboutDNS-over-HTTPS'></a>A Word About DNS-over-HTTPS

Some browsers, like [Mozilla Firefox](https://support.mozilla.org/en-US/kb/firefox-dns-over-https#w_about-dns-over-https) or [Google Chrome/Chromium](https://blog.chromium.org/2019/09/experimenting-with-same-provider-dns.html) have [DNS-over-HTTPS proxy](https://en.wikipedia.org/wiki/DNS_over_HTTPS) built-in. Their requests to web-sites listed in policies cannot be properly routed if the `resolver_set` is set to either `dnsmasq.ipset` or `dnsmasq.nftset`. To fix this, you can try either of the following:

1.  Disable the DNS-over-HTTPS support in your browser and use the OpenWrt's `net/https-dns-proxy` (README on [GitHub](https://docs.openwrt.melmac.net/https-dns-proxy)/[jsDelivr](https://cdn.jsdelivr.net/gh/stangri/docs.openwrt.melmac.net/https-dns-proxy/)) package with optional `https-dns-proxy` WebUI/luci app. You can then continue to use either `dnsmasq.ipset` or `dnsmasq.nftset` setting for the `resolver_set` in Policy-Based Routing.

2.  Continue using DNS-over-HTTPS in your browser (which, by the way, also limits your options for router-level AdBlocking as described in `net/simple-adblock` README on [GitHub](https://docs.openwrt.melmac.net/simple-adblock/#dns-resolution-option)/[jsDelivr](https://cdn.jsdelivr.net/gh/stangri/docs.openwrt.melmac.net/simple-adblock/README.md#dns-resolution-option)), you than would either have to switch the `resolver_set` to `none`. Please note, you will lose all the benefits of [`dnsmasq.ipset`](#UseDNSMASQipsetSupport) option.

### 10.4. <a name='AWordAboutHTTP3QUIC'></a>A Word About HTTP/3 (QUIC)

If you want to target traffic using HTTP/3 protocol, you can use the `AUTO` as the protocol (the policy will be either protocol-agnostic or `TCP/UDP`) or explicitly use `UDP` as a protocol.

### 10.5. <a name='AWordAboutIPv6Routing'></a>A Word About IPv6 Routing

Due to the nature of IPv6, it's not supposed to be routed as IPv4 is with this package. A fellow user has graciously contributed a [gist detailing their experience to get IPv6 routing working](https://gist.github.com/aliicex/3bd8413029b1f728c1f00bc1ac0e98b4).

### 10.6. <a name='AWordAboutRoutingNetflixAmazonPrimeHuluTraffic'></a>A Word About Routing Netflix/Amazon Prime/Hulu Traffic

There are two following scenarios with VPN connections and Netflix/Amazon Prime/Hulu traffic.

#### 10.6.1. <a name='RoutingNetflixAmazonPrimeHuluTrafficviaVPNTunnel'></a>Routing Netflix/Amazon Prime/Hulu Traffic via VPN Tunnel

If you live in a country where Netflix/Amazon Prime/Hulu are not available and want to circumvent geo-fencing, this package can't help you. The Netflix/Amazon Prime/Hulu do a great job detecting VPN usage when accessing their services and circumventing geographical restrictions is not only dubiously legal, it's also technically very challenging.

#### 10.6.2. <a name='RoutingNetflixAmazonPrimeHuluTrafficviaWAN'></a>Routing Netflix/Amazon Prime/Hulu Traffic via WAN

If you live in a country where Netflix/Amazon Prime/Hulu are available, you obviously do NOT want to use VPN tunnel for their traffic.

If the VPN tunnel is not used as a default gateway on your router, you should not have a problem accessing Netflix/Amazon Prime/Hulu (just make sure that your DNS requests are not routed via VPN tunnel either).

If the VPN tunnel is used as a default gateway, either:

- send ALL traffic from your multimedia devices (by using their IP addresses or device names in the `src_addr` option in config file or Local addresses /devices field in WebUI) accessing Netflix/Amazon Prime/Hulu to WAN; this is the more reliable and recommended method.
- use the [Netflix/AWS custom user files](https://docs.openwrt.melmac.net/pbr/#custom-user-files) in combination with the [Netflix](https://docs.openwrt.melmac.net/pbr/#NetflixDomains)/Amazon Prime/Hulu domains and `dnsmasq.ipset`/`dnsmasq.nftset` option to route traffic to Netflix/Amazon via WAN; this is definitely less reliable method and may not work in all regions.

Either way make sure that your DNS requests are not routed via VPN Tunnel!

### 10.7. <a name='AWordAboutInterfaceHotplugScript'></a>A Word About Interface Hotplug Script

Sometimes[<sup>#8</sup>](#footnote8) the service doesn't get reloaded when supported interfaces go up or down. This can be an annoying experience since the service may start before all supported VPN connections are up and then not get updated when the VPN connections get established. In that case, run the following command from CLI to create the interface hotplug script to cause the service to be reloaded in interface updates:

```sh
mkdir -p /etc/hotplug.d/iface/
cat << 'EOF' > /etc/hotplug.d/iface/70-pbr
#!/bin/sh
logger -t pbr "Reloading $INTERFACE due to $ACTION of $INTERFACE ($DEVICE)"
/etc/init.d/pbr reload_interface "$INTERFACE"
EOF
```

### 10.8. <a name='AWordAboutDifferencesfromvpn-policy-routing'></a>A Word About Differences from `vpn-policy-routing`

Unlike the `vpn-policy-routing`, the `pbr` package:

- creates set and resolver's set (like `dnsmasq.ipset` or `dnsmasq.nftset`)-based policies respecting policy's priority.
- can only make minimal changes on a single interface reload (it does not reload the whole service).
- can only reload policies on service reload (it does not reload the network-related parts).
- implements the new `secure_reload` option to kill all traffic during the service start/restart/reload.
- only supports OpenWrt 21.02 and newer
- supports fw4 and nft/nft sets (and dnsmasq nft set support) on OpenWrt 22.03
- chain names in config file should be in lower case, not upper case

### 10.9. <a name='AWordAboutMigratingfromvpn-policy-routing'></a>A Word About Migrating from `vpn-policy-routing`

The new `pbr` package is largely config-compatible with the `vpn-policy-routing` package, if you install `pbr` version 1.0.0 and newer on a system where `vpn-policy-routing` was previously installed and `pbr` was not, the `vpn-policy-routing` config will be migrated to `pbr` config (and the `vpn-policy-routing` disabled) automatically upon install.

In the rare case that you need to manually migrate it can be accomplished by running:

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
  sed -i "s/option fw_mask '0x(._)'/option fw_mask '\\1'/g" /etc/config/pbr
  sed -i "s/option wan_mark '0x(._)'/option wan_mark '\\1'/g" /etc/config/pbr
  uci set vpn-policy-routing.config.enabled=0; uci commit;
fi
```

Please note that the ipset/nft set names which service creates for use in the custom user files have changed. Refer to the new custom user files supplied with the package for reference.

## 11. <a name='GettingHelp'></a>Getting Help

If things are not working as intended, please first set verbosity to 2 by running these commands

```sh
uci set pbr.config.verbosity='2'; uci commit pbr;
```

and then run the following commands and include their output (you can and should mask sensitive information) in your post:

```sh
ubus call system board
uci export dhcp
uci export firewall
uci export network
uci export pbr
/etc/init.d/pbr status
/etc/init.d/pbr reload
/etc/init.d/pbr status
```

If you want to encrypt the files before sharing them, you can do so with the commands below (you'll need to have OpenSSL installed on your router):

```ssh
openssl rand 214 > /tmp/keyfile.key
wget -O /tmp/dev.melmac.net.pem https://dev.melmac.net/repo/dev.melmac.net.pem
openssl pkeyutl -encrypt -inkey /tmp/dev.melmac.net.pem -pubin -in /tmp/keyfile.key -out /tmp/keyfile.enc
openssl enc -in /etc/config/dhcp -out /tmp/dhcp.enc -e -aes256 -pbkdf2 -kfile /tmp/keyfile.key
openssl enc -in /etc/config/firewall -out /tmp/firewall.enc -e -aes256 -pbkdf2 -kfile /tmp/keyfile.key
openssl enc -in /etc/config/network -out /tmp/network.enc -e -aes256 -pbkdf2 -kfile /tmp/keyfile.key
openssl enc -in /etc/config/pbr -out /tmp/pbr.enc -e -aes256 -pbkdf2 -kfile /tmp/keyfile.key
```

Then share the following files: `/tmp/keyfile.enc`, `/tmp/dhcp.enc`, `/tmp/firewall.enc`, `/tmp/network.enc`, `/tmp/pbr.enc`.

### 11.1. <a name='FirstTroubleshootingStep'></a>First Troubleshooting Step

If your router is set to use [default routing via VPN tunnel](#AWordAboutDefaultRouting) and the WAN-targeting policies do not work, you need to stop your VPN tunnel first and ensure that you still have an Internet connection. If your router is set up to use default routing via a VPN tunnel and when you stop the VPN tunnel you have no Internet connection, this package can't help you. You first need to make sure that you do have an Internet connection when the VPN tunnel is stopped.

## 12. <a name='Thanks'></a>Thanks

I'd like to thank everyone who helped create, test and troubleshoot this service. Without contributions from [@hnyman](https://github.com/hnyman), [@dibdot](https://github.com/dibdot), [@danrl](https://github.com/danrl), [@tohojo](https://github.com/tohojo), [@cybrnook](https://github.com/cybrnook), [@nidstigator](https://github.com/nidstigator), [@AndreBL](https://github.com/AndreBL), [@dz0ny](https://github.com/dz0ny), [@tew42](https://github.com/tew42), [bogorad](https://forum.openwrt.org/u/bogorad), rigorous testing/bugreporting by [@dziny](https://github.com/dziny), [@bluenote73](https://github.com/bluenote73), [@buckaroo](https://github.com/pgera), [@Alexander-r](https://github.com/Alexander-r), [@n8v8R](https://github.com/n8v8R), [psherman](https://forum.openwrt.org/u/psherman), [@Vale-max](https://github.com/Vale-max), [@aliicex](https://github.com/aliicex), [dscpl](https://forum.openwrt.org/u/dscpl), [@egc112](https://github.com/egc112) and multiple contributions from [@dl12345](https://github.com/dl12345) and [trendy](https://forum.openwrt.org/u/trendy) and feedback from other OpenWrt users it wouldn't have been possible. WireGuard/IPv6 support is courtesy of [IVPN](https://www.ivpn.net/).

<!-- markdownlint-disable MD033 -->

<script defer src='https://static.cloudflareinsights.com/beacon.min.js' data-cf-beacon='{"token": "911798f2c34b45338f8f8182830a3eb6"}'></script>
