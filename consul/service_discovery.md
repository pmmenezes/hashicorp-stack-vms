# Deploy da aplicação para os testes.

## Deploy Backend

Aplicação de exemplo

```sh
cd ~ 
curl -LO https://github.com/consul-up/birdwatcher/releases/download/v1.0.0/backend-linux-amd64 
sudo mv backend-linux-amd64 /usr/local/bin/backend
sudo chmod +x /usr/local/bin/backend
```
```sh
sudo vim  /etc/systemd/system/backend.service

```

```conf
[Unit]
Description="Backend service"
Requires=network-online.target
After=network-online.target
[Service]
ExecStart=/usr/local/bin/backend
Restart=on-failure
# We will set the backend service to listen
# on port 7000.
Environment=BIND_ADDR=0.0.0.0:7000
[Install]
WantedBy=multi-user.target
```

**OBS**: Caso preciso adicione regra no iptables para liberar acesso ao serviço na porta 7000
```sh
sudo vim /etc/iptables/rules.v4 
```
Adicione a liberação.
```sh
-A INPUT -p tcp --dport  7000 -j ACCEPT
```
```sh
sudo iptables-restore < /etc/iptables/rules.v4 
```

Inicie o Backend

```sh
sudo systemctl start  backend
sudo systemctl enable  backend
sudo systemctl status  backend
```

Use o comando `ss -tunelp` para conferir o backend ouvindo na porta 7000
```sh
tcp      LISTEN    0         4096                         *:7000                    *:*        ino:54304 sk:f cgroup:/system.slice/backend.service v6only:0 <
```


## Delpoy do Frontend


Aplicação de exemplo

```sh
cd ~ 
curl -LO https://github.com/consul-up/birdwatcher/releases/download/v1.0.0/frontend-linux-amd64 
sudo mv frontend-linux-amd64 /usr/local/bin/frontend
sudo chmod +x /usr/local/bin/frontend
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
Environment=BACKEND_URL=http://*IP-BACKEND*:7000
# The Install section configures this service to start
# automatically if the VM reboots.
[Install]
WantedBy=multi-user.target
```


Inicie o Frontend

```sh
sudo systemctl start  frontend
sudo systemctl enable  frontend
sudo systemctl status  frontend
```

Use o comando `ss -tunelp` para conferir o backend ouvindo na porta 7000
```sh
cp     LISTEN   0        4096                      *:6060                *:*      ino:51997 sk:c cgroup:/system.slice/frontend.service v6only:0 <->   
```

**OBS**: Caso preciso adicione regra no iptables para liberar acesso ao serviço na porta 7000
```sh
sudo vim /etc/iptables/rules.v4 
```
Adicione a liberação.
```sh
-A INPUT -p tcp --dport  6060 -j ACCEPT
```
```sh
sudo iptables-restore < /etc/iptables/rules.v4 
```

Teste o funcionamento `http://IP-FRONTEND:6060`.

# Service Discovery
## Registro dos serviços no consul

### Backend

```sh
sudo vim /etc/consul.d/backend.hcl
```

```conf
service {
    name = "backend"
    # backend runs on port 7000.
    port = 7000
    meta {
        version = "v1"
    }
}
```
```sh
consul reload 
```
### Frontend

```sh
sudo vim /etc/consul.d/frontend.hcl
```

```conf
service {
    name = "frontend"
# frontend runs on port 6060.
    port = 6060
}
```

## ConfigurandoEncaminhamento para DNS serviço de descoberta de serviço do consul

Até este ponto o comando `nslookup frontend.service.consul`  não consegue resolver os nomde dos serviços. 
```sh 
Server:		127.0.0.53
Address:	127.0.0.53#53

** server can't find frontend.service.consul: NXDOMAIN
```
Você pode usar o serviço resolvido pelo systemd para resolver nomes de aplicativos locais em sua rede. Para usar o serviço, configure o systemd-resolved para enviar consultas de domínio .consul ao Consul criando o arquivo consul.conf localizado no diretório /etc/systemd/resolved.conf.d/.

```sh
sudo mkdir /etc/systemd/resolved.conf.d/
sudo vim /etc/systemd/resolved.conf.d/consul.conf
```

```conf
[Resolve]
DNS=127.0.0.1:8600
DNSSEC=false
Domains=~consul
```
```sh
sudo systemctl restart systemd-resolved
```

Ajuste o frontend para acesso ao backend via nome do serviço.

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
Environment=BACKEND_URL=http://backend.service.consul:7000
# The Install section configures this service to start
# automatically if the VM reboots.
[Install]
WantedBy=multi-user.target

```
```sh
sudo systemctl daemon-reload
sudo systemctl restart frontend.service
sudo systemctl status frontend.service
```

