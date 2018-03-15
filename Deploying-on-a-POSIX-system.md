## Deploying guerrillad on a POSIX system (Linux/Darwin/BSD/etc)

### Create a special user 

Create a user just for running the server, eg ‘gmail’ user:

`$ useradd -m gmail`

Place the guerrillad executable in the home directory (or any location of your choice, eg `/usr/local/bin` would be nice too)

Edit the configuration [Configuration](https://github.com/flashmob/go-guerrilla/wiki/Configuration) - configure a server with your host-name that is listening on your public IP, via port 25.

### Permissions

Give permission for the guerrillad executable so that it can access port 25 (and all other privileged ports) like so:

```
$ sudo setcap 'cap_net_bind_service=+ep' /home/gmail/guerrillad
```

If the host is behind a firewall, open port 25:

```
$ sudo iptables -I INPUT -p tcp -m tcp --dport 25 -j ACCEPT
```

### Starting command 

This will put the guerrillad process in the background:

```shell
$ /home/gmail/guerrillad -c /home/gmail/goguerrilla.conf serve >> /home/gmail/smtpd_out.log 2>&1 &
```

Notice that the errors (stdout) are redirected to standard output (stdout). If the server doesn’t start, please inspect the smtpd_out.log file for errors.

If the process is started by a wheel user, (typically root), use sudo to drop down to a lower user:

```shell
$ sudo -i -u gmail /home/gmail/guerrillad -c /home/gmail/goguerrilla.conf serve >> /home/gmail/smtpd_out.log 2>&1 &
```

#### Starting automatically on boot


Place the starting command at the bottom of the /etc/rc.local file.

## DNS Settings


Once your server is running, you need to tell others how to find your server. You do that by setting an MX record for your DNS Zone. The MX record tells everyone the host-name of the server that accepts email for your domain. A domain may have multiple MX records for redundancy, but here we’ll have just one. Your mail server will also need an A record (i.e a host-name that is pointing to some IP address)

Create a sub-domain for your server, by adding an A record. A popular subdomain host-name choice could be ‘smtp’, eg. smtp.example.com - point it to the IP address of your server.

Add a new MX record, with the above host-name. Use 0 for the priority, typically enter @ if there is a Name field, and the full host-name in the Host field. Important: Delete any other previous MX records.

Tip: The sub-domain that you have created in step 1 should also match the ‘host_name’ configuration setting in the goguerrilla.conf config file. If setting up an SSL certificate, make sure that the certificate subject also matches the ‘host_name’

## Let’s Encrypt (LE)

If you're using LE, the server would need to have permission to access the keys located in `/etc/letsencrypt`, if running with under a low privileged user then we need to do some extra steps:

Add a new ssl-group

```
$ sudo addgroup ssl-cert
```

Give access to the ssl-cert group, including all sub-directories. 

```
$ sudo chgrp -R ssl-cert /etc/letsencrypt
$ sudo chmod -R g=rX /etc/letsencrypt
```

Now add the gmail user to the ssl-cet group

```
$ sudo sudo usermod -a -G ssl-cert gmail
```

### See also
[Configuration](https://github.com/flashmob/go-guerrilla/wiki/Configuration)

[Log file rotation](https://github.com/flashmob/go-guerrilla/wiki/Automatic-log-file-management-with-logrotate)

[Use Nginx as a proxy](https://github.com/flashmob/go-guerrilla/wiki/Using-Nginx-as-a-proxy)