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
* `missingok` don't complain of log file is missing
* `notifempty` don't rotate if empty
* `create` create new files with mode, owner, group permissions
* `su` Use this user to perform all actions

The secret sauce above is the `postrotate` ... `endscript` section. This tells go-guerrilla to re-open the log files after they have been rotated. Make sure to change `/var/run/go-guerrilla.pid` to the file where go-guerrilla writes out the pid (process id).

Save and check status like this:

    $ cat /var/lib/logrotate/status 

Check out the manual here for a complete list of options and examples: http://linuxcommand.org/man_pages/logrotate8.html