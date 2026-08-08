# Meta Quest Research

A collection of research, discoveries, and notes from exploring the Meta Quest 2 and Horizon OS with root and ADB access.

This repository documents things found while looking through the Quest's Android userspace, boot chain, partitions, system applications, kernel, hardware interfaces, and Horizon OS components.

Most of this is experimental research rather than a guide.

---

# Device

The Quest 2 runs an Android-based operating system with a heavily modified userspace provided by Meta.

Current observations from a rooted Quest 2:

```text
Architecture: aarch64
Kernel:       4.19.325-cip128-st12-g4b63ca9fd613
Shell:        Toybox
```

The underlying Android environment exposes the usual `/proc`, `/sys`, `/data`, `/system` and `/system_ext` filesystems, alongside a number of Meta-specific services and components.

---

# Root and ADB

Root access provides considerably more visibility into the headset than the normal Android shell.

Things investigated so far include:

* Android system properties
* Kernel information
* Block devices and partitions
* Power supply interfaces
* LED interfaces
* Horizon OS applications
* VrShell
* Boot animation resources
* Boot images
* ABL
* Fastboot
* System applications
* Android framework components
* GPU interfaces
* Qualcomm hardware interfaces

The rooted shell is running Android's Toybox utilities rather than a normal GNU userspace.

---

# Horizon OS Home

The Android HOME activity resolves to Meta's VrShell application.

Running:

```text
cmd package resolve-activity -c android.intent.category.HOME -a android.intent.action.MAIN
```

returns:

```text
Activity:
com.oculus.vrshell.HomeActivity

Package:
com.oculus.vrshell

Process:
com.oculus.vrshell
```

The application is located at:

```text
/system_ext/priv-app/VrShell/VrShell.apk
```

Its application class is:

```text
com.oculus.vrshell.ShellApplication
```

The package runs as UID `10048`.

This makes VrShell one of the most important parts of the Horizon OS userspace and one of the main areas worth investigating.

---

# System Applications

A large portion of the Quest's functionality is provided by Android system and privileged applications.

Applications can be found under locations such as:

```text
/system/app/
/system/priv-app/
/system_ext/app/
/system_ext/priv-app/
```

The VrShell package is an example of a privileged system application:

```text
/system_ext/priv-app/VrShell/VrShell.apk
```

Some system applications use Android's optimized bytecode format, meaning the APK may not contain all of the original executable bytecode directly.

This makes deodexing and reverse engineering necessary when investigating older Quest system applications.

---

# Boot Chain

The Quest uses a Qualcomm-based Android boot chain with multiple bootloader components and A/B partitions.

Known components include:

```text
XBL
CDT
DDR
RPM
TZ
HYP
PMIC
MODEM
ABL
KEYMASTER
BOOT
CMNLIB
CMNLIB64
DEVCFG
```

Many of these exist as both `_a` and `_b` slots.

The block-device names can be inspected through:

```text
/dev/block/by-name/
```

On the Quest 2, examples include:

```text
boot_a
boot_b
abl_a
abl_b
vbmeta_vendor_a
vbmeta_vendor_b
```

Not every partition described by older research is necessarily present under the same name on newer firmware.

---

# ABL

ABL is responsible for the Quest's fastboot environment and a number of Meta-specific bootloader operations.

Older Quest research identified several ABL versions, including:

```text
213561.4150.0
256550.6810.0
333700.2680.0-396520.6170.115
```

ABL contains Oculus-specific modifications to the normal Android fastboot implementation.

Known OEM commands include:

```text
oem device-info
oem reboot-edl
oem reboot-sideload
oem shutdown
oem sha1
oem partition-info
oem set-verity
oem set-verified-boot
oem get-kernel-flavor
oem update-all-slots
oem off-mode-charge
oem enable-charger-screen
oem disable-charger-screen
oem set-retail-keymaster
oem read-persist
oem write-persist
oem set-serial-number
oem set-retail-device
```

The exact commands and restrictions depend on the ABL version and device state.

---

# EDL and Fastboot

The Quest has a Qualcomm Emergency Download mode in addition to its Android fastboot environment.

EDL can be entered through the hardware button combination on supported firmware, while fastboot can be entered using the bootloader controls or ADB when available.

The bootloader exposes additional commands for moving between these modes, including:

```text
oem reboot-edl
oem reboot-sideload
```

---

# Partition Layout

The Quest uses multiple logical and physical partitions.

Older Quest research identified partitions including:

