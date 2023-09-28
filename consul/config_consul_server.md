# Configurando Consul Server
## Requisitos 
https://developer.hashicorp.com/consul/tutorials/production-deploy/reference-architecture

## Gerando chave de encripitação gossip 
[Documentação](https://developer.hashicorp.com/consul/tutorials/production-deploy/deployment-guide#prepare-the-security-credentials)

```sh
consul keygen
```
```sh
khSlWCTeg1alCvE5IOj1a+fnfcXYZD32lpOkkt+YenU=
```
Armazene a chave gerada em local segura para uso posterior na opção `encrypt` da congiguração em todos os clientes e servidores consul.

Mais sobre comunicação gossip no consul neste [link](https://developer.hashicorp.com/consul/tutorials/security/gossip-encryption-secure)

## Gerando Certificado TLS para encripitação RPC
Consul pode usar TLS para verificar a autenticade de servidores e clientes e para isto todos os agentes deve ter certificados assinados por uma  CA.

Mais informações cobre certificados TLS no consul [aqui](https://developer.hashicorp.com/consul/tutorials/security/tls-encryption-secure)

**Avançado:** [Vault as Consul service mesh certification authority](https://developer.hashicorp.com/consul/tutorials/vault-secure/vault-pki-consul-connect-ca)

### Criando autoridade certificadora
Iniciar a CA que será usado para assinar os certificados TLS dos servidores e agentes. 
```sh
consul tls create
```
```sh
├── consul-agent-ca-key.pem
└── consul-agent-ca.pem
```

### Criando certificados TLS (Datacenter Single Consul)
Crie os certificados para cada um dos servidores consul existentes. O nome dos arquivos serão nomeados de forma incremental.
```sh
consul tls cert create -server -dc dc1 -domain consul
```
**OBS**: As opçẽos `-dc` define o nome do datacenter e `-domain`, o nome do domínio. No exemplo estamos usando os valores default, que podem ser omitidos `consul tls cert create -server` 

```sh
==> WARNING: Server Certificates grants authority to become a
    server and access all state in the cluster including root keys
    and all ACL tokens. Do not distribute them to production hosts
    that are not server nodes. Store them as securely as CA keys.
==> Using consul-agent-ca.pem and consul-agent-ca-key.pem
==> Saved dc1-server-consul-0.pem
==> Saved dc1-server-consul-0-key.pem
```
Criar diretório para armazenar os certificados 
```sh
sudo chown --recursive consul:consul /etc/consul.d
sudo chmod 640 /etc/consul.d/consul.hcl
sudo mkdir /etc/consul.d/certs
sudo mv *.pem /etc/consul.d/certs
```
### Configuração do Consul Server
O arquivo  `/etc/consul.d/consul.hcl` será utilizado para definir as configurações dos servidores e agentes do consul.

```sh
cd /etc/consul.d/
sudo vim consul.hcl
```

```conf
datacenter = "dc1"
data_dir = "/opt/consul"
encrypt = "rBBGBaiwJAkp8On365dMdHNCML4A6ysLQ0IlGDmr5oE="
tls {
   defaults {
      ca_file = "/etc/consul.d/certs/consul-agent-ca.pem" 
      cert_file = "/etc/consul.d/certs/dc1-server-consul-0.pem" 
      key_file = "/etc/consul.d/certs/dc1-server-consul-0-key.pem"
      verify_incoming = true 
      verify_outgoing = true
   }
   internal_rpc {
      verify_server_hostname = true 
   }
}
auto_encrypt {
  allow_tls = true
}
client_addr = "0.0.0.0"
ui_config{
  enabled = true
}
server = true
bind_addr = "0.0.0.0" 
bootstrap_expect=1
retry_join = ["10.186.0.41"]
connect {
  enabled = true
}
addresses {
  grpc = "127.0.0.1"
}
ports {
  grpc_tls  = 8502
}
```
Para um arquivo de configuração com comentários [aqui](./config_consul_server.md).

**OBS** : 
- [Liberar portas no iptables](iptables_rules.md)

  


Acesso ao UI do consul use `IP:8500` 