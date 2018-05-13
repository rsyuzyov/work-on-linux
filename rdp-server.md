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
echo "deb http://ftp.debian.org/debian/ stretch-backports main contrib non-free" > /etc/apt/sources.list.d/backports.list
sudo apt -t stretch-backports install xrdp
```

## Включаем в домен

В домен можно включать разными способами, о чем замечательно написано в блоге red hat: 
https://rhelblog.redhat.com/2015/02/04/overview-of-direct-integration-options/
https://rhelblog.redhat.com/2015/04/02/sssd-vs-winbinлоту winbind из пакета samba или утилиту realm из пакета sssd.  
Второй способ более современный и простой, и даже приведен в [официальной инструкции microsoft](https://docs.microsoft.com/ru-ru/sql/linux/sql-server-linux-active-directory-authentication?view=sql-server-linux-2017#join).

```
sudo apt install realmd krb5-user software-properties-common packagekit
sudo realm join contoso.com -U 'adadmin@CONTOSO.COM' -v
```
Если получили ошибку разрешения имени contoso.com: в /etc/nssswitch.conf для параметра hosts поставить первым методом "dns":
```
hosts: dns files ...
```
Если при входе через rdp получаем ошибку "login failed", при этом "su adadmin" отрабатывает успешно:
в /etc/sssd/sssd.conf [изменить параметр](https://github.com/neutrinolabs/xrdp/issues/906):
```
access_provider = simple
```

## Дополнения

Чтобы хомяк создавался автоматически: в /etc/pam.d/common-session-noninteractive в самый конец добавить:
```
session optional pam_mkhomedir.so
```

Можно изменить /etc/sssd/sssd.conf:
```
# Чтобы не вводить имя домена при авторизации
use_fully_qualified_names = False
# Чтобы хомяк создавался по адресу /home/contoso/<user>
fallback_homedir = /home/contoso/%u
# Если написать /home/%u@%d, то будет так: /home/user@contoso.com
```

