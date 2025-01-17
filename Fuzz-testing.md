# go-guerrilla fuzz testing

We use fuzz testing to further improve the stability and security of the software,
making usage of the [go-fuzz](https://github.com/dvyukov/go-fuzz) package.
The tests are all in the *fuzz.go* file if the tests branch.

## Preparations

### Branches

The fuzz test lives in the [Test Branch](https://github.com/phires/go-guerrilla/tree/tests)

We use a separate, as not everyone needs to run the fuzz tests, and there's a lot specific files. The files of interest are fuzz.go & fuzz_test.go, plus the 'corpus' data in the /workdir

Please checkout the branch *tests* for this:


    $ git checkout tests

    $ git pull

We try to keep the branch up-to-date with the *master* branch.

### go-fuzz

If you don’t have *go-fuzz* already installed, do that now.

    $ go get github.com/dvyukov/go-fuzz/go-fuzz
    $ go get github.com/dvyukov/go-fuzz/go-fuzz-build

Build the packages and you should get the *go-fuzz* as well as the *go-fuzz-build* tools. 
(Assuming `go get` will build these automatically and leave them in `$GOPATH/bin`)

### Corpus files

If you have pulled the recent *tests* branch you should now also have the *workdir* and within that
the *corpus* folder. This contains some files that are used to initiate the fuzzing.
You are very welcome to create further corpus files, the ones we use are the example smtp files of the
go-fuzz package.

## Generating your own corpus files

See fuzz_test.go - run the TestGenerateCorpus test by itself to generate the corpus files, or add additional files there. The reason why a program is used to generate the corpus was because we couldn't use the text editor to insert non-printable characters that we want. For example, SMTP likes to have CR + LF at the end. To run TestGenerateCorpus by itself:

`go test -v github.com/phires/go-guerrilla -run ^TestGenerateCorpus$`

The go-fuzz program will also generate its own corpus files during execution and leave them there so that it can resume the tests from where it left.

## Putting it all together

After everything is prepared you can start. Build the package with *go-fuzz*:

`$ go-fuzz-build github.com/phires/go-guerrilla`

This will take a while and create a file named *guerrilla-fuzz.zip*
Now the fuzzing process itself can be started:

`$ go-fuzz -bin=guerrilla-fuzz.zip -workdir=workdir -procs=250`

This will run for quite a while. Eventually you will get an output that contains *crashers*.

You can investigate on those crashes looking at the *.output* files in the *workdir/crashers* folder.

## Fuzz Function details

So here is our initial Fuzz function below. The go-fuzz program tries to re-use the function without
restarting, which means we can initialize the server once, and then use the client pool to re-use our clients to speed things up. So we setup using the [init function](https://golang.org/doc/effective_go.html#init) function in fuzz.go.



When you run the go-fuzz program, it will print out some statistics every second.
eg.

`2017/02/05 02:30:16 slaves: 500, corpus: 336 (1m17s ago), crashers: 2, restarts: 1/1490, execs: 1761505 (12490/sec), cover: 1040, uptime: 2m21s`

What you want is a Fuzz function that has a high 'cover' number, and a low restart rate. You can see that with 500 clients, it can execute 12490 times per second! See go-fuzz readme for more details about these statistics.

Our function doesn't use a real TCP connection, it uses a mock connection which consists of 2 pipes. We are the client end of the pipe. For each call to Fuzz() we borrow a client from the pool, which may be recycled after.
Next, we read the greeting from the server. After the greeting, we use io.Copy to inject raw input 
from the fuzzer to the server. We wait a little for the server to process the input, if we see that the server buffered something, try to read something, but don't block. 


```go

// Fuzz passes the data to the mock connection
// Data is random input generated by go-fuzz, note that in most cases it is invalid.
// The function must return 1 if the fuzzer should increase priority of the given input during subsequent
// fuzzing (for example, the input is lexically correct and was parsed successfully); -1 if the input must
// not be added to corpus even if gives new coverage; and 0 otherwise
func Fuzz(data []byte) int {

	var wg sync.WaitGroup
	// grab a new mocked tcp connection, it consists of two pipes (io.Pipe)
	conn := mocks.NewConn()

	// Get a client from the pool
	poolable, err := fuzzServer.clientPool.Borrow(conn.Server, 1, logOff)
	if c, ok := poolable.(*client); !ok {
		panic("cannot borrow from pool")
	} else {
		mockClient = c
	}

	defer func() {
		conn.Close()
		// wait for handleClient to exit
		wg.Wait()
		// return to the pool
		fuzzServer.clientPool.Return(mockClient)
	}()

	wg.Add(1)
	go func() {
		fuzzServer.handleClient(mockClient)
		wg.Done()
	}()
	b := make([]byte, 1024)
	if n, err := conn.Client.Read(b); err != nil {
		return 0
	} else if isFuzzDebug {
		fmt.Println("Read", n, string(b))
	}


	// Feed the connection with fuzz data (we are the _client_ end of the connection)
	if _, err = io.Copy(conn.Client, bytes.NewReader(data)); err != nil {
		return 0
	}

	// allow handleClient to process
	time.Sleep(time.Millisecond + 10)


	if mockClient.bufout.Buffered() == 0 {
		// nothing to read - no complete commands sent?
		return 0
	}
	if n, err := conn.Client.Read(b); err != nil {
		return 0
	} else if isFuzzDebug {
		fmt.Println("Read", n, string(b))
	}


	return 1
}
```

## Crash reports and fixing them



The crash reports are dumped in the workdir/crashes dir

This data becomes input for your automated test cases. 

Crash report files contain the binary input, or quoted ASCII input (.quoted) 
that triggered the crash/timeout. They also contain the stack trace report 
(.output files).

The next step is to start a new branch from master, take the crash output and 
write a test for it. 

See test/guerrilla_test.go for functions that start with TestFuzz... for 
example. Run the test to see if you get a crash / hang, then fix the test and 
submit a pull request.

Tip: In case of binary input, then perhaps it can be base64 encoded if you 
want to include with your test case. Eg, to encode a binary file:

`$ cat workdir/crashers/21c56f89989d19c3bbbd81b288b2dae9e6dd2150 | base64 > encoded.data.txt`

