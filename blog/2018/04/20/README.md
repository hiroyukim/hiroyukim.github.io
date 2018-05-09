## Golang Standard library (net.http ServeMux)

+ https://golang.org/pkg/net/http/#NewServeMux

### NewServeMux

> NewServeMux allocates and returns a new ServeMux.

```go
func NewServeMux() *ServeMux { return new(ServeMux) }
```

### ServeMux

> ServeMux is an HTTP request multiplexer.

```go
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry
	hosts bool // whether any patterns contain hostnames
}
```

### muxEntry

```go
type muxEntry struct {
	h       Handler
	pattern string
}
```

### Handler

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

### ServeMux Handle

`mux.mu.Lock()` is an Important points.

```go
func (mux *ServeMux) Handle(pattern string, handler Handler) {
	mux.mu.Lock()
	defer mux.mu.Unlock()

	if pattern == "" {
		panic("http: invalid pattern")
	}
	if handler == nil {
		panic("http: nil handler")
	}
	if _, exist := mux.m[pattern]; exist {
		panic("http: multiple registrations for " + pattern)
	}

	if mux.m == nil {
		mux.m = make(map[string]muxEntry)
	}
	mux.m[pattern] = muxEntry{h: handler, pattern: pattern}

	if pattern[0] != '/' {
		mux.hosts = true
	}
}
```

### ServeMux HandleFunc

```go
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	mux.Handle(pattern, HandlerFunc(handler))
}
```

### ListenAndServe

> ListenAndServe always returns a non-nil error.

```go
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}
```

### Server

```go
type Server struct {
	Addr    string  // TCP address to listen on, ":http" if empty
	Handler Handler // handler to invoke, http.DefaultServeMux if nil

	TLSConfig *tls.Config

	ReadTimeout time.Duration

	ReadHeaderTimeout time.Duration

	WriteTimeout time.Duration

	IdleTimeout time.Duration

	MaxHeaderBytes int
	TLSNextProto map[string]func(*Server, *tls.Conn, Handler)
	ConnState func(net.Conn, ConnState)
	ErrorLog *log.Logger

	disableKeepAlives int32     // accessed atomically.
	inShutdown        int32     // accessed atomically (non-zero means we're in Shutdown)
	nextProtoOnce     sync.Once // guards setupHTTP2_* init
	nextProtoErr      error     // result of http2.ConfigureServer if used

	mu         sync.Mutex
	listeners  map[net.Listener]struct{}
	activeConn map[*conn]struct{}
	doneChan   chan struct{}
	onShutdown []func()
}
```

### Server ListenAndServe

```go
func (srv *Server) ListenAndServe() error {
	addr := srv.Addr
	if addr == "" {
		addr = ":http"
	}
	ln, err := net.Listen("tcp", addr)
	if err != nil {
		return err
	}
	return srv.Serve(tcpKeepAliveListener{ln.(*net.TCPListener)})
}
```

### Server Serve

```go
func (srv *Server) Serve(l net.Listener) error {
	defer l.Close()
	if fn := testHookServerServe; fn != nil {
		fn(srv, l)
	}
	var tempDelay time.Duration // how long to sleep on accept failure

	if err := srv.setupHTTP2_Serve(); err != nil {
		return err
	}

	srv.trackListener(l, true)
	defer srv.trackListener(l, false)

	baseCtx := context.Background() // base is always background, per Issue 16220
	ctx := context.WithValue(baseCtx, ServerContextKey, srv)
	for {
		rw, e := l.Accept()
		if e != nil {
			select {
			case <-srv.getDoneChan():
				return ErrServerClosed
			default:
			}
			if ne, ok := e.(net.Error); ok && ne.Temporary() {
				if tempDelay == 0 {
					tempDelay = 5 * time.Millisecond
				} else {
					tempDelay *= 2
				}
				if max := 1 * time.Second; tempDelay > max {
					tempDelay = max
				}
				srv.logf("http: Accept error: %v; retrying in %v", e, tempDelay)
				time.Sleep(tempDelay)
				continue
			}
			return e
		}
		tempDelay = 0
		c := srv.newConn(rw)
		c.setState(c.rwc, StateNew) // before Serve can return
		go c.serve(ctx)
	}
}
```

