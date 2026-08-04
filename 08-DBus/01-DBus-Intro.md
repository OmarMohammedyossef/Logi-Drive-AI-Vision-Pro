
**DBus (Desktop Bus)** is a message bus system that enables communication between multiple processes running on the same machine. It's widely used on Linux systems for inter-process communication (IPC).

**Qt** provides the `QtDBus` module to simplify DBus integration.

This guide introduces DBus, how to use it in Qt, and provides a complete example.

---

## 📑 table-of-contents

1. [introduction-to-dbus](#1-introduction-to-dbus)

2. [dbus-concepts-service-object-interface](#2-dbus-concepts-service-object-interface)
    
3. [session-bus-vs-system-bus](#3-session-bus-vs-system-bus)
    
4. [dbus-naming-conventions](#4-dbus-naming-conventions)
    
5. [using-dbus-in-qt](#5-using-dbus-in-qt)
    
6. [debugging-dbus-applications](#6-debugging-dbus-applications)    

---

## 1-introduction-to-dbus

### what-is-dbus?

DBus is an IPC system that allows multiple applications to communicate on the same machine. It supports both system-wide and per-user communication channels.

### dbus-architecture

- **System Bus**: For communication between system services (e.g., hardware-related daemons like NetworkManager).
    
- **Session Bus**: For communication between user applications in a graphical or shell session.
    

### core-concepts-summary

| Concept          | Description                                                                                     |
| ---------------- | ----------------------------------------------------------------------------------------------- |
| **Bus**          | A communication channel (System or Session Bus).                                                |
| **Service Name** | Unique identifier for a DBus application (e.g., `com.example.MyService`).                       |
| **Object Path**  | Path within the service to expose functionalities (e.g., `/com/example/MyService`).             |
| **Interface**    | Logical grouping of methods, signals, and properties (e.g., `com.example.MyService.Interface`). |
| **Method**       | Callable function exposed via DBus.                                                             |
| **Signal**       | Event notification emitted by a service.                                                        |
| **Property**     | Readable and/or writable values exposed by the object.                                          |
|                  |                                                                                                 |

---
## 2-dbus-concepts-service-object-interface

### service

- A service is the DBus application itself. It registers a unique **service name** (like `com.example.MyService`) on the bus. Only one process can own a service name at a time.

### object

- Each service can register one or more **objects** using an **object path** (e.g., `/com/example/MyService`). Objects expose methods, signals, and properties.
- bject paths are **logical identifiers** within the context of the D-Bus messaging system. They are **defined by developers**, and their structure is meant to **organize and locate D-Bus-exposed objects**, not files.

### interface

An interface defines the functionality (methods, signals, and properties) that an object provides. Interfaces group related behavior and are identified by dotted names (e.g., `com.example.MyService`).

**Summary Table**:

| Concept   | Role                                  | Example                  |
| --------- | ------------------------------------- | ------------------------ |
| Service   | Owns the name on the bus              | `com.example.MyService`  |
| Object    | Exposes functionality via object path | `/com/example/MyService` |
| Interface | Defines API on an object              | `com.example.MyService`  |
> **Note:**
>  - These names can be different. They don’t need to match. But it is strongly recommended to follow a consistent naming convention for clarity and maintainability. 
---

## 3-session-bus-vs-system-bus

| Feature       | Session Bus                            | System Bus                              |
| ------------- | -------------------------------------- | --------------------------------------- |
| Scope         | Per user session                       | System-wide                             |
| Access        | Apps running in a user session         | Services like `NetworkManager`, `udev`  |
| Usage Example | GUI apps communicating with each other | Background daemons exposed to all users |
| Command       | `QDBusConnection::sessionBus()`        | `QDBusConnection::systemBus()`          |
| Debug Tool    | `dbus-monitor --session`               | `dbus-monitor --system`                 |

---

## 4-dbus-naming-conventions

D-Bus uses reverse domain notation. Here's the suggested structure:

|Component|Format Example|Notes|
|---|---|---|
|Service Name|`com.company.Component`|Identifies the application/service|
|Object Path|`/com/company/Component`|Identifies the object within the service|
|Interface Name|`com.company.Component.Interface`|Logical contract exposed by the object|

### Example

If your company is `example.com` and your component is `MyService`:

- **Service**: `com.example.MyService`
    
- **Object**: `/com/example/MyService`
    
- **Interface**: `com.example.MyService.Interface`

> **Notes:**
> -  **Service name** Must be unique per bus .
> -  **Object path** Must begin with `/`.


---

## 5-using-dbus-in-qt

### implement-the-service

```cpp
class MyService : public QObject {
    Q_OBJECT
    Q_CLASSINFO("D-Bus Interface", "com.example.MyService")

public:
    QString SayHello(const QString &name) {
        QString response = "Hello, " + name + "!";
        emit MessageReceived(response);
        return response;
    }

signals:
    void MessageReceived(const QString &message);
};
```

### register-the-service

```cpp
QDBusConnection connection = QDBusConnection::sessionBus();
connection.registerService("com.example.MyService");
connection.registerObject("/com/example/MyService", &service,
    QDBusConnection::ExportAllSlots | QDBusConnection::ExportAllSignals);
```

---

### create-a-dbus-client

```cpp
QDBusInterface interface(
    "com.example.MyService",
    "/com/example/MyService",
    "com.example.MyService",
    QDBusConnection::sessionBus()
);

if (interface.isValid()) {
    QDBusReply<QString> reply = interface.call("SayHello", "World");
    qDebug() << "Response:" << reply.value();
}
```

---

### handle-dbus-signals

```cpp
QObject::connect(&interface, SIGNAL(MessageReceived(QString)),
                 &app, SLOT(handleMessage(QString)));
```

---


---

## 6-debugging-dbus-applications

### use-dbus-monitor

```bash
dbus-monitor --session
```

### common-issues

|Issue|Solution|
|---|---|
|Service not registering|Ensure the name is unique and not in use|
|Invalid interface or path|Check for typos in interface/path names|
|Permission denied|May need systemd config or policy override|


---
---

## check

- [DBus Tutorial - SoftPrayog](https://www.softprayog.in/programming/d-bus-tutorial)
    
- [YouTube: DBus Overview](https://www.youtube.com/watch?v=3ZwzfZgU598)
    
- [Bootlin DBus Slides (PDF)](https://bootlin.com/pub/conferences/2016/meetup/dbus/josserand-dbus-meetup.pdf)
    