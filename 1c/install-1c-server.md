# Установка сервера 1С

Сервер ставим в контейнер  
Скачиваем deb-пакет сервера, устанавливаем:

```
VERSION="8.3.25.1394"
pkg -i 1c-enterprise-$VERSION-*
systemctl link /opt/1cv8/x86_64/$VERSION/srv1cv8-$VERSION@.service
systemctl enable srv1cv8-$VERSION@.default.service
systemctl start srv1cv8-$VERSION@.default.service
```

# Установка hasp:

```
#apt install make libc6-i386
wget https://download.etersoft.ru/pub/Etersoft/HASP/stable/x86_64/Debian/11/haspd_8.53-eter1debian_amd64.deb
dpkg -i haspd_8.53-eter1debian_amd64.deb
systemctl enable haspd
systemctl start haspd
```

# Проброс ключа в контейнер

Драйвер hasp содает ссылки на ключи в /dev/aks/hasp - это именно то место, где 1С будет их искать.  
Как оно должно выглядеть:

```
ls -l /dev/aks/hasp/
>total 0
>lrwxrwxrwx 1 root root 21 Nov 30 14:38 5-1 -> ../../bus/usb/005/002
```

Если бы мы ставили 1С на физическое железо и воткнули ключ, то так оно бы и было, но у нас контейнер.
Плюс ко всему у нас несколько нод, между которыми контейнер может ездить.
Поэтому, чтобы при миграции контейнера не перетыкать ключ руками, удобно купить какую-нибудь железку для проброса ключей по сети, либо раздавать его через virtualhere с любого пк.
После переезда и втыкания ключа нужно создать ссылку и пробросить его в контейнер

## Создание ссылки

Ссылку можно сделать руками:

```
lsusb
> Bus 006 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
> Bus 005 Device 002: ID 0529:0001 Aladdin Knowledge Systems HASP copy protection dongle
> Bus 005 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
mkdir -p /dev/aks/hasp
ln -s /dev/bus/usb/005/002 /dev/aks/hasp/99-1
```

> 005/002 - шина и устройство из вывода lsusb, 99-1 - произвольный номер по такому шаблону, гллавное,

Но лучше процесс автоматизировать:  
Положить скрипт [link-aladdin](./scripts/link-aladdin) в /usr/local/bin/ (не забыть chmod +x)  
Положить [правило udev](./scripts/99-aladdin.rules) в /etc/udev/rules.d/  
Перезапустить udev:

```
wget https://raw.githubusercontent.com/rsyuzyov/work-on-linux/refs/heads/master/1c/scripts/link-aladdin -P /usr/local/bin
chmod +x /usr/local/bin/link-aladdin
wget https://raw.githubusercontent.com/rsyuzyov/work-on-linux/refs/heads/master/1c/scripts/99-aladdin.rules -P /etc/udev/rules.d/
udevadm control --reload-rules
udevadm trigger
```

В результате на любом хосте сразу при втыкании/пробрасывании ключа будет создаваться ссылка /dev/aks/hasp/99-1, которую и будем пробрасывать в контейнер

## Проброс в контейнер:

Через веб-интерфейс proxmox добавить проброс устройства:  
Add - Device passthrough: /dev/aks/hasp/99-1  
Либо в конфиг контейнера после ядер добавить:

```
dev3: /dev/aks/hasp/99-1
```

Контейнер готов, можно сделать шаблон (бэкап) и тиражировать.

# Клонирование контейнера

После клонирования шаблона удалить теукущий кластер:

```
rm -r /home/usr1cv8/.1cv8/1C/1cv8/reg_1541
rm /home/usr1cv8/.1cv8/1C/1cv8/1cv8wsrv.lst
```
