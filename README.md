# put - Yet another stream editor

`put` is a stream editor with a limited subset of `sed`'s functionalities at a fraction of the complexity, exposed through a modern CLI; it is designed to be used as a shell filter, with readability and simplicity in mind.

## Installation

To install `put` you need to compile it from sources; follow the [Golang](https.//www.golang.org) installation instructions, then clone this repository under your `$GOPATH/src` and compile it, as follows:

```bash
$ > cd $GOPATH/src
$ > mkdir -p github.com/dihedron
$ > cd github.com/dihedron
$ > git clone https://github.com/dihedron/put.git
$ > go install github.com/dihedron/put
```

If you added `$GOPATH/bin` to your `$PATH`, the command will be readily available, otherwise you have to copy it to some place where it will be accessible.

The application can be compiled on all Golang-supported OSs, including most flavours of *nix and Windows (the latter I have not tested, though); it does not require configuration or INI files, libraries, DLLs or Registry keys and can be placed anywhere on your filesystem or on portable USB sticks.

If you want to compress it to save space, you can safely use [UPX](https://upx.github.io/) to do it, like this:

```bash
$ > upx --brute $GOPATH/bin/put
```

## Usage

`put` works on text files, which are read and processed line by line; it supports the following line operations on lines that match a regular expression:

1. replace the line with other text
2. insert a line with some text before the matching line 
3. insert a line with some text after the matching line
4. delete the matching line 
   
Moreover it can be used to insert or drop a line of text at a given (0-based) index. 

`put` is meant to be used like this:

```bash
$ > cat infile.txt | 
        put "some replacement text" where "^a\s*line$" | 
        put --once "{1} but not least" before "^and this is the (last)$" |
        put "some text" after "^yet\s+another\s*line$" |
        put nil where "^a\s+line\s*to drop$"  
        > out.txt 
```

The example above shows 4 types of operations:

1. replace lines matching a pattern with other text (`where` clause with replacement text);
2. add a line before each line matching a pattern (`before` clause);
3. add a line after each line matching a pattern (`after` clause);
4. drop lines matching a pattern (`where` clause with `nil` as replacement text).

The `--once` flags indicates that the operaton should be performed only against the first occurrence of a matching line; if omitted, each matching line is "edited" as instructed.

Replacement text can include substitution anchors, such as the `{1}` in the example above; it will be substituted with the value of the first capturing group in the user-provided pattern (regular expression); if you write a regular expression matching the whole line (e.g. ending with `.*$`) the zero-th anchor (`{0}`) represents the whole expression, and the following (`{1}`, `{2}`...) each a pair of capturing brackets (`(...)`). To check how `put` interprets your regular expression, see [Debugging](#debugging) below.

Last, it can be used to insert or drop a line of text at a specified 0-based index, e.g.

```bash
$ > cat infile.txt | 
        put "# This comment is inserted between the first and the (old) second line" at 1
```

To drop a line, use `nil` as the text to insert:

```bash
$ > cat infile.txt | 
        put nil at 0
```

will drop the first line. `nil` can also be used with pattern matching, e.g.:

```bash
$ > cat infile.txt | 
        put nil where "^\s*#.*$"
```

will remove all bash-style comments from the file.

## Example

As an example let's take an `/etc/hosts`; say you want to add the host name on the `localhost` line (the one starting with `127.0.0.1`) to prevent complaints by your `sudo` commands. The following sequence copies the matching line to a comment (first invocation of `put`), then replaces whatever is after the `localhost` word with the current hostname:

```bash
$ > cat hosts | 
        put "# {0} (changed on $(date +%Y/%m/%d))" before "^(127\.0\.0\.1\s+localhost).*" | 
        put "{1} $(hostname)" where "^(127\.0\.0\.1\s+localhost).*" 
        > hosts2
```

so that the following original file:

```
127.0.0.1	localhost
127.0.1.1	myhost.example.com	myhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
``` 

is turned into the following:

```
# 127.0.0.1	localhost (changed on 2018/06/18)
127.0.0.1	localhost myhost
127.0.1.1	myhost.example.com	myhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Please note the original line is saved as a comment __before__ the changed line.

If you want to prepend some text to the file, you can do the following: 

```bash
$ > cat hosts |
        put "# NOTE: THIS FILE WAS AUTOMATICALLY EDITED:" at 0 |
        put "# ALL CHANGES WILL BE OVERWITTEN: DO NOT MODIFY!" at 1 |
        put "{1} $(hostname)" where "^(127\.0\.0\.1\s+localhost).*" 
        > hosts2
```

## Debugging

If you want to see what the command is doing internally, simply run it with the `PUT_DEBUG` environment variable set to one of `debug`, `info`, `warning` or `error`, e.g. as follows:

```bash
$ > PUT_DEBUG=debug put [args] < /etc/hosts
```

This can be helpful also if you need to take a look at the available bindings for your regular expression (`{0}`, `{1}`...) in order to write the correct replacement expression. 

## Suggestions and contributions

... are very welcome: please open an issue to give a feedback or to offer bug-fixes and enhancements via a pull request.
