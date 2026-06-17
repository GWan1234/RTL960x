# Configuration Table

The following tables contain known default values used by various ONUs. These values may be useful when cloning ONU identities, emulating vendor-specific devices, or troubleshooting OMCI provisioning issues.

> [!NOTE]
>
> Values may differ between firmware versions and hardware revisions. Always verify the original ONU configuration before applying changes.
>
> [Read More on english odi configuration guide by rajkosto](https://gist.github.com/rajkosto/b684b7bb2697baa342cd2601ed5717d2)

---

## Authentication

| Model       | `GPON_SN`    | `PON_VENDOR_ID` | `OMCC_VER` |
| ----------- | ------------ | --------------- | ---------- |
| DFP-34X-2C2 | XPON1234ABCD | HSGQ            | 128        |
| DPN-FX3060V | DLKI1234ABCD | DLNK            | 128        |
| HG8245W5    | HWTC1234ABCD | HWTC            | 134        |

Authentication-related parameters used during ONU registration and capability negotiation.

---

## ONU Identity

| Model       | `GPON_ONU_MODEL` | `HW_HWVER` | `OMCI_SW_VER1` | `OMCI_SW_VER2` |
| ----------- | ---------------- | ---------- | -------------- | -------------- |
| DFP-34X-2C2 | `DFP-34X-2C2`      | V2.0       | V1.0-220923    | V1.0-220304    |
| DPN-FX3060V | `DPN-FX3060V`      | A1         | V1.0.6         | V1.1.0         |
| HG8245W5    | `HG8245W5`         | 1A3D.A     | V5R020C10S202  | V5R020C10S202  |

These values define the ONU identity presented to the OLT.

---

## OMCI Configuration

| Model       | `OMCI_OLT_MODE` | `OMCI_FAKE_OK` | `OMCI_TM_OPT` |
| ----------- | --------------- | -------------- | ------------- |
| DFP-34X-2C2 | 0               | 1              | 2             |
| DPN-FX3060V | 0               | 0              | 0             |
| HG8245W5    | 3               | 1              | 2             |

OMCI compatibility and provisioning behaviour.

---

## CWMP Configuration

| Model       | `HW_CWMP_MANUFACTURER`       | `HW_CWMP_PRODUCTCLASS` |
| ----------- | ---------------------------- | ---------------------- |
| DFP-34X-2C2 | HSGQ                         | DFP-34X-2C2            |
| DPN-FX3060V | DLINK                        | DPN-FX3060V            |
| HG8245W5    | Huawei Technologies Co., Ltd | HG8245W5-6T            |

CWMP (TR-069) information exposed to management systems.

---

## Advanced Parameters

| Model       | `OMCI_VENDOR_PRODUCT_CODE` | `OMCI_VEIP_SLOT_ID` | `OMCI_CUSTOM_ME` |
| ----------- | -------------------------- | ------------------- | ---------------- |
| DFP-34X-2C2 | 15                         | 255                 | 65536            |
| DPN-FX3060V | 0                          | 14                  | 262400           |
| HG8245W5    | 29750                      | 14                  | 65792            |

Vendor-specific and OMCI implementation parameters.

---

## VLAN Configuration

| Model       | `VLAN_CFG_TYPE` | `VLAN_MANU_MODE` |
| ----------- | --------------- | ---------------- |
| DFP-34X-2C2 | 0               | 0                |
| DPN-FX3060V | 0               | 0                |
| HG8245W5    | 1               | 0                |

VLAN provisioning and management behaviour.

---

## Parameter Reference

### `GPON_SN`

GPON Serial Number used during ONU registration.

### `PON_VENDOR_ID`

Four-character vendor identifier embedded within the GPON Serial Number.

### `OMCC_VER`

OMCC protocol version advertised to the OLT.

### `GPON_ONU_MODEL`

ONU model string reported through OMCI.

### `HW_HWVER`

Hardware revision reported by the ONU.

### `OMCI_SW_VER1`

Primary software version string reported through OMCI.

### `OMCI_SW_VER2`

Secondary software version string reported through OMCI.

### `OMCI_OLT_MODE`

OMCI compatibility profile used by the RTL960x firmware.

| Value | Description                         |
| ----- | ----------------------------------- |
| `0`   | Default                             |
| `1`   | Huawei                              |
| `2`   | ZTE                                 |
| `3`   | Custom                              |
| `21`  | Force Custom (DFP-34X-2C2 specific) |

### `OMCI_FAKE_OK`

Acknowledge unsupported OMCI messages instead of returning an error.

### `OMCI_TM_OPT`

Traffic Management profile selection used by certain OMCI implementations.

### `OMCI_VENDOR_PRODUCT_CODE`

Vendor-specific ONU Product Code.

### `OMCI_VEIP_SLOT_ID`

VEIP slot identifier reported through OMCI.

### `OMCI_CUSTOM_ME`

Custom Managed Entity bitmap used by vendor-specific OMCI implementations.

### `HW_CWMP_MANUFACTURER`

CWMP (TR-069) manufacturer string.

### `HW_CWMP_PRODUCTCLASS`

CWMP (TR-069) product class identifier.

### `VLAN_CFG_TYPE`

VLAN configuration mode.

### `VLAN_MANU_MODE`

Manual VLAN provisioning mode.
