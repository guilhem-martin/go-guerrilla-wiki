Welcome to the go-guerrilla wiki!

See [Readme.md](https://github.com/flashmob/go-guerrilla) for information about the project.

# Testing
* [Fuzz Testing](https://github.com/flashmob/go-guerrilla/wiki/Fuzz-testing)


# Dev Environment

## InteliJ IDEA

### Debugger configuration

If you're using InteliJ IDEA for your IDE with the Go plugin, you may be able to get debugging working with the following settings:

![InteliJ Idea debuger config for Golang](https://raw.githubusercontent.com/wiki/flashmob/go-guerrilla/go-guerrilla-debug.png)

(Note: for the program arguments, add `-c goguerrilla.debug.conf` - if you want the debugging session to use a different config file)

Happy debuggin'

# Server Limitations

## Size extension

The server doesn't fully implement the SIZE extension.
Only SIZE with an argument is supported. The argument is a number of maximum bytes for each message.
It does not support SIZE argument at the end of MAIL FROM.
Such a feature may be useful for giving different clients custom sizes for various reasons. However, no such need is required yet. PR welcome if you need this.
See more discussion about SIZE https://cr.yp.to/smtp/size.html

# Random Links

## On buffering & pooling

https://gist.github.com/colm-mchugh/ee3b0b7b062a235a871b - source examples 

https://elithrar.github.io/article/using-buffer-pools-with-go/ - nice writeup about Sized buffer pools 

https://github.com/djherbis/buffer - composite buffers (the unbounded bounder could be really useful, the ring buffer is interesting

https://talks.golang.org/2014/readability.slide#1 - Readable Go style