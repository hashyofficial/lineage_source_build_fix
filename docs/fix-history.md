# Build-fix history and scope

Device: Xiaomi Redmi Pad (yunluo). Target: unofficial LineageOS 24.0 / Android 17 / cp2a / user. Host during investigation: wattOS R13 (Debian 12), Intel i3-12100, 16 GB-class RAM.

## Actual source fixes committed to hashyofficial/main

| Area | Symptom | Resolution |
| --- | --- | --- |
| vendor/xiaomi/yunluo/Android.bp | Soong imports non-existent hardware/lineage/compat namespace | Remove only that namespace import |
| hardware/mediatek/Android.bp | libpixelusb-aidl inaccessible to MediaTek namespace | Add hardware/google/pixel/usb to Soong namespace imports |
| hardware/mediatek/Android.bp | libpixelstats and pixelatoms-cpp inaccessible | Add hardware/google/pixel/pixelstats to imports |
| vendor/xiaomi/yunluo/Android.bp | Proprietary codec2 libformatter resolves to wrong (host-only) library variant | Rename prebuilt Soong module to libformatter_mtk, add stem: "libformatter", change the two codec2 shared_libs |
| vendor/xiaomi/yunluo/yunluo-vendor.mk | Product package still references old module name | Change package to libformatter_mtk |
| device/xiaomi/yunluo/BoardConfig.mk | Missing vendor/lineage/config/device_framework_matrix.xml | Remove absent matrix reference, retain MediaTek/Xiaomi matrices |
| device/xiaomi/yunluo/device.mk | Soong select on charging_control_supports_bypass expects bool but receives string | Use soong_config_set_bool(...,false) |
| device/xiaomi/yunluo/device.mk | Make reports android.hardware.power-service.pixel-libperfmgr as missing | Add hardware/google/pixel/power-libperfmgr to PRODUCT_SOONG_NAMESPACES; retain the power service |
| local_manifests/mt6789.xml | Old manifest referenced original developers / lineage-22.0 | Point device/vendor/MediaTek repos to hashyofficial/main and Xiaomi hardware to LineageOS/lineage-24.0 |

The five source-file transformations (device tree, vendor, hardware) are replayable with scripts/apply-known-fixes.py. The manifest is maintained in a separate repository and should be obtained from hashyofficial/local_manifests before syncing.

## Milestones confirmed during local compilation

1. Lunch succeeds as lineage_yunluo-cp2a-user, Android 17 / LineageOS 24.0.
2. Soong finished Android.bp analysis after a long, swap-heavy run.
3. First identified failure: health boolean type mismatch. Fixed on hashyofficial device.mk.
4. Next identified failure: missing Pixel power service in PRODUCT_PACKAGES due to namespace visibility. Fixed on hashyofficial device.mk.
5. **The full ROM build has not been confirmed successful.** Further errors may appear. No boot validation has occurred.

## Non-automatic / experimental actions

- external/libjxl/Android.bp: a local attempt removed vendor_available: true following missing vendor variants of libskia_skcms and libhwy. It was a workaround in an earlier environment, not proven correct long-term. Do not blanket-apply; instead check the exact error again in the current tree.
- build/soong/cmd/soong_build/main.go: a temporary test imported runtime/debug and called SetGCPercent(40) and SetMemoryLimit(12 << 30). The build remained highly memory-constrained even after this experiment. Do not apply automatically. Prefer comparing against an unmodified Soong checkout after getting a baseline.
- USB/thermal Soong namespace imports, codec2 libraries and Pixel power HAL were fixed to satisfy source dependencies, but functionality on a flashed tablet is **not yet verified**.
- Do not set ALLOW_MISSING_DEPENDENCIES=true to conceal genuinely missing modules. Investigate each failure instead.

## Build practices

- Do not perform mka clean, remove out/, or repo sync simply to retry a single config fix. Change only the faulty source line and rerun the same build target.
- When reusing on a different release, validate original module and dependency names again.
- Keep developer-source updates in their own Git repositories; this toolkit contains patching instructions and history.
