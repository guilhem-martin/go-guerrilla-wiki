**Note: Work in progress. Not merged yet.**

## Go-guerrilla API

The following examples start from the basics and progress to more advanced features. 

To get started, import the guerrilla package to your project.

```go
import (
	"github.com/flashmob/go-guerrilla/guerrilla"
)
```

### Starting a server

This will start a server with the default settings, listening on `127.0.0.1:2525`


```go

d := guerrilla.Daemon{}
err := d.Start()

if err == nil {
    fmt.Println("Server Started!")
}
```

`smtp.Start()` does not block after the server has been started, so make sure you keep your program busy doing something else.

The defaults are: 
* Server listening to 127.0.0.1:2525
* use your hostname to determine your which hosts to accept email for
* 100 maximum clients
* 10MB max message size 
* log to Stderror, 
* log level set to "`debug`"
* timeout to 30 sec 
* Backend configured with the following processors: `HeadersParser|Header|Debugger` where it will log the received emails.

### Starting a server - Suppressing log output

Same as above, except here things get more interesting as we start configuring

```go
import (
        "github.com/flashmob/go-guerrilla/guerrilla"
        "github.com/flashmob/go-guerrilla/log"
)

cfg := &AppConfig{LogFile: log.OutputOff.String()}

d := Daemon{Config: cfg}

err := d.Start()
if err != nil {
	fmt.Println(err)
}

```

Here we've set the Daemon's `Config` field with an instance of `AppConfig` type with our own setting for the `LogFile` field. We had to import `github.com/flashmob/go-guerrilla/log` to get the `log.OutputOff`. `LogFile` could also be a string to a path, or set it with `log.log.OutputStderr.String()`, `log.OutputStdout.String()`

### Starting a server - Custom listening interface

The default server listens to `127.0.0.1:2525` - what if want `127.0.0.1:2526` instead?

```go

cfg := &AppConfig{LogFile: log.OutputStdout.String()}

sc := ServerConfig{
	ListenInterface: "127.0.0.1:2526",
	IsEnabled:       true,
}
cfg.Servers = append(cfg.Servers, sc)

d := Daemon{Config: cfg}

err := d.Start()
if err != nil {
	fmt.Printlnl("start error", err)
}

```

Notice here we've used the `ServerConfig` struct to build our server configuration, and then it
was appended to `AppConfig.Servers` field. Notice that we've initialized the ServerConfig 
with two properties: `ListenInterface` and `IsEnabled` - these are the minimal fields for configuring 
a new server. The server will use default values for all unspecified fields.

### What else can be configured?

Here is the `AppConfig` type

```go
// AppConfig is the holder of the configuration of the app
type AppConfig struct {
	// Servers can have one or more items.  
        /// Defaults to 1 server listening on 127.0.0.1:2525
	Servers       []ServerConfig         `json:"servers"`
	// AllowedHosts lists which hosts to accept email for. Defaults to os.Hostname
	AllowedHosts  []string               `json:"allowed_hosts"`
	// PidFile is the path for writing out the process id. No output if empty
	PidFile       string                 `json:"pid_file"`
	// LogFile is where the logs go. Use path to file, or "stderr", "stdout" 
        // or "off". Default "stderr"
	LogFile       string                 `json:"log_file,omitempty"`
	// LogLevel controls the lowest level we log. 
        // "info", "debug", "error", "panic". Default "info"
	LogLevel      string                 `json:"log_level,omitempty"`
	// BackendConfig configures the email envelope processing backend
	BackendConfig backends.BackendConfig `json:"backend_config"`
}
```

