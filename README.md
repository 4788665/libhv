[![Build Status](https://travis-ci.org/ithewei/libhv.svg?branch=master)](https://travis-ci.org/ithewei/libhv)
[![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20Windows%20%7C%20Mac-blue)](.travis.yml)

## Intro [中文版](readme_cn.md)

Like `libevent, libev, and libuv`,
`libhv` provides event-loop with non-blocking IO and timer,
but simpler api and richer protocols.

## Features

- Cross-platform (Linux, Windows, Mac, Solaris)
- EventLoop (IO, timer, idle)
- TCP/UDP client/server
- SSL/TLS support: WITH_OPENSSL or WITH_MBEDTLS
- HTTP client/server (include https http1/x http2 grpc)
- HTTP file service, indexof service, api service (support RESTful)
- WebSocket client/server

## Build

see [BUILD.md](BUILD.md)

Makefile:
```shell
./configure
make
sudo make install
```

or cmake:
```shell
mkdir build
cd build
cmake ..
cmake --build .
```

or vcpkg:
```shell
vcpkg install libhv
```

or xmake:
```shell
xrepo install libhv
```

## Getting Started

run `./getting_started.sh`:

```shell
git clone https://github.com/ithewei/libhv.git
cd libhv
make

bin/httpd -h
bin/httpd -d
#bin/httpd -c etc/httpd.conf -s restart -d
ps aux | grep httpd

# http file service
bin/curl -v localhost:8080

# http indexof service
bin/curl -v localhost:8080/downloads/

# http api service
bin/curl -v localhost:8080/ping
bin/curl -v localhost:8080/echo -d "hello,world!"
bin/curl -v localhost:8080/query?page_no=1\&page_size=10
bin/curl -v localhost:8080/kv   -H "Content-Type:application/x-www-form-urlencoded" -d 'user=admin&pswd=123456'
bin/curl -v localhost:8080/json -H "Content-Type:application/json" -d '{"user":"admin","pswd":"123456"}'
bin/curl -v localhost:8080/form -F "user=admin pswd=123456"
bin/curl -v localhost:8080/upload -F "file=@LICENSE"

bin/curl -v localhost:8080/test -H "Content-Type:application/x-www-form-urlencoded" -d 'bool=1&int=123&float=3.14&string=hello'
bin/curl -v localhost:8080/test -H "Content-Type:application/json" -d '{"bool":true,"int":123,"float":3.14,"string":"hello"}'
bin/curl -v localhost:8080/test -F 'bool=1 int=123 float=3.14 string=hello'
# RESTful API: /group/:group_name/user/:user_id
bin/curl -v -X DELETE localhost:8080/group/test/user/123
```

### HTTP
#### http server
see [examples/http_server_test.cpp](examples/http_server_test.cpp)
```c++
#include "HttpServer.h"

int main() {
    HttpService router;
    router.GET("/ping", [](HttpRequest* req, HttpResponse* resp) {
        return resp->String("pong");
    });

    router.GET("/data", [](HttpRequest* req, HttpResponse* resp) {
        static char data[] = "0123456789";
        return resp->Data(data, 10);
    });

    router.GET("/paths", [&router](HttpRequest* req, HttpResponse* resp) {
        return resp->Json(router.Paths());
    });

    router.POST("/echo", [](HttpRequest* req, HttpResponse* resp) {
        resp->content_type = req->content_type;
        resp->body = req->body;
        return 200;
    });

    http_server_t server;
    server.port = 8080;
    server.service = &router;
    http_server_run(&server);
    return 0;
}
```
#### http client
see [examples/http_client_test.cpp](examples/http_client_test.cpp)
```c++
#include "requests.h"

int main() {
    auto resp = requests::get("http://www.example.com");
    if (resp == NULL) {
        printf("request failed!\n");
    } else {
        printf("%d %s\r\n", resp->status_code, resp->status_message());
        printf("%s\n", resp->body.c_str());
    }

    resp = requests::post("127.0.0.1:8080/echo", "hello,world!");
    if (resp == NULL) {
        printf("request failed!\n");
    } else {
        printf("%d %s\r\n", resp->status_code, resp->status_message());
        printf("%s\n", resp->body.c_str());
    }

    return 0;
}
```

#### benchmark
```shell
# webbench (linux only)
make webbench
bin/webbench -c 2 -t 10 http://127.0.0.1:8080/
bin/webbench -k -c 2 -t 10 http://127.0.0.1:8080/

# sudo apt install apache2-utils
ab -c 100 -n 100000 http://127.0.0.1:8080/

# sudo apt install wrk
wrk -c 100 -t 4 -d 10s http://127.0.0.1:8080/
```

**libhv(port:8080) vs nginx(port:80)**
![libhv-vs-nginx.png](html/downloads/libhv-vs-nginx.png)

## Examples
### c version
- [examples/hloop_test.c](examples/hloop_test.c)
- [examples/tcp_echo_server.c](examples/tcp_echo_server.c)
- [examples/tcp_chat_server.c](examples/tcp_chat_server.c)
- [examples/tcp_proxy_server.c](examples/tcp_proxy_server.c)
- [examples/udp_echo_server.c](examples/udp_echo_server.c)
- [examples/nc.c](examples/nc.c)

### c++ version
- [evpp/EventLoop_test.cpp](evpp/EventLoop_test.cpp)
- [evpp/EventLoopThread_test.cpp](evpp/EventLoopThread_test.cpp)
- [evpp/EventLoopThreadPool_test.cpp](evpp/EventLoopThreadPool_test.cpp)
- [evpp/TcpServer_test.cpp](evpp/TcpServer_test.cpp)
- [evpp/TcpClient_test.cpp](evpp/TcpClient_test.cpp)
- [evpp/UdpServer_test.cpp](evpp/UdpServer_test.cpp)
- [evpp/UdpClient_test.cpp](evpp/UdpClient_test.cpp)
- [examples/http_server_test.cpp](examples/http_server_test.cpp)
- [examples/http_client_test.cpp](examples/http_client_test.cpp)
- [examples/websocket_server_test.cpp](examples/websocket_server_test.cpp)
- [examples/websocket_client_test.cpp](examples/websocket_client_test.cpp)

## echo-servers
```shell
cd echo-servers
./build.sh
./benchmark.sh
```

**echo-servers/benchmark**<br>
![echo-servers](html/downloads/echo-servers.png)
