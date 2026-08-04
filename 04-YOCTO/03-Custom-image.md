steps
1) clone rarpberrypi meta layer from openembedded
	- `git clone -b kirkistone git://git.yoctoproject.org/meta-raspberrypi`
2) soure oe...

3) Add a custom layer
	- `bitbake-layers create-layer ../poky/meta-refat`
	- `bitbake-layers add-layer ../poky/meta-refat` 
		- or you can add it manually in the `bblayer.con`
4) Adding necessary layer and recipes
**`bblayer.conf`**
```
# POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf
# changes incompatibly
POKY_BBLAYERS_CONF_VERSION = "2"

BBPATH = "${TOPDIR}"
BBFILES ?= ""

BBLAYERS ?= " \
  /home/mohamed/yocto2024/kirkstone/poky/meta \
  /home/mohamed/yocto2024/kirkstone/poky/meta-openembedded/meta-oe \
  /home/mohamed/yocto2024/kirkstone/poky/meta-openembedded/meta-python \
  /home/mohamed/yocto2024/kirkstone/poky/meta-openembedded/meta-networking \
  /home/mohamed/yocto2024/kirkstone/poky/meta-qt5 \
  /home/mohamed/yocto2024/kirkstone/poky/meta-raspberrypi \
  /home/mohamed/yocto2024/kirkstone/poky/meta-refat \
  "
```

**local.conf**
```
MACHINE ??= "raspberrypi4-64"

#LICENSE_FLAGS_ACCEPTED += "commercial"


# Setting for CAN 2-CH FD
KERNEL_DEVICETREE:append = " \
                        overlays/mcp251xfd.dtbo \
"

# Setting for i2c
ENABLE_I2C = "1"
KERNEL_MODULE_AUTOLOAD:rpi += "i2c-dev i2c-bcm2708"


# Set systemd instead sysV
DISTRO_FEATURES:append = " systemd"
DISTRO_FEATURES:remove = "sysvinit"
VIRTUAL-RUNTIME_init_manager = "systemd"
VIRTUAL-RUNTIME_initscripts = "systemd-compat-units"
DISTRO_FEATURES_BACKFIL_CONSIDERED = "sysvinit"
VIRTUAL-RUNTIME_initscript = "systemd-compat-units"

# Set wayland for Qt
DISTRO_FEATURES:remove = "x11 vulkan"
DISTRO_FEATURES:append = " wayland"
IMAGE_INSTALL:append   =  "qtbase qtwayland"
CORE_IMAGE_EXTRA_INSTALL = "wayland weston"
```


