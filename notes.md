# Notes


### Module Path
- **What?** globally unique name or identifier for project
- **Why Globally Unique?** to avoid potential conflicts with other people's projects or the standard library
- **How?** base module paths on a URL we own or if it has to be used and downloaded from some site like GitHub base it on download path (not necessary but good practice) e.g., `github.com/mubashirmalik/pyro`

### Turn Project Directory into a module

Run below command in root directory of project
```
go mod init github.com/mubashirmalik/pyro

go: creating new go.mod: module github.com/mubashirmalik/pyro
```

### Why set project as module?
1. Makes it easier to manage third-party dependencies (imagine go.mod as package.json)
2. Avoid supply-chain attacks
3. Ensure reproducible builds of application in future

### Handlers
- Equate to Controller of MVC or Controller/Service of Nest
- Responsible for executing application logic (Service of Nest) and writing HTTP response headers & bodies (Controller of Nest)

### Servemux
- Router
- Stores mapping between URL routing patterns & corresponding handlers
- **Example:** What logic/code to execute when `/any-url` is hit.

### Server
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
Compiles, creates an executable binary in /tmp directory, and then runs this binary in one step.
```c++
    go run . // . treated as current working directory
    go run main.go // space-separated list of go files
    go run github.com/mubashirmalik/pyro
```