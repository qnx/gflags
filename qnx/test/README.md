# Testing gflags on QNX

gflags has three types of tests: regular tests, NC tests and config tests (please see INSTALL.md for more information).

NC tests and config tests test the compilation process itself, by calling a python script which compiles different sample gflags programs. The python script calls cmake in a subprocess. hence most of our qnx cmake configurations/toolchain will not propagate to this compilation process. As a result, it will try to use the default toolchain on the host machine, which means that it will fail when it tries to links the libraries compiled using qnx toolchain. Luckily, there are only a handful of NC and config tests, so we do not worry about them too much.

Most of the tests fall in the category of regular tests. They are designed to be invoked using ctest. All but one of them are invoked using the macro `add_gflags_test(name expected_rc expected_output unexpected_output cmd)`. Which executes the cmd (with some default args appended to it) and compare the output with expected output. Since we do not have cmake on the target, we simulate this behavior in `qnxtest.sh` using function `add_gflags_test`. It should behave in the same way as the `add_gflags_test` cmake macro, except that it cannot handle regex string inputs. But we are lucky that there is no real regex involved in gflag tests.

Only the shared libraries and `/usr/bin/gflags_tests` are really needed for the test. But to be consistent with other osg repos, we still offer the makqnximage option to include the whole source tree.

# Running the Test Suite
Compile the gflags source for the desired architecture, e.g.

    OSLIST=nto CPULIST=x86_64 make -C qnx/build install

Then build your QNX image using mkqnximage and the following options:

    # <repo_path> is where the code was checked out e.g. /mnt/dev/gflags
    mkdir test_image
    cd test_image
    mkqnximage --extra-dirs=<repo_path>/qnx/test/mkqnximage --clean --run --force --test-gflags=<repo_path>

Once the target has booted, the gflags source tree will be located in /data/gflags:

    # cd /data/gflags/qnx/test
    # ./qnxtest.sh
    ...
    TEST: version-overrides-fromenv
    running command: ./gflags_unittest --test_tmpdir=. --srcdir=. --fromenv=test_bool,version,ok 2>&1
    rc: 0
    QNXPASS: expected rc = 0. got rc = 0
    QNXPASS: "gflags_unittest" in out
    QNXPASS: "/gflags_unittest.cc:" in out

    TEST: dashdash
    running command: ./gflags_unittest --test_tmpdir=. --srcdir=. -- --help 2>&1
    rc: 0
    QNXPASS: expected rc = 0. got rc = 0
    QNXPASS: "PASS" in out

    TEST: always_fail
    running command: ./gflags_unittest --test_tmpdir=. --srcdir=. --always_fail 2>&1
    rc: 1
    QNXPASS: expected rc = 1. got rc = 1
    QNXPASS: "ERROR: failed validation of new value 'true' for flag 'always_fail'" in out
    # echo $?       
    0
    ...
