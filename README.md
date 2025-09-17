Основная статья на GitHUB: [machine-to-proxmox-lxc-ct-converter](https://github.com/my5t3ry/machine-to-proxmox-lxc-ct-converter)


Выполнять конвертацию на “железных” серверах ProxMox.
<br/>

# Последовательность:

### Посмотреть ostype вашего ProxMox сервера:
```
man pct | grep -A 10 "ostype"
```

### Склонировать репу на Proxmox host сервер и выдать права на исполнение скрипту:

Переходим в директорию: **`/usr/local/bin/ConvertVMtoCT/machine-to-proxmox-lxc-ct-converter`**
```plaintext
git clone https://github.com/my5t3ry/machine-to-proxmox-lxc-ct-converter.git
cd machine-to-proxmox-lxc-ct-converter
chmod +x convert.sh bashconvert
```

> ### Проверить VID виртуалок и контейнеров 
> Используй скрипты по пути **~/scripts/**
{.is-warning}


### Запустить скрипт от root:
```plaintext
./convert.sh
```
```plaintext
./bashconvert
```

Пример 1:
```plaintext
./bashconvert \
--name astra-template \
--target 192.168.87.103 \
--port 22 \
--id 196 \
--root-size 20 \
--ip 192.168.87.130 \
--bridge vmbr0 \
--gateway 192.168.87.2 \
--memory 4096 \
--disk-storage ssd_1tb \
--password runtelorg
```

Пример 2:
```bash
./bashconvert_redos \
--name redos8-template \
--target 192.168.87.44 --port 22 --id 888 \
--root-size 20 --ip 192.168.87.131 \
--bridge vmbr0 --gateway 192.168.87.2 \
--memory 4096 --disk-storage ssd_1tb \
--password runtelorg
```

Првоерка контейнера:
```bash
pct status 196
pct config 196
pct exec 196 ip addr
pct enter 196
pct reboot 196
```
<br/>


# Convert Astra1.7 , RedOS7/8, ALT11

### Делаем backup'ы release файлов:

Бекапы:
```bash
root@redos8-template ~
12:39:55 # ll /etc/*.backup
-rw-r--r-- 1 root root 298 сен 17 12:11 /etc/os-release.backup
-rw-r--r-- 1 root root  31 сен 17 12:11 /etc/redhat-release.backup
-rw-r--r-- 1 root root  31 сен 17 12:11 /etc/redos-release.backup
-rw-r--r-- 1 root root  31 сен 17 12:11 /etc/system-release.backup
```

Source file:
```bash
root@redos8-template ~
12:41:26 # ccat /etc/os-release.backup; ccat /etc/redhat-release.backup; ccat /etc/redos-release.backup; ccat /etc/system-release.backup
NAME="RED OS"
VERSION="8.0.2"
PLATFORM_ID="platform:red80"
ID="redos"
ID_LIKE="rhel centos fedora"
VERSION_ID="8.0.2"
PRETTY_NAME="RED OS 8.0.2"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:redos:redos:8"
HOME_URL="https://redos.red-soft.ru"
BUG_REPORT_URL="https://support.red-soft.ru"
EDITION="Standard"

RED OS release (8.0.2) MINIMAL
RED OS release (8.0.2) MINIMAL
RED OS release (8.0.2) MINIMAL
```

Dest file:
```bash
root@redos8-template ~
12:43:25 # ccat /etc/os-release; ccat /etc/redhat-release; ccat /etc/redos-release; ccat /etc/system-release
NAME="CentOS Linux"
VERSION="8"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="8"
PLATFORM_ID="platform:el8"
PRETTY_NAME="CentOS Linux 8"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:8"
HOME_URL="https://www.centos.org"
BUG_REPORT_URL="https://bugs.centos.org"
CENTOS_MANTISBT_PROJECT="CentOS-8"
CENTOS_MANTISBT_PROJECT_VERSION="8"
CentOS Linux release 8.0.2 (Core)
CentOS Linux release 8.0.2 (Core)
CentOS Linux release 8.0.2 (Core)
```

Аналогичные действия для  Astra Linux.
```bash
┌─ root
├─ alt11-server ~ ▶ ccat /etc/os-release; echo; ccat /etc/altlinux-release
NAME="ALT Server"
VERSION="11.0"
ID=altlinux
VERSION_ID=11.0
PRETTY_NAME="ALT Server 11.0 (Mendelevium)"
ANSI_COLOR="1;33"
CPE_NAME="cpe:/o:alt:server:11.0"
BUILD_ID="ALT Server 11.0"
ALT_BRANCH_ID="p11"
HOME_URL="https://basealt.ru/"
BUG_REPORT_URL="https://bugs.altlinux.org/"
DOCUMENTATION_URL="https://docs.altlinux.org/"
SUPPORT_URL="https://support.basealt.ru/"
VARIANT_ID=edition_server
VARIANT="ALT Server"
LOGO=alt-server-logo

ALT Server 11.0 (Mendelevium)

```
```bash
┌─ root
├─ alt11-server /etc ▶ ccat /etc/os-release; echo; ccat /etc/altlinux-release
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"

ALT Server 11.0 (Mendelevium)
```


После успешных действий, создать шаблоны:
```
pvesh get /nodes/pmx5/storage
vzdump 196 --storage home --mode stop --compress zstd --remove 0
 #или/и
vzdump 196 --storage ssd_1tb --mode stop --compress zstd --remove 0
 #или/и
vzdump 196 --compress gzip --dumpdir /home/template/cache
```

Зайти в контейнер и устанвоить\проверить сеть:
```bash
pct enter 888

# Полная настройка сети
ip addr flush dev eth0
ip addr add 192.168.87.131/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.87.1 dev eth0

# Проверяем
ip addr show eth0
ip route show
```
Создаем правильный network configuration (/etc/network/interfaces):
```bash
## /etc/network/interfaces
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
    address 192.168.87.131
    netmask 255.255.255.0
    gateway 192.168.87.1
    dns-nameservers 8.8.8.8 1.1.1.1
```


Полученные архивы поместить по пути: `/home/template/cache`

Однако, стоит иметь в виду, что ISO образы должны храниться в `/var/lib/vz/template/iso`,   
хотя можно и по пути: `/stg/ssd/template/iso`:

```plaintext

# Добавьте существующую директорию как хранилище
# В веб-интерфейсе: Datacenter → Storage → Add → Directory
# Укажите:
# - ID: например, `ssd-iso`
# - Directory: `/stg/ssd/template/iso`
# - Content: ISO images
```