# Bash to Basics

I was inspired to brush up on my bash by Adam Drake's [Command Line Tools Can Be 235x Faster Than Your Hadoop Cluster](https://adamdrake.com/command-line-tools-can-be-235x-faster-than-your-hadoop-cluster.html). Here lies the fruit of my exploration.

---

## Portability

To ensure that your bash script can run in environments other than your own make sure this is the first line of your script:

```
#!/usr/bin/env bash
```
This "shebang" tells UNIX to use the bash binary from the local user's environment.

## Interoperability

In UNIX everything is a file and text streams are the common interface. To read from STDIN is to read `/dev/stdin`. This is what it means to read from a pipe.

For example the script `pipe_echo.sh`:

```
#!/usr/bin/env bash
cat /dev/stdin
```

It will promptly read from STDIN and write to STDOUT.

```
> echo "Hello, world." | ./pipe_echo.sh
"Hello, world."
```

We may want to read from a specified file if given as an argument. Bash gives us a concise way to express this without resorting to writing conditional logic.

```
#!/usr/bin/env bash
cat ${1:-/dev/stdin}
```

This will read a file matching the first passed argument (`$1`) if it exists otherwise it will read from `/dev/stdin`. Now our script will easily compose with other UNIX tools.

## Concurrency

Bash gives us elegant concurrency primitives for writing scripts with the capacity to run in parallel. We can easily pipe the output of one program to two and vise versa.

For example the script `fan_in_fan_out.sh`:

```
#!/usr/bin/env bash
cat ${1:-/dev/stdin} | { grep "left filter"; grep "filter right" } | sort -u
```

The two `grep` statements enclosed in curly braces and separated by a semicolon will both read the output of `cat`. The output of each of these `grep` commands will in turn be read by the `sort` command.

## Subshell Writing

Writing to both STDOUT and another file is simple with the `tee` command. You can even filter the text stream in the background before writing to a file.

Let's modify `fan_in_fan_out.sh` to print the filtered text to stdout and write the sorted deduped text to a file:

```
#!/usr/bin/env bash
cat ${1:-/dev/stdin} | {
  grep "left filter"
  grep "filter right"
} |
tee >(sort -u > "sorted.txt")
```

Writing the output of `tee` to a subshell enables us to create a pipeline of commands to further filter and transform the text stream before writing to a file.

## Subshell Reading

Some commands will expect files as arguments instead of text streams. In these cases you can read from a subshell to pass command output.

For example the `comm` command takes two files and outputs the lines that are common as well as unique to each.

```
comm -23 <(./new_jokes.sh) "jokes.txt"
```

The result will be the lines generated by the new_jokes.sh script absent jokes.txt.

---

Exploring how to redirect output and interchange files for text streams has deepened my appreciation for the standards for inputs and outputs. Following them enable your scripts to compose well with others.

UNIX is a joy when you're a good UNIX citizen.