# Compiling and Flashing QMK Firmware for Keychron K15 Pro (leroy keymap)

This guide outlines the steps to compile and flash the QMK firmware for the Keychron K15 Pro keyboard with the `leroy` keymap, using `pixi` for dependency management.

## Prerequisites

Ensure you have `pixi` installed on your system.

## 1. Set up the Pixi Environment and Install Dependencies

First, initialize a `pixi` environment and install the necessary Python packages, including the QMK CLI and its dependencies.

```bash
pixi init
pixi add --pypi qmk
pixi add --pypi appdirs argcomplete colorama dotty-dict hid hjson "jsonschema>=4" "milc>=1.4.2" pygments pyserial pyusb pillow
```

Then, run the QMK setup command within the `pixi` environment:

```bash
pixi run qmk setup
```

## 2. Apply Code Modifications (Fixing Compilation Errors)

During compilation, several issues were encountered and resolved with the following code modifications:

### a. Fix Missing `eeconfig_language.h` Include Path

The build system was not finding `eeconfig_language.h`. This was resolved by adding the `language` subdirectory to the `VPATH` in `keyboards/keychron/common/common.mk`.

**File:** `keyboards/keychron/common/common.mk`

**Change:**
```diff
--- a/keyboards/keychron/common/common.mk
+++ b/keyboards/keychron/common/common.mk
@@ -1,5 +1,6 @@
 COMMON_DIR = common
 SRC += $(COMMON_DIR)/matrix.c
 
 VPATH += $(TOP_DIR)/keyboards/keychron/$(COMMON_DIR)
+VPATH += $(TOP_DIR)/keyboards/keychron/common/language
 
 include $(TOP_DIR)/keyboards/keychron/$(COMMON_DIR)/debounce/debounce.mk
```

### b. Fix `EECONFIG_KB_DATA_SIZE` Redefinition

A redefinition error for `EECONFIG_KB_DATA_SIZE` occurred due to a conflict with the core QMK firmware. This was resolved by wrapping the definition in an `#ifndef` guard in `keyboards/keychron/common/eeconfig_kb.h`.

**File:** `keyboards/keychron/common/eeconfig_kb.h`

**Change:**
```diff
--- a/keyboards/keychron/common/eeconfig_kb.h
+++ b/keyboards/keychron/common/eeconfig_kb.h
@@ -57,5 +57,7 @@
 #define EECONFIG_BASE_WIRELESS_CONFIG EECONFIG_END_CUSTOM_RGB
 #define EECONFIG_END_WIRELESS_CONFIG (EECONFIG_BASE_WIRELESS_CONFIG + __EECONFIG_SIZE_WIRELESS_CONFIG)
 
+#ifndef EECONFIG_KB_DATA_SIZE
 #define EECONFIG_KB_DATA_SIZE (EECONFIG_END_WIRELESS_CONFIG - EECONFIG_BASE_LANGUAGE)
+#endif
```

### c. Enable Custom Debounce

A linker error indicated multiple definitions of debounce functions. This was resolved by explicitly telling the build system to use the custom debounce implementation provided by Keychron in `keyboards/keychron/k15_pro/rules.mk`.

**File:** `keyboards/keychron/k15_pro/rules.mk`

**Change:**
```diff
--- a/keyboards/keychron/k15_pro/rules.mk
+++ b/keyboards/keychron/k15_pro/rules.mk
@@ -1,5 +1,7 @@
 # Enter lower-power sleep mode when on the ChibiOS idle thread
 OPT_DEFS += -DCORTEX_ENABLE_WFI_IDLE=TRUE
 OPT_DEFS += -DNO_USB_STARTUP_CHECK -DENABLE_FACTORY_TEST
 
+DEBOUNCE_TYPE = custom
+
 include keyboards/keychron/bluetooth/bluetooth.mk
 include keyboards/keychron/common/common.mk
```

## 3. Compile the Firmware

Once the environment is set up and the code modifications are applied, compile the firmware using the following command:

```bash
pixi run make keychron/k15_pro/ansi_encoder/rgb:leroy
```

This will generate the firmware files (e.g., `.bin`, `.hex`) in the `.build/` directory.

## 4. Flash the Firmware

Before flashing, put your keyboard into **reset/bootloader mode**. For the Keychron K15 Pro:

1.  Connect the USB cable.
2.  Toggle the mode switch to "Off".
3.  Hold down the *Esc* key or the reset button underneath the space bar.
4.  Toggle the mode switch to "Cable".

Once your keyboard is in reset mode, flash the firmware using:

```bash
pixi run make keychron/k15_pro/ansi_encoder/rgb:leroy:flash
```

This will flash the compiled firmware onto your keyboard, and your custom layout should now be active.
