# D-Bus-Signal-Example  
*A Qt-based guide to emitting and receiving D-Bus signals between processes using the system bus*  

---

## Table-of-Contents  
- [Sender-Process-(Signal-Emitter)](#sender-process-signal-emitter)  
- [Receiver-Process-(Signal-Listener)](#receiver-process-signal-listener)  
- [Verification](#verification)  
- [Troubleshooting](#troubleshooting)  

---

## Introduction  
This guide demonstrates how to:  
1. **Emit a D-Bus signal** from one Qt process  
2. **Receive the signal** in another process  
3. **Use the system bus** (requires root privileges)  

D-Bus signals enable **asynchronous communication** between processes on Linux systems.  


---

## Sender-Process-(Explicit-Signal-Emitter)  
```cpp
#include <QCoreApplication>
#include <QDBusConnection>
#include <QDBusMessage>
#include <QDebug>

int main(int argc, char *argv[]) {
    QCoreApplication app(argc, argv);

    // Connect to system bus (requires root)
    QDBusConnection bus = QDBusConnection::systemBus();
    if (!bus.isConnected()) {
        qWarning("Cannot connect to system bus!");
        return 1;
    }

    // Emit signal with value 42
    QDBusMessage signal = QDBusMessage::createSignal(
        "/com/example/MyService",       // Object path
        "com.example.MyService.Interface", // Interface
        "DataUpdated"                   // Signal name
    );
    signal << 42;  // Attach signal payload

    if (!bus.send(signal)) {
        qWarning("Failed to emit signal!");
    } else {
        qDebug() << "Signal emitted with value: 42";
    }

    return app.exec();
}
```

> Note this is how explicitly adding a signal

## Sender-Process(Class-Signal)
```cpp
class MyService : public QObject {
    Q_OBJECT
    Q_CLASSINFO("D-Bus Interface", "com.example.MyService.Interface")

public:
    MyService(QObject *parent = nullptr) : QObject(parent) {}

signals:
    void CurrentVersion(double version);
};

////////////////////////////////////
   QDBusConnection connection = QDBusConnection::systemBus();

    // Register the servic  e
    if (!connection.registerService("com.example.MyService")) {
        qWarning() << "Failed to register service:" << connection.lastError().message();
        return 1;
    }

    // Register the object
    if (!connection.registerObject("/com/example/MyService", &service, QDBusConnection::ExportAllSlots | QDBusConnection::ExportAllSignals)) {
        qWarning() << "Failed to register object:" << connection.lastError().message();
        return 1;
    }

```

> Note by this way the signal will appears when you try to introspect the dbus service  
---

## Receiver-Process-(Signal-Listener)  
```cpp
#include <QCoreApplication>
#include <QDBusConnection>
#include <QDBusInterface>
#include <QDebug>

class Receiver : public QObject {
    Q_OBJECT
public slots:
    void handleUpdate(int value) {
        qDebug() << "Received update:" << value;
    }
};

int main(int argc, char *argv[]) {
    QCoreApplication app(argc, argv);
    Receiver receiver;

    QDBusConnection bus = QDBusConnection::systemBus();
    bus.connect(
        "",                             // Any service
        "/com/example/MyService",      // Object path
        "com.example.MyService.Interface", // Interface
        "DataUpdated",                 // Signal name
        &receiver,
        SLOT(handleUpdate(int))        // Slot signature
    );

    qDebug() << "Listening for signals...";
    return app.exec();
}

#include "main.moc"  // For Q_OBJECT macro
```

---

## Verification  

### Using `dbus-send` (CLI)  
```bash
dbus-send --system \
  --type=signal \
  /com/example/MyService \
  com.example.MyService.Interface.DataUpdated \
  int32:42
```

### Using `busctl` (CLI)  
```bash
busctl emit \
  --system \
  /com/example/MyService \
  com.example.MyService.Interface \
  DataUpdated i 42
```

---

### Using `busctl monitor` (CLI)  
```bash
busctl monitor --system com.example.MyService
# Output:
# Signal DataUpdated i 42
```

### Using `dbus-monitor` (CLI)  
```bash
dbus-monitor --system \
  "type='signal',interface='com.example.MyService.Interface'"
```


---

## Troubleshooting  
| Error | Solution |
|-------|----------|
| `"Could not connect to signal"` | Verify interface/path names match exactly |
| `"Permission denied"` | Ensure policy allows your user in XML config |
| `"No such signal"` | Recompile the XML with `qdbusxml2cpp` |

---

## Final-Notes  
1. **System Bus**: Requires root privileges and proper policy configuration  
2. **Thread Safety**: DBus connections are thread-safe by default  
3. **Debugging**: Use `journalctl -f` to monitor system bus messages  

This provides **reliable IPC** for system-level applications! 🚀