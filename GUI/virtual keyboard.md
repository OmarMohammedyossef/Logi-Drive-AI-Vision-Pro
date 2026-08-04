
Here’s a **complete README** for setting up and using the **Qt Virtual Keyboard** in **Qt 5** with QML. It includes a summary of the problems you encountered, their solutions, and a minimal working example.

---

# 🧰 Qt Virtual Keyboard – README

This guide walks you through enabling and using the **Qt Virtual Keyboard** in your Qt 5 application, especially for embedded or touchscreen systems like Raspberry Pi.

---

## 📋 Requirements

- Qt 5.5 or later (you are using Qt 5.15 — ✅)
    
- The following packages must be installed:
    

```bash
sudo apt install qml-module-qtquick-virtualkeyboard
sudo apt install libqt5virtualkeyboard5 libqt5virtualkeyboard5-dev 
sudo apt install qtvirtualkeyboard-plugin
```

Ensure your Qt supports the virtual keyboard plugin:

```bash
ls /usr/lib/x86_64-linux-gnu/qt5/qml/QtQuick/VirtualKeyboard
```

You should see files like:

```
InputPanel.qml
Keyboard.qml
plugins.qmltypes
```

---

## 🐞 Problems You Encountered & Solutions

|❌ Problem|✅ Solution|
|---|---|
|`MessageDialog is not a type`|Use `Dialog` from `QtQuick.Controls` instead of `MessageDialog` (was removed or changed in some versions)|
|`InputPanel is not a type`|You didn't have `qml-module-qtquick-virtualkeyboard` installed or not using the correct import|
|`"module QtQuick.VirtualKeyboard.Plugins" is not installed`|Missing keyboard plugin dependencies or plugin not loaded due to environment config|
|Virtual keyboard not showing up|You **must set** the environment variable `QT_IM_MODULE=qtvirtualkeyboard`|
|Keyboard appears blank|Caused by missing QML modules or wrong plugin config|
|Can't interact with keyboard|No `InputPanel` in view or `Qt.inputMethod.show()` was not called correctly|

---

## ✅ Solution Summary

1. **Install the virtual keyboard QML module**:
    
    ```bash
    sudo apt install qml-module-qtquick-virtualkeyboard
    ```
    
2. **Set this environment variable** (before running the app):
    
    ```bash
    export QT_IM_MODULE=qtvirtualkeyboard
    ```
    
3. (Optional) In C++:
    
    ```cpp
    qputenv("QT_IM_MODULE", QByteArray("qtvirtualkeyboard"));
    ```
    
4. **Import modules in QML**:
    
    ```qml
    import QtQuick.VirtualKeyboard 2.2
    ```
    
5. **Add the keyboard UI (`InputPanel`) to your QML**.
    

---

## 🧪 Minimal Working Example

### `main.qml`

```qml
import QtQuick 2.15
import QtQuick.Controls 2.15
import QtQuick.VirtualKeyboard 2.2

ApplicationWindow {
    visible: true
    width: 400
    height: 300
    title: "Virtual Keyboard Example"

    Column {
        spacing: 20
        anchors.centerIn: parent

        TextField {
            id: inputField
            placeholderText: "Type something"
            focus: true

            onActiveFocusChanged: {
                if (activeFocus) {
                    Qt.inputMethod.show()
                }
            }
        }
    }

    // Always visible virtual keyboard
    InputPanel {
        id: virtualKeyboard
        z: 100
        anchors.left: parent.left
        anchors.right: parent.right
        anchors.bottom: parent.bottom
        visible: Qt.inputMethod.visible
    }
}
```

### How to Run

```bash
export QT_IM_MODULE=qtvirtualkeyboard
qmlscene main.qml
```

---

## 📌 Tips for Embedded Devices

- Use **fullscreen** mode: `ApplicationWindow { visibility: "FullScreen" }`
    
- Adjust `InputPanel` height if needed.
    
- If you're using C++, add this in `main.cpp` before creating `QApplication`:
    

```cpp
qputenv("QT_IM_MODULE", QByteArray("qtvirtualkeyboard"));
```

---

## 📦 Optional Features

You can customize:

- Keyboard layout: `InputPanel { locale: "en_US" }`
    
- Style: use `QtQuick.VirtualKeyboard.Styles`
    
- Embedded prediction, handwriting: requires extra modules
    

---

## 🧩 Useful Imports

```qml
import QtQuick.VirtualKeyboard 2.2
import QtQuick.VirtualKeyboard.Settings 2.2
import QtQuick.VirtualKeyboard.Styles 2.2
```

---

Would you like me to save this as a `README.md` file for your project directory?