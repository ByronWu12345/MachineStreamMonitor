# MachineStreamGateway

---

## This project use the package of Ocelot to route requests

```
"Routes": [
    {
      "DownstreamPathTemplate": "/ws",
      "DownstreamScheme": "ws",
      "DownstreamHostAndPorts": [
        {
          "Host": "192.168.1.65",
          "Port": 8008
        }
      ],
      "UpstreamPathTemplate": "/ws",
      "UpstreamHttpMethod": [ "GET", "POST", "PUT", "DELETE", "OPTIONS" ]
    },
    {
      "DownstreamPathTemplate": "/api/MachineStream",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "192.168.1.65",
          "Port": 8080
        }
      ],
      "UpstreamPathTemplate": "/MachineStream",
      "UpstreamHttpMethod": [ "Get", "POST" ]
    },
    {
      "DownstreamPathTemplate": "/api/MachineStream/{id}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "192.168.1.65",
          "Port": 8080
        }
      ],
      "UpstreamPathTemplate": "/MachineStream/{id}",
      "UpstreamHttpMethod": [ "Get", "POST", "PUT" ]
    }
  ]
```

