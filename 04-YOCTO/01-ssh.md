
Adding ssh layer 
https://wiki.yoctoproject.org/wiki/Cookbook:Example:Adding_packages_to_your_OS_image

Great question! Let's break this down to understand why you can't just add the `openembedded-core` layer to `bblayers.conf` and expect SSH to be included in your image.

---

### **1. Layers vs. Features**
- **Layers (`bblayers.conf`)**:
  - Layers provide metadata (recipes, classes, configurations, etc.) that define how software components are built and packaged.
  - The `openembedded-core` layer (often referred to as `meta`) is a **core layer** that contains essential recipes and configurations, including the `openssh` recipe.
  - Adding a layer to `bblayers.conf` makes its recipes and configurations available to the build system, but it does **not automatically include** those recipes in your final image.

- **Features (`local.conf`)**:
  - Features are specific functionalities or packages that you explicitly request to be included in your image.
  - The `EXTRA_IMAGE_FEATURES` variable in `local.conf` is used to specify additional features or packages to include in the image, such as `ssh-server-openssh`.

---

### **2. Why Adding `openembedded-core` to `bblayers.conf` Isn't Enough**
- The `openembedded-core` layer is **already included by default** in most Yocto builds. You don't need to explicitly add it to `bblayers.conf` unless you've removed it for some reason.
- Even if the `openssh` recipe is available in the `openembedded-core` layer, it won't be included in your image unless you explicitly tell the build system to do so.
- Yocto follows a **modular and customizable approach**: it doesn't assume you want every available package in your image. You must explicitly specify what you need.

---

### **3. Why `EXTRA_IMAGE_FEATURES` is Necessary**
- The `EXTRA_IMAGE_FEATURES` variable is a convenient way to add common features (like SSH) to your image without manually editing the image recipe.
- When you add `ssh-server-openssh` to `EXTRA_IMAGE_FEATURES`, the build system knows to include the `openssh` package and configure it as an SSH server in your image.
- This approach is more flexible and maintainable than hardcoding package dependencies into your image recipe.

---

### **4. What Happens If You Only Add `openembedded-core` to `bblayers.conf`?**
- If you only add `openembedded-core` to `bblayers.conf` (or if it's already included by default), the `openssh` recipe will be available in the build system.
- However, the `openssh` package will **not be included in your image** unless:
  - You explicitly add it to `IMAGE_INSTALL` in your image recipe or `local.conf`.
  - Or you use `EXTRA_IMAGE_FEATURES` to include `ssh-server-openssh`.

---

### **5. Example Workflow**
1. **Ensure `openembedded-core` is in `bblayers.conf`** (it usually is by default):
   ```bash
   BBLAYERS = " \
     /path/to/yocto/meta \
     /path/to/yocto/meta-poky \
     /path/to/yocto/meta-yocto-bsp \
   "
   ```

2. **Add `ssh-server-openssh` to `EXTRA_IMAGE_FEATURES` in `local.conf`**:
   ```bash
   EXTRA_IMAGE_FEATURES = "ssh-server-openssh"
   ```

3. **Build your image**:
   - The build system will include the `openssh` package and configure it as an SSH server in your image.

---

### **6. Alternative: Manually Add `openssh` to `IMAGE_INSTALL`**
If you don't want to use `EXTRA_IMAGE_FEATURES`, you can manually add `openssh` to the `IMAGE_INSTALL` variable in your image recipe or `local.conf`:
```bash
IMAGE_INSTALL:append = " openssh"
```

---

### **Summary**
- Adding `openembedded-core` to `bblayers.conf` makes the `openssh` recipe available, but it doesn't automatically include it in your image.
- You must explicitly request the `openssh` package to be included in your image, either by:
  - Using `EXTRA_IMAGE_FEATURES = "ssh-server-openssh"` in `local.conf`.
  - Or adding `openssh` to `IMAGE_INSTALL`.

This design ensures that Yocto builds are lean and customizable, as only the explicitly requested packages are included in the final image.
