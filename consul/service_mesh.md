# Configurando Service Mesh

## Instalar evoy 
```sh
sudo apt install hashicorp-envoy 
```
## Configurar service mesh no backnd Service

```sh
sudo vim   /etc/consul.d/backend.hc
```

```conf
service {
    name = "backend"
    # backend runs on port 7000.
    port = 7000
    meta {
        version = "v1"
    }
    # The backend service doesn't call
    # any other services so it doesn't
    # need an "upstreams" stanza.
    #
    # The connect stanza is still required to
    # indicate that it needs a sidecar proxy.
    connect {
        sidecar_service {
        # backend's proxy will listen on port 21000.
        port = 21000
        }
    }
}
```
```sh
consul reload
```

```sh
sudo vim /etc/systemd/system/backend-sidecar-proxy.service
```

```conf
[Unit]
Description="Backend sidecar proxy service"
Requires=network-online.target
After=network-online.target
[Service]
ExecStart=/usr/bin/consul connect proxy -sidecar-for backend 
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

```sh
sudo systemctl start backend-sidecar-proxy.service
sudo systemctl enable backend-sidecar-proxy.service
sudo systemctl status backend-sidecar-proxy.service
```

## Configurar service mesh no Frontend Service

```sh
sudo vim   /etc/consul.d/frontend.hcl
```

```conf
service {
    name = "frontend"
# frontend runs on port 6060.
    port = 6060
# The "connect" stanza configures service mesh
# features.
    connect {
        sidecar_service {
        # frontend's proxy will listen on port 21000.
            port = 21000
            proxy {
            # The "upstreams" stanza configures
            # which ports the sidecar proxy will expose
            # and what services they'll route to.
            upstreams = [
                {
                # Here you're configuring the sidecar proxy to
                # proxy port 6001 to the backend service.
                destination_name = "backend"
                local_bind_port = 6001
                }
            ]
        }
    }
  }
}

```

```sh
consul reload
```

```sh
sudo vim /etc/systemd/system/frontend-sidecar-proxy.service
```

```conf
[Unit]
Description="Frontend sidecar proxy service"
Requires=network-online.target
After=network-online.target
[Service]
ExecStart=/usr/bin/consul connect proxy -sidecar-for frontend 
Restart=on-failure
[Install]
WantedBy=multi-user.target
```
```sh
sudo systemctl start frontend-sidecar-proxy.service
sudo systemctl enable frontend-sidecar-proxy.service
sudo systemctl status frontend-sidecar-proxy.service
```

```sh
sudo vim  /etc/systemd/system/frontend.service
```

```conf
[Unit]
Description="Frontend service"
# The service requires the VM's network
# to be configured, e.g., an IP address has been assigned.
Requires=network-online.target
After=network-online.target
[Service]
# ExecStart is the command to run.
ExecStart=/usr/local/bin/frontend
# Restart configures the restart policy. In this case, we
# want to restart the service if it fails.
Restart=on-failure
# Environment sets environment variables.
# We will set the frontend service to listen
# on port 6060.
Environment=BIND_ADDR=0.0.0.0:6060
# We set BACKEND_URL to http://localhost:7000 because
# that's the port we'll run our backend service on.
Environment=BACKEND_URL=http://localhost:6001
# The Install section configures this service to start
# automatically if the VM reboots.
[Install]
WantedBy=multi-user.target
~                                  
```

```sh
sudo systemctl daemon-reload
sudo systemctl restart frontend.service
sudo systemctl status frontend.service
```
## ref
- https://developer.hashicorp.com/consul/docs/connect/proxies/envoy#supported-versions
- https://developer.hashicorp.com/consul/tutorials/developer-mesh/service-mesh-with-envoy-proxy?utm_source=docs
- [Encaminhamento para DNS serviço de descoberta de serviço do consu](https://developer.hashicorp.com/consul/tutorials/networking/dns-forwarding)
- https://developer.hashicorp.com/consul/commands/connect/envoy
- 
