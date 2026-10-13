# Nexus 5X source admission for LineageOS 22.2

The device starts from nexus5x-dev lineage-21.0 at
`5fbaea520b210a9689f3e5b0256be080db9442a8`. The matching kernel is
`995f521bafa30055079fcb19eca37b360c037932`; the matching vendor is
`ddf72b76c78c9a08a2ea10d4b631de729914290b`. These inputs describe a
community Android 14 donor. Android 15 compilation and device acceptance
remain separate gates.

## Source ownership

The public [dependency manifest](manifests/lineage-22.2.xml) pins every
additional device dependency selected for the candidate. After checking out
this device tree at `device/lge/bullhead` in the Android source root, install
the manifest before product configuration:

```sh
mkdir -p .repo/local_manifests
cp device/lge/bullhead/manifests/lineage-22.2.xml .repo/local_manifests/bullhead.xml
repo sync -c
```

The base manifest is
`LineageOS/android@98c181d61f032a69c817835238cbb40fd049100e`. Use a dedicated
source tree with a Repo-managed device project at the selected device commit,
keep a single bullhead dependency overlay, and record every project revision
with `repo manifest -r` after sync. An unmanaged Git clone requires its own
project entry before a complete Repo lock can represent the device revision.
The dependency overlay deliberately leaves the existing device checkout in
place, so its own commit does not require a circular self-pin.

The dependency list stays empty because inherited automatic requests select
a different kernel/vendor generation, and Android 15 roomservice prefixes
non-AOSP repository names with `LineageOS/`. XML supplies the exact repositories,
revisions, replacements and generated links that roomservice cannot express.
The private workbench supplies an equivalent candidate overlay with an explicit
device pin, analysis receipts and the ranked roadmap; access to that workbench
is optional for obtaining the public source dependencies.

The legacy common source maps MSM8992 into the MSM8994 CAF family.
`BOARD_USES_QCOM_HARDWARE=true` enables the real path dispatcher. The manifest
preserves the base common project's generated links and adds the selected
family's namespace and Make entry point. GPS occupies the CAF family directory
because the Qualcomm guard skips legacy Make projects under `hardware/qcom`.
The retained GPS flags select the donor's MSM8994 implementation and export
`gps.msm8992`. `TARGET_USES_AOSP=true` remains the camera's VANILLA_HAL selector.

These pins establish candidate source ownership. Legacy platform interfaces,
warning cleanup in imported code, partition packaging, full module linking and
runtime qualification remain admission gates. Preserve vendor binaries under
their existing redistribution terms.

## Sensor module contract

Android 15 nanohub exports `sensors.nanohub` and consumes its sensor list through
`nanohub_sensorhal.sensorlist_library`. The device links the unchanged bullhead
sensor list through the fully qualified `libsensorlist_bullhead` module, retains
lid-state and direct-report flags, and selects the installed HAL with
`ro.hardware.sensors=nanohub`. Explicit qualification crosses the device's Soong
namespace without adding a global duplicate. The host syntax check uses the
actual Android headers and `-Wall -Wextra -Werror`; product linking, event rates,
direct channels, flush completion and suspend/wake remain device gates.

## Camera media-header ownership

Camera Makefiles obtain legacy metadata and OMX include paths through
`project-path-for,qcom-media`. The selected MSM8994 CAF media project owns
`libstagefrighthw/QComOMXMetadata.h` and `mm-core/inc`; the inherited hardcoded
`hardware/qcom/media/msm8974` directory falls outside the candidate manifest.
The path dispatcher keeps camera metadata tied to the selected codec source.
Source-path evaluation establishes header ownership; compilation, ARM32 binary
layout, recording-buffer lifetime and camera/encoder lifecycle remain gates.

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
