build-lists: true

## From one buzzword to another
#<br>
#### Sebastian Osiński

---
# About me

iOS Developer at Tooploox
![inline 45%](tplx_logo.png)

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
[.build-lists: false]
## Modelling API requests with enums

---
* External library, e.g. Moya

---
* External library, e.g. Moya
* Hand made solutions

```swift
// Part of networking code from Let Swift app
enum NetworkRouter: URLRequestConvertible {

    // MARK: Event
    case eventsList(Parameters)
    case eventDetails(Int)

    // MARK: Speakers
    case speakersList(Parameters)
    case speakerDetails(Int)
    case latestSpeakers

    // MARK: Contact
    case contact(Parameters)

    var method: HTTPMethod {
        ...
    }

    var path: String {
        ...
    }

    func asURLRequest() throws -> URLRequest {
        ...
    }
}
```

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

---
## First version

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

    var method: String {
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
# Pros

* All commands / queries are namespaced and easy to find when they need to be used
* 

---
# Cons

* To get all info about request, we need to scroll through whole file and visit each `switch`
* Doesn't scale well - enum grows with each endpoint added
* Handling similar requests causes `switch`es to grow horizontally
* You can't (directly) declare return type of Query

---
```swift
enum CommandMethod: String {
    case post
    case put
    case delete
    case patch
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
typealias CommandSuccessCallback = () -> Void
typealias FailureCallback = (Error) -> Void
typealias QuerySuccessCallback<Entity> = (Entity) -> Void

class ApiClient {

    private let session = URLSession.shared

    func perform(_ command: Command, success: CommandSuccessCallback?, failure: FailureCallback?) {
        let task = session.dataTask(with: command.urlRequest) { (_, response, error) in
            if (response as! HTTPURLResponse).statusCode == 200 {
                success?()
            } else {
                failure?(error!)
            }
        }
    }

    func send(_ query: Query)
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
    static let method = .post

    let id: String
}
```

--- 

