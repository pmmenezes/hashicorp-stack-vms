# Instalação do Consul

## Ubuntu/Debian

### Instalando requisitos
```sh
sudo apt update && sudo apt upgrade -y
sudo apt install curl gnupg lsb-release
```

### Adicionando chave gpg do repositŕio da Hasicorp

```sh
curl --fail --silent --show-error --location https://apt.releases.hashicorp.com/gpg | \
      gpg --dearmor | \
      sudo dd of=/usr/share/keyrings/hashicorp-archive-keyring.gpg
```

### Adicionando Repositório da Hashicorp
```sh
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
 sudo tee -a /etc/apt/sources.list.d/hashicorp.list
```
### Selecione a versão do consul 

```sh
sudo apt update
sudo apt-cache policy consul | tac
```
```sh
1.15.5-1 500
        500 https://apt.releases.hashicorp.com jammy/main amd64 Packages
     1.15.6-1 500
        500 https://apt.releases.hashicorp.com jammy/main amd64 Packages
     1.16.0-1 500
        100 /var/lib/dpkg/status
        500 https://apt.releases.hashicorp.com jammy/main amd64 Packages
 *** 1.16.1-1 500
        500 https://apt.releases.hashicorp.com jammy/main amd64 Packages
     1.16.2-1 500
  Version table:
  Candidate: 1.16.2-1
```

```sh
sudo apt install consul=1.16.2-1
```

### Verifique a instalação
```sh
consul version
consul help
```

### Diretório de configuração do consul 

```sh
/etc/consul.d/
├── consul.env
└── consul.hcl
```

### Pŕoximos Passos
- [Configurando Consul Server](./config_consul_server.md)
- [Configurando Consul Client](./config_consul_client.md)