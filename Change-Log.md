1.6.6 - 6 June 2024
- Support of the PROXY command for HAProxy
- Increased buffer size for header from 40KiB to ~10MiB

1.6.5
Takeover of the project by phires

1.6.0
Large refactoring of the code. 
- Introduced "backends": modular architecture for saving email
- Issue: Use as a package in your own projects! https://github.com/phires/go-guerrilla/issues/20
- Issue: Do not include dot-suffix in emails https://github.com/phires/go-guerrilla/issues/24
- Logging functionality: logrus is now used for logging. Currently output is going to stdout
- Incompatible change: Config's allowed_hosts is now an array
- Incompatible change: The server's command is now a command called `guerrillad`
- Config re-loading via SIGHUP: reload TLS, add/remove/enable/disable servers, change allowed hosts, timeout.
- Begin writing automated tests
- Plus all the pull requests since 6th Jan, https://github.com/phires/go-guerrilla/pulls?q=is%3Apr+is%3Aclosed
 

1.5.1 - 4nd Nov 2016
- Small optimizations to the way email is saved

1.5 - 2nd Nov 2016
- Fixed a DoS vulnerability, stop reading after an input limit is reached
- Fixed syntax error in Json goguerrilla.conf.sample
- Do not load certificates if SSL is not enabled
- check database back-end connections before starting

1.4 - 25th Oct 2016
- New Feature: multiple servers!
- Changed the configuration file format to support multiple servers,
this means that a new configuration file would need to be created form the
sample (goguerrilla.conf.sample)
- Organised code into separate files. Config is now strongly typed, etc
- Deprecated nginx proxy support


1.3 14th July 2016
- Number of saveMail workers added to config (GM_SAVE_WORKERS)
- convenience function for reading int values form config
- advertise PIPELINING
- added HELP command
- rcpt to host validation: now case insensitive and done earlier (after DATA)
- iconv switched to: go get gopkg.in/iconv.v1

1.2 1st July 2016
- Reload config on SIGHUP
- Write current process id (pid) to a file, /var/run/go-guerrilla.pid by default