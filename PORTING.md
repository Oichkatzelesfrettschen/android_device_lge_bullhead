# Nexus 5X source admission for LineageOS 22.2

The device starts from nexus5x-dev lineage-21.0 at
`5fbaea520b210a9689f3e5b0256be080db9442a8`. The matching kernel is
`995f521bafa30055079fcb19eca37b360c037932`; the matching vendor is
`ddf72b76c78c9a08a2ea10d4b631de729914290b`. These inputs describe a
community Android 14 donor. Android 15 compilation and device acceptance
remain separate gates.

## Source ownership

The private Android workbench owns the candidate Android-path overlay at
`notes/bullhead-lineage22-local-manifest.xml` and its dependency audit at
`notes/BULLHEAD_BUILD_SOURCE_OWNERSHIP.md`. Contributors need access to those
inputs or an equivalent reviewed composition before selecting the product.
The dependency list stays empty because inherited automatic requests select a
different kernel/vendor generation, and Android 15 roomservice prefixes
non-AOSP repository names with `LineageOS/`.

The candidate overlay pins the device, kernel, vendor, timekeep, MSM8994-family
CAF audio/display/media, and nanohub sources. The source-import XML uses separate
research paths. Neither XML establishes a complete resolved Android build:
GPS, Qualcomm path selection, legacy platform interfaces and partition packaging
remain explicit admission gates. Preserve vendor binaries under their existing
redistribution terms.

## Sensor module contract

Android 15 nanohub exports `sensors.nanohub` and consumes its sensor list through
`nanohub_sensorhal.sensorlist_library`. The device links the unchanged bullhead
sensor list through the fully qualified `libsensorlist_bullhead` module, retains
lid-state and direct-report flags, and selects the installed HAL with
`ro.hardware.sensors=nanohub`. Explicit qualification crosses the device's Soong
namespace without adding a global duplicate. The host syntax check uses the
actual Android headers and `-Wall -Wextra -Werror`; product linking, event rates,
direct channels, flush completion and suspend/wake remain device gates.

## ART and kernel contract

`PRODUCT_ENABLE_UFFD_GC=false` selects ART artifacts compatible with the
kernel's missing userfaultfd and MREMAP_DONTUNMAP interfaces. The mechanism
follows the same-generation Nexus 6P Android 15 product at
`de673dadd95e7ced3e717f6f8bcc55d49044c4a6`. Runtime acceptance checks odrefresh
artifacts across two boots and an OTA, peak dex2oat RSS and launch latency.
A later kernel change reopens the collector decision only after its complete
userfaultfd interface passes the ART kernel feature checks.

## Graphics preload boundary

The shipped 64-bit Adreno EGL loader calls InitEsx from eglGetDisplay. The
matched GLES driver writes the AArch64 x18 register on InitEsx's successful
path and returns without restoring the caller's value. Zygote's GL preload
therefore crosses a legacy ABI boundary before application forks.
`ro.zygote.disable_gl_preload=1` prevents that shared-process initialization;
applications retain the Adreno renderer. The workbench retains the library
hashes, bounded disassembly and Android 15 call path in the source-admission
evidence at `evidence/bullhead-source-admission/graphics-abi.txt`. Static proof supports the setting; Bluetooth startup, application
rendering, frame timing and repeated context lifecycles remain device gates.

## Product and runtime gates

The product retains both ARM64 and ARM32, the 32-bit camera HAL, Adreno EGL/GLES,
Vulkan declarations, hardware OMX video components, DSP compressed-audio routes,
legacy ABI shims, thermal/BCL policy and the inherited 1 GiB LZ4 zram policy.
Two identical protobuf copy declarations collapse into one declaration per ABI;
the source and destination of each installed library remain identical.

The user-build verity fstab remains a separate unresolved contract: device.mk
names a missing root path, while rootdir/etc/fstab-verity.bullhead mounts a
standalone vendor partition and carries an older encryption policy that differs
from the selected embedded-vendor product. Correcting only the filename creates
an inconsistent boot image. Resolve partition, encryption and module ownership
together before admitting user images.

Source validation checks JSON parsing, exact donor object IDs, unique protobuf
copy destinations and the build-system consumer of PRODUCT_ENABLE_UFFD_GC.
Full product configuration, compilation, boot, Enforcing policy and feature
lifecycle checks remain required. Hardware throughput and RAM improvements
require matched measurements on the exact installed artifacts.
