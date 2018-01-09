build-lists: true
slidenumbers: true
theme: Next, 1

## From one buzzword to another
#<br>
#### Sebastian Osiński

---
# About me

### iOS Developer at Tooploox
![inline 43%](tplx_logo.png)

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
* Everything in one file

---
# Cons

* Everything in one file :sob:
* To get all info about request, we need to scroll through whole file and visit each `switch`
* Doesn't scale well - enum grows with each endpoint added
* Handling similar requests causes `switch`es to grow horizontally
* You can't (directly) declare return type of Query

---
## Protocol oriented approach

---
## Key components:
* Base protocols with default implementations for creating `URLRequest`
* Specialized protocols for common types of Commands / Queries
* Each command / query is defined as a simple struct

---

## Base protocols: Command

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

    var body: JSON { get } // typealias JSON = [String: Any]
}

extension Command {

    var urlRequest: URLRequest {
        // generates proper urlRequest using `path`, `method` and `body`
        ...
    }
}
```
---
## Base protocols: Query

```swift
protocol Query {
    associatedtype Result: Decodable

    static var path: String { get }

    var parameters: [String: String]? { get }
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
    private let jsonDecoder = JSONDecoder()

    func execute(_ command: Command, success: CommandSuccessCallback?, failure: FailureCallback?) {
        let task = session.dataTask(with: command.urlRequest) { (_, response, error) in
            if 200...299 ~= (response as! HTTPURLResponse).statusCode {
                success?()
            } else if let error = error {
                failure?(error)
            } else {
                // additional error handling
            }
        }

        task.resume()
    }

    func execute<Q: Query, Result>(_ query: Q, success: QuerySuccessCallback<Result>?, failure: FailureCallback?) where Result == Q.Result {
        let task = session.dataTask(with: query.urlRequest) { [jsonDecoder] (data, response, error) in
            if let error = error {
                failure?(error)
            } else if let data = data {
                do {
                    let result = try jsonDecoder.decode(Result.self, from: data)
                    success?(result)
                } catch {
                    failure?(error)
                }
            } else {
                // additional error handling ...
            }
        }

        task.resume()
    }
}

```

---
## Specialized command protocol example

```swift
protocol IdCommand: Command {
    var id: String { get }
}

extension IdCommand {

    var body: JSON {
        return ["id": id]
    }
}
```

---
## Usage - command for liking videos

```swift
struct LikeVideoCommand: IdCommand {

    static let path = "like_video"
    static let method = .post

    let id: String
}

apiClient.execute(LikeVideoCommand(id: "let_swift_13_speaker_1_intro"), success: {
    print("👏 🎉 👍")
}, failure: nil)
```

---

## Specialized command protocol example

```swift
protocol CommandBody {
    var json: JSON { get }
}

protocol BodyCommand: Command {
    associatedType Body: CommandBody

    var body: Body
}

extension BodyCommand {

    var body: JSON {
        return body.json
    }
}
```

---

## Usage - command for adding new person

```swift
struct PersonCommandBody: CommandBody {
    
    let id: String
    let firstName: String
    let lastName: String

    var json: JSON {
        // create json from fields above
        // or just make it Encodable and tell compiler to do the dirty job 
        ...
    }
}

struct PersonCommand: BodyCommand {

    static let path = "add_person"
    static let method = .post

    let body: PersonCommandBody
}

```

---

# Pros
* All request's configuration in one place
* It's easy to model similar requests

---
# Cons
* Lack of namespacing. It can be added though - nested structs
* It's possible to forget to implement extension for given combination of protocols and compiler won't warn us. Solution: unit tests
* Some code repetition in extensions.
Solution: extracting building body/parameters dictionaries to static methods

---
# THANK YOU!

###This presentation can be found here:
https://github.com/SebastianOsinski/LetSwiftSlides

---
## QUESTIONS?

