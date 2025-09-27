# BLE Sniffing — Step-by-Step README

> Quick guide for capturing and analyzing Bluetooth Low Energy (BLE) advertising packets using the **nRF52840 USB Dongle**, **nRF Sniffer**, and **Wireshark**.  
> Based on the user's "Master BLE Sniffing" notes.

---

## Table of contents
1. Overview
2. Requirements (hardware & software)
3. Prep & flashing the dongle
4. Integrate nRF Sniffer with Wireshark
5. Start a capture (advertising packets)
6. Make analysis easier (toolbar, columns, coloring, profile)
7. Useful Wireshark display filters (examples)
8. Troubleshooting & tips
9. Next steps & further reading
10. License / Attribution

---

## 1. Overview
This README covers a practical workflow to sniff and analyze BLE **advertising** packets — a great place to start because all BLE devices advertise before connecting. The setup uses the low-cost Nordic nRF52840 USB Dongle + Nordic's nRF Sniffer firmware + Wireshark for capture and analysis.

---

## 2. Requirements

### Hardware
- **Nordic nRF52840 USB Dongle** (or any nRF52 DK that supports the current sniffer firmware).  
  _Note: avoid older nRF51 dongles — they miss newer BLE features._

### Software (minimum suggestions)
- **nRF Sniffer for Bluetooth LE** (Nordic)
- **nRF Connect for Desktop** (to flash the dongle)
- **SEGGER J-Link Software** (required for programmer support)
- **Wireshark** (version compatible with your OS)
- **Python 3.6+** (required by some sniffer utilities)

OS: Windows, macOS, Linux — all supported (confirm Wireshark prerequisites for your OS).

---

## 3. Prep & flashing the dongle (high level)
1. Install **nRF Connect for Desktop**.
2. Install **SEGGER J-Link** tools.
3. Download the **nRF Sniffer** package (contains sniffer firmware `.hex` and Wireshark integration files).
4. Open **nRF Connect → Programmer** (or the Programmer app inside nRF Connect for Desktop).
5. Plug in the **nRF52840 USB Dongle** (it should appear in the programmer).
6. Flash the **sniffer firmware hex** onto the dongle using the Programmer app.
7. Confirm the dongle enumerates as expected.

> After flashing, the dongle will act as your BLE radio for Wireshark capture.

---

## 4. Integrate nRF Sniffer with Wireshark
1. Install **Wireshark**.
2. Copy the **nRF Sniffer** plugin files into Wireshark's plugin/protocol folder as instructed by Nordic (the sniffer package normally includes instructions).  
3. Optionally copy the **nRF Sniffer Wireshark Profile** (if provided) into your Wireshark profiles folder.

> Nordic's documentation / user guide includes specific folder paths for each OS. Follow those steps when present in the sniffer package.

---

## 5. Start a capture (advertising packets)
1. Launch **Wireshark**.
2. Select the **nRF Sniffer interface** from the interface list (it will appear once the plugin is installed and the dongle is plugged in).
3. Click the **start/green** capture button.
4. Wireshark should show BLE traffic (lots of packets — many devices advertise around you).
5. Click the **red** stop button to stop capturing.

> If data scrolls off-screen too fast, disable Wireshark's auto-scroll (use the capture view icon to toggle auto-scrolling) so you can examine packets without interruption.

---

## 6. Make analysis easier

### Enable the nRF Sniffer toolbar
Enable the nRF Sniffer toolbar in Wireshark so you can:
- select interface/dongle (useful with multiple dongles),
- select device via Bluetooth address or IRK,
- pick keys for decrypting connections,
- select RF channels to hop on (adv channels: **37, 38, 39**).

(Enable the toolbar from Wireshark's menu for toolbars — the exact menu may vary slightly by Wireshark version.)

### Add helpful columns
Right-click a packet field in the packet details pane → **Apply as Column**. Useful columns:
- Device Name (`btcommon.eir_ad.entry.device_name`)
- Manufacturer Specific Company ID (`btcommon.eir_ad.entry.company_id`)
- RSSI (`nordic_ble.rssi`)
- Channel Index, Advertising Type, Packet time/delta

Adding these makes it much easier to find and track devices.

### Colorize important packets
Use coloring rules to highlight e.g. **bad CRC** packets:
- Find a packet with bad CRC → expand `nRF Sniffer for Bluetooth LE` → Flags → CRC status → right-click → **Colorize with Filter** → choose a color (e.g., red for bad CRC).
This helps visually spot transmission problems quickly.

### Use / Install a Wireshark Profile
A tuned profile (columns, buttons, filters, layout) speeds your analysis. You can import a profile via:
- Wireshark → bottom right → click current profile → **Import from zip** → select profile zip.

A good advertising profile includes columns for:
- Packet No., Transmit Time, Delta, Source/Dest Address, RSSI, Advertising Type, Channel Index, Company ID, Device Name.

---

## 7. Useful Wireshark display filters (examples)
Use display filters to focus on devices/packets you care about. Paste these into Wireshark's Display Filter bar:

```text
# Show packets from a specific Bluetooth address
btle.advertising_address == 06:05:04:03:02:01

# Show scans that contain a device name "MySensor"
btcommon.eir_ad.entry.device_name == "MySensor"

# Only connectable advertising packets
btle.advertising_header.pdu_type == 0x0

# Only non-connectable advertising packets
btle.advertising_header.pdu_type == 0x2

# Packets from advertising channels 37 and 38 (exclude 39)
nordic_ble.channel == 37 || nordic_ble.channel == 38

# RSSI threshold
nordic_ble.rssi >= -70

# Packets with a bad CRC
nordic_ble.crc.bad == 1

# Packets with valid CRC
nordic_ble.crcok == 1

# Manufacturer-specific data by company ID (replace ID)
btcommon.eir_ad.entry.company_id == 0x004C
```

> You can combine filters with `&&` and `||`. Applying multiple filters is easiest from the filter bar (right-click based filters can override previously applied filters).

---

## 8. Troubleshooting & tips

- **Too much noise / many devices**: Use filters for RSSI (closer devices), device name, or manufacturer ID to narrow down.
- **Randomized addresses (phones)**: Smartphones often use random MACs — match by device name or manufacturer payload if present.
- **Bad CRCs**: Could be range/noise. Colorizing bad CRC helps find weak transmitters.
- **Can't see packets**: Ensure sniffer firmware is flashed correctly and Wireshark plugin files were copied to the correct Wireshark folders. Check that Python & J-Link are installed if required by the sniffer package.
- **Decrypting connection packets**: Use the nRF Sniffer toolbar to input pairing/bonding keys (type & value) so Wireshark can decrypt encrypted traffic during sniffing.

---

## 9. Next steps / advanced topics
- Sniffing **BLE connections** (timing + pairing/bonding decryption) — more advanced, will require keys and careful channel hopping.
- Analyze advertising interval, manufacturer-specific data payloads, and reverse-engineering beacon formats (iBeacon, Eddystone).
- Automate captures or parse PCAPs programmatically with Python (e.g., `pyshark` or `scapy`).

The original notes promise deeper tutorials on these topics — consider them as follow-ups.

---

## 10. References
- Nordic Semiconductor nRF52840 Dongle:
  https://www.nordicsemi.com/Products/Development-hardware/nrf52840-dongle

- Nordic Semiconductor nRF Connect for Desktop:
  https://www.nordicsemi.com/Products/Development-tools/nRF-Connect-for-Desktop

---

