## Go net/http CanonicalHeaderKey

https://golang.org/pkg/net/http/#CanonicalHeaderKey

+ CanonicalHeaderKey returns the normal form of the header key
+ All characters following the first letter and hyphen are converted to uppercase

### CanonicalHeaderKey

```go
func CanonicalHeaderKey(s string) string { return textproto.CanonicalMIMEHeaderKey(s) }
```

### textproto.CanonicalMIMEHeaderKey

+ https://golang.org/pkg/net/textproto/#CanonicalMIMEHeaderKey

```go
func CanonicalMIMEHeaderKey(s string) string {
	// Quick check for canonical encoding.
	upper := true
	for i := 0; i < len(s); i++ {
		c := s[i]
		if !validHeaderFieldByte(c) {
			return s
		}
		if upper && 'a' <= c && c <= 'z' {
			return canonicalMIMEHeaderKey([]byte(s))
		}
		if !upper && 'A' <= c && c <= 'Z' {
			return canonicalMIMEHeaderKey([]byte(s))
		}
		upper = c == '-'
	}
	return s
}
```

### textproto.validHeaderFieldByte

+ filed name: RFC 7230
+ https://tools.ietf.org/html/rfc7230

```go
// validHeaderFieldByte reports whether b is a valid byte in a header
// field name. RFC 7230 says:
//   header-field   = field-name ":" OWS field-value OWS
//   field-name     = token
//   tchar = "!" / "#" / "$" / "%" / "&" / "'" / "*" / "+" / "-" / "." /
//           "^" / "_" / "`" / "|" / "~" / DIGIT / ALPHA
//   token = 1*tchar
func validHeaderFieldByte(b byte) bool {
	return int(b) < len(isTokenTable) && isTokenTable[b]
}
```

### textproto.isTokenTable

+ It is probably checking if it is ascii
+ copy net/http/lex.go
+ https://httpwg.github.io/specs/rfc7230.html#rule.token.separators

```go
var isTokenTable = [127]bool{
	'!':  true,
	'#':  true,
	'$':  true,
	'%':  true,
	'&':  true,
	'\'': true,
	'*':  true,
	'+':  true,
	'-':  true,
	'.':  true,
	'0':  true,
	'1':  true,
	'2':  true,
	'3':  true,
	'4':  true,
	'5':  true,
	'6':  true,
	'7':  true,
	'8':  true,
	'9':  true,
	'A':  true,
	'B':  true,
	'C':  true,
	'D':  true,
	'E':  true,
	'F':  true,
	'G':  true,
	'H':  true,
	'I':  true,
	'J':  true,
	'K':  true,
	'L':  true,
	'M':  true,
	'N':  true,
	'O':  true,
	'P':  true,
	'Q':  true,
	'R':  true,
	'S':  true,
	'T':  true,
	'U':  true,
	'W':  true,
	'V':  true,
	'X':  true,
	'Y':  true,
	'Z':  true,
	'^':  true,
	'_':  true,
	'`':  true,
	'a':  true,
	'b':  true,
	'c':  true,
	'd':  true,
	'e':  true,
	'f':  true,
	'g':  true,
	'h':  true,
	'i':  true,
	'j':  true,
	'k':  true,
	'l':  true,
	'm':  true,
	'n':  true,
	'o':  true,
	'p':  true,
	'q':  true,
	'r':  true,
	's':  true,
	't':  true,
	'u':  true,
	'v':  true,
	'w':  true,
	'x':  true,
	'y':  true,
	'z':  true,
	'|':  true,
	'~':  true,
}
```

### textproto.canonicalMIMEHeaderKey

+ Before returning a string, you can change the specified byte slice.

```go
func canonicalMIMEHeaderKey(a []byte) string {
	// See if a looks like a header key. If not, return it unchanged.
	for _, c := range a {
		if validHeaderFieldByte(c) {
			continue
		}
		// Don't canonicalize.
		return string(a)
	}

	upper := true
	for i, c := range a {
		// Canonicalize: first letter upper case
		// and upper case after each dash.
		// (Host, User-Agent, If-Modified-Since).
		// MIME headers are ASCII only, so no Unicode issues.
		if upper && 'a' <= c && c <= 'z' {
			c -= toLower
		} else if !upper && 'A' <= c && c <= 'Z' {
			c += toLower
		}
		a[i] = c
		upper = c == '-' // for next time
	}
	// The compiler recognizes m[string(byteSlice)] as a special
	// case, so a copy of a's bytes into a new string does not
	// happen in this map lookup:
	if v := commonHeader[string(a)]; v != "" {
		return v
	}
	return string(a)
}
```
