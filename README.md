# twemperf (mcperf)

**twemperf** (pronounced "two-em-perf"), aka **mcperf** is a tool for
measuring memcached server performance. mcperf is like httperf, but for
memcached protocol. It speaks memcached ASCII protocol and is capable of
generating connections and requests at a high rate.

## Building mcperf ##

To build mcperf from distribution tarball:

    $ ./configure
    $ make
    $ sudo make install

To build mcperf from distribution tarball in debug mode:

    $ CFLAGS="-ggdb3 -O0" ./configure --enable-debug
    $ make
    $ sudo make install

To build mcperf from source in debug mode:

    $ git clone git://github.com/twitter/twemperf.git
    $ cd twemperf
    $ autoreconf -fvi
    $ CFLAGS="-ggdb3 -O0" ./configure --enable-debug
    $ make
    $ src/mcperf -h

## Design ##

1. Single threaded.
2. Asynchronous I/O through non-blocking sockets and Linux epoll(7) syscall.
3. Horizonal scaling through many concurrent mcperf processes on several
   different machines.

mcperf is composed of three main subsystems:

1. Core Engine
2. Load Generator
3. Stats Collector

The core engine is responsible for the event handling and parsing of
memcached protocol and drives the main event loop.

The load generator is resposible for generating load either periodically
or as a one-shot event. Each load generator is implemented as a self
contained module. Some examples of load generators are:

1. Connection generator creates connections to a server at a given rate.
2. Call generator issue calls (requests) on a connection at a given rate.
3. Size generator generates item sizes.

The stats collector is responsible for collecting statistics. Each stats
collector is implemented as a self contained module. Some examples of
stats collectors are:

1. Connection stats collects connection stats.
2. Call stats collects call (request and response) stats.

## Examples ##

The following example creates **1000 connections** to a memcached server
running on **localhost:11211**. The connections are created at the rate of
**1000 conns/sec** and on every connection it sends **10 'set' requests** at
the rate of **1000 reqs/sec** with the item sizes derived from a uniform
distribution in the interval of [1,16) bytes.

    $ mcperf --linger=0 --timeout=5 --conn-rate=1000 --call-rate=1000 --num-calls=10 --num-conns=1000 --sizes=u1,16

    Total: connections 1000 requests 10000 responses 10000 test-duration 1.009 s

    Connection rate: 991.1 conn/s (1.0 ms/conn <= 23 concurrent connections)
    Connection time [ms]: avg 10.3 min 10.1 max 14.1 stddev 0.1
    Connect time [ms]: avg 0.2 min 0.1 max 0.8 stddev 0.0

    Request rate: 9910.5 req/s (0.1 ms/req)
    Request size [B]: avg 35.9 min 28.0 max 44.0 stddev 4.8

    Response rate: 9910.5 rsp/s (0.1 ms/rsp)
    Response size [B]: avg 8.0 min 8.0 max 8.0 stddev 0.0
    Response time [ms]: avg 0.2 min 0.1 max 13.4 stddev 0.00
    Response time [ms]: p25 1.0 p50 1.0 p75 1.0
    Response time [ms]: p95 1.0 p99 1.0 p999 1.0
    Response type: stored 10000 not_stored 0 exists 0 not_found 0
    Response type: num 0 deleted 0 end 0 value 0
    Response type: error 0 client_error 0 server_error 0

    Errors: total 0 client-timo 0 socket-timo 0 connrefused 0 connreset 0
    Errors: fd-unavail 0 ftab-full 0 addrunavail 0 other 0

    CPU time [s]: user 0.64 system 0.35 (user 63.6% system 35.1% total 98.7%)
    Net I/O: bytes 428.7 KB rate 424.8 KB/s (3.5*10^6 bps)

The following example creates **100 connections** to a memcached server
running on **localhost:11211**. Every connection is created after the previous
connection is closed. On every connection we send **100 'set' requests** and
every request is created after we have received the response for the previous
request. All the set requests generated have a fixed item size of 1 byte.

    $ mcperf --linger=0 --call-rate=0 --num-calls=100 --conn-rate=0 --num-conns=100 --sizes=d1

    Total: connections 100 requests 10000 responses 10000 test-duration 1.268 s

    Connection rate: 78.9 conn/s (12.7 ms/conn <= 1 concurrent connections)
    Connection time [ms]: avg 12.7 min 12.6 max 13.5 stddev 0.1
    Connect time [ms]: avg 0.0 min 0.0 max 0.1 stddev 0.0

    Request rate: 7886.1 req/s (0.1 ms/req)
    Request size [B]: avg 28.0 min 28.0 max 28.0 stddev 0.0

    Response rate: 7886.1 rsp/s (0.1 ms/rsp)
    Response size [B]: avg 8.0 min 8.0 max 8.0 stddev 0.0
    Response time [ms]: avg 0.1 min 0.1 max 1.0 stddev 0.00
    Response time [ms]: p25 1.0 p50 1.0 p75 1.0
    Response time [ms]: p95 1.0 p99 1.0 p999 1.0
    Response type: stored 10000 not_stored 0 exists 0 not_found 0
    Response type: num 0 deleted 0 end 0 value 0
    Response type: error 0 client_error 0 server_error 0

    Errors: total 0 client-timo 0 socket-timo 0 connrefused 0 connreset 0
    Errors: fd-unavail 0 ftab-full 0 addrunavail 0 other 0

    CPU time [s]: user 0.51 system 0.75 (user 40.0% system 59.0% total 99.0%)
    Net I/O: bytes 351.6 KB rate 277.2 KB/s (2.3*10^6 bps)

## Contributors ##

* Manju Rajashekhar (@manju)
* Clojure Janet (@clojurejanet)