```text
system_a
system_b
private
vision
userdata

xbl_a
xbl_b
cdt
ddr
rpm_a
rpm_b
tz_a
tz_b
hyp_a
hyp_b
pmic_a
pmic_b
modem_a
modem_b
bluetooth_a
bluetooth_b
abl_a
abl_b
keymaster_a
keymaster_b
boot_a
boot_b
cmnlib_a
cmnlib_b
cmnlib64_a
cmnlib64_b
devcfg_a
devcfg_b
```

Additional partitions include areas used for:

```text
persist
misc
keystore
frp
devinfo
dip
apdp
msadp
splash
logfs
logdump
storsec
modemst1
modemst2
fsg
fsc
```

Partition layouts can change between firmware versions, so older layouts should not be assumed to exactly match newer Horizon OS releases.

Partition information can be queried from the bootloader using:

```text
oem partition-info
```

---

# Boot Images

The boot partitions have also been investigated directly through:

```text
/dev/block/by-name/boot_a
/dev/block/by-name/boot_b
```

ABL images have been extracted for further analysis.

One ABL image investigated was:

```text
2,097,152 bytes
```

Standard Android Image Kitchen tooling did not directly unpack the image, suggesting that the Quest's ABL packaging differs from a normal Android boot image.

Further reverse engineering is still needed here.

---

# Verified Boot

The bootloader contains controls for Android verified boot and dm-verity.

Known commands include:

```text
oem set-verity
oem set-verified-boot
```

The ability to use these commands depends on the state of the device and the particular ABL version.

The Quest therefore combines standard Android verified boot mechanisms with Meta-specific bootloader restrictions.

---

# Unlocking

Older Quest research found that legitimate bootloader unlocking involves an `unlock_token` partition.

The token contains a bootloader script and a signature.

The signature contains information identifying the target device, including its serial number.

The signature uses RSA-PSS with SHA-256.

The bootloader verifies the token before accepting the unlock operation.

The exact unlocking behaviour is firmware-dependent.

---

# OTA Updates

Quest firmware is distributed through OTA packages.

Factory firmware and OTA packages can be extracted using Android OTA extraction tools.

Incremental OTA packages require additional handling because of their structure.

Firmware research is useful for comparing changes between Horizon OS versions, especially changes to:

* ABL
* Kernel
* VrShell
* System applications
* Boot images
* Partition layouts
* Security mechanisms

---

# Boot Animation

Horizon OS contains Meta-specific boot animation resources.

On the Quest 2, these were found under:

```text
/system_ext/etc/bootanim/
```

Files discovered include:

```text
meta.bin
meta-horizonos.glb
meta-colors.png
meta-animation-config.json
```

The presence of:

```text
meta-horizonos.glb
```

suggests that part of the boot animation uses a 3D GLB asset.

The configuration and supporting resources are also stored directly on the system partition.

---

# LEDs

The headset exposes its status LEDs through:

```text
/sys/class/leds/
```

The Quest 2 exposes:

```text
blue
green
red
```

These can be inspected and controlled through the Linux LED subsystem when sufficient permissions are available.

This provides a simple example of how hardware functionality is exposed to the Android userspace.

---

# Battery and Power

Battery information is exposed through Android's Linux power-supply subsystem:

```text
/sys/class/power_supply/
```

The battery interface includes information such as:

```text
capacity
status
```

For example:

```text
/sys/class/power_supply/battery/capacity
/sys/class/power_supply/battery/status
```

This allows battery information to be read directly without using the Horizon OS interface.

---

# GPU

The Quest's GPU is exposed through the Qualcomm KGSL driver.

One of the interfaces investigated is:

```text
/sys/class/kgsl/kgsl-3d0/
```

This provides access to information exposed by the Qualcomm GPU driver.

The GPU information can also be discovered through Android system properties depending on the firmware.

---

# CPU and SoC

CPU and SoC information can be obtained through a combination of:

```text
/proc/cpuinfo
```

and Android properties such as:

```text
ro.soc.model
ro.hardware
```

The Quest 2 uses a Qualcomm Snapdragon platform and exposes the underlying hardware through the normal Linux and Android hardware interfaces.

---

# QuestFetch

`qfetch` is a small system information utility written specifically for the Quest.

It is designed to work directly from the rooted Android shell without requiring a large dependency chain.

Current information includes:

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

The HOME process is detected dynamically by resolving Android's HOME activity:

```text
cmd package resolve-activity \
    -c android.intent.category.HOME \
    -a android.intent.action.MAIN
```

The current stock Quest environment resolves this to:

```text
com.oculus.vrshell
```

---

# Companion System

The Quest has a companion system used by the Meta/Oculus mobile application.

The phone communicates with a server running on the headset.

Older Quest research identified this component as:

```text
CompanionServer.apk
```

The companion system communicates over Bluetooth Low Energy.

---

# Bluetooth Low Energy

