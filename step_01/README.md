# Hello World

Let's start by writing the customary "Hello World" program. Our our case it'll
be an HTTP server.

## Code Directory

Go code should be under `$GOPATH/src`, if you didn't set `$GOPATH` it'll default to `~/go`.

Create a directory (say `restless`) under `$GOPATH/src` (e.g.
`$GOPATH/src/restless`) and place you code there.

## Code

[httpd.go](httpd.go)


```go
package main

import (
	"fmt"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "שלום Gophers\n")
}

func main() {
	http.HandleFunc("/", handler)
	http.ListenAndServe(":8080", nil)
}
```

## Line by Line

Let's go over the code line by line. We can clearly see the influence `C` has on
Go.

The formatting of the code is done by the `go fmt` tools. Most Go-aware editors
run it on save. This ends most discussion on code format and styling - leaving
time for more productive things.

We [say][proverbs] that "Gofmt's style is no one's favorite, yet gofmt is
everyone's favorite." - get used to it.

[proverbs]: https://go-proverbs.github.io/


```go
package main
```

Go is designed for large project. To enable modularity and reuse, code lives in
packages. The package `main` is special and is the one Go executes initially.


```go
import (
	"fmt"
	"net/http"
)
```

This is how we import other packages into our program. Go packages are binary
and contain all the type information so there no need for header files.

The `go` tool looks for packages in several places. First it'll try to look for
packages from the standard library, which are under `$GOROOT`. Then it'll try
to look for them in a folder called `vendor` in the current directory and then
under `$GOPATH` (which defaults to `$HOME/go`).

As you can see in `net/http`, packages can be nested to help organize code.

```go
func handler(w http.ResponseWriter, r *http.Request) {
```

We define functions with the `func` keyword. We then have function name and then
a list of parameters in parenthesis.

Go is a typed language and every parameter has a type. In reverse from `C`, we
first state the parameter name and then its type.

`w` is of type `http.ResponseWriter` which is an interface. An interface
defines a set of functions and let us substitute one type with another as long
as they define the same interface. For example many types implement the
`io.Reader` interface which allows the same function to read from a file or a
socket the same way.

`r` is a pointer to `http.Request`, a struct that holds the current HTTP request
information (we'll talk about structs later). Go has pointers but just as a
mean for passing objects by reference. We don't have the C/C++ pointer
arithmetics which makes Go safer.

We imported `net/http`, however when accessing types we use only the last part
of the package name, e.g. `http.Request` and not `net.http.Request`


```go
	fmt.Fprintf(w, "שלום Gophers\n")
```

This is an example of a function call. We must prefix the function with the
package that contains is (e.g. `fmt.Printf` and *not* `Printf`).

The rule for exporting is simple - everything that start with an upper case
letter is exported and can be used from other packages. If there are function
defined in `fmt` which start with lower case letter - we don't have access to
them. Only the code inside the `fmt` package can use them.

If you look at [fmt.Fprintf][fprintf] signature, you'll see it expects an
`io.Writer` as first parameter. `w` is of type `http.ResponseWriter` - so how
does this work? `io.ResponseWriter` is a superset of `io.Writer` and can be
used as one.

Strings in Go are unicode, when iterating over a string you'll get the unicode
code points which in Go are called runes.

[fprintf]: https://golang.org/pkg/fmt/#Fprintf


```go
func main() {
```

`main is the function the Go runtime will call at the beginning of the program.
Program execution will start here.

```go
	http.HandleFunc("/", handler)
```

Here we tell the `http` package that HTTP calls to `/` should be routed to the
`handler` function. Functions in Go are what's called "first class objects",
meaning we can use them like regular objects - pass them as paremeters, assign
variables to them, create new ones on the fly ...

```go
	http.ListenAndServe(":8080", nil)
```

Here we start the HTTP server on port 8080. The `:` before the port number
means our server listen on all interface (if you're not sure what this sentence
mean - don't worry about it right now).

`nil` is the "nothing" value. `ListenAndServe` expects an extra parameter and we
currently don't need to pass it so we pass `nil` instead. You can't assign
default values to function parameter in Go.

`ListenAndServe` will run in infinite loop waiting for HTTP requests from
clients.

The Go HTTP server is very fast and powerful. It can serve many requests, handle
SSL and supports HTTP 2.

## Running

Go compile our program run

    go build -o httpd

The `-o httpd` tells the `go` tool the name of the executable to create. If you
don't specify it it'll be named in the name of the current directory.

(On Windows the executable will be called `httpd.exe`)

Now we can run it

    ./httpd

Once you run your program, point your browser to `http://localhost:8080` and you
should see our message there. Or you can use `curl`

    curl http://localhost:8080

Since go compiles quicky, you can also execute 

    go run httpd.go

which compiles and runs the program in a single step. It's a great way to
quickly test changes to your code.
