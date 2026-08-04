# Qt6 Integration

## Steps to Build/Run a Qt App

From [OpenEmbedded](https://layers.openembedded.org/layerindex/branch/master/layers/):

1. Choose the `kirkistone` branch and search for `qt6`, or [click here](https://layers.openembedded.org/layerindex/branch/master/layer/meta-qt6/).

2. Clone the qt6 layer:

    ```bash
    git clone git://code.qt.io/yocto/meta-qt6.git
    ```

3. You can see that qt6 has some dependencies:

    - openembedded-core
    - meta-python
    - meta-oe

4. Add the required layers in `bblayer.conf`, in `build/conf`:

    ```
    # POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf
    # changes incompatibly
    POKY_BBLAYERS_CONF_VERSION = "2"

    BBPATH = "${TOPDIR}"
    BBFILES ?= ""

    BBLAYERS ?= " \
      /home/mohamed/yocto2024/kirkstone/poky/meta \
      /home/mohamed/yocto2024/kirkstone/poky/meta-poky \
      /home/mohamed/yocto2024/kirkstone/poky/meta-openembedded/meta-oe \
      /home/mohamed/yocto2024/kirkstone/poky/meta-openembedded/meta-python \
      /home/mohamed/yocto2024/kirkstone/poky/meta-qt6 \
      /home/mohamed/yocto2024/kirkstone/poky/meta-yocto-bsp \
      "
    ```

5. Add the required recipes in `local.conf`:

    ```
    CORE_IMAGE_EXTRA_INSTALL += "python3"
    IMAGE_INSTALL:append = " qtbase qtdeclarative qtbase-tools qtbase-plugins"
    IMAGE_INSTALL:append = " mesa"
    PREFERRED_PROVIDER_virtual/libgl = "mesa"
    ```

6. Bitbake the image:

    ```bash
    bitbake core-image-minimal
    ```

## References

- [Learn-Yocto: Recipe-qt5.md](https://github.com/joaocfernandes/Learn-Yocto/blob/master/develop/Recipe-qt5.md)
- [Video walkthrough](https://www.youtube.com/watch?v=2HyUCWOQhr8)
