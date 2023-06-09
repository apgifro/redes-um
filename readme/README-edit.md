# Serviços de Redes

## Imagens

![Moodle](/readme/images/Moodle.png)

## Sobre

Configurações e passo a passo para a prova de Serviços de Redes I.

Ferramentas e serviços:

- VirtualBox `7.0` + Debian `11 Bullseye`
- DHCP `isc-dhcp-server` (dns1)
- DNS `bind9` (dns1 e dns2)
- Apache `apache2` (web)
- FTP + NFS (storage)

## Configurar

### Iniciando

1. Baixe o Debian standart, por torrent, neste [link](https://cdimage.debian.org/debian-cd/current-live/amd64/bt-hybrid/debian-live-11.7.0-amd64-standard.iso.torrent). Julgo essa versão mais prática de ser instalada, já que é live e, portanto, inclui todos os pacotes necessários já na própria ISO.

2. Configure a máquina virtual, sem esquecer de adicionar corretamente três placas de rede:

```
1. NAT
2. Rede Interna ("prova")
3. Host Only
```

3. Não esqueça de adicionar `Host Only` no `VirtualBox` com o endereço `192.168.56.1`.

4. Inicie a máquina virtual com a ISO, siga o que é pedido e instale o Debian. 

5. Faça quatro vezes um clone completo.

6. Não esqueça de ajustar as placas de rede para `Rede Interna`.

7. Adicione seu usuário ao `sudoers`, para que se possa usar `sudo`:

```
$ su -
$ usermod -aG sudo username
$ su - username
```

8. Atualize o `/etc/hostname`.

5. Ative o serviço de rede:

```
$ sudo mv /etc/network/interfaces /etc/network/interfaces.save
$ systemctl enable systemd-networkd
```

3. Adicione cada arquivo de rede dentro das pastas `network/` de cada `servers/` em `/etc/systemd/network`. Não esqueça dos `/etc/resolv.conf`. Atualize com `sudo reboot`.

4. Instale o SSH com `sudo apt install openssh-server`.

### Configurando o Gateway

1. Adicione e execute o script `nat.sh`:

```
$ sudo chmod +x nat.sh
$ sudo ./nat.sh
```

2. Verifique as regras com `sudo iptables -t nat -L`.

### Configurando o DNS Primário

1. Instale o DHCP com `sudo apt install isc-dhcp-server`.

2. Atualize `/etc/default/isc-dhcp-server` para usar IPv4 com [esta configuração](servers/dns1/dhcp/isc-dhcp-server).

3. Adicione as configurações em `/etc/dhcp/dhcpd.conf` com [esta configuração](servers/dns1/dhcp/dhcpd.conf).

4. Reinicie o serviço com `sudo systemctl restart isc-dhcp-server`.

5. Instale o DNS com `sudo apt install bind9`.

6. Adicione os arquivos do DNS [daqui](/servers/dns1/dns) em `/etc/bind` e faça as alterações necessárias.

7. As chaves presentes no [named.conf.local](/servers/dns1/dns/named.conf.local) são para correto funcionamento do DNS secundário (configurado na sequência) com a utilização de Views/ACLs. As Views foram utilizadas para entregar um endereço diferente para os servidores `receberão (172.16.1.x)` e a máquina hospedeira `receberá (apenas 192.168.56.2)`. O [nat.sh](servers/gateway/firewall/nat.sh) dará o caminho certo para o hospedeiro acessar os servidores. O propósito é que o hospedeiro, que seria a simulação de um cliente na Internet, conheça através do DNS apenas o servidor Gateway `(192.168.56.2)`, que encaminhará cada serviço, respectivamente, SSH (porta 22), DNS (porta 53) e principalmente o Apache/Web (porta 443 e 80) para os servidores internos. O cliente externo não precisa conhecer as regras de DNS da rede interna / dos servidores. Ao estar na rede externa, o principal serviço acessível é o Web, que entregará os sites configurados depois.

8. Para gerar a chave, utilize os seguintes comandos e atualize a chave gerada no [named.conf.local](servers/dns1/dns/named.conf.local):

```
tsig-keygen -a HMAC-MD5 chaveinterna
tsig-keygen -a HMAC-MD5 chaveexterna
```

9. Reinicie o serviço com `sudo systemctl restart bind9`.

10. Verifique o status com `sudo systemctl status bind9`.

10. Caso aconteça algum erro, entre como superusuário e veja os logs:

```
rndc querylog
journalctl
```

### Configurando o DNS Secundário

### Configurando o Web / Apache

### Configurando o Storage

1. Instale o FTP com `sudo apt install proftpd`.

2. Edite as configurações em `/etc/proftpd/proftpd.conf`:

```
ServerName  "Server_Aula"
DefaultRoot ~
```

3. Reinicie o serviço `sudo systemctl restart proftpd`

4. Teste pelo terminal, pode ser necessário instalar o pacote `ftp`, com `ftp storage.laboratorio.lan` ou use o Filezilla.

5. Caso ocorra um erro na `obtenção de pastas` no Filezilla, verifique:

```
1. Com o FileZilla aberto clique no menu "Editar" depois em "Configurações";
2. Na tela que se abrir, procure e clique na opção "Conexão" depois em "FTP";
3. Em "Modo de Transferência" selecione a opção "Ativo";
4. Clique em "OK" e reinicie o FileZilla;
```


