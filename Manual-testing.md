Manual Testing
============================================

Once you have built & launched the `go-guerrilla` server, you can test it manually by sending SMTP message manually. This is useful for debugging and testing the server's capabilities.

The following sections will guide you through the process of sending a email message using the `nc` (netcat) command, starting with a simple ascii message, and gradually increasing the complexity by adding utf-8 encoded body, multi-part, attachments, and nested boundaries.

The `Content-Disposition` header can have the following values:
- `inline` - most email clients will try to display the attachment within the body of the email
- `attachment` - most email clients will display the attachment as a separate file, to be opened or saved by the user

You can run the following sections sent with the help of `nc` (netcat). Be sure to export the `PORT` number before running the commands. The port output is displayed when you start the `go-guerrilla` server. You can configure it in the `goguerrilla.conf.json` file, by updating the `server.listen_interface` value.

```bash
export PORT=2525
```

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
nc localhost ${PORT} < ascii_simple_payload.txt
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
Subject: utf8 body

VGhpcyBib2R5IHN1cHBvcnRzIFVURi04IGNoYXJhY3RlcnMgbGlrZSDDqcOow6DDtiBhbmQgY2hpbmVzZSBjaGFyYWN0ZXJzIGxpa2Ug5L2g5aW9
.
QUIT
```

The above is a simple message with a subject and a body using utf-8 charset. The `.` on a line by itself signifies the end of the message. The `QUIT` command closes the connection. Copy and paste the above in a file called `utf8_simple_payload.txt`

Once this is done, start the `go-guerrilla` server, and then run the below command in a terminal:

```bash
nc localhost ${PORT} < utf8_simple_payload.txt
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
INFO[0002] Handle client [127.0.0.1], id: 2
# and then other information depending on the go-guerrilla processors you have enabled
```

If you display the email, you'll see the utf-8 decoded body, with proper characters.

## Testing with a multi-part message

The previous examples were simple messages with a single part. This example demonstrates a multi-part message with a text and html part.
Note the new `Content-Type` header with the `multipart/alternative` value, and the `boundary` parameter that separates the parts.

```bash
EHLO hostname
MAIL FROM: <test@example.com>
RCPT TO: <test@example.com>
DATA
Date: date
Subject: A multipart message
Content-Type: multipart/alternative; boundary=bcaec520ea5d6918e204a8cea3b4

--bcaec520ea5d6918e204a8cea3b4
Content-Type: text/plain; charset=ISO-8859-1

*hi!*

--bcaec520ea5d6918e204a8cea3b4
Content-Type: text/html; charset=ISO-8859-1
Content-Transfer-Encoding: quoted-printable

<p><b>hi!</b></p>

--bcaec520ea5d6918e204a8cea3b4--
.
QUIT
```

Create a new file called `html-and-plain-multiparts_payload.txt` with the above content.

Start the `go-guerrilla` server, and then run the below command in a terminal:

```bash
nc localhost ${PORT} < html-and-plain-multiparts_payload.txt
```

`go-guerrilla` will parse each part. Note that it's up to the client to decide which part to display. Most clients will display the html part if it's available. Other more basic clients will prefer display the plain text part.

## Introducing attachments

This example is still multipart, but now includes an attachment. The attachment is PNG image encoded in base64.

```bash
EHLO hostname
MAIL FROM: <test@example.com>
RCPT TO: <test@example.com>
DATA
Date: date
Subject: A simple test message subject
Content-Type: multipart/alternative; boundary=bcaec520ea5d6918e204a8cea3b4

--bcaec520ea5d6918e204a8cea3b4
Content-Type: text/plain; charset=ISO-8859-1

*hi!*

--bcaec520ea5d6918e204a8cea3b4
Content-Type: image/png
Content-Transfer-Encoding: base64
Content-Disposition: attachment;
        filename="image.png"

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

--bcaec520ea5d6918e204a8cea3b4--
.
QUIT
```

`go-guerrilla` will parse each part and may save the attachment to a file, depending on the backend processor you have enabled.

As the attachment file name is specified in the `Content-Disposition` header, most SMTP email clients will use this name to display and save the attachment.

## Example with nested boundaries

This example demonstrates a message with nested boundaries.
In the parent boundary, you'll find 2 parts:
- a multipart/alternative part that has its own boundary and contains 2 parts: text/plain and text/html
- an image/png attachment

Note that a part - whether nested or not - end is marked by the boundary string followed by `--`.

```bash
EHLO hostname
MAIL FROM: <test@example.com>
RCPT TO: <test@example.com>
DATA
Date: date
Subject: Nested Multipart
Content-Type: multipart/alternative; boundary=e89a8ff1c1e83553e304be640612

--e89a8ff1c1e83553e304be640612
Content-Type: multipart/alternative; boundary=e89a8ff1c1e83553e004be640610

--e89a8ff1c1e83553e004be640610
Content-Type: text/plain; charset=UTF-8

*this is a nested text plain body*

--e89a8ff1c1e83553e004be640610
Content-Type: text/html; charset=UTF-8

<b>this is a nested html body</b>

--e89a8ff1c1e83553e004be640610--
--e89a8ff1c1e83553e304be640612
Content-Type: image/png; name="x.png"
Content-Disposition: attachment;
	filename="x.png"
Content-Transfer-Encoding: base64
X-Attachment-Id: f_h1edgigu0

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
--e89a8ff1c1e83553e304be640612--
.
QUIT
```

