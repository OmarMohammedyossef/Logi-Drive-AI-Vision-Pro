# D-Bus Data Exposure Guide 

## Types of D-Bus Exposures

You can expose the following elements through D-Bus:

1. **Interfaces** (collections of methods, signals and properties)
2. **Methods** (callable functions)
3. **Signals** (event notifications)
4. **Properties** (data values with change notifications)
5. **Objects** (instances containing interfaces)

## Supported Data Types
D-Bus supports these basic types in XML definitions:

| Type    | Description    | Example                  |
| ------- | -------------- | ------------------------ |
| `b`     | Boolean        | `true`                   |
| `i`     | 32-bit integer | `42`                     |
| `x`     | 64-bit integer | `10000000000`            |
| `d`     | Double         | `3.14159`                |
| `s`     | String         | `"hello"`                |
| `o`     | Object path    | `/com/example/Object`    |
| `a{sv}` | Dictionary     | `{"key": variant_value}` |

## How to Expose Data on D-Bus

### 1. As Properties (Recommended for Data)
```cpp
// In your class header:
Q_CLASSINFO("D-Bus Interface", "com.example.MyService.Interface")
Q_PROPERTY(QString Version READ version CONSTANT)
Q_PROPERTY(double Temperature READ temperature WRITE setTemperature NOTIFY temperatureChanged)

// Implementation:
QDBusConnection::systemBus().registerObject(
    "/com/example/MyService",
    this,
    QDBusConnection::ExportAllProperties
);
```

### 2. As Regular Method
```cpp
QDBusConnection::systemBus().registerObject(
    "/com/example/MyService",
    this,
    QDBusConnection::ExportAllSlots
);

public Q_SLOTS:
    QString GetData() { return m_data; }
```

### 3. Accessing Exposed Data
Using `busctl`:
```bash
# Read property
busctl get-property com.example.MyService \
    /com/example/MyService \
    com.example.MyService.Interface \
    Version

# Set property
busctl set-property com.example.MyService \
    /com/example/MyService \
    com.example.MyService.Interface \
    Temperature d 36.5
```

## Policy File Example
`/etc/dbus-1/system.d/com.example.MyService.conf`:
```xml
<!DOCTYPE busconfig PUBLIC "-//freedesktop//DTD D-Bus Bus Configuration 1.0//EN"
 "http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">
<busconfig>
  <policy user="root">
    <allow own="com.example.MyService"/>
    <allow send_destination="com.example.MyService"/>
    <allow receive_sender="com.example.MyService"/>
  </policy>

  <policy user="normaluser">
    <allow receive_interface="com.example.MyService.Interface"/>
    <allow send_destination="com.example.MyService"
           send_interface="org.freedesktop.DBus.Properties"/>
  </policy>
</busconfig>
```

## Best Practices
1. Use properties for state data
2. Use methods for actions
3. Use signals for event notifications
4. Document your interfaces in XML
5. Restrict access via policy files

Remember to reload D-Bus after changes:
```bash
sudo systemctl reload dbus
```