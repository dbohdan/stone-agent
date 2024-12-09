# stone-agent

This repository houses **stone-agent**, a crude version of
[ssh-agent](https://en.wikipedia.org/wiki/Ssh-agent)
or
[gpg-agent](https://www.gnupg.org/documentation/manuals/gnupg/Invoking-GPG_002dAGENT.html)
made by a third party for
[age](https://github.com/FiloSottile/age).
The agent only performs decryption.
It is meant as a proof of concept for password managers based on age and shell.
Bugs and security vulnerabilities are expected.

stone-agent operates as follows:

- On launch, the agent stores an age private key in memory.
  The key stays in memory until the agent shuts down.
- The agent opens a Unix domain socket.
  When there is a connection, it reads the incoming data and sends it back decrypted using age.
- When the client closes the socket, the agent closes its end and reopens the socket to wait for a new connection.

The agent passes the key to `age -d -i` with Bash process substitution.
It does not report errors to the user accessing it directly through the socket or through `stone-agent decrypt`.

## Requirements

- Bash (chosen over POSIX shell for the process substitution)
- [pkill(1)](https://manpages.debian.org/bookworm/procps/pkill.1.en.html)
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
$ nc -NU "$XDG_RUNTIME_DIR"/stone-agent/socket < hello.age
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
