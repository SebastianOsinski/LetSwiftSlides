## From one buzzword to another
#<br>
#### Sebastian Osiński

---
# About me

iOS Developer at Tooploox
![inline 50%](tplx_logo.png)

---
## Usages of enums in Swift

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

---
## First version

```swift
enum Query {

    case project(id: String)
    case projects(filters: [String: String])
    case users
    case teams

    var path: String {
        switch self {
        case .project, .projects:
            return "projects"
        case .users:
            return "users"
        case .teams:
            return "teams"
    }

    var urlRequest: URLRequest {
        ...
    }
}
```

```swift
enum Command {
    case saveDraft(title: String, description: String)
    case updateDraft(title: String, description: String, id: String)
    case submitProject(id: String)

    var path: String {
        switch self {
            ...
        }
    }

    var body: [String: Any] {
        switch self {
            ...
        }
    }

    var urlRequest: URLRequest {
        ...
    }
}
```

---

### Looks fine at the beginning, but has one big problem - doesn't scale well

---
```swift
enum CommandMethod: String {
    case POST
    case PUT
    case DELETE
    case PATCH
}

protocol Command {
    static var path: String { get }
    static var method: CommandMethod { get }

    var body: [String: Any] { get }
    var urlRequest: URLRequest { get }
}

extension Command {

    var urlRequest: URLRequest {
        // generates proper urlRequest using `path`, `method` and `body`
    }
}
```
---

```swift
protocol Query {
    static var path: String { get }
    var parameters: [String: String]? { get }
    var urlRequest: URLRequest { get }
}

extension Query {

    var urlRequest: URLRequest {
        // always GET
        // generates proper urlRequest using `path` and `parameters`
        ...
    }
}
```
---
```swift
protocol IdCommand: Command {
    var id: String { get }

    init(id: String)
}

extension IdCommand {

    var body: [String: Any] {
        return ["id": id]
    }
}
```

---
## Usage - command for liking posts

```swift
struct LikePostCommand: IdCommand {

    static let path = "like"
    static let method = .POST

    let id: String
}
```

--- 

