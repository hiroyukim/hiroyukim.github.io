## Golang: Difference between http.Handle and ServeMux

Currently, when you try to write a server with Go there are a form to register with http.Handle and a form to create an object with ServeMux and register Handle there.

What are the differences between the two?

### What does http.Handle do?

When http.Handle receives an argument it registers with DefaultServeMux.

```go
func Handle(pattern string, handler Handler) { DefaultServeMux.Handle(pattern, handler) }
```

What is this DefaultServe Mux?
Actually it is written in the beginning of the document.

> ListenAndServe starts an HTTP server with a given address and handler. The handler is usually nil, which means to use DefaultServeMux. Handle and HandleFunc add handlers to DefaultServeMux:

This is declared to the upper part of net / http package

```go
// DefaultServeMux is the default ServeMux used by Serve.
var DefaultServeMux = &defaultServeMux

var defaultServeMux ServeMux
```

By looking at the declaration of defaultServeMux, it is ServeMux, so it becomes a difference whether you declare an object explicitly or use the default one as a result.

Is it to use it because you want to declare multiple objects etc.
Basically, the default provided with http should be enough.

### See

+ https://golang.org/pkg/net/http/

