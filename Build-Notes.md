
### Quick Start

(assuming Go and Dep is available with default paths)
```
$ go get github.com/flashmob/go-guerrilla
$ cd ~/go/src/github.com/flashmob/go-guerrilla
$ dep ensure
$ make test
$ make guerrillad
```
Then head over to the configuration https://github.com/flashmob/go-guerrilla/wiki/Configuration

### Build prerequisites

This project requires the following tools to build: GNU Make and ~~Glide~~Go's "Dep" dependency manager.

"Dep" is used to manage dependencies.

`dep ensure` to install all dependencies. You can also customize your Gopkg.toml file, 
eg. if you're not using Redis or MySQL, or bring your own dependencies.


Note: Build before May 2019 used Glide package dependency manager.

If using glide, one would typically run `$ glide install` and all dependencies will be resolved and placed in the /vendor directory (which is excluded from git)

GNU Make is available on most Linux distributions by default, at least where the C/C++ compilers are installed.
On Windows, GNU Make is [available through CygWin](https://stackoverflow.com/questions/16135945/is-there-a-cygwin-version-of-gnu-make).

Finally, to build, use:

`$ make guerrillad`

To run tests:

`$ make test`

### Decoding charsets: Using GNU iconv

To parse the email headers and to convert 7-bit MIME encoded headers to UTF-8, the internal mail package provides a `MimeHeaderDecode` function. This function uses the default charset decoder found in the standard mime package. However, this decoder only supports a handful of the most popular encodings, more are needed when dealing with email.

There are currently two alternatives. One, is to use the GNU iconv libray, which is part of the GNU library. 
The other is to use `golang.org/x/net/html/charset`. 

The end-user of the internal `mail` package can specify which to use by either importing:
`github.com/flashmob/go-guerrilla/mail/iconv`
or `github.com/flashmob/go-guerrilla/mail/encoding`

Also, use the underscore `_` character in front of the import. 

For an example, see `cmd/guerrillad/serve.go` - it imports `github.com/flashmob/go-guerrilla/mail/encoding` which causes the `MimeHeaderDecode` function to use `golang.org/x/net/html/charset`

### Getting errors when building with iconv?

Note that when using iconv, to build, your system would need the GNU library headers. I.e. typically install with `sudo apt-get install libc6-dev`. 