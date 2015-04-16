Cron
====

This formula does not perform any action.  It exists only to add useful functions to the Wick environment.


Functions
---------

### cron-add

Add a job to cron.  Takes additional arguments and passes them to `wick-make-file` from [wick-base].

    cron-add [WICK_MAKE_FILE_ARGS] JOB_NAME FILE

* `WICK_MAKE_FILE_ARGS`: Additional arguments as understood by `wick-make-file` from [wick-base].  These arguments can be placed anywhere in the argument list.
* `JOB_NAME`: Name of the cron job to create.  For best results, try to avoid characters that make regular expressions hard or ones that do not work well as filenames.  For example, `*super* job!` is a bad name, `super-job` is far better.
* `FILE`: File to use for the cron job.  If `--template` is used, this can be a [template]. This might be placed into a new file or appended to a list of cron jobs in one file, so it is best to avoid setting environment variables.

File contents need to have the following fields for a job definition, in order:  minute, hour, day, day of month, month, day of week, username, command.  For more information, see `man 5 crontab` and remember that these are system jobs so they require an account name.

Example:

    # Adds a job from files/delete_files.cron
    cron-add delete_files delete_files.cron


### cron-remove

Removes a job from cron.  Works only for jobs that were added with `cron-add`.

    cron-remove JOB_NAME

* `JOB_NAME`: Name of the cron job to remove.

Example:

    # Add and remove a cron job
    cron-add update-mongo-v2 --template update-mongo.mo
    cron-remove update-mongo-v1

### formula-template

Process a formula's template file.  Automatically detects the template engine by the filename extension.  Writes results to stdout.  (See [Bash concepts] for stdout; [templates] about the template system.)

This is what `wick-make-file` may use when processing templates.

    formula-template TEMPLATE

* `TEMPLATE`: Name of template file inside the formula's `templates/` folder.

Example:

    formula-template my-template-file.sh > /tmp/the-result


### wick-make-dir

Creates a directory on the target machine.

*Only the last folder will have its ownership changed.*  See the examples for further information.

    wick-make-dir [--mode=MODE] [--owner=OWNER] PATH

* `[--mode=MODE]`: Specify a mode for the directory using `chmod` syntax.  When specified, the mode is always set, even if the directory already existed.  Optional; does not change the mode unless the option is set.
* `[--owner=OWNER]`:  Designate an owner and possibly a group for the directory using `chown` syntax.  When specified, the ownership is always set even if the directory already existed.  Optional; does not change the ownership unless the option is set.
* `PATH`: The directory to create.  Uses `mkdir -p` so all parents directories would be created if they do not exist.  Ownership and mode is only changed on the specified path, not all parents.

Examples:

    # Creates /etc/consul.d (/etc already existed) with the mode 0755
    # and make consul the owner.
    wick-make-dir --mode=0755 --owner=consul:consul /etc/consul.d

    # Creates a folder named /a/b/c/d/ and changes the ownership of
    # /a/b/c/d/ to nobody:nogroup.  NOTE: All of the parent directories
    # will be created automatically if they didn't already exist and
    # they will be owned by root:root, NOT nobody:nogroup.
    wick-make-dir --owner nobody:nogroup /a/b/c/d/


### wick-make-file

Copies a file from the formula to the target machine.  Files are stored in `files/` within the formula.  When `--template` is used, then files come from `templates/` instead.  (See [templates] for more about templates.)

*It is good practice to have the destination either include the filename or else end in a slash (`/`) to avoid ambiguity.  See the examples for more information.*

    wick-make-file [--mode=MODE] [--owner=OWNER] [--template] FILE DESTINATION

* `[--mode=MODE]`: Specify a mode for the file using `chmod` syntax.  Optional; does not change the mode unless the option is set.
* `[--owner=OWNER]`:  Designate an owner and possibly a group for the file using `chown` syntax.  Optional; does not change the ownership unless the option is set.
* `[--template]`: Switch to using a template as the source.  (See [templates].)
* `FILE`: The source file to use.  It must be in `files/` for normal files or `templates/` when using the template engine.
* `DESTINATION`: The file to create or overwrite.  This file is always written, even if it exists.  Does not check to make sure the directory exists first.

If the destination is a directory, the file will keep its original name and be copied to the directory.  When using templates, the template engine suffix will be stripped off.

If you want the destination directory to be created automatically, make sure you use a `/` at the end of the directory name.  It will be created with the installed file's owner and a default mode.  See the notes for the `wick-make-dir` function regarding ownership of directories.

