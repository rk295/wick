Wick-Init-D-Lib - Wick Formula
==============================

Not all systems are created equally.  In that regard, not all `/etc/init.d/` environments are the same either.  Instead of rewriting a shell that's extremely similar for multiple services, this formula will install a library that can be used to make init-style scripts quickly.

    wick-formula wick-init-d-lib

* There are no parameters.

There are many hooks that you can plug into in order to run commands right before/after startup, shutdown and reloads.  Just define a new function to override the old, empty one.

This is a sample `/etc/init.d` script that uses the library.

    #!/bin/bash

    # Source in the init.d library
    . /usr/local/bin/wick-init-d-lib

    # This is the one function that MUST exist.  It starts the process.
    # It takes two parameters: the log file and the PID file.
    process-start() {
        # For this example, the command doesn't fork
        # Append all stdout/stderr to the log file
        /usr/local/bin/some-command >> "$1" 2>&1 &

        # Disown, which is similar to using "nohup"
        disown %+

        # Write the PID file
        echo "$!" > "$2"
    }

    # Before we start this command, let's wipe out its temporary data
    start-before() {
        rm -rf /tmp/some-command/
    }

    # This part is crucial.  After you define all of the necessary functions,
    # you must call this one.  It handles the arguments and coordinates the
    # calling of the right functions.
    handle-command