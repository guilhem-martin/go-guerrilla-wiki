## Introduction

Logrotate is a popular log rotation utility that, according to its authors, "allows for the automatic rotation compression, removal and mailing of log files. Logrotate can be set to handle a log file daily, weekly, monthly or when the log file gets to a certain size.". It's available for most Linux distributions as a package, or you can build it from source here: https://github.com/logrotate/logrotate

Chances are, you already may have logrotate installed!

## Configuring

Look in `/etc/logrotate.d/`

The above directory is the most common directory where logrotate looks for customized configurations for different software installed on your system. 

Create a new file there, eg. `guerrilla` Then begin editing your configuration by changing the following to your preferences:

```
/home/gmail/*.log {
    daily
    size 200M
    rotate 3
    compress
    missingok
    notifempty
    create 0644 gmail gmail
    su gmail gmail
    postrotate
        kill -USR1 `cat /var/run/go-guerrilla.pid`
    endscript
}
```

Where:

* `/home/gmail/*.log` - is the location of the log files. * is a wildcard.
* `daily` rotate after a day
* `size` or, rotate after 200M or any size you choose. 
* `rotate 3` - leave 3 days of logs
* `compress` gzip old files
* `missingok` don't complain if log file is missing
* `notifempty` don't rotate if empty
* `create` create new files with mode, owner, group permissions
* `su` Use this user to perform all actions

The secret sauce above is the `postrotate` ... `endscript` section. This tells go-guerrilla to re-open the log files after they have been rotated. Make sure to change `/var/run/go-guerrilla.pid` to the file where go-guerrilla writes out the pid (process id).

Save and run it to see if it works:

    $ logrotate /etc/logrotate.conf

It may barf at the beginning that it can't compress, but that should be OK.

check status like this:

    $ cat /var/lib/logrotate/status 

### RTFM ;-)
Check out the logrotate manual for a complete list of options and examples: http://linuxcommand.org/man_pages/logrotate8.html

### Implementation details and limitations

Log file re-opening is triggered by listening to the `USR1` [Unix signal] (https://en.wikipedia.org/wiki/Unix_signal) - therefore it will only work on systems following POSIX. Windows users may try using Cygwin which supports the kill command or the new [Windows Subsystem for Linux](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux)

When using go-guerrilla as a package, it will not listen to any signal. It would be your choice on how to best capture an event and then deal with the event (call the log re-opening functions, etc). There's a handy helper function on AppConfig that can fire all the events for you:

```go
func (c *AppConfig) EmitLogReopenEvents(app Guerrilla)
```