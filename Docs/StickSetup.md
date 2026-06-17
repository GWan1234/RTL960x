# RTL960x SFP xPON ONU Configuration Guide

## Overview

This guide describes how to configure a Realtek RTL960x-based SFP xPON ONU to emulate an existing ONU and obtain successful authentication and service provisioning from the OLT.

Most GPON and EPON operators provision services based on a combination of authentication credentials and ONU identity information. Simply reaching the O5 operational state does not guarantee that the OLT will deliver a usable OMCI configuration. Many operators restrict provisioning to approved ONU models, vendor profiles, or device identities.

The goal of this guide is to replicate the identity of a provisioned ONU so that the SFP xPON ONU receives the same service profile as the original device.

Huawei HG8240H device information is used throughout this guide as an example.

---

# Default Credentials

| Model       | Username | Password    |
| ----------- | -------- | ----------- |
| DFP-34X-2C2 | `admin`  | `admin`     |
| TWCGPON657  | `admin`  | `system`    |
| UF-Instant  | `ubnt`   | `ubnt`      |
| V2801F      | `admin`  | `stdONU101` |
| V2802RH     | `admin`  | `stdONUi0i` |
| LXT-010S-H  | `leox`   | `leolabs_7` |

## TWCGPON657

If you are using a TWCGPON657 module, Telnet access must be enabled before configuration.

| State   | URL                                      |
| ------- | ---------------------------------------- |
| Enable  | `http://192.168.1.1/bd/telnet_open.asp`  |
| Disable | `http://192.168.1.1/bd/telnet_close.asp` |

---

# Understanding OLT Authentication and O5

The OLT determines which OMCI profile and provisioning parameters should be delivered based on the ONU identity reported during registration.

An ONU may successfully reach the O5 operational state while still receiving incomplete or incompatible OMCI provisioning. This typically occurs when the reported ONU identity does not match a profile recognized by the OLT.

Many OLT platforms support vendor-specific ONU profiles and may reject, limit, or modify provisioning for unsupported devices.

The following parameters are commonly evaluated during ONU authentication and service provisioning.

## Authentication Parameters

Most important:

* PLOAM Password
* LOID Credentials
* GPON Serial Number
* MAC Address (operator dependent)

## ONU Identity Parameters

Used for OMCI provisioning:

* Vendor ID
* Device Model
* Hardware Version
* Software Version

## Additional Parameters

Vendor-specific requirements:

* OUI
* Hardware Serial Number
* CWMP Information

It is recommended to clone the original ONU configuration whenever possible.

---

# Collect ONU Information

Before configuring the SFP xPON ONU, collect information from the original ONU.

Most ONUs provide this information through the web interface, Telnet, SSH, or exported configuration files.

| Information            | Flash Variable                                                               | Example              |
| ---------------------- | ---------------------------------------------------------------------------- | -------------------- |
| MAC Address            | `ELAN_MAC_ADDR`                                                              | `781735000000`       |
| Hardware Version       | `HW_HWVER`                                                                   | `BF9.A`              |
| Software Version       | `OMCI_SW_VER1`, `OMCI_SW_VER2`, `CUSTOM_OMCI_SW_VER1`, `CUSTOM_OMCI_SW_VER2` | `V3R017C10S100`      |
| GPON Serial Number     | `GPON_SN`                                                                    | `HWTC35000000`       |
| Vendor ID              | `PON_VENDOR_ID`                                                              | `HWTC`               |
| Device Model           | `GPON_ONU_MODEL`                                                             | `HG8240H`            |
| OUI                    | `OUI`                                                                        | `875773`             |
| Hardware Serial Number | `HW_SERIAL_NO`                                                               | `UONHUWH12341234123` |

---

# GPON and EPON Authentication

Authentication methods vary between operators.

## GPON

Commonly uses:

* GPON Serial Number
* PLOAM Password

## EPON

Commonly uses:

* LOID
* LOID Password
* MAC Address
* Vendor-specific credentials

Always verify the authentication method used by your ISP before changing parameters.

---

# Backup Original Configuration

Before making any changes, record the original values.

