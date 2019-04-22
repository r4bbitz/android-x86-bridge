# android_vendor_google_chromeos
This project provides scripts to extract (proprietary) files
from the Android container in ChromeOS recovery images,
for use in x86-based Android installations.

These files (mainly binaries) complement what is available in AOSP,
and provide the following functionality:

 - **Widevine:** DRM system from Google used in certain media streaming
   apps like Netflix. ChromeOS contains a HIDL HAL implementation that provides
   security level L3 (software decoding).
 - **Houdini:** ARM on x86 native bridge by Intel. Allows running apps built
   only for ARM platforms on x86-based devices.

Currently, this project extracts files suitable for Android 9.0 (Pie).
Older ChromeOS versions contain files for Android 7 (Nougat),
but ChromeOS skipped Android 8 (Oreo) entirely.

## Usage
In general, the scripts can be used in any Linux distribution.
Run `./extract-files.sh` to download the ChromeOS recovery image
and extract it to a folder called `proprietary`.

**Note:** The script requires `sudo` to mount the partitions from the
ChromeOS recovery image.

The resulting subdirectories (`widevine` and `houdini`) contain a directory
structure that can be copied as-is to the `/system` partition of the target
device.

### Flashable ZIP
`zip` contains scripts to build flashable ZIP packages that can be flashed
through the recovery to install the additional proprietary files.
This is recommended for all Android ROMs that are installed through the recovery.

Run `zip/build.sh` after extracting files to build the ZIPs to `zip/out`.

This project does not provide pre-built packages. However, the ZIPs _should_
be built reproducible. As such, the expected hash-sums are listed as
tags/releases in this project.

**Note:** Some build options need to be set for the Android build even when
using this option to install Houdini optionally.  
(This is not necessary for Widevine.) See [Make files](#make-files) below.

### Make files
`board` and `target` contain make files that can be used to bundle the
proprietary files in an Android build.

  - **Houdini:**
    `board/native_bridge_arm_on_x86.mk` must be included from `BoardConfig.mk`
    even if a native bridge should be optionally installed. It configures the
    build system to build additional code needed to run ARM apps.

    - `BoardConfig.mk`: Always required.

        ```make
        -include vendor/google/chromeos/board/native_bridge_arm_on_x86.mk
        ```

    - `device.mk`: Optional: Bundle Houdini directly with the Android build.

        ```make
        # WITH_NATIVE_BRIDGE := true
        $(call inherit-product-if-exists, vendor/google/chromeos/target/houdini.mk)
        ```

  - **Widevine:** `device.mk`: Bundle Widevine directly with the Android build.

    ```make
    $(call inherit-product-if-exists, vendor/google/chromeos/target/widevine.mk)
    ```

### SELinux Policy
`sepolicy` contains a base policy for both Widevine and Houdini and will be needed
to use them in enforcing mode. It should be included in the base system since
the ZIPs do not attempt to patch the prebuilt policy.

```make
BOARD_SEPOLICY_DIRS += vendor/google/chromeos/sepolicy
```

## License
Please see [`LICENSE`](/LICENSE).
