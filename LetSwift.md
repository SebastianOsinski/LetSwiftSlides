# From one buzzword to another
## Sebastian Osiński

---
# About me

iOS Developer at Tooploox

---
# Usages of enums in Swift

---
# Plain old enumerations

```swift
    enum Device {
        case iPhone
        case iPad
        case mac
    }
```

---
# Algebraic data types

```swift
    indirect enum Tree<T> {
        case leaf(T)
        case node(Tree<T>, Tree<T>)
        case empty
    }

    let tree: Tree<Int> = 
        .node(
            .node(
                .leaf(1),
                .leaf(2)
            ),
            .node(
                .empty,
                .leaf(3)
            )
        )
```

---
# Real life's a bitch
---
# CQRS
## Command Query Responsibility Segregation
---
## Command
Changes state, does not return anything
## Query
Returns some data, does not change state


