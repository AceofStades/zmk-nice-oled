# Nice Peripheral View

This is a zmk module version of the nice!view alternative widgets with peripheral status by Tobias Arndt.

You can find the [original version in his zmk-config](https://github.com/tobiasarndt/zmk-config).

## Custom Display

It includes custom display code for a screen, which is displayed in sleep and rotated widgets and small pictogramms to display whether the peripheral half is connected.
<p align="middle">
<img src="https://github.com/tobiasarndt/zmk-config/assets/54204861/53c7cc19-ce8d-4848-8c77-9eeaa39a5d66" height="400"/>
<img src="https://github.com/tobiasarndt/zmk-config/assets/54204861/5a872f72-4adb-41ab-b733-b1440e55ae4e" height="400"/>
<img src="https://github.com/tobiasarndt/zmk-config/assets/54204861/2ab47ef1-7d45-4489-bb01-68d5bcdb517f" height="400"/>
</p>

## Setup Instructions

This module is a drop-in replacement for the nice!view widgets. Make sure to get the regular nice!view widgets working first.

Please refer to the [nice!view documentation](https://nicekeyboards.com/docs/nice-view/getting-started), the [nice_view shield readme](https://github.com/zmkfirmware/zmk/tree/4dfc45d4ab38c23cae61dae4699bd0fbef4e33eb/app/boards/shields/nice_view), and the [nice_view_adapter shield](https://github.com/zmkfirmware/zmk/tree/4dfc45d4ab38c23cae61dae4699bd0fbef4e33eb/app/boards/shields/nice_view_adapter).

Once your keyboard shows the nice!view widgets, you can switch to these widgets by making the following changes...

### If you are building using GitHub Actions

First adjust your `zmk-config/config/west.yml` to add the module and switch zmk to a fork that has [PR #2036](https://github.com/zmkfirmware/zmk/pull/2036).

```diff
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
+   - name: ReFil
+     url-base: https://github.com/ReFil
+   - name: tobiasarndt
+     url-base: https://github.com/tobiasarndt
+   - name: infused-kim
+     url-base: https://github.com/infused-kim

  projects:
    - name: zmk
-      remote: zmkfirmware
-      revision: main
+      remote: ReFil
+      revision: split-data-xfer
      import: app/west.yml
+
+   # zmk module for nice_peri_view
+   - name: tmp-npv
+     remote: infused-kim
+     revision: main

  self:
    path: config
```

Next, adjust your `zmk-config/build.yaml`. Make sure to add the `nice_peri_view` shield only do your central side (usually the left).

```diff
include:
  - board: nice_nano_v2
-   shield: corne_left nice_view
+   shield: corne_left nice_peri_view

  - board: nice_nano_v2
    shield: corne_right nice_view
```

If you get an error like...

```bash
                 from /tmp/zmk-config/zmk/app/src/display/main.c:7:
/tmp/zmk-config/zephyr/include/zephyr/device.h:85:41: error: '__device_dts_ord_DT_CHOSEN_zephyr_display_ORD' undeclared here (not in a function)
```

Then either make sure nice_view is enabled on your peripheral side or only enable `CONFIG_ZMK_DISPLAY=y` on the central side where the `nice_peri_view` shield is included.

You can use `zmk-config/config/your_keyboard_left.conf` and `zmk-config/config/your_keyboard_right.conf` to apply different settings to different sides.

Alternatively, you can also just remove `CONFIG_ZMK_DISPLAY` from your config as it will automatically be enabled when you include the nice_peri_view shield.

### If you are building using the local toolchain (Developers only)


> **Notice**
>
> This sub-section is for developers who are building zmk locally on their computers instead of using GitHub Actions. Most people can ignore this sub-section.

#### Set up the toolchain <!-- omit from toc -->

Follow the [toolchain setup instructions in the zmk documentation](https://zmk.dev/docs/development/setup).

#### Add the zmk-config to the toolchain (docker only) <!-- omit from toc -->

Create a volume with the directory where you will store the zmk-config:

```bash
docker volume create --driver local -o o=bind -o type=none -o device="/full/path/to/your/zmk-config/" zmk-config
```

#### Add this module to the toolchain (docker only) <!-- omit from toc -->

At the time of publishing, the docker VSCode toolchain is not set up to work with modules yet.

First, create a volume with the directory of your zmk-config:

```bash
docker volume create --driver local -o o=bind -o type=none -o device="/full/path/to/your/zmk-modules/" zmk-modules
```

Make sure this module is cloned into that `zmk-modules` directory.

Next, edit in your zmk clone `zmk/.devcontainer/devcontainer.json`:

```diff
{
  "name": "ZMK Development",
  [...]
  "mounts": [
    "type=volume,source=zmk-root-user,target=/root",
    "type=volume,source=zmk-config,target=/workspaces/zmk-config",
+   "type=volume,source=zmk-modules,target=/workspaces/zmk-modules",
    "type=volume,source=zmk-zephyr,target=${containerWorkspaceFolder}/zephyr",
    "type=volume,source=zmk-zephyr-modules,target=${containerWorkspaceFolder}/modules",
    "type=volume,source=zmk-zephyr-tools,target=${containerWorkspaceFolder}/tools"
  ],
  [...]
}
```

Then, in VSCode press `Command + Shift + P` and execute the command `Dev Containers: Rebuild Without Cache and Reopen in Container`.

Once the container is rebuilt, you can delete the above line from your `devcontainer.json` so that it doesn't pollute your git history. The container will continue mounting the `zmk-modules/` volume as long as you don't rebuild it.

#### Build the firmware <!-- omit from toc -->

Run the build command...

```bash
west build -d build/corne/right -p -b nice_nano_v2 -- -DZMK_EXTRA_MODULES="/workspaces/zmk-config/;/workspaces/zmk-modules/nice_peri_view/" -DZMK_CONFIG="/workspaces/zmk-config/config" -DSHIELD="corne_right nice_peri_view"
```

The important thing to note here is that you have to add the `/workspaces/zmk-config/` to your `-DZMK_EXTRA_MODULES` and use `;` as the separator between multiple modules.

And `-DZMK_CONFIG` must include not just the path to the `zmk-config`, but also the sub-directory `zmk-config/config`.

You can [learn more in the zmk dev documentation](https://zmk.dev/docs/development/build-flash#building-with-external-modules).

## Configuration Options

You can also adjust the following config options:

```c
# Rotate the display by 180 degrees
CONFIG_NICE_PERI_VIEW_ROTATE_DISPLAY=n

# If your central side is on the right, enable this option
CONFIG_NICE_PERI_VIEW_RIGHT_SIDE_CENTRAL=n

# Control when the sleep art should be shown
CONFIG_NICE_PERI_VIEW_SHOW_SLEEP_ART_ON_IDLE=y
CONFIG_NICE_PERI_VIEW_SHOW_SLEEP_ART_ON_SLEEP=y
```
