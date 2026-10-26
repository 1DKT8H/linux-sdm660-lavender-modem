# SDM660 Mainline IPA Modem Port (Xiaomi Redmi Note 7)

Patch series and kernel packaging to enable the Qualcomm IPA (v2.6L) modem data path on mainline Linux (postmarketOS, kernel 7.0.x) for the Xiaomi Redmi Note 7 (*lavender*, Qualcomm Snapdragon 660 / SDM660).

## Project Goal

The primary goal of this project was to bring up the cellular modem data path under mainline Linux on the Redmi Note 7 to repurpose the device as a low-power, standalone 4G/LTE router (providing internet access via Wi-Fi hotspot or USB tethering).

While basic modem functions (AT commands, SMS, audio calls) can work through shared memory / QRTR / BAM-DMUX on some Qualcomm platforms, high-speed mobile data on SDM660 requires the Qualcomm IP Accelerator (IPA v2.6L) hardware block to route IP packets between the Application Processor (AP) and the modem Hexagon DSP (MSS).

## Current Status

### What Works

* **Kernel & System Boot:** Mainline kernel 7.0.x boots reliably on *lavender* using postmarketOS. USB networking and SSH access work out of the box.
* **Modem Initialization:** Qualcomm Hexagon modem remoteproc (`qcom-q6v5-mss`) loads firmware and boots without crashing.
* **Cellular Control Plane:**
  * ModemManager detects the modem, reads the SIM card, and registers on the LTE network (tested with Aero2 / Plus in Poland).
  * Data bearer / PDP context activation succeeds over QMI.
  * QMAP multiplexing interface (`qmapmux0.0`) is spawned with an assigned carrier IPv4 address, subnet mask, and default gateway.
* **Hardware & Stability Fixes:**
  * **IPA Hardware Reset:** Configured component reset and active clock scaling matching downstream SDM660 register values (`COMP_CFG=0x11`).
  * **SMMU Translated Domain:** Replaced bypass/identity client mapping with translated DMA domain for AP queues, resolving initial IOVA 0 translation faults and `BAD_SNOC_ACCESS`.
  * **System Header Table:** Ported DRAM system header table allocation (`IPA_HDR_INIT_SYSTEM`), fixing the modem firmware crash (`ipa_hal.c:2163:IPA Assert`).
  * **Driver Lifecycle & SSR:** Fixed device resource cleanup order and NULL dereference races during Subsystem Restart (SSR).
  * **Filter Rule Installation:** Implemented QMI filter rule decoder for legacy IPA v2.x enums; all 24 modem-requested uplink rules (11 IPv4, 13 IPv6) are accepted and acknowledged by the modem firmware.

### What is Missing / Current Roadblock

* **Uplink Packet Transmission (TX):** Transmitting packets (e.g. sending an ICMP ping to `1.1.1.1` bound to `qmapmux0.0`) does not complete out to the network.
  * Packets trigger an AP BAM bus access error on pipe 4 (`ctrl=0x3112 addr=...`), indicating a descriptor or FIFO address/sizing discrepancy on the AP-to-IPA DMA boundary.
  * Neither matching the downstream 0x1000 TX descriptor FIFO size nor explicit DMA cache synchronization cleared the BAM error.
* **Downlink (RX):** Inbound traffic reception remains unverified until outgoing traffic can traverse the IPA pipe.
* **Router Setup:** Network routing, NAT masquerading (`nftables`), DHCP server (`dnsmasq`), and Wi-Fi hotspot (`hostapd`) were deferred pending working bidirectional packet flow.

## Patch Series & Revisions

* **Patch count:** 28 patches (`01-ipa2-lite-driver.patch` to `28-sdm660-ipa-sync-tx-buffer.patch`).
* **Package revisions & builds:** The postmarketOS kernel package went through multiple iterative test builds from `r44` up to `r61` (`pkgrel=61`, kernel build `#62`). Milestone builds included `r50` (kernel build `#51`, system-header fix), `r52` (kernel build `#53`, QMI filter decoding), and `r61` (final BAM DMA sync and FIFO adjustments).

### Patch Summary:
* `01-03`: Initial `ipa2-lite` driver import and mainline SDM630/SDM660 device tree IPA / uC SMMU node bindings.
* `04-06`: Downstream SDM660 IPA 2.6L component reset values, bus interconnect voting, and filter engine enablement.
* `07-13`: QMAP big-endian header handling, uC event/response interrupt handling, downstream-matching TX completion polling, and system pipe routing.
* `14-17`: Driver lifecycle fixes, BAM error capture, SMMU translated domain enablement, and system-memory header table initialization.
* `18-22`: QMI uplink filter rule decoding, legacy action handling, and filter pointer re-application after data call initiation.
* `23`: PM8998/PM660 PS_HOLD power-off handling.
* `24-26`: Filter table reassertion and unconditional catch-all rule encoding.
* `27-28`: AP TX descriptor FIFO adjustment and DMA buffer synchronization.

## Device Package (`device-xiaomi-lavender`)

The accompanying `device-xiaomi-lavender/` directory contains the postmarketOS device package with `ipa2-lite` enabled in `modules-initfs` (for Tianma, Shenchao, and BOE display panel variants) to ensure the IPA driver is loaded early during the initramfs stage.

## Building with pmbootstrap

This repository mirrors the `linux-postmarketos-qcom-sdm660` package structure:

1. Clone this repository into your `pmaports` tree (or overlay):
   ```bash
   cp -r . /path/to/pmaports/device/testing/linux-postmarketos-qcom-sdm660/
   ```

2. Build the kernel package:
   ```bash
   pmbootstrap build linux-postmarketos-qcom-sdm660 --force
   ```

3. Export a boot image:
   ```bash
   pmbootstrap export
   ```

4. Temporary boot via fastboot without flashing:
   ```bash
   fastboot boot /tmp/postmarketOS-export/boot.img
   ```
