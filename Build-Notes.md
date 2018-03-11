
### Build prerequisites

This project requires the following tools to build: GNU Make and Glide dependency manager.

The "Glide" package dependency manager can be obtained from https://glide.sh/

After cloning this project, one would typically run `$ glide install` and all dependencies will be resolved and placed in the /vendor directory (which is excluded from git)

GNU Make is available on most Linux distributions by default, at least where the C/C++ compilers are installed.
On Windows, GNU Make is [available through CygWin](https://stackoverflow.com/questions/16135945/is-there-a-cygwin-version-of-gnu-make).

Finally, to build, use:

`$ make guerrillad`

To run tests:

`$ make test`