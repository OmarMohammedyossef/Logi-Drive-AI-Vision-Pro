	
# Setting Up Static IP on Ubuntu Wi-Fi (Using Netplan)

## Table of Contents

- [What-Is-Netplan](#what-is-netplan)
- [Static-IP-Versus-DHCP](#static-ip-versus-dhcp)
- [Configuring-Static-IP-for-Wi-Fi](#configuring-static-ip-for-wi-fi)
- [Applying-Your-Changes](#applying-your-changes)
- [Troubleshooting-Common-Issues](#troubleshooting-common-issues)
- [Understanding-Renderer](#understanding-renderer)
    

---

## What-Is-Netplan

In Ubuntu (and other modern Linux distros), **Netplan** is the tool used to configure your network settings. It replaced older tools like `ifupdown` in Ubuntu 17.10 and beyond. Netplan works by reading configuration files in **YAML** format and applying those settings to your system’s network manager.

When it comes to configuring a **static IP address** (as opposed to using DHCP), Netplan gives you a simple and effective way to define your IP settings.

---

## Static-IP-Versus-DHCP

You have two options when configuring the IP address on your Ubuntu system, and that is either a **static IP address** or **DHCP**. Here's the difference:

- **Static IP address**: This means you manually set your IP address. It's like picking an address for your house and saying, "This is where I live, and I won't move unless I choose to." This can be useful for things like servers or if you need consistent access to a particular device.
    
- **DHCP**: Stands for Dynamic Host Configuration Protocol. This means your device asks the router for an IP address, and the router gives you one. It’s a more automatic process, and your device could get a new IP each time it connects, depending on how the router is set up.
    

For most users, **DHCP** is enough. But if you want **stability** and need your machine to always be reachable at the same IP (for things like network servers), you’d go with a **static IP**.

---

## Configuring-Static-IP-for-Wi-Fi

Here’s the fun part – configuring your **static IP** for a Wi-Fi connection on Ubuntu! You’re going to edit a YAML file to set things like the **IP address**, **gateway**, and **DNS servers**.

Here’s a sample configuration:

```yaml
network:
  version: 2
  renderer: NetworkManager
  wifis:
    wlo1:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      routes:
        - to: 0.0.0.0/0
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
      access-points:
        "Your-WiFi-SSID":
          password: "your_wifi_password"
```

### Let’s break this down:

- **`dhcp4: no`**: This tells Ubuntu **not** to use DHCP (because we want to use a static IP).
    
- **`addresses: 192.168.1.100/24`**: This is your **static IP** (`192.168.1.100`). You can change this to whatever IP you need (just make sure it’s in the same network range as your router).
    
- **`routes`**: The `0.0.0.0/0` is basically a shortcut for saying, "Any traffic that’s not local should go through this gateway", which is usually your router’s IP (`192.168.1.1`).
    
- **`nameservers`**: These are DNS servers, which translate website names into IP addresses. We’re using Google's DNS (8.8.8.8) and Cloudflare’s DNS (1.1.1.1), but you can choose any you like.
    

Replace `"Your-WiFi-SSID"` with your Wi-Fi network’s name and `"your_wifi_password"` with the actual Wi-Fi password.

---

## Applying-Your-Changes

Once you’ve edited and saved your YAML file, it’s time to **apply** the changes. Here’s what to do:

1. **Check your configuration** (just in case you made a typo):
    
    ```bash
    sudo netplan try
    ```
    
    This will apply the changes temporarily. If something goes wrong, it will automatically revert in 120 seconds to prevent locking you out.
    
2. **Apply permanently**:
    
    ```bash
    sudo netplan apply
    ```
    

After running this, your network settings should be updated, and your system should have a **static IP**.

---

## Troubleshooting-Common-Issues

Here are a few common issues you might run into while configuring your static IP:

1. **Permissions Too Open Warning**: If you get a warning like:
    
    ```
    Permissions for /etc/netplan/*.yaml are too open.
    ```
    
    This just means your YAML files are too permissive. Netplan wants these files to be secure. Fix it with:
    
    ```bash
    sudo chmod 600 /etc/netplan/*.yaml
    ```
    
    
2. **Wi-Fi Not Connecting**: If your Wi-Fi isn’t connecting after applying your changes, you might have conflicting configurations from NetworkManager or other network tools. You can delete any old Wi-Fi profiles using:
    
    ```bash
    nmcli connection delete <connection-name>
    ```
    
3. **Forgotten Wi-Fi Password**: If you forgot your Wi-Fi password, just make sure to **double-check** that you’ve typed the correct one in the `access-points` section of the YAML file.
    

---

## Understanding-Renderer

In Netplan, the **`renderer`** specifies which tool will handle your network configuration. There are generally two options for this:

- **NetworkManager**: This is the default option for managing network connections, especially when you're using **Wi-Fi** or **laptops** that move between networks. NetworkManager provides an easy-to-use interface for handling Wi-Fi connections, VPNs, and other network services.
    
- **Networkd**: This is a more basic renderer, often used in **server environments** where you don’t need complex Wi-Fi management. It’s simpler and typically handles **Ethernet connections** more effectively than NetworkManager.
    

In the configuration example above, **`renderer: NetworkManager`** tells Netplan to let **NetworkManager** handle the Wi-Fi configuration.

---

## Final Thoughts

Setting a static IP on Ubuntu using **Netplan** is simple once you get the hang of it. It’s a great way to ensure your device always has the same IP address, which is especially useful for things like servers, printers, or if you need a consistent setup for accessing your machine remotely.

But remember, **DHCP** works just fine for most casual users. So, only go static if you really need it.

If you have any issues or run into problems, don’t hesitate to check the **common issues** section, or feel free to reach out! Happy networking! 🌐🚀

---

https://linuxconfig.org/how-to-configure-static-ip-address-on-ubuntu-18-04-bionic-beaver-linux