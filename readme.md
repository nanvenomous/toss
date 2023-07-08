# (o)ptic

A generic web extension to [net/http](https://pkg.go.dev/net/http)

Optic helps you call backend functions from your frontend by sending a regular go struct and recieving a struct back

It is especially useful when making requests to a go service from a go client (WASM app, cli, tui ...)

## Install
```bash
go get github.com/nanvenomous/optic
```
Then import with
```go
import (
	"github.com/nanvenomous/optic"
)
```

## Quick example

Define the entities
```go
type Division struct {
	Top    int
	Bottom int
}
type Solution struct {
	Answer int
}
```

Setup the service route
```go
func Divide(recieved *Division, r *http.Request) (*Solution, *optic.Exception) {
	if recieved.Bottom == 0 { // return an error
		return nil, &optic.Exception{Code: http.StatusUnprocessableEntity, Message: "Impossible to divide by Zero"}
	}
	return &Solution{Answer: recieved.Top / recieved.Bottom}, nil
}

func main() {
	var (
		err error
		mux *http.ServeMux
	)
	mux = http.NewServeMux()
	optic.SetupService(PORT, OPTIC_ROUTE, mux)

	// An optic mirror simply recieves information and sends information back
	optic.Mirror(Divide) // by default optic will use function name as route

    err = optic.Serve() // run the service
    if err != nil {
        panic(err)
    }
}
```

Setup the client and make a request
```go
func main {
	optic.SetupClient(HOST, PORT, OPTIC_ROUTE, false)
	// Make requests
	var (
		err error            // internal error
		exn *optic.Exception // service exception
		sln Solution         // output
	)
	//                                  send                          receive
	exn, err = optic.Glance("/Divide/", &Division{Top: 4, Bottom: 2}, &sln)
	fmt.Println(err, exn)   // <nil> <nil>
	fmt.Println(sln.Answer) // 2
}
```

## net/http compatibility
Optic is drop in compatible with [net/http](https://pkg.go.dev/net/http)

Give optic a `*http.ServerMux` & a special route where it will handle all you functions

Then do whatever else you want with that mux
```go
func main() {
	var (
		err error
		mux *http.ServeMux
	)
	mux = http.NewServeMux()
	optic.SetupService(PORT, OPTIC_ROUTE, mux)

    // Add other routes not handled by optic, as you would with any net/http service
    mux.HandleFunc("/health-check/", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
    })

    // optic can register middleware for you
    optic.RegisterMiddleware(exampleMiddleware)
    // or you can do it yourself
    var (
        handler http.Handler
    )
    handler = exampleMiddleware(mux)
}

func exampleMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// add some middleware (like CORS for example)
		w.Header().Set("Access-Control-Allow-Origin", "*")
		next.ServeHTTP(w, r)
	})
}
```

## Examples
For the full example in code see [./examples/main.go](https://github.com/nanvenomous/optic/blob/mainline/example/main.go) 

Run the example like so:
![run example](.rsrc/run-example.gif)

## Simplicity
https://github.com/nanvenomous/optic/blob/b8a94eb20e2ae535252c56ea8d283f2b794cffd4/go.mod#L1-L3


## Inspiration
I drew some inspiration from [leptos server functions](https://leptos-rs.github.io/leptos/server/25_server_functions.html)
