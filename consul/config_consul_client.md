# Configurando Client Consul
## [Instalação](./Install.md)
## Diretótórios para armazenar os certificados
Transfira o certificado CA do servidor consul para os clientes
```sh
scp consul-agent-ca.pem -i < ssh-key.pem > ubuntu@cliente:/tmp 
```
```sh
sudo chown --recursive consul:consul /etc/consul.d
sudo chmod 640 /etc/consul.d/consul.hcl
sudo mkdir /etc/consul.d/certs
sudo mv /tmp/consul-agent-ca.pem /etc/consul.d/certs

```

## Configure o Consul client

O arquivo  `/etc/consul.d/consul.hcl` será utilizado para definir as configurações dos servidores e **agentes** do consul.

```sh
cd /etc/consul.d/
sudo mv consul.hcl consul.hcl.bkp
sudo vim consul.hcl
```
```conf
datacenter = "dc1"

data_dir = "/opt/consul"

encrypt = "rBBGBaiwJAkp8On365dMdHNCML4A6ysLQ0IlGDmr5oE="

tls {
   defaults {
      ca_file = "/etc/consul.d/certs/consul-agent-ca.pem" 
   
      verify_incoming = true 
      verify_outgoing = true
   }
   internal_rpc {

      verify_server_hostname = true 
   }
}
auto_encrypt {
  tls = true
}


server = false

retry_join = ["10.186.0.41"]

```
Para um arquivo de configuração com comentários [aqui](./config_consul_client.md).

**OBS** : [Liberar portas no iptables](iptables_rules.md)

Validar o fucionamento do consul 
```sh
sudo systemctl start consul
sudo systemctl enable consul
sudo systemctl status consul
```