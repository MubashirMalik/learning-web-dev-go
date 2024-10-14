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

### Project Structure
- Project Folder e.g., `pyro-go`
    - **cmd** application-specific code for the executable applications in the project
        - **web**
            - handlers.go
            - main.go
    - **internal** contain non-application-specific code used in the project e.g., reusable code like validation helpers and the SQL database models for the project.
    - **ui** contain the user-interface assets used by the web application
        - **html** contain html templates
        - **static** contain static files (like CSS and images).
    - **go.mod**

#### Benefits?
- Separation between Go and non-Go assets.
- Scales.. if we want to add another executable app e.g., CLI to automate some tasks we can create **cmd/cli** and it will be able to use existing code in **internal**
- internal: 
    - name carries significance
    - code inside internal can only be used in parent of internal i.e., our web app.
    - no third-party library can use code from here.

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
- When a route pattern **doesn't end** in trailing slash e.g., `/snippet/create`, then only when the request URL matches exactly, corresponding handler will be called.

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

### Wildcard route patterns
- Query Params of Nest
- denoted by an wildcard identifier inside `{}`

```go
mux.HandleFunc("/products/{category}/item/{itemID}", exampleHandler)

// /products/hammocks/item/sku123456789
// /products/seasonal-plants/item/pdt-1234-wxyz
// /products/experimental_foods/item/quantum%20bananas

// Patterns like "/products/c_{category}", /date/{y}-{m}-{d} or /{slug}.html are not valid
```

- Use `r.PathValue()` in handler to retrieve values.
- `r.PathValue()` method always returns a string value.

```go
    // Extract the value of the id wildcard from the request using r.PathValue()
    // and try to convert it to an integer using the strconv.Atoi() function. If
    // it can't be converted to an integer, or the value is less than 1, we
    // return a 404 page not found response.
    id, err := strconv.Atoi(r.PathValue("id"))
    if err != nil || id < 1 {
        http.NotFound(w, r)
        return
    }
```
- stick a `{$}` at the end — like `/user/{id}/{$}` to avoid subtree path pattens with wildcards
### Precedence and conflicts
- Most specific pattern wins!
- Consider two route patterns: `/post/{id}` & `/post/edit`, what will happen on request with the path `/post/edit`?
- `/post/edit` will only be invoked when exactly `/post/edit` is requested.
- Avoid these at all costs as there is an edge case where go panics!  

#### Edge case?

For example, the patterns `/post/new/{id}` and `/post/{author}/latest` overlap because they both match the request path `/post/new/latest`, but it’s not clear which one should take precedence. In this scenario, Go’s servemux considers the patterns to conflict, and will panic at runtime when initializing the routes.


### Remainder Wildcards

- I don't think so I will be needing this. If I even want to use it; will update notes first.

### Method-based routing
```go
mux.HandleFunc("GET /{$}", home)
```
- Method is case sensitive (uppercase).
- Should be followed by spaces (at least 1) or tab
- `GET` will also me matched for `HEAD` requests.
- Go’s servemux automatically sends a **405 Method Not Allowed** response for us, including an Allow header which lists the HTTP methods that are supported for the request URL.

- Most specific pattern wins applies here too. For example: `/article/{id}` will match incoming HTTP requests with any method. In contrast, a route like `POST /article/{id}` will only match requests which have the method POST.

### Why third-party routers?
- The wildcard and method-based routing functionality that we’ve been using in the past two chapters is relatively new to Go — it only became part of the standard library in Go 1.2

- Sending custom 404 Not Found and 405 Method Not Allowed responses to the user (although there is an open proposal regarding this).
- Using regular expressions in your route patterns or wildcards.
- Matching multiple HTTP methods in a single route declaration.
- Automatic support for OPTIONS requests.
- Routing requests to handlers based on unusual things, like HTTP request headers.

- Recommended by Author: **chi**, **flow**, **gorilla/mux** and **httprouter**

### Sending Customize Responses (HTTP Status Codes)

```go
func snippetCreatePost(w http.ResponseWriter, r *http.Request) {
    // Use the w.WriteHeader() method to send a 201 status code.
    w.WriteHeader(201)

    // Then w.Write() method to write the response body as normal.
    w.Write([]byte("Save a new snippet..."))
}
```

#### w.WriteHeader
- It can only be called once per response.
- Status code can't be changed once written. Trying to do so will log a warning.
- If it isn't called then `w.Write` automatically sends status code `200`.
- It should be called before call to `w.Write`
- Use constant status codes rather than writing number ourselves e.g., 201 -> `http.StatusCreated`

#### Adding additional header
- We can add additional headers by changes response header map.
- Example: ` w.Header().Add("Server", "Go")`
- We have to make sure that response header map contains all the headers we want before the call to `w.WriteHeader()` or `w.Write()`.
- There are other useful methods like `Get`, `Del`, `Set` and `Values` that can be used to manipulate response header map

### io.Write Interface 
```go
// Instead of this...
w.Write([]byte("Hello world"))

// We can do this...
io.WriteString(w, "Hello world")
fmt.Fprint(w, "Hello world")
```
### http.DetectContentType()
- Go automatically tries to set `Content-Type` header by sniffing the response body.
- If can't detect, falls back to `Content-Type: application/octet-stream` instead.
- It can't distinguish between plain text and JSON.
- Sets header to `Content-Type: text/plain; charset=utf-8` for responses of JSON by default
- To prevent we can set manually: `w.Header().Set("Content-Type", "application/json")`

### Header canonicalization
<mark>Didn't understand</mark>

### HTML Templating

```go
func home(w http.ResponseWriter, r *http.Request) {
    w.Header().Add("Server", "Go")

    // absolute path or relative to cwd
    ts, err := template.ParseFiles("./ui/html/pages/home.tmpl")
    if err != nil {
        log.Print(err.Error())
        http.Error(w, "Internal Server Error", http.StatusInternalServerError)
        return
    }

    err = ts.Execute(w, nil)
    if err != nil {
        log.Print(err.Error())
        // It is a lightweight helper function sends a plain text error message 
        http.Error(w, "Internal Server Error", http.StatusInternalServerError)
    }
}
```

```html
{{define "base"}}
<!doctype html>
<html lang='en'>
    <head>
        <meta charset='utf-8'>
        <title>{{template "title" .}} - Snippetbox</title>
    </head>
    <body>
        <header>
            <h1><a href='/'>Snippetbox</a></h1>
        </header>
        <main>
            {{template "main" .}}
        </main>
        <footer>Powered by <a href='https://golang.org/'>Go</a></footer>
    </body>
</html>
{{end}}
```