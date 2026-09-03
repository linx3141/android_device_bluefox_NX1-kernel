# Bluefox NX1 kernel prebuilts

Prebuilt stock kernel artifacts for the Bluefox NX1 (MediaTek Helio G81,
MT6768 platform), consumed by `device/bluefox/NX1/BoardConfig.mk`.

Layout:

| Path | Contents |
|---|---|
| `Image.gz` | stock kernel image (GKI-style, `BOARD_KERNEL_IMAGE_NAME`) |
| `dtbo.img` | stock DTBO image (`BOARD_PREBUILT_DTBOIMAGE`) |
| `dtb/nx1.dtb` | device DTB packed into boot (`BOARD_PREBUILT_DTBIMAGE_DIR`) |
| `vendor_ramdisk_modules/` | first-stage (eMMC/UFS/PMIC/...) modules + `modules.load` |
| `vendor_dlkm/lib/modules/` | vendor dlkm modules + `modules.load` |
| `system_dlkm/lib/modules/` | system dlkm modules + `modules.load` |

## Module loading

`BoardConfig.mk` installs exactly the modules listed in each `modules.load`
(in list order, which mirrors the stock load order - do not alphabetize) plus
the WMT/connsys modules explicitly appended for the vendor dlkm image:

* `vendor_ramdisk_modules/modules.load`: every `.ko` in that directory
  (148/148) is staged into the vendor ramdisk; init cannot find any block
  device without them (order matters, see BoardConfig comment).
* `vendor_dlkm/lib/modules/modules.load`: 166 modules, plus the WMT extras
  listed in BoardConfig (`wmt_drv.ko`, `wlan_drv_gen4m_6768.ko`, ...).
* `system_dlkm/lib/modules/modules.load`: 27 modules. The original stock
  list was trimmed to the 27 modules that are actually required on this
  tree - verified during bring-up, do not blindly restore the full list.

Keeping the directories in sync: when a module is added/removed, update both
the `.ko` file and the matching `modules.load` entry together. Modules that
are not listed are never installed into the image.

## Provenance / updating

Artifacts come from the stock build
`BLUEFOX-NX1/T69_S39_BLUEFOX_NX1_B1USA_20260112` (boot.img / vendor_boot.img
/ super.img in the vendor dump). The old full `system_dlkm` module set is
kept in this repo's git history if a module ever needs to be restored.

To refresh from a new stock release: unpack the stock images, replace
`Image.gz`/`dtbo.img`/`dtb/` and the three module directories, then rerun the
BoardConfig `modules.load` checks above.
