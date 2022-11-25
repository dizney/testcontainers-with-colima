# Connection refused when using testcontainers together with colima

When using [Colima](https://github.com/abiosoft/colima) as docker engine instead of eg Docker desktop there seems to be
some
problems with connecting to containers, at least when eg
using [testcontainers](https://github.com/testcontainers/testcontainers-java).

First of all you need to do some configuration to let testcontainers know on how to connect to docker. You need to set
at least the following environment variables:

| Environment variable | Value |
|----------------------|-------|
| DOCKER_HOST          | unix://$HOME/.colima/default/docker.sock      |
| TESTCONTAINERS_DOCKER_SOCKET_OVERRIDE          |/var/run/docker.sock|

See [this testcontainers github issue](https://github.com/testcontainers/testcontainers-java/issues/5034) for more
information.

After setting these environment variables it mostly works. But for some reason, if you try to connect to a just started
Postgresql container you will get a connection refused. The code in this repository does just that. In a test it starts
up a postgresql container and when that is done it tries to setup a jdbc connection to that postgresql container. It only
works if you sleep for eg 2 seconds (in my testing eg 1 second sleep was not enough).

