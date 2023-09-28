### Adicione as portas do consul no firewall
https://developer.hashicorp.com/consul/tutorials/production-deploy/reference-architecture#network-connectivity

```sh
sudo vim /etc/iptables/rules.v4 
```
```sh
-A INPUT -p tcp --dport  8300:8302 -j ACCEPT
-A INPUT -p udp --dport  8301:8302 -j ACCEPT
-A INPUT -p tcp --dport  8500:8502 -j ACCEPT
-A INPUT -p tcp --dport  8600 -j ACCEPT
-A INPUT -p udp --dport  8600 -j ACCEPT
-A INPUT -p tcp --dport  21000:21255  -j ACCEPT
```
```sh
sudo iptables-restore < /etc/iptables/rules.v4 
```