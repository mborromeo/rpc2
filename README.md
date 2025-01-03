rpc2
====

[![GoDoc](https://pkg.go.dev/badge/github.com/mborromeo/rpc2)](https://pkg.go.dev/github.com/mborromeo/rpc2)

> This fork of [cenkalti/rpc2](https://github.com/cenkalti/rpc2) includes a change to the interface by adding a method to retrieve the underlying connection. Since this change would break backward compatibility, it was not included in the original project. To address this need, I have decided to maintain this fork.
> 
> If you require this additional functionality ([underlying connection getter](https://pkg.go.dev/github.com/mborromeo/rpc2?#Client.Conn)), you can use this fork by updating your go.mod file to reference this repository instead.
> 
> This README has been updated to reflect the usage documentation for this fork, but all original credit goes to [cenkalti/rpc2](https://github.com/cenkalti/rpc2)

rpc2 is a fork of net/rpc package in the standard library.
The main goal is to add bi-directional support to calls.
That means server can call the methods of client.
This is not possible with net/rpc package.
In order to do this it adds a `*Client` argument to method signatures.

Install
--------

    go get github.com/mborromeo/rpc2

Example server
---------------

```go
package main

import (
	"fmt"
	"net"

	"github.com/mborromeo/rpc2"
)

type Args struct{ A, B int }
type Reply int

func main() {
	srv := rpc2.NewServer()
	srv.Handle("add", func(client *rpc2.Client, args *Args, reply *Reply) error {

		// Reversed call (server to client)
		var rep Reply
		client.Call("mult", Args{2, 3}, &rep)
		fmt.Println("mult result:", rep)

		*reply = Reply(args.A + args.B)
		return nil
	})

	lis, _ := net.Listen("tcp", "127.0.0.1:5000")
	srv.Accept(lis)
}
```

Example Client
---------------

```go
package main

import (
	"fmt"
	"net"

	"github.com/mborromeo/rpc2"
)

type Args struct{ A, B int }
type Reply int

func main() {
	conn, _ := net.Dial("tcp", "127.0.0.1:5000")

	clt := rpc2.NewClient(conn)
	clt.Handle("mult", func(client *rpc2.Client, args *Args, reply *Reply) error {
		*reply = Reply(args.A * args.B)
		return nil
	})
	go clt.Run()

	var rep Reply
	clt.Call("add", Args{1, 2}, &rep)
	fmt.Println("add result:", rep)
}
```
