# Meta Quest 2 Root Research

A collection of my findings while exploring the Meta Quest 2 with root access, ADB, and direct access to the Android filesystem.

This isn't meant to be a rooting guide. It's mainly a record of things I've found, tested, and investigated while digging around Horizon OS.

## Device

**Device:** Meta Quest 2
**Architecture:** `aarch64`
**Kernel:** `4.19.325-cip128-st12-g4b63ca9fd613`
**Shell:** Android Toybox

The device exposes a fairly standard Android/Linux environment underneath Horizon OS, although a lot of the interesting parts are Meta-specific.

---

## Root Access

I've been experimenting with temporary root access and tools such as:

* Singularity
* Root Terminal
* FreeXR
* Frida-Server
* ADB shell

One important thing I've found is that temporary root access is not the same thing as having a permanently unlocked bootloader. The Quest can be explored quite deeply from a rooted shell without necessarily having a traditional unlocked Android bootloader.

---

## ADB and Shell

ADB gives access to the underlying Android shell, where a lot of the system can be inspected directly.

The shell reports:

```text
Linux localhost 4.19.325-cip128-st12-g4b63ca9fd613
#1 SMP PREEMPT
aarch64
Toybox
```

From there I've been looking through `/system`, `/system_ext`, `/data`, `/proc`, `/sys` and the Android properties exposed through `getprop`.

---

## Horizon OS Home

The Android HOME activity resolves to Meta's VrShell:

```text
com.oculus.vrshell.HomeActivity
```

The package is:

```text
com.oculus.vrshell
```

And its process is:

```text
com.oculus.vrshell
```

The APK is located at:

```text
/system_ext/priv-app/VrShell/VrShell.apk
```

The application uses:

```text
com.oculus.vrshell.ShellApplication
```

This makes VrShell one of the more interesting parts of the system to investigate.

---

## Boot Partitions

The Quest exposes several Android boot-related partitions through:

```text
/dev/block/by-name/
```

Some of the partitions found include:

```text
boot_a
boot_b
vbmeta_vendor_a
vbmeta_vendor_b
```

I also investigated `vendor_boot`, although the expected:

```text
vendor_boot_a
```

was not present on the device.

This is still being investigated.

---

## Boot Images

I've been looking into the Quest's boot images and trying to determine how Meta's boot chain is structured.

I've also experimented with extracting and inspecting ABL images. One ABL image investigated was around:

```text
2,097,152 bytes
```

Android Image Kitchen wasn't able to directly unpack it, so its structure appears to differ from a normal Android boot image.

---

## Boot Animation

The Quest's boot animation resources can be found under:

```text
/system_ext/etc/bootanim/
```

Files found there include:

```text
meta.bin
meta-horizonos.glb
meta-colors.png
meta-animation-config.json
```

The presence of a `.glb` file is particularly interesting because part of the boot animation appears to use a 3D asset rather than just a collection of normal 2D frames.

---

## LEDs

The Quest exposes LED devices through:

```text
/sys/class/leds/
```

The available LED entries include:

```text
blue
green
red
```

These can be inspected directly from the rooted shell and appear to provide access to the headset's status LED system.

---

## Battery and Power

Battery information is exposed through:

```text
/sys/class/power_supply/
```

This includes the battery capacity and charging state.

For example:

```text
/sys/class/power_supply/battery/capacity
/sys/class/power_supply/battery/status
```

This also makes it possible to pull basic battery information without relying on the normal Horizon OS UI.

---

## System Information

Android properties expose a lot of useful information through:

```text
getprop
```

Some of the properties I've been using include:

```text
ro.product.model
ro.product.device
ro.build.version.release
ro.build.display.id
ro.boot.hardware.revision
ro.soc.model
```

I'm using these for a small utility called `qfetch`, which prints Quest system information directly from the shell.

---

## QuestFetch

`qfetch` is a lightweight system information tool made specifically for the Quest.

It currently gathers things such as:

```text
Device
Codename
Hardware
Android
Build
Kernel
Architecture
CPU
GPU
User
Home
Home Process
Battery
Status
```

The Home Process is obtained by resolving Android's HOME activity and extracting its `processName`.

For the stock Quest environment, this resolves to:

```text
com.oculus.vrshell
```

---

## Things Still Being Investigated

There are still a lot of unanswered questions around the Quest's boot chain and system layout.

Some of the areas I'm continuing to look at:

* ABL structure
* Boot and recovery partitions
* `vbmeta` and verified boot
* VrShell internals
* Horizon OS boot animation
* Kernel configuration
* GPU drivers
* Qualcomm hardware interfaces
* System services
* Root persistence
* Early boot behaviour
* Android framework modifications

This repository is basically a log of what I find along the way rather than a finished explanation of how the Quest works.

## Disclaimer

This research is being done on my own device.

Nothing here should be treated as a guaranteed rooting method or a complete guide. Some findings may be incomplete, device-specific, or change between Horizon OS versions.
