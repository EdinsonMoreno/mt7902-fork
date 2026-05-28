# mt7902e

This is basically the mainline version of mt76 with [patches to support MT7902](https://lore.kernel.org/all/20260219004007.19733-1-sean.wang@kernel.org/) & firmware provided by Mediatek. I also stripped down all the unnecessary files & other hardware support to ensure only MT7902 card is supported.

> [!WARNING]
> This out-of-tree driver only support the PCIe version of MT7902, for SDIO support it's better if you just merge Mediatek patches on your own !

> [!TIP]
> For Bluetooth support, check out [this branch](https://github.com/hmtheboy154/mt7902/tree/bluetooth_backport).


## Status

The driver supports kernel 6.6+ (including 7.0) and is usable according to users reported in this [spreadsheet](https://docs.google.com/spreadsheets/d/1G2mQEeLQAu4oB85G-y4A9OduA1ZP0rUcY-b6MRnZhFU/edit?usp=drive_link&pli=1&authuser=0).

## Patches in this fork

This fork includes the following fixes on top of upstream:

- **`mt76_get_txpower()` reports 0 dBm** ([commit 48a0589](../../commit/48a0589)): On systems where the ACPI SAR table exposes an invalid power limit (`frp[i].power == -1`), `mt76_get_sar_power()` capped TX power at -0.5 dBm, causing the driver to always report 0 dBm. Fixed by falling back to `chan->max_power` (regulatory limit) when `txpower_cur` has not been initialized by firmware.

- **`mt76_vif_phy()` returns NULL, STA insertion fails with -22** ([commit 964b718](../../commit/964b718)): On single-radio setups or during early STA association events, `mlink->ctx` may not be assigned yet. Returning `NULL` propagated as `-EINVAL` to callers, causing `failed to insert STA entry for the AP (error -22)`. Fixed by returning `hw->priv` instead, consistent with the upstream behavior for kernels >= 6.15.

- **Build failure on kernel >= 6.17** ([commit ca26d5a](../../commit/ca26d5a)): `pp_page_to_nmdesc()` was introduced in 6.13 and removed in 6.17 in favour of `__netmem_get_pp()` + `page_to_netmem()`. The previous guard used `pp_page_to_nmdesc` under `>= 6.17`, causing a build error on that exact version. Replaced with a three-way guard covering `>= 6.17`, `>= 6.13`, and `< 6.13`. (Based on [PR #14](https://github.com/hmtheboy154/mt7902/pull/14) by [@georgettica](https://github.com/georgettica).)

## Installation

> [!IMPORTANT]
> Before building & installing this driver, remember to install essential packages to build a kernel driver like linux kernel's headers & toolchain. I will not cover it here.

- Get the source using `git`

```bash
git clone https://github.com/hmtheboy154/mt7902
cd mt7902
```

- To only build the driver, use this command

```bash
make -j$(nproc)
```

- To build the driver & install it:

```bash
sudo make install -j$(nproc)
```

> [!TIP]
> As per the [20260309](https://gitlab.com/kernel-firmware/linux-firmware#linux-firmware) release, the firmware should be already provided by your distribution, for Debian as of present, check in the unstable repo.

- To install the firmware required for the driver:

```bash
sudo make install_fw
```

- To remove the driver:

```bash
sudo make uninstall
```

- To remove the firmware:

```bash
sudo make uninstall_fw
```

Once you got the driver & firmware installed, reboot to see changes.


