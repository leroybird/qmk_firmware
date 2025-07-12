# Compiling and Flashing QMK Firmware for Keychron K15 Pro (leroy keymap)

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
