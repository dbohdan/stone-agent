# stone-agent

This repository contains a crude decryption agent for
[age](https://github.com/FiloSottile/age)
inspired by
[ssh-agent](https://en.wikipedia.org/wiki/Ssh-agent)
and
[gpg-agent](https://www.gnupg.org/documentation/manuals/gnupg/Invoking-GPG_002dAGENT.html).
It is meant as a proof of concept for password managers based on age and shell.
Bugs and security vulnerabilities are to be expected.

The agent uses a private key given on launch to decrypt anything sent through a Unix domain socket.
It keeps the private key in memory and passes it to `age -d -i` using Bash process substitution.
It does not report errors to the socket user.

## Requirements

- Bash (chosen over POSIX shell for the process substitution)
- [socat(1)](https://manpages.debian.org/bookworm/socat/socat.1.en.html)

## Demo

```none
$ age-keygen -o test.key
$ echo 'Hello, world!' | age -r "$(age-keygen -y test.key)" > hello.age
$ file hello.age
hello.age: age encrypted file, X25519 recipient
$ STONE_AGENT_KEY=$(tail -n 1 test.key) ./stone-agent listen &
[1] 713750
$ ./stone-agent decrypt < hello.age
Hello, world!
$ nc -NU "$XDG_RUNTIME_DIR"/stone-agent < hello.age
Hello, world!
```

## Usage

```none
Usage: STONE_AGENT_KEY=AGE-SECRET-KEY-1... stone-agent listen [unix-socket]
       stone-agent decrypt [unix-socket] < encrypted.age
       STONE_AGENT_KEY=AGE-SECRET-KEY-1... stone-agent pipe < encrypted.age

Commands:
  listen
          Start agent
  decrypt
          Decrypt standard input to standard output by connecting to running
agent
  pipe
          Decrypt standard input to standard output by running age. (Used
internally by 'listen'.)

Arguments:
  [unix-socket]
          Unix domain socket to listen on or connect to. If the argument
doesn't include a path, the directory for the socket is chosen automatically.

Options:
  -h, --help
          Print this help message and exit

  -V, --version
          Print version number and exit
```

## License

MIT.
