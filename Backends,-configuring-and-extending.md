# Backends 

This documents the backends package.

Note: Work in progress. Not merged yet.

## Introduction

### What does a backend do?

The main job of a go-guerrilla backend is to save email. Think of it as the middleware.

The default go-guerrilla backend is called a "_Gateway_". The Gateway manages a set of workers which run in their own goroutines. These workers can be started, shutdown, and new work can be distributed to the workers. The gateway provides a common interface to these workers.

The number of workers to start can be controlled by changing the backend's `save_workers_size` config option.

### Gateway internals

The Gateway receives new email envelopes from the server via the Process function. These envelopes are passed via the gateway's **conveyor** channel and picked up by an available _Worker_. The envelope is a value of `github.com/flashmob/go-guerrilla/envelope.Envelope` it's passed as a pointer. Users of the package don't need to be concerned with the conveyor channel details, only use the exposed Process function provided by the gateway, and it takes care of the rest.

The Gateway can perform different tasks, not only save email. Another function it provides is `ValidateRcpt` which validates the envelope's last recipient pushed. It still uses the conveyor channel to distribute the work, and the `task` field in the `workerMsg` 

Internally, the task is selected using the SelectTask type. So far, there are two types of tasks: `TaskSaveMail` to save email, `TaskValidateRcpt` to validate the recipient of an envelope. 

### What are Workers?

Each _Worker_ can be composed of individual _Processors_ and each Processor is called sequentially to process each envelope. Think of it as production line in a factory. Each worker works on one envelope which they pick up from a conveyor belt (in this case, a channel). Workers must do a number of steps to complete their work before sending their results back. The steps (Processors) that the worker must do can be controlled by changing the backend's `save_process` config option.

### Workers internals

Each worker is structured using a Decorator pattern. See footnote [1].

The decorator pattern stacks each Processor on a call-stack, making a single Processor out of many different Processors. Each processor in the stack must either chain the call to the next Processor by passing the envelope, or return the result back to the caller. You do not need to be concerned with these details, however, it may be worth to checkout the footnote and also decorate.go (where the decorator stack is built) and processor.go (where the interfaces and types are defined). There's also a `DefaultProcessor` which is always called as the final processor, if everything went OK.

Here is the interface of a processor
```go
import "github.com/flashmob/go-guerrilla/mail"

// Our processor is defined as something that processes the envelope and returns a result and error
type Processor interface {
	Process(*mail.Envelope, SelectTask) (Result, error)
}
```
In go-guerrilla's code, Go source files that define a Processor are prefixed with 'p_'

There are a few other details about Processors, if you're interested, please see
"Extending" section.

## Configuration

The `"backend_config"` property holds the configuration for the backend, including configuration values for all the Processors.

The Gateway requires the following settings: 

- `save_process` - this a processor for saving email, composed by chaining the smaller processors together. 
Each processor name is separated by a `|` character and is executed left-to-right. It is called when we receive mail data from a client.
- `validate_process` - (optional) similar to `save_process`, but for validating recipients. It is called when we receive recipients from a client.
- `save_workers_size` - how many workers to spawn

Then add any other settings that each processor may require.
```json
    "backend_config" :
       {
          "save_process": "HeadersParser|Debugger|Hasher|Header|Compressor|Redis|MySql",
          "save_workers_size" : 1,
          "log_received_mails" : true,
          "mysql_db":"gmail_mail",
          "mysql_host":"127.0.0.1:3306",
          "mysql_pass":"ok",
          "mysql_user":"root",
          "mail_table":"new_mail",
          "redis_interface" : "127.0.0.1:6379",
          "redis_expire_seconds" : 7200,
          "primary_mail_host":"sharklasers.com"
       }
```
To recap, the **save_process** configures how the worker will process each envelope. 
Each processor is separated using a "|" (pipe) character, and execution is from left to right.
So the one above will parse the MIME headers, print some debug info, generate some hashes, add a delivery header, compress the email, save the body to redis, and finally save some info to MySQL.

