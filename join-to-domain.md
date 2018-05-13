Практически все возможные способЫ кратко, но емко описаны [здесь](https://rhelblog.redhat.com/2015/02/04/overview-of-direct-integration-options/).  
По большому счету есть 2 способа: через samba и через sssd (realm).  
Samba - проект с долгой историей и кучей возможностей, но он достаточно тяжел, не совсем тривиален в настройке и менее богат возможностями, чем realm.
Sssd - проект от RedHat, болеее свежий, более простой и более функциональный.  
Сравнение samba и sssd [здесь] (https://rhelblog.redhat.com/2015/04/02/sssd-vs-winbind/).  
Второй способ более современный и простой, и даже приведен в [официальной инструкции microsoft](https://docs.microsoft.com/ru-ru/sql/linux/sql-server-linux-active-directory-authentication?view=sql-server-linux-2017#join).
```
sudo apt install realmd krb5-user software-properties-common packagekit
sudo realm join contoso.com -U 'adadmin@CONTOSO.COM' -v
```
Если получили ошибку разрешения имени contoso.com: в /etc/nssswitch.conf для параметра hosts поставить первым методом "dns":
```
hosts: dns files ...
```
Если при входе через rdp получаем ошибку "login failed", при этом "su adadmin" отрабатывает успешно: в /etc/sssd/sssd.conf изменить параметр:
```
access_provider = simple
```

## Дополнения
Чтобы хомяк создавался автоматически: в /etc/pam.d/common-session-noninteractive в самый конец добавить:
```
session optional pam_mkhomedir.so
```
Полезные правки /etc/sssd/sssd.conf:
```
# Чтобы не вводить имя домена при авторизации
use_fully_qualified_names = False
# Чтобы хомяк создавался по адресу /home/contoso/<user>
fallback_homedir = /home/contoso/%u
# Если написать /home/%u@%d, то будет так: /home/user@contoso.com
```