> Warning
>
> Incorrect configuration values may prevent ONU registration or service provisioning.
>
> Always keep a backup of the original settings.

```shell
flash get PON_VENDOR_ID
flash get HW_CWMP_MANUFACTURER
flash get HW_CWMP_PRODUCTCLASS
flash get HW_HWVER
flash get OMCI_VEIP_SLOT_ID
flash get OMCI_OLT_MODE
flash get OMCI_FAKE_OK
flash get OMCI_SW_VER1
flash get OMCI_SW_VER2
flash get GPON_ONU_MODEL
flash get GPON_SN
flash get GPON_PLOAM_PASSWD
flash get ELAN_MAC_ADDR
flash get VS_AUTH_KEY
flash get MAC_KEY
flash get OUI
flash get HW_SERIAL_NO
```

---

# Apply Configuration

Most `flash set` operations require a reboot before changes become active.

```shell
reboot
```

Apply all required parameters before rebooting the device.

---

# Authentication Configuration

## PLOAM Password

Used primarily on GPON networks.

### DFP-34X-2C2 (HEX Format)

```shell
flash set GPON_PLOAM_FORMAT 0
flash set GPON_PLOAM_PASSWD 44454641554C54303132
```

> Firmware version `220304` and newer accepts only hexadecimal PLOAM values through Telnet. Use the WebUI if ASCII input is required.

### ASCII Format

Supported by:

* V2801F
* TWCGPON657
* UF-Instant

```shell
flash set GPON_PLOAM_PASSWD DEFAULT012
```

## LOID Authentication

Commonly used on EPON networks.

```shell
flash set LOID 0123456789
flash set LOID_PASSWD 0123456789
```

## GPON Serial Number

```shell
flash set GPON_SN HWTC12345678
```

The Vendor ID portion of the serial number must match `PON_VENDOR_ID`.

## MAC Address

```shell
flash set ELAN_MAC_ADDR 000000111111
```

### Notes

1. GPON transports traffic through GEM frames and does not use the ONU MAC address for normal upstream transport.
2. If the ISP binds service to a MAC address, the router WAN MAC may need to be modified.
3. EPON deployments may require the ONU MAC address to match the original device.

---

# OMCI Device Information

OMCI (ONT Management and Control Interface) is used by the OLT to provision and manage ONU services.

Many OLT platforms maintain vendor-specific ONU profiles. If the reported ONU identity does not match a supported profile, the ONU may authenticate successfully but fail to receive complete service provisioning.

The following parameters define the ONU identity presented to the OLT.

## Hardware Version

```shell
flash set HW_HWVER 168D.A
```

## Vendor ID

The Vendor ID identifies the ONU manufacturer.

Known examples:

| ID     | Vendor                 |
| ------ | ---------------------- |
| `HWTC` | Huawei                 |
| `ZTEG` | ZTE                    |
| `ALCL` | Nokia / Alcatel-Lucent |
| `UBNT` | Ubiquiti               |
| `FHTT` | FiberHome              |
| `RTKG` | Realtek                |
| `TPLG` | TP-Link                |
| `SCOM` | Sercomm                |
| `LEOX` | LeoLabs                |

```shell
flash set PON_VENDOR_ID HWTC
```

> The Vendor ID must match the GPON Serial Number prefix.

## Device Model

```shell
flash set GPON_ONU_MODEL HG8240H5
```

## Software Version

```shell
flash set OMCI_SW_VER1 V5R019C00S125
flash set OMCI_SW_VER2 V5R019C00S125
```

If the firmware prevents modification of these values, use:

```shell
flash set OMCI_OLT_MODE 3
```

## OUI

```shell
flash set OUI
```

The Organizationally Unique Identifier (OUI) identifies the manufacturer assigned to a MAC address block.

## Hardware Serial Number

```shell
flash set HW_SERIAL_NO
```

This is separate from `GPON_SN`.

In Universal ONU deployments, the Hardware Serial Number may be used by the OLT to identify the original hardware platform and determine compatibility behaviour.

---

# OMCI Additional Settings

## Fake OK

