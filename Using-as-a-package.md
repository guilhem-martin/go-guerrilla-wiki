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

`d.Start()` does not block after the server has been started, so make sure that you keep your program busy.

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

Here we've set the Daemon's `Config` field with an instance of `AppConfig` type with our own setting for the `LogFile` field. We had to import `github.com/flashmob/go-guerrilla/log` to get the `log.OutputOff`. `LogFile` could also be a string to a path, or set it with `log.log.OutputStderr.String()`, or `log.OutputStdout.String()`

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
Finally the `BackendConfig` configures how your email transaction will be processed.
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

Let's continue for some more examples.

###  Backend Configuration

Here we use `backends.BackendConfig` to configure the default _Gateway_ backend.
The _Gateway_ backend is composed of multiple components, therefore it does not define any static configuration fields. Instead, it uses a map to configure the settings.

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


Notice that we instantiated a new `bcfg` variable and initialized with a literal, just like initializing a map. 
The keys of the map correspond the jason struct stags, these struct tags are defined in individual `Processor` components. (The above components are defined in the backend package, go file names prefixed with 'p_'.

See the [Backends Documentation](https://github.com/flashmob/go-guerrilla/wiki/About-Backends:-introduction,-configuring-and-extending) page for more details

### Loading config from Json

Say you have a json configuration file like so:

```json

{
    "log_file" : "./tests/testlog",
    "log_level" : "debug",
    "pid_file" : "tests/go-guerrilla.pid",
    "allowed_hosts": ["spam4.me","grr.la"],
    "backend_config" :
        {
            "log_received_mails" : true,
            "process_stack": "HeadersParser|Header|Hasher|Debugger",
            "save_workers_size":  3
        },
    "servers" : [
        {
            "is_enabled" : true,
            "host_name":"mail.guerrillamail.com",
            "max_size": 100017,
            "private_key_file":"config_test.go",
            "public_key_file":"config_test.go",
            "timeout":160,
            "listen_interface":"127.0.0.1:2526",
            "start_tls_on":false,
            "tls_always_on":false,
            "max_clients": 2
        }
    ]
}

```

Then you can load it in like this:

```go
d := Daemon{}
_, err = d.LoadConfig("guerrillad.conf.json")
if err != nil {
	fmt.Println("ReadConfig error", err)
		
}

err = d.Start()
if err != nil {
	fmt.Println("server error", err)
}
// .. 

```

There's also a sister `Daemon.SetConfig` function which allows you to pass the AppConfig
directly, without unmarshalling from Json.

### Config hot-reloading

Use `d.ReloadConfigFile` to re-load the config file after the daemon has started

Almost every config can be hot-reloaded. This means changing things without re-starting the daemon, 
kind of like changing a tyre while the car is still moving! 🚗 🚗 🚗
It can change the TLS configuration, add/remove/start/stop servers, log output destinations & levels, 
pid file name, allowed-hosts, connection timeout, backend configuration. The only limitation right now
is that the max-clients cannot be resized since our current pool implementation doesn't support it (yet).

```go

d := Daemon{}
_, err = d.LoadConfig("guerrillad.conf.json")
if err != nil {
	fmt.Println("ReadConfig error", err)
		
}

err = d.Start()
if err != nil {
	fmt.Println("server error", err)
}

// ... somewhere else

d.ReloadConfigFile("guerrillad.conf.json")


```

### Log reopening

To re-open all log files, use:

`d.ReopenLogs()`

Why would you want to reopen log files? A common way to rotate logs is to rename
the file, and then tell the daemon to close the file descriptor and open a new one.
This way no log entries are lost when the file is rotated. 

Here is how you can [setup log rotation](https://github.com/flashmob/go-guerrilla/wiki/Automatic-log-file-management-with-logrotate) using logrotate(8)

### Custom processor

todo

### Graceful shutdown

In all examples above, we did not shutdown the server, assuming that your program will keep
busy by doing something else. When it's time to close, we do not want to abruptly close and leave any transactions halfway, we want to do a graceful shutdown and finish off any emails, close files/connections
and then quit. We do this by calling:

`s.Shutdown()`

This will block until everything has been shuttered. It may take a while if your server is busy.
The way it works is, all connections are given very low timeouts while new connections are not accepted.
 If any clients are in the `command` state, the server will respond to all client's commands with 
`421 Server is shutting down. Please try again later. Sayonara!`, then close. 
If the client is in the DATA state, the transaction will not be interrupted and will try to 
complete with a low timeout, then close.
Once all connections close, the backend gets shuttered and then the Shutdown function returns.
n.b Why Sayonara? The section of that code was written in Japan ;-)

### Pub/Sub 