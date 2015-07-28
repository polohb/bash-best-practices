# Bash best practices and style-guide

Just simple methods to keep the code clean.

Inspired by [progrium/bashstyle](https://github.com/progrium/bashstyle) and [Kfir Lavi post](http://www.kfirlavi.com/blog/2012/11/14/defensive-bash-programming/).


## Quick big rules
* All code goes in a function
* Always double quote variables
* Avoid global variables and declare them as `readonly`
* Always have a `main()` function for runnable scripts
* Always use `set -eo pipefail` : fail fast and be aware of exit codes
* Define functions as `myfunc() { ... }`, not `function myfun {...}`
* Always use `[[` instead of `[` or `test`
* Use `$( ... )` instead of backticks
* Prefer absolute paths and always qualify relative paths with `./.`
* Warnings and errors should go to `STDERR`, anything parsable should go to `STDOUT`
* Use `.sh` or `.bash` extension if file is meant to be included or sourced

## More specific rules with some example

### Global variables
* Avoid global vars
* Always UPPER_CASE naming
* Readonly declaration
* Globals that can be always use in any program :
```
readonly PROGNAME=$(basename $0)
readonly PROGDIR=$(readlink -m $(dirname $0))
readonly ARGS="$@"
```

### Other variables
* All variables should be local (they can only be used in functions)
* Always lowercase naming
* Self documenting parameters
```
fn_example() {
    local explicit_name = $1 ;
    local expName = $1 ;
}
```
* Usually use `i` for loop, so it is very important to declare it as local

### Main()
* Use always a `main()` function
* The only global command in the code is : `main` or `main "$@"`
* If script is also usable as library, call it using `[[ "$0" == "$BASH_SOURCE" ]] && main "$@"`



### Everything is a function
* Only the `main()` function and global declarations are run globaly
* Short code portion can be functions
* Define functions as `myfunc() { ... }`, not `function myfun {...}`


### Debugging
* Run with -x flag : `bash -x prog.sh`
* Debug just a small section of code using set -x and set +x
* Printing function name and its arguments `echo $FUNCNAME $@`

### Each line of code does just one thing
* Break expression with back slash `\`
* Use symbols at the start of the indented line
```
print_dir_if_not_empty() {
    local dir=$1
    is_empty $dir \
        && echo "dir is empty" \
        || echo "dir=$dir"
}
```


### Command line arguments

```
cmdline() {
    local arg=
    for arg
    do
        local delim=""
        case "$arg" in
            #translate --gnu-long-options to -g (short options)
            --config)         args="${args}-c ";;
            --pretend)        args="${args}-n ";;
            --test)           args="${args}-t ";;
            --help-config)    usage_config && exit 0;;
            --help)           args="${args}-h ";;
            --verbose)        args="${args}-v ";;
            --debug)          args="${args}-x ";;
            #pass through anything else
            *) [[ "${arg:0:1}" == "-" ]] || delim="\""
                args="${args}${delim}${arg}${delim} ";;
        esac
    done

    #Reset the positional parameters to the short options
    eval set -- $args

    while getopts "nvhxt:c:" OPTION
    do
         case $OPTION in
         v)
             readonly VERBOSE=1
             ;;
         h)
             usage
             exit 0
             ;;
         x)
             readonly DEBUG='-x'
             set -x
             ;;
         t)
             RUN_TESTS=$OPTARG
             verbose VINFO "Running tests"
             ;;
         c)
             readonly CONFIG_FILE=$OPTARG
             ;;
         n)
             readonly PRETEND=1
             ;;
        esac
    done

    if [[ $recursive_testing || -z $RUN_TESTS ]]; then
        [[ ! -f $CONFIG_FILE ]] \
            && eexit "You must provide --config file"
    fi
    return 0
}
```

### Unit Testing
* Very important in higher level languages
* Use [shunit2](https://shunit2.googlecode.com/svn/trunk/source/2.1/doc/shunit2.html) for unit testing
* Good intro to shunit2 : [shUnit2 - Bash Testing](http://www.mikewright.me/blog/2013/10/31/shunit2-bash-testing/)
* Another good ressource : [Test Driving Shell Scripts](http://code.tutsplus.com/tutorials/test-driving-shell-scripts--net-31487)

* The list of current assertions (as of version 2.1.6) :
  * `assertEquals [message] expected actual`
  * `assertSame [message] expected actual`
  * `assertNotEquals [message] expected actual`
  * `assertNotSame [message] expected actual`
  * `assertNull [message] value` # used to compare a null in bash which is a zero length string
  * `assertNotNull [message] value` # used to compare a null in bash which is a zero length string
  * `assertTrue [message] condition`
  * `assertFalse [message] condition`


* The list of current failures (do not use them for value comparisons, use assertions for this) :
  * `fail [message]`
  * `failNotEquals [message] unexpected actual`
  * `failSame [message] expected actual`
  * `failNotSame [message] unexpected actual`


* More specific functions :
  * `setUp` : run automatically before each test
  * `tearDown` run automatically after each test
  * `|| startSkipping` automatically skip after a test failure (default is to continue)

### Usefull links and  good references
* Obsolete and deprecated bash syntax :
[http://wiki.bash-hackers.org/scripting/obsolete](http://wiki.bash-hackers.org/scripting/obsolete)
* Beginner mistakes [http://wiki.bash-hackers.org/scripting/newbie_traps](http://wiki.bash-hackers.org/scripting/newbie_traps)
* Advanced Bash-Scripting Guide [http://tldp.org/LDP/abs/html/](http://tldp.org/LDP/abs/html/)
* Google's Bash styleguide [http://google-styleguide.googlecode.com/svn/trunk/shell.xml](http://google-styleguide.googlecode.com/svn/trunk/shell.xml)
