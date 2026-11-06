# sane

[![Go Reference](https://pkg.go.dev/badge/github.com/Ghardo/sane.svg)](https://pkg.go.dev/github.com/Ghardo/sane)

Go bindings for the [SANE](http://www.sane-project.org) scanner API.

This is a fork of [tjgq/sane](https://github.com/tjgq/sane) by
Tiago Quelhas. It adds Go module support and fixes for current Go versions.

## Requirements

- Go 1.27 or newer
- cgo and a C compiler
- the SANE development files (`libsane`)

| Platform        | Package                                   |
| --------------- | ----------------------------------------- |
| Debian / Ubuntu | `apt install libsane-dev`                 |
| Fedora          | `dnf install sane-backends-devel`         |
| Arch Linux      | `pacman -S sane`                          |
| macOS           | `brew install sane-backends`              |

## Installation

```sh
go get github.com/Ghardo/sane
```

## Usage

```go
package main

import (
	"image/png"
	"log"
	"os"

	"github.com/Ghardo/sane"
)

func main() {
	if err := sane.Init(); err != nil {
		log.Fatal(err)
	}
	defer sane.Exit()

	c, err := sane.Open("") // "" opens the first available device
	if err != nil {
		log.Fatal(err)
	}
	defer c.Close()

	img, err := c.ReadImage()
	if err != nil {
		log.Fatal(err)
	}

	f, err := os.Create("scan.png")
	if err != nil {
		log.Fatal(err)
	}
	defer f.Close()
	if err := png.Encode(f, img); err != nil {
		log.Fatal(err)
	}
}
```

Options are listed with `c.Options()` and changed with `c.SetOption(name, value)`.
The value must match the option's type, which depends on the device: the same
option (e.g. `resolution`) may be an `int` on one scanner and a `float64` on
another. Check `Option.Type` before setting a value.

The package documentation is on
[pkg.go.dev](https://pkg.go.dev/github.com/Ghardo/sane). Details about the
underlying API are in the
[SANE standard](http://www.sane-project.org/html/).

## Example program

The `example` directory contains a small command-line tool that mostly mimics
the `scanimage` utility shipped with SANE:

```sh
go run ./example list                              # list devices
go run ./example show <device>                     # show device options
go run ./example scan <device> out.png [OPTIONS]   # scan to file
```

The output format is chosen by the file extension (`.png`, `.jpg`, `.tif`).

## Testing

The test suite runs against the SANE `test` backend, so no real scanner is
needed. The backend must be enabled in `/etc/sane.d/dll.conf`, which is the
default on most distributions.

```sh
go test -race ./...
```

To run the tests in a container:

```sh
docker run --rm -v "$PWD:/src" -w /src golang:1.27 \
  sh -c 'apt-get update -qq && apt-get install -y -qq libsane-dev && go test -race ./...'
```

## Limitations

All SANE functionality is supported except authentication callbacks.
Testing with real-world devices is always welcome.

## License

BSD 3-clause, see [LICENSE](LICENSE).
