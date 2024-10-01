# Notes


### Module Path
- **What?** globally unique name or identifier for project
- **Why Globally Unique?** to avoid potential conflicts with other people's projects or the standard library
- **How to avoid?** base module paths on a URL we own or if it has to be used and downloaded by others from some site like GitHub base it on download path (not necessary but good practice) e.g., `github.com/mubashirmalik/pyro`

### Turn Project Directory into a module

Run below command in root directory of project
```
go mod init github.com/mubashirmalik/pyro

go: creating new go.mod: module github.com/mubashirmalik/pyro
```

### Why set project as module?
1. Makes it easier to manage third-party dependencies (imagine go.mod as package.json)
2. Avoid supply-chain attacks <mark>How?</mark>
3. Ensure reproducible builds of application in future <mark>How?</mark>

### Three absolute essentials for Web App?
#### 1. Handlers
- Equate to Controller of MVC or Controller/Service of Nest
- Responsible for executing application logic (Service of Nest) and writing HTTP response headers & bodies (Controller of Nest)

#### 2. Servemux
- Router
- Stores mapping between URL routing patterns & corresponding handlers
- **Example:** What logic/code to execute when `/any-url` is hit.

#### 3. Server
- No third-party library needed like Apache, Caddy or Nginx

### Example of Web Server
```go
// Define a home handler function which writes a byte slice containing
// "Hello from Snippetbox" as the response body.
func home(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("Hello from Snippetbox"))
}

func main() {
    // Use the http.NewServeMux() function to initialize a new servemux, then
    // register the home function as the handler for the "/" URL pattern.
    mux := http.NewServeMux()
    mux.HandleFunc("/", home)

    // Use the http.ListenAndServe() function to start a new web server. We pass in
    // two parameters: the TCP network address to listen on (in this case ":4000")
    // and the servemux we just created. If http.ListenAndServe() returns an error
    // we use the log.Fatal() function to log the error message and exit. Note
    // that any error returned by http.ListenAndServe() is always non-nil.
    err := http.ListenAndServe(":4000", mux)
    log.Fatal(err)
}
```

Serve mux treats route pattern `/` like a catch-all. So, currently all requests will be handled by home handler.

### Named ports
- We can provide `http` or `http-alt` as network address.
- `ListenAndServe` will check `/etc/services` for value of named port, returns an error if no match is found.
- TCP network address should be in format `"host:port"`
- If we omit host e.g., `":4000"`, then server will listen on available network interface. 
- If we have more than one network interface (which I don't know how & when?), then we need to specify host as well.

### Go Run
Compiles, creates an executable binary in `/tmp` directory, and then runs this binary in one step.
```c++
    go run . // . treated as current working directory
    go run main.go // space-separated list of go files
    go run github.com/mubashirmalik/pyro
```

### Routing Patterns

#### Exact Match
- When a route pattern doesn't end in trailing slash e.g., `/snippet/create`
- When the request URL matches exactly, only then the corresponding handler will be called

#### Subtree Path Pattern

- When a route pattern ends with a trailing slash — like `/` or `/static/`
- These are matched whenever the start of a request URL path matches the subtree path
- For understanding consider above like `/**` or `/static/**`
- This is why `/` acts as catch-all

#### Restricting subtree paths
- To avoid `/` as a catch-all, use `{$}`
- For example: `/{$}` will only match requests where the URL path is exactly `/`
- It is only permitted to be used at the end. Else, Panic!

#### Automatic Sanitization
- `/foo/bar/..//baz` they will automatically be sent a 301 Permanent Redirect to `/foo/baz` instead.
- **Why?**  if the request path contains any `.` or `..` elements or repeated slashes, the user will automatically be redirected to an equivalent clean URL

- If a subtree path is registered e.g., `/foo/` then any request received for `/foo` will be automatically redirected (301) to `/foo/`. Opposite is not true :) 

#### Host name matching
- Useful for redirects
- Useful when app is working as a backend for multiple different sites.. Maybe a load balancer?

```go
mux := http.NewServeMux()

// any host-specific patterns will be checked first
mux.HandleFunc("foo.example.org/", fooHandler)
mux.HandleFunc("bar.example.org/", barHandler)

// Only when there isn’t a host-specific match found will the non-host specific patterns also be checked
mux.HandleFunc("/baz", bazHandler)
```


### Default Serve Mux
- Pass `nil` as the second argument to `ListenAndServe`
- `http.DefaultServeMux` is a **global** variable in standard library
- **Don't use! Why?**
    - **Security**: As it it global, any Go code can register a handler (even any third-party library)
    - **Clarity & maintainability**: Difficult to locate routes as many people working on code can register from any file.
