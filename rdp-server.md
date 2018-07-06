# Организация терминального сервера (Remote Desktop Services)

## Что есть
- [xrdp]()
- [x2go]()
- [LTSP]()
- [...]()

## Исходные данные
- ОС: debian 9
- DE: lxde (можно выбрать установку сразу при установке ОС или доставить позже: sudo apt install lxde)
- Домен: contoso.com
- Учетная запись администратор домена: adadmin
- 2 DC: dc1 (192.1.1.1) и dc2 (192.1.1.2)
- имя сетевой карты: etho (посмотреть: ifconfig)

## Готовим сеть
в /etc/network/interfaces в конец добавить:
```
dns-nameservers 192.1.1.1 192.1.1.2
dns-search contoso.com
```

sudo ifdown eth0 && sudo ifup eth0

Проверка: "cat /etc/resolv.conf | grep nameserver" должен вернуть:
```
nameserver 192.1.1.1
nameserver 192.1.1.2
```

## Ставим xrdp
Чтобы использовать свежие пакеты, подключаем backports:
```
sudo echo "deb http://ftp.debian.org/debian/ stretch-backports main contrib non-free" > /etc/apt/sources.list.d/backports.list
sudo apt update 
sudo apt -t stretch-backports install xrdp -y
```

## Включаем в домен

Выполняем простенькую [инструкцию](join-to-domain.md)


