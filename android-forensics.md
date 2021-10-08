# Forense em Android

## Dispositivo Bloqueado

Para começar a extração de dados de um dispositivo Android o mesmo tem que estar desbloqueado. Se ele estiver bloqueado você pode:

* Verificar se o dispositivo está com o modo debugging via USB ativado.
* Verificar por um possível [smudge attack](https://www.usenix.org/legacy/event/woot10/tech/full_papers/Aviv.pdf)
* Tentar um ataque de [força bruta](https://www.cultofmac.com/316532/this-brute-force-device-can-crack-any-iphones-pin-code/)

## Aquisição de Dados

Crie um [backup android usando o adb](mobile-apps-pentesting/android-app-pentesting/adb-commands.md#backup) e extraia ele usando o [Android Backup Extractor](https://sourceforge.net/projects/adbextractor/): `java -jar abe.jar unpack arquivo.backup arquivo.tar`

### Acesso root ou conexão física com uma interface JTAG

* `cat /proc/partitions` \(procure o caminho para a memória flash, geralmente a primeira entrada é _mmcblk0_ e corresponde a toda memória flash\).
* `df /data` \(descubra o tamanho do bloco do sistema\).
* `dd if=/dev/block/mmcblk0 of=/sdcard/blk0.img bs=4096` \(execute-o com as informações coletadas do tamanho do bloco\).

### Memória

Use o Linux Memory Extractor \(LiME\) para extrair a informação da RAM. É uma extensão do kernel que deve ser carregada via adb.