None of the above processors validate recipients, but if you have a processor that does
recipient validation, you may add it by setting the `validate_process` config in a similar fashion.
For example, [MailDiranasaurus](https://github.com/flashmob/maildiranasaurus) a MailDir MTA, demonstrates
the use of a processor that can both validate recipients and save mail/ 

### Gateway timeouts

As detailed above, the gateway distributes the envelope to process via the **conveyor** channel.
The envelope is submitted to the gateway via the `Process` function. If the email is not
processed in time, it will return with an error. Currently the defaul is to 30 seconds.

The default can be customized via Backend config:

- `gw_save_timeout` - is the duration before timeout when saving an email. String, eg "10s" 
- `gw_val_rcpt_timeout` - is the duration before timeout when validating a recipient, string, eg "1s"

For formatting options for the duration string, see Go's [time.ParseDuration](https://golang.org/pkg/time/#ParseDuration)

## Extending

The decorator pattern makes it easy to create your own Processors. To get started, create a new `.go` file, then import
```go
import (
   "github.com/flashmob/go-guerrilla/backends"
   "github.com/flashmob/go-guerrilla/mail"
)
```
From there, if your processor needs a configuration, define your own configuration struct. The struct can only have string, float or numeric fields. Each field must be public and be annotated with a [struct tag](http://stackoverflow.com/questions/10858787/what-are-the-uses-for-tags-in-go) to map it to the json file. eg.
```go
type myFooConfig struct {
   SomeOption string `json:"maildir_path"`
}
```
From there, declare your processor as a decorator using the following pattern:

```go
// The MyFoo decorator [enter what it does]
var MyFooProcessor = func() backends.Decorator {
	return func(p backends.Processor) backends.Processor {
		return backends.ProcessWith(
			func(e *mail.Envelope, task backends.SelectTask) (backends.Result, error) {
				if task == backends.TaskValidateRcpt {

					// if you want your processor to validate recipents,
					// validate recipient by checking
					// the last item added to `e.RcptTo` slice
					// if error, then return something like this:
					/* return backends.NewResult(
					   response.Canned.FailNoSenderDataCmd),
					   backends.NoSuchUser
					*/
					// if no error:
					return p.Process(e, task)
				} else if task == backends.TaskSaveMail {

					// if you want your processor to do some processing after
					// receiving the email, continue here.
					// if want to stop processing, return
					// errors.New("Something went wrong")
					// return backends.NewBackendResult(fmt.Sprintf("554 Error: %s", err)), err
					// call the next processor in the chain
					return p.Process(e, task)
				}
				return p.Process(e, task)
			},
		)
	}
}
```
Next, assuming that you're [using go-guerrilla as a package](), do this before you call s.Start():
```go
d := guerrilla.Daemon{}
d.AddProcessor("MyFoo", MyFooProcessor)

```
There are additional features which allow you to do some things at initialization and shutdown.

### Processor initialization

To load in the config into your processor, you will need to declare and add your own initializer function.
Initializer may do other things, such as open database connections, etc.

The interface looks like this (complete example further down):
```go
    type ProcessorInitializer interface {
	    Initialize(backendConfig BackendConfig) error
    }
```
There's also convenience type defined so that you can create your initializer as a closure or an anonymous function. 
```go
    type InitializeWith func(backendConfig BackendConfig) error
```
Once you've declared your initializer, use the `backends.Service.AddInitializer` function to register it.

So putting it together, your Processor will look something like this

```go

var MyFooProcessor = func() backends.Decorator {
	config := &myFooConfig{}
	// our initFunc will load the config.
	initFunc := backends.InitializeWith(func(backendConfig backends.BackendConfig) error {
		configType := backends.BaseConfig(&myFooConfig{})
		bcfg, err := backends.Service.ExtractConfig(backendConfig, configType)
		if err != nil {
			return err
		}
		config := bcfg.(*myFooConfig)
		return nil
	})
	// register our initializer
	backends.Svc.AddInitializer(initFunc)
	return func(p backends.Processor) backends.Processor {
		return backends.ProcessWith(
			func(e *mail.Envelope, task backends.SelectTask) (backends.Result, error) {
				if task == backends.TaskValidateRcpt {
					// optionally, validate recipient
					return p.Process(e, task)
				} else if task == backends.TaskSaveMail {
					/*
						do some work here..
						if want to stop processing, return

						 errors.New("Something went wrong")
						 return backends.NewBackendResult(fmt.Sprintf("554 Error: %s", err)), err
					*/

					_ = config // use config somewhere here..

					// call the next processor in the chain
					return p.Process(e, task)
				}
				return p.Process(e, task)
			},
		)
	}
}

```
### Processor shutdown

Following the same pattern of initialization, there is also a sister Shutdown function

The interface looks like this
```go
    type ProcessorShutdowner interface {
        Shutdown() error
    }
```
And the convenience type to help you pass an anonymous/closure function looks like this
```go
    type ShutdownWith func() error
```
To pass your function to go-guerrilla, use backends.Service.AddShutdowner function

eg. just after `backends.Service.AddInitializer(initFunc)` add something like this:
```go
    backends.Svc.AddShutdowner(backends.ShutdownWith(func() error {
        if db != nil {
            return db.Close()
        }
        return nil
    }))
```


***

### Examples

Perhaps it can be best described by giving some examples.

- Example 1: In the api_test.go file in go-guerrilla, about 2/3 down, there is a `funkyLogger` processor
defined for testing. This processor logs a funky message for tasks `backends.TaskValidateRcpt` and `backends.TaskSaveMail`. See the TestSetAddProcessor where the processor is added, configured, and a test email is sent to the server so it will get processed.
- Example 2: [MailDir processor](https://github.com/flashmob/maildir-processor) - This saves emails to a maildir folder. It also validates recipients to make sure that their Maildir folder exists (set via the config)
- Example 3: [FastCGI processor](https://github.com/flashmob/fastcgi-processor) - This forwards the task of processing an email to a Fast CGI backend, such as PHP-FPM. It can save/process emails or validate addresses.
- Example 4: [MailDiranasaurus](https://github.com/flashmob/maildiranasaurus) - This is an example of using go-guerrilla as a package, and also using MailDir and FastCgi processors.

### Share your processors

If you made a processor and want to share it with others? Please send a PR to update our README with a link your repo.

## Footnotes

[1]: Footnote 1. This pattern is described in detail here: https://github.com/smaxwellstewart/golang-decorator-pattern and this video has a man with an awesome moustache giving a good presentation: https://www.youtube.com/watch?v=xyDkyFjzFVc