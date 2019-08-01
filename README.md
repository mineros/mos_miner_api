# minerOS specification of miner API

revision:  v0.1.0
last modified: 2019/07/29

I. Motivation
-------------------------

For a long time, many users of minerOS complained about the same issue that when they switched miner from one algorithm to another, they had to restart the miner process. Most of the time, the only way to stop a miner process is to send SIGTERM or SIGKILL. But most softwares cannot quit gracefully when any single GPU runs to its extreme margin, such as overclocked or overloaded, as a consequence the miner process will enter uninterruptible sleep state, aka "D state". Once the miner process enters into D state, GPU may never function as normal.

To solve this issue and deliver a better experience, we introduce a guideline for miner applications to implement their management API. Developers of miner can either modify their current API implementation or develop a new API for minerOS use only. You can add a new command line option, and keep your legacy api port at the same time, e.g.  --api-port 3333 --mineros-api-port 3334. minerOS can then invoke this new management port, 3334, to call miner functions. For security consideration, minerOS can also set password to this api port, but this is not a "MUST" requirement.


II. Specification
-------------------------

* graceful stop miner

request
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "miner.stop",
  "params": {
    "secret": "x"                 // secret, optional
  }
}
```

response
none

* get miner stats

request
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "miner.stats",
  "params": {
    "secret": "x"                 // secret, optional
  }
}
```

response
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "version": "a.b.c",           // miner version
    "time": 3600,                 // running time, in seconds
    "primary": {
      "pool": "eth.f2pool.com:8008",
      "hashrate": 123.00,
      "shares": {
        "accept": 456.00,
        "reject": 78.00,
        "invalid": 0.00,          // invalid share, optional
      },
      "gpu": [{
        "id": "#0",
        "hashrate": 123.00,
        "shares": {
          ...
        },
        "temperature": 53.00,     // gpu temperature, optional
        "fan": 71.00,             // gpu fan speed, optional
        "power": 90.00,           // power usage, optinal
        "error": ""               // error message, optional
      }]
    },
    "secondary": {                // dual mining status, if any
                                  // optional
      ...
    }
  }
}
```

* restart miner (optional)

request
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "miner.restart",
  "params": {
    "secret": "x"                 // secret, optional
    "devices": "0,1,2",           // available GPUs
                                  // use all GPU if missing or empty
                                  // optional
    "primary": {
      "algorithm": "ethash",
      "pools": [{                 // more than 1 pool
                                  // if miner supports pool failover
        "url": "stratum+tcp://eth.f2pool.com:8008",
        "username": "0xabcdefg",
        "password": "x",
        "worker": "worker"
      }]
    },
    "secondary": {                // dual mining options, if any
                                  // optional
      ...
    },
    "extra": ""                   // extra args
                                  // if miner has special options
  }
}
```

response
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": "ok",
  "error": {
    "code": -32600,
    "message": "Invalid Request"
  }
}
```

* error codes

|       code       |     message      |
| ---------------- | ---------------- |
| -32700           | Parse error      |
| -32600           | Invalid Request  |
| -32601           | Method not found |
| -32602           | Invalid params   |
| -32603           | Internal error   |
| -32000 to -32099 | Server error     |

