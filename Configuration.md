

Configuration
============================================

The configuration is in strict JSON format. Here is an annotated configuration.

To get started, copy [goguerrilla.conf.sample](https://github.com/flashmob/go-guerrilla/blob/master/goguerrilla.conf.sample) to `goguerrilla.conf.json`


    {
        "allowed_hosts": ["guerrillamail.com","guerrillamailblock.com","sharklasers.com","guerrillamail.net","guerrillamail.org"], // What hosts to accept
        "pid_file" : "/var/run/go-guerrilla.pid", // pid = process id, so that other programs can send signals to our server
        "log_file" : "stderr", // can be "off", "stderr", "stdout" or any path to a file
        "log_level" : "info", // can be  "debug", "info", "error", "warn", "fatal", "panic"
        "backend_config" :
            {
                "log_received_mails": true,
                "save_workers_size": 1,
                "save_process" : "HeadersParser|Header|Debugger", // chain of processors to be executed when saving email
                "validate_process" "", // similar to save_process, chain of processors that validate a recipient
                "primary_mail_host" : "mail.example.com", // used by the Header processor. 
                "gw_save_timeout" : "30s", // how long before giving up on saving an email (optional, default 30s)
                "gw_val_rcpt_timeout" : "3s" // how long before giving up in validating a recipient (optional, default 5s)
            },
        "servers" : [ // the following is an array of objects, each object represents a new server that will be spawned
            {
                "is_enabled" : true, // boolean
                "host_name":"mail.test.com", // the hostname of the server as set by MX record
                "max_size": 1000000, // maximum size of an email in bytes
                "timeout":180, // timeout in number of seconds before an idle connection is closed
                "listen_interface":"127.0.0.1:25", // listen on ip and port
                "max_clients": 1000, // max clients at one time
                "log_file":"/dev/stdout", // optional. Can be "off", "stderr", "stdout" or any path to a file. Will use global setting of empty.
                "tls" : { // optional, recommended
                    "start_tls_on":true, // supports the STARTTLS command?
                    "tls_always_on":false, // always connect using TLS? If true, start_tls_on will be false
                    "private_key_file":"/path/to/pem/file/test.com.key",  // full path to pem file private key
                    "public_key_file":"/path/to/pem/file/test.com.crt", // full path to pem file certificate
                    "protocols" : ["tls1.0", "tls1.2"], // minimum protocol on the left, maximum on the right
                    // the following is a list of cipher suites to use.
                    "ciphers" : ["TLS_FALLBACK_SCSV", "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256", 
                        "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305", "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305", 
                        "TLS_RSA_WITH_RC4_128_SHA", "TLS_RSA_WITH_AES_128_GCM_SHA256", 
                        "TLS_RSA_WITH_AES_256_GCM_SHA384", "TLS_ECDHE_ECDSA_WITH_RC4_128_SHA", 
                        "TLS_ECDHE_RSA_WITH_RC4_128_SHA", "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", 
                        "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384"],
                    "curves" : ["P256", "P384", "P521", "X25519"],
                    "prefer_server_cipher_suites" : true
                }
            },
            // the following is a second server, but listening on port 465 and always using TLS
            {
                "is_enabled" : true,
                "host_name":"mail.test.com",
                "max_size":1000000,
                
                "timeout":180,
                "listen_interface":"127.0.0.1:465",
                
                "max_clients":500,
                "tls" : {
                    "start_tls_on":false,
                    "tls_always_on":true,
                    "private_key_file":"/path/to/pem/file/test.com.key",
                    "public_key_file":"/path/to/pem/file/test.com.crt",
                    "protocols" : ["tls1.0", "tls1.2"],
                    "ciphers" : ["TLS_FALLBACK_SCSV", "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256", 
                        "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305", "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305", 
                        "TLS_RSA_WITH_RC4_128_SHA", "TLS_RSA_WITH_AES_128_GCM_SHA256", 
                        "TLS_RSA_WITH_AES_256_GCM_SHA384", "TLS_ECDHE_ECDSA_WITH_RC4_128_SHA", 
                        "TLS_ECDHE_RSA_WITH_RC4_128_SHA", "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", 
                        "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384"],
                    "curves" : ["P256", "P384", "P521", "X25519"],
                    "prefer_server_cipher_suites" : true
                }
            }
            // repeat as many servers as you need
        ]
    }
    

The Json parser is very strict on syntax. If there's a parse error and it
doesn't give much clue, then test your syntax here:
http://jsonlint.com/#

#### TLS Configuration options

A range of protocols, cipher suites and curves can be configured. 
It's recommended to use the new cipher suites added since Go 1.8, 
avoid SSLv3, and avoid `CBC` based cipher suites. However, be warned
that disabling old stuff might also prevent some email getting through -
as many SMTP senders still use the old ones.

If any of the following settings are absent, GO's defaults will be used.

##### Protocols

The `protocols` setting is an array with just two elements.
The first element is the minimum protocol version, the second is the maximum. 
Avoid ssl3.0. All lowercase.

* ssl3.0
* tls1.0
* tls1.1
* tls1.2

##### Ciphers

Here is a list of cipher suites that can be used. Note that `TLS_FALLBACK_SCSV`
is not a cipher, but should be included to prevent downgrade attacks.
Generally, avoid CBC when you can.

* TLS_RSA_WITH_3DES_EDE_CBC_SHA
* TLS_RSA_WITH_AES_128_CBC_SHA
* TLS_RSA_WITH_AES_256_CBC_SHA
* TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
* TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
* TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA
* TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
* TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
* TLS_RSA_WITH_RC4_128_SHA
* TLS_RSA_WITH_AES_128_GCM_SHA256
* TLS_RSA_WITH_AES_256_GCM_SHA384
* TLS_ECDHE_ECDSA_WITH_RC4_128_SHA
* TLS_ECDHE_RSA_WITH_RC4_128_SHA
* TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
* TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
* TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
* TLS_FALLBACK_SCSV

Since Go 1.8 -

* TLS_RSA_WITH_AES_128_CBC_SHA256
* TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
* TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
* TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
* TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
* TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305

##### Curves

`curves` contains the elliptic curves that will be used in an ECDHE handshake, 
in preference order.

* P256
* P384
* P521

Since Go 1.8 -

* X25519

### Backend Configuration

backend configuration details here:
https://github.com/flashmob/go-guerrilla/wiki/Backends,-configuring-and-extending

#### Saving to MySQL & Redis
Here is the page discussing how to configure saving to [Redis and MySQL](https://github.com/flashmob/go-guerrilla/wiki/Configuration-example:-save-to-Redis-&-MySQL): 