The Quest exposes a custom BLE GATT service for companion communication.

The service identified by earlier research is:

```text
Companion
UUID: 0000FEB8-0000-1000-8000-00805F9B34FB
```

Two known characteristics are:

```text
ccs
7a442881-509c-47fa-ac02-b06a37d9eb76

status
7a442666-509c-47fa-ac02-b06a37d9eb76
```

The characteristics use the standard BLE Client Characteristic Configuration descriptor:

```text
00002902-0000-1000-8000-00805F9B34FB
```

---

# Companion Protocol

The companion protocol is layered.

The transport layer splits larger messages into BLE-sized chunks.

The chunks contain a two-byte header with a sequence number, with the high bit used to indicate the final chunk.

The application protocol uses Protocol Buffers.

Messages are exchanged as `Request` and `Response` structures.

---

# Companion Authentication

The companion connection uses public-key cryptography to establish a shared secret.

During the connection handshake:

1. The Quest generates a key pair.
2. The client generates a key pair.
3. Public keys are exchanged.
4. A shared secret is derived.
5. Later communication is encrypted.

Older research found the cryptographic implementation inside:

```text
libauthentication.so
```

which wraps `libsodium`.

---

# Companion Commands

Research into the companion protocol identified a large number of commands, including:

```text
ADB_MODE_SET
ADB_MODE_STATUS

APP_LAUNCH

AUTHENTICATE
HELLO

DEV_MODE_SET
DEV_MODE_STATUS

HMD_STATUS
HMD_VERSION
HMD_CAPABILITIES

CONTROLLER_PAIR
CONTROLLER_SCAN
CONTROLLER_SCAN_AND_PAIR
CONTROLLER_STATUS
CONTROLLER_UNPAIR

WIFI_CONNECT
WIFI_DISABLE
WIFI_ENABLE
WIFI_FORGET
WIFI_RECONNECT
WIFI_SCAN
WIFI_STATUS

MTP_MODE_SET
MTP_MODE_STATUS

OTA_ENABLED_SET
OTA_ENABLED_STATUS

PIN_LOCK
PIN_RESET
PIN_SET
PIN_STATUS
PIN_UNLOCK
PIN_VERIFY

SYSTEM_UNLOCK

TEXT_SEND

TIME_SET

WIPE_DATA
```

There are also commands for settings, accounts, controllers, autosleep, locale, managed mode, crash reporting, mirroring and other headset functionality.

---

# Reverse Engineering

System applications can be investigated by extracting their optimized Android bytecode and converting it into a form suitable for analysis.

Older Quest research used tools including:

```text
baksmali
smali
dex2jar
Bytecode Viewer
```

A typical workflow is:

```text
.odex
  |
  v
baksmali
  |
  v
.smali
  |
  v
smali
  |
  v
.dex
  |
  v
dex2jar
  |
  v
.jar
```

The resulting JAR can then be inspected using a Java decompiler.

---

# Kernel

The Quest uses a modified Qualcomm Linux kernel.

Older Quest firmware was affected by CVE-2018-9568.

A kernel change was eventually introduced to address the vulnerability.

The Quest kernel is particularly interesting because it provides the interface between Horizon OS and the underlying Qualcomm hardware.

Current research includes looking at:

* Kernel configuration
* Drivers
* KGSL
* Power management
* Hardware interfaces
* Android framework interaction
* Meta-specific changes

---

# Filesystem

Some of the more useful locations discovered during investigation include:

```text
/system
/system_ext
/data
/proc
/sys
/dev
/dev/block/by-name
```

Meta-specific components are particularly concentrated in:

```text
/system_ext/
```

This includes VrShell and several other Horizon OS components.

---

# Current Research

Areas still being investigated include:

* ABL internals
* Boot image formats
* Verified boot
* `vbmeta`
* Horizon OS internals
* VrShell
* CompanionServer
* Qualcomm hardware interfaces
* KGSL
* Kernel configuration
* System services
* Root persistence
* Boot-time execution
* Boot animation
* Partition differences between firmware versions
* Meta-specific Android modifications
* Fastboot behaviour
* EDL behaviour
* OTA changes

This repository is intended to grow as more of the Quest's internals are documented.

---

# Notes

A lot of Quest behaviour changes between firmware versions.

Something that exists on one version may be renamed, moved, removed, or protected on another.

Older research from the QuestEscape project is useful for understanding the earlier Quest boot chain and software architecture, but it should not automatically be assumed to apply to current Quest 2 firmware.

The goal here is to document what can actually be observed and verified on the device rather than making assumptions about how Horizon OS works.

Credits to the repo https://github.com/QuestEscape for extra info about the bootloader and check out his github
