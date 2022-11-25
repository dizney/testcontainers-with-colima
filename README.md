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

Output from running the tests:

```shell
> ./gradlew -i test
....
    16:04:23.414 [Test worker] INFO  🐳 [postgres:latest] -- Creating container for image: postgres:latest
    16:04:23.431 [Test worker] INFO  🐳 [postgres:latest] -- Container postgres:latest is starting: 78cc212eff618afa526b3297f7cf49ad412bd4c0039482791dedfc750ab453eb
    16:04:24.321 [Test worker] INFO  🐳 [postgres:latest] -- Container postgres:latest started in PT0.918404S

Gradle Test Executor 16 finished executing tests.

> Task :test FAILED

MainTest > connect_to_postgres_container() FAILED
    java.lang.RuntimeException: org.postgresql.util.PSQLException: Connection to localhost:49184 refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.
        at org.example.MainTest.connect_to_postgres_container(MainTest.java:35)

        Caused by:
        org.postgresql.util.PSQLException: Connection to localhost:49184 refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.
            at app//org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:319)
            at app//org.postgresql.core.ConnectionFactory.openConnection(ConnectionFactory.java:49)
            at app//org.postgresql.jdbc.PgConnection.<init>(PgConnection.java:247)
            at app//org.postgresql.Driver.makeConnection(Driver.java:434)
            at app//org.postgresql.Driver.connect(Driver.java:291)
            at platform/java.sql@17.0.5/java.sql.DriverManager.getConnection(DriverManager.java:681)
            at platform/java.sql@17.0.5/java.sql.DriverManager.getConnection(DriverManager.java:190)
            at app//org.example.MainTest.connect_to_postgres_container(MainTest.java:32)

            Caused by:
            java.net.ConnectException: Connection refused
                at java.base/sun.nio.ch.Net.pollConnect(Native Method)
                at java.base/sun.nio.ch.Net.pollConnectNow(Net.java:672)
                at java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:549)
                at java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:597)
                at java.base/java.net.SocksSocketImpl.connect(SocksSocketImpl.java:327)
                at java.base/java.net.Socket.connect(Socket.java:633)
                at org.postgresql.core.PGStream.createSocket(PGStream.java:241)
                at org.postgresql.core.PGStream.<init>(PGStream.java:98)
                at org.postgresql.core.v3.ConnectionFactoryImpl.tryConnect(ConnectionFactoryImpl.java:109)
                at org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:235)
                ... 7 more

1 test completed, 1 failed
```

One workaround for this is to run colima with an ip address assigned by using the --network-address option when starting
colima. See [this FAQ entry](https://github.com/abiosoft/colima/blob/main/docs/FAQ.md#enable-reachable-ip-address) for
more information.
Note that this does not work in the current released (as of nov 2022) version of Colima if you use Macos Ventura, so 
you will need the HEAD version of Colima. See [issue 458](https://github.com/abiosoft/colima/issues/458).

You also need to set the `TESTCONTAINERS_HOST_OVERRIDE` to `$(colima ls -j | jq -r '.address')` so that testcontainers
knows how to reach a container. See also [this comment](https://github.com/testcontainers/testcontainers-java/issues/5034#issuecomment-1319812252). 
