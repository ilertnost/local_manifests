# local_manifests — Xiaomi 15 (`dada`)

Local manifest for the unofficial **Evolution X 17** build for the Xiaomi 15
(`dada`, SM8750 / "sun"), applied on top of the official Evolution X manifest.

All device-specific repositories are hosted under
[`ilertnost`](https://github.com/ilertnost). The manifest replaces the AOSP,
LineageOS and Evolution X projects that need device changes and adds the trees
that are not part of the upstream manifest at all.

## Firmware

**The ROM ships no firmware partitions.** The phone must already run stock
**HyperOS 3** (`OS3.0.x`) for its own region:

- **China devices:** China HOS3, e.g. `OS3.0.307.0.WOCCNXM`. Flash it with
  `flash_all.sh` ("clean all"), never `flash_all_lock.sh`.
- **EEA/global devices:** the matching EEA/global HOS3 build.

Booting a build on the wrong region firmware (or an older HyperOS) ends in a
boot loop: the bootloader rejects the firmware of the other region. The OTA
contains no firmware, so flashing it over the correct stock firmware does not
overwrite it. The same applies when coming from an older ROM: flash the HOS3
fastboot ROM first, then install the Evolution X OTA.

## Build

```bash
repo init -u https://github.com/Evolution-X/manifest -b cnb --git-lfs
git clone -b lineage-24.0 https://github.com/ilertnost/local_manifests.git .repo/local_manifests
repo sync -c -j8 --force-sync --no-clone-bundle --no-tags

source build/envsetup.sh
lunch lineage_dada-user
m -j8 evolution
```

`repo init --git-lfs` is required: the vendor trees store their largest blobs
in Git LFS. The commands above use `-j8`; raise it if the machine has enough
RAM, lower it if r8/d8 get OOM-killed, and add swap if needed.

The kernel is built with clang `r614150`
(`TARGET_KERNEL_CLANG_VERSION` in `device/xiaomi/sm8750-common`). The prebuilt
clang repository does not carry that toolchain at the pinned revision, so put
the AOSP `clang-r614150` prebuilt into
`prebuilts/clang/host/linux-x86/clang-r614150` before building.

The finished OTA is written to
`out/target/product/dada/EvolutionX-17.0-*-dada-*.zip`; matching
`boot`/`init_boot`/`vendor_boot`/`dtbo` images are in the same directory.

## What this manifest does

| Path | Source | Why |
|---|---|---|
| `device/xiaomi/dada` | `ilertnost/device_xiaomi_dada` | Device tree |
| `device/xiaomi/sm8750-common` | `ilertnost/device_xiaomi_sm8750-common` | SM8750 common tree |
| `vendor/xiaomi/dada` | `ilertnost/vendor_xiaomi_dada` | Dada blobs |
| `vendor/xiaomi/sm8750-common` | `ilertnost/vendor_xiaomi_sm8750-common` | SM8750 blobs |
| `kernel/xiaomi/sm8750` | `ilertnost/android_kernel_qcom_sm8750` | Kernel (`lineage-24.0-oss`) |
| `kernel/xiaomi/sm8750-devicetrees` | `ilertnost/android_kernel_xiaomi_sm8750-devicetrees` | Kernel DT |
| `kernel/xiaomi/sm8750-modules` | `ilertnost/android_kernel_xiaomi_sm8750-modules` | Kernel modules |
| `hardware/xiaomi` | `ilertnost/android_hardware_xiaomi` | Fingerprint, MiSys camera, IR HALs (not upstream) |
| `hardware/qcom-caf/sm8750/display/hal` | `ilertnost/android_hardware_qcom_display` | CAF display hal with panel size and RGB notifier fixes |
| `vendor/qcom/opensource/display-core` | `ilertnost/android_vendor_qcom_opensource_display-core` | Open-source SDM/SDE-DRM core (not upstream) |
| `vendor/qcom/opensource/display-intf` | `ilertnost/android_vendor_qcom_opensource_display-intf` | Open-source display interfaces (not upstream) |
| `frameworks/av` | `ilertnost/android_frameworks_av` | Xiaomi camera implementation-defined stream sizes |
| `frameworks/native` | `ilertnost/android_frameworks_native` | Protected camera layer fix |
| `external/crosvm` | `ilertnost/android_external_crosvm` | qcrosvm crates exposed to the platform |
| `external/rust/rutabaga_gfx` | `ilertnost/android_external_rust_rutabaga_gfx` | Trusted VM graphics |
| `packages/modules/Virtualization` | `ilertnost/android_packages_modules_Virtualization` | Trusted VM apex |
| `vendor/qcom/opensource/crosvm-gunyah` | `xiaomi-dada/android_vendor_qcom_opensource_crosvm-gunyah` | Qualcomm qcrosvm launcher (not upstream) |
| `hardware/dolby` | [`ilertnost/android_hardware_dolby`](https://github.com/ilertnost/android_hardware_dolby) | Dolby Atmos with `dada` ODM tuning and Xiaomi prebuilt AIDL ABI |
| `packages/modules/Bluetooth` | [`ilertnost/android_packages_modules_Bluetooth`](https://github.com/ilertnost/android_packages_modules_Bluetooth) | QTI aptX Adaptive offload, `AVDT_CODEC_SIZE` and LE Audio HAL fixes |

The CAF copies of `hardware/qcom-caf/sm8750/display/{core,intf}` are removed:
the open-source `vendor/qcom/opensource/display-*` projects provide the same
modules, and keeping both makes kati fail with duplicate module definitions.
