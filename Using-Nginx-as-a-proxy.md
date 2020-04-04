### Using Nginx as a proxy

Nginx can be used to proxy SMTP traffic for GoGuerrilla SMTPd

Why proxy SMTP with Nginx?

 *	Terminate TLS connections: (eg. Early Golang versions were not there yet when it came to TLS.)
 OpenSSL on the other hand, used in Nginx, has a complete implementation of TLS with familiar configuration.
 *	Nginx could be used for load balancing and authentication

 1.	Compile nginx with --with-mail --with-mail_ssl_module (most current nginx packages have this compiled already)

 2.	Configuration:


		mail {
	        server {
	                listen  15.29.8.163:25;
	                protocol smtp;
	                server_name  ak47.example.com;
	                auth_http smtpauth.local:80/auth.txt;
	                smtp_auth none;
	                timeout 30000;
	                smtp_capabilities "SIZE 15728640";

	                # ssl default off. Leave off if starttls is on
	                #ssl                  on;
	                ssl_certificate      /etc/ssl/certs/ssl-cert-snakeoil.pem;
	                ssl_certificate_key  /etc/ssl/private/ssl-cert-snakeoil.key;
	                ssl_session_timeout  5m;
	                # See https://mozilla.github.io/server-side-tls/ssl-config-generator/ Intermediate settings
	                ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	                ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
	                ssl_prefer_server_ciphers on;
	                # TLS off unless client issues STARTTLS command
	                starttls on;
	                proxy on;
	        }
		}

		http {

		    # Add somewhere inside your http block..
		    # make sure that you have added smtpauth.local to /etc/hosts
		    # What this block does is tell the above stmp server to connect
		    # to our golang server configured to run on 127.0.0.1:2525

		    server {
                    listen 15.29.8.163:80;
                    server_name 15.29.8.163 smtpauth.local;
                    root /home/user/http/auth/;
                    access_log off;
                    location /auth.txt {
                        add_header Auth-Status OK;
                        # where to find your smtp server?
                        add_header Auth-Server 127.0.0.1;
                        add_header Auth-Port 2525;
                    }

                }

		}


### Configuring Go-guerrilla

In your server configuration json file, add `xclient_on` which is a boolean value, set to `true`. Eg.

    {
            "is_enabled" : true,
            "host_name":"mail.test.com",
            "max_size": 1000000,
            "private_key_file":"/path/to/pem/file/test.com.key",
            "public_key_file":"/path/to/pem/file/test.com.crt",
            "timeout":180,
            "listen_interface":"127.0.0.1:25",
            "start_tls_on":true,
            "tls_always_on":false,
            "max_clients": 1000,
            "log_file" : "stderr",
            "xclient_on" : true
        }

Resources:

https://www.nginx.com/resources/admin-guide/mail-proxy/

Notice that we've also setup a http server with the web root pointing to /home/user/http/auth/
which can be an arbitrary directory. We've placed a blank file named auth.txt in there, it's just
so nginx doesn't return 404. Next, we tell nginx to add some custom headers to the http response,
these headers tell the proxy where to find our go-guerrilla server. This is so much faster than 
executing a cgi script - although you can always customize to a cgi script if you need 
more control, such as load balancing.

