# envconfig

[![Build Status](https://travis-ci.org/axiomzen/envconfig.png)](https://travis-ci.org/axiomzen/envconfig)
[![Coverage Status](https://coveralls.io/repos/github/axiomzen/envconfig/badge.svg?branch=master)](https://coveralls.io/github/axiomzen/envconfig?branch=master)

```Go
import "github.com/axiomzen/envconfig"
```

## Documentation

See [godoc](http://godoc.org/github.com/axiomzen/envconfig)

## Usage

Set some environment variables:

```Bash
export MYAPP_DEBUG=false
export MYAPP_PORT=8080
export MYAPP_USER=Kelsey
export MYAPP_RATE="0.5"
export MYAPP_TIMEOUT="3m"
export MYAPP_USERS="rob,ken,robert"
export MYAPP_COLORCODES="red:1,green:2,blue:3"
```

Write some code:

```Go
package main

import (
    "fmt"
    "log"
    "time"

    "github.com/axiomzen/envconfig"
)

type Specification struct {
    Debug       bool
    Port        int
    User        string
    Users       []string
    Rate        float32
    Timeout     time.Duration
    ColorCodes  map[string]int
}

func main() {
    var s Specification
    err := envconfig.Process("myapp", &s)
    if err != nil {
        log.Fatal(err.Error())
    }
    format := "Debug: %v\nPort: %d\nUser: %s\nRate: %f\nTimeout: %s\n"
    _, err = fmt.Printf(format, s.Debug, s.Port, s.User, s.Rate)
    if err != nil {
        log.Fatal(err.Error())
    }

    fmt.Println("Users:")
    for _, u := range s.Users {
        fmt.Printf("  %s\n", u)
    }
    
     fmt.Println("Color codes:")
        for k, v := range s.ColorCodes {
            fmt.Printf("  %s: %d\n", k, v)
        }
}
```

Results:

```Bash
Debug: false
Port: 8080
User: Kelsey
Rate: 0.500000
Timeout: 3m0s
Users:
  rob
  ken
  robert
Color codes:
  red: 1
  green: 2
  blue: 3  
```

## Struct Tag Support

Envconfig supports the use of struct tags to specify alternate, default, and required
environment variables.

For example, consider the following struct:

```Go
type Specification struct {
    MultiWordVar string `envconfig:"multi_word_var"`
    DefaultVar   string `default:"foobar"`
    RequiredVar  string `required:"true"`
    IgnoredVar   string `ignored:"true"`
}
```

Envconfig will process value for `MultiWordVar` by populating it with the
value for `MYAPP_MULTI_WORD_VAR`.

```Bash
export MYAPP_MULTI_WORD_VAR="this will be the value"

# export MYAPP_MULTIWORDVAR="and this will not"
```

If envconfig can't find an environment variable value for `MYAPP_DEFAULTVAR`,
it will populate it with "foobar" as a default value.

If envconfig can't find an environment variable value for `MYAPP_REQUIREDVAR`,
it will return an error when asked to process the struct.

If envconfig can't find an environment variable in the form `PREFIX_MYVAR`, and there
is a struct tag defined, it will try to populate your variable with an environment
variable that directly matches the envconfig tag in your struct definition:

```shell
export SERVICE_HOST=127.0.0.1
export MYAPP_DEBUG=true
```
```Go
type Specification struct {
    ServiceHost string `envconfig:"SERVICE_HOST"`
    Debug       bool
}
```

Envconfig won't process a field with the "ignored" tag set to "true", even if a corresponding
environment variable is set.

## Supported Struct Field Types

envconfig supports supports these struct field types:

  * string
  * int8, int16, int32, int64
  * bool
  * float32, float64
  * slices of any supported type
  * maps (keys and values of any supported type)

Embedded structs using these fields are also supported.

## Custom Decoders

Any field whose type (or pointer-to-type) implements `envconfig.Decoder` can
control its own deserialization:

```Bash
export DNS_SERVER=8.8.8.8
```

```Go
type IPDecoder net.IP

func (ipd *IPDecoder) Decode(value string) error {
    *ipd = IPDecoder(net.ParseIP(value))
    return nil
}

type DNSConfig struct {
    Address IPDecoder `envconfig:"DNS_SERVER"`
}
```
