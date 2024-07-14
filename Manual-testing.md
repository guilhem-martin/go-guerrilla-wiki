

Manual Testing
============================================

Once you have built the server, you can test it manually by sending SMTP message manually. This is useful for debugging and testing the server's capabilities.

You can run the following sections sent with the help of `nc` (netcat).

## Testing with basic SMTP message and encoding

You can start with a basic smtp ascii message.

```bash
EHLO hostname
MAIL FROM: <test@example.com>
RCPT TO: <test@example.com>
DATA
Date: date
Content-Type: text/plain; charset=us-ascii
Subject: A simple test message subject

A simple test message body
.
QUIT
```

The above is a simple message with a subject and body using ascii charset. The `.` on a line by itself signifies the end of the message. The `QUIT` command closes the connection. Copy and paste the above in a file called `ascii_simple_payload.txt`

Before to run this example, edit the `goguerrilla.conf.json` to add `example.com` to the `allowed_hosts` list.

Once this is done, start the `go-guerrilla` server, and then run the below command in a terminal:

```bash
nc localhost 2525 < ascii_simple_payload.txt
```

As output of the netcat command, you'll get something similar to:

```
220 localhost SMTP Guerrilla(v1.6.6-26-g7ad89cb) #1 (1) 2024-07-14T11:48:20+02:00
250-localhost Hello
250-SIZE 1000000
250-PIPELINING
250-ENHANCEDSTATUSCODES
250 HELP
250 2.1.0 OK
250 2.1.5 OK
354 Enter message, ending with '.' on a line by itself
250 2.0.0 OK: queued as 921c84607da3
```

As output in the `go-guerrilla` server, you'll see something similar to:

```
INFO[0002] Handle client [127.0.0.1], id: 1
# and then other information depending on the processors you have enabled
```

## Testing with basic SMTP message and utf8 encoding for the body

This example uses a utf8 encoded body with accented characters as well as Chinese characters.

```bash
EHLO hostname
MAIL FROM: <test@example.com>
RCPT TO: <test@example.com>
DATA
Date: date
Content-Type: text/plain; charset=utf-8
Subject: ascii subject and utf8 body

VGhpcyBib2R5IHN1cHBvcnRzIFVURi04IGNoYXJhY3RlcnMgbGlrZSDDqcOow6DDtiBhbmQgY2hpbmVzZSBjaGFyYWN0ZXJzIGxpa2Ug5L2g5aW9
.
QUIT
```

The above is a simple message with a subject and body using utf-8 charset. The `.` on a line by itself signifies the end of the message. The `QUIT` command closes the connection. Copy and paste the above in a file called `utf8_simple_payload.txt`

As output in the `go-guerrilla` server, you'll see something similar to:

```
220 localhost SMTP Guerrilla(v1.6.6-26-g7ad89cb) #1 (1) 2024-07-14T11:48:20+02:00
250-localhost Hello
250-SIZE 1000000
250-PIPELINING
250-ENHANCEDSTATUSCODES
250 HELP
250 2.1.0 OK
250 2.1.5 OK
354 Enter message, ending with '.' on a line by itself
250 2.0.0 OK: queued as 921c84607da3
```

As output in the `go-guerrilla` server, you'll see something similar to:

```
INFO[0002] Handle client [127.0.0.1], id: 2
# and then other information depending on the processors you have enabled
```

If you display the email, you'll see the utf-8 decoded body, with proper characters.