Examples:

    # Writes /etc/rc.local from a template.
    wick-make-file --mode=0755 --template rc.local.mo /etc/

    # Writes a root-only configuration file, renaming it as it is written.
    wick-make-file --mode=0600 --owner=root:root secret.txt /root/super-secret-key.txt

    # Installs a file into a directory that does not exist.
    # The directory will be created and owned by "nobody", just like the file.
    # Note the / at the end of the destination
    wick-make-file --mode=600 --owner=nobody:nobody config.ini /etc/a/b/c/d/

    # The same as above but the directory will NOT be created automatically
    # if it doesn't already exist.
    # If /etc/a/b/c/d exists as a directory then config.ini will be written
    # to that folder.  Otherwise, this will create the file /etc/a/b/c/d (not
    # /etc/a/b/c/d/config.ini), so be careful.
    wick-make-file --mode=600 --owner=nobody:nobody config.ini /etc/a/b/c/d


### wick-hash

Return a hash for a file.  The type of hash returned is based on what's available on the system.  You can use this to see if files change their contents.

    wick-hash DESTINATION FILENAME

* `DESTINATION`: Name of environment variable that will get the resulting hash value
* `FILENAME`: File to hash.


### wick-package

Install or remove packages on the target system.  This handles the OS-specific tools that are used to install or remove the packages.  If the package is named differently on various systems, it is up to the formula to address that (see the [apache2] formula).

    wick-package [--uninstall] PACKAGE [...]

* `[--uninstall]`: Remove the packages instead of installing it.
* `PACKAGE`: Name of package to install

Example:

    wick-package --uninstall apache
    wick-package apache2


### wick-service

Control services.  Add services, enable and disable them at boot up.  Start, stop, reload, restart services.  Helps with the creation of override files (for `chkconfig`).

    wick-service COMMAND SERVICE

* `COMMAND`: Action to perform.
* `SERVICE`: Service name.

Actions:

* `add [--force] SERVICE FORMULA_FILE` - Use `wick-make-file` to copy the formula file to `/etc/init.d/` for the named service.  Does not enable nor start the service.  Does not add the service if the file already exists unless `--force` is also used.
* `disable SERVICE` - Disable the service from starting at boot.  Does not stop the service if it is already running.
* `enable SERVICE` - Enable the service at boot.  Does not start the service.
* `make-override [--force] SERVICE` - Creates `/etc/chkconfig.d/SERVICE` that is used by `chkconfig` to help determine order.  This override file can be modified to list additional dependencies to influence the boot order of scripts.  Make sure to call `wick-service override` when you update an override file.  When using `--force`, this will overwrite any override file that may already exist.
* `override SERVICE` - Calls `chkconfig override` to apply any changes made to override files.
* `reload SERVICE` - Reloads the given service.
* `restart SERVICE` - Stops and starts the given service.
* `start SERVICE` - Starts the service.
* `stop SERVICE` - Stops the service.

Example:

    # Creating a new service using the formula's files/consul.init
    # and copying it to the right spot.
    wick-service add consul consul.init
    wick-service enable consul
    wick-service start consul

    # Make Apache require Consul before starting
    wick-service make-override apache2
    sed -i 's/Required-Start:/Required-Start: consul/' /etc/chkconfig.d/apache2
    wick-service override apache2


### wick-set-config-line

Adds or updates a line in a config file.  This is a very basic tool that ensures a line exists in a file, not that it is in any particular order.

    wick-set-config-line FILE LINE [KEY]

* `FILE`: File to update.
* `LINE`: Line to add.  This line will be placed at the end.
* `KEY`: The "key" that we are setting.  Optional and defaults to a portion of `LINE`.

The line is not repeatedly added to the config file.  First, we attempt to get the "key" for the line, either automatically or use a value that is passed in.  Most config files use a key value of some sort and this script usually can detect them - more on this later.  Next, we remove any lines with the same key from the file and finally we append the line you want onto the file.

The key can be automatically detected.  It is anything in LINE that is to the left of a space, colon, or equals.  If you are having difficulty understanding what is used as the key, check out the examples.  I have listed what `wick-set-config-line` uses as the key.

This is not suitable for updating shell scripts and other similar-looking files.  For instance, if you tried to modify `/etc/rc.local` and add two lines (`bash /script1` and `bash /script2`) then only one would ever exist in the file because "bash" would be considered the key.

Example:

    # This wipes out any previous settings for 127.0.0.1 and replaces them.
    # Key is "127.0.0.1"
    wick-set-config-line /etc/hosts "127.0.0.1 localhost some-funky-name"

    # Set the bind IP for mongo
    # Detects the current IP using `wick-get-iface-ip`
    # Key is "bind_ip"
    wick-set-config-line /etc/mongod.conf "bind_ip=$(wick-get-iface-ip)"

    # Update cloud-init
    # Key is "preserve_hostname"
    wick-set-config-line /etc/cloud/cloud.cfg "preserve_hostname: true"

    # Update DHCP settings
    # Key is "prepend nameservers" because we specify it as a third
    # parameter.
    wick-set-config-line /etc/dhcp/dhclient.conf \
        "prepend nameservers 127.0.0.1" "prepend nameservers"


[Apache]: ../apache2/README.md
[Bash concepts]: ../../doc/bash-concepts.md
[Formulas]: ../../formulas/README.md
[template]: ../../doc/templates.md
[wick-base]: ../wick-base/README.md