Notice that it has [struct tags](http://stackoverflow.com/questions/10858787/what-are-the-uses-for-tags-in-go)
 - this maps each value to a JSON file, we'll show you how to read the 
config from a file later. Notice that `Servers` is a slice, you can have as many servers as you like. 
Finally the `BackendConfig` is the configuration for how your email transaction will be processed.
All servers share the same backend.

Here is the `Servers` struct:

```go
type ServerConfig struct {
	// IsEnabled set to true to start the server, false will ignore it
	IsEnabled       bool   `json:"is_enabled"`
	// Hostname will be used in the server's reply to HELO/EHLO. If TLS enabled
	// make sure that the Hostname matches the cert. Defaults to os.Hostname()
	Hostname        string `json:"host_name"`
	// MaxSize is the maximum size of an email that will be accepted for delivery. 
        // Defaults to 10 Mebibytes
	MaxSize         int64  `json:"max_size"`
	// PrivateKeyFile path to cert private key in PEM format. Will be ignored if blank
	PrivateKeyFile  string `json:"private_key_file"`
	// PublicKeyFile path to cert (public key) chain in PEM format. 
        // Will be ignored if blank
	PublicKeyFile   string `json:"public_key_file"`
	// Timeout specifies the connection timeout in seconds. Defaults to 30
	Timeout         int    `json:"timeout"`
	// Listen interface specified in <ip>:<port> - defaults to 127.0.0.1:2525
	ListenInterface string `json:"listen_interface"`
	// StartTLSOn should we offer STARTTLS command. Cert must be valid. 
        // False by default
	StartTLSOn      bool   `json:"start_tls_on,omitempty"`
	// TLSAlwaysOn run this server as a pure TLS server, i.e. SMTPS
	TLSAlwaysOn     bool   `json:"tls_always_on,omitempty"`
	// MaxClients controls how many maxiumum clients we can handle at once. 
        // Defaults to 100
	MaxClients      int    `json:"max_clients"`
	// LogFile is where the logs go. Use path to file, or "stderr", "stdout" or "off". 
	// defaults to AppConfig.Log file setting 
	LogFile         string `json:"log_file,omitempty"`

	// private fields omitted for brevity
}
```

Lets continue for some more examples.

###  Backend Configuration

Here we use `backends.BackendConfig` to configure the default _Gateway_ backend.
The _Gateway_ backend is composed of multiple components, therefore it does not define static configuration fields. Instead, it uses a map to configure the settings.

```go
cfg := &AppConfig{LogFile: log.OutputStdout.String()}
sc := ServerConfig{
	ListenInterface: "127.0.0.1:2526",
	IsEnabled:       true,
}
cfg.Servers = append(cfg.Servers, sc)
bcfg := backends.BackendConfig{
	"save_workers_size":  3,
	"process_stack":      "HeadersParser|Header|Hasher|Debugger",
	"log_received_mails": true,
        "primary_mail_host" : "example.com",
}
cfg.BackendConfig = bcfg

d := Daemon{Config: cfg}

err := d.Start()

if err != nil {
	fmt.Println("start error", err)
} 

```

### A bit about the backend system. 

A 'backend' is something that implements `guerrilla.Backend` interface. 
You don't have to implement this interface yourself. By default, go-guerrilla will use `backends.BackendGateway` - which we refer to as the _Gateway_ backend. So in the above example, the configuration will be passed to the _Gateway_ backend.

The _Gateway_ is actually quite powerful. Think of it as middleware. It can be composed by chaining individual 
components, which we refer to as _Processors_. In the above example, we chained 
`"HeadersParser|Header|Hasher|Debugger"` which means that we'll start processing with the HeadersParser
processor and finish with the Debugger. You'll need to refer to individual documentation for 
each _Processor_ to see what fields are available for configuration. 

The default `Gateway` has its own configuration too. It takes the following fields:

* `save_workers_size`   a number representing the number of workers to run at the same time
* `process_stack`       A string that configures

The other options, `log_received_mails` is part of the Debugger processor, and `primary_mail_host`
is from the Header processor.


Notice that we instantiated a new `bcfg` variable and initialized with a literal it as if we initialized a map. 
The keys of the map correspond the jason struct stags, these struct tags are defined in individual `Processor` components. (The above components are in the backend package, go file names prefixed with 'p_'.

See the [Backends Documentation](https://github.com/flashmob/go-guerrilla/wiki/About-Backends:-introduction,-configuring-and-extending) page for more details

### Loading config from Json

todo

### Config hot-reloading

todo

### custom processor

todo