# NeoPortKit

NeoPortKit is a zero-dependency Go library for discovering local listening network ports.

It provides a synchronous, OS-agnostic public API backed by a Linux-specific `procfs` implementation. NeoPortKit is intended to serve as a foundational primitive for downstream server monitoring, administration, networking, security tooling, and future server management UI panels.

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

Because `/proc` directory permissions vary based on the user executing the program (e.g., a non-root user cannot read root-owned process directories), PID resolution may be prevented by the OS.

When a process ID cannot be resolved due to permissions or other `/proc` limitations, NeoPortKit gracefully falls back to returning the constant sentinel value `neoportkit.UnresolvedPID` (`0`). It will not return an error or panic.

## Intended Use

NeoPortKit is designed to provide local network information for higher-level applications and services, including:

* Server monitoring
* Server administration and management tools
* Backend services
* Security and networking tools
* Future server management and monitoring UI panels

## Limitations

* **Hardcoded `/proc` paths:** NeoPortKit relies on the standard `/proc` mounting location. It currently does not support custom `/proc` roots (e.g., scanning a host filesystem mounted inside a container at `/host/proc`).
* **Shared Sockets:** If a single listening socket is shared by multiple processes (e.g., via `fork`), the `Listener` will report exactly one owning `ProcessID`.

## Development and Validation

The repository maintains standard Go test conventions. To validate modifications locally:

```bash
go fmt ./...
go vet ./...
go build ./...
go test ./...
go test -race ./...
```

To run the internal PID-resolution benchmarks:

```bash
go test -bench=. -benchmem ./...
```

## License

NeoPortKit is licensed under the Apache License 2.0. See [LICENSE](LICENSE) for details.
