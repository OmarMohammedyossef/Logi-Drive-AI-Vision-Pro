
---

# Qt D-Bus System Service Example

## Table of Contents
  
- [dbus-policy-configuration-xml](#dbus-policy-configuration-xml)
- [creating-a-qt-d-bus-system-service](#creating-a-qt-d-bus-system-service)
- [testing-the-service-from-terminal](#testing-the-service-from-terminal)


---
## D-Bus-Policy-Configuration-XML

When using the **system bus**, you **must** provide a D-Bus **policy configuration file** (`.conf`) and place it in:


```bash
/etc/dbus-1/system.d/
```


This file **controls access** to your D-Bus service—who can own the service, send messages to it, and call its interfaces. Without it, even if your service runs successfully, **other processes won't be allowed** to communicate with it.

---

### Why It’s Required

The **system bus** is shared across the entire system and must be secure. D-Bus **does not allow unconfigured services** to expose interfaces on the system bus, to avoid security risks.


Create a file `/etc/dbus-1/system.d/com.example.MyService.conf`:

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

  <!-- Allow everyone else to send messages to this service -->
  <policy context="default">
    <allow send_destination="com.example.MyService"/>
    <allow send_interface="com.example.MyService.Interface"/>
  </policy>

</busconfig>
```

Then reload the D-Bus service:

```bash
sudo systemctl reload dbus
```


---

## Creating-a-Qt-D-Bus-System-Service

1. Define your Qt service class with slots and signals:

```cpp
class MyService : public QObject {
    Q_OBJECT
    Q_CLASSINFO("D-Bus Interface", "com.example.MyService.Interface")

public slots:
    QString SayHello(const QString &name);
    int AddIntegers(int num1, int num2);

signals:
    void MessageReceived(const QString &message);
};
```

2. In `main.cpp`, connect to the system bus and register your service:
    

```cpp
QDBusConnection connection = QDBusConnection::systemBus();

if (!connection.registerService("com.example.MyService")) {
    qWarning() << "Failed to register service:" << connection.lastError().message();
    return 1;
}

if (!connection.registerObject("/com/example/MyService", &service,
     QDBusConnection::ExportAllSlots | QDBusConnection::ExportAllSignals)) {
    qWarning() << "Failed to register object:" << connection.lastError().message();
    return 1;
}
```

> Note: You must run this as a privileged user (e.g., `sudo`) when using the system bus.

## Testing-the-Service-from-Terminal

Call the `SayHello` method using `dbus-send`:

```bash
dbus-send --system \
  --dest=com.example.MyService \
  --type=method_call \
  --print-reply \
  /com/example/MyService \
  com.example.MyService.Interface.SayHello \
  string:"Qt User"
```

### 1. **Check that the service is registered**


```bash
busctl --system list | grep com.example.MyService
```

### 2. **Explore the object tree**
```bash
busctl --system tree com.example.MyService
```

### 3. **Introspect the object**

```bash
busctl --system introspect com.example.MyService /com/example/MyService
```

### 4. **Call a method**

```bash
busctl --system call com.example.MyService /com/example/MyService com.example.MyService SayHello s "Mohamed"
```

> **Note:**
> `busctl` is a command-line tool used for interacting with the D-Bus system bus. It provides an interface to manage services, objects, and interfaces, as well as query or send messages over D-Bus. It's a powerful utility for debugging and testing D-Bus services.

---

###  Tips

- If you see an error like `UnknownInterface`, double-check that:
    
    - Your class has the correct `Q_CLASSINFO`.
        
    - The interface name in the `.conf` and code matches.
        
- Always test on the **system bus** using `--system`.

---
