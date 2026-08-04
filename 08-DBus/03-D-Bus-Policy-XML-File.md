

## Table of Contents

- [what-is-the-d-bus-policy-xml-file](#what-is-the-d-bus-policy-xml-file)
    
- [structure-of-the-xml-file](#structure-of-the-xml-file)
    
- [tips](#tips)
    

---

## What-Is-the-D-Bus-Policy-XML-File

When you register a D-Bus service on the **system bus**, D-Bus enforces access control using XML policy files located in:

```
/etc/dbus-1/system.d/
```

These files define:

- **Which users or programs are allowed** to own a service
    
- **Who can send messages** to that service
    
- **Which interfaces are accessible**
    

> Without this file, your service may fail to start, or external apps might not be able to call its methods.

---

## Structure-of-the-XML-File

```xml
<!DOCTYPE busconfig PUBLIC "-//freedesktop//DTD D-Bus Bus Configuration 1.0//EN"
 "http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">
<busconfig>

  <!-- Allow a specific user to own and access the service -->
  <policy user="mohamedrefat">
    <allow own="com.example.MyService"/>
    <allow send_destination="com.example.MyService"/>
    <allow send_interface="com.example.MyService.Interface"/>
  </policy>

  <!-- Allow everyone else to send messages to the service -->
  <policy context="default">
    <allow send_destination="com.example.MyService"/>
    <allow send_interface="com.example.MyService.Interface"/>
  </policy>

</busconfig>
```

### Doctype-busconfig

This line declares the DTD (Document Type Definition) for validation. It tells D-Bus how to interpret this config.

---

### Busconfig-tag

Root element of the policy file. All D-Bus policy rules go inside here.

---

### Policy-user-tag

This policy applies to a **specific user** — in this case, `mohamedrefat`.

- `allow own="com.example.MyService"`  
    → Lets this user register (own) the service name on the bus.
    
- `allow send_destination="com.example.MyService"`  
    → Lets this user send messages to this service.
    
- `allow send_interface="com.example.MyService.Interface"`  
    → Lets this user use this specific D-Bus interface.
    

---

### Policy-context-default-tag

This policy applies to **everyone else** (i.e., all other users or apps).

- Grants permission to send messages to the service and access the interface, but not to own it.
    

> This is useful when only one app (your service) is allowed to register the name, but others can still communicate with it.

---

## Tips

- Use your actual system username in the `user="..."` field.
    
- If your interface name changes, make sure to update the `send_interface` field accordingly.
    
- After editing or adding a new file, **you must reload D-Bus**:
    

```bash
sudo systemctl reload dbus
```

Or reboot if needed.


```
sudo apt install libqt5serialbus5-dev
```