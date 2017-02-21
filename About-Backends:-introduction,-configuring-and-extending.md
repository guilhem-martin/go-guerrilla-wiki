# Backend System

Note: Work in progress. Not merged yet.

## Introduction

### What does a backend do?

The job of a go-guerrilla backend is to save email. Think of it as the middleware.

The default go-guerrilla backend is called a "_Gateway_". The Gateway further abstracts the middleware layer by managing a set of independent workers. These workers can be started, shutdown, or new work can be sent to the workers via a channel.

You can also control how many workers to start by changing the `save_workers_size` config option.

### How does the defult Gateway work?

The gateway receives new email envelopes from the server via the Process function. These envelopes are passed via the gateway's **saveMailChan** and picked up by an available worker. The envelope is a value of `github.com/flashmob/go-guerrilla/envelope.Envelope`

### What are the workers?

Each Worker is composed of individual _Processors_ and each Processor is called sequentially to process each envelope. Think of it as production line in a factory. Each worker works on one envelope which they pick out form a bin. Each Processor defines a step the worker must do to complete their work and send a result back.

### How workers work?

These are composed using a Decorator pattern. See footnote [1].

The decorator works by stacking each Processor on a stack, making a single Processor out of many different Processors. Each processor in the stack must either chain the call to the next Processor by passing the envelope, or return the result to its caller.

Here is the interface of a processor

    // Our processor is defined as something that processes the envelope and returns a result and error
    type Processor interface {
	    Process(*envelope.Envelope) (BackendResult, error)
    }

In the code, Go source files that define a Processor are prefixed with 'p_'

There are a few other details about Processors, if you're interested, please see
"Creating your own processors" section.

## Configuration

The `"backend_config"` property holds the configuration for the backend, including configuration values for all the Processors.

The Gateway requires the following settings: `process_stack` and `save_workers_size`.
Then add any other settings that each processor may require.

    "backend_config" :
       {
          "process_stack": "HeadersParser|Debugger|Hasher|Header|Compressor|Redis|MySql",
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

The **process_stack** configures how the worker will process each envelope. 
Each processor is separated using a | character, and each will be executed from left to right.
So the one above will parse the Mime headers, print some debug info, generate some hashes, add a delivery header, compress the email, save the body to redis, and finally save some info to MySQL.

## Extending

The decorator pattern makes it easy to add your own Processors. To get started, create a new .go file, then import

    "github.com/flashmob/go-guerrilla/backends"
    "github.com/flashmob/go-guerrilla/envelope"

From there, if your processor needs a configuration, define your own configuration struct. The struct can only have string, float or numeric fields. Each field must be public. Also add a struct tag to map it to the json file. eg.

    type myFooConfig struct {
       SomeOption string `json:"maildir_path"`
    }

From there, declare your processor as a decorator using the following pattern:


    // The MyFoo decorator [enter what it does]
    var MyFooProcessor = func backends.Decorator {
        return func(c backends.Processor) backends.Processor {
            return ProcessorFunc(func(e *envelope.Envelope) (backends.BackendResult, error) {

                // do some work here
                // if something went wrong and we want to stop processing, return
                // errors.New("Something went wrong")
                // return backends.NewBackendResult(fmt.Sprintf("554 Error: %s", err)), err
                        
                // call the next processor in the chain
                return c.Process(e)
            })
        }
    }

Next, somewhere in the beginning of your code (perhaps in another file), before you create a new go-guerrilla, do this:

    backends.Service.AddProcessor("MyFoo", MyFooProcessor)

There are additional features which allow you to do some things at initialization and shutdown.

### Processor initialization

To load in the config into your processor, you will need to declare and add your own initializer function.
Initializer may do other things, such as open database connections, etc.

The interface looks like this (complete example further down):

    type ProcessorInitializer interface {
	    Initialize(backendConfig BackendConfig) error
    }

There's also convenience type defined so that you can create your initializer as a closure or an anonymous function. 

    type Initialize func(backendConfig BackendConfig) error

Once you've declared your initializer, use the `backends.Service.AddInitializer` function to register it.

So putting it together, your Processor will look something like this


    // The MyFoo decorator [enter what it does]
    var MyFooProcessor = func backends.Decorator {
        config := &myFooConfig{}
        // our initFunc will load the config.
        initFunc := backends.Initialize(func(backendConfig backends.BackendConfig) error {
	    configType := backends.BaseConfig(&myFooConfig{})
	    bcfg, err := backends.Service.ExtractConfig(backendConfig, configType)
	    if err != nil {
	        return err
	    }
	    config := bcfg.(*myFooConfig)
                return nil
        })
        // register our initializer
        backends.Service.AddInitializer(initFunc)
        return func(c backends.Processor) backends.Processor {
            return ProcessorFunc(func(e *envelope.Envelope) (backends.BackendResult, error) {

                // do some work here
                // if something went wrong and we want to stop processing, return
                // errors.New("Something went wrong")
                // return backends.NewBackendResult(fmt.Sprintf("554 Error: %s", err)), err
                        
                // call the next processor in the chain
                return c.Process(e)
            })
        }
    }

### Processor shutdown

Following the same pattern of initialization, there is also a sister Shutdown function

The interface looks like this

    type ProcessorShutdowner interface {
        Shutdown() error
    }

And the convenience type to help you pass an anonymous/closure function looks like this

    type Shutdown func() error

To pass your function to go-guerrilla, use backends.Service.AddShutdowner function

eg.

    backends.Service.AddShutdowner(backends.Shutdown(func() error {
        if db != nil {
            return db.Close()
        }
        return nil
    }))


## Footnotes

[1]: Footnote 1. This pattern is described in detail here: https://github.com/smaxwellstewart/golang-decorator-pattern and this video has good information: https://github.com/smaxwellstewart/golang-decorator-pattern