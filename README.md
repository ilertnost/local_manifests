# local_manifests — Xiaomi 15 (`dada`)

Repo local manifest for the unofficial LineageOS port of the Xiaomi 15
(`dada`, SM8750 / "sun"), built on top of official LineageOS.
Device-specific blobs come from `OS3.0.305.0.WOCEUXM`.

## Build

```bash
repo init -u https://github.com/LineageOS/android.git -b lineage-24.0 --git-lfs
git clone -b lineage-24.0 https://github.com/xiaomi-dada/local_manifests.git .repo/local_manifests
repo sync -c -j"$(nproc --all)" --force-sync --no-clone-bundle --no-tags

source build/envsetup.sh
lunch lineage_dada-cp2a-userdebug
m bacon
```

`repo init --git-lfs` is required: both vendor trees and `vendor/gapps` store
their largest blobs in git-lfs, and without it they sync as pointer files.

For the build with GApps built in (MindTheGapps), set `WITH_GMS=true`. Keep each
build in its own output directory so the two never mix, and make `out` a symlink
to the one being built: LineageOS builds the kernel with relative paths only when
`OUT_DIR` is `out`, and Siso needs a relative `OUT_DIR` inside the source tree.

```bash
mkdir -p out-vanilla out-gapps
ln -sfn out-gapps out
export WITH_GMS=true
lunch lineage_dada-cp2a-userdebug
m bacon
```

## What this manifest does

| Path | Source | Why |
|---|---|---|
| `device/xiaomi/{dada,sm8750-common}` | `xiaomi-dada` | device trees |
| `vendor/xiaomi/{dada,sm8750-common}` | `xiaomi-dada` | proprietary blobs |
| `vendor/gapps` | MindTheGapps | GApps for the `WITH_GMS=true` build |
| `kernel/xiaomi/sm8750{,-devicetrees,-modules}` | `xiaomi-dada` | kernel, device trees and techpack modules, built from source (latest CLO for SM8750 plus LineageOS and Xiaomi changes) |
| `hardware/xiaomi` | `xiaomi-dada` fork | ultrasonic fingerprint HAL modules, MiSys camera file service, SPI IR HAL, PowerShare HAL |
| `hardware/qcom-caf/sm8750/display/{core,hal,intf}` | `xiaomi-dada` forks | display attributes by hardware config index, light sensor frame notifier, panel size in tenths of a millimetre |
| `frameworks/av` | `xiaomi-dada` fork | vendor stream sizes for the Xiaomi camera |
| `frameworks/native` | `xiaomi-dada` fork | protected camera layers next to secure camera planes |
| `external/crosvm`, `external/rust/rutabaga_gfx`, `packages/modules/Virtualization`, `vendor/qcom/opensource/crosvm-gunyah` | `xiaomi-dada` forks | trusted VM launcher (qcrosvm) |

`hardware/xiaomi` and `crosvm-gunyah` are not in the upstream manifest, so they
are added rather than replaced. The other repositories are, so they are removed
first.
