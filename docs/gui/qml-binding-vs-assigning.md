# QML: Binding vs. Assigning

## 🧠 In Simple Terms

|Concept|What it means|Example|
|---|---|---|
|**Binding**|Links a property to an expression|`text: someValue + "!"`|
|**Assigning**|Sets a fixed value (breaks any binding)|`text = "Hello"`|

---

## 📌 1. **Binding** (Reactive / Dynamic)

- A **binding** means the value of a property is **automatically updated** when any part of the expression changes.
    
- It’s **declarative** and **reactive**.
    

### ✅ Example:

```qml
property int count: 0
text: "Count is: " + count
```

Here, `text` is **bound** to `count`.  
So if `count` changes → `text` automatically updates.

---

## 📌 2. **Assigning** (Static / Manual)

- An **assignment** means you're setting a **fixed value** at runtime.
    
- It **breaks the binding**. No automatic updates anymore.
    

### ⚠️ Example:

```qml
Keys.onEscapePressed: {
    text = "reset"  // breaks the previous binding
}
```

After this, `text` won't react to changes in `count` anymore.

---

## 🧪 Demo Example

```qml
Rectangle {
    width: 200; height: 100

    property int counter: 0

    Text {
        id: label
        text: "Counter: " + counter   // 🔗 binding
    }

    MouseArea {
        anchors.fill: parent
        onClicked: {
            counter += 1              // label updates automatically
            // label.text = "Clicked" // ❌ this would break the binding
        }
    }
}
```

---

## 📍 Summary

|Feature|Binding|Assignment|
|---|---|---|
|Reactivity|✅ Automatically updates|❌ Static after change|
|Syntax|`text: "Value: " + count`|`text = "Manual value"`|
|Use case|Dynamic UI|One-time or manual override|
|Break binding?|❌ No|✅ Yes|

---

**Pro tip:**  
Avoid assigning to properties that were set via bindings unless you **intend to break the reactivity**.

Would you like to see how to **restore a broken binding** or protect it using `Binding {}`?