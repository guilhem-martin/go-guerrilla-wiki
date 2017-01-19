Welcome to the go-guerrilla wiki!

See [Readme.md](https://github.com/flashmob/go-guerrilla) for information about the project.


# Dev Environment

## InteliJ IDEA

### Debugger configuration

If you're using InteliJ IDEA for your IDE with the Go plugin, you may be able to get debugging working with the following settings:

![InteliJ Idea debuger config for Golang](https://raw.githubusercontent.com/wiki/flashmob/go-guerrilla/go-guerrilla-debug.png)

(Note: for the program arguments, add `-c goguerrilla.debug.conf` - if you want the debugging session to use a different config file)

Happy debuggin'

# Random Links

## On buffering & pooling

https://gist.github.com/colm-mchugh/ee3b0b7b062a235a871b - source examples 
https://elithrar.github.io/article/using-buffer-pools-with-go/ - nice writeup about Sized buffer pools 
https://github.com/djherbis/buffer - composite buffers (the unbounded bounder could be really useful, the ring buffer is interesting