---
{"dg-publish":true,"permalink":"/0-notes/mit-missing-semester/","noteIcon":"","created":"2024-01-27T07:45:29.549+01:00","updated":"2024-01-27T07:48:22.810+01:00"}
---

## Lecture 1

```bash
mkdir missing && cd missing
touch semester
# ./semester does not work: not an executable
# sh ./semester gets as the input file: sh is an interpreter which can execute the bash script
chmod 755 ./semester
./semester

# Output:
# HTTP/2 200
# server: GitHub.com
# content-type: text/html; charset=utf-8
# last-modified: Sat, 06 May 2023 11:21:52 GMT
# access-control-allow-origin: *
# etag: "64563850-1f86"
# expires: Tue, 06 Jun 2023 05:27:22 GMT
# cache-control: max-age=600
# x-proxy-cache: MISS
# x-github-request-id: 3E8E:11118:3490D78:3655F67:647EC161
# accept-ranges: bytes
# date: Tue, 06 Jun 2023 05:56:11 GMT
# via: 1.1 varnish
# age: 0
# x-served-by: cache-osl6534-OSL
# x-cache: HIT
# x-cache-hits: 1
# x-timer: S1686030971.232251,VS0,VE117
# vary: Accept-Encoding
# x-fastly-request-id: 538052f7759f52d5035537ef6e9865c74f7d423e
# content-length: 8070

./semester | grep "last.*modified" | cut -d : f2-
#Output:
# Sat, 06 May 2023 11:21:52 GMT

# -d : f2-
# the input is split by the specified delimiter
# which turns into a list
# 2- means the second element until the last in the list

# Question:
# why grep "last.*modified.*" does not work? new line character?

./semester | grep "last.*modified" | cut -d : f2- > last_modified.txt


```

----
## Lecture 2

### Takeaway
* whitespace in bash scripts indicates for argument splitting
    ```bash
    foo = var
    # foo program with the arguments = and var
    ```
* ' and "
    * ` in bash scripts is used for string literals which doesn't support variable substitution
    * " in bash scripts supports variable substitution
    ```bash
    foo="var"
    echo $foo
    fooChild="$foo_child"
    echo $foo
    ```
* special variables ([cheat list](https://tldp.org/LDP/abs/html/special-chars.html))
    ```bash
    $0          # name of the script
    $1 - $9     # arguments to the script
    $@          # all the arguments
    $#          # number of arguments
    ```
* `stdout` and `stderr`
* return code / exit status
* `&&` and `||` are short-circuiting (seems to be supported in most modern programming lanuages?)
* `;` accommodate multiple commands in the single line
* command substitution: `$(CMD)`
    ```bash
    $(ls)
    # substitute the \$(ls) with the output
    ```
* process substitution: `<(CMD)`
    ```bash
    # <(CMD) execute the command and place the output in a temporary file
    # and substitute the <(CMD) with that file name 

    # assume there are two sub-dir
    diff <(ls bash) <(ls lecture2)
    ```
* redirect `stdout` and `stderr` to null register
    ```bash
    command > /dev/null 2> /dev/null
    ```
* comparisons in bash
    ```bash
    if [[ $? -ne 0 ]]
    ```
    - prefer to use `[[]]`: [rationale](http://mywiki.wooledge.org/BashFAQ/031) #TODO
* shell globbing
    ```bash
    mv *{.py,.sh} folder
    # mv all *.py and *.sh files

    mkdir foo bar
    touch {foo,bar}/{a..h}
    # create files foo/a, foo/b, ..., foo/h, bar/a, bar/b, ..., bar/h
    ```
* `shellcheck` [bash syntax check](https://github.com/koalaman/shellcheck)
* `shebang` and `env`: `#! usr/bin/env bash`
* bash functions are executed in the current shell environment (environment variables modifiable)
* bash scripts are executed in their own process *(Scripts will be passed by value environment variables that have been exported using export???)*
* `brew install tldr`: simplified `man`
* finding files: `find` (alternative: [fd](https://github.com/sharkdp/fd#installation))
* finding files: `locate` (index)
* finding code: `grep` (alternative: `ack`, `rg`)
* finding shell commands: `histor`, `command + R`, `fzf`
* directory navigation: `fasd`, `autojump`
* directory structure: `tree`, `broot`

### Example
- print all input arguments
    ```bash
    #!/bin/bash

    # print all input arguments
    num=$#
    echo "Total number of input arguments: $num"
    for arg in $@; do
        echo "$arg"
    done
    ```

- print date time and pid
    ```bash
    #!/bin/bash

    echo "Starting program at $(date)"
    echo "Running program $0 with $# arguments with PID $$"
    ```

- lecture example
    ```bash
    #!/bin/bash

    echo "Starting program at $(date)"
    echo "Running program $0 with $# arguments with PID $$"

    # iterate all input file name
    for file in "$@"; do
        grep foobar "$file" > /dev/null 2> /dev/null
        # redirect the stdout to the null register
        # redirect the stderr to the null register

        if [[ $? -ne 0 ]]; then
            echo "File $file does not have any foobar, adding one"
            echo "# foobar" >> "$file"
        fi
    done
    ```

- find
    ```bash
    # find all file with name *.py and save paths into txt file
    find ~ -name '*.py' -type f >> find_result.txt

    # find all files modified in the late day
    find ~ -mtime -1

    # find all zip files with size in range 500k to 10m
    find ~ -size +500k -size -10M -name '*.tar.gz'

    mkdir files
    touch files/file_{a..h}.txt
    # Delete all files with .txt extension under files
    find ./files -name '*.txt' -exec rm {}; \;
    ```

### Exercise
#### HW1
```bash
# Includes all files, including hidden files
ls -a
# Sizes are listed in human readable format (e.g. 454M instead of 454279954)
ls -lhs
# Files are ordered by recency
ls -t
# Output is colorized
ls --color=auto
# ls command
ls --color=auto -lahst
```
#### HW2
```bash
#!/bin/bash

# History file to read directory path from
histFilePath="$HOME"
currDirHistFile=".hist"

# Read the directory saved in .hist file
# if the first record does not exist anymore, continue reading
# if no record exists, throw the errors and terminate the function
function polo() {
    tail -r "$histFilePath/$currDirHistFile" | while read -r dirAtTop
    do
        if [[ -d "$dirAtTop" ]]; then
            echo "Entering $dirAtTop"
            break;
        fi
    done

    if [[ $? -ne 0  ]]; then
        echo "Last entered directory does not exist anymore..."
        return 1
    fi

    return 0
}


# Run...
polo
```
```bash
#!/bin/bash

# History file to save directory path in
histFilePath="$HOME"
currDirHistFile=".hist"

function marco() {
    # Append the current directory into the history file
    echo "$PWD" >> "$histFilePath/$currDirHistFile"
}

# Run...
marco
```
- usage
    - `source macro.sh` in the directory to be recorded
    - `source polo.sh` to recover the last entered directory which still exists
- drawback
    - `.hist` file name and path should be maintained in two files
    - not a good idea to do `I/O` manipulation in such a simple program
    - if the top record doesn't exist anymore, it should be removed (require more `I/O` manipulation)

- **Takeaway**
    - `./polo.sh` doesn't enter the directory: bash script is run in a `subshell`, which can't change the parent shell working directory and its side-effects are lost when it finishes
    - `source polo.sh` and `. polo.sh`: run the script in the current shell environment

---
