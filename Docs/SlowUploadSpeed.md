# Slow Upload Speed

Some users experience significantly reduced upload performance when using an RTL9601x operating in 2.5GbE mode.

This issue has been observed on certain GPON networks where the ISP-provided ONU is based on a Realtek platform and the OLT provisions vendor-specific traffic management parameters.

Affected users may observe:

* Upload speeds significantly below the subscribed rate
* Upload performance lower than the ISP-provided ONU
* Increased latency during heavy uploads
* Packet loss or packet drops under sustained upload traffic

---

# Affected Hardware

The issue has been observed on networks where the original ONU uses a Realtek-based platform.

Known examples:
1. RTL9601CI
2. RTL9601D
3. RTL9602
4. RTL9607DQ (prior V1.1.1)

The issue is not limited to these devices and may affect other Realtek-based ONUs depending on OLT provisioning behaviour.

---

# Root Cause

Some OLT deployments provision ONU traffic management settings using OMCI.

Although the RTL9601x successfully authenticates and reaches the O5 operational state, the traffic management configuration delivered by the OLT may not be interpreted identically to the ISP-provided ONU.

As a result, ingress and egress shaping parameters may be applied differently, leading to reduced upload performance.

This behaviour has primarily been observed when the ISP-provided ONU is also based on a Realtek platform, such as the RTL9607x.

---

# Diagnostics

Compare the bandwidth configuration of the ISP-provided ONU and the RTL9601x.

Differences in ingress or egress shaping may indicate that the OLT delivered vendor-specific traffic management settings.

## RTL9607x

```shell
bandwidth get egress port all
port: 0  rate:1024000

bandwidth get ingress port all
port: 0 rate:32767999
port: 1 rate:32767999
port: 2 rate:32767999
port: 3 rate:32767999
port: 4 rate:32767999
port: 5 rate:32767999
port: 6 rate:32767999
port: 7 rate:32767999
```

## RTL9601x

```shell
bandwidth get egress port all
port: 0  rate:1048568
         queue: 0  apr-index: 0
         queue: 1  apr-index: 0
         queue: 2  apr-index: 0
         queue: 3  apr-index: 0
         queue: 4  apr-index: 0
         queue: 5  apr-index: 0
         queue: 6  apr-index: 0
         queue: 7  apr-index: 0

port: 2  rate:4194296
         queue: 0  apr-index: 0
         queue: 1  apr-index: 0
         queue: 2  apr-index: 0
         queue: 3  apr-index: 0
         queue: 4  apr-index: 0
         queue: 5  apr-index: 0
         queue: 6  apr-index: 0
         queue: 7  apr-index: 0

bandwidth get ingress port all
port: 0 rate:1048568
port: 2 rate:4194296
```

---

# Workarounds

The following methods may reduce or eliminate upload performance issues depending on the network configuration.

---

## Adjust `OMCI_TM_OPT`

Some OLT deployments expect a different traffic management mode.

Try changing `OMCI_TM_OPT` and test upload performance after each change.

### Configuration

```shell
flash set OMCI_TM_OPT 0
reboot
```

### Available Values

| Value | Mode                         |
| ----- | ---------------------------- |
| `0`   | Priority controlled          |
| `1`   | Rate controlled              |
| `2`   | Priority and rate controlled |

---

## Manual Bandwidth Configuration

Traffic management values can be manually overridden using `diag`.

### Configuration

```shell
/bin/diag port set auto-nego port all ability asy-flow-control
/bin/diag bandwidth set egress port all rate 4194296
/bin/diag bandwidth set ingress port all rate 4194296
```

> Note
>
> Changes made through `diag` are runtime only and will not persist after reboot.

---

# MikroTik Workarounds

GPON provides approximately 1.25Gbps upstream capacity.

When connected through a 2.5GbE interface, some MikroTik devices can transmit traffic faster than the available GPON upstream bandwidth, resulting in excessive buffering, packet drops, and increased latency.

The following workarounds help align Ethernet transmission rates with GPON upstream capacity.

---

## MikroTik GPON Flooding Mitigation

Devices such as the RB5009 can benefit from limiting switch egress bandwidth.

### Configuration

```rsc
/interface/ethernet/switch/port/set sfp-sfpplus1 egress-rate=1200M
```

```rsc
/interface/ethernet/set sfp-sfpplus1 auto-negotiation=no speed=2.5G-baseX rx-flow-control=on tx-flow-control=on
```

```rsc
/queue interface set ether1 queue=multi-queue-ethernet-default
/queue interface set ether2 queue=multi-queue-ethernet-default
/queue interface set ether3 queue=multi-queue-ethernet-default
/queue interface set ether4 queue=multi-queue-ethernet-default
/queue interface set ether5 queue=multi-queue-ethernet-default
/queue interface set ether6 queue=multi-queue-ethernet-default
/queue interface set ether7 queue=multi-queue-ethernet-default
/queue interface set ether8 queue=multi-queue-ethernet-default
/queue interface set sfp-sfpplus1 queue=multi-queue-ethernet-default
```

Credit: [Luckygecko1 @ Reddit](https://www.reddit.com/r/mikrotik/comments/14ky6s1/rb5009_poor_25g_ethernet_performance/jq0gjer/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)

---

## MikroTik Without Switch Port Egress Support

Some routers, such as the CCR2004-1G-12S+2XS, do not include a switch chip or hardware egress rate limiting.

In this case, CAKE can be used to shape traffic before it reaches the ONU.

### Configuration

```rsc
/queue type add name=cake-egress-gpon kind=cake cake-bandwidth=1200M
/queue interface set sfp-sfpplus1 queue=cake-egress-gpon
```

---

# Related Hardware

Example ISP-provided ONU based on a Realtek platform.

## D-Link DPN-FX3060V

![D-Link DPN-FX3060V Front](https://pictr.com/images/2023/06/24/EuA0cO.md.jpg)

![D-Link DPN-FX3060V Rear](https://pictr.com/images/2023/06/24/EuAPnr.md.jpg)

---

# Conclusion

If the above methods do not improve upload performance, the issue is likely caused by firmware-level traffic management behaviour that cannot be corrected through user configuration.

In such cases, using the ISP-provided ONU or waiting for a firmware update from the ONU vendor may be the only available solutions.

Additional investigation may be possible by comparing OMCI provisioning, traffic management parameters, and bandwidth profiles between the ISP-provided ONU and the ODI DFP-34X-2C2.
