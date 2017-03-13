This page is a stub


Configuration
============================================

The configuration is in strict JSON format. Here is an annotated configuration.
Copy `goguerrilla.conf.sample` to `goguerrilla.conf.json`


    {
        "allowed_hosts": ["guerrillamail.com","guerrillamailblock.com","sharklasers.com","guerrillamail.net","guerrillamail.org"], // What hosts to accept
        "pid_file" : "/var/run/go-guerrilla.pid", // pid = process id, so that other programs can send signals to our server
        "log_file" : "stderr", // can be "off", "stderr", "stdout" or any path to a file
        "log_level" : "info", // can be  "debug", "info", "error", "warn", "fatal", "panic"
        "backend_name": "guerrilla-db-redis", // what backend to use for saving email. See /backends dir
        "backend_config" :
            {
                "mysql_db":"gmail_mail",
                "mysql_host":"127.0.0.1:3306",
                "mysql_pass":"ok",
                "mysql_user":"root",
                "mail_table":"new_mail",
                "redis_interface" : "127.0.0.1:6379",
                "redis_expire_seconds" : 7200,
                "save_workers_size" : 3,
                "primary_mail_host":"sharklasers.com"
            },
        "servers" : [ // the following is an array of objects, each object represents a new server that will be spawned
            {
                "is_enabled" : true, // boolean
                "host_name":"mail.test.com", // the hostname of the server as set by MX record
                "max_size": 1000000, // maximum size of an email in bytes
                "private_key_file":"/path/to/pem/file/test.com.key",  // full path to pem file private key
                "public_key_file":"/path/to/pem/file/test.com.crt", // full path to pem file certificate
                "timeout":180, // timeout in number of seconds before an idle connection is closed
                "listen_interface":"127.0.0.1:25", // listen on ip and port
                "start_tls_on":true, // supports the STARTTLS command?
                "tls_always_on":false, // always connect using TLS? If true, start_tls_on will be false
                "max_clients": 1000, // max clients at one time
                "log_file":"/dev/stdout" // optional. Can be "off", "stderr", "stdout" or any path to a file. Will use global setting of empty.
            },
            // the following is a second server, but listening on port 465 and always using TLS
            {
                "is_enabled" : true,
                "host_name":"mail.test.com",
                "max_size":1000000,
                "private_key_file":"/path/to/pem/file/test.com.key",
                "public_key_file":"/path/to/pem/file/test.com.crt",
                "timeout":180,
                "listen_interface":"127.0.0.1:465",
                "start_tls_on":false,
                "tls_always_on":true,
                "max_clients":500
            }
            // repeat as many servers as you need
        ]
    }
    }

The Json parser is very strict on syntax. If there's a parse error and it
doesn't give much clue, then test your syntax here:
http://jsonlint.com/#

### Backend Configuration

backend configuration details here:
https://github.com/flashmob/go-guerrilla/wiki/Backends,-configuring-and-extending#configuration