# Performance testing
========================

## 使用工具和以及关键参数设置
- [wrk](https://github.com/wg/wrk)
- sudo sysctl -w kern.ipc.somaxconn=2048
- sudo sysctl -w kern.maxfiles=12288
- ulimit -n 10000
- httpserver pool size 1024
- redis pool size 1024
- mysql maxopen 512


## 200 并发,使用固定用户登录
    wrk -t8 -c200 -d30s --latency -s  ./scripts/fixed_user.lua http://localhost:8080/login
    
output:

    Running 30s test @ http://localhost:8080/login
    8 threads and 200 connections
    Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency    34.94ms   13.26ms 142.92ms   71.22%
        Req/Sec   721.55     98.78     1.08k    75.92%
    Latency Distribution
        50%   34.53ms
        75%   42.94ms
        90%   50.85ms
        99%   67.51ms
    172511 requests in 30.03s, 62.35MB read
    Requests/sec:   5744.11
    Transfer/sec:      2.08MB

## 2000 并发，使用固定用户登录
    wrk -t8 -c2000 -d30s --latency -s  ./scripts/fixed_user.lua http://localhost:8080/login

output:

    Running 30s test @ http://localhost:8080/login
    8 threads and 2000 connections
    Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency   178.65ms   62.03ms 481.58ms   68.11%
        Req/Sec   751.64    127.53     1.20k    69.36%
    Latency Distribution
        50%  184.36ms
        75%  223.30ms
        90%  252.65ms
        99%  309.27ms
    179157 requests in 30.06s, 63.12MB read
    Socket errors: connect 0, read 57, write 0, timeout 17
    Requests/sec:   5959.22
    Transfer/sec:      2.10MB

## 200 并发，使用随机用户登录
    wrk -t8 -c200 -d30s --latency -s  ./scripts/random_user.lua http://localhost:8080/login

output:

    Running 30s test @ http://localhost:8080/login
    8 threads and 200 connections
    Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency    75.39ms   71.92ms 452.22ms   85.67%
        Req/Sec   436.19    268.55     0.87k    43.90%
    Latency Distribution
        50%   46.09ms
        75%   73.13ms
        90%  192.97ms
        99%  303.36ms
    104031 requests in 30.09s, 30.93MB read
    Requests/sec:   3457.63
    Transfer/sec:      1.03MB


## 2000 并发，使用随机用户登录
    wrk -t8 -c2000 -d30s --latency -s  ./scripts/random_user.lua http://localhost:8080/login
output:

    Running 30s test @ http://localhost:8080/login
    8 threads and 2000 connections
    Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency   387.87ms  279.93ms   2.00s    85.06%
        Req/Sec   540.89    332.22     1.11k    55.94%
    Latency Distribution
        50%  345.02ms
        75%  464.87ms
        90%  575.38ms
        99%    1.65s 
    127496 requests in 30.09s, 25.03MB read
    Socket errors: connect 0, read 41, write 0, timeout 4937
    Non-2xx or 3xx responses: 23354
    Requests/sec:   4236.84
    Transfer/sec:    851.82KB
