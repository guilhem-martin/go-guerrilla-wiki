## HAProxy configuration

Add the following to your haproxy.conf

```
frontend ft_smtp
	bind 0.0.0.0:25
	mode tcp
	timeout client 3m
	log global
	option tcplog
	default_backend bk_smtp

backend bk_smtp
	mode tcp
	log global
	option tcplog
	timeout server 3m
	timeout connect 10s
	server guerrilla 127.0.0.1:2525 send-proxy
```

This will let HAProxy listen on Port 25 and forward the traffic to localhost Port 2525.
For multiple backend servers adjust the `server` line in the backend config accordingly, e.g. for two backends:
```
	server guerrilla-1 192.168.1.1:2525 send-proxy
	server guerrilla-2 192.168.1.2:2525 send-proxy
```

## go-guerrilla config

If using HAProxy with the default configuration (and up to version 1.6.5), every connecting will be greeted with
```
220 mail.test.de SMTP Guerrilla(v1.6.5-12-g13718a2) #1 (1) 2024-06-06T18:42:18+02:00
554 5.5.1 Unrecognized command
```
Since version 1.6.6 you can enable the PROXY mode within the config for every server.
Add the line `"proxyon" : true,` to your server config. go-guerrilla will accept and parse
the PROXY command and will set the RemoteIP given by the remote server.