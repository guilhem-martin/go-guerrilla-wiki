# go-guerrilla fuzz testing

For further improving the stability of the software, some simple fuzz testing was implemented.
This is making usage of the [go-fuzz](https://github.com/dvyukov/go-fuzz) package and can easily be performed.

## Preperations

### Branches

We do have an extra branch for this, as not everyone needs the extended code for just running the software.
Please checkout the branch *tests* for this:

`$ git checkout tests
$ git pull`

We try to keep the branch up to date with the *master* branch.

### go-fuzz

If you don’t have *go-fuzz* already installed, do that now.

`$ go get github.com/dvyukov/go-fuzz/go-fuzz
$ go get github.com/dvyukov/go-fuzz/go-fuzz-build`

Build the packages and you should get the *go-fuzz* as well as the *go-fuzz-build* tools.

### Corpus files

If you have pulled the recent *tests* branch you should now also have the *workdir* and within that
the *corpus* folder. This contains some files that are used to initate the fuzzing.
You are very welcome to create further corpus files, the ones we use are the example smtp files of the
go-fuzz package.

## Putting it together

After everything is prepared you can start. Build the package with *go-fuzz*:

`$ go-fuzz-build github.com/flashmob/go-guerrilla`

This will take a while and create a file named *guerrilla-fuzz.zip*
Now the fuzzing process itself can be started:

`$ go-fuzz -bin=guerrilla-fuzz.zip -workdir=workdir`

This will run for quite a while. Eventually you will get an output that contains *crashers*.

You can investigate on those crashes looking at the *.output* files in the *workdir/crashers* folder.