### testHookServerServe

```go
var testHookServerServe func(*Server, net.Listener) // used if non-nil
```

### setupHTTP2_Serve

```go
func (srv *Server) setupHTTP2_Serve() error {
	srv.nextProtoOnce.Do(srv.onceSetNextProtoDefaults_Serve)
	return srv.nextProtoErr
}
```

### onceSetNextProtoDefaults_Serve

```go
func (srv *Server) onceSetNextProtoDefaults_Serve() {
	if srv.shouldConfigureHTTP2ForServe() {
		srv.onceSetNextProtoDefaults()
	}
}
```

### shouldConfigureHTTP2ForServe

```go
func (srv *Server) shouldConfigureHTTP2ForServe() bool {
	if srv.TLSConfig == nil {
		// Compatibility with Go 1.6:
		return true
	}
	return strSliceContains(srv.TLSConfig.NextProtos, http2NextProtoTLS)
}
```

### strSliceContains

```go
func strSliceContains(ss []string, s string) bool {
	for _, v := range ss {
		if v == s {
			return true
		}
	}
	return false
}
```

### nextProtoOnce.Do

```go
nextProtoOnce     sync.Once // guards setupHTTP2_* init
```

```go
type Once struct {
	m    Mutex
	done uint32
}

func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 1 {
		return
	}
	// Slow-path.
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```

### atomic.LoadUint32

```go
func LoadUint32(addr *uint32) (val uint32)
```

### trackListener

```go
func (s *Server) trackListener(ln net.Listener, add bool) {
	s.mu.Lock()
	defer s.mu.Unlock()
	if s.listeners == nil {
		s.listeners = make(map[net.Listener]struct{})
	}
	if add {
		// If the *Server is being reused after a previous
		// Close or Shutdown, reset its doneChan:
		if len(s.listeners) == 0 && len(s.activeConn) == 0 {
			s.doneChan = nil
		}
		s.listeners[ln] = struct{}{}
	} else {
		delete(s.listeners, ln)
	}
}
```

### context.Background()

```go
func Background() Context {
	return background
}

var (
	background = new(emptyCtx)
	todo       = new(emptyCtx)
)

type emptyCtx int
```

### context.WithValue

```go
func WithValue(parent Context, key, val interface{}) Context {
	if key == nil {
		panic("nil key")
	}
	if !reflect.TypeOf(key).Comparable() {
		panic("key is not comparable")
	}
	return &valueCtx{parent, key, val}
}

type valueCtx struct {
	Context
	key, val interface{}
}
```

### Context

```go
type Context interface {
	Deadline() (deadline time.Time, ok bool)
	Done() <-chan struct{}
	Err() error
	Value(key interface{}) interface{}
}
```

### l.Accept

```go
func (l *TCPListener) Accept() (Conn, error) {
	if !l.ok() {
		return nil, syscall.EINVAL
	}
	c, err := l.accept()
	if err != nil {
		return nil, &OpError{Op: "accept", Net: l.fd.net, Source: nil, Addr: l.fd.laddr, Err: err}
	}
	return c, nil
}
```
+ https://golang.org/src/net/tcpsock_posix.go?h=accept#L135

```go
func (ln *TCPListener) accept() (*TCPConn, error) {
	fd, err := ln.fd.accept()
	if err != nil {
		return nil, err
	}
	return newTCPConn(fd), nil
}
```

+ https://golang.org/src/net/tcpsock.go?h=newTCPConn#L193 

```go
func newTCPConn(fd *netFD) *TCPConn {
	c := &TCPConn{conn{fd}}
	setNoDelay(c.fd, true)
	return c
}
```

```go
func (c *TCPConn) SetNoDelay(noDelay bool) error {
	if !c.ok() {
		return syscall.EINVAL
	}
	if err := setNoDelay(c.fd, noDelay); err != nil {
		return &OpError{Op: "set", Net: c.fd.net, Source: c.fd.laddr, Addr: c.fd.raddr, Err: err}
	}
	return nil
}
```
