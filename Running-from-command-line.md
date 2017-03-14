
### guerrillad command

```
Usage:
  guerrillad [command]

Available Commands:
  serve       start the small SMTP server
  version     Print the version info

Flags:
  -v, --verbose   print out more debug information
```

### Starting

Use `./guerrillad start` to start.

When the daemon starts running, it will read the config and create the servers.
Logging some info messages to the stderr. It will then change the stderr outpout
to whatever has been specified in the config. 

Should the reading of the configuration fail, or starting of a server fail,
the daemon will abort starting up, exit with a 1, and log the error.

### Re-loading the config

use `kill -HUP <process-id>` to send a signal to the deamon. The daemon will
write out the process-id (pid) to a file.

## Re-open log file

Use `kill -USER1 <process-id>`

This is used for [log rotation](https://github.com/flashmob/go-guerrilla/wiki/Automatic-log-file-management-with-logrotate).

### Examples 

For example, if you want to discard everything:

`./guerrillad serve >> /dev/null 2>&1 &`

Or you want to save it:

`./guerrillad serve >> /home/mike/smtpd_log 2>&1 &`

The >> means redirect-and-append.
The 2>&1 means redirect errors and output.
The & at the end means run it in the background. 

If you want to look at the messages live, you can than open another console and tail it
