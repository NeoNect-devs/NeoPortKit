# NeoPortKit

NeoPortKit is a zero-dependency Go library for discovering local listening network ports.

It provides a synchronous, OS-agnostic public API backed by a Linux-specific `procfs` implementation. NeoPortKit is intended to be used as a building block for a future server management panel and server monitoring system.

## V1 Scope and Supported Platforms

The v1.0.0 release is strictly scoped to **Linux local listening-port discovery**.

Calling the discovery API on unsupported operating systems (such as Windows or macOS) will safely return `ErrNotSupported`.

Supported protocols:

* TCP IPv4 (`tcp4`)
* TCP IPv6 (`tcp6`)
* UDP IPv4 (`udp4`)
* UDP IPv6 (`udp6`)

## Installation

```bash
go get github.com/NeoNect-devs/NeoPortKit@v1.0.0
```

## Basic Usage

```go
package main

import (
	"fmt"

	"github.com/NeoNect-devs/NeoPortKit"
)

func main() {
	listeners, err := neoportkit.Discover()
	if err != nil {
		if err == neoportkit.ErrNotSupported {
			fmt.Println("NeoPortKit is not supported on this OS.")
			return
		}
		panic(err)
	}

	for _, l := range listeners {
		fmt.Printf("Protocol: %s, IP: %s, Port: %d, PID: %d\n",
			l.Protocol, l.LocalIP, l.LocalPort, l.ProcessID)
	}
}
```

## Public API

NeoPortKit's public surface is intentionally minimal.

### `Listener`

The `Listener` struct represents an active local listening network port:

* `LocalIP` (`netip.Addr`): The IP address the socket is bound to.
* `LocalPort` (`uint16`): The port number.
* `Protocol` (`string`): The transport protocol. Possible values are exactly: `"tcp4"`, `"tcp6"`, `"udp4"`, `"udp6"`.
* `ProcessID` (`int`): The OS process ID that owns the listening socket.

### Best-Effort PID Resolution and `UnresolvedPID`

Process ID resolution is best-effort. NeoPortKit iterates through `/proc/[pid]/fd` to correlate socket inodes to running processes.

Because `/proc` directory permissions vary based on the user executing the program, PID resolution may be prevented by the OS.

When a process ID cannot be resolved, NeoPortKit returns the sentinel value `neoportkit.UnresolvedPID` (`0`). It will not return an error or panic.

## Limitations

* **Hardcoded `/proc` paths:** NeoPortKit relies on the standard `/proc` mounting location.
* **Shared Sockets:** If a listening socket is shared by multiple processes, the `Listener` will report one owning `ProcessID`.

## Development and Validation

```bash
go fmt ./...
go vet ./...
go build ./...
go test ./...
go test -race ./...
```

To run benchmarks:

```bash
go test -bench=. -benchmem ./...
```

## License

NeoPortKit is licensed under the Apache License 2.0. See [LICENSE](LICENSE) for details.

## Origin

NeoPortKit is based on the original [41NI/NetWard](https://github.com/41NI/NetWard) project.