Some OLT platforms send vendor-specific OMCI messages that are not fully supported by RTL960x firmware.

Enabling Fake OK causes the ONU to acknowledge unsupported OMCI messages instead of reporting an error.

```shell
flash set OMCI_FAKE_OK 1
```

This setting is commonly used when emulating vendor-specific ONUs.

## OMCI OLT Mode

OMCI OLT Mode controls how the RTL960x presents itself to the OLT.

This feature allows the ONU to emulate behaviour expected by specific vendors.

| Mode    | Value | Description                                                            |
| ------- | ----- | ---------------------------------------------------------------------- |
| Default | `0`   | Default Realtek profile                                                |
| Huawei  | `1`   | Huawei compatibility                                                   |
| ZTE     | `2`   | ZTE compatibility                                                      |
| Custom  | `3`   | User-defined profile                                                   |
| Custom  | `21`  | Force custom profile using segmentation fault behaviour on DFP-34X-2C2 |

```shell
flash set OMCI_OLT_MODE 0
```

When using a custom profile, configure:

1. `PON_VENDOR_ID`
2. `GPON_ONU_MODEL`
3. `HW_HWVER`
4. `OMCI_SW_VER1`
5. `OMCI_SW_VER2`

---

# CWMP (TR-069)

Most deployments do not require CWMP configuration.

However, some operators validate CWMP information during provisioning.

## Manufacturer

```shell
flash set HW_CWMP_MANUFACTURER 'Huawei Technologies Co., Ltd'
```

## Product Class

```shell
flash set HW_CWMP_PRODUCTCLASS HGU
```

| Code | Description             |
| ---- | ----------------------- |
| IGD  | Internet Gateway Device |
| HGU  | Home Gateway Unit       |
| SFU  | Single Family Unit      |

---

# Example Configuration

The following example emulates a Nokia G-240G-E ONU.

```shell
flash set GPON_PLOAM_PASSWD 0123456789
flash set GPON_SN ALCL00000000

flash set GPON_ONU_MODEL G-240G-E
flash set PON_VENDOR_ID ALCL

flash set OMCI_SW_VER1 3FE46606BGCB45
flash set OMCI_SW_VER2 3FE46606BGCB45
flash set CUSTOM_OMCI_SW_VER1 3FE46606BGCB45
flash set CUSTOM_OMCI_SW_VER2 3FE46606BGCB45

flash set HW_HWVER 3FE48153CBAA
flash set ELAN_MAC_ADDR 781735000000
flash set VS_AUTH_KEY EF0FC1A970A27D52DF340835F0561507

flash set OMCI_FAKE_OK 1
flash set OMCI_OLT_MODE 1
```

Reboot the ONU after applying all configuration changes.

---

# MAC Address and Hardware Version Locking

Certain vendors implement licensing mechanisms to prevent modification of ONU identity parameters.

These mechanisms are commonly found in carrier-grade deployments.

## DFP-34X-2C2

Firmware `V1.0-220304` and newer requires a matching `MAC_KEY` whenever `ELAN_MAC_ADDR` is modified.

### Generate MAC_KEY

```shell
echo -n "hsgq1.9aMAC_ADDR_UPPERCASE" | md5sum
```

Example:

```shell
echo -n "hsgq1.9aFFFFFF000000" | md5sum
```

Output:

```text
46f4ea2e3f18ba3bc1f2671b5f7e1f62
```

Apply the generated value:

```shell
flash set MAC_KEY 46f4ea2e3f18ba3bc1f2671b5f7e1f62
```

## V2801F

Changing `ELAN_MAC_ADDR` or `HW_HWVER` requires a valid `VS_AUTH_KEY`.

### Generate VS_AUTH_KEY

```shell
VsAuthKeyGen.exe <mac_address> [HW_HWVER]
```

Example:

```shell
VsAuthKeyGen.exe 000000111111 168D.A
```

Output:

```text
9E7E54597511D721D3A2932B048C0494
```

Apply the generated value:

```shell
flash set VS_AUTH_KEY 9E7E54597511D721D3A2932B048C0494
```
